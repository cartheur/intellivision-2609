# CP1600 agent working index

Use this as the entry point for reasoning about CP1600 code, hardware interfaces, and documentation in this folder. It is a cross-reference and action guide—not a replacement for the primary User’s Manual.

## Document map

| Need | Start here |
| --- | --- |
| Exact original CPU semantics, pin definitions, bus encodings, instruction formats, timing, and 1975 examples | [CP-1600_Microprocessor_Users_Manual_May75.md](CP-1600_Microprocessor_Users_Manual_May75.md) and its [source PDF](CP-1600_Microprocessor_Users_Manual_May75.pdf) |
| Concise later overview; CP1600A/CP1610 variants; CP1680 I/O Buffer | [Osborne-16bit-General-Instrument-CP1600.md](Osborne-16bit-General-Instrument-CP1600.md) and its [source PDF](Osborne-16bit-General-Instrument-CP1600.pdf) |
| Physical CP1600 signal layout | [CP1600-pinout.png](CP1600-pinout.png) |
| Intellivision system-specific hardware context | [Intellivision_Service_Manual_Model_2609.pdf](Intellivision_Service_Manual_Model_2609.pdf) |

## First principles to preserve

1. **This is a 16-bit general-register CPU, not an 8-bit accumulator CPU.** It has `R0`–`R7`; `R6` is SP and `R7` is PC, but both are usable as operands.
2. **Memory and peripherals share one 16-bit address space.** `MVI`, `MVO`, and the other external-reference instructions access either; there are no separate port-I/O instructions.
3. **The CPU’s external `D0`–`D15` pins are multiplexed address/data.** `BC1`, `BC2`, and `BDIR` identify the current bus action. Hardware needs latching/decoding or compatible support devices.
4. **Instructions are fundamentally 10 bits wide.** In a 10-bit ROM configuration the high six bits are ignored; direct/immediate operands and `SDBD` must be interpreted in the context of actual program-memory width.
5. **Addressing side effects are program semantics.** `R4/R5` post-increment, stack access through `R6` is asymmetric, and many operations can target `R7` to compute a jump.
6. **Interrupt priority and vectors are externally supplied.** The CPU pushes PC and signals acknowledge, but external logic arbitrates and returns the ISR address.

## Register and addressing cheat sheet

| Item | Meaning / safe mental model |
| --- | --- |
| `R0` | General working register / conventional accumulator. |
| `R1`–`R3` | General registers; indirect address through them does not increment. |
| `R4`, `R5` | General registers and sequential pointers; indirect reference post-increments. |
| `R6` | Upward-growing external-memory stack pointer. Read through it pre-decrements; `MVO` through it writes then post-increments. |
| `R7` | PC. Writing it creates a computed transfer; instruction sequencing normally increments PC before the operation. |
| Direct | Address/literal in a following program word. |
| `@` | Register-indirect addressing (for example, `MVI@ R4,R0`). |
| `I` / immediate | Literal in a following program word (assembly syntax varies by assembler). |
| `SDBD` | Causes the next supported external/immediate operation to consume double-byte data from low bytes. Check mode restrictions. |

### Stack aliases

| Mnemonic | Equivalent | Effect |
| --- | --- | --- |
| `PSHR r` | `MVO@ r,R6` | Store `r` at `[R6]`, then increment SP. |
| `PULR r` | `MVI@ R6,r` | Decrement SP, then load from `[R6]`. |

## Mnemonic index by intent

| Intent | Mnemonics | Notes |
| --- | --- | --- |
| Load/store | `MVI`, `MVO`, `MVII`, `MVOI`, `MOVR` | `MVOI` writes an instruction’s immediate field; it needs writable program memory. |
| Arithmetic | `ADD`, `ADDI`, `ADDR`, `ADCR`, `SUB`, `SUBI`, `SUBR`, `NEGR`, `INCR`, `DECR` | `ADCR` propagates carry for multiword arithmetic. |
| Compare/test | `CMP`, `CMPI`, `CMPR`, `TSTR` | Set status without retaining arithmetic result. |
| Logical | `AND`, `ANDI`, `ANDR`, `XOR`, `XORI`, `XORR`, `COMR`, `CLRR` | AND/XOR/clear/test are the everyday mask operations. |
| Shift/rotate | `SLL`, `SLR`, `SAR`, `SLLC`, `SARC`, `RLC`, `RRC`, `SWAP` | Shift count is one or two; distinguish logical/arithmetic and through-carry behavior. |
| Relative branching | `B`, condition-specific `B...`, `BEXT` | Branch displacement is PC-relative. `BEXT` tests an external condition selected by 4 bits. |
| Direct/register jump | `J`, `JR`, `JSR`, `JE`, `JD`, `JSRE`, `JSRD` | `JSR` saves return PC in `R4`, `R5`, or `R6`; `JR r` is effectively `MOVR r,R7`. |
| Interrupt control | `EIS`, `DIS`, `SIN`, `TCI` | `INTR*` remains non-maskable; `INTRM*` requires `EIS`/`INTFF`. |
| CPU/system control | `HLT`, `NOP`, `NOPP`, `CLRC`, `SETC`, `GSWD`, `RSWD`, `SDBD` | `TCI` is required after CP1680-chain service. |

### Branch-condition guidance

The family includes unconditional `B`, status-based conditional branches (for example `BEQ`, `BNEQ`, `BNZE`, relational signed tests such as `BLT`/`BGE`), and `BEXT`. Confirm an assembler’s exact accepted spelling and condition-code encoding in the User’s Manual instruction table before emitting source. Do not assume branch spellings from another CPU assembler are accepted.

## Goal-to-action recipes

| Goal | Typical CP1600 idiom |
| --- | --- |
| Read a word and advance pointer | Put address in `R4`/`R5`; `MVI@ R4,R0`. |
| Write a word and advance pointer | `MVO@ R0,R5`. |
| Traverse backward | Use `R1`–`R3` and explicitly `DECR`, or carefully arrange pointer arithmetic; auto-increment modes are forward only. |
| Read/write 8-bit peripheral data | Use `SDBD` with a supported implied/immediate mode; see CP1680 section in Osborne reference. |
| Set/clear one control bit | Read the register, apply the appropriate mask/update sequence, then write it back—preserve unrelated control bits. (CP1680 needs this discipline.) |
| Loop N times | Load N, `DECR`, then `BNZE LOOP`. |
| Lookup/dispatch through table | Compute entry address in a register, then load target into `R7` using `MVI@`. |
| Call a routine | `JSR R4|R5|R6,label`; return with `JR` through the chosen register. Save it if making nested calls. |
| Preserve context | `PSHR` registers; use `GSWD`/`RSWD` to preserve status; restore in reverse. |
| Accept device interrupt | External logic drives vector during `IAB`; ISR saves needed state and eventually invokes `TCI` if the priority chain requires it. |
| Poll external ready/status | `BEXT` with an `EBCA`-selected condition; external logic returns result on `EBCI`. |
| Stop or give DMA bus ownership | `HLT`/`STPST` for halt; external `BUSRQ*` and `BUSAK*` for DMA. |

## Bus-control quick map

The following uses the primary manual’s `BC1 BC2 BDIR` bit order:

| Code | Signal | External action |
| --- | --- | --- |
| `001` | `BAR` | Latch CPU-driven address. |
| `011` | `DWS` | Strobe CPU-driven write data. |
| `101` | `DW` | Earlier write-data phase. |
| `111` | `INTAK` | Resolve interrupt priority. |
| `010` | `IAB` | Drive initial/interrupt vector into CPU. |
| `100` | `ADAR` | Latch addressed data as an address for direct access. |
| `110` | `DTB` | Drive instruction/data/address into CPU. |
| `000` | `NACT` | Leave bus unused/high-Z. |

## Timing and hardware guardrails

- **Never use an ordinary `BDRDY` wait beyond 40 µs.** CP1600 state is dynamic and can be lost. DMA handling is distinct because its `NACT` cycles refresh the CPU.
- `BDRDY` must be asserted during the prescribed `TS1` window following a `BAR`/`ADAR`; see the primary manual’s timing section for the 50 ns constraints.
- `MSYNC*` is startup synchronization, not a casual reset pin: hold it low for at least 10 ms after power-up and release on the required `φ1` edge; it also starts instruction execution via an externally supplied `IAB` vector.
- The original manual’s CPU needs three supplies (`+12 V`, `+5 V`, `−3 V`) and non-overlapping clocks. Verify the exact speed-grade data sheet before applying its 5 MHz / 400 ns numbers.
- `PCIT*` has both input (hold PC increment) and output (`SIN` trap) roles. Change its external input only under the documented stopped-state precautions, especially with multiword instructions.

## CP1680 IOB checklist

When a task touches the General Instrument CP1680 companion I/O buffer, read the CP1680 section in the Osborne reference first.

- It uses CPU low bus byte `D0`–`D7`, but its peripheral port is 16 bits (`PD0`–`PD15`).
- It exposes control, low/high data, low/high timer, and three vector registers at offsets 0–7 of a selected 256-address block.
- Use byte-mode accesses and plan `SDBD` carefully.
- Its port is pseudo-bidirectional: write ones before using pins as inputs.
- Its interrupt priority is `ERROR` > `AR` handshake > timer; up to eight devices can form a daisy chain.
- End a CP1680 ISR with `TCI`, otherwise priority arbitration remains latched.

## Questions to resolve before editing code or hardware

1. Which CPU/revision and clock rate is actually on the target board—original CP1600, CP1600A, or CP1610?
2. What is program-memory width: 10-bit, 16-bit, or another arrangement? This governs direct/immediate encoding and `SDBD` data layout.
3. Is the address in question RAM, ROM, memory-mapped I/O, or an external vector source?
4. Which registers may a routine clobber, and does it run within an interrupt/nested-call context?
5. Does use of `R4`, `R5`, `R6`, or `R7` deliberately rely on its side effect?
6. Is the interrupt request `INTR*` (always honored) or `INTRM*` (requires enabled maskable interrupts), and who supplies the vector?
7. Is an apparent bus delay a BDRDY wait (bounded) or a DMA grant (different refresh behavior)?
8. If CP1680 is involved, is the port configured as 16-bit or two 8-bit ports and who owns direction/handshake interpretation?

## Suggested agent workflow

1. Start with this index to identify the relevant subsystem and instruction family.
2. Follow the link to the primary manual reference for semantics; open the source-PDF page/table when an encoding or timing edge case matters.
3. Use the Osborne reference to cross-check later revisions and CP1680-specific behavior.
4. State assumptions about revision, ROM width, register convention, and mapped address before proposing changes.
5. For assembly changes, validate instruction word count, address-mode side effects, status effects, interruptibility, and bus/wait requirements—not only mnemonic spelling.
