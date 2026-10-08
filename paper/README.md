# Paper prospectus: accountable intuitive programs

## Working title

*Toward Accountable Intuition: A Traceable Live-Play Architecture for Bounded
Machine Action on the Intellivision*

## Current paper type

This is presently a design-theory and experimental-protocol paper. Its proposed
first intelligence threshold is an **intuitive program**: a bounded program
that identifies task-relevant differences, ranks permitted candidate actions,
and revises that tendency from observed consequences.

It does not yet claim general machine intelligence, completed learning results,
or a controller-emulation connection to the 2609. Its current contribution is
an accountable action architecture: observation, candidate set, ranking,
authority decision, bounded action, verification, and trace are distinct
records.

## Apparatus

- Original Intellivision Master Component (2609): authoritative game and
  display system.
- M6x09-I SBC: operational 6x09 reference computer with verified 19,200 8N1
  RS-232 monitor path; first RAM-loaded experiment platform.
- Wire-wrapped HD6309 board: primary future hardware artifact.
- Host computer: video capture, trial archive, and optional learned policy.
- Human: normal player, opponent/co-player, and immediate takeover authority.

## arXiv readiness

Before submission, the manuscript needs a stable versioned data/code archive,
clear reproducibility instructions, final figures, a full related-work section,
and a claim audit that separates demonstrated measurements from proposals. If
it presents empirical intelligence/performance claims, it also needs preserved
trial data, fixed baselines, stated evaluation criteria, and an error analysis.

The first LaTeX draft intentionally uses a conventional `article` class and
standard packages so it can later move to the arXiv `article` template with
minimal changes.

## Literature position

The paper is adjacent to programmatic reinforcement learning, explainable
reinforcement learning, and formal/verified decision systems. Its claim should
not be that policies represented as programs are new. Its proposed novelty is
an embodied, provenance-first system in which a candidate-ranking policy is
separated from authority to act, constrained by a physical fault-neutral and
human-takeover boundary, and evaluated in live legacy-game sequences with an
end-to-end replayable trace.

The initial survey includes work on gridworld programmatic RL, recent
interpretable programmatic RL for scheduling, LLM-guided programmatic-policy
synthesis, interactive temporal explanations for RL, and a recent survey of
explainable RL. It will need expansion and bibliographic normalization before
submission.

## Files

- [First LaTeX draft](intuitive-program.tex)

Build locally:

```sh
cd paper
pdflatex intuitive-program.tex
```
