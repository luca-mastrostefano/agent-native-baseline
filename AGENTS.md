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

## Lifecycle: cutting a release

A release **is** its git tag plus the `content_hash` over `profile.json` + `templates/` that the
[community index][idx] pins. The index stores **no version field** — the tag in its `url`
(`…@vX.Y.Z`) is the only version an adopter can ever see.

The loop, every time you change `profile.json` or anything under `templates/`:

```bash
agent-native-setup profile validate .   # zero errors; resolve every advisory ⚠
# bump `version` in profile.json — pre-1.0, a minor bump counts as breaking (adopters' `update` pauses)
git commit -am "…" && git tag v<version> && git push --follow-tags
agent-native-setup profile publish . --release   # attaches the release asset, refreshes the index entry
```

**Never re-use a version; never move a tag.** A pinned tag is cached forever on an adopter's
machine and never re-fetched, so re-tagging the same version reaches nobody who already has it —
while the moved bytes stop matching the hash the index vouched for, and every *new* install by
name is refused (*"no longer matches the hash vetted in the community index"*). Nothing enforces
this for you: `publish` never checks that you bumped. Bumping `version` rewrites the hash by
itself (the hash covers `profile.json`), so **the tag and the index listing must always move
together** — publish every change you ship, and ship every change you publish.

`publish` hashes your **working tree**, while the release asset is built from the **tag's
committed tree**. `git status --porcelain profile.json templates/` must be empty before you
publish, or the `content_hash` you list won't match the artifact adopters download.

This profile carries a **second pin**: the engine vendors a copy and verifies it against
`profiles/baseline-pin.json` (tag + content hash). That pin and the community-index entry must
both move in the same release — see the "Releases are tagged and pinned" rule above.

Adopters who scaffolded with the **default** (`builtin:agent-native-baseline`, i.e. no
`--profile`) recorded `builtin:` as their `source`, so they pick up a new baseline by upgrading
the engine itself (`uv tool upgrade agent-native-setup`), then running `update`. Only those who
installed it **by name** need the `profile add` dance below.

## How adopters pick up a new version

**Not with `update` alone.** `agent-native-setup update` re-resolves the `source` recorded in the
adopter's project; for a name install that is their own copy under
`~/.config/agent-native-setup/profiles/agent-native-baseline`, which never re-consults the index. And
`profile add` refuses to overwrite an existing copy. So the real upgrade is:

```bash
rm -rf ~/.config/agent-native-setup/profiles/agent-native-baseline \
  && agent-native-setup profile add agent-native-baseline && agent-native-setup update
```

Say this in your release notes: an adopter who only runs `update` sees nothing.

[idx]: https://github.com/luca-mastrostefano/agent-native-setup/blob/main/contributions/index.json
