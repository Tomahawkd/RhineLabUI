# Feature specifications

Write a spec here **before implementing a new feature**. Maintain that same spec throughout implementation, later extensions, and bug fixes. The repository rule is in [AGENTS.md](../AGENTS.md#功能规格与变更记录).

Use the next unused four-digit ID and a descriptive slug. Choose either `NNNN-feature.md` for a single file or `NNNN-feature/README.md` when diagrams, research, or supporting files need their own directory. Keep the ID stable and register the spec below. `_template.md` is a starting point, not a feature spec.

Every spec must contain background, a landing plan, a test plan, and a change log. Include its status and distinguish proposed behavior from shipped behavior. Suggested states are Draft, Planned, In progress, Implemented, and Verified; state outstanding work explicitly.

Before modifying a feature, find its existing spec. If a legacy feature has none, create a scoped spec before changing it. For each change to scope, behavior, implementation, acceptance criteria, or plan—including bug fixes—update the relevant sections and append a dated change-log entry in the same change. Record what changed, why, and what was validated or remains untested. Keep previous entries; Git history alone does not replace the log.

Reference shared visual requirements in [DESIGN.md](../DESIGN.md) and detailed evidence in [verification/](../verification/). Test plans describe intended checks; only report a pass after running them. A planning request does not authorize feature implementation.

| ID | Feature | Status |
| --- | --- | --- |
| 0001 | [Mobile support](0001-mobile-support.md) | Draft — planning only |

Start a new spec from [the template](_template.md), removing its placeholder text and adding an initial change-log entry.
