# Contributing to prompt-atlas

Thanks for considering a contribution. prompt-atlas is built on the **matrix-citation method** — every claim about model behavior should be auditable from a documented source. Contributions follow that standard.

---

## What kinds of contributions are welcome

1. **New model row** — a vendor released a new model and you want to add coverage
2. **New vendor family** — a vendor not currently covered (e.g., new entrant; Meta Muse Spark as data matures)
3. **Updated cell** — a vendor changed a documented behavior (e.g., Gemini 3.5 Flash `thinking_level` default shift) and the current cell is now wrong
4. **New anti-pattern** — you observed a reproducible failure mode and have measurement evidence
5. **New technique snippet** — a wording pattern you've validated against a real workload
6. **New agentic system** — a tool (Cursor, Aider, Windsurf, etc.) you want covered
7. **Multilingual finding** — non-EN behavior worth documenting
8. **Fact corrections** — citations that point at the wrong source, version numbers that drifted, dates that need updating

## What is NOT in scope

- Personal anecdotes without citation ("I tried X and it didn't work for me")
- API parameter values (`temperature`, `max_tokens`, etc.) — those belong to vendor docs, not this skill
- Code that wraps the skill (SDK clients, scripts that call models) — separate projects
- LoRA / SFT training data design — separate domain

---

## How to add a new model row

1. **Add a row to `references/matrix.md`** in all five tables (A through E). Use `?` for axes you can't document; `?` is honest, guesses are dangerous.

2. **Create a vendor model file** `references/models/<vendor>.md` if the quirks exceed one cell per table. Follow the layout of existing files (`kimi.md`, `glm.md`, `deepseek.md` are good references for structure):
   - Family-wide rules (the cross-version invariants)
   - Per-version subsections
   - Cross-vendor rules (when this vendor is one of several targets)
   - **Source notes** at the bottom — cite vendor docs, model cards, benchmark publications, independent practitioner guides

3. **Update `SKILL.md`**:
   - **Step 2b** — add vendor signals so the workflow recognizes the new target
   - **Step 2c** — add model version options
   - **Step 3** — add the new model file path to the vendor-specific model file list

4. **Update `references/models/_universal.md`** if the new vendor changes the cross-vendor compromise (e.g., new opposite-default axis vs Claude / GPT / Gemini).

5. **Update `CHANGELOG.md`** with the date, vendor, and what changed.

6. **Update `README.md`** "Coverage" table to include the new vendor.

---

## How to update an existing cell

Vendor changes happen. When a vendor publishes new guidance that contradicts the current matrix:

1. **Verify with multiple sources** if possible — vendor blog + independent practitioner guide is the standard
2. **Update the matrix cell** with the new behavior
3. **Update the corresponding model file** with version-specific notes (don't lose the history — note "as of v.X, behavior changed from Y to Z")
4. **If the change is large** (e.g., the Gemini 3.5 Flash `thinking_level` default drop), surface it in `CHANGELOG.md` with a `Migration note:` for prompts that depended on the old behavior

---

## How to add an anti-pattern

Anti-patterns require **reproducible measurement**:

1. Concrete failure example (input → wrong output)
2. Measured regression size when applicable (e.g., "−15 pp on test set X")
3. Cited workaround or fix

Anti-patterns from theory ("this probably hurts") don't go in unless they're supported by published vendor guidance.

For small-model anti-patterns (`antipatterns-small.md`), include the test suite or evaluation framework that reproduced the regression — those are uniquely valuable because they're harder to publish in vendor docs.

---

## Citation standards

Every claim should be traceable:

- **Vendor doc URL** when available (model card, release notes, prompting guide)
- **Independent practitioner guide URL** when vendor docs are thin
- **Benchmark publication** for performance claims
- **Date** of the source — vendor docs change

When a citation is to a transient source (vendor blog that may be edited later), include the access date and key quoted phrases so the claim survives source drift.

---

## Pull request template

When opening a PR:

```
## Type of change
[ ] New model row
[ ] New vendor family
[ ] Updated cell (cite the source + date)
[ ] New anti-pattern (include reproducible measurement)
[ ] New technique
[ ] Fact correction
[ ] Other

## Files touched
- references/matrix.md (rows added)
- references/models/<vendor>.md (new file or updates)
- SKILL.md (Step 2b/2c/3)
- CHANGELOG.md
- README.md (if Coverage table changed)

## Source citations
- <vendor docs URL or independent source>
- <additional sources>

## Verification
[ ] Matrix row uses `?` for unverified axes (no guesses)
[ ] All sources are cited in the new/updated file's "Source notes" section
[ ] CHANGELOG.md updated
[ ] README.md Coverage table updated if applicable
```

---

## Style notes

- **Prose over bullets** in vendor model files — readers need narrative context for prompting guidance
- **One claim per cell** in the matrix — multi-claim cells become unreadable
- **Markdown over XML** for skeleton — XML inline only for examples
- **Honest `?` over guess** — readers downstream will trust the file; a `?` lets them know to verify
- **No model name pinning** in the methodology layer — `principles.md` and `techniques.md` should be vendor-neutral
- **Cite the model file from the matrix row reading guide** when a vendor has a quirk worth noting at the matrix level

---

## License

By contributing, you agree your contribution is licensed under the same [CC-BY-SA 4.0](LICENSE) as the rest of the project. ShareAlike means derivative works must remain open under the same license — this protects the curated vendor-behavior data from closed-source forks.
