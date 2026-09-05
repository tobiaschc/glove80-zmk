---
name: keymap-commit
description: Write and create git commit messages for glove80.keymap changes in this repo's dated-changelog format (YYMMDD - vX.Y header + type(scope) bullet list). Use whenever committing changes to config/glove80.keymap or other files under config/.
---

# Keymap commit messages

This repo's `config/glove80.keymap` history uses a changelog-style commit
message instead of a normal conventional-commit subject line. Every commit
that touches keymap/config files must follow this exact shape:

```
YYMMDD - vX.Y
- type(scope): description
- type(scope): description
```

Real examples from this repo's history:

```
250915 - v2.4
- feat(base, base_win): add &MO_KP on thumb cluster
- refactor(base, base_win): reduce all tapping term: 20ms

250827 - v2.3
- switch(base, base_win/&HRM): (SHIFT, ALT, GUI, CTRL) -> (ALT, SHIFT, GUI, CTRL)

250824 - v2.2
- feat(layer): add base_win layer and map current mac macros
- refactor(base_win/&HRM): set RALT on both ALT keys, due to accent win requirements
- refactor(ALL): (RH C5R1, mac-layer0), (RH C6R1, win-layer1), (RH C5R2, basic-layer4), (RH C5R2, symbol-layer2)* [on symbol go to mac]
- add(base, base_win/&kp): (LH C4R2, #), (LH C2R2, @), (RH C2R2, ~)

250802 - v2.1
- refactor(base/&HRM): increase tapping term: 220/250
```

Each block above is one full commit message (a header line plus one or more
bullets for everything that commit changed). When asked to commit keymap
changes, produce ONE such block for the current diff — do not reproduce the
whole history.

## Steps

1. **Get the diff.** Run `git diff` / `git diff --staged` against
   `config/glove80.keymap` (and any other changed files under `config/`) to
   see exactly what changed. If nothing is staged, stage the relevant files
   first (ask the user before staging anything unexpected).

2. **Determine the date.** Use today's date, formatted `YYMMDD` (2-digit
   year, month, day — e.g. 2026-09-05 → `260905`).

3. **Determine the next version.** Run
   `git log --grep='^[0-9]\{6\} - v[0-9]' --pretty=%B -n 20` (or just
   `git log -20 --pretty=%B`) and find the most recent `vX.Y` used. Bump the
   minor number by 1 (`v2.4` → `v2.5`). If none is found anywhere in history,
   start at `v1.0`. Never reuse or go backwards from the highest version seen.

4. **Identify scopes from the diff, not from memory.** The `scope` in
   `type(scope):` is almost always one or more *layer names*. Determine the
   current layer names by reading the keymap's layer nodes/defines directly
   (e.g. `grep -n "LAYER_\|layer_.* {" config/glove80.keymap`) — do not assume
   old names like `base`/`base_win`/`Magic`/`BASIC` still apply; use whatever
   the file currently defines (this repo has renamed layers before, e.g. to
   `unix`, `windows`, `symbol`, `magic`, `basic`).
   - If a change touches one layer's bindings block, scope is that layer name:
     `(unix)`.
   - If the same logical change was applied identically to multiple layers,
     combine them: `(unix, windows)`.
   - If a change is to a *behavior* used within a layer (e.g. a home-row-mod
     behavior, a macro, a specific key), append it after a slash:
     `(base_win/&HRM)`, `(unix/&kp)`.
   - If a change applies uniformly across every layer, use `(ALL)`.
   - If a change adds/removes a whole layer (not just bindings in it), scope
     is the bare word `layer`: `feat(layer): add ...`.
   - If a change is to shared infra outside any single layer (behaviors {},
     macros {}, combos {} blocks, dtsi/config files, etc.) and isn't
     layer-specific, pick the most descriptive scope available (behavior
     name, macro name, or file name) rather than forcing a layer name.

5. **Pick a type per bullet**, conventional-commit style, matching this
   project's observed vocabulary:
   - `feat` — new capability (new layer, new binding for a previously-empty
     key, new behavior).
   - `add` — a smaller additive change, typically adding specific key
     mappings (see the `add(base, base_win/&kp): (LH C4R2, #), ...` example).
   - `refactor` — behavior-preserving restructuring (tapping-term tweaks,
     moving where a shortcut lives, reorganizing without changing net
     capability).
   - `switch` — swapping the order/assignment of an existing set of things
     (e.g. reordering which modifier sits on which home-row-mod finger).
   - `fix` — correcting a bug in an existing binding/behavior.
   - `chore` — non-functional changes (comments, formatting, renames that
     don't change behavior).
   Use one bullet per distinct logical change; do not combine unrelated
   changes into one bullet, and do not split one change into multiple bullets.

6. **Write bullet descriptions tersely**, matching the terse, abbreviation-
   heavy style of the examples (e.g. `reduce all tapping term: 20ms`,
   `(LH C4R2, #)` for "left-hand column 4 row 2 now types #"). Prefer physical
   key coordinates (`LH`/`RH`, `C<col>R<row>`) when describing individual key
   moves, as the history does.

7. **Assemble the commit message**:
   ```
   YYMMDD - vX.Y
   - type(scope): description
   - type(scope): description
   ```
   Then append whatever attribution footer this session's standing
   instructions require (e.g. `Co-Authored-By:` trailer) as a separate
   trailing block, exactly as those instructions specify — it is not part of
   the changelog format itself.

8. **Create the commit** with `git commit -F -` via a heredoc (so the blank
   line and bullet formatting survive exactly), never `-m` with embedded
   `\n`. Show the assembled message to the user before committing if there is
   any ambiguity in scope/type/version.

## Notes

- Only use this format for commits touching keymap/config files in this
  repo. Unrelated commits (e.g. tooling, docs, this skill file itself) should
  use normal conventional-commit subject lines instead.
- Version numbers are per-repo and monotonic — always derive the next one
  from git history, never guess or hardcode.
