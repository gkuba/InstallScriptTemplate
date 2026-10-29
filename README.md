# Install Script Template

A starting point for interactive Bash installer scripts in the Violet-Noir
style: a full-screen wizard that fits 80×24, runs every system change through
one dry-run-aware helper, and shows live command output in a framed install
screen.

`install-script-template.sh` works as-is. Every action in it is a harmless
placeholder (echo output and `sudo -n true`), so you can run it to see every
screen before building anything.

## The wizard

1. **Welcome:** the system, the user and the desktop session, plus what the
   installer can set up.
2. **Selection pages:** package groups on page 1, then two pages of actions.
3. **Review selections:** every item with its status (installed, install,
   apply, skip), in two columns.
4. **Review the plan:** a summary of what will run. Problems that would make
   the run fail are shown as blockers, and Enter stays disabled until they
   are fixed.
5. **Authenticate:** the sudo password goes to `sudo -S -v` through a pipe,
   never argv, the environment or the log. A keepalive refreshes it during the
   run.
6. **Install steps:** each step runs with its output in a frame. A failure
   stops for a choice: fatal steps end the run, the others offer Continue or
   Stop. Ctrl+C stops after the current step, so a package manager is never
   interrupted mid-transaction.
7. **Completion:** one line per step with its status and duration, any reboot
   or log-out notes, and Ctrl+D for the full log.

Keys: ↑↓ or j/k move, ←→ or h/l switch columns, Space toggles, G toggles a
group, A/N select all or none, Enter goes on, B or Backspace goes back, Esc
quits.

## Try it

```bash
./install-script-template.sh            # the full wizard
./install-script-template.sh -n         # dry run: commands are shown, not run
./install-script-template.sh -pt        # preselect groups, open at the review screen
DEMO_FAIL_STEP=3 ./install-script-template.sh   # see how a failed step looks
DEMO_BLOCK=1 ./install-script-template.sh       # see a plan blocker
./install-script-template.sh --help
```

## Making a new installer

Copy the template, then edit only the four sections marked **CUSTOMIZE**:

| Section | What goes there |
|---|---|
| 1. Identity and flags | `APP_NAME`, `SCRIPT_SLUG`, `usage`, one short flag per selection group, `apply_preselection` |
| 2. Items, groups and steps | the item arrays, `GROUP_*` tables, page titles, `PHASE_*` step tables and `phase_enabled` |
| 3. Checks | `prerequisite_problem`, `package_installed`, `collect_plan_blockers`, `plan_group_notes`, `after_step_hook` |
| 4. Step functions | one function per install step |

Everything else (layout, keys, the output frame, dry run, the sudo keepalive)
is engine code that should not need changes. Remove the demo helpers
(`DEMO_INSTALLED`, `demo_output`, `demo_fail_check`) from real scripts.

Rules for step functions:

- Send every command that changes the system through `run` or `as_root`
  (or a wrapper built on them, such as `pacman_install`). In a dry run they
  print `→ would run: …` instead. Read-only checks always run for real.
- Print progress with `info`, `success`, `warn` and `error`; the frame colors
  lines by their prefix.
- Steps run with no keyboard input (stdin is `/dev/null`), so anything that
  would prompt must fail instead. Check every exit status and return nonzero
  on failure.
- Back up a config file before replacing it.

## Checks before release

```bash
bash -n script.sh
shellcheck -S warning -e SC2034,SC2178 script.sh   # those two are nameref false positives
script.sh --dry-run
```

Then look at every screen at 80×24 and 100×24, and with `LC_ALL=C` for the
ASCII fallback, and do a real run in a VM snapshot.

## Requirements

Bash 4.4 or newer, a terminal of at least 80×24, and sudo. Run it as your
normal user; it refuses to run as root.

## Scripts built on it

- [Arch-GKuba](https://github.com/gkuba/Arch-GKuba) `post-install.sh`: the
  reference implementation of the engine.
