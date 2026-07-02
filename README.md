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

## License

AGPL-3.0-or-later.
