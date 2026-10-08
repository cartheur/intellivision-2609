# Pre-prototype readiness: four remaining gates

The documentary and architectural preparation for the Intellivision live-play
system is substantially complete. Before active hardware work is treated as
ready for open-ended iteration, close these four gates with recorded evidence.

## 1. Verify the physical interface on the reference hardware

Do not derive active electrical behaviour from a plausible pinout alone.

- Identify the exact Master Component (2609) and controller revisions used as
  the reference bench system; record an ECS separately only if it is part of a
  later corroboration experiment.
- Confirm connector orientation, signal identity, voltage levels, timing,
  loading limits, reset behaviour, and signal ownership from primary documents
  and measurement.
- Use passive fixtures, high-impedance observation, and continuity testing
  before attaching active circuitry.
- Record observations, equipment, conditions, and any disagreement with the
  documentary record.

**Gate evidence:** a revision-specific interface dossier and a bench log that
states which signals may be observed, which may be driven, under what limits,
and which remain unresolved.

## 2. Validate the HD6309 companion as an independent computer

The first board should succeed as a self-contained, debuggable computer before
it is connected to an original console.

- Bring up regulated power, reset supervisor, clock, ROM/RAM, serial monitor,
  LEDs, and trace memory on the bench.
- Verify address/data/control timing and every local bus cycle with the board
  disconnected from the Intellivision.
- Test cold start, reset, power interruption, host loss, buffer exhaustion, and
  watchdog behaviour.
- Make fault state safe by default: no controller or console line may be driven
  after reset or when the board's state is unknown.

**Gate evidence:** reproducible monitor output, memory and timing test results,
a documented fault-state table, and a versioned schematic/wire-wrap record.

## 3. Establish one-way observation before actuation

First build the ability to observe a real game without influencing it.

- Capture stable video with timestamps, preferably through an external path
  that leaves the console unmodified.
- Record human controller events and synchronise them to video frames or a
  common clock.
- Define the minimum features available to an agent: raw frames, extracted
  features, controller history, timer, and experiment state.
- Demonstrate that the observation system is passive: the game works normally
  with it connected, disconnected, powered off, or faulted.

**Gate evidence:** a replayable human-play recording with video, controller
events, timing metadata, and a documented capture latency/error estimate.

## 4. Specify the first experiment before teaching the machine

Choose one bounded game task and make success falsifiable.

- Select a legally obtainable game with repeatable maze-like navigation,
  manageable actions, and a practical reset/start procedure.
- State the fixed game version, console configuration, start state, action
  vocabulary, action duration, and trial termination conditions.
- Define baselines: a human run and at least one transparent fixed policy such
  as wall-following or bounded exploration.
- Require an Ariadne trace for every machine turn: observation identifier,
  state, allowed actions, selected action, authority, duration, consequence,
  and firmware/policy version.

**Gate evidence:** a short experiment protocol, a valid replay trace, and
published criteria for success, failure, comparison, and human takeover.

## Go/no-go rule

Proceed from one gate to the next only when its evidence is archived in the
repository. Controller emulation comes after the observation gate; any console
bus/mailbox interface comes only after the interface dossier and independent
board validation demonstrate that it is electrically justified.
