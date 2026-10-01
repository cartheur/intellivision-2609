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
