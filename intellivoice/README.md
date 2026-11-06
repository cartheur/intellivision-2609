# Intellivoice ROM reconstruction

`intellivoice.asm` is a byte-exact, 8-bit assembly representation of the
preserved SP0256-012 speech ROM at
`roms/IntelliVoice BIOS (1981) (Mattel).int`.  It is a 2,048-byte mask-ROM
image addressed locally by the Intellivoice SP0256, rather than CP1610 code.

Regenerate and verify it from the repository root with:

```sh
python3 intellivoice/reconstruct.py
python3 intellivoice/reconstruct.py --verify
```

## Preserved assembler output

`build/` contains the output of a clean SDK-1600 `as1600` assembly:

- `intellivoice.bin` — 4,096-byte assembler output, with each 8-bit speech
  ROM byte widened to a big-endian 16-bit container.
- `intellivoice.raw.bin` — the low-byte stream of `intellivoice.bin`; this is
  the 2,048-byte physical SP0256-012 ROM payload.
- `intellivoice.cfg` and `intellivoice.lst` — assembler memory-map
  configuration and listing.

The assembly completed with zero errors and zero warnings.  The low-byte
stream, retained as `intellivoice.raw.bin`, matches the preserved dump
byte-for-byte; both have SHA-256
`dabade432af1523f43029057071ca8c1ed3604498c784603e8733023382ad7a5`.

## Board reverse engineering

`board/` holds the photo-based board-reverse-engineering work.  The received
photos document two otherwise comparable assemblies: one with an unpopulated
30-position (2x15) internal-connector footprint and one with its 2x15 female
socket installed.  The populated board's underside shows all 30 solder joints,
and a mating male header is pictured with it.

This confirms an assembly variant omitted from the earlier Internet-based
disassembly.  The original circuit schematic remains preserved in
`manuals/Intellivoice_Service_Manual_1979_Mattel_US.pdf` (Figure 5, printed
pages 10–11).  The photo-derived connector pinout is intentionally unassigned
until it is corroborated by continuity measurements; see `board/README.md` and
`board/SCHEMATIC.md`.
