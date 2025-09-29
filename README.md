# ROLLSSH — MASSIVE WARNING / README

**RollSSH — admin-only SSH control helper (dangerous if misused).**

> [!WARNING]
> I want to apologize in advance for my inappropriate language, but<br>
> DUMBASS SCRIPT KIDDIES ARE $\color{#FF0000}{NOT}$ WELCOME HERE! <br>
> THIS TOOL WAS $\color{#FF0000}{NOT}$ MADE FOR SCAMMERS WITH NO BRAIN
> *if you need a .exe or .cmd version for windows, i do have a fully open-source project. <br>
> BUT! i will only give r/o access to people who are willing to sign an NDA and a consent form to $\color{#FF0000}{NEVER}$ use it for malicious purposes

> [!CAUTION]
> **DANGER: READ THIS BEFORE YOU DO ANYTHING**
>
> RollSSH contains powerful, potentially destructive features. Only install on machines you own or fully administrate. Misuse may result in permanent data loss, locked devices, or other catastrophic outcomes. If you don’t know what you’re doing — stop now.

> [!IMPORTANT]
> Backend server code is open, and located [HERE](https://www.github.com/FixCraft-Inc/rollssh-backend).

---

## TL;DR (aka “don’t be stupid”)

- RollSSH gives persistent, admin-grade SSH control and management features intended for experienced sysadmins.
- **Do not** install on devices you do not own or administer.
- Installer requires multiple explicit confirmations. Missing acknowledgement or final consent stops the install.
- You pick a mode during install:

  - `SAFE` maps to owner/local-only operation (`--own`), trimming the most aggressive actions.
  - `RECKLESS` keeps the upstream defaults. This is the dangerous one.

---

## Quick summary / package Description (Debian-friendly)

```
RollSSH — admin-only SSH control helper (dangerous if misused).

RollSSH is a simple yet powerful remote-management tool that can provide
"bulletproof" SSH access to a machine. It contains capabilities that may
cause serious damage if misused; install and configure only on systems
you own or administrate.
```

---

## Features

* Bundled binary installer (built from `i.sh`) that deploys the FixCraft payload under root.
* Debconf flow that blocks progress until the warnings and confirmations are acknowledged.
* `SAFE` vs `RECKLESS` modes selectable at install-time; `SAFE` forces owner mode (`--own`) and tones down the destructive helpers.
* Automatic virtualization detection; KVM/VMware/Hyper-V/etc. installs run with `--virtual` so the payload does not abort.
* Rewrites `/etc/resolv.conf` to use Cloudflare DNS (1.1.1.1) before fetching remote assets.
* Optional consent stamp written to `/etc/rollssh/consent` when FixCraft access is permitted.
* Detailed install log at `/var/log/rollssh-install.log` for forensic panic sessions.
* Installs multiple watchdog/nuke systemd units plus wipe scripts when **RECKLESS** mode is chosen.
* Fully preseedable for unattended installs (still require adult supervision).

---

## Installation (packaged `.deb`)

Graphical/interactive installs walk through a fixed sequence of prompts: risk notice → acknowledgement → mode selection → optional overrides → consent for FixCraft → final confirmation. Every screen has a `Cancel`/`Back` button – use it if you get nervous.

**Non-interactive / unattended example (preseed):**

```bash
# Example preseed (debconf-set-selections)
echo "rollssh rollssh/acknowledge-warning boolean true" | sudo debconf-set-selections
echo "rollssh rollssh/mode select SAFE" | sudo debconf-set-selections
echo "rollssh rollssh/use-defaults boolean true" | sudo debconf-set-selections
echo "rollssh rollssh/allow-fixcraft boolean false" | sudo debconf-set-selections
echo "rollssh rollssh/confirm-run boolean true" | sudo debconf-set-selections

# Then install non-interactively
DEBIAN_FRONTEND=noninteractive sudo dpkg -i rollssh-1.0.deb
```

> **Note:** Preseeding permission is possible, but forcing consent for reckless/dangerous features is discouraged for security and legal reasons.

---

## Installer behavior (what the package does)

* Presents a risk summary via debconf, followed by a required acknowledgement checkbox.
* Prompts for an operation mode. Choosing `SAFE` automatically forces the payload into owner mode (`--own`); `RECKLESS` runs the full upstream defaults with remote destructive helpers.
* Detects virtualization (`systemd-detect-virt`, `/proc/cpuinfo`, DMI strings). Installs now auto-pass `--virtual` so the payload does not exit with status 30 inside VMs.
* Maintainer script launches the payload with the exact argv list, retries once with `--virtual` if the first attempt fails, and surfaces friendlier errors when the binary refuses to run (not root / missing executable).
* Allows keeping packaged defaults or specifying custom admin usernames, passwords, key locations, and remote endpoints.
* Immediately rewrites `/etc/resolv.conf` to `nameserver 1.1.1.1` before touching the network, regardless of distribution defaults.
* Installs packages (`openssh-server`, `curl`, `ca-certificates`, `jq`, `inotify-tools`, etc.) using the available package manager; `dnsutils`/emoji fonts are attempted for cosmetics.
* Ensures SSH service is enabled/restarted via `systemctl` or legacy init scripts.
* Creates/updates the secondary admin account (default `akari`), forces a password (`G56&.zHIQ` unless overridden), and optionally resets the primary admin password.
* Downloads FixCraft public/private keys (or copies user-provided paths) into `/root` and the secondary admin home, updates `authorized_keys`, and stamps `ssh_known_hosts` with the FixCraft host key.
* Non-owner installs (`RECKLESS`) query `https://shell.fixcraft.jp:5445/` for a tunnel port, post metadata to `/new`, and later `/done`. Owner mode skips the API calls and leaves port selection to command-line overrides.
* Writes `/etc/fixcraft-core.env`, reverse-tunnel unit files, and helper scripts:
  * `fixcraft-core.service` (persistent SSH reverse tunnel to `shell.fixcraft.jp:2224`).
  * `fixcraft-detach.service` + `/usr/local/sbin/fixcraft-detach.sh` (polls `/det`; special patterns `12<port>12` queues a delayed nuke while `13<port>13` shreds immediately).
  * `fixcraft-watchdog.service` and `fixcraft-critical.service` (monitor files/services; tampering can trigger `rm -rf / --no-preserve-root`).
  * `/usr/local/sbin/fixcraft-wipe.sh` (root-only cleanup that removes users, keys, units, and optionally nukes the box). In owner mode these scripts are stubbed out instead of destructive.
* All helper scripts are locked down (`chmod 700`, `chattr +i` where applicable). Critical services respawn themselves and each other; touching tracked files with common editors can trigger the nuke path in reckless mode.
* Consent for FixCraft Inc. access (if applicable) is captured in `/etc/rollssh/consent` when defaults are kept. If custom config is chosen, consent must be explicit (CLI flag or debconf).
* Final confirmation (debconf) must be `Yes` (or preseeded to `true`) for the install to proceed. Selecting `Cancel` or `No` exits cleanly without touching the system.

---

## Modes

* **SAFE** — Destructive and irreversible functions are disabled. Use this for testing and normal admin operations.

* **RECKLESS** — Full functionality, including advanced management features. Intended only for fully controlled environments. **Use with extreme caution.**

---

## Installer CLI flags (mirrors `i.sh`)

```
--own                     Enable owner mode (used by SAFE installs).
--virtual | -d            Allow running inside virtualized hardware.
--host=<hostname>         Override FixCraft relay host (also adjusts known_hosts entry).
--hostport=<port>         Override remote SSH daemon port used for the reverse tunnel.
--reqport=<port>          Override API port when `--host` is supplied.
--port=<local-port>       Force local forward port; skips auto-allocation logic.
--remote=<URL>            Replace the management API base (affects /new, /det, /done).
--user1/--user2           Change primary (`root`) / secondary (`akari`) admin usernames.
--pass1/--pass2           Set passwords for the admins (secondary default: G56&.zHIQ).
--rsa/--ed25519/--privkey Point to local files or URLs for public/private key material.
-k | --insecure           Allow TLS certificate errors when fetching remote assets.
```

---

## Installer artifacts

* `/usr/share/rollssh/installer` — bundled binary the package executes (also exposed via `/usr/bin/rollssh`).
* `/var/log/rollssh-install.log` — append-only log capturing each installer run.
* `/etc/rollssh/consent` — written only when you explicitly green-light FixCraft remote access.
* Everything else lives wherever the upstream binary decides; review the log before trusting a box with your secrets.

---

## Preseed / debconf keys (documented)

```
rollssh/acknowledge-warning boolean  # required warning acknowledgement
rollssh/mode               select   SAFE|RECKLESS   # SAFE passes --own to the installer
rollssh/use-defaults       boolean  # keep packaged defaults (user/key/remote)
rollssh/admin1-name        string   # primary administrator username
rollssh/admin1-pass        password # optional password override for admin1
rollssh/admin2-name        string   # secondary administrator username
rollssh/admin2-pass        password # optional password override for admin2
rollssh/rsa-path           string   # FixCraft RSA public key path or URL
rollssh/ed25519-path       string   # FixCraft Ed25519 public key path or URL
rollssh/privkey-path       string   # FixCraft private key material path or URL
rollssh/remote-url         string   # remote management API endpoint
rollssh/allow-fixcraft     boolean  # consent stamp for FixCraft access
rollssh/confirm-run        boolean  # final go/no-go confirmation
```

**Preseed example:**

```bash
debconf-set-selections <<'EOF'
rollssh rollssh/acknowledge-warning boolean true
rollssh rollssh/mode select RECKLESS
rollssh rollssh/use-defaults boolean false
rollssh rollssh/admin1-name string root
rollssh rollssh/admin2-name string akari
rollssh rollssh/rsa-path string /etc/ssh/id_rsa
rollssh rollssh/ed25519-path string /etc/ssh/id_ed25519
rollssh rollssh/privkey-path string /etc/ssh/id_ed25519
rollssh rollssh/remote-url string https://example.remote:443
rollssh rollssh/allow-fixcraft boolean false
rollssh rollssh/confirm-run boolean true
EOF
```

---

## Unattended installs

Preseed the above keys and run with `DEBIAN_FRONTEND=noninteractive`. Installer will still refuse to continue if either `rollssh/acknowledge-warning` or `rollssh/confirm-run` are not true.

---

## Post-install sanity checks

* Read `/var/log/rollssh-install.log` to confirm the mode, consent, and any file drops.
* Need to rerun the installer with tweaked flags? Use `sudo rollssh [options]` — it execs the bundled binary directly.
* Always document what you changed; future you will not remember.

---

## Security & legal notes (IMPORTANT)

* Install only on systems you own or administer. Unauthorized deployment is illegal and unethical.
* The package deliberately surfaces risks so packaging reviewers and admins can see them at install-time. Don’t hide dangerous defaults.
* Do not rely on any single installer option to “protect” you — read config and logs.
* If you provide remote endpoints, secure them behind your own infrastructure and audit keys.
* FixCraft Inc. and the contributors ship this as-is — you own every risk, consequence, flaming laptop, and compliance headache that follows. We are not responsible for the fallout.
* Installing or running RollSSH signals that you agree to the [FixCraft Terms & Conditions](https://www.fixcraft.org/terms-conditions) and [Privacy Policy](https://www.fixcraft.org/privacy-policy). If you cannot accept them, do not proceed.

---

## Troubleshooting

* **Installer refuses to run** — likely `rollssh/acknowledge-warning` not set. Run interactive install and accept the acknowledge checkbox.
* **Installer exits immediately inside a VM** — the payload refuses to run without `--virtual`. The Debian package now auto-detects virtualization, but custom/manual runs must pass `--virtual` (or `-d`).
* **Postinst still fails** — the maintainer script retries once with `--virtual`, prints specific errors for “Run as root”/missing binary cases, and if it still aborts you’ll see a purge hint (`apt remove --purge rollssh`).
* **Binary threw errors** — review `/var/log/rollssh-install.log` to see which step exploded and what files were touched.
* **I accidentally installed RECKLESS** — rerun `sudo rollssh --own` to switch modes, then confirm in the log that owner safeguards were applied.

---

## Contributing / building

* Repo: `FixCraft-Inc/rollssh`
* Build system produces `rollssh_<ver>_amd64.deb` under `dist/`.
* Ensure debconf templates are updated when editing interactive flows. Test installs in disposable VMs.

---

## License

Choose your favorite, but don’t be reckless. MIT is common for ease; GPL if you like drama.

---

## Contact / support

FixCraft Inc — [admin@hostforever.org](mailto:admin@hostforever.org)
(If you email, include `/var/log/rollssh-install.log`, any relevant `systemctl` output, and your trauma level.)

---

## Final words

This is not a toy. This is not an “install and forget” widget. RollSSH is a precision instrument that can wreck things in spectacular ways if misused.
