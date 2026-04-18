# Autoebiten Integration — Design Specification

> **Created**: 2026-04-18
> **Status**: Approved for implementation
> **Scope**: Add Autoebiten automation support via skill routing (no new agent, no redundant docs)

## Overview

Add Autoebiten (Playwright-like automation for Ebitengine) support to Claude Code Game Studios via minimal coordination skill modifications. Autoebiten provides input injection, screenshot capture, testkit, and autoui for automated gameplay testing.

**Key decision:** Skill routing, not agent hierarchy. The `using-autoebiten` skill (global user skill) already contains complete reference documentation. CCGS coordination skills route to it when engine is Ebitengine.

## Design Rationale

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Agent type | None (skill routing) | using-autoebiten skill already exists as global user skill; no duplication needed |
| Documentation | None (reference existing) | Skill has complete CLI/testkit/autoui docs; engine reference would be redundant |
| Coordination | Modify existing skills | test-setup, qa-plan, team-qa detect Ebitengine → route to using-autoebiten skill |
| Integration point | Cross-references only | game-loop.md and ui.md add single-line references to the skill |

## Integration Architecture

```
test-setup / qa-plan / team-qa
    ↓ (detect engine = Ebitengine)
    ↓
using-autoebiten skill (global user skill)
    ↓ (provides CLI, testkit, autoui patterns)
    ↓
tests/ directory (testkit tests)
```

**Flow:**
1. Coordination skill reads engine config from `technical-preferences.md`
2. If engine = Ebitengine/Ebiten, skill adds routing note to output
3. User invokes `using-autoebiten` skill for automation patterns
4. Skill provides CLI commands, testkit API, autoui queries

## Skill Modifications

### test-setup/SKILL.md

**Phase 3 (Engine-specific files)** — Add Ebitengine section:

```markdown
#### Ebitengine (`Engine: Ebitengine` or `Engine: Ebiten`)

Note in the README: **Ebitengine Automation**
```
For automated gameplay testing, use the `using-autoebiten` skill (global user skill).
Autoebiten provides input injection, screenshot capture, and testkit for black-box testing.
No test runner scaffold needed — Autoebiten handles game control via RPC.
```
```

No CI workflow scaffold for Ebitengine (Autoebiten is CLI-based, CI uses `go test` for testkit tests).

---

### qa-plan/SKILL.md

**Phase 2 (Load Inputs)** — After reading engine config:

If engine is Ebitengine, add to plan output:

```markdown
**Automation for Ebitengine**: Invoke `using-autoebiten` skill for:
- CLI automation patterns (input injection, screenshots)
- Testkit integration (Launch, Mock, StateQuery)
- Autoui for EbitenUI widget querying
```

---

### team-qa/SKILL.md

**Phase 2 (QA Strategy)** — After engine detection:

If engine is Ebitengine:

```markdown
**Ebitengine Automation**: Use `using-autoebiten` skill for black-box test patterns.
For Integration stories without automated tests, spawn a subagent with the using-autoebiten skill context to write testkit tests.
```

---

## Cross-References (Existing Docs)

### game-loop.md

Add to end:

```markdown
## Automated Testing

For automated gameplay verification with Autoebiten, invoke the `using-autoebiten` skill (global user skill). Autoebiten provides:
- Input injection via CLI
- Screenshot capture for visual verification
- Testkit for tick-based assertions
- RPC control of running games

See the skill's reference docs for integration patterns.
```

---

### ui.md

Add to end:

```markdown
## Widget Testing

For automated EbitenUI widget testing, invoke the `using-autoebiten` skill. Autoui provides:
- Widget querying by coordinates, attributes, or XPath
- Widget method invocation (Click, SetText)
- Integration with testkit tests

See the skill's autoui reference docs for query patterns.
```

---

## Test Specifications

### Location

Update existing test specs:
- `CCGS Skill Testing Framework/skills/utility/test-setup.md` — add Case 6
- `CCGS Skill Testing Framework/skills/utility/qa-plan.md` — add Case 6
- `CCGS Skill Testing Framework/skills/team/team-qa.md` — add Case 6

### Test Cases

#### test-setup.md — Case 6: Ebitengine project, routes to using-autoebiten

**Fixture:**
- `technical-preferences.md` has engine set to Ebitengine/Ebiten, language Go
- `tests/` directory does not exist

**Input:** `/test-setup`

**Expected behavior:**
1. Skill reads engine → Ebitengine
2. Skill does NOT create GdUnit4/Unity/Unreal scaffold
3. Skill reports: "Ebitengine project — use `using-autoebiten` skill for automation setup (input injection, testkit, autoui). No test runner scaffold needed."
4. Skill may create `tests/` directory structure (unit/, integration/) for testkit test placement
5. Verdict is COMPLETE

**Assertions:**
- [ ] Engine correctly detected as Ebitengine/Ebiten
- [ ] `using-autoebiten` skill is explicitly mentioned as the automation tool
- [ ] No GdUnit4, Unity asmdef, or Unreal runner config is created
- [ ] Verdict is COMPLETE

---

#### qa-plan.md — Case 6: Ebitengine project, routing note added

**Fixture:**
- `technical-preferences.md` has engine set to Ebitengine
- Sprint with 3 stories: 1 Logic, 1 Integration, 1 UI
- `coding-standards.md` present

**Input:** `/qa-plan sprint-005`

**Expected behavior:**
1. Skill reads engine config → Ebitengine
2. Skill reads stories and assigns test types
3. QA plan includes routing section: "**Automation for Ebitengine**: Invoke `using-autoebiten` skill for CLI automation, testkit integration, and autoui"
4. Integration story test recommendation mentions testkit patterns
5. Plan written after approval

**Assertions:**
- [ ] Ebitengine routing note appears in QA plan output
- [ ] `using-autoebiten` skill is explicitly named
- [ ] Integration story test recommendation aligns with testkit patterns
- [ ] Verdict is COMPLETE

---

#### team-qa.md — Case 6: Ebitengine project, Phase 2 routes to skill

**Fixture:**
- `technical-preferences.md` has engine set to Ebitengine
- Sprint with 2 stories: 1 Logic (automated test passes), 1 Integration (needs automation)
- Smoke check passes

**Input:** `/team-qa sprint-08`

**Expected behavior:**
1. Phase 1: Reads engine config → Ebitengine
2. Phase 2: Strategy includes routing note: "Ebitengine Automation — use `using-autoebiten` skill for Integration story testkit test"
3. Phase 2: User approves strategy
4. Phase 3: QA plan includes Ebitengine routing section
5. Phase 5: Integration story test case writing references testkit patterns (not manual-only)
6. Phase 7: Sign-off report shows Integration story covered via automation

**Assertions:**
- [ ] Phase 2 strategy explicitly mentions `using-autoebiten` skill for Ebitengine
- [ ] Integration story receives automation recommendation, not manual-only
- [ ] QA plan includes Ebitengine routing section
- [ ] Verdict: COMPLETE

---

## File Summary

| Type | Count | Files |
|------|-------|-------|
| Create | 0 | No new agent, no new engine reference module |
| Modify | 5 | test-setup/SKILL.md, qa-plan/SKILL.md, team-qa/SKILL.md, game-loop.md, ui.md |
| Test Spec Update | 3 | test-setup.md, qa-plan.md, team-qa.md (add Case 6 for Ebitengine routing) |

---

## Exclusions

- NOT included: CLI command reference, testkit API signatures, autoui XML format
- NOT included: ebiten-autoebiten-specialist agent
- NOT included: testing.md engine reference module (redundant with skill docs)
- Focus only: Routing coordination, cross-references

---

## Source Reference

- Autoebiten skill: `~/.claude/skills/using-autoebiten/SKILL.md`
- Autoebiten source: `~/Desktop/go-autoebiten/` (if available)
- Key skill references: integration.md, testkit.md, autoui-reference.md, commands.md