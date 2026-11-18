# Decisions

Log of what we decide, with dates. Whoever implements a decision logs it here in their PR. Newest first; if a decision changes an earlier one, the newest wins.

## 2026-11-18

- **Repository:** [claudiamoya8/RoughSeas](https://github.com/claudiamoya8/RoughSeas), owned by Claudia, with Rodrigo as a collaborator. The documentation goes in through the first PR (task T02) and replaces the initial brief.
- **Name:** the game is called **Rough Seas** (*Mar Revuelto*). `RoughSeas` is used in code and folders:
  - Unity project in `RoughSeas/`;
  - namespace `RoughSeas.<Module>`;
  - deliverable zips `…_RoughSeas_26_27`.
- **Language:** everything in the repository is in English: code, comments, assets, scenes, documentation, commit messages, branch names, issues and pull requests.
- **Folders:**
  - documentation in `Docs/`;
  - course material in `CourseMaterial/`, which is never uploaded;
  - ML-Agents configuration in `Training/`, at the repository root.
- **No UML for now:** deliverable 1 requires it, so it stays in the plan as task T16.
- **Modular design in layers:**
  - dependencies only go downwards; upward communication uses events;
  - modules and classes are described in `Docs/02-technical-design.md`;
  - our own assets go in `Assets/_Project/`.
- **Design patterns:**
  - main ones: **State** (match flow and the pirate's brain) and **Observer** with C# events, as stressed in class;
  - also: Strategy (pirate brain), Factory (ships), Flyweight (type ScriptableObjects), Command (Input Actions), a single Singleton (`GameSession`), Adapter (ML-Agents agent) and Object Pool;
  - we do not use Bridge, Decorator or Facade.
- **Art:** we do not make our own art. First basic shapes; then existing free assets (Kenney, CC0; or free Unity Asset Store assets, keeping the repository private).
- **From simple to complex:** versions 0.1, 0.2, 0.3 and 1.0, split into playable milestones (A to H). Tasks have codes (T01–T34) in `Docs/03-plan.md`.
- **Ownership by module:**
  - Rodrigo: `Routes`, `Ships`, `Pirates` and `AI`;
  - Claudia: `World`, `Spawning`, `Core`, `UI` and `Training`;
  - each of us only changes our own modules and scenes.
- **Restart:** playing again reloads the `Game` scene.
- **Issues:** use the task card from class (user story, acceptance criteria and *Definition of Done*).
- **Open** (the "For now" column in `Docs/01-game.md` applies):
  - whether a crash between merchant ships counts as 2 sunk or 1;
  - whether we write tests;
  - hours per week and exam weeks.

## 2026-11-11

- **Unity:** 6000.6.8f1 with the Universal 3D template (URP).
- **Editor automation:** the official Unity CLI (`unity`) with the Pipeline package (`com.unity.pipeline`, pinned to 1.0.0-pre.1). Usage data collection turned off.
- **Workflow:** issue → branch → PR → approval by the other person → "Squash and merge". No signatures in commits. Changes requested in a review go in a new commit whose message says what changes.
- **Development in versions:** first a simple version with basic shapes, then it gets improved, especially the visuals.
- **Pirate cannon:** it only fires when a merchant ship is very close. It aims on its own at the nearest one within range, predicting where it will be if it keeps going straight. Starting range: 3 units (6 in the initial brief).
- **Levels:** later, a few of them (not endless), for example with more pirates or more islands. We start with one, but the match data lives in ScriptableObjects so adding levels does not require rewriting code.
- **Team:** we both work on Windows.
