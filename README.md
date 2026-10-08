# Intellivision 2609

This is a working archive and development workbench for the original
Intellivision Master Component: its CP1600-family architecture, the Intellivision
II and ECS ecosystem, repair material, software, and new hardware experiments.

The immediate project focus is a wire-wrapped HD6309 companion for a 2609 that
supports live-play research. It begins as a standalone, observable computer,
then adds passive video/controller observation and fail-safe controller
emulation for real games; a host may later supply learning and analysis. The
2609 remains the game and display authority. The Intellivision II/ECS path is a
separate corroborative compatibility track for keyboard, expansion, and
bus-connected computer experiments.

## Start here

- [Documentation summary](DOCUMENTATION-SUMMARY.md) is the contextual index to
  the console manuals, processor references, component data, historical
  engineering files, SDK, ROM collection, and current notes.
- [Custom Intellivision companion computer](custom/README.md) is the focused
  design brief: the primary 2609 live-play path, its evidence and safety gates,
  and the separate ECS compatibility track.
- `docs/CP1600-Agent-Index.md` is the concise guide to CP1600 registers,
  instructions, bus signals, timing, and CP1680 I/O behavior.
- `manuals/Intellivision_Service_Manual_Model_2609.pdf` is the primary repair
  reference for the original console.

## Working principles

The first live-play work uses passive video/controller observation and must
leave the console functional when the companion is absent, unpowered, reset,
or faulted. Controller emulation must default to neutral and have explicit
human takeover. The CP1600 uses a multiplexed address/data bus; address
latching, bus-control qualification, data direction, and timely bus release
are essential for any later ECS or console-bus design. Confirm the exact model,
expansion signals, address map, voltage levels, power budget, and ownership of
every shared signal before connecting active bus hardware.

Primary manuals, schematics, data sheets, and observed behavior of the board on
the bench take precedence over a later summary or a historical proposal.

## Archive contents

The repository contains console and expansion service manuals, CP1600 and
speech-chip documentation, internal Mattel engineering records, game memory
maps, SDK-1600 and emulator packages, ROM images, game/source archives, and a
September 2026 autonomous-player lab log in `ProjectNotes.txt`.

## All model numbers

The set of all manufactured Intellivision systems during the period of Mattel, is as follows:

| Hardware | Model number(s) |
|---|---|
| Mattel Intellivision / Master Component | 2609, 2609A, 2609B, 3668 |
| Intellivoice Speech Synthesis Module |	3330 |
| Keyboard Component (unreleased) | 1149 |
| Mattel Intellivision II | 5872, 5878 |
| Entertainment Computer System (ECS) | 4182, 4184, 4187, 4629, 4631, 4690 |
| System Changer | 4610 |
| GTE/Sylvania Intellivision | MC100 |
| Sears Super Video Arcade | 49-75011, 49-75022 |
| Radio Shack Tandyvision | 58-100 |
| Bandai Intellivision | 16287 |
| Digiplay Intellivision | 5368 |
| Digiplay II | 5872 |

## Errata

The only known existing 2609 system was described [here](https://intellivisionrevolution.com/entries/intellivision-auction/mega-rare-intellivision-lot-for-sale-).
