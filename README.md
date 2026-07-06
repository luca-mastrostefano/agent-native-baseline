# agent-native-baseline

The flagship [agent-native-setup](https://github.com/luca-mastrostefano/agent-native-setup)
profile: a complete, agent-native project setup —

- a canonical **`AGENTS.md`** contract for coding agents and humans (with `CLAUDE.md` /
  `GEMINI.md` links for tools that load their own file),
- **docs + RFCs** (`docs/architecture/`, a `proposed/ → active/` RFC lifecycle, embedded
  gate scripts),
- **linters, pre-commit hooks, secret + dependency scanning**, per-language configs
  (Python / Node / Go / Rust / HTML), a Make or Task command surface,
- **CI** (GitHub Actions quality + security gates),
- a **`.claude/`** agents & commands library and a self-deleting first-run
  `ONBOARDING.md`.

This is what `agent-native-setup -o ./my-app` scaffolds by default — the engine vendors a
hash-pinned copy of a tagged release of this repo, so scaffolding is offline and the
artifact you get is exactly the one reviewed here.

## Use it

```bash
pip install agent-native-setup
agent-native-setup my-app -o ./my-app          # this profile is the default
# or explicitly / pinned:
agent-native-setup my-app -o ./my-app --profile git+https://github.com/luca-mastrostefano/agent-native-baseline.git@v0.1.0
```

Its prompts (languages, AI tools, which parts to scaffold, runner, adoption strategy, …)
are the wizard's questions; headless runs use `-y` and `--answer name=value`.

## Extend it — fork, don't overlay

There is deliberately no `extends` mechanism: to build "the baseline plus our house
files", **fork this repo** and add your templates —

```bash
gh repo fork luca-mastrostefano/agent-native-baseline --clone my-team-profile
cd my-team-profile
# add files under templates/, set your own name/version in profile.json, tag, publish
git fetch upstream && git merge upstream/main   # later: take baseline improvements
```

Git's three-way merge handles shared-file changes an overlay never could, and you review
upstream changes before releasing them to your own consumers (who get each release through
the normal `agent-native-setup update` flow).

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
same PR — CI verifies the pin matches this repo's tagged artifact.

## License

[MIT](./LICENSE) — fork it, ship it, make it yours.
