# Changelog

## [1.0] - 2026-08-02

First release of Mint Calendar (Community Edition) as its own independently-versioned project.
Based on GNOME Calendar's latest stable release (v50), with Linux Mint's compatibility patch
applied and independent Mint-Y-Dark branding. Releases build and publish automatically via
GitHub Actions, with CodeQL security scanning on every change. App ID is
`org.mint.calendar.community.edition` (corrected from an earlier `calender` misspelling).

A real CI run confirmed the build fails at the dependency-install step on Mint 22 / Ubuntu
24.04 — GTK4/libadwaita new enough for this release aren't available there, and the backport
PPA first tried for it doesn't work either. CI and the local build instructions now target
Ubuntu 26.04 instead, whose own repos do satisfy the requirement (verified directly against
Ubuntu's package database). That comes with a real tradeoff: a `.deb` built this way targets
Ubuntu 26.04-based systems, not current Mint 22. See [BUILD.md](BUILD.md) for the full picture,
including a Distrobox-based way to build against Ubuntu 26.04 locally on a Mint 22 host.

## How this compares

| | Linux Mint's `gnome-calendar` | GNOME Calendar (stock) | Mint Calendar (Community Edition) |
|---|---|---|---|
| Based on | GNOME Calendar 48 | GNOME Calendar 50 | GNOME Calendar 50 |
| Linux Mint's compatibility patch | Yes | No | Yes |
| Branding | Stock GNOME | Stock GNOME | Mint-branded, Mint-Y-Dark icon |
| Official Mint project | Yes | — | No — independent/community |
| Where to get it | Mint's own repos | GNOME / Flathub | This project's [Releases page](https://github.com/Sumon-Kayal/mint-calendar-community-edition/releases) |

In short: the newer GNOME Calendar, with the same Mint compatibility patch Mint's own package
uses, under independent Mint branding.
