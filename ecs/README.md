# ECS EXEC-BASIC ROM reconstruction

`ecs.asm` is a byte-exact 10-bit-decle assembly representation of the
preserved 24 KiB ECS EXEC-BASIC image at
`roms/Entertainment Computer System EXEC-BASIC (1978) (Mattel) [!].int`.

The physical image has three consecutive 4,096-decle pages.  They are mapped
to the ECS address windows `$2000`–`$2FFF`, `$7000`–`$7FFF`, and
`$E000`–`$EFFF`, respectively; ECS bank selection controls their visibility.
The listing declares each page at that documented CP1610 address while
preserving the image's page order.

Regenerate and verify it from the repository root with:

```sh
python3 ecs/reconstruct.py
python3 ecs/reconstruct.py --verify
```

## Preserved assembler output

`build/` contains the output of a clean SDK-1600 `as1600` assembly:

- `ecs.bin` — 24,576-byte 10-bit-word container.
- `ecs.cfg` — assembler memory-map configuration.
- `ecs.lst` — assembly listing.

The assembly completed with zero errors and zero warnings.  `ecs.bin` matches
`roms/Entertainment Computer System EXEC-BASIC (1978) (Mattel) [!].int`
byte-for-byte; both have SHA-256
`2fcea60039c4a8836b9bb71e391d70a145c724b4dae516be733fe842ba6687c7`.
