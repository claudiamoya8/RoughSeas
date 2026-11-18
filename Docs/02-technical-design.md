# Technical design

This document explains how the code is organised: which modules there are, what each class does, how they communicate and which design patterns we use and why. It is not a UML diagram: it is the guide we follow when programming.

- **Everything is in English:** classes, methods, variables, comments, asset and scene names, and the documentation.
- **It is a proposal:** it can change, but through a PR, logging the change in [decisions.md](decisions.md).

The game rules are in [The game](01-game.md), and who programs each module is in the [Plan](03-plan.md).

## 1. Guiding ideas

1. **Small modules with a single responsibility** (the SRP of SOLID). Each module is a folder and a namespace.
2. **Dependencies only go downwards.**
   - A module may use modules on lower levels, never the other way round.
   - When a lower module has to tell a higher one something, it raises an event (Observer pattern).
   - This way there are no circular references and no *spaghetti code*.
3. **Data lives in ScriptableObjects, not in code.**
   - Tuning numbers or adding a ship type or a level does not require touching classes (open/closed principle, OCP).
4. **Where there are variants, we program against an abstraction** (dependency inversion principle, DIP).
   - For example, the pirate ship does not know whether it is driven by a state machine or by a neural network.
5. **Patterns only where they help.**
   - Only where they solve a problem we actually have.
   - We avoid the anti-patterns from unit 6.3: *golden hammer*, *god object* and *spaghetti code*.
6. **Playing again means reloading the scene.** It is the simplest and most reliable way to make every match start clean.

## 2. Modules

All namespaces start with `RoughSeas.` (for example, `RoughSeas.Ships`). Each module has its own folder in `RoughSeas/Assets/_Project/Scripts/`.

| Level | Module | Responsibility | Uses | Owner |
|---|---|---|---|---|
| 0 | `Common` | Shared types: enumerations, basic interfaces and the generic object pool | — | Shared |
| 1 | `World` | Islands, ports and obstacles | `Common` | Claudia |
| 1 | `Routes` | Routes: points, smoothing, drawing them with the mouse and displaying them | `Common` | Rodrigo |
| 2 | `Ships` | Ships: common movement, merchant ships, docking, collisions and collision warning | `Common`, `World`, `Routes` | Rodrigo |
| 3 | `Pirates` | Pirate ship, cannon, cannonballs and the pirate's state machine | `Common`, `World`, `Ships` | Rodrigo |
| 4 | `AI` | ML-Agents agent that drives the pirate | `Pirates`, `Ships` | Rodrigo |
| 4 | `Spawning` | Creating ships and the pirate: factory, spawn rate and pools | `Common`, `World`, `Ships`, `Pirates` | Claudia |
| 5 | `Core` | The match: states, scoreboard, score, high scores and session | `Common`, `Ships`, `Spawning` | Claudia |
| 6 | `UI` | Menus, HUD and screens | `Core` | Claudia |
| 6 | `Training` | Pirate training scene | `AI`, `Spawning`, `Ships`, `Routes`, `World` | Claudia |

```
Level 6   UI          Training
Level 5   Core
Level 4   Spawning    AI
Level 3   Pirates
Level 2   Ships
Level 1   World       Routes
Level 0   Common

Downwards: method calls.   Upwards: events.
```

**Nobody uses `UI` or `Training`.** `Core` does not know about the UI: the UI listens to `Core` events and calls its public methods when a button is pressed.

## 3. The classes of each module

### Common

| Class | What it is |
|---|---|
| `IslandColor` | Enumeration: `Red`, `Blue`, `Yellow`. |
| `Difficulty` | Enumeration: `Easy`, `Hard`. |
| `IPoolable` | Interface for recyclable objects: `OnTakenFromPool()` and `OnReturnedToPool()`. |
| `ObjectPool<T> where T : Component, IPoolable` | Generic pool. `Get()` returns a deactivated object from storage, or creates one if there is none. `Release(item)` deactivates it and stores it for reuse. |

### World

| Class | What it is |
|---|---|
| `Island` | An island with its `IslandColor`. Its coast has a `Hazard`. |
| `Port` | A port: its `Color`, its `Kind` (`PortKind.Departure` or `PortKind.Arrival`) and its spawn point. `IsClear()` says whether no ship is blocking the exit. |
| `PortKind` | Enumeration: `Departure`, `Arrival`. |
| `Hazard` | Marks rocks, coasts and the reef at the edge. A merchant ship that touches it sinks; the pirate bounces off. |

### Routes

| Class | What it is |
|---|---|
| `Route` | A plain C# class, not a `MonoBehaviour`. It stores the list of points, smooths them (Chaikin's algorithm) and gives the next point to head for, discarding the ones already passed. Because it does not depend on Unity, it can be tested if we write tests. |
| `IRouteFollower` | Interface for anything that can follow a route: `Position`, `RouteColor` and `SetRoute(Route route)`. Implemented by `MerchantShip`. |
| `RouteDrawer` | Turns the mouse into routes. It reads the `Draw` and `Point` actions from the Input Actions. On press it selects the ship under the cursor; while dragging, it intersects the camera ray with the water plane and adds a point every so often; on release, it calls `SetRoute`. It only knows `IRouteFollower`, not the ship class. |
| `RouteView` | Draws what is left of the route with a `LineRenderer`, in the ship's colour. |

### Ships

| Class | What it is |
|---|---|
| `Ship` | Abstract class with what merchant ships and the pirate have in common: movement on the water with limited turning. Its `FixedUpdate` moves the ship, and each subclass decides where to go and how fast by overriding `GetDesiredHeading()` and `GetTargetSpeed()`. |
| `MerchantShip : Ship, IPoolable, IRouteFollower` | The merchant ship. It knows its `ShipType`, its destination colour and its route. With a route, it follows it; without one, it sails straight. It docks with `Dock()` when it enters the port of its colour and sinks with `Sink(SinkCause cause)`. It raises the `OnDelivered` and `OnSunk` events. |
| `ShipType` | ScriptableObject with what all ships of one type share: name, speed, turn rate, size and model. There are three assets: slow, normal and fast. |
| `SinkCause` | Enumeration: `ShipCollision`, `Hazard`, `Pirate`, `Cannonball`. |
| `CollisionWarning` | Checks whether the ship will collide with another merchant ship in less than 1.5 s if nothing changes and, if so, highlights it in red. |

### Pirates

| Class | What it is |
|---|---|
| `PirateShip : Ship` | The pirate ship. Every step it asks its `IPirateController` for a `PirateAction` and applies it. It does not sink: with obstacles it slows down and bounces off. It raises `OnRammedShip` and `OnHitObstacle`. |
| `IPirateController` | Interface of the pirate's "brain": `PirateAction DecideAction(PirateShip ship)`. |
| `PirateAction` | Struct with one step's decision: `Turn` (`TurnInput.Left`, `Straight`, `Right`), `Throttle` (`ThrottleInput.SlowDown`, `Keep`, `SpeedUp`) and `Fire` (`bool`). |
| `Cannon` | The cannon. `IsLoaded` says whether it is loaded. `TryFire()` looks for the nearest merchant ship in range, predicts where it will be and fires a cannonball from the pool. It raises `OnShotResolved(bool hit)` when the cannonball hits or is lost. |
| `Cannonball : MonoBehaviour, IPoolable` | The cannonball: it travels in a straight line and, if it touches a merchant ship, sinks it. |
| `PirateConfig` | ScriptableObject with the pirate's numbers: speeds, turn rate, range, reload time, cannonball speed and detection radius. |
| `StateMachinePirateController : MonoBehaviour, IPirateController` | The state-machine brain (State pattern). States: `PatrolState`, `ChaseState` and `AttackState`, which inherit from `PirateState`. |

### AI

| Class | What it is |
|---|---|
| `PirateAgent : Agent, IPirateController` | The trained brain. It works as an adapter (Adapter pattern) between ML-Agents and our game: it turns the ship's state into observations and the network's output into a `PirateAction`. It gives rewards by listening to `PirateShip` and `Cannon` events. |

### Spawning

| Class | What it is |
|---|---|
| `ShipFactory` | Merchant ship factory. `Create(ShipType type, Port departure, IslandColor destination)` takes a ship from the pool, applies its type and destination, places it at the port and raises `OnShipSpawned`. When the ship is delivered or sinks, it returns it to the pool. |
| `SpawnScheduler` | Decides when and where each ship appears: rate from 8 s down to 3 s, a random free departure port, a random type and a destination different from the departure island. It has `Begin()` and `Stop()`. |
| `PirateSpawner` | When the time comes (30 s), it creates the pirate from the prefab of the chosen difficulty. It has `Begin(Difficulty difficulty)`, `Stop()` and the `OnPirateSpawned` event. |

### Core

| Class | What it is |
|---|---|
| `GameManager` | Runs the match with the State pattern. `ChangeState(...)` changes state, `TogglePause()` pauses or resumes, `Replay()` reloads the scene and `GoToMainMenu()` goes back to the menu. It raises `OnPaused`, `OnResumed` and `OnMatchEnded(MatchResult result)`. |
| `GameState` | Abstract class for the match states: `PlayingState`, `PausedState` and `GameOverState`. |
| `MatchStats` | The scoreboard: it counts deliveries, sinkings and time by listening to events. It raises `OnChanged`, `OnVictory` and `OnDefeat`. |
| `MatchResult` | Struct with the result: whether the match was won, deliveries, sinkings, time and score. |
| `ScoreCalculator` | Static class with the score formula. |
| `HighScoreTable` | The 5 best scores, sorted. Plain C#. |
| `HighScoreStorage` | Saves and loads the high scores as JSON in `Application.persistentDataPath`. |
| `GameSession` | The game's only Singleton. It keeps the chosen difficulty when going from the menu to the match. `StartMatch(Difficulty difficulty)` stores it and loads the `Game` scene. If `Game` is opened directly in the editor, it creates itself with Easy difficulty. |
| `MatchConfig` | ScriptableObject with the match numbers and the pirate prefab for each difficulty. |

### UI

`MainMenuView`, `HudView`, `PauseMenuView`, `EndScreenView` and `HighScoresView`, built with uGUI (Canvas) and TextMeshPro. They only show data and forward button presses to `Core`: no game logic.

### Training

| Class | What it is |
|---|---|
| `TrainingArea` | A copy of the map with its own pirate and merchant ships. It starts and ends training episodes and resets everything. |
| `RandomRouteDriver` | Gives the training merchant ships random routes and sometimes changes them, so the pirate does not learn that ships always sail straight. |

## 4. How modules communicate: events

Lower modules notify higher ones with C# events (`event Action<…>`), as in unit 6.3 (pattern 9.2, *Event-Driven Architecture*). Whoever raises an event does not know who is listening.

| Who notifies | Event | Who listens | What for |
|---|---|---|---|
| `ShipFactory` | `OnShipSpawned(MerchantShip ship)` | `MatchStats` | Subscribe to that ship's events |
| `MerchantShip` | `OnDelivered(MerchantShip ship)` | `MatchStats`, `ShipFactory` | Count a delivery and return the ship to the pool |
| `MerchantShip` | `OnSunk(MerchantShip ship, SinkCause cause)` | `MatchStats`, `ShipFactory` | Count a sinking and return the ship to the pool |
| `Cannon` | `OnShotResolved(bool hit)` | `PirateAgent` | Reward for hitting or missing |
| `PirateShip` | `OnRammedShip`, `OnHitObstacle` | `PirateAgent` | Rewards for ramming or crashing |
| `PirateSpawner` | `OnPirateSpawned(PirateShip pirate)` | (in version 1.0, sound or a HUD warning) | Announce the pirate's arrival |
| `MatchStats` | `OnChanged(MatchStats stats)` | `HudView` | Update the scoreboard |
| `MatchStats` | `OnVictory`, `OnDefeat` | `GameManager` | Switch to `GameOverState` |
| `GameManager` | `OnPaused`, `OnResumed` | `PauseMenuView` | Show or hide the pause screen |
| `GameManager` | `OnMatchEnded(MatchResult result)` | `EndScreenView` | Show victory or defeat |

Rules for events:

- They are declared as `public event Action<…> OnSomething;` and raised with `OnSomething?.Invoke(...)`.
- An event can only be raised from the class that declares it. If a state needs to raise one of `GameManager`'s events, it calls a `GameManager` method that raises it.
- Whoever subscribes with `+=` in `OnEnable` unsubscribes with `-=` in `OnDisable`.
- With pooled ships, `MatchStats` unsubscribes from each ship when it is delivered or sinks. Otherwise, when the ship is reused, it would be counted twice.

It is the same scheme as the Unity example from class (the factory that notifies when it creates a cube, and the flamingo that subscribes to each cube):

```csharp
// Ships/MerchantShip.cs
public class MerchantShip : Ship, IPoolable, IRouteFollower
{
    public event Action<MerchantShip> OnDelivered;
    public event Action<MerchantShip, SinkCause> OnSunk;

    public void Dock()
    {
        OnDelivered?.Invoke(this);
    }
}

// Core/MatchStats.cs
public class MatchStats : MonoBehaviour
{
    [SerializeField] private ShipFactory shipFactory;

    private void OnEnable() => shipFactory.OnShipSpawned += HandleShipSpawned;
    private void OnDisable() => shipFactory.OnShipSpawned -= HandleShipSpawned;

    private void HandleShipSpawned(MerchantShip ship)
    {
        ship.OnDelivered += HandleShipDelivered;
        ship.OnSunk += HandleShipSunk;
    }

    private void HandleShipDelivered(MerchantShip ship)
    {
        Unsubscribe(ship);
        // Count the delivery, raise OnChanged and check for victory.
    }
}
```

## 5. Design patterns

The numbers are the ones from unit 6.3. **State and Observer are the main ones**, as stressed in class.

| Pattern | Where | What problem it solves |
|---|---|---|
| **10. State** | `GameManager` (playing, paused, game over) and the pirate's brain (patrol, chase, attack) | Without it, the match and the pirate would be a tangle of `if`s and booleans (`isPaused`, `isChasing`…). With it, each state is a class, transitions are clear and each state sets up and undoes its own things on entering and leaving. |
| **9. Observer / events** | Ships, factory, scoreboard, match and UI (table in section 4) | Ships do not know about the scoreboard or the UI: they just notify. New listeners (sound, statistics) can be added without touching whoever notifies. |
| **8. Strategy** | `IPirateController`: state machine or ML-Agents agent | Swap the pirate's brain without touching `PirateShip`. The difficulty only decides which brain it gets. |
| **2. Factory** | `ShipFactory` | Ship creation lives in one place. A new type is a new asset, not a new `if`. |
| **6. Flyweight** | `ShipType` and `PirateConfig` (ScriptableObjects) | The data shared by a type is stored once and used by all its ships. Each ship only stores its own state: position, route and destination. |
| **7. Command** | Unity Input Actions (`Draw`, `Point`, `Pause`), as in the class example | The code does not ask about keys or buttons but about actions. Changing the controls does not touch the logic. |
| **1. Singleton** | Only `GameSession` | One piece of data (the difficulty) has to survive the scene change. We use a single one, because overusing it leads to a *god object*. |
| **Adapter** (mentioned in class) | `PirateAgent` | Adapts the ML-Agents interface to our `IPirateController`. |
| **Object Pool** (not in the unit) | Ships and cannonballs, with `ObjectPool<T>` | Avoids creating and destroying objects all the time. It complements the factory. |
| Inheritance | `Ship` → `MerchantShip`, `PirateShip` | The common code is written once. Each subclass only decides where to go and how fast. |

**We do not use Bridge, Decorator or Facade:** right now they do not solve any problem in the game. If one shows up, we propose it.

### State in detail

We follow the structure of the dinosaur example from class: an abstract state class with `OnStateEnter` and `OnStateExit`, and a context that changes state by leaving the current one, storing the new one and entering it.

**Match states** (the context is `GameManager`):

| State | On enter | On exit | Goes to |
|---|---|---|---|
| `PlayingState` | `Time.timeScale = 1`. The first time, starts `SpawnScheduler` and `PirateSpawner`. | — | `PausedState` with Esc or the pause button; `GameOverState` on victory or defeat |
| `PausedState` | `Time.timeScale = 0` and raises `OnPaused` | `Time.timeScale = 1` and raises `OnResumed` | `PlayingState` with Esc or the button |
| `GameOverState` | Stops ship spawning and the pirate, calculates the score, saves the high score and raises `OnMatchEnded` | — | Play again reloads the scene; Menu loads `MainMenu` |

```csharp
// Core/GameState.cs
public abstract class GameState
{
    protected readonly GameManager game;

    protected GameState(GameManager game)
    {
        this.game = game;
    }

    public virtual void OnStateEnter() { }
    public virtual void OnStateExit() { }
    public virtual void Tick(float deltaTime) { }
}

// Core/GameManager.cs
public void ChangeState(GameState newState)
{
    if (newState == null || newState == currentState)
    {
        return;
    }

    currentState?.OnStateExit();
    currentState = newState;
    currentState.OnStateEnter();
}
```

**Pirate states** (the context is `StateMachinePirateController`):

| State | What it does | Goes to |
|---|---|---|
| `PatrolState` | Follows a set of patrol points near the straits | `ChaseState` if it detects a merchant ship within its radius |
| `ChaseState` | Heads for the nearest merchant ship | `AttackState` if the ship is in range; `PatrolState` if it loses it |
| `AttackState` | Stays close and fires when the cannon is loaded | `ChaseState` if the ship leaves its range |

Here, as in the class example, each state returns the next state, or `null` to stay in the same one:

```csharp
// Pirates/PirateState.cs
public abstract class PirateState
{
    public virtual void OnStateEnter(StateMachinePirateController brain) { }
    public virtual void OnStateExit(StateMachinePirateController brain) { }

    // Decides this step's action and returns the next state, or null to stay.
    public abstract PirateState Tick(StateMachinePirateController brain, out PirateAction action);
}
```

## 6. A match, step by step

1. In `MainMenu` you choose the difficulty. `GameSession.StartMatch(difficulty)` stores it and loads the `Game` scene.
2. `GameManager` enters `PlayingState`, which starts `SpawnScheduler` and `PirateSpawner`.
3. `SpawnScheduler` asks `ShipFactory` for ships. The factory raises `OnShipSpawned`, and `MatchStats` subscribes to that ship.
4. The player draws a route: `RouteDrawer` creates a `Route` and gives it to the ship with `SetRoute`.
5. The ship enters its port: `Dock()` raises `OnDelivered`. `MatchStats` counts the delivery and notifies the HUD with `OnChanged`, and the factory returns the ship to the pool.
6. At 30 s, `PirateSpawner` creates the pirate from the prefab for the chosen difficulty. Every step, the pirate asks its brain what to do.
7. With 20 deliveries or 3 sinkings, `MatchStats` raises `OnVictory` or `OnDefeat`. `GameManager` switches to `GameOverState`, which raises `OnMatchEnded`, and `EndScreenView` shows the result.
8. Play again reloads the `Game` scene (with `Time.timeScale` back at 1). Menu loads `MainMenu`.

## 7. The pirate AI

### State machine (version 0.2)

This is `StateMachinePirateController` (section 5). It is used:

- as the pirate in version 0.2;
- as plan B if training fails;
- as the agent's heuristic: ML-Agents lets us drive the agent with our own code, so the training scene can be tested before training.

### ML-Agents agent (version 0.3)

- **Tools:** the `com.unity.ml-agents` package (4.x) in Unity and, in Python 3.10.12, `mlagents` 1.1.0, installed with uv. Task T04 checks that they work in Unity 6000.6.8f1 and logs the versions in decisions.md.
- **Observations:** all normalised between −1 and 1, and relative to the pirate. That way what it learns is valid wherever it is, and also on other maps.

  | Observation | Values |
  |---|---|
  | Own velocity, in local axes, divided by the maximum | 2 |
  | Own position on the map, divided by the map size | 2 |
  | Whether the cannon is loaded | 1 |
  | For each of the 3 nearest merchant ships within the detection radius: whether it exists, its relative position in local axes and its relative velocity | 3 × 5 = 15 |
  | **Total** | **20** |

  In addition, a 3D ray sensor (`RayPerceptionSensor3D`) detects islands, rocks, the reef and merchant ships, each with its own tag.
- **Actions:** three discrete branches: turn (3 options), speed (3) and fire (2).
- **Starting rewards:**

  | Situation | Reward |
  |---|---|
  | Sinks a ship, with a cannonball or by ramming | +1 |
  | Misses a shot | −0.1 |
  | Hits a rock, a coast or the reef | −0.5 |
  | Every step, so it does not stand still | −0.0005 |

- **Training:**
  - `Training` scene with 8 to 16 copies of `TrainingArea` and accelerated time;
  - PPO algorithm;
  - the configuration in `Training/config/pirate.yaml` (at the root of the repository, not inside Unity);
  - results saved in `Training/results/`, which is not uploaded to GitHub;
  - only the chosen models are copied to `Assets/_Project/AI/Models/`.
- **Difficulty:** the `PirateShip_Easy` prefab uses a model from an early checkpoint and `PirateShip_Hard` the final model. If training fails, both prefabs use the state machine with different parameters.

## 8. Data (ScriptableObjects)

They are in `Assets/_Project/Data/`.

| Asset | Class | What it stores |
|---|---|---|
| `MatchConfig` | `MatchConfig` | Target (20) and sinking limit (3), spawn rate, pirate arrival, collision warning, score, number of high scores and the pirate prefab for each difficulty |
| `ShipType_Slow`, `ShipType_Normal`, `ShipType_Fast` | `ShipType` | Name, speed, turn rate, size and model |
| `PirateConfig` | `PirateConfig` | Speeds, turn rate, cannon range and reload time, cannonball speed and detection radius |

When there are levels, a `LevelConfig` will be added for each one.

## 9. Folders, scenes and prefabs

```
RoughSeas/Assets/
├── _Project/
│   ├── Scripts/      Common/ World/ Routes/ Ships/ Pirates/ AI/ Spawning/ Core/ UI/ Training/
│   ├── Prefabs/      Ships/ Pirates/ World/ UI/
│   ├── Scenes/       MainMenu, Game, Training
│   │   └── Sandbox/  Sandbox_Rodrigo, Sandbox_Claudia
│   ├── Data/         MatchConfig, ShipType_*, PirateConfig
│   ├── Input/        GameControls (Input Actions)
│   ├── Materials/
│   └── AI/Models/    PirateEasy.onnx, PirateHard.onnx
└── ThirdParty/       downloaded assets (Kenney, Asset Store)
```

- **Scenes:** `MainMenu`, `Game` and `Training`, plus a test scene for each of us in `Sandbox/`.
- **Main prefabs:** `MerchantShip` (a single one; the factory applies the `ShipType`), `PirateShip_Easy`, `PirateShip_Hard`, `Cannonball`, `Island`, `Port_Departure`, `Port_Arrival`, `Rock` and the UI prefabs.

## 10. Technical decisions

- **Physics:**
  - ships have a kinematic `Rigidbody` and *trigger* colliders;
  - they move in `FixedUpdate` on the XZ plane;
  - collisions are detected with `OnTriggerEnter`;
  - no "real" physics (forces, waves).
- **Mouse:** the camera ray is intersected with a mathematical `Plane` at water level (y = 0), not with colliders.
- **Input:** the new Input System, with an Input Actions asset (`GameControls`).
- **UI:** uGUI (Canvas) and TextMeshPro.
- **Pause:** `Time.timeScale = 0`.
- **Restart:** reload the scene.
- **High scores:** a JSON file in `Application.persistentDataPath`.
- **References:** through the inspector or through events. Never `GameObject.Find` or `FindObjectOfType` during the match.

## 11. Ready for levels

- **`LevelConfig`:** each level will have its own map scene, number of pirates, spawn rate and target.
- **Several pirates:** `PirateSpawner` is designed to create more than one.
- **Other maps:** the agent observes in relative coordinates, so its model can work on them. To help it generalise, the training scene can vary the rocks and straits.

## 12. Code conventions

- **Language:** everything in English: classes, methods, variables, comments, file names, assets, scenes and prefabs.
- **Files:** one class per file, with the same name. Namespace `RoughSeas.<Module>`, matching the folder.
- **Casing:**
  - `PascalCase` for classes, methods, properties, events and enumerations;
  - `camelCase` for local variables, parameters and private fields;
  - interfaces start with `I`: `IPirateController`.
- **Inspector:** fields edited in the inspector are `[SerializeField] private`. No public fields: if another class needs to read a value, use a `{ get; }` property.
- **Events:** `public event Action<…> OnSomething;`. Raised with `?.Invoke`, and every subscriber unsubscribes.
- **Comments:** only when they explain the why. A `///` summary on important public classes and methods.
- **Clean code:** no dead or commented-out code (Git keeps the history) and no classes that do everything.
