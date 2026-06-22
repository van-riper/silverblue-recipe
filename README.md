# silverblue-recipe

Capture the reproducible OS layer of a Fedora Atomic (Silverblue) machine, then
replay it onto a fresh install. It records what Fedora and Flathub can hand back:
layered `rpm-ostree` packages, flatpaks, and your `/etc` changes. It does not
back up your data. Pair it with a `home` backup for that.  All data is stored in
`~/.local/share/silverblue-recipe/`.

## Commands

- `capture`: write the recipe (run as root)
- `restore-packages`: re-layer your rpm-ostree packages
- `restore-flatpaks`: reinstall your flatpaks
- `restore-config`: restore captured `/etc` and `/var` bits

Add `--dry-run` to any restore command to see what it would run.
