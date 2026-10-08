# Documentation summary

This repository is a working Intellivision 2609 archive. It brings together
console service material, CP1600/CP1610 processor references, speech and video
chip documents, internal Mattel engineering records, development tools, ROMs,
and current hardware experiments. The material describes both the production
console and the larger family of systems and ideas built around it.

## Orientation

For a repair or board-identification question, begin with the console manuals
and compare the exact board in hand with the applicable model revision. For
software or cartridge hardware, begin with the CP1600 references and system
memory maps. For expansions, consult the corresponding speech, Keyboard
Component, or peripheral documents. Historical project papers are valuable
primary evidence, but do not supersede a production specification or a
measured behavior of the target hardware.

## Console operation, repair, and visual references

- `manuals/Intellivision_Service_Manual_Model_2609.pdf` is the principal
  service reference for the original 2609 Master Component: circuitry,
  troubleshooting, adjustment, and repair procedure.
- `manuals/Intellivision_Subassembly_Service_Manual_1978_Mattel_US.pdf`
  focuses on the original console's component assemblies.
- `manuals/Intellivision_II_Service_Manual_1982_Mattel_US.pdf` documents the
  later redesigned console, so its assumptions should not be applied blindly to
  a 2609.
- `manuals/Master_Component_Owners_Manual_1978_Mattel_AU.pdf` and
  `manuals/Sears_Telegames_Owners_Manual_1978_Sears_US.pdf` cover consumer
  setup, controls, and everyday operation of the Mattel and Sears systems.
- `manuals/Intellivoice_Service_Manual_1979_Mattel_US.pdf` covers the speech
  expansion; `manuals/Keyboard_Component_Service_Manual.pdf` and
  `manuals/Computer_Module_Guide.pdf` cover the Keyboard Component / Computer
  Module; `manuals/AquariusPrinter.pdf` preserves related printer material.
- `manuals/Intellivision-Schematic.png`, `manuals/motherboard.jpg`, and the
  images in `manuals/printer/` are quick visual aids rather than substitutes for
  the service documentation.

## CPU, bus, and core system material

- `docs/CP1600-Agent-Index.md` is the fast working index for CP1600 code and
  interfaces. It covers registers, addressing side effects, instruction
  families, bus states, timing, interrupts, and CP1680 I/O.
- `docs/CP-1600_Microprocessor_Users_Manual_May75.pdf` is the primary 1975
  General Instrument CPU manual. Its Markdown companion makes the same
  architecture, pin, timing, and instruction material searchable.
- `docs/Osborne_16-Bit_Microprocessor_Handbook_1981.pdf` is the complete
  handbook source. Its extracted CP1600 chapter and Markdown transcription add
  concise coverage of the CP1600A, CP1610, and CP1680 I/O Buffer.
- `docs/CP1600-pinout.png` shows the physical signal layout, and
  `docs/bitsavers_osborneOsbssorHandbook1981_31682257_archive.torrent`
  identifies the archival source for the complete handbook.
- `documents/intellivision/cp1600-instruction-set.pdf` is a period
  instruction-set memo. `documents/intellivision/cartridge-connector-pinout.pdf`
  identifies the cartridge signals that a peripheral must obey.
- `documents/intellivision/memory-map.pdf` is a compact address-space diagram;
  `documents/intellivision/memory-map-81-82.pdf` is the detailed 1981–82 map.
- `documents/intellivision/parts-list.pdf`,
  `documents/intellivision/product-specification.pdf`,
  `documents/intellivision/video-game-spec-general-instruments.pdf`, and
  `documents/intellivision/engineering-change-notice.pdf` respectively capture
  the system component list, 2609 product definition, GI system specification,
  and a 1979 revision record.

## Speech, video, and custom-chip references

- `docs/SP0250_Applications_Manual.pdf`, `docs/SP0256B_Datasheet.pdf`, and
  `docs/US4296279.pdf` provide application, component, and patent views of the
  speech-synthesis family used to understand Intellivoice hardware.
- `docs/SPR-16_Datasheet.pdf`, `docs/SPR-32_Datasheet.pdf`, and
  `docs/SPR-128_Datasheet.pdf` are companion component data sheets.
- `documents/intellivoice/CCF10232011_00021.pdf` is an Intellivoice engineering
  packet containing correspondence and referenced product/schematic material.
- `documents/videographic-data/CCF10212011_00010.pdf` explains a dynamic-RAM
  Videographic System and `CCF10212011_00009.pdf` is an accompanying diagram.
  `CCF10242011_00007.pdf` is a larger chemical-bank-terminal design record;
  `CCF10242011_00008.pdf` preserves later Communications Connection planning.
- `documents/intellivision-magic/CCF10272011_00005.pdf` is a confidential draft
  advanced specification for the MAGIC graphic-interface circuit: design
  history, not a service guide for a retail console.

## Keyboard Component, Intelliputer, and successor systems

- `documents/intelliputer/KeyboardComponentOwnersBook.pdf` and
  `documents/intelliputer/KeyboardComponentServiceManual.pdf` are the core
  owner and service references. Duplicate-access copies appear in `manuals/`.
- `documents/intelliputer/CCF10212011_00001.pdf` is a revised Keyboard
  Component owner-book production record; `CCF09012012_00000.pdf` is related
  correspondence and enclosure material.
- `CCF10232011_00018.pdf` is an Intelliputer status and planning report.
  `CCF10232011_00020.pdf`, `CCF10232011_00027.pdf`, `CCF10242011_00006.pdf`,
  `CCF10242011_00012.pdf`, and `CCF10242011_00013.pdf` are supporting project
  records and diagrams. `CCF10242011_00004.pdf` describes an ID-ROM scheme to
  protect Decade diskettes. `CCF10282011_00006.pdf` and `CCF10282011_00007.pdf`
  preserve Keyboard advertising and production art.
- `documents/intellivision-ii/master-component-parts-list.pdf` is the
  confidential Intellivision II parts list. `documents/intellivision-iii/CCF10232011_00017.pdf`
  is the preliminary target specification for the planned Intellivision III.

## Development resources and preserved software

- `sdk/sdk1600_3.zip` contains the SDK-1600 assembler, disassembler, ROM
  conversion, cartridge-test, and support utilities. `sdk/README.md` gives its
  historical context.
- `src/jzintv-20200712-linux-x86-64-sdl2.zip` and
  `src/jzintv-20200712-win32-sdl2.zip` are Linux and Windows emulator packages
  for running and examining software in the collection.
- `roms/` contains 178 images spanning the commercial library, prototypes,
  alternate revisions, demonstrations, BIOS images, and homebrew.
  `roms/readme.md` explains its title-level inventory, aliases, and verification
  limits.
- `data/Intellivision-game-memory-maps.csv` associates game filenames and
  titles with memory-map identifiers for comparative cartridge work.
- `zips/` contains game and source archives, including Soccer 2, Voochko,
  Blowout, Spacehawk, League of Light, ECSCAL, Space Shuttle, Astrosmash
  Competition, and Robot Rubble. `zips/todd.zip` is Tag-Along Todd;
  `zips/toddsource.zip` contains its assembly source and support routines; and
  `zips/todd.txt` records the Beta 3.13 premise, feature changes, and backlog.

## Current work

- `ProjectNotes.txt` is the September 2026 lab log for an autonomous player.
  It records a working LED-indicated joystick-control circuit and the intended
  extension to Intellivision keypad support.
- `custom/README.md` is the design brief for a proposed Intellivision companion
  computer. Its primary track is a 2609 live-play system: standalone HD6309
  bring-up, passive video/controller observation, and fail-safe controller
  emulation. It keeps Intellivision I/II + ECS serial-monitor work as a
  separate corroborative compatibility track. Both tracks require primary
  sources and target-hardware measurement before active electrical connections
  are made.

## Source hierarchy

Use original manuals, data sheets, schematics, and service documents for exact
electrical or instruction-level decisions. Use the Markdown indexes for quick
orientation and the project notes for current intent. Keep the distinction
clear between an archival proposal, a production document, a later summary,
and behavior verified on the board being studied.
