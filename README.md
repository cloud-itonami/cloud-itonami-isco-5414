# cloud-itonami-isco-5414

Open Occupation Blueprint for **ISCO-08 5414**: Security Guards.

This repository designs a forkable OSS business for an independent security guard practice: a patrol-support and access-log robot manages perimeter patrol and access records under a governor-gated actor, so the practice keeps its own security records instead of renting a closed guard-management SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a patrol-support and access-log robot performs perimeter patrol logging, badge scanning and incident-report printing under an actor that proposes
actions and an independent **Security Guard Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
use-of-force or detention action) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
site contract + access policy + incident report
        |
        v
Security Advisor -> Security Guard Governor -> dispatch patrol/log incident, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `5414`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
