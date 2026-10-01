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
