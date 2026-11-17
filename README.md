# paseo-packages

Unofficial APT and DNF repositories for the [Paseo](https://github.com/getpaseo/paseo) desktop
releases, so Paseo updates with the rest of the system.

Setup: **https://hazptr.github.io/paseo-packages/**

```sh
# Fedora / RHEL / openSUSE
sudo curl -fsSLo /etc/yum.repos.d/paseo.repo https://hazptr.github.io/paseo-packages/paseo.repo
sudo dnf install Paseo

# Debian / Ubuntu
sudo install -d /etc/apt/keyrings
sudo curl -fsSLo /etc/apt/keyrings/paseo.gpg https://hazptr.github.io/paseo-packages/paseo.gpg
sudo curl -fsSLo /etc/apt/sources.list.d/paseo.list https://hazptr.github.io/paseo-packages/paseo.list
sudo apt update && sudo apt install paseo
```

### Turn off Paseo's own updater

Paseo has no setting for it: it auto-downloads new versions and installs them on quit. Its
updater (electron-updater) only handles deb/rpm installs when `/opt/Paseo/resources/package-type`
exists, so exclude that file from installs, then delete the copy already there:

```sh
# Fedora / RHEL / openSUSE
echo '%_netsharedpath /opt/Paseo/resources/package-type' | sudo tee /etc/rpm/macros.paseo-no-updater
# Debian / Ubuntu
echo 'path-exclude=/opt/Paseo/resources/package-type' | sudo tee /etc/dpkg/dpkg.cfg.d/paseo-no-updater
# both
sudo rm -f /opt/Paseo/resources/package-type
```

## How it works

Nothing is re-hosted. A scheduled workflow (`.github/workflows/update.yml`, every 3 hours) checks
upstream for new stable releases, downloads the newest three, and builds signed indexes whose
entries point at the upstream release assets:

- **RPM**: `repodata/` on GitHub Pages, built with `createrepo_c --baseurl`, so each package's
  `xml:base` is `https://github.com/getpaseo/paseo/releases/download/`. `repomd.xml` is signed
  (`repo_gpgcheck=1`); the upstream RPMs themselves are unsigned (`gpgcheck=0`).
- **APT**: a flat repository whose index (`InRelease`, `Packages`) is attached to this repo's
  [`apt` release](https://github.com/hazptr/paseo-packages/releases/tag/apt). The archive root is
  `https://github.com/`, so `Filename:` resolves to the upstream `.deb`.

The checksums in both indexes are signed, so a changed upstream asset fails to install.

Signing key: `BB7D A1A4 7F09 DE1C 4667  D46C 0ECE 9DDF 0722 B2C5`
([paseo.asc](https://hazptr.github.io/paseo-packages/paseo.asc)), stored as the
`GPG_PRIVATE_KEY` Actions secret.

Run the workflow by hand with **force** to rebuild without a new release.
