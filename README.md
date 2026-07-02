# cloud-itonami-isco-1223

Open Occupation Blueprint for **ISCO-08 1223**: Research and Development Managers.

This repository designs a forkable OSS business for an independent R&D manager: a lab-walkthrough robot performs equipment-status checks and experiment-evidence capture under a governor-gated actor, so the practice keeps its own project and safety records instead of renting a closed R&D-management SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a lab-walkthrough robot performs equipment-status checks and experiment-evidence capture under an actor that proposes
actions and an independent **R&D Management Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
approving a hazardous-material experiment protocol, or releasing unpublished research data) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
research portfolio + project scope + safety protocol
        |
        v
R&D Advisor -> R&D Management Governor -> coordinate/review, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `1223`). Required capabilities:

- :robotics
- :identity
- :forms
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## Reference implementation (`:maturity :implemented`)

Full itonami Actor pattern (per ADR-2607011000 / CLAUDE.md's Actors
section, alongside `cloud-itonami-isco-6130`, `-8160`, `-2166`, `-2641`,
`-2651`, `-2652`, `-2654` and `-1219`): a real
[`kotoba-lang/langgraph`](https://github.com/kotoba-lang/langgraph)
`StateGraph`, with the Advisor and Governor as distinct graph nodes and
human-in-the-loop interrupt/resume via checkpointing.

```text
:intake -> :advise -> :govern -> :decide -+-> :commit            (:ok? true)
                                           +-> :request-approval   (:escalate? true, interrupt-before)
                                           +-> :hold               (:hard? true)
```

- `src/rd_management/store.cljc` — `Store` protocol + `MemStore`:
  registered projects, committed records, an append-only audit ledger.
- `src/rd_management/advisor.cljc` — `Advisor` protocol; `mock-advisor`
  (deterministic, default) proposes an R&D operation from a request;
  `llm-advisor` wraps a `langchain.model/ChatModel` — either way the
  advisor only ever produces a `:propose`-effect proposal, never a
  committed record, and LLM parse failures always yield `confidence 0.0`
  (forces escalation, never fabricated confidence).
- `src/rd_management/governor.cljc` — `RDManagementGovernor/check`: a
  pure function, wired as its own `:govern` node. Hard invariants
  (unregistered project, a proposal whose `:effect` isn't `:propose`)
  always route to `:hold`. Escalation invariants
  (`:approve-hazmat-protocol`, `:release-research-data`, or low advisor
  confidence) always route to `:request-approval` — an
  `interrupt-before` node that the graph checkpoints and only resumes on
  explicit human approval (`actor/approve!`), matching the README's
  robotics-premise statement that approving a hazardous-material
  experiment protocol and releasing unpublished research data always
  require human sign-off.
- `src/rd_management/actor.cljc` — `build-graph`, `run-request!`,
  `approve!`: the `langgraph.graph/state-graph` wiring itself.

```bash
clojure -M:test
```

This is what backs this repo's `:maturity :implemented` entry in
[`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation).

## License

AGPL-3.0-or-later.
