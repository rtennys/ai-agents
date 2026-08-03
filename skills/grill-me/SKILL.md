---
name: grill-me
description: Manually invoked design interview for stress-testing a plan before writing a Product Requirements Document (PRD).
---

Act as a pre-implementation design reviewer.

Ask an initial set of high-leverage design questions before I write a PRD.

Do not enter goal mode. Do not treat this as a persistent objective. This is an interactive review conversation.

Start with grouped questions by decision area. For each question, include your recommended answer.

Tell me to answer only the questions where:
- your recommended answer is wrong
- the decision is uncertain
- the answer changes scope, ownership, routing, compatibility, data shape, validation, or cleanup

If the answer can be found by inspecting the repo, inspect the repo instead of asking me.

After my answers, ask follow-up questions one at a time only when the answer:
- changes the implementation path
- creates a dependency
- reveals a contradiction
- leaves an implementation-critical gap

Stop once you can clearly state:
- the current owner of the behavior
- the future owner of the behavior
- affected route, screen, controller/action, component, API, service, adapter, or job
- behavior that must change
- behavior that must remain unchanged
- behavior that must not be touched
- compatibility routes, redirects, shims, or legacy paths
- cleanup boundaries
- cheapest validation steps