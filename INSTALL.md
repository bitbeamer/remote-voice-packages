# Install Remote Voice companion packages

This public package-repository excerpt is derived from `docs/manual.md`, the
canonical user manual in the application source repository. Update the manual
first when changing instructions. The manual publishing workflow copies this
file into the public package repository; Linux installation below is standalone
and does not require private source access.

For iOS/macOS developer installation and the complete manual, source readers can
use [the user manual](https://github.com/bitbeamer/remote_voice/blob/main/docs/manual.md).
Signed Mac installers and a public iOS release are not yet established. Published
companion binaries can predate current source; inspect `remote-voice --help` and
`remote-voice version --json` for your installation's available features.

## Install the Linux companion

Use the signed package repository for your exact distribution, release and CPU.
Published targets are Debian 13 and Ubuntu 24.04/26.04 on amd64/arm64, Fedora 44
on x86_64/aarch64, Arch Linux x86_64, and a separate Arch Linux ARM aarch64 feed.
openSUSE is deferred. Ubuntu packages do not establish Linux Mint support.
Build/publication success does not establish audio qualification on every target.

The host needs a working PipeWire/WirePlumber user session, a systemd user bus,
and the PipeWire ALSA plugin for ALSA applications. Packages declare native library
dependencies; Go, compilers and development headers are not user prerequisites.
Package installation does not replace your audio stack, enable/start the service,
open firewall ports or start capture.

### Repository signing key

Obtain the public key from
[remote-voice.asc](https://bitbeamer.github.io/remote-voice-packages/remote-voice.asc).
Before importing it into a package manager, inspect its full fingerprint with
`gpg --show-keys --with-fingerprint remote-voice.asc` and compare it with:

```text
9E17 C0F1 53DE B9C1 2BAC 63D7 0F85 7067 D7AD 4C77
```

The recorded key expires on 2027-11-12. Resolve any fingerprint mismatch through
a trusted maintainer channel; do not disable signature checks.

### Debian and Ubuntu

Install the verified ASCII public key as `/etc/apt/keyrings/remote-voice.asc`,
readable by APT. For example, after downloading and checking it:

```sh
sudo install -d -m 755 /etc/apt/keyrings
sudo install -m 644 remote-voice.asc /etc/apt/keyrings/remote-voice.asc
```

Create `/etc/apt/sources.list.d/remote-voice.sources` with this content for
**Debian 13 amd64**:

```text
Types: deb
URIs: https://bitbeamer.github.io/remote-voice-packages/apt/debian/13/amd64/
Suites: ./
Signed-By: /etc/apt/keyrings/remote-voice.asc
```

For Debian arm64, use `apt/debian/13/arm64/`. Ubuntu uses
`apt/ubuntu/24.04/amd64/`, `apt/ubuntu/24.04/arm64/`,
`apt/ubuntu/26.04/amd64/` or `apt/ubuntu/26.04/arm64/` under the same HTTPS root.
Choose exactly your release and architecture, then install:

```sh
sudo apt-get update
sudo apt-get install remote-voice
```

### Fedora 44

After checking the signing key, create
`/etc/yum.repos.d/remote-voice.repo` with administrator authorization:

```ini
[remote-voice]
name=Remote Voice
baseurl=https://bitbeamer.github.io/remote-voice-packages/rpm/fedora/44/amd64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://bitbeamer.github.io/remote-voice-packages/remote-voice.asc
```

Use `arm64` in that URL for a Fedora aarch64 host. Keep package and repository
signature checks enabled, and confirm the expected key when DNF asks to import it.

```sh
sudo dnf install remote-voice
```

### Arch Linux and Arch Linux ARM

Import and locally sign only the verified key:

```sh
sudo pacman-key --add remote-voice.asc
sudo pacman-key --lsign-key 9E17C0F153DEB9C12BAC63D70F857067D7AD4C77
```

Add this repository to `/etc/pacman.conf` for **Arch Linux x86_64**:

```ini
[remote-voice]
SigLevel = Required DatabaseRequired
Server = https://bitbeamer.github.io/remote-voice-packages/pacman/arch/x86_64
```

For **Arch Linux ARM aarch64**, set Server to
`https://bitbeamer.github.io/remote-voice-packages/pacman/archlinuxarm/aarch64`.
Use a full-system transaction; partial Arch upgrades are unsupported:

```sh
sudo pacman -Syu remote-voice
```

## Set up the Linux user service

Run companion commands as the intended ordinary desktop/audio user, inside that
user's working login session. Use `sudo` for package installation only.
The package provides `/usr/bin/remote-voice` and
`/usr/lib/systemd/user/remote-voice.service`.

```sh
remote-voice endpoints
remote-voice setup --name Studio --endpoint lan@192.0.2.20:8765
```

Replace `192.0.2.20` with your actual local address from `endpoints`. Setup creates
or reuses the uniquely named profile, preserves an existing valid host identity,
saves listener settings, configures/enables/starts the packaged user service,
verifies the pinned listener and prints a sensitive single-use pairing link.
For an existing profile, use `--profile PROFILE_ID` instead of `--name Studio`.
If multiple profiles share a name, select the exact ID from `profile list`.

Setup does not start microphone capture or change desktop audio defaults.
Permit only the selected control TCP and media UDP ports in your firewall where
needed. Default ports are TCP 8765 and UDP 8766. Listener readiness does not prove
native audio is ready; inspect `remote-voice audio status` independently.

The service controls are:

```sh
remote-voice service status --json
remote-voice service stop
remote-voice service start
remote-voice service restart
remote-voice service disable
remote-voice service enable
```

`stop` ends sessions now; `disable` removes automatic startup at the user target
but does not stop an already running process. Setup already performs
`service configure` and `service enable`. For deliberate removal, stop and disable
before `service unconfigure`, then remove the package through the package manager.
Private identity, pairing and profile data are preserved by package removal.

Configure records private state/configuration paths in an owned systemd drop-in.
Foreign/edited definitions or conflicting roots are refused. Use matching absolute
`--state-dir DIR` and `--config-dir DIR` on setup, service and profile commands
when using custom storage. See [storage](#private-storage).

Stop the service before changing saved listener settings, then rerun setup with
the desired endpoints. Repeating setup after a partial failure retains saved
identity/profile state. Setup with unchanged settings can issue another link
without restarting an active service. Ordinary startup failures may happen after
a start job returns, so inspect service status when diagnosing them.

Enabling follows the user's normal login/logout/reboot policy. Remote Voice does
not enable lingering or create an audio/login session. Headless operation needs
an intentional administrator-managed session decision and working per-user audio.
The unit limits transient retries; permanent configuration exit 78 does not loop.

### Private storage

| Data | Default location |
| --- | --- |
| Runtime settings | `~/.config/remote-voice/runtime.toml` |
| Profile name, command definitions and executables | `~/.config/remote-voice/PROFILE_ID/` |
| Host key/certificate, credentials, invitations and profile slots | `~/.local/state/remote-voice/` |
| Saved app pairing credentials | Device-only Keychain |

Absolute `XDG_CONFIG_HOME`/`XDG_STATE_HOME` override the corresponding parent roots;
CLI `--config-dir`/`--state-dir` override the Remote Voice directories themselves.
Keep roots owned by the service user with mode `0700` and sensitive files `0600`.
Reject permission/symlink errors rather than loosening protection. Keep backups
private and outside Git/support bundles; they contain keys and authorization data.
Never share real pairing links, bearer tokens, raw state, SDP/ICE credentials or
audio recordings in diagnostics. Retired trust must not be automatically restored
from an older backup, particularly after suspected key compromise.

## Updates, trust and private data

Install/update through the same package manager and signed repository used above:
APT `install --only-upgrade remote-voice`, DNF `upgrade remote-voice`, or Arch's
full-system `pacman -Syu`. APT/DNF updates require an explicit user-service
restart afterward. Arch packages containing the upgrade hook automatically
`try-restart` only already-running `remote-voice.service` instances in existing
non-root user managers, including linger users. Active connections briefly
disconnect while the new binary starts. Stopped, failed, absent and masked
services stay untouched; installation/removal and unrelated package updates do
not trigger this hook. Pacman also classifies reinstalls/downgrades of an already
installed package as Upgrade operations.

The hook never enables services or creates login sessions/user managers. Offline
image/chroot installations skip service work. Unavailable buses, restart failures
or timeouts produce a pacman hook error and follow-up guidance; the package update
has already completed and is not rolled back. A timeout may leave a queued systemd
job finishing afterward. Affected users should inspect
`systemctl --user status remote-voice.service` in their own session and restart it
when ready. The hook waits at most 60 seconds overall, with a five-second forced
termination allowance; large sets of users may need manual follow-up. This source
change is available only after a package containing the hook is published; older
packages, including 0.0.1 build 3, still need an explicit service restart.

To opt out before an upgrade, an administrator can override the package hook with
`/etc/pacman.d/hooks/remote-voice-user-restart.hook` as a symlink to `/dev/null`;
then users must restart their own services explicitly after updating. No identities,
profiles or pairings are changed by package maintenance.

The app's compatibility preflight can prompt you to update through those tools;
it never installs software remotely. If status is unavailable, compatibility remains unknown
rather than being reported as a confirmed version mismatch.
