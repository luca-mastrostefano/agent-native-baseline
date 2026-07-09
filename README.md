# agent-native-baseline

🌐 Part of the [agent-native-setup registry](https://lucamastrostefano.com/agent-native-setup/) — browse all community profiles.

The default [agent-native-setup](https://github.com/luca-mastrostefano/agent-native-setup)
profile: a complete, agent-native project setup. This is what
`agent-native-setup -o ./my-app` scaffolds by default — the engine vendors a hash-pinned
copy of a tagged release of this repo, so scaffolding is offline and the artifact you get
is exactly the one reviewed here.

## What you get

Scaffolded into a project, this profile ships:

- **The `AGENTS.md` + `INSTRUCTION.md` contract** — `INSTRUCTION.md` carries the four
  execution principles and when-to-write-an-RFC rules (managed, so an `update` keeps it
  fresh); `AGENTS.md` is the thin project map (navigation + live command surface) that
  points at it. `CLAUDE.md` and `GEMINI.md` (symlinks), `.cursor/rules/`, and
  `.github/copilot-instructions.md` all point back to `AGENTS.md`, so the rules never fork
  across tools.
- **A `.claude/` agent library** — focused subagents (`code-reviewer`, `rfc-reviewer`,
  `planner`), slash commands (`/review`, `/rfc`, `/update-agent-scaffolding`, `/onboard`),
  a permission allowlist for the contract's own commands, and hooks that inject the live
  command surface at session start and auto-format files as they're edited.
- **`docs/` + an RFC lifecycle** — a pre-seeded architecture map (reflecting the active
  RFCs), the `proposed → active → (superseded | retired)` RFC flow (with a template), a root
  `CONTRIBUTING.md` dev-loop guide, and an improvements backlog — kept in folder-sync and
  freshness by hooks.
- **`tools/checks/`** — small enforcement scripts (RFC ↔ folder sync, "new component
  needs a doc," "structural change needs an RFC") that ship with their own tests.
- **Per-language lint, format & types** — ruff (Python), ESLint + Prettier + tsc (JS/TS),
  golangci-lint + gofmt (Go), clippy + rustfmt (Rust), htmlhint + lychee (HTML), with
  their config files.
- **A three-layer quality gate** — the same checks wired at **pre-commit**, a
  self-documenting **`make`/`task`** command surface, and **CI**, so local-green means
  CI-green.
- **A security baseline** — committed-secret scanning (gitleaks) and dependency/vuln
  audits, in both pre-commit and a dedicated CI job, plus `SECURITY.md` and Dependabot.
- **Engineering baseline files** — `.editorconfig`, `.gitattributes`, per-language
  `.gitignore`, a PR template, a provenance manifest (`.agent-native-setup.json`, recording
  what generated the project for a future `update`), and (on existing repos) a
  `.git-blame-ignore-revs`.
- **A self-deleting `ONBOARDING.md`** — a one-time runbook that walks an agent through
  activating the setup on first run, then removes itself.

It's **non-destructive**: existing files are never overwritten, and on a repo that
already has code it **grandfathers the legacy code** — the gate checks only what a
pull request changes, so day one isn't a wall of red.

## Guardrails for your agent

These aren't just config files — they actively keep an agent (and you) on the rails:

- **A real testing bar.** Every change ships the test that *proves* it, at the right
  level — **unit** (logic + edge cases), **integration** (module / public-contract /
  boundary crossings), and **regression** (a failing test written *first* for every bug).
  Tests must prove behavior, not restate the code: cover the boundaries (empty/zero/one/max),
  bad input, and error paths — not just the happy path.
- **A self-review pass before "done".** A `code-reviewer` subagent (`/review`) reads the
  diff and flags real bugs, over-engineering, drive-by changes, stale docs, weak or
  happy-path-only tests, and changes that hurt cohesion or sneak in coupling (scoped to the
  change — it never nags about legacy file size) — caught before they land, not after.
- **Security in two layers.** `gitleaks` (committed secrets) and dependency/vulnerability
  audits run mechanically in pre-commit and CI; for changes touching auth, untrusted input,
  secrets, or network I/O, the contract routes the agent to a `/security-review` for the
  logic-level flaws scanners can't see.
- **Docs, tests, and decisions can't silently drift.** `commit-msg` hooks require an RFC
  for a structural change, an architecture-doc update for a new component, and a test
  alongside a source change — each waivable with a logged trailer; RFCs auto-file into
  their lifecycle folder.
- **Every rule, enforced three ways.** The same checks run at **pre-commit**, on the
  **command surface**, and in **CI** — so a violation can't slip past whichever layer the
  agent skips.
- **…and the small operational foot-guns.** Followable background processes (no silent
  buffering), re-staging after `git add`, verifying CI after a workflow change — the
  reminders that turn a frustrating session into a smooth one.

## Philosophy

Three pillars, lifted from real production setups:

1. **Context** — one canonical contract (`AGENTS.md` → `INSTRUCTION.md`, plus `docs/` and
   RFCs) so intent is discoverable and never forks across tools.
2. **Mechanical enforcement** — linters, hooks, and CI catch violations
   automatically; error messages tell you how to fix them.
3. **Feedback loops** — subagents, tests, and reviews compound quality.

Every scaffolded project ships the **four execution principles**: think before
coding, simplicity first, surgical changes, goal-driven execution.

## Use it

**One command, no install.** This profile is the default, so you don't even name it:

```bash
brew install uv   # if you don't have uv yet — docs.astral.sh/uv
uvx --from git+https://github.com/luca-mastrostefano/agent-native-setup agent-native-setup -o ./my-app
```

To pick it explicitly, or to pin a release:

```bash
uvx --from git+https://github.com/luca-mastrostefano/agent-native-setup agent-native-setup \
  -o ./my-app --profile git+https://github.com/luca-mastrostefano/agent-native-baseline.git@v0.2.1
```

`profile validate` classifies this profile as **unsafe** — it ships pre-commit hooks and
onboarding steps that run on your machine — so the wizard shows what it will do and asks
you to approve before writing anything (a headless `-y` run needs `--allow-code`).

Prefer a persistent command? `uv tool install git+https://github.com/luca-mastrostefano/agent-native-setup`
(or `pipx install …`). The engine is not on PyPI, so `pip install agent-native-setup` will not work.

Its prompts (languages, AI tools, which parts to scaffold, runner, adoption strategy, …)
are the wizard's questions; headless runs use `-y` and `--answer name=value`.

## Extend it

To build "the baseline plus our house files", **fork this repo**, add your templates, and
publish your fork to the community index:

```bash
gh repo fork luca-mastrostefano/agent-native-baseline --clone my-team-profile
cd my-team-profile
# add files under templates/, set your own name/version in profile.json, tag, publish
git fetch upstream && git merge upstream/main   # later: take baseline improvements
```

Your consumers get each release through the normal `agent-native-setup update` flow.

## Layout

- `profile.json` — identity, prompts, `seed` (write-once files), `transient` (self-deleting
  first-run files), `empty_files`, `links` (the per-tool `CLAUDE.md`/`GEMINI.md` symlinks).
- `templates/` — the shipped files; `.j2` renders against `answers.<name>` + the sensed
  `env.<name>` facts, everything else ships verbatim. `@DATE@` in a path becomes the
  scaffold date.

Full format reference: [`docs/architecture/profiles.md`](https://github.com/luca-mastrostefano/agent-native-setup/blob/main/docs/architecture/profiles.md).

## Releasing (maintainers)

Until the engine's legacy generators are deleted (RFC 2026-07-05 stage D), the templates
here are **derived** — `profiles/agent-native-baseline/build.py` in the engine repo emits
them from the generators' own constants, and the engine's parity gate asserts whole-tree
byte equality. Don't hand-edit a built template; change it at the source and rebuild.
A release is: build → commit here → tag `vX.Y.Z` (matching `profile.json`) → update the
engine's `profiles/baseline-pin.json` (tag + content hash) and its vendored copy in the
same PR — CI verifies the pin matches this repo's tagged artifact. Release tags are
**immutable** (GitHub ruleset): a bad release gets a new tag and a pin bump, never a
moved tag.

## License

[MIT](./LICENSE) — fork it, ship it, make it yours.
