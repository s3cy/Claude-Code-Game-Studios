# Autoebiten Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Autoebiten routing to coordination skills and cross-references to engine reference docs.

**Architecture:** Minimal modifications — skill routing logic detects Ebitengine engine config and adds routing notes to output. No new agent, no redundant documentation. Cross-references added to existing engine reference modules.

**Tech Stack:** Markdown documentation, skill frontmatter, CCGS skill testing framework

---

## File Structure

| File | Responsibility | Change Type |
|------|----------------|-------------|
| `.claude/skills/test-setup/SKILL.md` | Ebitengine section in Phase 3 | Modify (append section) |
| `.claude/skills/qa-plan/SKILL.md` | Engine detection + routing in Phase 2 | Modify (insert logic) |
| `.claude/skills/team-qa/SKILL.md` | Engine detection + routing in Phase 2 | Modify (insert logic) |
| `docs/engine-reference/ebiten/modules/game-loop.md` | Automated Testing section | Modify (append section) |
| `docs/engine-reference/ebiten/modules/ui.md` | Widget Testing section | Modify (append section) |
| `CCGS Skill Testing Framework/skills/utility/test-setup.md` | Case 6 test case | Modify (append case) |
| `CCGS Skill Testing Framework/skills/utility/qa-plan.md` | Case 6 test case | Modify (append case) |
| `CCGS Skill Testing Framework/skills/team/team-qa.md` | Case 6 test case | Modify (append case) |

---

### Task 1: Add Ebitengine Section to test-setup/SKILL.md

**Files:**
- Modify: `.claude/skills/test-setup/SKILL.md` (insert after Unreal section in Phase 3)

- [ ] **Step 1: Read test-setup/SKILL.md to find insertion point**

Read the file and locate the Unreal Engine section in Phase 3. The Ebitengine section should be added after Unreal, before Phase 4.

- [ ] **Step 2: Insert Ebitengine section**

Insert after the Unreal section (approximately line 205):

```markdown
#### Ebitengine (`Engine: Ebitengine` or `Engine: Ebiten`)

Note in the README: **Ebitengine Automation**
```
For automated gameplay testing, use the `using-autoebiten` skill (global user skill).
Autoebiten provides input injection, screenshot capture, and testkit for black-box testing.
No test runner scaffold needed — Autoebiten handles game control via RPC.
```

No CI workflow scaffold for Ebitengine. Autoebiten is CLI-based; CI uses `go test` for testkit tests.
```

- [ ] **Step 3: Verify insertion**

Read the modified section to confirm the Ebitengine section appears after Unreal and before Phase 4.

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/test-setup/SKILL.md
git commit -m "feat: add Ebitengine routing to test-setup skill"
```

---

### Task 2: Add Ebitengine Routing to qa-plan/SKILL.md

**Files:**
- Modify: `.claude/skills/qa-plan/SKILL.md` (insert in Phase 2 after engine detection)

- [ ] **Step 1: Read qa-plan/SKILL.md to find insertion point**

Read Phase 2 (Load Inputs) section. After reading engine config from `technical-preferences.md`, insert Ebitengine detection logic.

- [ ] **Step 2: Insert engine detection and routing logic**

Insert after the supporting context loading in Phase 2 (approximately line 75):

```markdown
If engine is **Ebitengine** or **Ebiten**, add to the plan output:

```markdown
**Automation for Ebitengine**: Invoke `using-autoebiten` skill for:
- CLI automation patterns (input injection, screenshots)
- Testkit integration (Launch, Mock, StateQuery)
- Autoui for EbitenUI widget querying
```
```

- [ ] **Step 3: Verify insertion**

Read the modified Phase 2 section to confirm Ebitengine detection appears after engine config reading and before Phase 3.

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/qa-plan/SKILL.md
git commit -m "feat: add Ebitengine routing to qa-plan skill"
```

---

### Task 3: Add Ebitengine Routing to team-qa/SKILL.md

**Files:**
- Modify: `.claude/skills/team-qa/SKILL.md` (insert in Phase 2)

- [ ] **Step 1: Read team-qa/SKILL.md to find insertion point**

Read Phase 2 (QA Strategy) section. After engine detection from `technical-preferences.md`, insert Ebitengine routing logic.

- [ ] **Step 2: Insert engine detection and routing logic**

Insert in Phase 2 after the scope detection (approximately line 35):

```markdown
Read `.claude/docs/technical-preferences.md` to detect the configured engine.

If engine is **Ebitengine** or **Ebiten**, include in the strategy output:

```markdown
**Ebitengine Automation**: Use `using-autoebiten` skill for black-box test patterns.
For Integration stories without automated tests, spawn a subagent with the using-autoebiten skill context to write testkit tests.
```
```

- [ ] **Step 3: Verify insertion**

Read the modified Phase 2 section to confirm Ebitengine detection appears at the start of Phase 2.

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/team-qa/SKILL.md
git commit -m "feat: add Ebitengine routing to team-qa skill"
```

---

### Task 4: Add Automated Testing Section to game-loop.md

**Files:**
- Modify: `docs/engine-reference/ebiten/modules/game-loop.md` (append to end)

- [ ] **Step 1: Read game-loop.md to find end point**

Read the final section (Reference) to determine where to append the new section.

- [ ] **Step 2: Append Automated Testing section**

Append after the Reference section:

```markdown
---

## Automated Testing

For automated gameplay verification with Autoebiten, invoke the `using-autoebiten` skill (global user skill). Autoebiten provides:

- **Input injection via CLI** — Send keyboard, mouse, and custom inputs to running games
- **Screenshot capture** — Visual verification of game state
- **Testkit for tick-based assertions** — Go test framework integration with Launch, Mock, StateQuery
- **RPC control of running games** — External process control via JSON-RPC

See the skill's reference docs for integration patterns:
- `integration.md` — Patch vs Library methods
- `testkit.md` — Black-box vs White-box testing
- `commands.md` — CLI command reference
```

- [ ] **Step 3: Verify insertion**

Read the end of the file to confirm the new section appears after Reference.

- [ ] **Step 4: Commit**

```bash
git add docs/engine-reference/ebiten/modules/game-loop.md
git commit -m "docs: add Automated Testing section to game-loop.md"
```

---

### Task 5: Add Widget Testing Section to ui.md

**Files:**
- Modify: `docs/engine-reference/ebiten/modules/ui.md` (append to end)

- [ ] **Step 1: Read ui.md to find end point**

Read the final section (Reference) to determine where to append the new section.

- [ ] **Step 2: Append Widget Testing section**

Append after the Reference section:

```markdown
---

## Widget Testing

For automated EbitenUI widget testing, invoke the `using-autoebiten` skill. Autoui provides:

- **Widget querying by coordinates, attributes, or XPath** — Locate widgets without hardcoded positions
- **Widget method invocation** — Click, SetText, and other actions via CLI or testkit
- **Integration with testkit tests** — Combine widget interaction with tick-based assertions

See the skill's autoui reference docs:
- `autoui-commands.md` — CLI commands for widget interaction
- `autoui-reference.md` — XML format, ae tags, XPath queries
```

- [ ] **Step 3: Verify insertion**

Read the end of the file to confirm the new section appears after Reference.

- [ ] **Step 4: Commit**

```bash
git add docs/engine-reference/ebiten/modules/ui.md
git commit -m "docs: add Widget Testing section to ui.md"
```

---

### Task 6: Add Case 6 to test-setup Test Spec

**Files:**
- Modify: `CCGS Skill Testing Framework/skills/utility/test-setup.md` (append Case 6)

- [ ] **Step 1: Read test-setup.md test spec to find insertion point**

Locate the Protocol Compliance section. Case 6 should be added before Protocol Compliance.

- [ ] **Step 2: Append Case 6**

Insert before Protocol Compliance section:

```markdown
---

### Case 6: Ebitengine Project — Routes to using-autoebiten skill

**Fixture:**
- `technical-preferences.md` has engine set to Ebitengine or Ebiten, language Go
- `tests/` directory does not exist

**Input:** `/test-setup`

**Expected behavior:**
1. Skill reads engine from `technical-preferences.md` → Ebitengine or Ebiten
2. Skill does NOT create GdUnit4, Unity asmdef, or Unreal runner scaffold
3. Skill reports: "Ebitengine project — use `using-autoebiten` skill for automation setup (input injection, testkit, autoui). No test runner scaffold needed."
4. Skill may create `tests/` directory structure (unit/, integration/) for testkit test placement
5. Verdict is COMPLETE

**Assertions:**
- [ ] Engine correctly detected as Ebitengine or Ebiten
- [ ] `using-autoebiten` skill is explicitly mentioned as the automation tool
- [ ] No GdUnit4 runner script, Unity asmdef, or Unreal headless runner config is created
- [ ] Verdict is COMPLETE
```

- [ ] **Step 3: Verify insertion**

Read the test spec to confirm Case 6 appears before Protocol Compliance.

- [ ] **Step 4: Commit**

```bash
git add CCGS Skill Testing Framework/skills/utility/test-setup.md
git commit -m "test: add Ebitengine routing case to test-setup spec"
```

---

### Task 7: Add Case 6 to qa-plan Test Spec

**Files:**
- Modify: `CCGS Skill Testing Framework/skills/utility/qa-plan.md` (append Case 6)

- [ ] **Step 1: Read qa-plan.md test spec to find insertion point**

Locate the Protocol Compliance section. Case 6 should be added before Protocol Compliance.

- [ ] **Step 2: Append Case 6**

Insert before Protocol Compliance section:

```markdown
---

### Case 6: Ebitengine Project — Routing note added to QA plan

**Fixture:**
- `technical-preferences.md` has engine set to Ebitengine or Ebiten
- Sprint with 3 stories: 1 Logic, 1 Integration, 1 UI
- `coding-standards.md` present

**Input:** `/qa-plan sprint-005`

**Expected behavior:**
1. Skill reads engine config from `technical-preferences.md` → Ebitengine or Ebiten
2. Skill reads stories and assigns test types per coding-standards.md
3. QA plan includes routing section: "**Automation for Ebitengine**: Invoke `using-autoebiten` skill for CLI automation, testkit integration, and autoui"
4. Integration story test recommendation mentions testkit patterns
5. Skill asks "May I write to `production/qa/qa-plan-sprint-005.md`?"
6. Plan written on approval; verdict is COMPLETE

**Assertions:**
- [ ] Ebitengine routing note appears in QA plan output
- [ ] `using-autoebiten` skill is explicitly named
- [ ] Integration story test recommendation aligns with testkit patterns
- [ ] Verdict is COMPLETE
```

- [ ] **Step 3: Verify insertion**

Read the test spec to confirm Case 6 appears before Protocol Compliance.

- [ ] **Step 4: Commit**

```bash
git add CCGS Skill Testing Framework/skills/utility/qa-plan.md
git commit -m "test: add Ebitengine routing case to qa-plan spec"
```

---

### Task 8: Add Case 6 to team-qa Test Spec

**Files:**
- Modify: `CCGS Skill Testing Framework/skills/team/team-qa.md` (append Case 6)

- [ ] **Step 1: Read team-qa.md test spec to find insertion point**

Locate the Protocol Compliance section. Case 6 should be added before Protocol Compliance.

- [ ] **Step 2: Append Case 6**

Insert before Protocol Compliance section:

```markdown
---

### Case 6: Ebitengine Project — Phase 2 routes to using-autoebiten skill

**Fixture:**
- `technical-preferences.md` has engine set to Ebitengine or Ebiten
- Sprint with 2 stories: 1 Logic (automated test passes), 1 Integration (needs automation)
- Smoke check passes

**Input:** `/team-qa sprint-08`

**Expected behavior:**
1. Phase 1: Reads engine config from `technical-preferences.md` → Ebitengine or Ebiten; reads sprint files; reports "Found 2 stories. Current stage: [stage]"
2. Phase 2: Spawns `qa-lead` via Task; strategy includes routing note: "**Ebitengine Automation** — use `using-autoebiten` skill for Integration story testkit test"
3. Phase 2: Strategy table classifies: Logic (automated), Integration (testkit needed); presents to user; AskUserQuestion: user selects "Looks good — proceed to test plan"
4. Phase 3: QA plan produced with Ebitengine routing section; asks "May I write?"; writes on approval
5. Phase 5: Integration story test case writing spawned via `qa-tester`; test cases reference testkit patterns (not manual-only)
6. Phase 6: Logic story marked PASS (automated); Integration story test execution documented
7. Phase 7: Sign-off report shows Integration story covered via automation; Verdict: APPROVED or APPROVED WITH CONDITIONS
8. Verdict: COMPLETE

**Assertions:**
- [ ] Phase 2 strategy explicitly mentions `using-autoebiten` skill for Ebitengine
- [ ] Integration story receives automation recommendation (testkit), not manual-only
- [ ] QA plan includes Ebitengine routing section
- [ ] Sign-off report Test Coverage Summary shows Integration story with automation coverage
- [ ] Verdict: COMPLETE
```

- [ ] **Step 3: Verify insertion**

Read the test spec to confirm Case 6 appears before Protocol Compliance.

- [ ] **Step 4: Commit**

```bash
git add CCGS Skill Testing Framework/skills/team/team-qa.md
git commit -m "test: add Ebitengine routing case to team-qa spec"
```

---

## Plan Self-Review

**1. Spec coverage:**
- Skill modifications (test-setup, qa-plan, team-qa) → Tasks 1, 2, 3 ✓
- Cross-references (game-loop.md, ui.md) → Tasks 4, 5 ✓
- Test spec updates → Tasks 6, 7, 8 ✓
- All 8 files covered ✓

**2. Placeholder scan:**
- No "TBD", "TODO", or vague instructions ✓
- All code blocks show exact content to insert ✓
- Exact file paths specified ✓
- Exact commit messages specified ✓

**3. Consistency:**
- "Ebitengine or Ebiten" used consistently across all tasks ✓
- `using-autoebiten` skill named consistently ✓
- Test spec assertions match expected behavior descriptions ✓

**No gaps found.**