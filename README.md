# silverblue-recipe

Capture the reproducible OS layer of a Fedora Atomic (Silverblue) machine, then
replay it onto a fresh install. It records what Fedora and Flathub can hand
back: layered `rpm-ostree` packages, flatpaks, and your `/etc` changes. It does
not back up your data. Pair it with a `home` backup for that.  All data is
stored in `~/.local/share/silverblue-recipe/`.

Required dependencies:

- [Fedora Silverblue 44](https://fedoraproject.org/atomic-desktops/silverblue/)

Optional dependencies:

- [btrbk](https://digint.ch/btrbk/index.html)


## Install

Start by cloning this repository:

```shell
git clone https://github.com/van-riper/silverblue-recipe.git
cd silverblue-recipe
```

Install the executable to a `bin_t` path so a systemd service can exec it under
enforcing SELinux, even if you also run it by hand:

```shell
sudo install -m0755 bin/silverblue-recipe /usr/local/bin/silverblue-recipe
sudo restorecon /usr/local/bin/silverblue-recipe
```

To regenerate the recipe automatically, install the templated unit and enable it
for your user:

```shell
sudo cp systemd/silverblue-recipe@.service.example /etc/systemd/system/silverblue-recipe@.service
sudo systemctl enable "silverblue-recipe@$USER.service"
```

Once enabled, the service captures at boot.

If you use btrbk and want the recipe to refresh before each backup, uncomment
the unit's `Before=btrbk.service` line before applying the template. Make sure
you have a `btrbk.service` drop-in that `Wants=` it.

## Capture

```shell
sudo silverblue-recipe capture
```

Run capture as root. Under `sudo` the recipe goes to `$SUDO_USER`'s home. As
bare root or from a service, name the target user:

```shell
silverblue-recipe capture --user <username>
```

## Commands

- `capture`: write the recipe (run as root)
- `restore-packages`: re-layer your rpm-ostree packages
- `restore-flatpaks`: reinstall your flatpaks
- `restore-config`: restore captured `/etc` and `/var` bits

Add `--dry-run` to any `restore` command to see what it would run.

## Restore onto a fresh install

1. Re-layer packages, then reboot:

   ```shell
   silverblue-recipe restore-packages
   ```

2. Reinstall flatpaks (`--user` for per-user installs):

   ```shell
   silverblue-recipe restore-flatpaks
   ```

3. Restore system config. This will extract the captured `/etc` and `/var` into
   a review directory and stops; nothing touches the live system:

   ```shell
   sudo silverblue-recipe restore-config
   ```

   From here, it's recommended to just manually copy what you need into `/etc`
   and `/var`. This is much safer than applying all the extracted files.

   If you still wish to apply all the extracted files, make sure to review them
   _carefully_, then overlay them onto `/` with `--apply`:

   ```shell
   sudo silverblue-recipe restore-config --apply
   ```

   **WARNING:** `--apply` overwrites `/etc` and parts of `/var`, so only run
   it on a fresh install. It skips machine-identity files (machine-id, fstab,
   crypttab, ssh host keys, passwd/shadow/group, nvme ids) and relabels
   SELinux afterward. Check hardware-specific config (graphics drivers,
   firmware) yourself before applying.

The restore steps stay separate on purpose. Real recovery also involves reboots
and a `home` restore that this tool leaves to you.

If you want the program to point to a different folder for storing data, use the
`--output-dir <path>` flag. This is useful when you want the data stored on an
external drive for keeping backups.

## How origins are kept

Each capture copies the current deployment's `.origin` files into `origins/`.
When a configuration rolls off the live set, it moves to `origins/history/`
rather than getting deleted. ostree names these files by deployment checksum,
which changes on every base update even when your package set does not, so
identical configs are deduped by content.
