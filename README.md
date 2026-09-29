# Intellivision 2609

This is a working archive and development workbench for the original
Intellivision Master Component: its CP1600-family architecture, the Intellivision
II and ECS ecosystem, repair material, software, and new hardware experiments.

The immediate project focus is a new computer system that works with an
Intellivision II and its ECS. It begins with a verified, memory-mapped serial
monitor and can grow into keyboard-aware software, expandable memory, storage,
and host communication. The design deliberately treats a safe electrical
interface as the first milestone, not an assumption.

## Start here

- [Documentation summary](DOCUMENTATION-SUMMARY.md) is the contextual index to
  the console manuals, processor references, component data, historical
  engineering files, SDK, ROM collection, and current notes.
- [Custom ECS-connected computer system](custom/README.md) is the focused
  design brief: goals, unresolved interface evidence, serial prototype,
  software roles, staged development, and safety constraints.
- `docs/CP1600-Agent-Index.md` is the concise guide to CP1600 registers,
  instructions, bus signals, timing, and CP1680 I/O behavior.
- `manuals/Intellivision_Service_Manual_Model_2609.pdf` is the primary repair
  reference for the original console.

## Working principles

The CP1600 uses a multiplexed address/data bus; address latching, bus-control
qualification, data direction, and timely bus release are essential. For an
ECS-connected design, confirm the exact model, expansion signals, address map,
voltage levels, power budget, and ownership of every shared signal before
connecting active hardware.

Primary manuals, schematics, data sheets, and observed behavior of the board on
the bench take precedence over a later summary or a historical proposal.

## Archive contents

The repository contains console and expansion service manuals, CP1600 and
speech-chip documentation, internal Mattel engineering records, game memory
maps, SDK-1600 and emulator packages, ROM images, game/source archives, and a
September 2026 autonomous-player lab log in `ProjectNotes.txt`.
