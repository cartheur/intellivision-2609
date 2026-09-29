## Focus - Intellivision 2609

The Original Intellivision Model 2609 was released in 1980.

A foray into 16-bit AI gaming.

_When 8-bits isn't wide enough_

Soon we will expand thusly to 18-bits.

### Documentation index

The repository-wide [documentation summary](DOCUMENTATION-SUMMARY.md) provides
contextual, plain-text summaries of the console manuals, CP1600 references,
engineering archives, development resources, ROM collection, and current
hardware work.

### Identification

The 2609A should be Intellivisions manufactured in Hong Kong (or Taiwan) for the US market. Serial numbers are larger than 300k.

The 2609 should be the early Intellivisions made in the USA. Serial numbers under 180k. Perhaps yours is one of the earliest because it lacks the lip on the controllers. Sounds like there are some other subtle differences.

A 2609 made in Hong Kong should be an Intellivision for the Canadian market and have a serial number over 1M prefixed with RH. There are some rare US models made in Hong Kong with serials starting at 4M.

As far as I know they should all have the same Exec ROM inside, and the same sound chip. Over time there were some minor changes to the mainboard e.g. number of ram chips. There were also sone changes with the power supply board as indicated by the maintenance guide. Not sure if that has anything to do with some models saying 15W others 18W.

The serial number [database](https://www.intellivisionrevolution.com/serial-number-database-mattel-intellivision).

### Emulators

Best is [this](http://spatula-city.org/~im14u2c/intv/) site. Requires [SDL](https://github.com/libsdl-org/SDL/releases/tag/release-3.4.16)

### Sources

Intellivision [us](https://intellivision.us/history.php)

### CP1600 documentation

The `docs` folder includes an extracted Osborne CP1600 chapter, the original May 1975 CP-1600 User’s Manual, and agent-oriented Markdown references. Start with [CP1600-Agent-Index.md](docs/CP1600-Agent-Index.md) for instruction mnemonics, register/addressing behavior, bus signals, CP1680 I/O, and links to the detailed sources.

### Custom 44-pin 

```markdown
# Custom Intellivision Cartridge Serial Interface

## Overview

This project describes a custom Intellivision cartridge/peripheral that connects to the Master Component through a 44-pin cartridge connector and provides a DB9 RS-232 serial interface to an external terminal computer.

It is independent of the ECS. The cartridge acts as a memory-mapped serial peripheral.

```text
Intellivision Master Component
              │
       44-pin cartridge connector
              │
      Address/data bus interface
              │
        74LS logic and decoder
              │
          UART or ACIA
              │
        MAX232 level converter
              │
            DB9 port
              │
       External terminal computer
```

## Proposed Hardware

Recommended components:

- 74LS373 or 74LS374 — address latch
- 2 × 74LS245 — 16-bit bidirectional data-bus interface
- 74LS138 or 74LS154 — address decoder
- 74LS00, 74LS08, 74LS32 — control-signal logic
- 6850 ACIA, 8251 USART, or compatible 5 V UART
- MAX232 or equivalent — TTL-to-RS-232 level converter
- 74LS244 — optional status/control buffer
- 5 V voltage regulator, if using an external supply
- 0.1 µF bypass capacitor at every logic IC
- 1–10 µF bulk capacitor near the board power input

## Bus Architecture

The Intellivision cartridge interface exposes a multiplexed 16-bit address/data bus. The same physical lines carry the address during one part of a bus cycle and data during another.

The address must be latched before the bus changes to data.

```text
AD0–AD15
   │
   ├──> 74LS373/374 address latch
   │          │
   │          └──> address decoder
   │
   └──> 2 × 74LS245 <──> UART data bus
```

The cartridge should also use the bus-control signals, including:

- BC1
- BC2
- BDIR
- Clock
- Reset
- Bus acknowledge, where required
- +5 V
- Ground

Address decoding alone is not sufficient. The control signals must be qualified to distinguish address, read, and write phases.

## Logic Arrangement

A general bus interface can be organized as follows:

```text
Master Component bus
        │
        ├── Address latch
        │       │
        │       └── Address decoder
        │
        ├── Bidirectional data transceivers
        │       │
        │       └── UART/ACIA data bus
        │
        └── Bus-control decoder
                │
                ├── UART chip select
                ├── UART read enable
                └── UART write enable
```

The 74LS245 transceivers must be disabled whenever the cartridge is not selected. Only the selected peripheral should drive the shared data bus.

The read and write enables must never be active simultaneously.

## Suggested Register Map

A simple four-register peripheral could use the following layout:

| Offset | Register | Function |
|---:|---|---|
| `00` | Data | Write a character for transmission or read a received character |
| `01` | Status | Indicates transmitter and receiver state |
| `02` | Control | UART configuration and control |
| `03` | Auxiliary | Optional baud-rate or interrupt register |

The actual addresses depend on the selected cartridge address window.

Example software behavior:

```text
if transmitter_ready:
    write character to UART data register

if receiver_has_data:
    read UART data register
```

Polling is recommended for the first prototype because it is simpler than implementing interrupts.

## UART Options

Possible UART devices include:

### 6850 ACIA

Advantages:

- Well suited to simple 8-bit bus systems
- Straightforward data and status registers
- Common in vintage computer designs

### 8251 USART

Advantages:

- Flexible serial configuration
- Supports synchronous and asynchronous modes

Disadvantages:

- More complicated initialization
- Requires careful handling of command and mode registers

### Modern 5 V UART

A modern UART can simplify the design, provided that:

- Its bus interface is compatible with the cartridge logic
- It operates at 5 V logic levels or has suitable buffers
- Its timing is compatible with the system bus
- It exposes registers that can be decoded by the cartridge

## RS-232 Interface

The UART uses TTL-level signals and must not be connected directly to an RS-232 DB9 connector.

Use a MAX232 or equivalent level converter:

```text
UART TX ──> MAX232 ──> DB9 RXD
UART RX <── MAX232 <── DB9 TXD
Ground   ────────────> DB9 signal ground
```

For a conventional DTE terminal or computer:

| Signal | DB9 Pin |
|---|---:|
| Computer receives data, RXD | 2 |
| Computer transmits data, TXD | 3 |
| Signal ground | 5 |

Typical minimum wiring:

```text
MAX232 TXD → DB9 pin 2
MAX232 RXD ← DB9 pin 3
Ground     → DB9 pin 5
```

The exact TX/RX arrangement depends on whether the external device is configured as DTE or DCE. A null-modem cable may be required.

Do not use a USB TTL-serial cable as though it were an RS-232 interface. A TTL adapter and a true RS-232 adapter use different voltage levels.

## Initial Serial Settings

A reasonable starting configuration is:

```text
9600 baud
8 data bits
No parity
1 stop bit
No hardware flow control
```

For a more conservative first test:

```text
1200 baud
8 data bits
No parity
2 stop bits
No hardware flow control
```

Start with transmit-only operation, then add reception after the output path is working.

## Software Requirements

The Intellivision does not automatically detect the serial cartridge. Cartridge software must provide the driver.

The software should:

1. Initialize the UART.
2. Configure the baud rate and frame format.
3. Poll the transmitter status.
4. Write characters to the transmit register.
5. Poll the receiver status.
6. Read incoming characters.
7. Optionally implement a command prompt or terminal protocol.

A minimal test program could:

1. Print a startup message.
2. Wait for a character from the terminal.
3. Echo the character back.
4. Repeat indefinitely.

Example logical behavior:

```text
initialize_uart()

send_string("Serial cartridge ready\r\n")

while true:
    if received_character():
        character = read_character()
        while not transmitter_ready():
            wait()
        write_character(character)
```

## Recommended Development Sequence

### Stage 1: Address and Bus Testing

Build a cartridge that responds to a single decoded address and returns a fixed value.

Verify:

- Address latching
- Address decoding
- Read timing
- Data-bus enable and disable behavior

### Stage 2: Read-Only Register

Add a status register that returns a fixed or changing value.

Verify:

- Data direction
- Bus contention avoidance
- Correct register selection

### Stage 3: UART Transmit

Add a UART and transmit a fixed character or startup message.

Verify:

- UART clock
- Baud rate
- MAX232 output
- DB9 wiring

### Stage 4: UART Receive

Add the receive path and echo characters back to the terminal.

Verify:

- UART receive logic
- Data-bus direction
- Software polling
- Terminal configuration

### Stage 5: RAM and Advanced Features

Optional additions include:

- Cartridge RAM
- Command interpreter
- File-transfer protocol
- Interrupt-driven serial I/O
- Bank-switched memory
- External keyboard support
- GPIO registers
- Additional serial ports

## Power and Protection

The cartridge connector provides 5 V power, but available current may be limited.

Recommendations:

- Use a regulated 5 V supply.
- Add a 0.1 µF bypass capacitor at every IC.
- Add bulk capacitance near the cartridge power input.
- Use socketed logic ICs during development.
- Add series resistors where appropriate on experimental bus connections.
- Use buffers or transceivers rather than directly connecting modern logic to the console bus.
- Consider an external 5 V supply if adding significant RAM or multiple peripherals.
- Ensure all grounds are connected correctly.

## Important Design Constraints

The interface is timing-sensitive because:

- The address/data bus is multiplexed.
- The bus-control signals define the transaction type.
- Multiple devices share the data bus.
- The cartridge must release the bus when it is not selected.
- Incorrect enable timing can cause bus contention.
- Directly connecting incompatible logic levels may damage the console or peripheral.

The 74LS157 is useful for selecting between unidirectional signals, but it is not a substitute for a tri-state bus transceiver. Use 74LS245, 74LS244, or equivalent devices where bus isolation is required.

## Summary

The proposed device is a custom Intellivision peripheral cartridge:

```text
44-pin cartridge connector
        │
74LS373/374 address latch
        │
74LS138/154 address decoder
        │
2 × 74LS245 data transceivers
        │
6850, 8251, or compatible UART
        │
MAX232 RS-232 converter
        │
DB9 serial connector
        │
External terminal computer
```

The most practical first version is a small, polling-based, transmit-and-receive UART cartridge using:

- 74LS373 for address latching
- 74LS245 transceivers for the data bus
- 74LS138 for address decoding
- 6850 ACIA or compatible UART
- MAX232 for RS-232 voltage conversion
- DB9 connector with TX, RX, and ground only
```
