# Building the agent-native-baseline profile — agent contract

You're working on the **flagship agent-native-setup profile**. A profile is two things:
`profile.json` (its config) and `templates/` (the files it ships into scaffolded projects).
Everything else here — including this file — is meta and never ships.

## Rules

- **Derived until stage D (RFC 2026-07-05 §7 in the engine repo):** the templates are built
  from the engine's generators by `profiles/agent-native-baseline/build.py` there, and the
  engine's parity gate (`tests/test_flagship_parity.py`) asserts whole-tree byte equality
  across the config matrix. **Do not hand-edit a built template** — change the generator
  constant and rebuild, or the parity gate will catch the drift.
- **Deliverables go in `templates/`.** A file at `templates/foo/bar.md` lands at `foo/bar.md`
  in every scaffolded project. `.j2` renders with `project_name` / `slug` / `description`,
  the prompts' `answers.<name>`, and the sensed `env.<name>` facts; anything else ships
  verbatim (a literal `${{ … }}` is safe). `env` never echoes a choice — user choices are
  this profile's own prompts.
- **Releases are tagged and pinned.** Bump `profile.json`'s `version`, tag `vX.Y.Z`, then
  update the engine's `profiles/baseline-pin.json` (tag + content hash over
  `profile.json` + `templates/`) and its vendored copy in the same engine PR. The engine's
  CI verifies the pin resolves to this exact artifact.
- **Before calling it done**, run `agent-native-setup profile validate .` and fix every
  finding.
