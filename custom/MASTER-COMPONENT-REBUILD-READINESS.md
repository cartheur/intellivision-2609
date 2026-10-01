# Master Component rebuild readiness

## Conclusion

This repository is a strong Intellivision Master Component (#2609)
preservation and reference archive.  It is **not yet a source-buildable Master
Component implementation**: it lacks the chip models/cores, system
integration, automated hardware verification, and physical-production design
files needed to reproduce the console.

The hardware baseline is the [Blue Sky Rangers Master Component technical
overview](https://history.blueskyrangers.com/hardware/2609.html#scratchpad).
That reference identifies the programmer-relevant components as the CP1610,
AY-3-8900-1 STIC, RA-3-9600 system RAM, AY-3-8914 sound chip, GROM/GRAM,
Executive ROM, scratchpad RAM, and hand controllers.

## Inventory

| Component | Original part/function | Repository evidence | Rebuild status |
| --- | --- | --- | --- |
| CPU | GI CP1610, 16-bit CPU | CP1600/CP1610 manuals, instruction reference, pinout | Documented; no CPU core or replacement implementation |
| Video | GI AY-3-8900-1 STIC | Service manuals and schematic image | Documented; no behavioral/HDL model |
| System RAM | GI RA-3-9600; BACKTAB, CPU RAM, and arbitration | Service documentation | No model or replacement design |
| Sound | GI AY-3-8914 PSG | Service documentation | No PSG core/model |
| EXEC | GI RO-3-9502 + RO-3-9504, two 2K × 10 ROMs | Preserved dump and byte-exact 4,096-decle assembly reconstruction | Firmware preserved and reproducible |
| GROM | GI RO-3-9503, 2K character ROM | Preserved dump and byte-exact 8-bit assembly reconstruction | Firmware preserved and reproducible; not semantically disassembled |
| GRAM | Two GTE 3539 256-byte SRAMs | Service documentation | No model or replacement design |
| Scratchpad | GTE 3539 256 × 8 SRAM | Service documentation | No model or replacement design |
| I/O and board support | Controllers, clocks, power, RF/video, glue logic | Service documentation and schematic image | No editable schematic, BOM, PCB, or production files |

The repository also contains a historical SDK-1600 archive, including
assembler/disassembler source and utilities, plus a large cartridge-ROM
collection useful as compatibility-test inputs.

## Preserved firmware

The Executive ROM dump is retained at:

`roms/Executive ROM, The (1978) (Mattel).int`

Its reassemblable, word-for-word equivalent is:

`executive/executive.asm`

Run the following from the repository root to regenerate and validate it:

```sh
python3 executive/reconstruct.py
python3 executive/reconstruct.py --verify
```

The Executive occupies 4,096 10-bit decles mapped at `$1000`–`$1FFF`.  Its
preserved input image is 8,192 bytes because each 10-bit decle is stored in a
16-bit big-endian container.

The GROM dump is retained at:

`roms/GROM, The (1978) (General Instruments) [!].int`

Its reassemblable, byte-for-byte equivalent is `grom/grom.asm`.  Regenerate
and validate it with `python3 grom/reconstruct.py` and
`python3 grom/reconstruct.py --verify`.

The accompanying Intellivoice and ECS firmware images also have byte-exact
source-form reconstructions:

| MiSTer boot image | Binary input | Source-form representation |
| --- | --- | --- |
| `boot0.rom` | `roms/Executive ROM, The (1978) (Mattel).int` | `executive/executive.asm` |
| `boot1.rom` | `roms/GROM, The (1978) (General Instruments) [!].int` | `grom/grom.asm` |
| `boot2.rom` | `roms/IntelliVoice BIOS (1981) (Mattel).int` | `intellivoice/intellivoice.asm` |
| `boot3.rom` | `roms/Entertainment Computer System EXEC-BASIC (1978) (Mattel) [!].int` | `ecs/ecs.asm` |

## Important compatibility constraint: EXEC code in GROM

The Executive cannot be treated as only the 4K EXEC-ROM image.  The hardware
reference records that additional EXEC code resides in GROM.  When required,
the system copies that code into BACKTAB locations in system RAM and executes
it while the STIC blanks the display.  A compatible rebuild therefore needs
to implement all of the following together:

1. GROM contents and addressing.
2. STIC/System-RAM bus arbitration.
3. BACKTAB RAM as executable CP1610 memory.
4. The display-blanking behaviour during this transfer/execution path.

Simply booting the EXEC-ROM dump is insufficient to reproduce reset and GRAM
loading behavior faithfully.

## Work required for a source-buildable system

- Implement or integrate cycle-appropriate models for CP1610, STIC,
  AY-3-8914, RA-3-9600 behaviour, GRAM, scratchpad RAM, controllers, and
  board glue logic.
- Define the complete address map and 10-bit ROM/decle memory interface.
- Turn the GROM dump into a verified asset with mapping metadata; verify the
  EXEC's GROM-resident routines and copy path.
- Build a top-level FPGA/ASIC target and a repeatable host build.
- Add automated boot, video, sound, controller, cartridge, and timing tests,
  using preserved cartridge ROMs as compatibility inputs where appropriate.
- For a physical replica, create editable schematics, a BOM, PCB layout,
  programmable-device configuration, and ROM-programming outputs.

## Practical status

The project is ready for the **firmware-preservation and architecture-model
phase**, not the final hardware-build phase.  The best next milestone is a
testable system model that boots the preserved EXEC and exercises the
GROM-to-BACKTAB execution path before committing to FPGA or PCB work.
