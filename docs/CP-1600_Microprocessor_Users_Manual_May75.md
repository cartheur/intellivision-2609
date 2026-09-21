# CP-1600 Microprocessor User’s Manual (May 1975) — agent reference

## Source, purpose, and authority

This reference assimilates [CP-1600_Microprocessor_Users_Manual_May75.pdf](CP-1600_Microprocessor_Users_Manual_May75.pdf), General Instrument document **S16DOC-CP1600-04**, May 1975. The manual describes the original CP1600 processor array: theory of operation, signal timing, electrical parameters, instruction set, programming examples, and typical system configurations.

This is the primary source in this folder for original CP1600 behavior. Its companion [Osborne-16bit-General-Instrument-CP1600.md](Osborne-16bit-General-Instrument-CP1600.md) is useful for a later, concise system overview and for CP1680 IOB coverage. Prefer this manual if those sources disagree on CPU semantics or timing, while noting that it is itself an early revision and labels its electrical specifications “Tentative.”

### Historical-version caution

This manual specifies a 5 MHz non-overlapping two-phase clock and a **400 ns microcycle**, with complete instructions taking 1.6–4.8 µs. The later Osborne chapter describes CP1600, CP1600A, and CP1610 speed grades at 3.3, 4, and 2 MHz respectively. Do not combine the figures without identifying the exact part/revision and data sheet being targeted.

## Architectural model

- The CP1600 is a 40-pin ceramic-DIP, NMOS, 16-bit general-register CPU with a 16-bit address/data transfer path and a single shared memory/peripheral address space of 65,536 words.
- Internally, it processes a 16-bit word as two 8-bit bytes. The ALU and shifter are 8 bits wide; the external bus transfers all 16 bits in parallel.
- The machine has eight 16-bit program-accessible registers (`R0`–`R7`), an ALU, shifter, instruction register, microcontrol unit, and TTL-compatible buffers.
- `R7` is the program counter (PC), pointing to the next instruction; `R6` is the stack pointer (SP), pointing at the next available external-memory stack location. Both remain ordinary register operands where an instruction permits them.
- The stack grows upward: a store through `R6` writes at `[R6]` then increments it; a read through `R6` first decrements it, then reads `[R6]`. Stack depth is limited only by external memory and software discipline.

## Instruction-word and data model

The processor interprets only the low 10 bits of each instruction word; high bits 15–10 are ignored by the microcontrol logic. This permits a compact 10-bit-wide instruction ROM. Registers, arithmetic, logical values, and addresses retain full 16-bit significance.

Most instructions split the low 10 bits into an operation field and two operand/mode fields. The operation field separates instructions into:

- **External-reference instructions**: move data between a register and the unified external address space, or branch relative to the PC. They have a one in the high bit of the operation field.
- **Internal-reference instructions**: operate entirely on the internal register array, including shifts, controls, and register-indirect control transfer. They have a zero in that bit.

All instruction timings are stated in CPU cycles and must be augmented by any wait cycles on each memory access. At a 400 ns cycle, the manual’s example treats a 1–700 ns memory as zero wait; 701–1100 ns as one wait; and 1101–1500 ns as three waits.

## Registers, flags, and control state

### Registers

| Registers | Architectural use |
| --- | --- |
| `R0`–`R5` | General registers; `R1`–`R5` are also address registers in external-reference modes. |
| `R4`, `R5` | Indirect modes can post-increment these after use. |
| `R6` | Stack pointer; special pre-decrement read / post-increment write behavior. |
| `R7` | Program counter; using it as an instruction destination changes control flow after the normal pre-operation increment. |

### Status bits

The ALU status state comprises Carry (`C`), arithmetic Overflow (`OV`), Zero (`Z`), and Sign (`S`), plus an interrupt-enable flip-flop (`INTFF`) referred to by control instructions.

- `C` is set when ADD, SUB, or CMP generates carry, otherwise cleared. SUB/CMP use addition of the two’s-complement subtrahend.
- `OV` is set for signed two’s-complement overflow in ADD, SUB, or CMP, otherwise cleared.
- `Z` is set only for an all-zero shifter output.
- `S` is set when the high-order result bit is one.
- `EIS` sets, and `DIS` clears, the maskable-interrupt enable (`INTFF`). `INTR*` bypasses it; `INTRM*` requires it.
- `GSWD` reads status into a register representation and `RSWD` restores status. Use them when preserving context around reentrant interrupt handling.

## External bus and its exact control states

`D0`–`D15` are a 16-bit bidirectional bus for addresses, instructions, and data. `BDIR`, `BC1`, and `BC2` are outputs which define an eight-state bus protocol. The manual presents the control combination in **BC1, BC2, BDIR** order:

| BC1 BC2 BDIR | Name | Meaning |
| --- | --- | --- |
| `001` | `BAR` | Bus-to-address-register: CPU drives an address and external logic latches it. |
| `011` | `DWS` | Data-write strobe: write enable; CPU drives data. |
| `101` | `DW` | Data write: same direction as `DWS`, one machine cycle earlier; supports extended writes. |
| `111` | `INTAK` | Interrupt acknowledge; lets devices resolve priority. CPU drives the bus. |
| `010` | `IAB` | Interrupt address to bus: external device drives the vector address into CPU; also used at power-up. |
| `100` | `ADAR` | Addressed data to address register: external addressed-data path drives/latches the address during direct addressing; CPU bus is high-Z. |
| `110` | `DTB` | Data to bus: external memory/peripheral drives instruction, address, or data into CPU. |
| `000` | `NACT` | No action: CPU does not use the bus; it is high-Z. |

This table is particularly important for glue logic: the CP1600 does not have separately pinned address and data buses. An external address latch/register and bus transceivers are normal architecture, especially in larger systems.

### Relevant pins and signal behavior

- `φ1`, `φ2`: high-level, high-speed, non-overlapping two-phase clock inputs. They generate internal `TS1`–`TS4` time slots; four slots form a microcycle.
- `MSYNC*`: hold low for at least 10 ms after power application, then release high on a rising `φ1` edge. It synchronizes startup and disables the interrupt system. The subsequent `IAB` input supplies the initial PC.
- `BDRDY`: active-high-ready input; low requests a wait after a `BAR` or `ADAR`. Wait duration must be less than 40 µs, or dynamic CPU state may be lost.
- `STPST`: high-to-low edge-triggered run/stop toggle. The CPU acts after an interruptible instruction and reports stopped state via high `HALT`. The manual says its internal debounce supports a momentary switch directly.
- `BUSRQ*` / `BUSAK*`: active-low DMA bus-request and bus-acknowledge pair. The processor only grants after an interruptible instruction ends.
- `PCIT*`: dual-purpose. An external low input suppresses PC increment during instruction fetch; it must be used carefully with multiword instructions. The CPU outputs a low pulse on this pin for `SIN` software interrupt, allowing external logic to turn that event into `INTR*` or `INTRM*`.
- `EBCA0`–`EBCA3` / `EBCI`: select and sample one of sixteen external Boolean conditions during `BEXT`.
- Supplies in this original specification are nominal `VDD = +12 V`, `VCC = +5 V`, and `VBB = −3 V`.

### BDRDY timing detail

The manual is more exact than a simple “wait under 40 µs” rule:

- CPU samples `BDRDY` in every `TS1` immediately after a `BAR` or `ADAR` state.
- A low request must arrive no later than 50 ns into that `TS1`, must remain low at least 50 ns, and should only assert in `TS1`.
- Its return high may be asynchronous; CPU synchronization occurs during `TS4`.

## Addressing modes and `SDBD`

External-reference data instructions use an internal register as source (output) or destination (input), and one of eight mode encodings:

| Mode | Effective-address / data source |
| ---: | --- |
| 0 | Direct address in the next memory word. |
| 1–3 | Indirect through `R1`, `R2`, or `R3`; no increment. |
| 4–5 | Indirect through `R4` or `R5`; post-increment after use. |
| 6 | Stack via `R6`: `MVO` writes then post-increments; non-`MVO` operations pre-decrement then access. |
| 7 | Immediate data in the next program word; architecturally also the `R7` post-increment interpretation. |

Direct address and immediate forms advance PC by two words; other data-access forms advance it by one. The operation takes place after PC advancement, so naming `R7` as an instruction’s register destination turns the result into a control-flow change.

`SDBD` (Set Double Byte Data) immediately before a supported external-reference operation makes a 16-bit operand from the low eight bits of two program/data words: the first is the low byte, the second the high byte.

- Supported double-byte external modes in this manual are indirect `R1`–`R5` and immediate (`mode 7`); direct (`mode 0`) and stack (`mode 6`) are explicitly unsupported.
- For `R1`–`R3`, both bytes come from the low byte of the same memory word. For post-increment `R4`/`R5`, they come from two consecutive word locations.
- For immediate double-byte operands, the assembler normally inserts `SDBD` automatically when the literal exceeds the configured memory word width; explicitly writing `SDBD` forces two-byte literal generation.

This behavior is a key compatibility constraint for 10-bit program ROM systems and must not be “simplified” into generic byte addressing.

## Instruction-set capabilities

### External-reference operations

The data-access family is `MVI`, `MVO`, `ADD`, `SUB`, `CMP`, `AND`, and `XOR`, each available with the modes above. It accesses memory and peripherals identically because the address space is unified. There are no special port-I/O opcodes.

The eighth external-reference family is PC-relative conditional branch. It tests 32 possible conditions:

- 16 internal conditions derived from status state;
- 16 external conditions selected by `EBCA0`–`EBCA3` and sampled through `EBCI` (`BEXT`).

On a true condition it applies the signed displacement in the second word to `PC + 2`; on false it continues sequentially. The condition field’s bit 4 selects external versus internal testing.

### Internal-reference operations

The register-register family provides `MOVR`, `ADDR`, `SUBR`, `CMPR`, `ANDR`, and `XORR`. The rest of the internal set includes:

- shifts/rotates and byte swap (`SLL`, `SLR`, `SAR`, `SLLC`, `SARC`, `RLC`, `RRC`, `SWAP`);
- single-register operations such as `CLRR`, `TSTR`, `INCR`, `DECR`, `COMR`, `NEGR`, and `ADCR`;
- control operations including `HLT`, `EIS`, `DIS`, `TCI`, `SIN`, `SDBD`, `CLRC`, `SETC`, `GSWD`, and `RSWD`;
- jumps/calls: direct `J`/`JSR`, register jump `JR`, and interrupt-enable/disable jump variants.

`JSR` saves the old PC in `R4`, `R5`, or `R6`; it is not restricted to a hardware call stack. `JR r` is assembler shorthand for moving register `r` to `R7`.

## Interrupt and DMA model

### Interrupts

- Requests are active-low: `INTR*` is always honored and is the highest-priority request input; `INTRM*` is honored only when `INTFF` is set.
- At the end of an interruptible instruction, an accepted interrupt causes CPU to push current PC onto the `R6` stack, output `INTAK` so devices can arbitrate, then receive the service address during `IAB` and load it into PC.
- Hardware priority and vector generation belong to the external system. A simple system can share `INTRM*`, vector to a common handler, disable maskable interrupts, and poll devices with `BEXT`. A daisy-chained `INTAK` network can implement nested priorities.
- `TCI` emits the termination pulse needed to release the currently highest-priority in-service device. Nested interrupts should terminate in reverse order.
- The manual’s reentrant ISR example saves `R0`, gets/saves status with `GSWD`, saves required other registers, and restores in reverse order with `RSWD` before returning to the saved PC.

### DMA

An external controller pulls `BUSRQ*` low. CPU waits for completion of an interruptible instruction, then releases control and asserts `BUSAK*` low. It remains granted until `BUSRQ*` returns high. The later Osborne text adds an important dynamic-logic distinction: DMA `NACT` cycles refresh the CPU whereas ordinary BDRDY wait cycles do not; preserve that distinction in any timing model.

## Programming idioms from the manual

The manual’s examples are valuable because they reveal intended idiomatic use rather than merely legal syntax.

- **Counted loop:** load count into a register, `DECR`, then `BNZE LOOP`.
- **Ascending table:** place base in `R4` then use `MVI@ R4,R0`; `R4` advances automatically. Descending traversal uses a normal pointer and explicit decrement.
- **Dispatch table:** load index, add table base, then `MVI@ R1,R7`; loading a target address into PC is a computed jump.
- **Double precision:** use ordinary add/sub on low words, use `ADCR` to propagate carry, then process high words.
- **Stack save/restore:** assembler aliases `PSHR r` = `MVO@ r,R6`; `PULR r` = `MVI@ R6,r`.
- **Subroutine arguments after call:** choose `R4`, `R5`, or `R6` as the saved return register; the callee reads sequential words through it, naturally advancing the return position, then `JR` returns through that register.
- **Nested subroutines:** save the return register on the `R6` stack before making another call; return with a pull into PC.
- **Polling:** repeatedly execute `BEXT target,condition`; use it for control-panel switches, device-ready signals, and 16-way external status selection.

## System-integration guidance from Chapter 4

### Bus implementation

For small systems, the manual permits direct use of the single CPU bus (typical full-speed load: one TTL load and 200 pF; AC-only load: 500 pF at 80% rated speed). GI ROM/RAM devices may include address/chip-select latches, reducing glue logic. Upper address bits can directly select chips to reduce decoder cost at the price of non-contiguous memory allocation.

For larger/modular systems, use bidirectional/bipolar bus buffers, an external address register, a 3-to-8 decode of `BC1/BC2/BDIR`, and separate address/data distribution. The illustrated system also routes `EBCA` through a 16-to-1 external-condition multiplexer and provides an external bus-request controller.

### Clocks, external sense, stop/start

- Supply non-overlapping `φ1`/`φ2`; MSYNC release must align to `φ1` rising (`TS3`). Chapter 4 supplies example clock-generator circuits.
- Implement mutually exclusive external tests with a 16-to-1 mux driven by `EBCA0`–`EBCA3`, or combine selected tests with external logic before returning `EBCI`.
- A momentary stop/start switch can connect to `STPST` because the CPU contains debounce logic. `HALT` is the true stopped/running indicator, not merely an echo of the switch.

### Basic I/O and communications

The manual shows simple program-controlled I/O from address decode plus input gates or output holding registers driven by `DTB`/`DWS`. Its serial example uses an AY-3-1014 UART with a byte buffer and says a 60 Kbaud interface can be handled with roughly 170 µs available per character because both receiver and transmitter are internally double-buffered. It uses external-condition (`BEXT`) polling for status.

## Electrical/timing facts worth retaining

These are period-specific and should be checked against the exact device’s data sheet before hardware work:

- Recommended supplies: `VDD` 11.4–12.6 V, `VCC` 4.75–5.25 V, `VBB` −2.7 to −3.3 V; operating temperature 0–70 °C.
- `φ1` and `φ2` minimum high pulse widths: 70 ns. Clock period is 0.2–5.0 µs in this tentative table.
- Bus output delay from `φ1`: up to 70 ns; bus input setup before `φ1`: 0 ns; hold after `φ1`: 10 ns.
- `EBCI` decision timing is constrained by `EBCA` output-to-input timing (`EBCA` output delay max 150 ns and EBCA-to-EBCI delay max 400 ns in the manual table).
- Inputs and outputs use the manual’s stated TTL-compatible electrical levels, but clock inputs are high-voltage relative to `VDD`; do not treat `φ1`/`φ2` as ordinary 5-V TTL pins.

## How to use this reference with the other documents

1. Use this file/manual for original CPU pin definitions, bus codes, instruction semantics, timing, and 1975 system examples.
2. Use [Osborne-16bit-General-Instrument-CP1600.md](Osborne-16bit-General-Instrument-CP1600.md) for a compact later overview, CP1600A/CP1610 comparison, and comprehensive CP1680 explanation.
3. For emulation, preserve the unusual register/stack/PC side effects, `SDBD` mode limits, and externally supplied interrupt vectors—these are central to software behavior.
4. For physical hardware, do not infer a universal clock rate or electrical limit from either narrative source; match the actual chip marking and its relevant data sheet.
