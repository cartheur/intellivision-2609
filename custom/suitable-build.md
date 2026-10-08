# Suitable first ECS-track build

For the corroborative ECS track, the suitable first build is a conservative
ECS-connected serial-monitor board. It is not the first build required for the
primary 2609 live-play companion, which begins with standalone HD6309 bring-up
and passive controller/video observation.
It may support an Intellivision I + ECS or an Intellivision II + ECS, but only
after each target has independently passed interface validation.

## Preconditions

Before building or connecting active custom hardware, have:

- A known-working Intellivision I or II, ECS, power supply, keyboard, and
  display setup.
- Recorded console and ECS model numbers, board revisions, and known working
  state.
- A verified ECS connection path, pinout, address window, signal directions,
  logic levels, timing, power budget, reset behavior, and bus-ownership model.
- Confirmation that the selected address and signals do not collide with the
  cartridge, ECS, EXEC, or another mapped device.

## Required first-board hardware

- 74LS373 or 74LS374 address latch for the multiplexed CPU bus.
- Two 74LS245 tri-state transceivers for the 16-bit data bus.
- 74LS138 or 74LS154 address decoder.
- Necessary 74LS00, 74LS08, and 74LS32 glue logic for qualified read, write,
  and chip-select signals.
- A 5 V, timing-compatible UART; a 6850 ACIA is the preferred simple
  prototype choice.
- MAX232-compatible RS-232 level converter and DB9 serial connection.
- Optional 74LS244 status/control buffer.
- 0.1 µF bypass capacitor at every IC and bulk capacitance at the board input.
- A confirmed, adequately rated power arrangement. Do not assume the console
  or ECS connection can power added RAM or multiple peripherals.

## Required initial software

- A small CP1610 driver that initializes the UART.
- A four-register mapped interface:

| Offset | Register | Purpose |
|---:|---|---|
| `00` | Data | Write a transmit character or read a received character. |
| `01` | Status | Report transmitter and receiver state. |
| `02` | Control | Configure and control the UART. |
| `03` | Auxiliary | Reserve for future baud-rate or interrupt control. |

- Polling-based transmit first, then receive, then echo/monitor operation.
- A minimal loader and diagnostic monitor before BASIC integration, storage,
  or a more elaborate user interface.

## Required validation sequence

1. Passively inspect the system and measure signals without driving them.
2. Attach a read-only proof board at one verified address.
3. Verify address latching, decoding, read timing, tri-state bus release, and
   the absence of contention.
4. Validate UART transmit at a conservative serial configuration.
5. Validate receive and echo behavior.
6. Repeat the proof on the second console target before calling the design
   shared-compatible.

## Explicitly deferred features

Expansion RAM, storage, ECS-keyboard-aware command software, GPIO, additional
serial ports, interrupts, and a second processor are not first-build
requirements. Add them only after the serial monitor proves the electrical and
software interface. A second processor requires independently proven bus
arbitration and reset behavior.
