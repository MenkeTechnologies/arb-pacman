```
██████╗  █████╗  ██████╗███╗   ███╗ █████╗ ███╗   ██╗
██╔══██╗██╔══██╗██╔════╝████╗ ████║██╔══██╗████╗  ██║
██████╔╝███████║██║     ██╔████╔██║███████║██╔██╗ ██║
██╔═══╝ ██╔══██║██║     ██║╚██╔╝██║██╔══██║██║╚██╗██║
██║     ██║  ██║╚██████╗██║ ╚═╝ ██║██║  ██║██║ ╚████║
╚═╝     ╚═╝  ╚═╝ ╚═════╝╚═╝     ╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝
```

[![arb](https://img.shields.io/badge/arb-package-05d9e8?style=flat-square)](https://github.com/MenkeTechnologies/arb)
![type](https://img.shields.io/badge/self--sourcing-dashboard-9b5de5?style=flat-square)
![license](https://img.shields.io/badge/license-MIT-ff2a6d?style=flat-square)

### `[OUT-OF-DATE PACMAN PACKAGES, WITH A LIVE COUNT]`

> *"The pending-update queue, watched — every ten seconds, without you asking."*

A `pacman` dashboard that lists the installed packages with pending updates and
keeps a live count of them beside the list. It is **self-sourcing** — it runs
`pacman -Qu` on its own timer, so nothing needs to be piped in; open it and it
starts polling. It is an `arb` package — a dashboard for
[`arb`](https://github.com/MenkeTechnologies/arb), the pipeline-to-TUI language
on the fusevm bytecode VM.

### [`arb`](https://github.com/MenkeTechnologies/arb) &middot; [`arb-registry`](https://github.com/MenkeTechnologies/arb-registry)

---

## Table of Contents

- [\[0x00\] Overview](#0x00-overview)
- [\[0x01\] Install](#0x01-install)
- [\[0x02\] Usage](#0x02-usage)
- [\[0x03\] The Spec](#0x03-the-spec)
- [\[0x04\] How It Works](#0x04-how-it-works)
- [\[0xFF\] License](#0xff-license)

---

## [0x00] OVERVIEW

`pacman -Qu` prints a flat block of text you have to re-run and re-read. This
dashboard turns that into a live view: the packages with pending updates in a
list, their count in a number beside it, both refreshing on a fixed interval.
It answers "how far behind is this box, and on what" without a manual command
and without piping anything in.

---

## [0x01] INSTALL

```sh
arb install pacman      # from the arb-registry git index
arb search pacman
```

---

## [0x02] USAGE

```sh
arb pacman                         # run it (self-sourcing; polls on its own timer)
arb pacman --serve                 # browser dashboard
arb pacman --html > pacman.html    # static HTML snapshot
```

---

## [0x03] THE SPEC

```arb
# pacman — Pacman packages with pending updates, self-sourced from `pacman -Qu`.
! pacman -Qu every 10s
list .pacman
source .pacman { in }
text .n
source .n { in; count }
grid .n -row 1 -col 0
grid .pacman -row 0 -col 0
```

- `! pacman -Qu every 10s` — the self-source. arb runs `pacman -Qu` every 10
  seconds and feeds its stdout into the stream the widgets read from; no
  external pipe is required.
- `list .pacman` / `source .pacman { in }` — a list widget named `.pacman`,
  whose query pipeline is `in`, passing the sourced lines straight through so
  each package with a pending update is a row.
- `text .n` / `source .n { in; count }` — a text widget named `.n` fed by the
  same stream; its pipeline takes `in` and applies `count`, so it renders the
  number of packages with pending updates.
- `grid .pacman -row 0 -col 0` / `grid .n -row 1 -col 0` — grid placement: the
  list on the top row, the count directly beneath it in the same column.

---

## [0x04] HOW IT WORKS

The `!` self-source runs `pacman -Qu` on the 10-second timer and feeds its
stdout into arb's stream. Each widget's `source` block is a query pipeline over
that stream: `.pacman` passes the lines through with `in` for the list, while
`.n` runs `in; count` to reduce them to a number. arb lowers the compute — the
`count` reduction — to `fusevm` bytecode executed on the fusevm VM with a
Cranelift JIT, then renders both widgets into the grid.

---

## [0xFF] LICENSE

MIT.
