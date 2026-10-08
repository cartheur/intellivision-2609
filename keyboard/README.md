# Intellivision Keyboard Component

This folder records findings for the Mattel Intellivision Keyboard Component
(`Intelligent Television`). The corresponding primary source is the scanned
[Keyboard Component Service Manual](../manuals/Keyboard_Component_Service_Manual.pdf).

## Technical summary

The Keyboard Component is an expansion computer, not merely a keyboard. It
docks a Master Component in the upper recess and provides a complete keyboard,
an internal cassette subsystem, a **Computer II** logic board, and a dedicated
power system. The service manual calls out the following serviceable
subassemblies:

- Computer II assembly (logic board)
- keyboard assembly
- tape-control assembly and cassette deck
- switching power-supply assembly and transformer

### Architecture and interfaces

- The Master Component remains necessary: the Keyboard Component connects to
  the Master Component's cartridge port through a signal connector and also
  supplies a separate power lead.
- The manual's start-up test reports `CPU1-UP` and `CPU2-UP`. This supports a
  **dual-processor** Computer II design, though the manual does not name the
  processors or their clocks.
- The self-test explicitly checks graphics RAM and a keyboard I/O port. The
  amount and type of RAM are not specified by this manual.
- The system uses a television and antenna-switch unit for output, via the
  connected Master Component. It should therefore not be treated as a
  self-contained monitor computer.

### Cassette and diagnostics

- The cassette subsystem has separate tape-control and tape-deck assemblies.
  Diagnostics exercise eject and play operation; maintenance instructions call
  for cleaning the heads, capstan, and pinch roller.
- Power-up initiates an automated logic self-test. On completion, the operator
  selects keyboard or tape testing. The keyboard test displays a keyboard map
  on the television.

### Power

| Point | Manual value |
| --- | --- |
| Transformer input (US version) | 105–125 VAC |
| Transformer output at power-supply J1 | 17–23 VAC |
| +5 V acceptance range | +4.65 to +5.15 VDC |
| +12 V acceptance range | +11.64 to +12.36 VDC |
| −5 V acceptance range | −4.85 to −5.00 VDC |
| Adjustable rails | +5 V and +12 V; −5 V is not adjustable |

The power values are service tolerances, not a modern external power-supply
specification. Treat the unit as mains-powered equipment and service it only
with suitable isolation and safety practices.

## LaTeX facsimile

`KeyboardComponentServiceManual.tex` creates a title and provenance page, then
embeds every page of the original 28-page service manual using the `pdfpages`
package. This preserves every original illustration, wiring figure, layout,
and service table without inventing a noisy OCR transcription or redrawing
technical figures inaccurately.

Build from this directory:

```sh
latexmk -pdf KeyboardComponentServiceManual.tex
```

The result is `KeyboardComponentServiceManual.pdf`. The source manual must
remain at `../manuals/Keyboard_Component_Service_Manual.pdf`.

### OCR-assisted edition

`KeyboardComponentServiceManual-OCR.tex` is a separate LaTeX edition backed by
`KeyboardComponentServiceManual-OCR-source.pdf`. That source preserves the
original page artwork and adds a searchable OCR text layer. Its sidecar text is
available as `KeyboardComponentServiceManual-OCR.txt` for searching, extraction,
and later human correction.

Build it with:

```sh
latexmk -pdf KeyboardComponentServiceManual-OCR.tex
```

The OCR is intentionally not presented as an authoritative transcription:
faint type, diagrams, and tables cause recognition errors. Use the page image
as the authority whenever a technical value or label matters.

## Extracted figures

The scanned service figures are available in [figures/](figures/). The three
test-setup drawings from source page 5 have been cropped into separate images;
the later assembly/service diagrams are preserved as complete source pages to
avoid losing callouts or annotations.

## Scope and confidence

Direct manual evidence supports the subassemblies, power rails, test flow,
dual-CPU indication, and Master Component dependence above. CPU identity,
clock rates, RAM capacity, cassette encoding, data rate, and video timing are
not specified in this service manual; they require board inspection,
schematics, or component markings.
