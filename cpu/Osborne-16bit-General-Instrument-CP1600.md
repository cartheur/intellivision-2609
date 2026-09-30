# General Instrument CP1600 — Osborne chapter reference

## Scope and source

This is an agent-oriented reference to **Chapter 2, “The General Instrument CP1600,”** from the extracted 44-page PDF [Osborne-16bit-General-Instrument-CP1600.pdf](Osborne-16bit-General-Instrument-CP1600.pdf). It is an early-1980s secondary source, accompanied by reproduced manufacturer data sheets for the CP1600, CP1600A, CP1610, and CP1680 I/O Buffer (IOB). Treat electrical limits, timings, and part availability as historical documentation; use the original data sheets/manual when an implementation decision depends on an exact value.

The same `docs` folder also contains [CP-1600_Microprocessor_Users_Manual_May75.pdf](CP-1600_Microprocessor_Users_Manual_May75.pdf), the CP1600 User’s Manual. It is the preferred primary source for chip-specific behavior; use this Osborne chapter for its concise overview, system-level explanations, and CP1680 coverage.

The chapter presents the CP1600 as a 40-pin NMOS 16-bit CPU with a multiplexed 16-bit address/data bus, external two-phase clock generation, separate external support logic, and a CP1680 companion parallel-I/O device. It notes three speed variants:

| Part | Two-phase clock | Machine-cycle time |
| --- | ---: | ---: |
| CP1600 | 3.3 MHz | 600 ns |
| CP1600A | 4 MHz | 500 ns |
| CP1610 | 2 MHz | 1 µs |

## Core model

- The CPU has eight 16-bit programmer-visible registers, `R0` through `R7`. All can participate in register operations, but most have conventional roles:
  - `R0`: unrestricted general register; effectively the primary accumulator in typical code.
  - `R1`–`R3`: general registers and implied-address data counters.
  - `R4`–`R5`: general registers and auto-increment implied-address data counters.
  - `R6`: stack pointer, while still directly programmable.
  - `R7`: program counter, while still directly programmable.
- Direct access to `R6` and `R7` is intentional and PDP-11-like. It supports multiple external-memory stacks, stack-as-data use, and computed jumps by writing an operation result to `R7`.
- Memory and I/O share one address space. The CPU has no dedicated port-I/O instructions in the usual 8-bit-CPU sense.
- The visible status bits are Sign (`S`), Zero (`Z`), Carry (`C`), and Overflow (`O`). `GSWD` reads status into portions of a general register; `RSWD` restores status from a register field.

### Program-word width matters

The architecture is 16-bit, but the handbook emphasizes that CP1600 program memory was often only 10 bits wide. In that arrangement, instruction object codes are 10 bits and direct addresses reach only the first 1,024 words. A 16-bit-wide program store permits 16-bit direct addresses but wastes six bits of many instruction words. Do not assume that “16-bit CPU” means every historical CP1600 program uses a 16-bit ROM word.

## Addressing and stack behavior

### Direct and implied addressing

- **Direct** memory references use an address encoded in subsequent instruction word(s); such instructions are at least two words.
- **Implied** references use `R1`–`R5` as an address register. `R1`–`R3` do not alter automatically; references through `R4` or `R5` post-increment after the access.
- `R6` supplies stack addressing. A write through `R6` stores then increments (push-like); a read through `R6` decrements then reads (pull-like). `PSHR` and `PULR` are aliases for the relevant memory operations through `R6`.
- There are no separate push/pull opcodes beyond those aliases, and no hardware-fixed stack convention beyond the `R6` behavior.

### Double-byte mode: `SDBD`

`SDBD` changes the next eligible memory-reference or immediate operation into byte-oriented behavior. This is crucial when interfacing the 8-bit CP1680.

- With implied addressing through `R1`–`R3`, the same byte address is used twice.
- With `R4` or `R5`, the post-increment produces two successive low-order bytes.
- For immediate operations, it consumes the low-order byte of each of two consecutive program locations.
- The handbook says direct and stack addressing are not allowed in this mode. It also identifies primary/secondary memory or I/O references and immediate/immediate-operate instructions as eligible next instructions.

### Control transfer

- `J` and `JSR` are direct, three-word control transfers. `JSR` saves the return PC in `R4`, `R5`, or `R6`; it does not implicitly use only a stack.
- `JR` jumps to the address held in a register.
- Relative unconditional and conditional branches are available (`B`, `Bcond`), as is `BEXT`, a branch based on an externally supplied condition.
- Jump and jump-to-subroutine encodings can enable or disable interrupts; the mnemonic forms include `JE`, `JD`, `JSRE`, and `JSRD`.

## Instruction-set map

The chapter’s Table 2-2 is the authoritative compact summary, and Table 2-4 gives object codes and cycle counts. At a practical level the families are:

| Family | Examples | Notes |
| --- | --- | --- |
| Memory/register arithmetic | `MVI`, `MVO`, `ADD`, `SUB`, `CMP`, `AND`, `XOR` and `@` implied forms | Direct forms normally take two words; `@` forms use address registers. |
| Immediate | `MVII`, `MVOI`, `ADDI`, `SUBI`, `CMPI`, `ANDI`, `XORI` | `MVOI` writes a register to the immediate field of a following instruction, so it requires writable program memory. |
| Register operate | `MOVR`, `ADDR`, `SUBR`, `CMPR`, `ANDR`, `XORR`, `CLRR`, `TSTR`, `INCR`, `DECR`, `COMR`, `NEGR`, `ADCR` | Because `R7` is general-purpose, these can form computed control transfers. |
| Shifts and rotates | `SLL`, `SLR`, `SAR`, `SLLC`, `SARC`, `RLC`, `RRC`, `SWAP` | Most support one- or two-bit forms; carry/overflow behavior differs by operation. |
| Branch/call | `J`, `JR`, `JSR`, `B`, conditional branch forms, `BEXT` | `BEXT` delegates condition selection to external logic. |
| Interrupt/system | `SIN`, `EIS`, `DIS`, `TCI`, `HLT`, `NOP`, `NOPP`, `CLRC`, `SETC`, `GSWD`, `RSWD`, `SDBD` | `TCI` is especially important with CP1680 daisy-chained interrupts. |

The example benchmark copies a counted buffer from `IOBUF` to a table using `R4` and `R5` auto-increment addressing and `R2` as the count. It is a useful minimal idiom for ordinary CP1600 word transfers.

## Bus, clocks, and machine cycles

`D0`–`D15` are a tri-state, bidirectional multiplexed address/data bus. `BC1`, `BC2`, and `BDIR` identify bus traffic and are decoded externally into named bus cycles. The chapter’s timing diagrams use these names, notably:

- `BAR`: address output / bus-address phase.
- `NACT`: no-action spacing/idle cycle.
- `DTB`: data input.
- `DW` and `DWS`: data output and write strobe.
- `IAB`: interrupt acknowledge / vector input.

Every machine cycle comprises four clock periods. A normal instruction fetch is three machine cycles: `BAR`, `NACT`, `DTB`. The intervening `NACT` cycles are not incidental; external logic must obey their timing. A simple write has address output, spacing, data write, then write strobe phases.

The CPU needs complementary external `φ1` and `φ2` clocks. `MSYNC` is effectively reset/initialization synchronization: after power-up it must be low for at least 10 ms, then rise on a `φ1` rising edge. On start-up an `IAB` cycle loads a 16-bit initial PC from the bus; external circuitry must provide that reset vector (for example, all zeroes for address `0000h`). Interrupts start disabled.

### Wait, halt, PC inhibition, and DMA

- **Wait:** Pull `BDRDY` low before the end of `BAR` when the addressed device needs more time. The CPU samples it in the next `NACT`; it repeats `NACT` until it sees it high. Because the CP1600 is dynamic, an ordinary wait must be under 40 µs or internal state can be lost.
- **Halt:** `HLT` or a high-to-low `STPST` transition enters halt after the appropriate instruction boundary; `HALT` indicates it. A later high-to-low `STPST` exits halt.
- **PCIT:** A low input inhibits PC increment, allowing the fetched instruction to repeat. The handbook says change it only while halted. CPIT is also pulsed by software interrupt execution, but timing separates input and output use.
- **DMA:** Pull `BUSRQ` low. At the conclusion of the next interruptible instruction, the CPU floats the bus and asserts `BUSAK` low while it executes refresh-providing `NACT` cycles. Unlike an ordinary wait, DMA can last arbitrarily long because those cycles refresh the dynamic CPU. Raise `BUSRQ` to resume.

## Interrupts and external conditions

- `INTR` and `INTRM` are active-low interrupt-request inputs; `INTR` has higher priority.
- After the next interruptible instruction, CPU interrupt acknowledge pushes `R7` to the `R6` stack, emits the interrupt-acknowledge bus pattern, and expects external logic to drive a 16-bit service address. That address loads into `R7`.
- External circuitry is responsible for interrupt-priority arbitration and supplying the vector. An ISR must save whatever registers it needs; only the PC is automatically pushed.
- `TCI` terminates the current interrupt and emits the signal required to release/re-arbitrate CP1680 interrupt priorities.
- `SIN` is a software interrupt that pulses `PCIT` low, permitting an external (for example console/front-panel) response.
- `BEXT` exports a four-bit external-condition selector on `EBCA0`–`EBCA3`; external logic returns the Boolean decision on `EBCI`. The four bits identify one of 16 externally defined conditions, not an internally fixed condition code.

## External support-device strategy

The book’s recommended support path is to derive an 8080A-compatible bus from the CP1600 bus, then use 8080-family support ICs for things such as priority interrupts, DMA, serial I/O, programmable parallel I/O, and timers. It explicitly says the MC6800 bus is incompatible and advises against using MC6800 support devices.

The conversion concept demultiplexes and latches the CPU address/data bus, decodes bus-control states, and creates high- and low-byte data paths. That advice reflects the era and should be read as an interoperability pattern, not as a modern board-level prescription.

## CP1680 Input/Output Buffer (IOB)

### What it provides

The CP1680 is a 40-pin, NMOS companion device with +5 V and +12 V supplies. It connects to the low eight CPU bus bits (`D0`–`D7`) and decodes `BC1`, `BC2`, `BDIR` internally. Its capabilities are:

- one 16-bit parallel port (`PD0`–`PD15`), configurable as one 16-bit port or two 8-bit ports;
- elementary input/output handshaking through `PE` (output) and `AR` (input);
- a 16-bit decrementing interval timer;
- error, handshake, and timer interrupt sources; and
- daisy-chain interrupt priority via `IMSKI`/`IMSKO`.

The port pins are **pseudo-bidirectional**, not freely driven bidirectional outputs. A written zero is a comparatively strong low; a written one is weakly sourced. Before reading a pin, write a one to it, then let external logic leave it high for input 1 or pull it low for input 0.

### Registers and addressing

Each CP1680 occupies an externally selected 256-address block. The low three address bits select these eight 8-bit locations:

| Offset | Register |
| ---: | --- |
| 0 | Control |
| 1 | Data, low byte |
| 2 | Data, high byte |
| 3 | Timer, low byte |
| 4 | Timer, high byte |
| 5 | I/O/handshake interrupt vector |
| 6 | Timer interrupt vector |
| 7 | Error interrupt vector |

Access it with byte-mode `MVI`/`MVO`, usually using `SDBD` plus implied addressing. For a single byte, `R1`–`R3` suffice; to access both consecutive bytes efficiently, use `SDBD` with `R4` or `R5`.

### Control register

The control register supplies status and enables:

- bit 0: complement of `PE` / ready status;
- bit 1: sampled `ERROR` input level;
- bit 2: select one 16-bit port (1) versus two 8-bit ports (0);
- bit 3: enable parallel-I/O and error interrupts;
- bit 4: enable timer interrupts;
- bit 5: enable timer decrementing;
- bits 6–7: high- and low-byte parity status (0 even, 1 odd).

It is read/write, so preserve unrelated bits when changing an enable—read, mask with AND/OR, then write back.

### Handshake and timer behavior

- For input, software clears the ready bit, which raises `PE`; external logic supplies data and signals completion by taking `AR` low. This clears `PE`, sets ready, and can request an interrupt.
- For output, writing data raises `PE`; external logic consumes data then takes `AR` low, which clears `PE` and can request an interrupt.
- There is no inherent direction bit: system design must dedicate an IOB to input or output, or otherwise provide its own direction identification.
- The 16-bit timer decrements once per eight `CK1` pulses when enabled. On reaching zero it requests an interrupt if timer interrupts are enabled, then continues from `FFFFh`; it is not a buffered periodic timer.
- Exact long timer periods are multiples of a full `FFFFh × 8 × CK1` rollover: 262.144 ms on CP1600A, 314.572 ms on CP1600, and 524.288 ms on CP1610. Reloading after an ISR adds non-deterministic interrupt-latency error, so use reloads for isolated delays rather than precision periodic timing.

### CP1680 interrupts

An IOB asserts active-low `INTRQ` for `ERROR`, an `AR` high-to-low event, or timer timeout. Priority within one device is **ERROR > handshake/AR > timer**. Multiple IOBs can wire-OR requests and form a daisy chain: the acknowledged device raises `IMSKO`, blocking lower-priority devices through their `IMSKI` inputs. The handbook limits a chain to eight devices due to propagation delay.

Each source has its own 8-bit vector register. On CPU `INTAK`, the highest-priority active source returns its vector, which selects a location in the first 256 words (commonly holding a `JSR`). Every ISR serving a CP1680 chain must execute `TCI`, otherwise the chain’s priority state is not reset for subsequent arbitration.

## Data sheets included in the PDF

The final portion reproduces timing/electrical sheets for CP1600, CP1600A, CP1610, and IOB 1680. It includes maximum ratings, DC/AC characteristics, bus/timing diagrams, and the IOB’s register-address/vector information. For hardware work, consult those pages directly and cross-check against [CP-1600_Microprocessor_Users_Manual_May75.pdf](CP-1600_Microprocessor_Users_Manual_May75.pdf), which is a more direct primary-source companion in this repository.

## Fast orientation for future agents

1. Think **16-bit CPU with a multiplexed bus and externally provided glue logic**, not a self-contained 8-bit microcontroller.
2. `R4/R5` are the natural sequential-memory pointers; `R6` is an upward-growing programmable stack; `R7` is both PC and an ordinary operand register.
3. I/O is memory-mapped. `SDBD` is the key adaptation mechanism for 8-bit peripherals such as CP1680.
4. Never ignore the dynamic-CPU timing distinction: a normal wait is limited to 40 µs, while DMA `NACT` cycles refresh the CPU.
5. Interrupt vectors and priority are largely external. With CP1680 devices, end the ISR using `TCI`.
6. Distinguish the handbook’s architectural explanation from exact electrical requirements in its final data sheets, and prefer original GI documentation when discrepancies arise.
