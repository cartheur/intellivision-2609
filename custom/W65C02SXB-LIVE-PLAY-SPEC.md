# W65C02SXB live-play companion specification

## Status and scope

**Working specification — bench and daughterboard design only.** This approach
uses a WDC W65C02SXB as the complete companion computer and adds a small,
separately built XBus daughterboard for visible decision logic, trace/status
registers, and a fault-neutral authority boundary.

It does not specify a connection to the Intellivision bus or controller. Those
are later interfaces that require primary documentation and bench measurement.
The first active relationship to a 2609 is one-way: external video capture and
recording of normal human controller use.

## Why use the SXB

The W65C02SXB already provides the processor and normal single-board-computer
support that would otherwise consume most of a wire-wrap build:

- W65C02S processor running at 8 MHz;
- 32 KiB SRAM and 128 KiB Flash;
- monitor ROM, reset, oscillator, USB power, and USB debug/programming path;
- two W65C22 VIAs, one W65C21 PIA, and one W65C51N ACIA;
- XBus connection exposing processor address/data/control signals and external
  chip-select lines.

These are properties of the supplied board, not requirements to reproduce on
the daughterboard. The board can be used with its existing monitor and USB
development path before custom hardware is connected.

## System boundary

```text
host computer ── USB ── W65C02SXB
                              │
                            XBus
                              │
              short, local daughterboard only
              ├── Target/Candidate display and comparator
              ├── trace/status registers
              ├── authority/fault-neutral hardware gate
              └── connector reserved for a later controller interface

2609 ── video ──► external capture       human controller ──► 2609
```

No daughterboard output is connected to the console in the first phase.

## Daughterboard functions

### 1. Visible bounded decision cycle

The board implements the minimum inspectable transaction:

```text
fixed Target → revisable Candidate → compare → authorise or reject
             → record verified local state → next state or halt
```

- `Target` and `Candidate` are local registers or manually loaded test words.
- A comparator result is evidence only; it must not directly enable an action.
- Software and a hardware authority condition must both approve an action.
- Accepted, rejected, error, halt, and fault states are visible and readable.
- Each decision receives a monotonically increasing trace identifier.

### 2. Trace/status interface

Use an externally decoded XBus chip-select to expose a small register block:

| Offset | Register | Direction | Purpose |
| ---: | --- | --- | --- |
| `0` | Target | R/W | Current fixed goal or finite-state subgoal. |
| `1` | Candidate | R/W | Proposed bounded action or state value. |
| `2` | Compare/status | R | Comparator result, guards, fault, and authority state. |
| `3` | Command | W | Explicit request to evaluate, record, or halt. |
| `4` | Trace sequence | R | Monotonically increasing decision identifier. |
| `5` | Fault/clear | R/W | Read fault cause; clear only under a defined safe procedure. |

This map is local to the SXB expansion; it is not an Intellivision map.

### 3. Authority and fault-neutral gate

The gate has three externally visible states:

| State | Meaning | Later controller-interface behavior |
| --- | --- | --- |
| Human | Human owns input | Machine interface electrically disabled. |
| Neutral | Neither owns input | Machine interface electrically disabled. |
| Machine | Machine is permitted to request bounded input | Still subject to timing, guard, and fault gating. |

The physical selector defaults to, or can be immediately returned to, Human or
Neutral. The circuit must force Neutral/disabled when any of the following is
true: SXB reset, loss of power, watchdog timeout, host-loss condition,
unrecognised daughterboard state, or asserted fault.

The later controller interface is a separately reviewed circuit. It cannot use
an SXB GPIO pin as a direct substitute for a measured controller closure.

## Recommended inventory-backed daughterboard parts

| Purpose | On-hand part | Notes |
| --- | --- | --- |
| Comparator | `SN74LS85N` | Make Target/Candidate relation visible. |
| Decode | `SN74LS138N` / `SN74LS139AN` | Decode an available SXB external chip-select/register range. |
| Bus isolation | `SN74HCT125N`, `SN74LS244N`, `SN74LS245N` | Tri-state only when the daughterboard is selected. |
| State/trace latches | `SN74LS273N`, `SN74LS373N` | Hold visible values and local status. |
| Fault timing | `SN74LS14N`, `SN74LS123N` | Reset shaping, debounce, bounded timing experiments. |
| Manual controls | `10TC405` switches, pushbuttons | Target/Candidate and authority test inputs. |
| Indicators | on-hand LEDs and resistors | Display target/candidate/result/state/fault. |
| Assembly | wire-wrap sockets, 28 AWG Kynar | Keep the daughterboard local and labelled. |

The SXB has its own memory, serial interface, and processor support; do not add
duplicate RAM, EEPROM, clock, reset, or UART circuitry to this first
daughterboard.

## Build sequence

1. **SXB alone.** Use USB, the supplied monitor, and a stock LED/monitor
   example. Confirm reset and repeated program loading.
2. **Passive XBus breakout.** Bring address, data, selected chip-select,
   read/write, and ground to a short labelled breakout. Observe only.
3. **Read-only status board.** Decode one local register and return a fixed,
   buffered value. Verify no bus drive when unselected.
4. **Comparator demonstration.** Add Target/Candidate registers, `74LS85`,
   LEDs, and serial trace reporting. Demonstrate accepted, rejected, halt, and
   fault paths without any console connection.
5. **Authority/fault gate.** Add a physical selector and inject reset, power,
   and host-loss faults. Confirm that each produces disabled/neutral output.
6. **One-way 2609 observation.** Use external video capture and human-control
   logs; exchange only host/SXB data.
7. **Separate controller-interface review.** Measure the exact controller
   signals and only then design a closure/emulation circuit with its own
   schematic, tests, and failure analysis.

## Electrical constraints

- Power the SXB from its intended USB supply for initial work; do not take
  power from the 2609 or ECS.
- Keep XBus wiring short and local. The SXB processor runs at 8 MHz, so a large
  point-to-point wire-wrap extension is not an appropriate first experiment.
- Treat every daughterboard data-bus connection as tri-stated unless the board
  is positively selected and a read is intended.
- Provide decoupling at every added IC and define unused-input states.
- A software flag, host packet, or comparator output alone must not activate a
  future controller interface.

## Explicit deferrals

- Direct 2609 cartridge, expansion, or ECS bus connection.
- Controller-line driving before a measured controller-interface dossier.
- Console reset, interrupt, bus arbitration, or mailbox control.
- A claim that the W65C02SXB itself performs modern machine learning.

## Acceptance criteria

The SXB approach is ready to leave the bench when it can repeatedly boot from
its monitor, communicate with the host, read/write the local register block,
produce an inspectable decision trace, and enter disabled/neutral state under
each tested fault—with no electrical dependence on the 2609.

## Primary references

- [W65C02SXB product overview](https://wdc65xx.com/single-board-computers/w65c02sxb/)
- [W65C02SXB getting-started guide](https://wdc65xx.com/gettingstarted/02-sxb-getting-started/)
- [W65C02SXB board schematic](https://www.westerndesigncenter.com/wdc/Schematics/W65C02SXB.pdf)
- [W65C02S data sheet](https://www.wdc65xx.com/wdc/documentation/w65c02s.pdf)
