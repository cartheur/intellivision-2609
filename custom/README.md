# Custom Intellivision companion computer

This directory develops a new, inspectable companion computer for an original
Intellivision Master Component (model 2609). Its primary purpose is live
gameplay research: the console runs the game and produces the display; a
wire-wrapped HD6309 board and a host system observe play, retain a trace of
decisions, and—only under explicit, fail-safe authority—offer controller input.

The project does **not** begin by assuming an Intellivision II + ECS expansion
interface. That remains valuable historical and technical work, but it is a
corroborative compatibility path, not a prerequisite for the first board.

## Two deliberate tracks

### Primary track: 2609 live-play companion

```text
M6x09-I-SBC  → validated monitor / expansion testbed
             → HD6309 wire-wrap companion
             → passive video/controller observation
             → isolated controller emulation
             → 2609 game and display output
             → optional host learning loop
```

The operational on-hand M6x09-I-SBC is the immediate reference platform: its
6xC09-family socket, RAM, EPROM, ACIA, verified RS-232 monitor workflow, and
40-pin expansion header let the project validate software and a small
daughterboard before new construction begins. The M6x09-II-SBC remains useful
as a separate diagnostic/reference design, but its ACIA serial acceptance is
still pending. The HD6309 wire-wrap board
remains the primary artifact: a self-contained, fully inspectable companion
with local RAM/ROM, serial diagnostics, trace memory, watchdog, and visible
fault state. Controller emulation comes only after the observation path is
validated and must default to neutral, with a physical human/machine selector.
A console-bus mailbox is optional and strictly later work.

Start here:

- [Live-play cyberneticist working proposal](LIVE-PLAY-CYBERNETICS-PROPOSAL.md)
  defines the traceable machine-play architecture and experiment design.
- [Pre-prototype readiness](PRE-PROTOTYPE-READINESS.md) gives the four evidence
  gates before active hardware is introduced.
- [First-approach build](FIRST-APPROACH-BUILD.md) turns the on-hand parts
  inventory into a staged HD6309 wire-wrap bench build, preceded by the
  M6x09-I-SBC reference-platform path.
- [W65C02SXB live-play specification](W65C02SXB-LIVE-PLAY-SPEC.md) defines an
  alternative rapid-prototyping path using an SXB and a small XBus daughterboard.
- [Master Component rebuild readiness](MASTER-COMPONENT-REBUILD-READINESS.md)
  distinguishes preserved console firmware/documentation from artifacts needed
  for a source-buildable #2609 replica.

### Corroborative track: Intellivision II/ECS compatibility

The ECS provides important historical keyboard and expansion context. A future
memory-mapped serial monitor or computer environment may target an
Intellivision I or II with ECS, but each physical route, address map, timing
contract, power budget, and bus-ownership rule must be proven independently.
No ECS result is assumed to establish the safe controller/video path above, and
no 2609 companion result is assumed to establish an ECS expansion interface.

- [ECS compatibility deployment plan](PLANNING.md) retains the separate,
  evidence-led plan for I + ECS and II + ECS validation.
- [ECS serial-monitor suitability notes](suitable-build.md) describe that
  later, bus-connected first board.
- [Platform comparison and modification notes](MODS.md) record differences
  relevant to choosing a corroborative target.

## Shared safety rule

The original console must remain functional when the companion is absent,
disconnected, unpowered, reset, or faulted. No plausible pinout authorises
active driving: every used electrical path requires primary evidence and bench
measurement. The console CPU remains the only assumed bus master; arbitration
is a separate research problem, never a shortcut.

## Relationship between the tracks

The primary track answers the immediate question: can a machine participate in
live Intellivision gameplay with an accountable record of what it saw, chose,
and did? The ECS track can later corroborate interface knowledge, provide
keyboard/software context, and support a more traditional computer environment.
It must add evidence or capability; it must not become an undocumented
dependency of the live-play companion.
