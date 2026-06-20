# CONTRIBUTING.md

This document covers how to branch, commit, and submit work in this repo — for both labs and the capstone.

---

## Branch Naming

**Labs:**
```
lab<number>-<teamname>
```
Example: `lab3-teamname`

**Capstone:**
```
capstone/<track-name>
```
One of: `capstone/perception`, `capstone/rl-agent`, `capstone/ros-integration`, `capstone/fleet-networking`

---

## Commit Messages

Use a semi-formal style — a short type prefix, then a clear, specific description. Doesn't need to be strict Conventional Commits, but should follow this shape:

```
<type>: <what changed>
```

Common types: `feat`, `fix`, `docs`, `refactor`, `test`

**Good:**
```
feat: add slot alignment detection to cv_state_node
fix: correct PWM duty cycle range in motor_node
docs: update lab3 README with prelab instructions
```

**Avoid:**
```
update stuff
fixed it
asdf
```

If a commit only makes sense alongside others, that's fine — just keep each message specific to what that commit actually does.

---

## Pull Requests

### Labs

- Open a PR from your branch into your team's fork `main`
- **Reviewed and approved by Zach or Parik** before merging
- PR description should briefly state what was completed and link the lab spec section it addresses

### Capstone

- Open a PR from your track branch into `capstone/main`
- **Must include proof of success** — not a teammate sign-off, but evidence the node actually works. Acceptable forms:
  - Terminal output of `ros2 topic echo <topic>` showing correctly formatted messages
  - A short video/gif of the car responding to the node's output
  - A screenshot of training curves or eval metrics (RL Agent track)
  - Logs from the coordinator server showing a successful claim/release cycle (Fleet Networking track)
- Reviewed and merged by Zach or Parik
- PR description should reference the relevant section of `INTERFACES.md` your node implements

---

## AI Usage Policy

AI tools are allowed for debugging, explaining errors, and exploring approaches.

**You must be able to explain every line of code you submit.** If you can't walk through what a piece of code does and why it's there during a check-in, it doesn't count — regardless of whether AI helped write it.

This applies to both labs and capstone work.

---

## Questions

Stuck on workflow, not code? Ask in Discord. Stuck on a lab or capstone concept? Same — we're always available.
