# Custom ECS-connected computer system

## Related notes

- [Master Component rebuild readiness](MASTER-COMPONENT-REBUILD-READINESS.md)
  distinguishes the preserved console firmware and documentation from the
  implementation artifacts still needed for a source-buildable #2609 replica.
- [Live-play cyberneticist working proposal](LIVE-PLAY-CYBERNETICS-PROPOSAL.md)
  sketches a safe, traceable HD6309/host companion system for observing and
  participating in original Intellivision gameplay.

This project proposes a new computer system that works alongside an
Intellivision I or II with its Entertainment Computer System (ECS). The objective is
to extend the console into a practical computer environment without treating
the ECS as incidental: the Intellivision remains the video/game host, the ECS
contributes keyboard and expansion context, and the new hardware contributes
memory, I/O, storage or host communication, and system software.

`PLANNING.md` is the deployment plan for supporting either an Intellivision I
or II with ECS. It keeps both as separate validation targets until their actual
interface contracts prove compatible.

`suitable-build.md` is the concise checklist for the first safe serial-monitor
board: prerequisites, components, software, validation sequence, and deferred
features.

## Intended shape

```text
Intellivision II + ECS
        │
  verified expansion interface
        │
custom computer system
  ├── memory and address-decode logic
  ├── console-visible control registers
  ├── serial / storage interface
  ├── optional local processing and RAM
  └── monitor, loader, and applications
```

The first implementation should leave the console CPU as bus master and expose
small memory-mapped devices. A second processor may be worthwhile later, but
only after bus arbitration, reset, and software ownership are understood; it
must not be introduced as an unexamined second bus master.

## What must be established first

The repository has strong console and CP1600 reference material, but it does
not make an assumed ECS pinout safe. Before an active board is attached, resolve
these questions with ECS-specific schematics and measurement of the exact
Intellivision II/ECS pair:

1. Which connector and signals are electrically available?
2. Which address ranges, expansion selects, interrupts, reset signals, and
   power rails can be used without colliding with console, ECS, cartridge, or
   other expansion hardware?
3. What are the clock, bus-cycle, voltage, loading, and power-current limits?
4. Who may drive data, request an interrupt, reset a processor, or own the bus
   in each state?
5. What role will ECS BASIC, EXEC, and the keyboard play in booting and using
   the new environment?

The CP1600-family interface has a multiplexed 16-bit address/data path.
`BC1`, `BC2`, and `BDIR` identify the bus action, so address decoding by itself
never produces a safe interface.

## First subsystem: serial monitor

Serial I/O is the best initial capability. It provides diagnostic output, a
monitor, program loading, and eventually host-assisted storage, while keeping
the first hardware small. It begins as a memory-mapped peripheral under
console-CPU control. There is an option to modernize such a setup by using a Teensy v4.1 microcontroller.

```text
Verified Intellivision II / ECS interface
              │
      address/data bus interface
              │
  address latch, decoder, and bus-control logic
              │
      UART or ACIA register interface
              │
         RS-232 level converter
              │
       terminal or development host
```

### Prototype hardware

- 74LS373 address latch
- Two 74LS245 devices for the 16-bit bidirectional data path
- 74LS138 address decoder
- 74LS00, 74LS08, and 74LS32 control logic as required
- 6850 ACIA, 8251 USART, or another timing-compatible 5 V UART
- MAX232 true RS-232 level converter
- 74LS244 status/control buffer
- A verified regulated 5 V supply arrangement, 0.1uF capacitors at each IC, and bulk capacitance at the board input

The address must be latched before the multiplexed bus becomes data. The data
transceivers must remain disabled unless the peripheral is selected, and read
and write enables must never overlap.

### Initial register contract

| Offset | Register | Purpose |
|---:|---|---|
| `00` | Data | Transmit a character or read one received character. |
| `01` | Status | Report transmitter and receiver state. |
| `02` | Control | Configure and control the UART. |
| `03` | Auxiliary | Reserve for baud-rate or interrupt control. |

The actual addresses are intentionally undecided until an ECS-compatible window
has been verified. Begin with polling; interrupts can be added once simple
read/write timing is proven.

### Serial boundary

A UART's TTL signals must not connect directly to a DB9 connector. A MAX232 or
equivalent performs the voltage conversion. A minimum connection uses transmit,
receive, and signal ground; the precise TX/RX crossing depends on whether the
other end is DTE or DCE. A USB TTL adapter is not a substitute for RS-232.

Start with 9600 baud, 8 data bits, no parity, one stop bit, and no hardware flow
control. If clocking or signal integrity is uncertain, use 1200 baud and two
stop bits for the first transmit-only test.

## Software roles

- The ECS keyboard and console display provide the local interface.
- A console-resident loader/driver initializes the first peripheral.
- A serial monitor supplies diagnostics, program transfer, and host control.
- A command environment, storage protocol, or BASIC integration follows only
  after the hardware register contract is stable.

A minimal monitor test initializes the UART, announces readiness, waits for a
character, and echoes it once the transmitter is ready.

## Staged development

1. **Interface reconnaissance.** Record exact models, ECS revision, connector
   path, rails, address map, and timing. Do not attach active logic until every
   used signal has known direction, level, and owner.
2. **Bus proof.** Respond at one verified address with a fixed read value and
   check latching, decoding, read timing, bus release, and contention absence.
3. **Status register.** Add a read-only changing value to validate data
   direction and register selection.
4. **Transmit.** Send a known byte or startup message; validate UART clock,
   baud rate, level conversion, and terminal wiring.
5. **Receive and monitor.** Echo received data, then add a minimal command and
   loader protocol.
6. **Computer services.** Add expansion RAM, storage, ECS-keyboard-aware
   software, GPIO, or additional serial ports only after the interface is
   reliably characterized.

## Safety constraints

The interface is timing-sensitive. It shares a multiplexed bus, its control
signals determine transaction type, and incorrect enable timing can create bus
contention or damage hardware. Use buffers and transceivers rather than direct
connections to modern logic; a 74LS157 is a selector, not a tri-state bus
buffer. Confirm the available power budget rather than assuming a cartridge-era
budget applies unchanged to the selected ECS connection.

## Outcome

The first board is not the finished computer. It is a carefully bounded proof
of the ECS interface contract: address latch, decode, read/write timing, bus
release, and power behavior on the actual Intellivision II/ECS system. Once
that proof is stable, the same foundation can grow into a keyboard-aware,
memory- and communications-equipped computer environment.
