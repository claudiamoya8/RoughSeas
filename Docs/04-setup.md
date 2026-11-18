# Setup

This guide gets the project ready on both computers. Follow it in order: each part says who does it.

| Part | Who | What | Approximate time |
|---|---|---|---|
| [A](#a-install-the-tools) | Both | Install the tools | 45 min, mostly waiting for downloads |
| [B](#b-the-repository) | Both | Configure and clone the repository, and store the course material | 10 min |
| [C](#c-create-the-unity-project) | Rodrigo | Create the Unity project (task T03) | 30 min |
| [D](#d-join-the-unity-project) | Claudia | Open the project and review its PR | 20 min |

The commands are for **PowerShell**. To open it: Start menu → type "PowerShell" → Windows PowerShell. When a step says "close and reopen PowerShell", it is because a newly installed program is not visible until you open a new window.

## A. Install the tools

Both of us, each on our own computer.

### A1. Unity Hub and Unity 6000.6.8f1

The assignment requires exactly version **6000.6.8f1**, and we both have to use the same one: if one of us opens the project with another version, Unity converts it and the other person will have problems.

1. If you do not have Unity Hub, download it from unity.com/download and install it.
2. Open Unity Hub → **Installs** (left column) → **Install Editor**.
3. Look for **6000.6.8f1** in the list of official versions. If it is not there, find it in the *Unity Download Archive* on Unity's website and click **Install**: it opens in Unity Hub.
4. No modules need to be ticked: the Windows build support is already included. Click **Install** and wait.

### A2. Git and Sourcetree

Git keeps the project's history. Check whether you have it:

```powershell
git --version
```

If it fails, install it:

```powershell
winget install --id Git.Git -e
```

To create branches, commit, push and browse the history you can use **Sourcetree**, as in the Git exercise from class, or the terminal. Who does what is in the [GitHub workflow](05-github-workflow.md).

### A3. Your name in Git

GitHub links each commit to an account through its email. Use the same email as your GitHub account:

```powershell
git config --global user.name "First Last"
git config --global user.email "your-github-account-email"
```

Check it with `git config --global user.email`. If you do not want your personal email in the commits, GitHub gives you a private one in *Settings → Emails* (like `12345+username@users.noreply.github.com`).

### A4. Unity CLI

The official Unity command-line tool, used to open, test and build the project from the terminal.

1. Install it in PowerShell:

   ```powershell
   irm https://unity.com/install.ps1 | iex
   ```

2. Close and reopen PowerShell, and check it works with `unity --version`.
3. Sign in with your Unity account (the same one as in Unity Hub) and turn off usage data collection:

   ```powershell
   unity auth login
   unity analytics opt-out
   ```

The CLI is in beta and updated often. When it announces a new version, we both update it at the same time with `unity self-update`.

### A5. A code editor

To read and debug the C#: Visual Studio or VS Code with the **C# Dev Kit** and **Unity** extensions. If you have neither, install VS Code:

```powershell
winget install --id Microsoft.VisualStudioCode -e
```

### A6. Smart merge for scenes (recommended)

Unity includes a tool, UnityYAMLMerge, that resolves many scene and prefab conflicts on its own. To make Git use it, run these three commands once Unity 6000.6.8f1 is installed:

```powershell
git config --global merge.unityyamlmerge.name "Unity SmartMerge"
git config --global merge.unityyamlmerge.driver "'C:/Program Files/Unity/Hub/Editor/6000.6.8f1/Editor/Data/Tools/UnityYAMLMerge.exe' merge -h -p --force %O %B %A %A"
git config --global merge.unityyamlmerge.recursive binary
```

If you installed Unity in another folder, change the path. The repository already says which files use this tool (`.gitattributes`).

### A7. Final check

```powershell
git --version
git config --global user.email
unity --version
unity auth status
```

All four must answer without errors, and `unity auth status` must say you are signed in. Unity Hub → **Installs** must list 6000.6.8f1.

## B. The repository

The repository already exists: [claudiamoya8/RoughSeas](https://github.com/claudiamoya8/RoughSeas). Claudia owns it and Rodrigo is a collaborator.

### B1. Settings (Claudia, once)

Only the owner can change them. In **Settings → General**, *Pull Requests* section:

- leave only **Allow squash merging** ticked and, in its drop-down, choose **Default to pull request title**;
- untick **Allow merge commits** and **Allow rebase merging**;
- tick **Automatically delete head branches**.

Optional: with the **GitHub Student Developer Pack** (education.github.com/pack), Claudia gets GitHub Pro for free and can protect `main` in **Settings → Branches**, so nothing gets in without an approved PR. Without it, we follow the rule ourselves.

### B2. Clone it (each of us, once)

In Sourcetree, **Clone** with the repository URL; or in PowerShell, from the folder where you want it:

```powershell
git clone https://github.com/claudiamoya8/RoughSeas.git
```

This creates the `RoughSeas` folder. The first time you push, a browser window opens to sign in to GitHub.

### B3. Course material

Inside the repository folder, create a `CourseMaterial` folder and keep the assignment, notes and slides there. **It is never uploaded** (it is in `.gitignore`), but it stays at hand next to the project.

## C. Create the Unity project

Rodrigo does it once the documentation PR (task T02) is merged into `main`.

1. On GitHub, create the issue "T03 · Base Unity project" with the task template and assign it to yourself.
2. Create the branch from an up-to-date `main` (N is the issue number with three digits):

   ```powershell
   git switch main
   git pull
   git switch -c feature/00N-base-unity-project
   ```

3. In Unity Hub:
   - **Projects** → **New project** → version 6000.6.8f1 → **Universal 3D** template;
   - *Project name*: `RoughSeas`;
   - *Location*: the repository folder;
   - untick "Connect to Unity Cloud" and "Use Unity Version Control";
   - **Create project**.
4. Close Unity and add the Pipeline package, which lets the Unity CLI talk to the open editor. Pin it to the agreed version:

   ```powershell
   unity pipeline install --project-path RoughSeas --package-version 1.0.0-pre.1
   ```

5. Open the project again and check in **Edit → Project Settings → Editor** that *Asset Serialization → Mode* is **Force Text** and *Version Control → Mode* is **Visible Meta Files**. These are the defaults.
6. Create the `Assets/_Project/` folders and the empty scenes from the [technical design](02-technical-design.md), and remove the template's sample content.
7. Check with `git status` that no generated folders (`Library/`, `Temp/`…) appear. Then commit, push and open the PR as described in the [GitHub workflow](05-github-workflow.md), and tell Claudia.

## D. Join the Unity project

Claudia does it. While Rodrigo creates the base project, she can work on task T04 (the ML-Agents test, in a separate project).

1. When the base project PR is open, switch to its branch (its name is shown on the PR page):

   ```powershell
   git fetch origin
   git switch BRANCH_NAME
   ```

2. In Unity Hub → **Projects** → **Add** → **Add project from disk** → choose the `RoughSeas\RoughSeas` folder → open it with 6000.6.8f1. The first time it takes several minutes: Unity generates the `Library` folder, which is not uploaded.
3. Check that the Unity CLI sees the open editor: `unity status --project-path RoughSeas` should list it as `ready`.
4. Review and approve the PR on GitHub (how, in the [GitHub workflow](05-github-workflow.md#7-review-the-other-persons-pr)).

## Troubleshooting

| Problem | Solution |
|---|---|
| `winget` is not recognised | Update "App Installer" from the Microsoft Store, or install the program from its website. |
| `unity` is not recognised right after installing it | Close and reopen PowerShell. |
| Unity Hub warns that the project uses another version | Do not open it with another version: install 6000.6.8f1. |
| `git push` asks for a username and password | Let the browser open and sign in to GitHub; Git remembers it. |
| `git push` says *rejected* | Someone pushed changes first: pull them before trying again. |
| Red errors in the console when opening the project | Wait for Unity to finish importing. If they remain, check the first error in the console. |
| `unity status` does not list the editor | Wait for Unity to finish importing and click on its window. If Unity opened in **Safe Mode**, there are compile errors: fix them and restart Unity. |
