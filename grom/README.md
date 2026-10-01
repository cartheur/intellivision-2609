# Intellivision GROM reconstruction

`grom.asm` is a byte-exact, 8-bit assembly representation of the preserved
GI RO-3-9503 Graphics ROM dump at
`roms/GROM, The (1978) (General Instruments) [!].int`.

The first 1,704 bytes (`$000`–`$6A7`) are 213 8×8 character patterns.  The
remaining 344 bytes (`$6A8`–`$7FF`) hold Executive overflow code/data.  The
STIC addresses GROM; the CPU executes the overflow routines only after the
EXEC copies them into BACKTAB system RAM.

The listing intentionally uses `ROMW 8` and `DECLE` data declarations rather
than guessing semantic boundaries inside the EXEC overflow area.  Regenerate
and verify it from the repository root with:

```sh
python3 grom/reconstruct.py
python3 grom/reconstruct.py --verify
```

## Preserved assembler output

`build/` contains the output of a clean SDK-1600 `as1600` assembly:

- `grom.bin` — 4,096-byte assembler output, with every 8-bit GROM byte
  widened to a big-endian 16-bit container.
- `grom.raw.bin` — the low-byte stream of `grom.bin`; this is the 2,048-byte
  physical GROM payload.
- `grom.cfg` and `grom.lst` — assembler memory-map configuration and listing.

The assembly completed with zero errors and zero warnings.  The low-byte
stream, retained as `grom.raw.bin`, matches the preserved dump byte-for-byte;
both have SHA-256 `a80b6841182547d08635ad30a6af71441c4c9eed9391b3dd22feb30d8e50cc85`.
