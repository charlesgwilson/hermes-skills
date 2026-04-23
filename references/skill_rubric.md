# Hermes Skill Rubric

Score a skill 0–100 before committing. Run this after the smoke test passes.

## Categories

| Category | Weight | What it measures |
|----------|--------|-----------------|
| Frontmatter | 25% | Required and appropriate optional fields present |
| Structure | 25% | Correct sections, correct order, no anti-patterns |
| Content accuracy | 30% | Verified examples, tested pitfalls, passing smoke test |
| Size discipline | 20% | Line count in range, references used for bulk content |

---

## Frontmatter (25 points)

Start at 25.

| Deduction | Condition |
|-----------|-----------|
| −5 | Any of the 6 required fields missing (`name`, `description`, `version`, `author`, `license`, `metadata.hermes.tags`) |
| −5 | Tool requires a CLI binary but `prerequisites.commands` is absent |
| −3 | Tool requires an env var but `prerequisites.env_vars` is absent |
| −3 | macOS-only tool missing `platforms: [macos]` |
| −5 | Description embeds routing instructions ("Load this skill when…") instead of a tool summary |
| −3 | `related_skills` and `homepage` both absent on a community skill (one is required) |

---

## Structure (25 points)

Start at 25.

| Deduction | Condition |
|-----------|-----------|
| −5 | `Prerequisites` section absent |
| −5 | `Rules for Hermes Agents` section absent |
| −5 | `Verification` section absent |
| −5 | `Pitfalls & Gotchas` section absent |
| −5 | Skill takes actions on behalf of the user but has no `When to Use` / `When NOT to Use` sections |
| −3 | Operations use "Mode N:" naming for general task categorization (not reserved for genuine dual-mode tools) |
| −3 | Section order does not follow the convention: Prerequisites → When to Use → When NOT to Use → Operations → CLI Reference → Pitfalls → Rules → Verification → References |
| −3 | `README.md` skills table row not added |

---

## Content Accuracy (30 points)

Start at 30.

| Deduction | Condition |
|-----------|-----------|
| −10 | Smoke test command fails or does not produce the stated success marker |
| −5 per flag | CLI flag or subcommand not found in `<tool> --help` or source (invented flag) |
| −5 | Pitfalls section contains only assumed gotchas — none confirmed by testing |
| −3 per item | Pitfall uses "may fail" / "might error" language without evidence of observed failure |
| −5 | Code examples contain pseudo-code or non-runnable syntax |
| −3 | JSON shape documented inline where field names are guessable (over-documentation) |
| −3 | Rules section contains restated defaults ("Use correct syntax", "Check tool is installed") |

---

## Size Discipline (20 points)

Start at 20.

| Deduction | Condition |
|-----------|-----------|
| −10 | Skill exceeds its ceiling without justification (see targets below) |
| −5 | Skill is between target and ceiling — extract candidates to `references/` |
| −3 | Reference file exceeds 250 lines |
| −5 | Substantial domain knowledge embedded in SKILL.md that belongs in `references/` |

**Size targets:**

| Skill type | Target | Ceiling |
|------------|--------|---------|
| Single-purpose / read-only | ~100 lines | 150 lines |
| Standard personal assistant | ~150–250 lines | 300 lines |
| Complex multi-mode tool | ~300–400 lines | 500 lines |
| Reference file | ~100–200 lines | 250 lines |

---

## Score Output

```
## Skill Audit: <skill-name> v<version>

| Category (weight)        | Score | Deductions |
|--------------------------|-------|------------|
| Frontmatter (25%)        | __/25 | |
| Structure (25%)          | __/25 | |
| Content accuracy (30%)   | __/30 | |
| Size discipline (20%)    | __/20 | |

**Total: __/100**

Minimum to commit: 80/100
Blocking issues (any score of 0 in a category): must fix before commit.
```

### Score thresholds

| Score | Action |
|-------|--------|
| 90–100 | Commit as-is |
| 80–89 | Commit with noted improvements for next version |
| 70–79 | Fix deductions before committing |
| < 70 | Do not commit — requires rework |

---

## Over-Engineering Signals

Flag these during the size pass. Each is grounds for a deduction or a reference extraction.

| Signal | Example | Fix |
|--------|---------|-----|
| Mode N: overuse | "Mode 1: List", "Mode 2: Read", "Mode 3: Search" for a single-mode tool | Rename to natural task headers |
| Inline JSON for obvious shapes | Documenting `{"id": "...", "name": "..."}` for a list endpoint | Remove — agent can infer |
| Rules restating examples | Rule says "use --json" and every example already uses --json | Remove rule |
| Pitfall without evidence | "Flag X might not work in some cases" | Test and confirm, or remove |
| Description as instructions | "Load this skill immediately when the user mentions..." | Rewrite as tool summary |
| Verbose pitfall narrative | 3-sentence pitfall that explains history and workaround | Cut to: what breaks, what to do instead |
