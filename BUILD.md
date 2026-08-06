# Building from Source

This covers building Mint Calendar (Community Edition) yourself — either a local dev build, or
a `.deb` for Linux Mint Cinnamon. For coding style, see [HACKING.md](HACKING.md); for the
contribution process, including how AI-assisted contributions are handled, see
[CONTRIBUTING.md](CONTRIBUTING.md).

## Before you start: version requirements

This tree is rebased onto upstream GNOME Calendar's tagged **v50.0 stable** release (it
previously tracked upstream's `main` branch mid-development, at `51.beta` — see
[README.md](README.md) for why that changed). Per [`meson.build`](meson.build), v50.0 needs
newer GTK4 and libadwaita than Linux Mint 22 / Ubuntu 24.04 (Noble) ship out of the box:

| Dependency | Required       | Noble ships | Ubuntu 26.04 (resolute) ships |
|------------|----------------|-------------|--------------------------------|
| GTK4       | `>= 4.21.2`    | `4.14.5`    | `4.22.2` (`4.22.4` in -updates) |
| libadwaita | `>= 1.8~alpha` | `1.5.0`     | `1.8.0`                        |
| GLib       | `>= 2.80.0`    | —           | —                               |

Two things were tried and ruled out before landing on the current approach:
[`ppa:savoury1/gtk4`](https://launchpad.net/~savoury1/+archive/ubuntu/gtk4) does not work —
confirmed via a real CI run, it 404s for the Noble suite, and its actual purpose (per its own
description) is backporting an *old* GTK4 (4.8.3) to ancient LTS releases, not providing a
current one for Noble. Building GTK4/libadwaita from source on Noble was considered but not
attempted — it would pull in its own cascade of newer-than-Noble build dependencies.

**Current approach: build on Ubuntu 26.04 instead of Mint 22.** Verified directly against
Ubuntu's own package database (table above) that resolute's own repos already satisfy this
project's floor — no third-party PPA or from-source build needed. This is what
[`.github/workflows/release-deb.yml`](.github/workflows/release-deb.yml) and
[`.github/workflows/codeql.yml`](.github/workflows/codeql.yml) both do now (`runs-on:
ubuntu-26.04`), and it's the approach the local build instructions below use too.

**The real tradeoff, stated plainly:** a `.deb` built against Ubuntu 26.04's libraries will
very likely **not** install/run correctly on an unmodified current Mint 22 system — this
targets Ubuntu 26.04-based systems (and whatever future Mint release ends up built on that
base), not current Mint 22. If installing on *today's* Mint 22 specifically is the actual goal,
this doesn't solve that; the remaining paths for that are building GTK4/libadwaita from source
against Noble yourself, or reconsidering Flatpak/Snap (the industry-standard answer to exactly
this problem, though deliberately removed from this project earlier — not undoing that
unilaterally, just naming it).

None of this has been confirmed by an actual successful run yet — see "Before opening a
release" below.

The full, versioned dependency list — including libical, evolution-data-server, gweather,
geoclue, and the rest — lives in [`meson.build`](meson.build)'s `dependency()` calls and in
[`debian/control`](debian/control)'s `Build-Depends`; both are kept in sync.

## Local build with Distrobox (Ubuntu 26.04)

Since the project now targets Ubuntu 26.04's own library versions (see above), the most direct
way to build and test locally — including on a Mint 22 host — is
[Distrobox](https://github.com/89luca89/distrobox): it creates a full Ubuntu 26.04 container
that shares your home directory and integrates with the host desktop, without touching the
host system's own packages.

```sh
# Install distrobox and a container backend, if you don't have them
sudo apt install distrobox podman

# Create an Ubuntu 26.04 container
distrobox create --name mint-calendar-2604 --image ubuntu:26.04

# Enter it — your home directory (including this repo, wherever it lives under $HOME)
# is already available inside
distrobox enter mint-calendar-2604
```

From here you're inside an Ubuntu 26.04 environment — everything in "Dev build" and "Debian
package build" below works as written, using the version-requirements table above directly
(no PPA, no source build). Exit the container with `exit`; re-enter any time with
`distrobox enter mint-calendar-2604`.

**What this does and doesn't solve:** Distrobox gets you a real Ubuntu 26.04 build/test
environment on your existing Mint 22 machine — genuinely useful for local development, and for
running the app itself day-to-day (`distrobox-export --app <binary>` adds it to your host's
app menu, still running inside the container's environment). It does **not** turn a `.deb`
built inside the container into something that installs cleanly outside it on bare Mint 22, for
the same library-version reasons described above — this is a development environment, not a
distribution mechanism for Mint 22 users.

## Dev build (meson)

For local development or testing, without producing a `.deb`:

```sh
meson setup builddir
ninja -C builddir
```

Install into a prefix with `ninja -C builddir install`, or run uninstalled via
`meson devenv -C builddir`. Run the test suite (`meson test -C builddir`) before opening a
pull request, per [CONTRIBUTING.md](CONTRIBUTING.md).

A few build options worth knowing about (see [`meson_options.txt`](meson_options.txt) for the
full list):

```sh
# Development profile build (installs alongside a stable version, distinct app ID/icon)
meson setup builddir -Dprofile=development

# Enable extra debugging/tracing output
meson setup builddir -Dtracing=true
```

## Debian package build

This is what the release workflow runs on every `v*` tag push, on an `ubuntu-26.04` runner (or
inside the Distrobox container above, on a Mint 22 host):

```sh
# Packaging tools
sudo apt-get install -y build-essential devscripts equivs dpkg-dev fakeroot

# Pull in this package's Build-Depends (see debian/control) — resolute's own repos already
# satisfy the GTK4/libadwaita floor, so no extra setup is needed first
sudo mk-build-deps --install --remove \
  --tool="apt-get -y --no-install-recommends" debian/control

# Build
dpkg-buildpackage -us -uc -b

# Result lands one directory up
ls ../*.deb
```

If `mk-build-deps` can't resolve `libgtk-4-dev`, `libadwaita-1-dev`, or `blueprint-compiler`
here, something's off from what's documented above — resolute's own repos should have these
directly. See [debian/control](debian/control) for the full, versioned Build-Depends list.

Install the resulting package with:

```sh
sudo apt install ./mint-calendar-community-edition_*.deb
```

## Icons and branding

The application icon, launcher entry (`data/org.gnome.Calendar.desktop.in.in`), and app ID
(`org.mint.calendar.community.edition`, set in `meson.build`) were restyled for this fork —
flat, Mint-Y-Dark-inspired palette in place of upstream's gradient purple. The symbolic icons
(`data/icons/hicolor/symbolic/`) were left as generic single-fill icons deliberately: GTK
recolors symbolic icons automatically based on the active theme, so hardcoding a dark-mode
color into them would actually break that adaptive behavior rather than help it.

If you have access to Linux Mint's actual Mint-Y-Dark theme assets and want a closer palette
match than this best-effort pass, `data/icons/hicolor/scalable/apps/` is where to start — see
the comments at the top of the app icon's `.svg` file.

## Before opening a release

This rebase was carried out at the source level and cross-checked against Linux Mint's own
`gnome-calendar` pin for the one behavioral patch that matters (the desktop-environment-aware
Settings launcher in `src/utils/gcal-utils.c`), but it has **not** been compiled end-to-end
anywhere yet. The GTK4/libadwaita version floor is now confirmed satisfiable on Ubuntu 26.04
(table above, checked directly against Ubuntu's package database) — the CI workflows and the
Distrobox instructions above both build on that basis — but no actual run has confirmed a full
build succeeds yet. Run a full dev build and `meson test` (and ideally the `.deb` build,
locally via Distrobox or by pushing a tag) before relying on it.
