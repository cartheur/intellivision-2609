# Intellivoice board reverse engineering

This directory is the workspace for an auditable, board-specific Intellivoice
schematic and connector map.  It complements the published circuit diagram in
the service manual; it does not replace that primary reference.

## Status

The received boards establish that the central 30-position footprint is a
two-row, 15-position-per-row internal connector:

| Photo set | Connector state | Evidence |
| --- | --- | --- |
| `../images/top-00.jpg`, `../images/bottom-00.jpg` | Not populated | Thirty plated through-holes are present, with no socket or solder joints. |
| `../images/top-01.jpg`, `../images/bottom-01.jpg` | Populated | A black 2x15 female socket is fitted; thirty solder joints are visible on the underside. |

`top-01.jpg` also includes a loose mating 2x15 male header.  The PCB silkscreen
beside the socket includes a `30` marking.  This is a confirmed assembly
variant, not a connector inferred from the Internet disassembly.

## Sources and confidence

- `../images/` is the primary evidence for assembly, footprint, connector
  orientation, and visible routing.
- `../../manuals/Intellivoice_Service_Manual_1979_Mattel_US.pdf`, Figure 4,
  provides the PCB component layout.
- The same manual, Figure 5 (printed pages 10–11), provides the original
  Intellivoice circuit schematic and connector names J1, J2, and J3.

The service-manual schematic is authoritative for the original circuitry.  A
photo-only trace is provisional until each ambiguous net is checked with a
continuity meter.  In particular, do not assign functions to the 2x15 header
solely from its physical position.

## Deliverables

- `SCHEMATIC.md` — source map and schematic reconstruction status.
- `intellivoice-board.sch` — editable KiCad legacy-schematic scaffold.  Open
  it with KiCad's Schematic Editor and save/import it as a modern
  `.kicad_sch` project file before making substantive edits.
- A future machine-readable schematic/netlist, after photo tracing and
  continuity validation.

## Next measurements

1. Photograph each board square-on, with a ruler and clear views of both
   connector ends.
2. Identify pin 1 on the populated 2x15 socket from silk, pad geometry, and
   continuity rather than assuming orientation from the photograph.
3. Meter every 2x15 pin to its first visible via, IC pin, connector pin, or
   passive endpoint; record opens as well as connections.
4. Reconcile those nets against Figure 5 and mark each result as measured or
   inferred.
