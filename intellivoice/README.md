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
