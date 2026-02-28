# Torch Development Workflow

## Branch Strategy (cornball-ai/torch fork)

All development work happens on the `fork-ci` branch first. This branch has
the CI workflow (`.github/workflows/fork-ci.yaml`) that tests lantern builds
and R CMD check on Ubuntu, Windows, and (when available) macOS.

**Workflow:**
1. Do all work on `fork-ci` (or a branch based on it)
2. Push to `origin/fork-ci` and wait for CI to pass
3. Once green, cherry-pick or merge the changes into the appropriate
   upstream-facing branch (e.g. `libtorch/v2.8.0`) that is shared with
   mlverse/torch via PR

Never push untested work directly to a PR branch.
