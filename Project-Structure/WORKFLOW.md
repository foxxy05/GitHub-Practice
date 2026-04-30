# Git & GitHub Workflow — Embedded Robotics Projects
**foxxy05 | Team-Automatons-2026**
*Last updated: 2026*

---

## Philosophy

Version control for embedded robotics isn't just about code backup. It's about being able to answer three questions at any moment:

1. **What is the last known-working state of this robot?**
2. **What is everyone currently working on, and does it conflict?**
3. **Why did we make this design decision?**

Every practice in this doc exists to answer one of those questions.

---

## Repository Structure

Every new project repo follows this layout:

```
project-name/
├── Core/
│   ├── Src/          ← Your application code (.c files)
│   └── Inc/          ← Your headers (.h files)
├── Drivers/          ← HAL/CMSIS (keep tracked if modified)
├── Middlewares/      ← FreeRTOS, USB, etc.
├── Documents/        ← Datasheets, wiring diagrams, PDFs
├── scripts/          ← Python tuning/calibration tools
├── .gitignore        ← Always present from day 1
├── README.md         ← Always present from day 1
└── CHANGELOG.md      ← Updated on every milestone
```

**Rule:** Create `.gitignore`, `README.md`, and `CHANGELOG.md` before writing a single line of firmware. Always.

---

## Branch Strategy

```
main         ← Competition-ready / milestone builds ONLY
 └── dev     ← Daily integration branch (your working branch)
      ├── feature/xyz    ← New subsystem or capability
      ├── fix/xyz        ← Bug fixes
      └── experiment/xyz ← Risky or exploratory work
```

### The Branches

**`main`**
- Only receives merges from `dev` via Pull Request
- Every commit on `main` should be tagged (see Tagging below)
- Think of it as: *"I would flash this on competition day"*
- Protected — no direct pushes, ever

**`dev`**
- Your everyday working branch
- Should compile and run at all times
- Teammates' feature branches merge back here
- Create this immediately after creating the repo

**`feature/name`**
- Branch from `dev`, merge back to `dev`
- Name describes what it does: `feature/bno055-kalman`, `feature/odrive-velocity-mode`
- Delete after merging

**`fix/name`**
- Branch from `dev` (or `main` for critical competition fixes)
- Name describes what broke: `fix/uart-overflow`, `fix/encoder-tick-overflow`
- Delete after merging

**`experiment/name`**
- Branch from `dev`
- May NEVER merge — that's fine
- For risky tries: `experiment/foc-control`, `experiment/rtos-migration`
- If abandoned, leave a final commit: `"experiment abandoned: [reason]"`

### When to Create a Branch

**Do branch when:**
- Adding a new sensor driver or communication protocol
- Attempting a control algorithm change (PID tuning strategy, new filter)
- Doing anything that could break currently working robot behavior
- A teammate is working on something simultaneously
- Trying an approach you're not sure will work

**Don't branch for:**
- Fixing a README typo → commit directly to `dev`
- Adding a debug `printf` → commit directly to `dev`
- Changing a single constant → commit directly to `dev`

### When to Merge

| From → To | When |
|-----------|------|
| `feature` → `dev` | Feature compiles, tested on hardware, doesn't break existing behavior |
| `fix` → `dev` | Bug confirmed fixed on hardware |
| `dev` → `main` | Milestone reached: working demo, competition-ready build, release |

**Never merge:**
- Code that doesn't compile
- Untested experiment branches (unless promoted to feature)
- Directly to `main` without a PR

---

## Commit Practice

### Commit Often, Commit Small

The goal is: if something breaks, you can pinpoint *exactly* which change caused it.

**Good rhythm:**
```
Implement encoder ISR             ← commit
Test encoder, fix tick overflow   ← commit  
Add encoder to odometry calc      ← commit
Test odometry on flat surface     ← commit
```

**Bad rhythm:**
```
Finished odometry module          ← one giant commit (you lose all history)
```

### Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short description>
```

| Type | Use for |
|------|---------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | README, comments, documentation only |
| `test` | Test sketches, hardware validation |
| `refactor` | Code restructure, no behavior change |
| `chore` | Gitignore, build config, dependencies |
| `wip` | Work in progress (use sparingly) |

**Examples:**
```
feat(odometry): implement 3-wheel dead reckoning with encoder feedback
fix(uart): handle DMA buffer overflow at 1MHz polling rate
feat(imu): add BNO055 I2C driver with interrupt-based data ready
docs(readme): add wiring diagram for SICK sensor connection
chore: apply .gitignore, remove .metadata from tracking
refactor(pid): extract gain calculation into separate function
```

**Link to Issues (when applicable):**
```
feat(mechanisms): add claw servo control — closes #14
```

---

## Pull Requests

Even working solo, open a PR when merging `feature` → `dev` or `dev` → `main`. The PR diff view catches last-minute issues. The PR description is future documentation.

### PR Description Template

```markdown
## What changed
Brief description of what this PR adds/fixes.

## Hardware tested on
- [ ] STM32F446ZE (RYUK)
- [ ] STM32F407 Discovery
- [ ] Nucleo board

## How to test
Step-by-step of what to do to verify this works.

## Known issues / limitations
Any caveats or things to watch out for.

## Closes
Closes #[issue number]
```

### Merge Strategy

- **feature → dev**: Use "Squash and merge" — keeps dev history clean
- **dev → main**: Use "Create a merge commit" — preserves milestone history

---

## Tagging Working Builds

**This is the most important habit for competition robotics.**

Before you start any new risky experiment, tag the current working state:

```bash
# Tag the current state of main
git tag -a v1.0 -m "Working 3-wheel omni with dead reckoning, tested on field"
git push origin --tags

# Later, if everything breaks, return instantly:
git checkout v1.0
```

### Tagging Convention

```
v1.0              ← First field-tested working build
v1.1              ← Added SICK sensor integration
v1.2-comp         ← Competition day build
v2.0              ← Major architecture change (e.g. added RTOS)
```

Always include a meaningful message. "What did this build do?" should be answerable from the tag alone.

---

## Issues & Task Tracking

Use GitHub Issues for every task. This creates a searchable log of decisions.

**Create an issue for:**
- Every new feature to implement
- Every bug found during testing
- Every hardware integration (new sensor, actuator, board)
- Design decisions worth documenting ("why did we choose CAN over UART?")

**Issue Labels to create:**
```
firmware     → Code changes
hardware     → Wiring, PCB, mechanical interface
driver       → Sensor/actuator driver work  
bug          → Something broken
test         → Hardware validation needed
docs         → Documentation needed
blocked      → Waiting on something external
competition  → Must be done before competition
```

---

## Protecting `main`

In GitHub repo Settings → Branches → Add rule for `main`:

- ✅ Require a pull request before merging
- ✅ Require at least 1 approval (even if it's yourself reviewing)
- ✅ Do not allow bypassing the above settings

This physically prevents anyone from pushing directly to `main` — including yourself at 2am before competition.

---

## Daily Workflow (Solo)

```bash
# Start of session
git checkout dev
git pull origin dev

# Start a new feature
git checkout -b feature/bno055-driver

# ... write code, test on hardware ...

# Commit incrementally
git add Core/Src/bno055.c Core/Inc/bno055.h
git commit -m "feat(imu): implement BNO055 I2C initialization and calibration read"

# ... more work ...
git add Core/Src/bno055.c
git commit -m "feat(imu): add interrupt-based data-ready handling"

# Done with feature, push and open PR
git push origin feature/bno055-driver
# → Open PR on GitHub: feature/bno055-driver → dev
# → Review diff yourself, merge
# → Delete branch

# When a milestone is reached
git checkout main
# → Open PR on GitHub: dev → main
# → Add tag after merge
git tag -a v1.2 -m "BNO055 integrated, odometry with heading correction working"
git push origin --tags
```

---

## Stashing (when interrupted mid-work)

```bash
# You're in the middle of something and need to switch branches urgently
git stash push -m "WIP: half-done CAN receive handler"

# Switch branches, fix the urgent thing, come back
git stash pop    # restores your WIP exactly
```

---

## Rollback (when hardware breaks)

```bash
# See recent commits
git log --oneline -10

# Undo the last commit safely (keeps your files, un-commits)
git reset --soft HEAD~1

# Revert a specific commit (safe — creates a new "undo" commit)
git revert abc1234

# Nuclear option: go back to a tag (ONLY if nothing else works)
git checkout v1.1   # detached HEAD, use carefully
```

---

## What NOT to Commit

```
.metadata/          ← CubeIDE workspace state (personal, local)
Debug/ Release/     ← Build outputs
*.elf *.hex *.bin   ← Compiled firmware
Makefile            ← Auto-generated by CubeIDE
secrets.h           ← Passwords, WiFi credentials, API keys
```

See `.gitignore` for the full list.

---

## CHANGELOG Format

Update `CHANGELOG.md` every time you merge to `main`:

```markdown
# Changelog

## [v1.2] - 2026-04-30
### Added
- BNO055 IMU driver with interrupt-based reading
- Heading correction integrated into odometry

### Fixed  
- UART DMA buffer overflow at high polling rates

### Known Issues
- Slight drift in Y-axis odometry over >5m travel

---

## [v1.0] - 2026-03-15
### Added
- Initial 3-wheel omni drive with dead reckoning
- ODrive velocity mode integration
- SICK sensor distance reading over UART
```

---

## Quick Reference Card

| Action | Command |
|--------|---------|
| Start feature | `git checkout -b feature/name dev` |
| Save WIP | `git stash push -m "description"` |
| Restore WIP | `git stash pop` |
| Stage selectively | `git add -p` |
| See what changed | `git diff HEAD` |
| Tag a build | `git tag -a v1.x -m "description"` |
| Push tags | `git push origin --tags` |
| Undo last commit | `git reset --soft HEAD~1` |
| Return to safe state | `git checkout v1.x` |
