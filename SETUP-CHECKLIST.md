# Repo Setup Checklist
**Run this for every new project. Takes ~10 minutes.**

---

## Step 1 — Create the Repo

On GitHub, create new repo:
- Name: `project-name` (lowercase, hyphens, descriptive)
- Visibility: Public or Private
- ✅ Add README file
- ✅ Add .gitignore → choose **none** (we use our own)
- License: MIT (for personal/learning repos)

---

## Step 2 — Clone and Initial Setup

```bash
git clone https://github.com/foxxy05/project-name
cd project-name

# Copy in your standard .gitignore
cp /path/to/.gitignore .

# Create the essential files
touch CHANGELOG.md

# Initial commit
git add .
git commit -m "chore: initial project structure with gitignore and changelog"
git push
```

---

## Step 3 — Create the `dev` Branch

```bash
git checkout -b dev
git push -u origin dev
```

Then on GitHub: **Settings → General → Default branch → change to `dev`**

From this point, all day-to-day work happens on `dev` or feature branches off `dev`.

---

## Step 4 — Protect `main`

**GitHub → Repo → Settings → Branches → Add branch ruleset**

Branch name pattern: `main`

Enable:
- ✅ Restrict deletions
- ✅ Require a pull request before merging
  - Required approvals: 1 (you can approve your own PRs solo)
- ✅ Block force pushes

This prevents anyone (including you at 2am) from pushing directly to `main`.

---

## Step 5 — Create Issue Labels

**GitHub → Issues → Labels → Edit labels**

Delete the default labels. Create these:

| Label | Color | Use |
|-------|-------|-----|
| `firmware` | `#0075ca` | Code changes |
| `hardware` | `#e4e669` | Wiring, PCB, mechanical interface |
| `driver` | `#7057ff` | Sensor/actuator driver work |
| `bug` | `#d73a4a` | Something broken |
| `test` | `#0e8a16` | Hardware validation needed |
| `docs` | `#cfd3d7` | Documentation only |
| `blocked` | `#b60205` | Waiting on external factor |
| `competition` | `#e99695` | Must complete before competition |

---

## Step 6 — Set Up GitHub Project (Optional but Recommended)

**GitHub → Projects → New Project → Board**

Columns:
- **Backlog** — Ideas and future tasks
- **This Week** — Currently planned
- **In Progress** — Actively being worked on
- **Testing on Hardware** — Code done, needs hardware validation
- **Done** — Merged and verified

Link the project to your repo: Project Settings → Link Repository

---

## Step 7 — Create Issue Templates

Create this folder structure in your repo:

```
.github/
└── ISSUE_TEMPLATE/
    ├── bug_report.md
    └── feature_request.md
```

Copy the `bug_report.md` and `feature_request.md` from this system into that folder. Commit to `dev`.

---

## Step 8 — First Real Issue

Create an Issue for your first actual task. This creates the habit. Title it:  
`[FEAT] Initial firmware: GPIO and clock config`

Assign it to yourself. Add label `firmware`. Add to your Project board under "In Progress."

---

## RYUK-Specific: Applying to an Existing Repo

Since RYUK already has 42 commits, don't restructure — just layer these practices on:

```bash
# 1. Create dev branch from current main state
git checkout main
git checkout -b dev
git push -u origin dev

# 2. Apply the .gitignore and clean up tracked junk
cp .gitignore-STM32CubeIDE .gitignore
git rm -r --cached .metadata/
git rm -r --cached **/*.o **/*.d **/*.su 2>/dev/null || true
git add .
git commit -m "chore: apply .gitignore, remove .metadata and build artifacts from tracking"
git push origin dev

# 3. Delete the duplicate folder (4002 case collision)
# Manually delete either 4002_ODRIVE_INTEGRATE or 4002_ODrive_Integrate
# Keep whichever has the newer/cleaner code
git rm -r 4002_ODrive_Integrate/   # adjust as needed
git commit -m "chore: remove duplicate 4002 folder (case collision on Windows)"
git push origin dev

# 4. From now on: all new work goes on feature branches off dev
git checkout -b feature/next-thing
```

**Do NOT apply `git rm -r --cached .` to RYUK.** With 30 CubeIDE project folders, the re-add will be messy. Instead, surgically remove only `.metadata/` and build artifacts as shown above.

---

## Gitignore Quick Fix Command Reference

```bash
# Check what's currently tracked that shouldn't be
git ls-files --cached | grep -E "\.metadata|Debug/|\.o$|\.elf$"

# Remove specific tracked directories
git rm -r --cached .metadata/
git rm -r --cached Debug/

# Verify gitignore is working (should show no output for ignored files)
git check-ignore -v Debug/
```
