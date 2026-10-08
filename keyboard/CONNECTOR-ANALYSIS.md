# Keyboard Component to Master Component Connector Analysis

## Conclusion

The Keyboard Component connects to the Intellivision Master Component through
the Master Component's standard 44-contact cartridge port. It is a broad,
flat-ribbon connection routed within the Keyboard Component's upper tray.

The surviving service and owner documentation establishes the physical routing
and the bus signals at the Master Component port, but **does not provide a
wire-colour, conductor-order, connector-keying, or pin-to-pin harness drawing**.
Do not assume that a replacement is a simple 44-way one-to-one cable without
continuity-testing an original cable or finding a Keyboard Component logic
schematic.

## Physical cable layout

1. The Master Component is seated in the Keyboard Component's upper recessed
   tray.
2. A wide, attached flat ribbon cable extends from the Keyboard Component into
   that recess.
3. The ribbon's signal plug is inserted into the Master Component cartridge
   port.
4. The Master Component's separate power lead is plugged into the Keyboard
   Component mating receptacle.
5. Excess ribbon and power cable are placed in their respective storage
   pockets before the Master Component is lowered into the tray and secured by
   five interconnecting screws.
6. A pass-through cartridge receptacle on the Keyboard Component allows game
   cartridges and compatible peripherals to remain usable.

### Connector gender

The 44-pin **female** edge-card receptacle is in the Master Component. The
Keyboard Component's ribbon terminates in the mating signal plug/edge-card
interface. The Keyboard Component parts record identifies part `2609-9399` as
the 44-pin edge-card connector, but does not state the harness conductor order.

## Master Component cartridge-port reference

The port has 44 contacts on 0.1-inch spacing. With the connector viewed in its
standard orientation, odd-numbered contacts are on the bottom/solder side and
even-numbered contacts are on the top/component side.

| Pair | Odd / bottom contact | Even / top contact |
| --- | --- | --- |
| 1 / 2 | GND | GND on Intellivision I; external video on Intellivision II |
| 3 / 4 | `~MSYNC` | `CBLNK` |
| 5 / 6 | `DB7` | external audio |
| 7 / 8 | `DB8` | external video |
| 9 / 10 | `DB6` | `MCLK` |
| 11 / 12 | `DB9` | `RESET` |
| 13 / 14 | `DB5` | `SR1` |
| 15 / 16 | `DB10` | expansion/speech function |
| 17 / 18 | `DB4` | expansion/speech function |
| 19 / 20 | `DB11` | expansion/speech function |
| 21 / 22 | `DB3` | GND |
| 23 / 24 | `DB12` | GND |
| 25 / 26 | `DB13` | GND |
| 27 / 28 | `DB2` | GND |
| 29 / 30 | `DB14` | `~BUSAK` |
| 31 / 32 | `DB1` | `BC1` return |
| 33 / 34 | `DB0` | `BC2` return |
| 35 / 36 | `DB15` | `BDIR` return |
| 37 / 38 | `BDIR` out | `BDIR` out |
| 39 / 40 | `BC2` out | `BC2` out |
| 41 / 42 | `BC1` out | `BC1` out |
| 43 / 44 | +5 V | GND |

## What the Keyboard Component requires

The connection is not merely a passive cartridge extension. The Keyboard
Component needs the multiplexed 16-bit data/address bus, power and ground,
timing and video signals, and bus-control signals. Independent hardware
documentation indicates that the Keyboard Component uses the outgoing `BC1`,
`BC2`, and `BDIR` lines on pins 41, 39, and 37; it also uses `CBLNK` and the
external-video path for its genlocked text display.

Accordingly, a full 44-contact physical interface is present, but its internal
termination and pass-through behavior must be treated as Keyboard-specific.

## Evidence

### Local primary material

- [Keyboard Component Service Manual](../manuals/Keyboard_Component_Service_Manual.pdf): removal procedure identifies the signal plug at the Master Component cartridge port, the ribbon and power-cord storage pockets, and the five mounting screws.
- [Keyboard Component Owner's Book](../documents/intelliputer/KeyboardComponentOwnersBook.pdf): installation procedure identifies the attached wide flat ribbon and instructs the user to plug it into the Master Component cartridge port.
- [GI cartridge connector drawing 39-149](../documents/intellivision/cartridge-connector-pinout.pdf): identifies the cartridge-bus signal families.
- [Intelliputer drawing/parts record](../documents/intelliputer/CCF10232011_00020.pdf): lists `2609-9399`, `CONNECTOR, 44 PIN EDGE CARD`.

### External technical references

- [Intellivision cartridge-port mapping](https://wiki.intellivision.us/index.php/Cartridge_Port)
- [Connector orientation and pin table](https://consolemods.org/wiki/Intellivision:Connector_Pinouts)
- [Keyboard Component hardware overview](https://wiki.intellivision.us/index.php?title=Keyboard_Component)

## Next step for a replacement cable

Use an intact Keyboard Component to continuity-map every contact from the
Master Component plug to the Keyboard Component logic/pass-through boards.
Record connector orientation before disassembly, then validate the result
against a known-good system. This is necessary to resolve the pin-by-pin
harness layout that the manuals do not publish.
