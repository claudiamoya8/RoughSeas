# GitHub workflow

We work as in the pair Git exercise from class: every change goes on its own branch, is proposed with a pull request (PR) and is approved by the other person before it gets into `main`.

## The idea

```
issue #12 → branch feature/012-route-drawing → commits → push → PR → review by the other person → Squash and merge → main
```

- **Issue:** the assignment of one of the tasks in the [Plan](03-plan.md). It uses the task card from class:
  - the user story;
  - the acceptance criteria (given, when, then);
  - the *Definition of Done*;
  - who does it, which modules and scenes it touches and what it depends on.
- **Branch:** a working copy for that task. `main` always works; branches can be half-done.
- **Commit:** a snapshot of the changes, with a message saying what was done.
- **Push:** upload the branch to GitHub.
- **Pull request:** the proposal to merge the branch into `main`. This is where the other person reviews.
- **Squash and merge:** when merging, all the branch's commits are combined into a single commit on `main`, titled with the PR title. That keeps `main` at one commit per feature, easy to read.

## A task from start to finish

Example: Rodrigo works on task T08, "Route drawing", which is issue #12 on GitHub.

### 1. Create the issue

On GitHub → **Issues** → **New issue** → **Task** template. The title starts with the task code ("T08 · Route drawing"). Fill in the card and assign it to whoever will do it (*Assignees*).

### 2. Create the branch

From an up-to-date `main`:

```powershell
git switch main
git pull
git switch -c feature/012-route-drawing
```

In Sourcetree: **Pull** on `main`, then **Branch** → new branch name.

### 3. Work

Write the code, set up the prefabs and test it in your `Sandbox_` scene. When it is done, **play it** for five minutes. Anything you do not like gets fixed now, before the PR.

### 4. Commits and push

Make one to four commits, each with a group of changes that compiles on its own:

```powershell
git add RoughSeas/Assets/_Project/Scripts/Routes
git commit -m "Add route drawing with LineRenderer"
git push -u origin feature/012-route-drawing
```

In Sourcetree it is the same with buttons: tick the files, write the message, **Commit**, then **Push**.

### 5. Open the PR

On the repository page a **Compare & pull request** button appears; if not, go to **Pull requests** → **New pull request** and choose your branch. Check that it goes from your branch to `main`. Fill in the template:

- title: same as the main commit;
- what changes, how to test it and which scenes and prefabs it touches;
- `Closes #12`, so the issue closes when it is merged.

Add the other person under **Reviewers** and click **Create pull request**.

### 6. Read your own PR

Before telling the other person, read the PR on GitHub (**Files changed** tab) until you understand every file. **We can be asked about any part of the code.**

### 7. Review the other person's PR

Claudia gets a notification from GitHub. On her computer:

1. She switches to the PR branch (its name is on the PR page) and plays it:

   ```powershell
   git fetch origin
   git switch feature/012-route-drawing
   ```

2. She reads the changes in the **Files changed** tab. She can comment on a specific line: hovering over the line number shows a **+**.
3. When she is done, she clicks the green review button (**Review changes** or **Submit review**, depending on the GitHub version) and chooses:
   - **Comment**, to just comment;
   - **Approve**, if it is fine;
   - **Request changes**, if something must change before merging.

What to check in a review:

- it only touches its owner's modules and scenes;
- dependencies only go downwards, and upward communication uses events;
- the patterns (especially State and Observer) are applied correctly;
- event subscribers unsubscribe;
- all code is in English;
- no `GameObject.Find` during the match;
- numbers live in ScriptableObjects;
- the `.meta` files are there, and there is no `Library/` or build output.

### 8. Address the review

Rodrigo makes the requested changes in a new commit whose message says what changes (for example, `Shorten collision warning distance`), never "fix review". He pushes the branch again with `git push` and Claudia takes another look. With "Squash and merge", `main` still ends up with a single commit.

### 9. Merge

Once approved, whoever opened the PR clicks **Squash and merge** at the bottom of the PR page, checks that the commit title is the PR title and clicks **Confirm squash and merge**. GitHub deletes the branch.

### 10. Stay up to date

Both of us, when starting to work each day:

```powershell
git switch main
git pull
```

If `main` brings scene or package changes, wait for Unity to finish importing.

## Commits

- In English, in the imperative and under 60 characters: "Add…", "Fix…", "Remove…".
- No emojis, no prefixes like `feat:`, no signatures.
- One to four per PR, and each one compiles.
- No "wip", "changes" or "fix review" commits.

| Good | Bad |
|---|---|
| `Add route drawing with LineRenderer` | `feat: ✨ Implement comprehensive route system` |
| `Smooth route curves` | `wip` |
| `Recycle sunk ships with an object pool` | `changes` |
| `Fix double sinking when two ships collide` | `fix review` |

## Branches

| Type | Name | Example |
|---|---|---|
| Feature | `feature/NNN-short-name` | `feature/012-route-drawing` |
| Fix | `fix/NNN-short-name` | `fix/027-double-sinking` |
| Documentation | `docs/NNN-short-name` | `docs/015-uml-deliverable` |

NNN is the GitHub issue number with three digits. Branches always start from an up-to-date `main`, never from another branch.

## Unity and Git

- **Never uploaded:** `Library`, `Temp`, `Logs`, `UserSettings`, `obj`, builds and `CourseMaterial`. The `.gitignore` takes care of it. If Sourcetree or `git status` shows hundreds of strange files, stop and check before committing.
- **Every file in `Assets` has its `.meta`**, and they are uploaded together. Move or rename assets only from Unity, never from Windows Explorer.
- **Do not open the project with any Unity version** other than 6000.6.8f1.

### Scenes and prefabs

Unity scenes merge badly when two people change them at the same time. So:

1. Each scene has an owner ([Plan](03-plan.md)), and each issue says which scenes it touches.
2. **Only one open PR at a time per scene.** If you need to change a scene that the other person has in an open PR, wait for it to be merged or talk about it.
3. As much as possible goes into **prefabs**: each of us works on our own, and the scene only places them.
4. For testing, each of us has a scene in `Scenes/Sandbox/` (`Sandbox_Rodrigo` and `Sandbox_Claudia`).

## If there is a conflict

A conflict happens when we both changed the same thing. It is not serious:

1. Bring `main` into your branch:

   ```powershell
   git fetch origin
   git merge origin/main
   ```

2. In C# code, look at both versions, decide together how to combine them, then commit the result.
3. In scenes and prefabs, if UnityYAMLMerge cannot solve it, the safest option is usually to keep one of the two versions and redo the other change in Unity.

## If something goes wrong

| Situation | What to do |
|---|---|
| You made changes while on `main` | Do not commit. Create the branch now with `git switch -c BRANCH_NAME`: the changes come with you. |
| `Library` or similar folders were uploaded | Remove them from the repository in a PR and check the `.gitignore`. |
| Unity shows red errors after updating | Wait for it to finish importing. If they remain, read the first error in the console. |
| The push says *rejected* | Someone pushed first: run `git pull` on your branch and try again. |
| The other person is slow to review | Remind them. Meanwhile you can start another issue from `main` (not from the pending branch). |

## Cheat sheet

| Command | What it does |
|---|---|
| `git status` | What changed and which branch you are on |
| `git switch main`, then `git pull` | Go to `main` and get the latest changes |
| `git switch -c feature/012-route-drawing` | Create a task branch and switch to it |
| `git switch BRANCH_NAME` | Switch to an existing branch |
| `git add FILES`, then `git commit -m "Message"` | Commit those files |
| `git push -u origin BRANCH_NAME` | Upload a branch to GitHub for the first time (afterwards, `git push` is enough) |
