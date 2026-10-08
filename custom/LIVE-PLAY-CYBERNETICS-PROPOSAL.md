# Working proposal: Intellivision live-play cyberneticist

## Status and purpose

**Draft for discussion — not a hardware specification.** This proposal turns
the repository's preservation work into an experimental platform: an original
Intellivision Master Component (model 2609) runs an unmodified game and drives
its normal display, while a new, inspectable companion board and host system
observe play, retain an explicit history, and—when authorised—supply player
inputs. The platform is intended for experiments in live machine gameplay,
maze navigation, and mixed human/machine play.

It is inspired by the late-2026 episodes of *The Last Cyberneticist*, especially
their deliberately modest account of memory, comparison, state, and traceable
program construction. That intellectual intersection supplies design questions;
it is **not** evidence of the 2609's undocumented electrical interface or an
argument that a small board is inherently intelligent.

## The central question

Can a machine take part in an Intellivision game as an accountable participant?

The answer must be observable. For each action the system should be able to
state:

1. What it observed.
2. What difference it treated as consequential.
3. What internal state and policy rule selected the action.
4. What action it offered at the controller boundary.
5. What occurred afterwards, including the path through a maze or a loss.

That is a stronger and more useful claim than calling a board or a trained model
"intelligent." It supports comparison between a fixed rule set, a learned host
policy, and a human opponent while leaving the original game machine intact.

## Cybernetic frame

The proposal adopts six practical readings of the series.

| Episode theme | Design consequence for the live-play system |
| --- | --- |
| Behavior diagrams and inherent stability | Express the agent as named modes, activities, transitions, and guard conditions; publish its state diagram before claiming behaviour. |
| A small wire-wrapped memory machine and the maze | Begin with a constrained game and an observable maze task. Route choice, memory, error, and revised behaviour can be measured without grand claims. |
| Memory, readback, reset, and revision | Use explicit, inspectable memory. Log every state write; treat reset and revision as designed operations, not failures. |
| A difference that makes a difference | Define signals operationally: a captured screen feature, game event, timer edge, or human action becomes a signal only if it can alter a permitted decision. |
| Machine language as a bounded finite-state system | Separate state, legal transition, timing, and evidence. LEDs and a comparator may expose state, but they are not learning by themselves. |
| Algebra, algorithm, and Ariadne thread | A vocabulary of states and lawful transforms is insufficient. The system must specify next-step rules and retain an ordered trace of choices, branches, errors, and termination. |
| A machine must know its next move | Implement a visible cycle of fixed target, revisable candidate, comparison, authorised next action, verified state update, and halt. A proposed action must never silently become an electrical act. |

The resulting discipline is deliberately compatible with modern ML while not
depending on its opacity: a host learner may propose a policy, but the edge
system must record the observation, action, timing, policy version, and outcome
needed to replay and challenge that proposal.

## System boundary

```text
             video                         capture / feature extraction
2609 Master Component ──► television ─────────────────────────────────┐
        │                                                              │
        │ verified, passive-first expansion interface                  ▼
        │                                                       host learner
        ▼                                                              │
HD6309 companion board ◄──── serial protocol / traces / policies ─────┘
        │
        └──── isolated controller-emulation path ────► controller port
                                      ▲
                              human/machine selector
```

### Roles

- **2609:** the original game's CPU, video producer, and timing authority.
  It must continue to work without the companion.
- **HD6309 wire-wrap board:** a deterministic, inspectable gateway: timers,
  mailbox registers, event capture, trace buffering, watchdogs, and controlled
  controller emulation. It is not presumed to run a contemporary learned model.
- **Host:** video ingestion, training, experiment control, archive management,
  and high-capacity policy evaluation. Host decisions are versioned and sent as
  bounded commands or policy tables.
- **Human:** may play normally, take over instantly, or serve as an opponent.
  Human/machine authority is explicit in the trace.

## Observation and action

The first observation channel should be external video capture, paired with
controller and system timestamps. It gives the learner substantially the same
evidence available to a human and avoids assuming that every game's private RAM
or bus timing is safely readable. A later, game-specific instrument may add
labels such as player position, score, lives, or maze cell; these are optional
annotations, never a substitute for the video record.

The first action channel is electrically isolated controller emulation, with a
physical human/machine selector and a default-disconnected state. It must never
drive a line until that exact controller interface has been measured. The board
may request a directional/keypad action only for a bounded interval; a watchdog
returns it to neutral on reset, timeout, host loss, or fault.

### The decision/actuation contract

Episode 12 makes the safety and interpretability requirement concrete. Each
machine turn is a small, auditable transaction—not an opaque instruction from a
host that the hardware merely obeys:

```text
fixed target → proposed candidate → compare against current evidence
             → check authority and guard conditions → issue bounded action
             → verify / record outcome → next state or explicit halt
```

For this project, `target` is an experiment-level objective or current
finite-state subgoal; `candidate` is a host- or board-proposed controller
action. The comparator and guards decide only whether the candidate is lawful
under the current mode, ownership selector, timing budget, and fault state.
They do not themselves actuate a controller line. The board records whether an
action was accepted, emitted, observed at its interface, and followed by an
expected or anomalous consequence. A failed verification enters a named error
state and returns control to neutral or the human; it cannot be silently
relabelled as success.

This applies the episode's distinction carefully: a comparison is evidence, not
a write, and a requested controller action is not an established act until the
relevant local state and trace record have been verified. The proposal does not
claim that this bounded controller has judgment or general intelligence. Its
value is that it knows the next permitted move within a named authority, refuses
the forbidden one, and leaves an inspectable record.

## Hardware principle: no accidental second bus master

The HD6309 board starts as a peripheral, not a peer processor that seizes the
Intellivision bus. Its early interface should be external controller/video
observation or a fully passive measurement fixture. A small memory-mapped
mailbox is optional later work, selected only after the exact 2609 signal,
voltage, timing, loading, reset, and ownership rules are verified on the bench.
An ECS setup may corroborate that work but is not a prerequisite. Bus
arbitration is a separate research milestone, not a convenience feature.

This preserves the repository's core rule: no active hardware is connected on
the basis of a plausible pinout alone.

## First experiment: maze as bounded world

Choose one game with repeatable maze-like navigation and a manageable action
set. Define a trial as:

```text
reset → observe → classify state → choose action → apply for N ticks
      → observe consequence → append trace → continue / halt
```

Success is not just a high score. The trial must yield a replayable account:

- input video-frame identifiers or extracted features;
- agent state and allowed transitions;
- selected action, duration, and authority (human or machine);
- observed consequence and reward/event;
- policy and firmware hashes;
- a monotonically ordered decision trace (the Ariadne thread).

Initial policies should be simple enough to falsify: wall-following, explicit
maze maps, target seeking, and bounded exploration. A learned policy may then
be evaluated against the same trace schema and against a human opponent or
co-player. The experiment therefore asks not merely whether an agent wins, but
when it generalises, when it becomes confused, and whether its choices can be
reconstructed.

## Phased deliverables

1. **Evidence and safety dossier.** Resolve the exact controller electrical
   behaviour, video capture path, power budget, and reset/fault states for the
   particular 2609 hardware. Build passive fixtures first. Treat any ECS
   interface as a separate, corroborative dossier.
2. **HD6309 bench computer.** Wire-wrap CPU, RAM/ROM, serial monitor, clock,
   reset supervisor, LEDs, and a trace-memory test. Verify every bus cycle
   locally before connecting to the console.
3. **One-way observatory.** Video capture plus timestamped human-controller
   logging; no controller injection and no console-bus driving.
4. **Fail-safe actuation.** Add optically or otherwise electrically isolated
   controller emulation, takeover switch, watchdog neutralisation, and a
   recorded authority flag.
5. **Inspectable agent.** Implement a finite-state maze policy and visualise
   its modes, transitions, state, and trace in real time.
6. **Host learning loop.** Train/evaluate a policy from recorded trials; retain
   model, data-set, firmware, and replay identifiers for each result.
7. **Optional console mailbox.** Only after evidence permits it, exchange small
   structured messages with software executing on the 2609.

## Falsifiable acceptance criteria

- The 2609 and game remain functional with the companion powered off,
  disconnected, and in fault/reset states.
- A human can regain exclusive control without host software or a successful
  firmware transition.
- Every non-neutral machine input is recorded with a timestamp and source.
- Recorded trials can be replayed sufficiently to account for a decision path,
  even when a learned policy cannot be reduced to a simple rule.
- A claimed behavioural improvement is evaluated against fixed game versions,
  starting states, trial counts, and a stated human or baseline policy.
- No undocumented expansion signal is actively driven until measured evidence
  establishes its ownership and limits.

## Open questions

- Which 2609 revision and controller hardware will serve as the reference
  bench system, and what separate ECS configuration—if any—will later be used
  for corroboration?
- What is the least invasive way to capture stable, low-latency video frames?
- Can controller observation/emulation be made entirely external to the
  expansion connector for the initial system?
- What game gives a legally obtainable, repeatable, and sufficiently observable
  maze benchmark?
- Which portions of the trace must reside on the HD6309 board during host loss?
- Is a later bus/mailbox interface valuable enough to justify its electrical and
  preservation risks?

## Source episodes

- [Episode 5: *The Inherent Stability Represented in Behavior Diagrams*](https://podscan.fm/podcasts/the-last-cyberneticist)
- [Episode 6: *A Four-Bit Wonder*](https://podscan.fm/podcasts/the-last-cyberneticist)
- [Episode 7: *The Benefits of Remembering*](https://podscan.fm/podcasts/the-last-cyberneticist)
- [Episode 8: *It is the Difference that Matters*](https://www.spreaker.com/episode/it-is-the-difference-that-matters--75047946)
- [Episode 9: *When the Machine Is Its Own Language*](https://www.spreaker.com/episode/when-the-machine-is-its-own-language--75185110)
- [Episode 10: *Multenions: A Structured Algebra*](https://www.spreaker.com/episode/multenions-a-structured-algebra--75331017)
- [Episode 11: *From Algebra to Algorithm: Multenions and Program Synthesis*](https://www.spreaker.com/episode/from-algebra-to-algorithm-multenions-and-program-synthesis--75480441)
- [Episode 12: *A Machine Must Know Its Next Move*](https://www.spreaker.com/episode/a-machine-must-know-its-next-move--75639925)

The episode descriptions support the conceptual readings above. All claims
about an Intellivision interface remain subject to primary manuals, schematics,
and bench measurement in this repository.
