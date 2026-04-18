# Agent Test Spec: ebiten-specialist

## Agent Summary
Domain: Ebitengine-specific patterns, Game interface, Update/Draw lifecycle, rendering pipeline, input handling.
Does NOT own: general Go code quality, idiomatic Go patterns (explicitly excluded).
Model tier: Sonnet (default).
No gate IDs assigned.

---

## Static Assertions (Structural)

- [ ] `description:` field is present and domain-specific (references Ebitengine patterns, Game interface, rendering)
- [ ] `allowed-tools:` list includes Read, Write, Edit, Bash, Glob, Grep, Task
- [ ] Model tier is Sonnet (default for specialists)
- [ ] Agent definition references `docs/engine-reference/ebiten/VERSION.md` as the authoritative API source
- [ ] Exclusion noted: does NOT provide general Go code quality advice

---

## Test Cases

### Case 1: In-domain request — appropriate output
**Input:** "How should I structure Update and Draw for a game with multiple screens (menu, gameplay, pause)?"
**Expected behavior:**
- Produces a pattern guide with rationale:
  - State machine in Update for screen transitions
  - Draw renders current screen state
  - Layout provides consistent logical resolution
- Provides concrete examples of screen manager pattern
- Does NOT write raw Go code for the full implementation — refers to gameplay-programmer for logic structure
- Notes the fixed TPS assumption for Update logic

**Assertions:**
- [ ] Agent provides pattern guide, not full implementation
- [ ] Collaborative protocol followed (ask → draft → approve)
- [ ] No general Go style advice provided (stays in Ebitengine domain)

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 2: Wrong-engine redirect
**Input:** "Write a Godot Node2D script with _ready() and _process() for a player sprite."
**Expected behavior:**
- Does NOT produce Godot GDScript code
- Clearly identifies that this is a Godot pattern, not an Ebitengine pattern
- Provides the Ebitengine equivalent: Game interface with Update() and Draw() instead of _ready() and _process()
- Confirms the project is Ebitengine-based and redirects the conceptual mapping

**Assertions:**
- [ ] Agent declines Godot-specific request
- [ ] Provides Ebitengine equivalent pattern
- [ ] Redirect is clear and educational

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 3: Post-cutoff API risk
**Input:** "Use FinalScreenDrawer.DrawFinalScreen from v2.8 for custom offscreen rendering."
**Expected behavior:**
- Identifies that FinalScreenDrawer is a specific Ebitengine interface
- Checks VERSION.md for version context
- If the API is documented, provides guidance
- If not documented, flags the risk: LLM knowledge may be incomplete
- Directs user to verify against pkg.go.dev if uncertain

**Assertions:**
- [ ] Agent checks engine reference before suggesting API
- [ ] Flags version risk if API is not in reference docs
- [ ] Provides verification guidance

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 4: Shader delegation
**Input:** "Create a Kage shader for water ripple effect with animated fragment function."
**Expected behavior:**
- Identifies this as shader domain work
- Does NOT author shader code directly
- Delegates to `ebiten-kage-specialist` via Task tool
- Provides context to shader specialist about the visual effect requirements

**Assertions:**
- [ ] Agent delegates shader request to correct sub-specialist
- [ ] Does not attempt to write shader code directly
- [ ] Provides full context in delegation prompt

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 5: UI delegation
**Input:** "Create a button with a dropdown menu using EbitenUI for the settings screen."
**Expected behavior:**
- Identifies this as EbitenUI widget work
- Does NOT author widget code directly
- Delegates to `ebiten-ui-specialist` via Task tool
- Provides context to UI specialist about the screen requirements, layout constraints, and theme expectations

**Assertions:**
- [ ] Agent delegates UI request to correct sub-specialist
- [ ] Does not attempt to write EbitenUI code directly
- [ ] Provides full context in delegation prompt

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 6: Context pass — offscreen rendering
**Input:** Game needs pixel art scaling. Request: "How do I implement nearest-neighbor scaling for a 320x240 logical screen on a 1280x960 window?"
**Expected behavior:**
- Provides Layout pattern: return (320, 240) for logical size
- Explains Draw pattern: render to offscreen Image, then DrawImage to screen with scaling GeoM
- Explains Filter option: FilterNearest for pixel art
- References rendering.md for detailed patterns

**Assertions:**
- [ ] Agent references engine reference docs
- [ ] Provides concrete pattern with correct API names (DrawImageOptions, GeoM, Filter)
- [ ] Explains premultiplied alpha considerations if color scaling involved

**Case Verdict**: PASS / FAIL / PARTIAL

---

## Protocol Compliance

- [ ] Stays within declared domain (Ebitengine patterns, Game interface, rendering)
- [ ] Redirects shader requests to ebiten-kage-specialist
- [ ] Redirects UI requests to ebiten-ui-specialist
- [ ] Returns structured findings (pattern guides, API recommendations)
- [ ] Treats `docs/engine-reference/ebiten/VERSION.md` as authoritative
- [ ] Does NOT provide general Go code quality advice (explicitly excluded)
- [ ] Defers implementation to gameplay-programmer when appropriate

---

## Coverage Notes
- Exclusion case verifies the agent does not drift into Go style enforcement
- Shader delegation case verifies correct sub-specialist invocation for Kage
- UI delegation case verifies correct sub-specialist invocation for EbitenUI
- Offscreen rendering case verifies practical pattern guidance
