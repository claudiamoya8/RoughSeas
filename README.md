# Rough Seas

Final project for **Programming Paradigms** (IMAT, Universidad Pontificia Comillas ICAI, 2026/27).

Guide merchant ships between three islands by drawing their routes with the mouse. Each ship must reach the port of its colour without crashing into other ships or rocks, while a pirate ship, controlled by an AI trained with Unity ML-Agents, tries to sink them with its cannon.

| | |
|---|---|
| Team | Rodrigo Sicilia and Claudia Moya |
| Engine | Unity 6000.6.8f1 (Universal Render Pipeline) |
| Deliverable 1 | 3 December 2026: project description and UML design |
| Deliverable 2 | 10 January 2027: playable build, Unity project and ML-Agents training setup |

## How to play

- Click a ship and drag to draw its route on the water. The ship follows it.
- Deliver 20 ships to the port that matches their flag colour to win.
- If 3 ships sink, you lose.
- Ships sink when they hit each other, a rock, a coast, the reef around the map, the pirate or a cannonball.
- Press Esc to pause.

## Documentation

1. [The game](Docs/01-game.md): game rules and what the assignment asks for.
2. [Technical design](Docs/02-technical-design.md): modules, classes, events and design patterns.
3. [Plan](Docs/03-plan.md): who does what, task list and schedule.
4. [Setup](Docs/04-setup.md): getting each computer ready, step by step.
5. [GitHub workflow](Docs/05-github-workflow.md): issues, branches, commits, pull requests and reviews.
6. [Decisions](Docs/decisions.md): log of the decisions we take.

## Design at a glance

- Modular code split into layers (`Common`, `World`, `Routes`, `Ships`, `Pirates`, `AI`, `Spawning`, `Core`, `UI`, `Training`). Dependencies only go downwards; upward communication uses C# events.
- Main design patterns:
  - **State:** match flow and the pirate's finite-state machine;
  - **Observer:** C# events.
- Other patterns: Strategy (interchangeable pirate brains), Factory, Flyweight (ScriptableObjects), Command (Input Actions), Adapter (ML-Agents) and Object Pool.

## Repository layout

```
.
├── RoughSeas/    Unity project (created in task T03)
├── Training/     ML-Agents configuration (.yaml) and results
├── Docs/         project documentation
└── .github/      issue and pull request templates
```

## Workflow

Every change goes through an issue, a branch and a pull request reviewed by the other team member, then is merged with *Squash and merge*. Everything in the repository (code, documentation, commits, branches and pull requests) is in English.
