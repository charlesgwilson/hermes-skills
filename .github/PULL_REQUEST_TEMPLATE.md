## Skill

**Name:** <!-- skill name from SKILL.md frontmatter -->
**Path:** `skills/<name>/SKILL.md`

## Summary

<!-- What does this skill do? What CLI tool or agent does it orchestrate? -->

## Verification

<!-- Paste the output of the smoke test from the Verification section of SKILL.md -->

```
$ <smoke test command>
<output>
```

## Checklist

- [ ] SKILL.md frontmatter has `name`, `description`, `version`, `author`, `license`, `metadata.hermes.tags`
- [ ] All CLI flags verified against `--help` or official docs (no invented APIs)
- [ ] Pitfalls section present and accurate
- [ ] Rules for Hermes Agents section present
- [ ] Verification / smoke test passes
- [ ] README.md Skills table updated
