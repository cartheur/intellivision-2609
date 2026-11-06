# Schematic reconstruction status

## Canonical circuit

The original circuit schematic is already preserved in the Intellivoice
service manual: `manuals/Intellivoice_Service_Manual_1979_Mattel_US.pdf`,
Figure 5 (printed pages 10–11).  It identifies the principal functional parts:

```text
J1 / J3 cartridge-path connectors
        |
        +-- U2 SPB-640 speech controller -- U3 SP-0256-012 speech ROM
        |                                      |
        +-- U1 74LS12 interface logic           +-- U4 LM324C / U5A LM358C
                                               analogue speech output
```

The received-board photographs agree with the visible major devices: SPB-640,
SP-0256-012, 74LS12, LM324N, and LM358C.  Component suffixes and values should
be taken from the service-manual diagram/parts list, then verified against the
particular board before producing a bill of materials.

## Photo-specific addition: internal 2x15 connector

Figure 5 does not document the central 2x15 footprint seen in the received
boards.  The board photos therefore add the following physical item to the
reconstruction:

```text
J?  internal connector
    form: 2 rows x 15 positions (30 contacts)
    populated state: optional by observed board variant
    known mating hardware: loose 2x15 male header shown with the populated board
    electrical pinout: unverified
```

`J?` is deliberately a temporary designator.  It must not be conflated with
the manual's J1, J2, or J3 until pin-by-pin continuity data establishes its
electrical role.

## Validation legend for future netlist work

| Mark | Meaning |
| --- | --- |
| `PHOTO` | Visible in a board photograph. |
| `MANUAL` | Shown in the preserved service-manual schematic. |
| `METERED` | Confirmed by direct continuity measurement. |
| `INFERRED` | Plausible from routing, but not yet confirmed. |

No electrical pin assignment for the internal connector has yet been made.

## KiCad scaffold

`intellivoice-board.sch` is an editable KiCad legacy-format starting sheet.
It places the documented ports and major devices, and adds the photo-confirmed
J4 2x15 connector as an explicitly unconnected optional part.  It is a
component/function scaffold, not a claim that the displayed symbols' pins are
wired correctly.  The KiCad installation is not available in this workspace,
so the file has not been opened or migrated here; open it in KiCad and save it
as a project-local `.kicad_sch` before continuing.
