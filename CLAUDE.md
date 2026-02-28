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

When cherry-picking to the PR branch, comment on the upstream PR with a
link to the passing fork CI run.

## Remotes

- `origin` → `cornball-ai/torch` (our fork)
- `upstream` → `mlverse/torch`

## Files that live only on fork-ci

- `CLAUDE.md` — don't put on PR branches to avoid cluttering upstream
- `.github/workflows/fork-ci.yaml` — CI only triggers for branches that contain it
