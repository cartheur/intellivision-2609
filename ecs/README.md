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
