# Building from Source

This covers building Mint Calendar (Community Edition) yourself — either a local dev build, or
a `.deb` for Linux Mint Cinnamon. For coding style, see [HACKING.md](HACKING.md); for the
contribution process, including how AI-assisted contributions are handled, see
[CONTRIBUTING.md](CONTRIBUTING.md).

## Before you start: version requirements

This tree is rebased onto upstream GNOME Calendar's tagged **v50.0 stable** release (it
previously tracked upstream's `main` branch mid-development, at `51.beta` — see
[README.md](README.md) for why that changed). Per [`meson.build`](meson.build), v50.0 itself
still needs newer GTK4 and libadwaita than Linux Mint 22 ships out of the box — rebasing to
stable did not lower this floor, since these version requirements were already present before
the fork started tracking `main`:

| Dependency | Required       | Mint 22 / Ubuntu 24.04 (Noble) ships |
|------------|----------------|---------------------------------------|
| GTK4       | `>= 4.21.2`    | `4.14.5`                              |
| libadwaita | `>= 1.8~alpha` | `1.5.0`                               |
| GLib       | `>= 2.80.0`    | —                                      |

Neither version is in Mint's or Ubuntu Noble's own repos — this isn't a Mint 22 point-release
gap that will close on its own, since LTS releases deliberately hold GTK4/libadwaita at their
initial version for the life of the release (bugfixes only, no feature bumps). Two ways to
actually satisfy the build:

**Option A — third-party backport PPA (what CI uses).** [`ppa:savoury1/gtk4`](https://launchpad.net/~savoury1/+archive/ubuntu/gtk4)
is a long-running, actively maintained community PPA that backports GTK4 (and libadwaita) to
Ubuntu LTS bases, including Noble:

```sh
sudo add-apt-repository ppa:savoury1/gtk4
sudo apt update
```

I confirmed this PPA exists, is currently maintained for Noble specifically (350+ source
packages as of its most recent update note), and is a well-known name in the Ubuntu/Mint
backport community — but I could not confirm the *exact* GTK4/libadwaita version numbers it
currently publishes (Launchpad blocks automated fetches of that page). **Check the actual
published versions there before relying on this** — `apt-cache policy libgtk-4-dev` after
adding the PPA will show you what's actually on offer. If it hasn't caught up to `4.21.2` yet,
this won't resolve the gap and you'll need Option B, or to wait.

It's a third-party PPA, not an official Ubuntu/Mint/GNOME source — the standard cautions apply
(review before adding, and per the maintainer's own notes, avoid combining with other
third-party GTK/GNOME PPAs on the same system).

**Option B — build GTK4/libadwaita yourself.** Slower and more involved, but doesn't depend on
a third party: build newer GTK4 and libadwaita from source against Noble, install them into
`/usr/local` (or a prefix on `PKG_CONFIG_PATH`), then proceed with the build below normally.

Once either is in place, the ordinary `apt build-dep` / `mk-build-deps` flow below picks up the
newer versions automatically — apt prefers the higher version number, no pinning needed.

The full, versioned dependency list — including libical, evolution-data-server, gweather,
geoclue, and the rest — lives in [`meson.build`](meson.build)'s `dependency()` calls and in
[`debian/control`](debian/control)'s `Build-Depends`; both are kept in sync.

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

This is what the release workflow runs on every `v*` tag push, inside a
`linuxmintd/mint22-amd64` container:

```sh
# Packaging tools
sudo apt-get install -y build-essential devscripts equivs dpkg-dev fakeroot software-properties-common

# Satisfy the GTK4/libadwaita version floor — see "Before you start" above
sudo add-apt-repository ppa:savoury1/gtk4
sudo apt update

# Pull in this package's Build-Depends (see debian/control)
sudo mk-build-deps --install --remove \
  --tool="apt-get -y --no-install-recommends" debian/control

# Build
dpkg-buildpackage -us -uc -b

# Result lands one directory up
ls ../*.deb
```

If `mk-build-deps` still can't resolve `libgtk-4-dev`, `libadwaita-1-dev`, or
`blueprint-compiler` after adding the PPA, that means the PPA's currently-published versions
haven't caught up to what this package needs yet — check with `apt-cache policy libgtk-4-dev`
(see "Before you start" above). It's not a packaging bug in this repo. See
[debian/control](debian/control) for the full, versioned Build-Depends list.

Install the resulting package with:

```sh
sudo apt install ./mint-calendar-community-edition_*.deb
```

## Icons and branding

The application icon, launcher entry (`data/org.gnome.Calendar.desktop.in.in`), and app ID
(`org.mint.calender.community.edition`, set in `meson.build`) were restyled for this fork —
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
Settings launcher in `src/utils/gcal-utils.c`), but it has **not** been compiled end-to-end in
the environment it was produced in — no network access and no GTK4/libadwaita toolchain were
available there. Run a full dev build and `meson test` (and ideally the `.deb` build) before
tagging a release.
