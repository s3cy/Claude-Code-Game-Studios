# Agent Test Spec: ebiten-ui-specialist

## Agent Summary
Domain: EbitenUI widget hierarchy, layout systems, theme configuration, input layer integration (UIHovered, Layer blocking), Game interface integration.
Delegates styling/theming to: art-director.
Does NOT own: general Go code quality, engine-level input APIs, gameplay input logic.
Model tier: Sonnet (default).
No gate IDs assigned.

---

## Static Assertions (Structural)

- [ ] `description:` field is present and domain-specific (references EbitenUI widgets, layouts, input layering)
- [ ] `allowed-tools:` list includes Read, Write, Edit, Bash, Glob, Grep, Task
- [ ] Model tier is Sonnet (default for specialists)
- [ ] Agent definition references `docs/engine-reference/ebiten/modules/ui.md` as authoritative source
- [ ] Delegation to art-director noted for visual styling decisions
- [ ] Exclusion noted: does NOT provide general Go code quality advice

---

## Test Cases

### Case 1: In-domain request — appropriate output
**Input:** "How do I create a main menu with three buttons: Start, Settings, Quit?"

**Expected behavior:**
- Produces widget hierarchy guide:
  - Root Container with AnchorLayout for center positioning
  - Button widgets with Text opts for labels
  - ClickedEvent handlers for each button
- Provides concrete EbitenUI code example with functional options pattern
- Does NOT write full implementation — asks "May I write this to [filepath]?" first
- Notes that visual styling should go to art-director

**Assertions:**
- [ ] Agent provides widget hierarchy pattern guide
- [ ] Collaborative protocol followed (ask → draft → approve)
- [ ] Styling delegated to art-director
- [ ] No general Go style advice provided (stays in EbitenUI domain)

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 2: Wrong-library redirect
**Input:** "Use Fyne to create a settings dialog with checkboxes."

**Expected behavior:**
- Does NOT produce Fyne code (different Go GUI library)
- Clearly identifies that Fyne is a separate GUI framework, not EbitenUI
- Provides the EbitenUI equivalent: Container + Checkbox widgets
- Explains EbitenUI is for Ebitengine games specifically
- Confirms project is Ebitengine-based and redirects

**Assertions:**
- [ ] Agent declines Fyne-specific request
- [ ] Provides EbitenUI equivalent pattern (Container + Checkbox)
- [ ] Redirect is clear and educational
- [ ] Does not confuse EbitenUI with other Go GUI libraries

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 3: Input blocking — modal dialog
**Input:** "I need a modal pause menu that blocks gameplay clicks when open."

**Expected behavior:**
- Explains UIHovered + input Layer pattern
- Provides Layer configuration with FullScreen: true, BlockLower: true
- Shows UI.Update() placement before gameplay input check
- References ui.md for integration pattern
- Explains that modal windows in EbitenUI have built-in Modal option

**Assertions:**
- [ ] Agent explains input layer blocking pattern
- [ ] Provides Layer configuration with FullScreen + BlockLower
- [ ] References engine reference docs (ui.md, game-loop.md)
- [ ] Shows UI.Update() + UIHovered check pattern

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 4: Styling delegation
**Input:** "Design a dark theme for my medieval RPG UI with gold accents."

**Expected behavior:**
- Identifies this as visual styling domain
- Does NOT make color/font decisions directly
- Delegates to art-director via Task tool with context about RPG aesthetic
- Explains technical theme implementation awaits art direction
- Offers to implement theme config once art-director provides colors/fonts

**Assertions:**
- [ ] Agent delegates styling request to art-director
- [ ] Does not make visual design decisions
- [ ] Provides full context in delegation prompt
- [ ] Offers technical implementation after art direction

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 5: Integration placement
**Input:** "Where should I call ui.Update() and ui.Draw() in my Game struct?"

**Expected behavior:**
- References game-loop.md for Update/Draw context
- Shows concrete integration code:
  - ui.Update() at start of Update(), before gameplay input
  - ui.Draw() at end of Draw(), after game world
- Explains UIHovered check for gameplay input blocking
- Notes Layout() typically returns window size for dynamic UI

**Assertions:**
- [ ] Agent references game-loop.md
- [ ] Provides correct integration order (Update first, Draw after game world)
- [ ] Shows UIHovered check pattern
- [ ] Explains reasoning (UI processes input first, renders on top)

**Case Verdict**: PASS / FAIL / PARTIAL

---

## Protocol Compliance

- [ ] Stays within declared domain (EbitenUI widgets, layouts, themes, input layering)
- [ ] Delegates styling/theming to art-director
- [ ] Returns structured findings (widget hierarchy guides, integration patterns)
- [ ] Treats `docs/engine-reference/ebiten/modules/ui.md` as authoritative
- [ ] Does NOT provide general Go code quality advice (explicitly excluded)
- [ ] Defers implementation approval to user (ask → draft → approve)
- [ ] Does NOT spawn further sub-specialists (leaf node in hierarchy)

---

## Coverage Notes
- Case 1 verifies widget hierarchy guidance without full implementation
- Case 2 verifies library confusion prevention (Fyne vs EbitenUI)
- Case 3 verifies input blocking pattern for modal dialogs
- Case 4 verifies styling delegation to art-director
- Case 5 verifies Game interface integration referencing game-loop.md
