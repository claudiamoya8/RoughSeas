# The game

Living document: it describes what we have decided so far. If something changes, update it here in the same PR and log it in [decisions.md](decisions.md).

- How it is built: [Technical design](02-technical-design.md).
- Who does what and when: [Plan](03-plan.md).

## In one sentence

**Rough Seas** (*Mar Revuelto* in Spanish): you guide merchant ships between three islands by drawing their routes with the mouse, so they reach their port without crashing, while a pirate ship controlled by an AI trained with ML-Agents tries to sink them with its cannon.

Inspiration: route-drawing games such as *Harbor Master* or *Flight Control*. What makes ours different is the trained pirate and a map with narrow straits.

## What a match looks like

The sea starts empty. After a few seconds the first ship leaves a departure port, flying the flag colour of the island it is heading to. You click on it and, without releasing the button, drag: you are drawing its route on the water. When you release, the ship follows the route to its port and disappears. That is one ship delivered.

More and more ships appear, closer and closer together, some slow and some fast. Their routes start to cross, especially in the straits, and you have to work out when each ship will pass. There is no stop button: if a ship needs to wait, you draw it a detour.

After 30 seconds the pirate arrives. It looks for ships to sink with its cannon or by ramming them, and your only defence is to change routes in time.

You win by delivering 20 ships and lose when 3 ships sink. At the end you see your score and can play again or go back to the menu.

## What the assignment asks for

| Requirement | How we meet it |
|---|---|
| Unity 6000.6.8f1 | We both use that exact version. |
| Closed game loop: play, win or lose, play again, with a correct reset | The match is a state machine (playing, paused, game over), and "play again" reloads the scene, so every match starts clean. |
| An AI that makes decisions to make the player lose | The pirate: first with a finite-state machine, then trained with ML-Agents. |
| Code quality and complexity, OOP and design patterns (the most important part) | Modular design with State and Observer as the main patterns, plus Strategy, Factory and others where they are useful. All of it is in the [technical design](02-technical-design.md). |
| UML design of the main systems, detailing the patterns (deliverable 1) | To be done later, based on the technical design. |
| Builds and runs on PC or Mac without crashes | A Windows build tested on the other person's computer at the end of every version. If it crashes, it is not graded. |
| Bonus: an AI trained by us with ML-Agents, with the right information about its environment | A pirate trained with ML-Agents that observes ships in relative, normalised coordinates, justified in the technical design. |
| Art and sound are valued | We use existing free assets, added at the end on top of a game that already works. |

## Deliverables

| Deliverable | Date | File | Contents |
|---|---|---|---|
| 1 | Thursday 3/12/2026 | `1_RodrigoSicilia_ClaudiaMoya_RoughSeas_26_27.zip` | Project description (goal, how it works and overall structure) and UML design of the architecture, detailing the patterns. |
| 2 | Sunday 10/01/2027 | `2_RodrigoSicilia_ClaudiaMoya_RoughSeas_26_27.zip` | A folder with the game built for PC. Another with the Unity project without `Library`, `Temp`, `UserSettings`, `Logs` or `Obj`. Another with the training `.yaml` (and the training project, if it were a separate one). |

## Rules

### The map

- 3D view with a fixed perspective camera looking diagonally down at the sea. The player cannot move it. Everything happens on the water plane, so the logic is practically 2D.
- A single hand-made map: a rectangle of sea with:
  - **3 islands**: red, blue and yellow;
  - **2 ports per island**:
    - a **departure** port, where ships appear. It faces open sea: a ship leaving in a straight line takes at least 5 s to hit anything;
    - an **arrival** port, the destination of ships flying that island's colour;
  - scattered **rocks**;
  - **a reef all around the edge**: a ship that reaches the edge sinks;
  - **2 or 3 straits** between islands and rocks, where only one ship fits. They are shortcuts, but also where crashes are most likely.

### Merchant ships

| Type | Starting speed | Look |
|---|---|---|
| Slow | 1 | Large cargo ship |
| Normal | 1.5 | Sailing ship |
| Fast | 2.2 | Speedboat |

- Each ship flies the flag colour of its destination island and appears at the departure port of **another** island.
- Without a route, it sails straight ahead until the player draws one or until it crashes. It never turns around on its own.
- If it enters the arrival port of its colour, it disappears and counts as delivered, whether or not it was following a route. If it passes through a port of another colour, it does not dock and keeps going.
- When two ships will collide in less than 1.5 s if nothing changes, both are highlighted in red.

### Drawing routes (the core mechanic)

- Left-click on a ship and drag without releasing: a line is drawn on the water, in the ship's colour.
- On release, the ship follows the line, smoothing the curves, and the line is erased as the ship advances.
- If the line ends inside the correct arrival port, the ship docks when it gets there. If it ends anywhere else, the ship sails straight on from the end of the line.
- A ship's route can be redrawn at any time; the new one replaces the old one.
- Time does not stop while you draw.
- There is no stop button: to make a ship wait, draw it a detour.

The challenge: with many ships, routes have to cross, especially in the straits. Drawing well is not enough: you have to work out **when** each ship passes each point, and that is why the different speeds matter.

### Collisions and sinking

| A merchant ship hits… | What happens | Counts as |
|---|---|---|
| Another merchant ship | Both sink | 2 sunk |
| A rock, an island's coast or the reef at the edge | It sinks | 1 sunk |
| The pirate ship | The merchant ship sinks | 1 sunk |
| A pirate cannonball | It sinks | 1 sunk |

The pirate never sinks. If it hits a rock, a coast or the reef, it slows down and bounces off.

### The pirate ship

- A black ship that enters the map 30 s into the match, from a fixed point on the edge.
- It moves freely: it does not follow routes, the player does not control it and it cannot be sunk.
- **Cannon:** it only fires when a merchant ship is very close, at most 3 units away. It aims on its own at the nearest merchant ship within that range, predicting where it will be if it keeps going straight. Reloading takes 3 s.
- **Cannonballs:** they travel in a straight line and take time to arrive. If the ship changes course, they can miss.
- **How the player defends:** only by redrawing routes, to keep ships away from the pirate or change their course at the last moment.

### The pirate AI

The pirate has two interchangeable "brains". The game picks one depending on the difficulty:

- **Finite-state machine (version 0.2, and plan B):** it patrols near the straits, chases the nearest merchant ship it detects and fires when it is in range.
- **Agent trained with ML-Agents (version 0.3):** it learns by itself what to do. It sees its own velocity, the relative position and velocity of the 3 nearest merchant ships and whether the cannon is loaded, and it "sees" obstacles and ships with ray sensors. Every step it decides whether to turn, slow down or speed up, and whether to fire. It earns points for sinking ships and loses points for missing shots, for crashing and for standing still.
- **Difficulty:**
  - **Easy:** a model saved early in training.
  - **Hard:** the final model.
  - **If training goes badly:** both difficulties use the state machine, with different parameters.

What we hope it learns: to lie in wait near the straits, because that is where ships have to pass.

The technical details are in the [technical design](02-technical-design.md).

### The match

- **Main menu:** Play (choosing Easy or Hard), High scores and Quit.
- **Controls:** left-click and drag to draw routes; Esc to pause.
- **On screen:** ships delivered (x/20), ships sunk (x/3), time and a pause button.
- **Ship spawning:** at first, one every 8 s. Every 30 s the interval drops by 1 s, down to a minimum of 3 s. The departure port and the ship type are chosen at random. If the chosen port is blocked, the ship waits until it is clear.
- **Victory:** deliver 20 ships.
- **Defeat:** 3 ships sunk.
- **Score:**
  - 100 points per ship delivered.
  - If you win, a bonus of 5 points for every second under 6 minutes. For example, winning in 4 minutes gives 20 × 100 + 120 × 5 = 2600 points.
  - If you lose, only the delivered ships count.
- **High scores:** the 5 best scores (wins and defeats) are saved, with date, difficulty and result. No name is asked for.
- **End of the match:** a victory or defeat screen with the score, and buttons to play again or go to the menu.
- **Restart:** playing again starts a new match from scratch: no ships, no cannonballs, no pirate and the scoreboard at zero.

## Starting values

All numbers are **starting values** that we will tune by playing. They are not written in the code: they live in data assets (ScriptableObjects), so they can be changed from the Unity inspector.

| What | Starting value | Where it is tuned |
|---|---|---|
| Ships to deliver to win | 20 | `MatchConfig` |
| Ships sunk to lose | 3 | `MatchConfig` |
| Spawn interval | starts at 8 s, drops 1 s every 30 s, minimum 3 s | `MatchConfig` |
| Pirate arrival | at 30 s | `MatchConfig` |
| Collision warning | 1.5 s before | `MatchConfig` |
| Score | 100 per delivery; bonus of 5 per second under 360 s | `MatchConfig` |
| High scores kept | 5 | `MatchConfig` |
| Merchant ship speed | slow 1, normal 1.5, fast 2.2 | one `ShipType` per type |
| Pirate cannon | range 3, reload 3 s, cannonball at 6 units/s | `PirateConfig` |

## Visual style

We do not make our own art.

- **First:** Unity's basic shapes (cubes, cylinders and flat colours), so the game works as soon as possible.
- **Once it works:** we replace those shapes with existing, free models, textures, sounds and UI.
- **Sources, in order of preference:**
  1. Kenney's packs (for example, the *Pirate Kit*), licensed CC0: they can be used without restrictions.
  2. Free assets from the Unity Asset Store, after checking their licence. If we use any, the repository has to stay private.
- **Animations:** simple ones, such as ships rocking, sinking or firing. Done in code or with the ones that come with the assets.

## From simple to complex

We start with something very simple and add difficulty step by step, both to the game and to the code. **Each version must be playable** before moving on to the next.

| Version | What it adds | Look |
|---|---|---|
| 0.1 | Map, ships, routes, collisions, deliveries, ship spawning, score, victory, defeat and play again. No pirate. | Basic shapes |
| 0.2 | Pirate with a state machine, cannon and cannonballs. Main menu with difficulty, and pause. | Basic shapes |
| 0.3 | Pirate trained with ML-Agents, with Easy and Hard. | Basic shapes |
| 1.0 | High scores, existing assets, sound and simple animations. This is the version we deliver. | Existing assets |
| Later | Levels, if there is time. | Existing assets |

The concrete steps of each version are in the [plan](03-plan.md).

## Later: levels

Once the base game is finished, we want to add a few levels (not endless ones), for example with more pirates or more islands. We start with a single level, but the design is already prepared for more:

- The match data lives in data assets. Adding a level means adding data and a scene, not rewriting code.
- The pirate observes the world in coordinates relative to itself, so its model can work on other maps. Even so, we will train it on somewhat varied maps.

Only if there is time left, and after agreeing on it: wind that speeds ships up or slows them down depending on their direction (it never pushes them off their route).

## Open questions

Until something else is decided, the "For now" column applies.

| Question | For now |
|---|---|
| Does a crash between two merchant ships count as 2 sunk or as 1? | It counts as 2, as in the original idea. It can be changed in `MatchConfig`. |
| Do we write automated tests for the pure logic (routes, spawning rate, score, high scores)? | Optional: we decide when version 0.1 is finished. |
| When and how do we make the UML for deliverable 1? | Later, based on the technical design. |
| How many hours per week, and which weeks have exams? | Unknown; the schedule has some slack. Adjust it when we know. |
