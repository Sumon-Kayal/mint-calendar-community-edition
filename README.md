<div align="center">
  <img src="data/icons/hicolor/scalable/apps/org.mint.calender.community.edition.svg" width="128" height="128">

  # Mint Calendar (Community Edition)

  Manage your schedule

  <img width="922" src="https://static.gnome.org/appdata/gnome-50/org.gnome.Calendar/month-view.png">
</div>

Mint Calendar (Community Edition) is a community-maintained, Linux Mint-flavored fork of
[GNOME Calendar](https://apps.gnome.org/Calendar/), tracking upstream GNOME Calendar development
and packaged independently for distribution outside the GNOME and Linux Mint release channels.

> [!WARNING]
> This build tracks upstream's `main` branch (currently `51.beta`), not a tagged stable release.
> It includes GNOME Calendar's still-in-progress internals rewrite and requires notably newer
> GTK4/libadwaita than Mint's own `48.1+mint1` release (GTK4 >= 4.21.2, libadwaita >= 1.8~alpha,
> plus a new `blueprint-compiler` build dependency). Whether your Mint/Ubuntu system's repos
> already carry those versions hasn't been verified — check before relying on this build, and
> expect the first CI run to surface anything that doesn't resolve cleanly.

## Installing

Built for Linux Mint Cinnamon (Ubuntu base). Grab the latest `.deb` from the
[Releases page](https://github.com/Sumon-Kayal/mint-calendar-community-edition/releases) and install it with:

```sh
sudo apt install ./mint-calendar-community-edition_*.deb
```

Each release is built and published automatically by
[.github/workflows/release-deb.yml](.github/workflows/release-deb.yml).

## Links

- Homepage: <https://github.com/Sumon-Kayal/mint-calendar-community-edition>
- Report an issue: <https://github.com/Sumon-Kayal/mint-calendar-community-edition/issues>
- Contact: <sumankayalsuman4@proton.me>

## Credits

Mint Calendar (Community Edition) is based on [GNOME Calendar](https://gitlab.gnome.org/GNOME/gnome-calendar)
and carries forward [Linux Mint's desktop-environment-compatibility patch](https://gitlab.com/linuxmint/pins/mint/gnome-calendar)
on top of current upstream. All credit for the original application goes to the GNOME Calendar
authors and the Linux Mint team; this repository adapts their work for independent, community
distribution.

## Contributing

For information on how to contribute, please check [CONTRIBUTING.md](CONTRIBUTING.md) and [HACKING.md](HACKING.md).
