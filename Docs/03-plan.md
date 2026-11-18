# Plan

This document says who does what, in what order and when.

## Who does what

Each of us owns some modules of the [technical design](02-technical-design.md). The owner programs them and the other person reviews them. That way we both have our own commits and reviews, and parts of the code we can explain.

| | Rodrigo | Claudia |
|---|---|---|
| In one sentence | Everything that sails: ships, routes and the pirate | Everything around the ships: the world, the match and the UI |
| Modules | `Routes`, `Ships`, `Pirates`, `AI` | `World`, `Spawning`, `Core`, `UI`, `Training` |
| Shared parts | Creates `Common` and the Input Actions (task T05) | Programs `ObjectPool<T>` in `Common` (task T11) |
| Scenes | `Sandbox_Rodrigo` | `Game`, `MainMenu`, `Training` and `Sandbox_Claudia` |
| ML-Agents | The agent, its rewards, the `.yaml` and the training runs | The training scene |
| Visual assets | Ships, flags, pirate, cannonballs and effects | Islands, water, sky, UI and sound |
| Deliverable 1 | UML | Project description and building the zip |
| GitHub | Uploads the documentation in the first PR | Created the repository and owns it |

### Rules

- **Modules:** each person only changes their own. If you need something in the other person's module (for example, a new event), open an issue for the owner, or agree on it first and mention it in the PR.
- **Shared parts:** `Common`, the Input Actions, `ProjectSettings/` and `Packages/` are changed in small PRs that the other person reviews carefully.
- **Scenes:** each one has an owner (table above). Rodrigo tests his work in `Sandbox_Rodrigo`. When he needs to add something to `Game`, for example the `RouteDrawer` or the pirate prefabs, he does it in a small PR, tells Claudia, and never while another open PR touches `Game`.

### Responsibilities of both

- Decide the design and what gets accepted.
- Test every change by playing before approving it.
- Review the other person's PR and comment in your own words.
- Understand every class and every pattern: we can be asked about any of them.
- Do by hand in the editor what works best by hand: placing the map by eye, adjusting the camera and judging whether the game is fun.

## From simple to complex

We move forward in versions (the ones in [The game](01-game.md)), and each version in small milestones. **At the end of each milestone the game can be opened and tested**, even if things are still missing.

| Version | Milestone | What you can do when it is done |
|---|---|---|
| 0.1 | A. A ship at sea | See the map and a ship sailing straight |
| 0.1 | B. Drawing routes | Draw a ship's route and have it follow it |
| 0.1 | C. Arrive or sink | Deliver ships to their port, or sink them in collisions |
| 0.1 | D. Ships keep coming | Watch ships appearing more and more often |
| 0.1 | E. A complete match | Win, lose, see the score and play again |
| 0.2 | F. The pirate | Have the pirate patrol, chase and fire, and choose the difficulty in the menu |
| 0.3 | G. The trained pirate | Play against the trained model on Easy and on Hard |
| 1.0 | H. Polish | Play with real assets, sound and high scores |

## Tasks

Each task becomes a GitHub issue using the task template (user story, acceptance criteria and *Definition of Done*). The issue title starts with the task code, for example "T08 · Route drawing". The branch uses the number GitHub gives the issue, not the task code.

The tasks at the top unblock the ones below: do not start a task until the tasks it depends on are merged.

### Phase 0: setup

| Code | Task | Owner | Depends on |
|---|---|---|---|
| T01 | Create the `RoughSeas` repository on GitHub, configure it and invite Rodrigo (done; merge settings still to check) | Claudia | — |
| T02 | Upload the documentation in a PR, replacing the initial brief (`Docs/game_brief.md`) | Rodrigo | T01 |
| T03 | Base Unity project: 6000.6.8f1, Universal 3D template, Pipeline package, `_Project/` folders and empty scenes (`MainMenu`, `Game`, `Training`, `Sandbox_Rodrigo`, `Sandbox_Claudia`) | Rodrigo | T02 |
| T04 | ML-Agents test in a separate throwaway project: install the package and Python with uv, train an example for a few minutes and log the working versions in decisions.md | Claudia | Unity 6000.6.8f1 installed |
| T05 | Shared contracts, so we can work in parallel: `Common` (enumerations and `IPoolable`), the `GameControls` Input Actions, and the classes with their events and interfaces but no logic yet (`MerchantShip`, `ShipFactory`, `IRouteFollower`, `IPirateController`, `PirateAction`, `ShipType`, `MatchConfig`) | Rodrigo (Claudia reviews) | T03 |

### Version 0.1: the base game, without the pirate

| Code | Task | Owner | Depends on |
|---|---|---|---|
| T06 | Milestone A. Test map in `Game` with basic shapes: water, 3 islands, 6 ports, rocks, reef, 2 or 3 straits and a fixed camera (`World` module) | Claudia | T05 |
| T07 | Milestone A. `Ship` and `MerchantShip` sailing straight, with the three `ShipType` assets | Rodrigo | T05 |
| T08 | Milestone B. `Route`, `RouteDrawer` and `RouteView`: draw, smooth and follow the route, erase it as the ship advances and allow redrawing it | Rodrigo | T07 |
| T09 | Milestone C. Docking at the port of the right colour (`Dock` and `OnDelivered`) | Rodrigo | T06, T07 |
| T10 | Milestone C. Collisions and sinking (with another merchant ship, obstacles and the reef) and the red warning | Rodrigo | T06, T07 |
| T11 | Milestone D. `ObjectPool<T>`, `ShipFactory` and `SpawnScheduler`: rate from 8 to 3 s, only from free ports | Claudia | T07 |
| T12 | Milestone E. `GameManager` with its states, `MatchStats`, score, pause with Esc, victory, defeat and play again | Claudia | T11 |
| T13 | Milestone E. HUD, pause screen and end screen (uGUI) | Claudia | T12 |
| T14 | Windows build of 0.1, tested on the other person's computer | Both | T08 to T13 |

### Deliverable 1

| Code | Task | Owner | Depends on |
|---|---|---|---|
| T15 | Project description (goal, how it works and overall structure), based on The game and the Technical design | Claudia | — |
| T16 | UML of the architecture, detailing the patterns, when we decide to make it | Rodrigo, with Claudia | T05 |
| T17 | Review together and build the deliverable 1 zip | Both | T15, T16 |

### Version 0.2: the pirate with a state machine

| Code | Task | Owner | Depends on |
|---|---|---|---|
| T18 | Milestone F. `PirateShip`, `IPirateController`, `PirateAction` and `PirateConfig`: a pirate that moves with a test brain | Rodrigo | T07 |
| T19 | Milestone F. `Cannon` and `Cannonball` with a pool: predictive aiming, short range and reload | Rodrigo | T11, T18 |
| T20 | Milestone F. `StateMachinePirateController`: patrol, chase and attack | Rodrigo | T19 |
| T21 | Milestone F. `PirateSpawner` (at 30 s and depending on the difficulty), `GameSession` and a simple main menu with Easy and Hard | Claudia | T12, T18 |
| T22 | Build of 0.2, tested on the other person's computer | Both | T20, T21 |

### Version 0.3: the trained pirate

| Code | Task | Owner | Depends on |
|---|---|---|---|
| T23 | Milestone G. `Training` scene: `TrainingArea` with 8 to 16 copies, `RandomRouteDriver` and episode reset | Claudia | T11, T20 |
| T24 | Milestone G. `PirateAgent`: observations, actions, rewards and the state machine as heuristic | Rodrigo | T20 |
| T25 | Milestone G. `Training/config/pirate.yaml`, PPO training runs and choosing the Easy and Hard models | Rodrigo | T23, T24 |
| T26 | Milestone G. Put the models in the `PirateShip_Easy` and `PirateShip_Hard` prefabs | Rodrigo | T21, T25 |

### Version 1.0: polish

| Code | Task | Owner | Depends on |
|---|---|---|---|
| T27 | Milestone H. High scores: `HighScoreTable`, `HighScoreStorage` and their screen | Claudia | T12 |
| T28 | Milestone H. Existing assets for ships, flags, pirate, cannonballs and effects | Rodrigo | T10, T20 |
| T29 | Milestone H. Existing assets for islands, water, sky and UI, and sound | Claudia | T13 |
| T30 | Final build tested on both computers, and remove the Pipeline package from the project | Both | Everything above |

### Deliverable 2

| Code | Task | Owner | Depends on |
|---|---|---|---|
| T31 | Build the deliverable 2 zip: build, project without generated folders and the `Training/` folder with the `.yaml` | Both | T30 |

### If there is time, or optional

| Code | Task | Owner |
|---|---|---|
| T32 | `LevelConfig` and a level 2 with more islands | Claudia |
| T33 | Several pirates at once | Rodrigo |
| T34 | Automated tests for the pure logic (`Route`, `SpawnScheduler`, `ScoreCalculator`, `HighScoreTable`), if we decide to write them | Each of us, for our own modules |

## Schedule

| Dates | What | Tasks |
|---|---|---|
| 18–22 Nov | Phase 0: setup | T01–T05 |
| 23 Nov – 4 Dec | Version 0.1 | T06–T14 |
| 26 Nov – 2 Dec | **Deliverable 1** (uploaded on Wednesday 2nd, one day early) | T15–T17 |
| 7–18 Dec | Version 0.2 | T18–T22 |
| 14 Dec – 3 Jan | Version 0.3. During the holidays, training can run overnight. | T23–T26 |
| 28 Dec – 7 Jan | Version 1.0 | T27–T30 |
| 8–10 Jan | **Deliverable 2**: fixes only. The zip is ready on Friday 8th or Saturday 9th. | T31 |

If we have exams, we move the dates here in a PR. The deliverable dates do not move.

## Routine

- **Every task:** create its issue, work on its branch, test it by playing, read your own PR until you understand it and tell the other person so they can review it.
- **Reviews:** try to review the other person's PR within a day, so you do not block them.
- **Twice a week** (for example, Monday and Thursday): 15 minutes together to pick the next tasks and settle design questions.
- **At the end of each version:** a build, tested on the other person's computer.
- **Before each deliverable:** a review of the design, with questions like the ones a teacher would ask.
