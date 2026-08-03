# Changelog

## [1.0] - 2026-08-02

First release of Mint Calendar (Community Edition) as its own independently-versioned project.
Based on GNOME Calendar's latest stable release (v50), with Linux Mint's compatibility patch
applied and independent Mint-Y-Dark branding. Releases build and publish automatically via
GitHub Actions, with CodeQL security scanning on every change. App ID is
`org.mint.calendar.community.edition` (corrected from an earlier `calender` misspelling).

This is the first release — it hasn't been build-tested end to end yet. See
[BUILD.md](BUILD.md) before relying on it.

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
