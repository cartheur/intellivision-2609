# Intellivision Executive ROM reconstruction

`executive.asm` is a byte-exact, 10-bit-decle assembly representation of the
preserved Executive ROM dump at
`roms/Executive ROM, The (1978) (Mattel).int`.  It maps 4,096 decles at
`$1000`–`$1FFF` and labels the documented routine entry points from `ROM.md`.

It intentionally represents every ROM word as `DECLE` data.  A conventional
disassembly cannot reliably distinguish instructions from embedded data and
tables without a full control-flow analysis; this form is therefore both
reassemblable and lossless.

Regenerate and validate it with:

```sh
python3 executive/reconstruct.py
python3 executive/reconstruct.py --verify
```

The verifier checks the 8,192-byte input size, 10-bit width of every word, and
the complete generated source including the source SHA-256 fingerprint.

## Preserved assembler output

`build/` contains the output of a clean SDK-1600 `as1600` assembly:

- `executive.bin` — 8,192-byte 10-bit-word container.
- `executive.cfg` — assembler memory-map configuration.
- `executive.lst` — assembly listing.

The assembly completed with zero errors and zero warnings.  `executive.bin`
matches `roms/Executive ROM, The (1978) (Mattel).int` byte-for-byte; both have
SHA-256 `1aeb614856beba95463166daf09304b414d5617d3f37d221724b3337fc4b2722`.
