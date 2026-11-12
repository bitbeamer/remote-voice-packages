# Install Remote Voice companion packages

Use the repository for your exact distribution/release and architecture. Native
build success is recorded in each manifest; it does not establish installation or
audio qualification. openSUSE is deferred and has no published packages. Mint and Arch Linux ARM are
not qualified by the
Ubuntu, Leap or Arch x86_64 builds. macOS signing/notarization is deferred; no
unsigned Mac installer is published here.

Public signing key: https://bitbeamer.github.io/remote-voice-packages/remote-voice.asc

Verify its full fingerprint before trusting it:
`9E17 C0F1 53DE B9C1 2BAC 63D7 0F85 7067 D7AD 4C77`.
The key expires on 2027-11-12. The private signing key is not part of this repository.

## Debian and Ubuntu

Download the public key, inspect its fingerprint with `gpg --show-keys`, and
install it as `/etc/apt/keyrings/remote-voice.asc` with administrator authorization.
Create `/etc/apt/sources.list.d/remote-voice.sources` using this Debian 13 amd64
example; use the matching distribution/version/architecture path for other builds:

```text
Types: deb
URIs: https://bitbeamer.github.io/remote-voice-packages/apt/debian/13/amd64/
Suites: ./
Signed-By: /etc/apt/keyrings/remote-voice.asc
```

Run `sudo apt-get update` and `sudo apt-get install remote-voice`. Future updates
use the same package manager. No application-owned updater or remote installation
API is involved. Keep your existing private configuration, state and profile IDs.

## Fedora

Use `rpm/fedora/44/amd64/` (replace `amd64` with
`arm64` for an aarch64 host). Configure the native package manager's repository
with that HTTPS base URL, `gpgcheck=1`, `repo_gpgcheck=1`, and the public key URL
above. Both package signatures and repository metadata are signed. Install/update
`remote-voice` through DNF with normal administrator authorization.

## Arch Linux x86_64

Import and locally trust the public signing key using the normal `pacman-key`
procedure after checking its fingerprint. Add this repository to pacman.conf:

```ini
[remote-voice]
SigLevel = Required DatabaseRequired
Server = https://bitbeamer.github.io/remote-voice-packages/pacman/arch/x86_64
```

Use a normal full-system `pacman -Syu` transaction to install/update the companion.
Do not perform a partial Arch system upgrade. The package and repository database
have standard pacman signatures.

## Setup and service

Package installation does not choose listeners, alter firewall/default audio
settings, enable a service or start capture. Configure the companion as the normal
service user with `remote-voice setup` and explicitly selected listeners. Use its
ordinary `service` controls when a restart is needed after an update. Compatibility
prompts in the client direct users back to these normal installation tools.
