# ECS compatibility track: deployment plan

## Purpose

This is the **corroborative ECS compatibility track** for the custom companion
project. It develops a new computer subsystem that can be deployed with either
an original Intellivision I or an Intellivision II and an ECS. It does not gate
the primary 2609 live-play companion, which begins with external video and
controller observation. The project must not assume
that the two consoles have identical electrical expansion behavior. They are
parallel deployment targets until documentation and measurement establish a
shared interface contract.

The first deliverable on this track is a safe, console-CPU-controlled serial
monitor. It proves the ECS interface before expansion RAM, storage, a second
processor, or other computer services are attempted.

## Target policy

| Target | Status | Deployment rule |
|---|---|---|
| Intellivision I + ECS | Candidate | Validate its physical connection, address map, timing, and power budget independently. |
| Intellivision II + ECS | Candidate | Validate the same properties independently; do not inherit them from Intellivision I. |
| One shared board | Desired, not assumed | Permit only when both targets pass the same interface and electrical tests. |
| Target-specific adapter or board revision | Acceptable fallback | Use when connector, signal, timing, loading, or power differences prevent a safe shared design. |

## Definition of success

The subsystem is successful when it can, on each supported target:

1. Boot without altering normal console or ECS behavior when absent or disabled.
2. Respond only within a verified expansion address window.
3. Safely latch addresses, return read data, accept writes, and release the
   shared bus outside its selected transaction.
4. Run a polling serial monitor that transmits, receives, and echoes data.
5. Use the ECS keyboard and console display through explicitly defined software
   behavior, not an undocumented electrical assumption.
6. Meet measured voltage, loading, current, reset, and timing limits.

## Phase 0 — inventory and source reconciliation

Create one target record for every console/ECS pair available for deployment.

Record:

- Console model, serial number, board revision, region, and power-supply type.
- ECS model, revision, cables/connectors, and any attached cartridge or
  peripheral.
- Service-manual and schematic source used for each observation.
- Known working state: video, controllers, cartridge slot, ECS keyboard, and
  ECS BASIC if available.

Use `manuals/Intellivision_Service_Manual_Model_2609.pdf` for the original
console, `manuals/Intellivision_II_Service_Manual_1982_Mattel_US.pdf` for the
later console, and the CP1600 materials in `docs/` for processor behavior. The
absence of ECS-specific primary documentation is itself a tracked risk, not a
gap to fill by guesswork.

**Exit gate:** every target pair is uniquely identified and known-good before
custom hardware is introduced.

## Phase 1 — ECS interface dossier

For Intellivision I + ECS and Intellivision II + ECS separately, determine and
record:

| Question | Evidence required |
|---|---|
| Physical route | Connector names, pin numbering, mating orientation, and cable path. |
| Signal function | Direction, nominal voltage, active polarity, and owner for each used signal. |
| Addressing | Free or decoded address windows, expansion selects, and collision risks. |
| Bus timing | Address/data phases, control-signal timing, clock relationship, and valid data windows. |
| Electrical limits | Loading, current budget, protection, and maximum safe logic family/voltage. |
| System control | Reset, interrupt, bus request/acknowledge, and boot behavior. |
| Software entry | How the driver is loaded, started, and made available to ECS BASIC or a monitor. |

Use passive observation first: continuity checks on unplugged equipment, then
high-impedance logic analysis or oscilloscope probes on a known-working setup.
Do not attach a driving prototype merely to discover a pin function.

**Exit gate:** each used signal has a source, measurement, direction, and
electrical limit. Any unknown signal is excluded from the first board.

## Phase 2 — compatibility decision

Compare the two dossiers and make one explicit decision:

- **Shared interface:** pinout, timing, address selection, and power behavior
  are compatible within measured margins.
- **Shared core with adapters:** the bus contract is equivalent but the physical
  route or passive adaptation differs.
- **Separate revisions:** a meaningful electrical or timing difference requires
  target-specific logic or layout.
- **Single-target prototype:** begin with the better-documented or more easily
  measured target, while retaining the other as a future validation target.

Do not choose “shared interface” from model-family naming alone.

**Exit gate:** an interface-control document names the supported target(s),
used pins, address window, timing assumptions, power source, and every adapter
or revision.

## Phase 3 — passive and read-only proof board

Build the lowest-risk board possible:

1. Start with no bus driver enabled by default.
2. Latch the address only on verified CP1600 bus phases.
3. Decode one approved address.
4. Return one fixed read value through tri-state buffers only when selected.
5. Verify that the board releases the bus in every other state.

Test this first on the chosen lead target, then repeat the same checks on the
other target before claiming compatibility. Capture logic traces and power
measurements as project evidence.

**Exit gate:** stable boot, correct fixed-value reads, no observed contention,
and acceptable supply behavior on each claimed target.

## Phase 4 — serial monitor

Add the memory-mapped UART register block:

| Offset | Function |
|---:|---|
| `00` | Read received data / write transmit data. |
| `01` | Read status. |
| `02` | Read/write UART control. |
| `03` | Reserved auxiliary or future interrupt/baud control. |

Begin with polling at a conservative speed. Validate transmit before receive;
then demonstrate an echo loop and a simple command protocol. Use a true
RS-232 level converter between the UART and DB9 connection. Do not expose
TTL-level UART pins as RS-232.

**Exit gate:** repeatable receive/transmit/echo tests with no console or ECS
regression on each supported target.

## Phase 5 — computer environment

After serial proof, add capabilities in this order:

1. Loader and diagnostic monitor.
2. ECS keyboard-aware console interface.
3. Expansion RAM, with an explicit allocation and memory test.
4. Storage or host-transfer protocol.
5. Command shell or ECS BASIC integration.
6. Optional local processor only with a separately proven arbitration and reset
   design.

Each new service requires its own address, interrupt, power, and regression
review for both targets.

## Safety and stop conditions

Stop testing immediately if either target shows unstable video, reset loops,
excessive current, warm components, bus activity while the custom board is not
selected, or altered behavior after the board is removed. Return to the last
known-safe configuration and inspect captured measurements before proceeding.

Never use an ordinary CP1600 wait state as a substitute for a slow or unknown
peripheral. The CPU documentation limits ordinary `BDRDY` waits because the CPU
has dynamic internal state. Treat bus ownership and timing as first-class design
requirements.

## Near-term checklist

- [ ] Identify the available Intellivision I, Intellivision II, and ECS units.
- [ ] Photograph and record each board and connector path.
- [ ] Locate or acquire ECS-specific schematics and connector documentation.
- [ ] Produce separate I + ECS and II + ECS interface dossiers.
- [ ] Decide shared board, adapter approach, separate revisions, or lead target.
- [ ] Design a passive/read-only proof board.
- [ ] Capture proof-board traces and measurements on every claimed target.
- [ ] Add the polling serial monitor only after the proof gate passes.
