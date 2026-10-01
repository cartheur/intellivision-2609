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
