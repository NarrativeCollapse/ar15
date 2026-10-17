# AR-15 lower receiver from a forging, in LinuxCNC G-code

LinuxCNC programs (inch units) for machining an AR-15 lower receiver from
a 0% forging, and for making the fixture plates that hold it. They follow
Ray Brandes' book *Machining the AR15 Lower Receiver Forging*
(<http://www.ray-vin.com/frtech.htm>), which has the setups, drawings and
manual procedures this repository assumes you have read.

This is a rewrite, inspired by Sebastian Kuzminsky's 2010 project of the
same name. It keeps his operations, their order and the dimensions he
worked out (which come from the book and the drawing), but the code is
new, and it fixes the problems the original had, listed under
[Changes from the original](#changes-from-the-original).

## Legal

Making a firearm is regulated. Depending on where you live, the law may
limit who can make one, require a serial number or registration, or ban
unserialized receivers outright, and selling one you made may need a
license. Check the laws that apply to you before you cut metal. Nothing in
this repository is legal advice.

## Safety

**There is no warranty.** These programs move a machine that can break
tools, wreck parts and hurt people, and they are written for one person's
tooling and work holding. Before you cut anything:

- Read the comments at the top of each program: they describe the work
  holding and the datums the program expects.
- Look at the whole toolpath in the LinuxCNC preview, with your own tool
  table loaded.
- Do an air cut first, with the work offset raised well above the part.
- Run the first part with single block, and with a hand on the stop button.

Every program here has been run through the LinuxCNC 2.9.10 interpreter
(`rs274`), with an automated check that no rapid move goes sideways below
the program's clear height. Below it, the only rapids are a drill dropping
to 0.050 above its hole and a cutter dropping back to 0.050 above a floor
it has already cut. That check knows nothing about your vise, clamps or
forging.

## What you need

- **LinuxCNC 2.9** or later.
- **The subroutine library on the search path.** Add the `lib` directory
  to `SUBROUTINE_PATH` in the `[RS274NGC]` section of your INI file.
- **Your machine's settings in `lib/ar15-machine.ngc`**: tool-change
  height, where to park at the end, whether you set the spindle speed by
  hand, and the tool number of your dial indicator.
- **A tool table with lengths and diameters** for the tools below. Every
  tool change compares the table's diameter with the one the program
  expects, and stops if they differ.

## The programs, in order

| Program | What it does |
|---|---|
| `fixtures/facemill-plate.ngc` | Faces a 6 x 2.5 x 0.5 aluminium blank for a fixture plate |
| `fixtures/drivers-side-plate.ngc` | Mills the driver's side fixture plate |
| `fixtures/passenger-side-plate.ngc` | Mills the passenger's side fixture plate |
| `fixtures/plate-angle.ngc` | Cuts a plate's 10 degree edge |
| `fixtures/passenger-side-chamfer.ngc` | Chamfers the passenger's side plate |
| `lower-receiver/setup-1-deck.ngc` | Deck and buffer tower (book chapter 2) |
| `lower-receiver/setup-2-trigger-guard.ngc` | Trigger guard bosses and a temporary clamping flat (chapter 2) |
| `lower-receiver/setup-3-measure.ngc` | Measures the driver's side with a dial indicator, for setup 3 |
| `lower-receiver/setup-3-drivers-side.ngc` | Driver's side holes, slots and pockets (chapter 3) |

**Setup 4, the passenger's side (chapter 4), is not written yet.** The
original project left only a placeholder for it.

`lib/` holds the subroutines all the programs share:

| Subroutine | What it does |
|---|---|
| `ar15-machine` | Your machine's settings |
| `ar15-start` | Puts the interpreter in a known state and records the program's clear Z |
| `ar15-tool` | Tool change: spindle off, Z to the top, change, diameter check, speed prompt |
| `ar15-goto` | Moves to an X Y position at the clear Z |
| `ar15-hole` | Spot drills, peck drills or reams one hole |
| `ar15-pocket` | Mills a rectangular pocket, slot or face, in levels, with a climb-milled finishing pass |
| `ar15-end` | Spindle off, Z to the top, optional park |

## Tools

The tool numbers are set near the top of each program.

| Tool | Description | Used by |
|---|---|---|
| T1 | 0.250 4-flute end mill | driver's side plate |
| T2 | 1.500 face mill (set your own size) | facemill-plate |
| T3 | 0.375 end mill | passenger's side plate |
| T4 | 0.250 90 degree drill-mill | chamfer |
| T5 | 0.500 2-flute end mill | setup 2, setup 3 |
| T6 | 0.750 4-flute end mill | setup 1, plate-angle |
| T16 | 1/8 2-flute end mill, at least 3/8 flute length | setup 3 |
| T17 | spot drill, body narrower than 0.157 (or set `spring_spot_tool`) | setup 3 |
| T18 | 1/8 drill | setup 3 |
| T19 | #24 drill (0.152) | setup 3 |
| T20 | 5/32 reamer | setup 3 |
| T21 | #5 drill (0.2055) | setup 3 |
| T22 | letter D drill (0.246) | setup 3 |
| T23 | .251 reamer | setup 3 |
| T24 | letter U drill (0.368) | setup 3 |
| T25 | .376 reamer | setup 3 |
| T26 | 1/4 2-flute end mill | setup 3 |
| T99 | plunger dial indicator, at least 0.75 travel | setup-3-measure |

All end mills that plunge must be center cutting.

## Changes from the original

- **No rapid moves across the part.** Every move between features goes
  through the program's clear Z. The original's pistol grip pocket rapided
  to Z-0.379 over the magazine release slot and then across the side of
  the receiver; with the current version of the original's subroutine
  library, that is a crash.
- **Safe tool changes.** Z goes to the top of travel before every change.
  The original changed tools 0.1 above the part and could move sideways
  at the new tool's height.
- **Cutter compensation always on** in setup 1. The original put the
  `G41.1` line behind block delete (`/`), so turning block delete on cut
  0.375 past the finished outline.
- **No placeholder setup 4.** The original's was setup 1's toolpath under
  setup 4's header, and would have cut into the receiver if its `FIXME`
  lines were removed.
- **Every operation can run on its own.** The original failed with an
  undefined parameter when only some of the drilling was switched on.
- **Reaming with G85,** which feeds the reamer back out instead of
  rapiding it out, at gentler feeds. The blind spring hole's reamer stops
  where the drill's full diameter ends, instead of grinding into the
  drill's cone point.
- **Checks before cutting:** tool-table diameters, a spot drill narrow
  enough for the spring hole, and the dial indicator loaded for measuring.
- **Operator prompts use M0,** so they always stop, and show the actual
  speed from the program (the original's reamer prompts said "FIXME rpm").
- **Measuring at Z-0.650** instead of Z-0.500, so a low casting can't run
  the dial indicator out of travel. The magazine release base measurement
  is now used, to limit the depth of each pass in the magazine release
  slot.
- **Runs on LinuxCNC 2.9.** The original's setup 3 used `if [...] then`,
  which LinuxCNC no longer accepts, and depended on an unpinned library
  that changed underneath it.
- **Feeds:** setup 1's feed is raised from 5 to 14.4 in/min. At 0.015
  radial engagement the original's chips were about 0.0002 thick, which
  rubs rather than cuts.

## License

Copyright (C) 2026 narrativecollapse.

This work is free. It comes without any warranty, to the extent permitted
by applicable law. You can redistribute it and/or modify it under the
terms of the Do What The Fuck You Want To Public License, Version 2, as
published by Sam Hocevar. See the `LICENSE` file or <http://www.wtfpl.net/>
for details.

Not covered by that license:

- `AR15-Schematic.pdf` and `ar15_receiver.pdf`, drawings of the receiver
  made by other people and included here for reference.
- The files in this repository's git history from before the rewrite,
  which are Sebastian Kuzminsky's and were published without a license.
