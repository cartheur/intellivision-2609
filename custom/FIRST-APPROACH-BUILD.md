# First-approach build: HD6309 live-play companion

## Status and intent

**Bench-build proposal — not yet a console-interface schematic.** This is the
primary physical implementation of the 2609 live-play companion. It uses parts
recorded in the local `parts-is-parts` inventory to make a self-contained,
wire-wrapped HD6309 computer that can be validated before it observes or
interacts with an Intellivision. Its immediate engineering reference is the
operational on-hand M6x09-I-SBC, not a second untested computer built in
parallel.

The board's purpose is modest and inspectable: retain state, exchange serial
messages with a host, record a decision trace, and enforce a bounded,
fail-safe action contract. The original 2609 remains the game and display
machine. No console bus or controller line is driven in the initial build.

## First execution platform: M6x09-I-SBC

Use the operational M6x09-I-SBC to validate the approach before committing its
logic to wire-wrap. It supplies a 6xC09-family system with RAM, EPROM, 6850
ACIA, keypad monitor, verified RS-232 link, 10 ms tick source, and 40-pin
expansion header. It can therefore prove the monitor protocol, trace record
format, local register decode, timer behavior, and a passive/read-only
daughterboard without making the wire-wrap board carry every early uncertainty.

The M6x09-I monitor banner, keypad-driven memory dump, LCD, and Linux RS-232
path are recorded as verified at 19,200 8N1. The next specific proof is a
known-good S19 RAM load and application run; preserve that transcript as the
first companion-project evidence. The M6x09-II-SBC remains a useful separate
reference, but it must not gate this work while its ACIA CTS/serial acceptance
is unresolved.

```text
M6x09-I-SBC S19 RAM-run proof → local trace/daughterboard proof
                              → HD6309 wire-wrap implementation
                              → 2609 observation, then later actuation
```

## Architecture

```text
                         serial
host ────────────────────────────────────────┐
                                                ▼
HD6309 board: CPU ─ RAM / EEPROM ─ ACIA ─ MAX232
       │             │                │
       │             └─ trace memory   └─ diagnostic monitor
       │
       ├─ Target/Candidate comparator and visible state LEDs
       └─ authority request only ──► later isolated controller interface

ATmega328P supervisor: watchdog, host-loss detection, authority interlock,
fault indication, and neutral-state request. It is not the sole safety layer;
hardware must also make the inactive/fault state electrically neutral.
```

## Parts selected from inventory

| Function | Part | Inventory quantity | First-build use |
| --- | --- | ---: | --- |
| Main CPU | `HD63C09RP` | 9 | Use one; retain the remainder as spares and for later boards. |
| Direct CPU fallback | `MC6809P` / `EF6809P` | 2 / 2 | Keep as a compatible-family fallback, not as an initial parallel design. |
| Supervisor | `ATMEGA328P-PU` | 10 | Use one for host/fault/authority supervision. |
| Main RAM | `AS6C62256-55PIN` | 10 | 32 KiB, 8-bit asynchronous RAM. |
| Program store | `AT28C256-15PU` | 5 | 32 KiB EEPROM for monitor and initial fixed policy tables. |
| Serial interface | `MC68B50P` | 10 | ACIA for monitor, trace export, and host protocol. |
| RS-232 conversion | `MAX232N` | 10 | Electrical conversion for a true RS-232 serial boundary. |
| Clock source | `ECS-100AX-018` | 5 | 1.8432 MHz oscillator; validate the selected 6309 clock connection before use. |
| Address decode | `SN74LS138N` | 32 | RAM, EEPROM, ACIA, and later local-register selection. |
| Controlled buffering | `SN74HCT125N`, `SN74LS244N`, `SN74LS245N` | 10 / 20 / 20 | Local bus isolation and diagnostic expansion; do not use as a reason to attach to the console bus. |
| Reset/pulse conditioning | `SN74LS14N`, `SN74LS123N` | 23 / 10 | Reset shaping, debounce, and bounded-pulse experiments. |
| Visible comparison | `SN74LS85N` | 35 | Target/Candidate comparison for the explicit decision cycle. |
| Indicators / manual input | LEDs, `10TC405` switches, pushbuttons | various | Bring-up display, reset, and manual test controls. |
| Wire-wrap infrastructure | 40-pin/28-pin/24-pin/etc. wire-wrap sockets; 28 AWG Kynar | recorded | Socket every active IC and label every net. |

The inventory also contains smaller SRAMs, serial SRAM, EPROMs, 555 timers,
and other legacy processors. They are useful for separate experiments but are
not dependencies of this first board.

## Memory and local I/O map

The following is a provisional, internal board map. It does not imply any
Intellivision address or expansion-interface mapping.

| Range | Device | Role |
| --- | --- | --- |
| `$0000`–`$7FFF` | AS6C62256 | Main RAM, monitor workspace, and trace ring buffer. |
| `$8000`–`$80FF` | MC68B50 | Serial data/status/control registers. Decode only the required low addresses. |
| `$8100`–`$81FF` | local status/LED latch, later | Optional read-only build ID, fault state, and comparator result. |
| `$C000`–`$FFFF` | AT28C256 | Reset vectors, serial monitor, diagnostics, and fixed initial policy tables. |

The exact decode equations, chip-select polarity, reset vector arrangement,
and clock wiring remain schematic-review items. Do not wire from this table
alone.

## Build stages

### Stage -1 — M6x09-I-SBC RAM-run and daughterboard proof

1. Preserve the existing 19,200 8N1 monitor/banner and keypad-DUMP evidence as
   the known-good transport baseline.
2. Load a known-good S19 application into RAM, execute it, and record its
   expected output or GPIO behavior.
3. Document the 40-pin expansion-header pin contract from its schematic and direct
   measurement; do not infer a daughterboard interface from header position.
4. Use a passive/read-only daughterboard to prove a fixed register read and
   then the Target/Candidate trace format.

**Exit evidence:** a working serial monitor, S19 RAM-run transcript, measured
expansion-header dossier, and a repeatable read-only daughterboard test.

## Agent-guided engineering shadow

The build proceeds with an agent as a documented engineering shadow. Its role
is to help turn evidence into the next bounded question, cross-check data
sheets and address maps, draft test procedures, compare captures with stated
expectations, and preserve the reasoning in a form suitable for later podcast
work.

The agent does not replace measurement or receive implicit authority to alter
hardware. For every material step, preserve:

1. The observed fact, including instrument, probe location, configuration, and
   raw capture or transcript where practical.
2. The current hypothesis and its alternatives.
3. The next smallest discriminating test.
4. The human decision to wire, measure, program, or stop.
5. The outcome, including a correction when the hypothesis was wrong.

This produces a useful public engineering narrative without turning an agent's
plausible explanation into an untested hardware claim. The M6x09 ACIA
register-select diagnosis is the model: source, data sheet, ROM readback, and
instrumented behavior must agree before a conclusion is accepted.

## First intelligence experiments: intuitive programs

The initial intelligence milestone is a bounded intuitive program, not an
assertion of general intelligence. It receives a small live or replayed history,
identifies differences relevant to the current task, ranks permitted candidate
actions, and updates that ranking from outcome records. The HD6309 or M6x09
board makes the chosen candidate, guard result, action duration, and trace
identifier visible; the host may later provide learned rankings from video.

Start with transparent policies such as wall following, target seeking, and
bounded exploration. Then compare them against a learned candidate-ranking
policy using the same replay and trace format. Any uncertainty, contradiction,
or missing observation must yield a named pause/neutral/human-takeover state,
not an unrecorded guess.

### Stage 0 — paper design and socket map

1. Carry forward only the M6x09-I-SBC behavior that was demonstrated on
   hardware; confirm the HD63C09RP pinout, oscillator requirements, reset
   polarity, and bus timing from its primary data sheet.
2. Draw the complete local schematic, including every power pin, decoupling
   capacitor, pull-up/pull-down, unused-input treatment, reset source, and
   wire-wrap socket position.
3. Define the fault-state table before wiring: power-on, reset asserted, CPU
   stopped, supervisor absent, serial host absent, and watchdog timeout must
   all prohibit controller actuation.

**Exit evidence:** reviewed schematic, socket/wire list, address map, and
fault-state table.

### Stage 1 — standalone CPU, ROM, RAM, and monitor

1. Wire regulated 5 V, local decoupling, clock, reset, HD6309, EEPROM, RAM,
   and minimal decode only.
2. Install a tiny monitor that writes a known RAM pattern, reads it back, and
   exposes progress on LEDs.
3. Test cold start, warm reset, repeated reset, and power cycling. Record the
   expected LED sequence and any current/temperature anomaly.

**Exit evidence:** repeatable RAM test and reset behavior with no peripheral
or console connection.

### Stage 2 — serial monitor

1. Add the MC68B50 and MAX232 after Stage 1 is stable.
2. Start with transmit-only diagnostics; then add receive, echo, and a simple
   command protocol.
3. Timestamp or sequence-number every received host command and emitted trace
   record.

**Exit evidence:** repeatable host/board echo and captured monitor transcript.

### Stage 3 — visible decision-cycle demonstrator

Implement the Episode 12 control cycle locally, using switches and LEDs before
any external actuation:

```text
fixed Target → revisable Candidate → compare → guard/authorise
             → record verified local state → next state or halt
```

- Use the `74LS85` to make Target/Candidate relations visible.
- A comparison is evidence, not an implicit write.
- A rejected candidate and a failed verification enter named error/halt states.
- The HD6309 trace contains target, candidate, comparison, guard result,
  selected next state, and a monotonically ordered decision identifier.

**Exit evidence:** a serial trace and LED-visible demonstration of accepted,
rejected, error, and halt paths.

### Stage 4 — supervisor and safe authority boundary

1. Add the ATmega328P only after the HD6309 board is independently stable.
2. Give the supervisor a narrow role: monitor heartbeat/watchdog conditions,
   observe host presence, expose fault status, and require explicit authority
   before the later controller interface can be enabled.
3. Ensure that a supervisor reset, missing heartbeat, power loss, or firmware
   fault yields the same result: controller interface disabled and neutral.
4. Keep the authority selector physical and visible; human control must not
   depend on host software or a successful processor state transition.

**Exit evidence:** fault-injection log showing that every tested fault state
returns the authority boundary to disabled/neutral.

### Stage 5 — observe the 2609, without influencing it

Connect no electrical control path to the console. Establish external video
capture and timestamped logging of normal human controller use. The HD6309 may
receive summary events from the host, but the game must work identically when
the board is absent, unpowered, reset, or host-disconnected.

**Exit evidence:** replayable human-play trace containing video identifiers,
controller events, timing, firmware version, and board fault/authority state.

### Stage 6 — separately designed controller interface

Only after Stage 5, measure the exact controller electrical behavior and design
an isolated closure/emulation interface. It must be reviewed as its own
schematic and must not be inferred from this board's local logic design. Its
default and fault condition is electrically neutral; it cannot silently become
active because the HD6309 or host requests an action.

## Explicit deferrals

The following are outside the first-approach build:

- Intellivision cartridge/ECS/shared-bus driving or a memory-mapped mailbox.
- Bus arbitration, interrupts into the console, or reset control of the console.
- Any claim that the 6309 board itself performs general machine learning.
- The `MC6829` MMU, serial SRAM, 8080/8085 processors, 8087 coprocessor, and
  other alternate-architecture experiments.
- Game-specific RAM reading or instrumentation.

These may become later experiments only if they add evidence or capability
beyond the validated video/controller/trace path.

## First software milestones

1. Reset and LED heartbeat.
2. RAM march/readback test.
3. Serial transmit banner.
4. Serial receive/echo monitor.
5. Trace-ring write/read test.
6. Target/Candidate comparator demo with accepted and rejected transitions.
7. Supervisor heartbeat timeout and demonstrated neutral fault state.

## Completion criterion

This first approach is complete when the board can repeatedly boot, test RAM,
communicate over serial, demonstrate an inspectable bounded decision cycle, and
enter a documented disabled/neutral state under every tested fault—without any
electrical dependency on the 2609.
