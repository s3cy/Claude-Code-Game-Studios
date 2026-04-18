# Ebitengine Support Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Ebitengine (Go 2D game engine) support with full parity with existing engine specialists.

**Architecture:** Two specialist agents (primary + shader) following existing three-tier hierarchy, plus engine reference docs and minimal setup-engine integration.

**Tech Stack:** Ebitengine v2.9.9 (Go), Kage shader language

---

## File Structure

| File | Purpose |
|------|---------|
| `.claude/agents/ebiten-specialist.md` | Primary engine specialist |
| `.claude/agents/ebiten-kage-specialist.md` | Kage shader sub-specialist |
| `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-specialist.md` | Test spec for primary |
| `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-kage-specialist.md` | Test spec for shader |
| `docs/engine-reference/ebiten/VERSION.md` | Version pin + knowledge gap |
| `docs/engine-reference/ebiten/modules/game-loop.md` | Game interface patterns |
| `docs/engine-reference/ebiten/modules/rendering.md` | Image/DrawImage patterns |

**Modifications:**
- `.claude/skills/setup-engine/SKILL.md` — add Ebitengine option
- `CCGS Skill Testing Framework/catalog.yaml` — add agent entries
- `.claude/docs/coordination-rules.md` — add ebiten tier

---

## Phase 1: Specialist Agents

### Task 1: Create ebiten-specialist Agent

**Files:**
- Create: `.claude/agents/ebiten-specialist.md`

- [ ] **Step 1: Write the agent definition file**

```markdown
---
name: ebiten-specialist
description: "The Ebitengine Specialist is the authority on all Ebitengine-specific patterns: Game interface implementation, Update/Draw lifecycle, rendering pipeline, input handling, and API usage. Focuses on Ebitengine APIs only — not general Go code quality."
tools: Read, Glob, Grep, Write, Edit, Bash, Task
model: sonnet
maxTurns: 20
---
You are the Ebitengine Specialist for a game project built with Ebitengine v2.9.9. You are the team's authority on Ebitengine-specific patterns.

## Collaboration Protocol

**You are a collaborative implementer, not an autonomous code generator.** The user approves all architectural decisions and file changes.

### Implementation Workflow

Before writing any code:

1. **Read the design document:**
   - Identify what's specified vs. what's ambiguous
   - Note any deviations from standard patterns
   - Flag potential implementation challenges

2. **Ask architecture questions:**
   - "Should the screen manager live in Update or as a separate struct?"
   - "Where should [data] live? (Game struct field? External config? EmbedFS?)"
   - "The design doc doesn't specify [edge case]. What should happen when...?"
   - "This will require changes to [other system]. Should I coordinate with that first?"

3. **Propose architecture before implementing:**
   - Show struct organization, file structure, data flow
   - Explain WHY you're recommending this approach (patterns, engine conventions, maintainability)
   - Highlight trade-offs: "This approach is simpler but less flexible" vs "This is more complex but more extensible"
   - Ask: "Does this match your expectations? Any changes before I write the code?"

4. **Implement with transparency:**
   - If you encounter spec ambiguities during implementation, STOP and ask
   - If rules/hooks flag issues, fix them and explain what was wrong
   - If a deviation from the design doc is necessary (technical constraint), explicitly call it out

5. **Get approval before writing files:**
   - Show the code or a detailed summary
   - Explicitly ask: "May I write this to [filepath(s)]?"
   - For multi-file changes, list all affected files
   - Wait for "yes" before using Write/Edit tools

6. **Offer next steps:**
   - "Should I write tests now, or would you like to review the implementation first?"
   - "This is ready for /code-review if you'd like validation"
   - "I notice [potential improvement]. Should I refactor, or is this good for now?"

### Collaborative Mindset

- Clarify before assuming — specs are never 100% complete
- Propose architecture, don't just implement — show your thinking
- Explain trade-offs transparently — there are always multiple valid approaches
- Flag deviations from design docs explicitly — designer should know if implementation differs
- Rules are your friend — when they flag issues, they're usually right
- Tests prove it works — offer to write them proactively

## Core Responsibilities
- Guide Game interface implementation (Update, Draw, Layout methods)
- Review rendering patterns (ebiten.Image, DrawImageOptions, GeoM transformations)
- Advise on fixed timestep handling (TPS defaults, no delta time measurement)
- Coordinate input handling patterns (ebiten.Inpututil, gamepad support)
- Review asset loading patterns (EmbedFS, ebitenutil helpers)
- Ensure proper offscreen rendering for resolution scaling

## Ebitengine Best Practices to Enforce

### Game Loop Architecture
- Update() handles game logic at fixed TPS (60 by default) — never measure delta time manually
- Draw(screen *ebiten.Image) handles rendering — frequency depends on monitor refresh rate
- Layout(outsideW, outsideH) returns logical screen size — called before first Update
- First frame guarantee: Update called at least once before Draw — use for initialization
- FPS and TPS are separate metrics — don't confuse them

### Rendering Patterns
- Use ebiten.NewImage() for offscreen buffers — create once, reuse every frame
- DrawImageOptions.GeoM for transforms (translate, rotate, scale)
- ColorScale for RGBA adjustments (premultiplied alpha)
- SubImage() for sprite sheets — returns a view into the original image
- Never create new Images in Draw() — causes GPU memory allocation every frame

### Input Handling
- Use ebiten.IsKeyPressed() for immediate state checks
- Use inpututil package for press/release detection (JustPressed, IsKeyJustReleased)
- Gamepad: ebiten.AppendGamepadIDs() to detect connected controllers
- Touch: ebiten.AppendTouchPositions() for mobile/web

### Common Pitfalls to Flag
- Heavy computation in Draw() (causes frame drops, not Update drops)
- Blocking operations in Update() (halts game loop entirely)
- Creating Images every frame (GPU memory leak)
- Using deprecated ColorM instead of ColorScale (deprecated v2.5+)
- Not disposing external images on some platforms
- Measuring delta time manually (use fixed TPS assumption)

## Delegation Map

**Reports to**: `technical-director` (via `lead-programmer`)

**Delegates to**:
- `ebiten-kage-specialist` for Kage shader code (custom shaders, uniforms, fragment functions)

**Escalation targets**:
- `technical-director` for engine version upgrades, major architecture decisions
- `lead-programmer` for code architecture conflicts

**Coordinates with**:
- `gameplay-programmer` for game logic architecture (state machines, entity systems)
- `ui-programmer` for code-centric UI patterns (Ebitengine has no visual UI editor)
- `performance-analyst` for profiling Update/Draw performance

## What This Agent Must NOT Do
- Make game design decisions (advise on engine implications, don't decide mechanics)
- Override lead-programmer architecture without discussion
- Provide general Go code quality advice (outside scope — focus on Ebitengine APIs only)
- Implement features directly (delegate to gameplay-programmer or implement with approval)
- Approve external dependencies without technical-director sign-off

## Sub-Specialist Orchestration

You have access to the Task tool to delegate to your sub-specialist:

- `subagent_type: ebiten-kage-specialist` — Kage shader syntax, uniforms, performance patterns, shader debugging

Provide full context in the prompt including relevant file paths, design constraints, and performance requirements.

## Version Awareness

**CRITICAL**: Your training data has a knowledge cutoff. Before suggesting
Ebitengine API code, you MUST:

1. Read `docs/engine-reference/ebiten/VERSION.md` to confirm the engine version
2. Check `docs/engine-reference/ebiten/modules/game-loop.md` for current patterns
3. Check `docs/engine-reference/ebiten/modules/rendering.md` for rendering APIs

If an API you plan to suggest does not appear in the reference docs and was
introduced after May 2025, use WebSearch to verify it exists in the current version.

When in doubt, prefer the API documented in the reference files over your training data.

## When Consulted
Always involve this agent when:
- Designing Game interface implementation for a new project
- Choosing between offscreen rendering vs direct screen drawing
- Setting up input handling patterns
- Implementing sprite sheet rendering
- Optimizing Update/Draw performance
- Debugging rendering issues
```

- [ ] **Step 2: Commit the agent**

```bash
git add .claude/agents/ebiten-specialist.md
git commit -m "feat: add ebiten-specialist agent definition"
```

---

### Task 2: Create ebiten-kage-specialist Agent

**Files:**
- Create: `.claude/agents/ebiten-kage-specialist.md`

- [ ] **Step 1: Write the agent definition file**

```markdown
---
name: ebiten-kage-specialist
description: "The Ebitengine Kage Shader specialist owns all Kage shader language: uniform handling, vertex/fragment functions, shader compilation, and performance patterns for Ebitengine's custom shader system."
tools: Read, Glob, Grep, Write, Edit, Bash, Task
model: sonnet
maxTurns: 20
---
You are the Ebitengine Kage Shader Specialist for a game project built with Ebitengine v2.9.9. You own everything related to Kage shaders.

## Collaboration Protocol

**You are a collaborative implementer, not an autonomous code generator.** The user approves all architectural decisions and file changes.

### Implementation Workflow

Before writing any shader code:

1. **Read the design document:**
   - Identify what visual effect is specified
   - Note any performance constraints
   - Flag potential shader complexity issues

2. **Ask shader architecture questions:**
   - "Should this effect use uniforms or be computed in the shader?"
   - "The design specifies [effect]. How many texture samples will this require?"
   - "Is this shader for sprites (2D) or full-screen effects?"

3. **Propose shader architecture before implementing:**
   - Show uniform structure, function signatures, expected GPU cost
   - Explain WHY you're recommending this approach
   - Highlight trade-offs: "More uniforms = more flexibility but more overhead"
   - Ask: "Does this match your expectations for visual output and performance?"

4. **Implement with transparency:**
   - If shader compilation fails, diagnose and explain the error
   - If performance exceeds budget, flag immediately
   - If a simpler alternative exists, present both options

5. **Get approval before writing files:**
   - Show the shader code
   - Explicitly ask: "May I write this to [filepath]?"
   - Wait for "yes" before using Write/Edit tools

6. **Offer next steps:**
   - "Want me to write a test Go file that loads and applies this shader?"
   - "I notice [optimization opportunity]. Should I refactor?"

### Collaborative Mindset

- Clarify visual intent before assuming — shaders are hard to debug
- Propose shader structure, don't just implement
- Explain GPU cost transparently — texture samples and branching matter
- Flag performance risks early — shaders run per-pixel
- Tests prove it works — offer integration tests

## Core Responsibilities
- Write Kage shader files for Ebitengine's shader system
- Design uniform structures for artist-friendly parameter control
- Implement vertex and fragment functions
- Optimize shader performance (minimize texture samples, avoid branching)
- Debug shader compilation errors

## Kage Shader Language Standards

### Shader Structure
```kage
//kage:unit pixels

uniform vec2 ScrollSpeed;
uniform vec4 Tint: source_color;

func Fragment(destPos vec2, srcPos vec2, color vec4) vec4 {
    // Fragment logic
    return color
}
```

### Key Kage Features
- `//kage:unit pixels` — shader unit declaration (pixels or texels)
- `uniform` — parameters passed from Go code
- `func Fragment()` — per-pixel fragment function
- `func Vertex()` — per-vertex transformation (optional)
- `destPos` — destination position on screen
- `srcPos` — source position in texture
- `color` — input color from texture

### Uniform Naming Conventions
- PascalCase for uniform names (e.g., `ScrollSpeed`, `TintColor`)
- Type hints for colors: `uniform vec4 Tint: source_color`
- Arrays for multiple values: `uniform vec2 LightPositions[8]`

### Performance Patterns
- Minimize texture samples — each sample costs GPU time
- Use `step()` and `mix()` instead of branching where possible
- Pre-compute values in Go and pass as uniforms
- Avoid loops in fragment functions — exponential cost

### Common Shader Effects

#### Color Inversion
```kage
//kage:unit pixels

func Fragment(destPos vec2, srcPos vec2, color vec4) vec4 {
    return vec4(1.0 - color.r, 1.0 - color.g, 1.0 - color.b, color.a)
}
```

#### Scroll/Animation
```kage
//kage:unit pixels

uniform vec2 ScrollSpeed;

func Fragment(destPos vec2, srcPos vec2, color vec4) vec4 {
    srcPos += ScrollSpeed * float(ebiten.Ticks())
    return color
}
```

#### Custom Lighting (2D)
```kage
//kage:unit pixels

uniform vec2 LightPos;
uniform float LightRadius;

func Fragment(destPos vec2, srcPos vec2, color vec4) vec4 {
    dist := distance(destPos, LightPos)
    intensity := clamp(1.0 - dist / LightRadius, 0.0, 1.0)
    return vec4(color.rgb * intensity, color.a)
}
```

## Common Shader Anti-Patterns
- Texture reads in a loop (exponential GPU cost)
- Dynamic branching on per-pixel data (unpredictable performance)
- Too many uniforms (pass arrays instead of many individual values)
- Complex math in fragment when vertex could pre-compute
- Not accounting for premultiplied alpha in color operations

## Shader Compilation in Go

```go
// Load and compile shader
shaderSrc, err := fs.ReadFile(assetsFS, "shaders/effect.kage")
if err != nil {
    return err
}
s, err := ebiten.NewShader(shaderSrc)
if err != nil {
    return err // Compilation error — check syntax
}

// Apply shader via DrawRectShader
opts := &ebiten.DrawRectShaderOptions{}
opts.Uniforms = map[string]interface{}{
    "ScrollSpeed": []float32{0.1, 0.05},
}
screen.DrawRectShader(w, h, s, opts)
```

## Version Awareness

**CRITICAL**: Your training data has a knowledge cutoff. Before suggesting
Kage shader syntax, you MUST:

1. Read `docs/engine-reference/ebiten/VERSION.md` to confirm the engine version
2. Use WebSearch if unsure about Kage syntax changes in newer versions

When in doubt, prefer the syntax documented at pkg.go.dev for the current version.

## Coordination
- Work with **ebiten-specialist** for overall rendering architecture
- Work with **art-director** for visual direction and shader effect requirements
- Work with **performance-analyst** for GPU performance profiling
```

- [ ] **Step 2: Commit the agent**

```bash
git add .claude/agents/ebiten-kage-specialist.md
git commit -m "feat: add ebiten-kage-specialist agent definition"
```

---

## Phase 2: Test Specs

### Task 3: Create ebiten-specialist Test Spec

**Files:**
- Create: `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-specialist.md`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p "CCGS Skill Testing Framework/agents/engine/ebiten"
```

- [ ] **Step 2: Write the test spec file**

```markdown
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

### Case 5: Context pass — offscreen rendering
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
- [ ] Returns structured findings (pattern guides, API recommendations)
- [ ] Treats `docs/engine-reference/ebiten/VERSION.md` as authoritative
- [ ] Does NOT provide general Go code quality advice (explicitly excluded)
- [ ] Defers implementation to gameplay-programmer when appropriate

---

## Coverage Notes
- Exclusion case verifies the agent does not drift into Go style enforcement
- Shader delegation case verifies correct sub-specialist invocation
- Offscreen rendering case verifies practical pattern guidance
```

- [ ] **Step 3: Commit the test spec**

```bash
git add "CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-specialist.md"
git commit -m "feat: add ebiten-specialist test spec"
```

---

### Task 4: Create ebiten-kage-specialist Test Spec

**Files:**
- Create: `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-kage-specialist.md`

- [ ] **Step 1: Write the test spec file**

```markdown
# Agent Test Spec: ebiten-kage-specialist

## Agent Summary
Domain: Kage shader language, uniforms, vertex/fragment functions, shader compilation, shader performance.
Reports to: ebiten-specialist via lead-programmer.
Model tier: Sonnet (default).
No gate IDs assigned.

---

## Static Assertions (Structural)

- [ ] `description:` field references Kage shader language and Ebitengine shader system
- [ ] `allowed-tools:` list includes Read, Write, Edit, Bash, Glob, Grep, Task
- [ ] Model tier is Sonnet
- [ ] Agent references `docs/engine-reference/ebiten/VERSION.md` for version context

---

## Test Cases

### Case 1: In-domain shader request
**Input:** "Write a Kage shader that inverts colors on a sprite."
**Expected behavior:**
- Provides complete Kage shader code:
  ```kage
  //kage:unit pixels
  
  func Fragment(destPos vec2, srcPos vec2, color vec4) vec4 {
      return vec4(1.0 - color.r, 1.0 - color.g, 1.0 - color.b, color.a)
  }
  ```
- Explains how to load and apply in Go (NewShader, DrawRectShader)
- Explicitly asks: "May I write this to [filepath]?"
- Waits for approval before writing

**Assertions:**
- [ ] Agent provides complete shader code
- [ ] Code uses correct Kage syntax (unit declaration, Fragment function signature)
- [ ] Collaborative protocol followed (ask before write)

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 2: Wrong-shader-language redirect
**Input:** "Create a GLSL fragment shader for a Godot Sprite2D with void fragment()."
**Expected behavior:**
- Does NOT produce GLSL/Godot shader code
- Clearly identifies this as a Godot shader request
- Provides the Kage equivalent pattern (func Fragment in Kage)
- Confirms the project is Ebitengine-based

**Assertions:**
- [ ] Agent declines GLSL/Godot shader request
- [ ] Provides Kage equivalent explanation
- [ ] Redirect is clear about language differences

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 3: Performance warning
**Input:** "Add a for loop with 50 iterations in my fragment shader to check 50 light positions."
**Expected behavior:**
- Flags performance risk immediately
- Explains why loops in fragment shaders are expensive (per-pixel execution × 50)
- Suggests alternatives:
  - Pass lights as uniform array, compute in Go
  - Use step/mix instead of branching
  - Reduce light count or use deferred approach
- Does NOT implement the expensive pattern without warning

**Assertions:**
- [ ] Agent flags performance risk before implementing
- [ ] Provides concrete alternatives
- [ ] Does not silently implement expensive pattern

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 4: Uniform array handling
**Input:** "Pass 10 light positions to my shader for 2D lighting."
**Expected behavior:**
- Explains Kage uniform array syntax:
  ```kage
  uniform vec2 LightPositions[10];
  ```
- Notes uniform count limits depend on GPU
- Provides Go code for passing uniforms:
  ```go
  opts.Uniforms["LightPositions"] = []float32{x1, y1, x2, y2, ...}
  ```
- Warns about uniform count performance if approaching limits

**Assertions:**
- [ ] Agent provides correct uniform array syntax
- [ ] Shows Go integration code
- [ ] Notes uniform limits

**Case Verdict**: PASS / FAIL / PARTIAL

---

### Case 5: Shader compilation error diagnosis
**Input:** "My shader fails with 'unexpected token' at line 5."
**Expected behavior:**
- Diagnoses common Kage syntax errors:
  - Missing unit declaration (`//kage:unit pixels`)
  - Wrong function signature (Fragment takes destPos, srcPos, color)
  - Type mismatch in return value
  - Missing colon in uniform type hints
- Suggests fixes based on the specific error location
- Offers to review full shader file if needed

**Assertions:**
- [ ] Agent diagnoses common Kage errors correctly
- [ ] Provides actionable fix suggestions
- [ ] Offers further debugging if needed

**Case Verdict**: PASS / FAIL / PARTIAL

---

## Protocol Compliance

- [ ] Stays within declared domain (Kage shaders, uniforms, performance)
- [ ] Provides complete shader code when asked (not just descriptions)
- [ ] Flags performance risks proactively
- [ ] Asks before writing files
- [ ] Explains Go integration for shader loading/application

---

## Coverage Notes
- Performance warning case verifies proactive risk flagging
- Uniform array case verifies practical uniform handling guidance
- Compilation error case verifies debugging capability
```

- [ ] **Step 2: Commit the test spec**

```bash
git add "CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-kage-specialist.md"
git commit -m "feat: add ebiten-kage-specialist test spec"
```

---

## Phase 3: Engine Reference Docs

### Task 5: Create VERSION.md

**Files:**
- Create: `docs/engine-reference/ebiten/VERSION.md`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p docs/engine-reference/ebiten/modules
```

- [ ] **Step 2: Write VERSION.md**

```markdown
# Ebitengine — Version Reference

| Field | Value |
|-------|-------|
| **Engine Version** | Ebitengine v2.9.9 |
| **Release Date** | 2024 (check ebitengine.org for exact date) |
| **Project Pinned** | 2026-04-18 |
| **Last Docs Verified** | 2026-04-18 |
| **LLM Knowledge Cutoff** | May 2025 |
| **Source Reference** | `~/Desktop/source/ebiten` (local checkout at v2.9.9) |

## Knowledge Gap Warning

The LLM's training data covers Ebitengine up to approximately v2.6-v2.7.
Version v2.9.9 introduces incremental changes but core patterns (Game interface,
Image, DrawImageOptions) remain stable. Always cross-reference this directory
and pkg.go.dev before suggesting Ebitengine API calls.

## Risk Level

**LOW** — v2.9.x patterns are stable and within training data range. Core APIs
(Game interface, Image, GeoM, DrawImageOptions) have not changed significantly.

## Key Packages

| Package | Purpose |
|---------|---------|
| `ebiten` | Core package (Game interface, Image, RunGame, input) |
| `ebiten/audio` | Audio context, players (Ogg/Vorbis, MP3, WAV) |
| `ebiten/inpututil` | Press/release detection utilities |
| `ebiten/text/v2` | Text rendering |
| `ebiten/vector` | Vector graphics (lines, paths) |
| `ebiten/colorm` | Color matrix operations |
| `ebiten/ebitenutil` | Debug printing, image loading helpers |

## Platform Support

- Windows (no CGo required)
- macOS
- Linux
- FreeBSD
- Android
- iOS
- WebAssembly
- Nintendo Switch (licensed)
- Xbox (limited availability)

## Key Features

- 2D Graphics (geometry/color matrices, composition modes, offscreen rendering)
- Custom shaders (Kage shader language)
- Input (keyboard, mouse, gamepad, touch)
- Audio (streaming, loops, multiple formats)

## Version History (Post-Cutoff)

| Version | Key Changes |
|---------|-------------|
| v2.7 | Single-thread mode opt-in via RunGameOptions |
| v2.8 | FinalScreenDrawer interface for custom final screen rendering |
| v2.9 | Incremental API refinements (check pkg.go.dev changelog) |

## When in Doubt

1. Check `docs/engine-reference/ebiten/modules/game-loop.md` for Game interface patterns
2. Check `docs/engine-reference/ebiten/modules/rendering.md` for rendering APIs
3. Use WebSearch to verify API on pkg.go.dev/github.com/hajimehoshi/ebiten/v2
4. Reference local source at `~/Desktop/source/ebiten` for exact implementations
```

- [ ] **Step 3: Commit VERSION.md**

```bash
git add docs/engine-reference/ebiten/VERSION.md
git commit -m "docs: add Ebitengine VERSION.md reference"
```

---

### Task 6: Create game-loop.md Module

**Files:**
- Create: `docs/engine-reference/ebiten/modules/game-loop.md`

- [ ] **Step 1: Write game-loop.md**

```markdown
# Ebitengine — Game Loop Module

## Core Pattern

Ebitengine uses a fixed-timestep game loop via the `Game` interface:

```go
type Game interface {
    Update() error          // Called at fixed TPS (60 by default)
    Draw(screen *Image)     // Called every frame (monitor refresh rate)
    Layout(outsideW, outsideH int) (screenW, screenH int)
}
```

Start the game loop with:

```go
func main() {
    game := &MyGame{}
    ebiten.SetWindowSize(640, 480)
    ebiten.SetWindowTitle("My Game")
    if err := ebiten.RunGame(game); err != nil {
        log.Fatal(err)
    }
}
```

---

## Update() — Game Logic

**Frequency:** Fixed TPS (ticks per second), default 60.
- Called exactly 60 times per second (or configured TPS)
- Time delta between Updates is always `1/TPS` seconds (1/60s by default)
- **Do NOT measure delta time manually** — rely on fixed TPS assumption

**Purpose:**
- Update game state (position, health, timers)
- Process input (ebiten.IsKeyPressed, inpututil.JustPressed)
- Handle transitions (screen switching, game over)
- Return `nil` to continue, `ebiten.Termination` to exit gracefully, or error to halt

```go
func (g *Game) Update() error {
    // Input handling
    if ebiten.IsKeyPressed(ebiten.KeyArrowRight) {
        g.playerX += g.playerSpeed
    }
    
    // Game logic
    g.enemies.Update()
    g.projectiles.Update()
    
    // State transitions
    if g.playerHealth <= 0 {
        g.state = StateGameOver
    }
    
    return nil
}
```

**First frame guarantee:** Update is called at least once before Draw.
Use the first Update call for initialization:

```go
func (g *Game) Update() error {
    if !g.initialized {
        g.initialize()
        g.initialized = true
    }
    // ... rest of Update
}
```

---

## Draw() — Rendering

**Frequency:** Monitor refresh rate (typically 60Hz, but may be 120Hz+ on high-refresh displays).
- FPS (frames per second) is NOT the same as TPS
- Draw may be called more or less frequently than Update
- **Do NOT put game logic in Draw** — only rendering

**Purpose:**
- Render current game state to screen Image
- Draw images, apply shaders, render text
- Clear screen or fill with background color

```go
func (g *Game) Draw(screen *ebiten.Image) {
    // Clear screen
    screen.Fill(color.RGBA{0, 0, 0, 255})
    
    // Draw entities
    for _, enemy := range g.enemies {
        opts := &ebiten.DrawImageOptions{}
        opts.GeoM.Translate(enemy.x, enemy.y)
        screen.DrawImage(enemy.sprite, opts)
    }
    
    // Draw player
    opts := &ebiten.DrawImageOptions{}
    opts.GeoM.Translate(g.playerX, g.playerY)
    screen.DrawImage(g.playerSprite, opts)
}
```

---

## Layout() — Screen Sizing

**Frequency:** Called before first Update, then almost every frame.

**Purpose:**
- Return the logical screen size (internal resolution)
- Ebitengine scales this to fit the window/monitor
- Allows fixed logical resolution (pixel art) or dynamic sizing

```go
// Fixed logical resolution (pixel art games)
func (g *Game) Layout(outsideW, outsideH int) (int, int) {
    return 320, 240  // Logical size, scaled to window
}

// Dynamic sizing (modern games)
func (g *Game) Layout(outsideW, outsideH int) (int, int) {
    return outsideW, outsideH  // Match window size
}
```

**LayoutF (float version):**
```go
func (g *Game) LayoutF(outsideW, outsideH float64) (float64, float64) {
    return 320.0, 240.0
}
```

---

## TPS Configuration

Set custom TPS before RunGame:

```go
ebiten.SetTPS(30)  // 30 ticks per second (slower games)
ebiten.SetTPS(120) // 120 ticks per second (fast-action games)
```

Default TPS: 60.

---

## Common Patterns

### Screen Manager

```go
type Game struct {
    currentScreen Screen
    screens       map[ScreenID]Screen
}

func (g *Game) Update() error {
    return g.currentScreen.Update()
}

func (g *Game) Draw(screen *ebiten.Image) {
    g.currentScreen.Draw(screen)
}

func (g *Game) SwitchScreen(id ScreenID) {
    g.currentScreen = g.screens[id]
}
```

### Offscreen Rendering (Pixel Art Scaling)

```go
type Game struct {
    offscreen *ebiten.Image
}

func (g *Game) Layout(outsideW, outsideH int) (int, int) {
    return 320, 240  // Logical size
}

func (g *Game) Draw(screen *ebiten.Image) {
    // Render to offscreen at logical size
    g.offscreen.Clear()
    g.renderGame(g.offscreen)
    
    // Scale offscreen to screen
    opts := &ebiten.DrawImageOptions{}
    scale := float64(screen.Bounds().Dx()) / 320.0
    opts.GeoM.Scale(scale, scale)
    screen.DrawImage(g.offscreen, opts)
}
```

---

## Common Pitfalls

| Pitfall | Consequence | Fix |
|---------|-------------|-----|
| Heavy computation in Draw | Frame drops, choppy rendering | Move logic to Update |
| Blocking in Update | Game loop halts | Use goroutines for async work |
| Measuring delta time manually | Inconsistent behavior | Use fixed TPS assumption |
| Creating Images in Draw | GPU memory leak | Create once in init/struct |
| FPS/TPS confusion | Wrong timing assumptions | FPS = monitor refresh, TPS = game logic |

---

## Performance Monitoring

```go
// Get current FPS and TPS
fps := ebiten.ActualFPS()
tps := ebiten.ActualTPS()

// Debug print
ebitenutil.DebugPrint(screen, fmt.Sprintf("FPS: %.1f, TPS: %.1f", fps, tps))
```

Use `ActualTPS()` to verify Update frequency. Use `ActualFPS()` for Draw frequency monitoring.

---

## Reference

- Source: `~/Desktop/source/ebiten/run.go` (Game interface definition)
- Source: `~/Desktop/source/ebiten/doc.go` (package documentation)
- pkg.go.dev: https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2
```

- [ ] **Step 2: Commit game-loop.md**

```bash
git add docs/engine-reference/ebiten/modules/game-loop.md
git commit -m "docs: add Ebitengine game-loop module reference"
```

---

### Task 7: Create rendering.md Module

**Files:**
- Create: `docs/engine-reference/ebiten/modules/rendering.md`

- [ ] **Step 1: Write rendering.md**

```markdown
# Ebitengine — Rendering Module

## Core Types

### ebiten.Image

Represents a 2D image with premultiplied RGBA pixels.

```go
// Create new image
img := ebiten.NewImage(320, 240)

// Create from file (via ebitenutil)
img, err := ebitenutil.NewImageFromFile("sprite.png")

// Create from embedded FS
img, err := ebitenutil.NewImageFromFS(assetsFS, "sprites/player.png")
```

**Important:** Images are GPU-backed. Creating new Images every frame causes GPU memory allocation.

### DrawImageOptions

Controls how an image is drawn:

```go
opts := &ebiten.DrawImageOptions{}
opts.GeoM.Translate(100, 50)           // Position
opts.GeoM.Scale(2.0, 2.0)              // Scale 2x
opts.GeoM.Rotate(math.Pi / 4)          // Rotate 45°
opts.ColorScale.Scale(1.0, 1.0, 1.0, 0.5) // 50% opacity
opts.Blend = ebiten.BlendSourceOver    // Alpha blending
screen.DrawImage(sprite, opts)
```

---

## GeoM — Geometry Matrix

Transform matrix for position, rotation, scale:

```go
opts := &ebiten.DrawImageOptions{}

// Translate (move)
opts.GeoM.Translate(x, y)

// Scale
opts.GeoM.Scale(scaleX, scaleY)

// Rotate (around origin, then translate)
opts.GeoM.Rotate(angle)
opts.GeoM.Translate(x, y)

// Combined: rotate around center, then position
opts.GeoM.Translate(-spriteW/2, -spriteH/2) // Center offset
opts.GeoM.Rotate(angle)
opts.GeoM.Translate(x, y)                   // Final position
```

**Order matters:** GeoM operations are applied in sequence. Translate → Rotate differs from Rotate → Translate.

---

## ColorScale

RGBA color scaling (premultiplied alpha):

```go
opts := &ebiten.DrawImageOptions{}

// Full opacity, normal colors
opts.ColorScale.Reset()

// 50% opacity (alpha = 0.5)
opts.ColorScale.Scale(1.0, 1.0, 1.0, 0.5)

// Color tint (reduce other channels)
opts.ColorScale.Scale(0.5, 0.5, 1.0, 1.0) // Blue tint

// Invert colors (set R/G/B to 1 - original)
// Use shader for complex color operations
```

**Note:** ColorScale uses premultiplied alpha. `Scale(r, g, b, a)` means the final color is `(r*a, g*a, b*a, a)`.

---

## SubImage — Sprite Sheets

Extract regions from a larger image:

```go
// Load sprite sheet
sheet, err := ebitenutil.NewImageFromFile("spritesheet.png")

// Extract individual sprites
playerSprite := sheet.SubImage(image.Rect(0, 0, 32, 32)).(*ebiten.Image)
enemySprite := sheet.SubImage(image.Rect(32, 0, 64, 32)).(*ebiten.Image)

// Draw sub-image
opts := &ebiten.DrawImageOptions{}
opts.GeoM.Translate(x, y)
screen.DrawImage(playerSprite, opts)
```

SubImage returns a view into the original — no additional GPU memory.

---

## Blend Modes

```go
opts.Blend = ebiten.BlendSourceOver    // Normal alpha blending (default)
opts.Blend = ebiten.BlendCopy           // Replace destination (no blending)
opts.Blend = ebiten.BlendLighten        // Keep lighter pixels
opts.Blend = ebiten.BlendAdd            // Additive blending (glows)
opts.Blend = ebiten.BlendMultiply       // Multiply colors (shadows)
```

---

## Filter Modes

```go
opts.Filter = ebiten.FilterNearest   // Pixel art (no smoothing)
opts.Filter = ebiten.FilterLinear    // Smooth interpolation
```

Use `FilterNearest` for pixel art games to preserve sharp edges when scaling.

---

## Offscreen Rendering

Render to an intermediate image, then draw to screen:

```go
type Game struct {
    offscreen *ebiten.Image
}

func (g *Game) init() {
    g.offscreen = ebiten.NewImage(320, 240) // Logical resolution
}

func (g *Game) Draw(screen *ebiten.Image) {
    // Clear and render to offscreen
    g.offscreen.Fill(color.Black)
    g.renderScene(g.offscreen)
    
    // Scale to screen size
    opts := &ebiten.DrawImageOptions{}
    scaleX := float64(screen.Bounds().Dx()) / 320.0
    scaleY := float64(screen.Bounds().Dy()) / 240.0
    opts.GeoM.Scale(scaleX, scaleY)
    opts.Filter = ebiten.FilterNearest // Pixel art
    
    screen.DrawImage(g.offscreen, opts)
}
```

**Use cases:**
- Pixel art scaling with fixed logical resolution
- Post-processing effects (apply shader to offscreen, then draw)
- Layering (render multiple passes, composite)

---

## Shader Rendering

Apply custom Kage shaders:

```go
// Load shader
shaderSrc, err := fs.ReadFile(assetsFS, "shaders/invert.kage")
shader, err := ebiten.NewShader(shaderSrc)

// Apply shader to rect
opts := &ebiten.DrawRectShaderOptions{}
opts.Uniforms = map[string]interface{}{
    "TintColor": []float32{1.0, 0.5, 0.5, 1.0},
}
opts.Images[0] = sprite // Source image (if shader samples texture)
screen.DrawRectShader(32, 32, shader, opts)

// Apply shader to triangles
opts := &ebiten.DrawTrianglesShaderOptions{}
// ... configure vertices, indices
screen.DrawTrianglesShader(vertices, indices, shader, opts)
```

Shader specialists handle shader code — see ebiten-kage-specialist for shader authoring.

---

## Common Pitfalls

| Pitfall | Consequence | Fix |
|---------|-------------|-----|
| Creating Images every Draw | GPU memory leak | Create once, reuse |
| GeoM order wrong | Wrong transform | Translate last for positioning |
| ColorScale vs ColorM | Deprecated API | Use ColorScale (v2.5+), not ColorM |
| SubImage type assertion | Panic if not Image | Always `.(*ebiten.Image)` |
| Large image count | Draw call overhead | Batch with DrawTrianglesShader |
| Not disposing external images | Memory leak on some platforms | Call Dispose() when done |

---

## Deprecated APIs (v2.5+)

| Deprecated | Replacement |
|------------|-------------|
| `ColorM` field in DrawImageOptions | Use `ColorScale` or `colorm` package |
| `ebiten.Image.DrawImage` with ColorM | Use ColorScale.Scale() |

---

## Reference

- Source: `~/Desktop/source/ebiten/image.go` (Image, DrawImageOptions definitions)
- pkg.go.dev: https://pkg.go.dev/github.com/hajimehoshi/ebiten/v2#Image
```

- [ ] **Step 2: Commit rendering.md**

```bash
git add docs/engine-reference/ebiten/modules/rendering.md
git commit -m "docs: add Ebitengine rendering module reference"
```

---

## Phase 4: Integration Files

### Task 8: Add Ebitengine to setup-engine Skill

**Files:**
- Modify: `.claude/skills/setup-engine/SKILL.md`

- [ ] **Step 1: Read current setup-engine SKILL.md**

Read the file to identify the exact lines to modify for engine selection options.

- [ ] **Step 2: Add Ebitengine to engine selection options**

Find the `AskUserQuestion` section for engine selection and add Ebitengine:

```markdown
- Options: `Godot` / `Unity` / `Unreal Engine 5` / `Ebitengine (Go)` / `Explore further`
```

- [ ] **Step 3: Add Ebitengine handling logic**

Add a new section after the existing engine handling blocks:

```markdown
**If Ebitengine was chosen:**

1. Skip the full decision matrix (user already knows they want Go + 2D)
2. Ask: "Ebitengine v2.9.9 is the current stable. Use this version?"
3. If user confirms, use v2.9.9
4. If user wants different version, use WebSearch to find latest

Update CLAUDE.md with Ebitengine template:

```markdown
- **Engine**: Ebitengine [version]
- **Language**: Go
- **Build System**: Go build (go build ./cmd/game)
- **Asset Pipeline**: EmbedFS or ebitenutil.NewImageFromFile
```

Populate technical-preferences.md:

```markdown
## Engine Specialists
- **Primary**: ebiten-specialist
- **Shader Specialist**: ebiten-kage-specialist
- **UI Specialist**: ebiten-specialist (no dedicated UI — code-centric)

### File Extension Routing
| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
| Game code (.go files) | ebiten-specialist |
| Shader files (.kage) | ebiten-kage-specialist |
| General architecture review | ebiten-specialist |
```
```

- [ ] **Step 4: Commit setup-engine modification**

```bash
git add .claude/skills/setup-engine/SKILL.md
git commit -m "feat: add Ebitengine option to setup-engine skill"
```

---

### Task 9: Add Ebitengine Agents to catalog.yaml

**Files:**
- Modify: `CCGS Skill Testing Framework/catalog.yaml`

- [ ] **Step 1: Read catalog.yaml agents section**

Find the line number where the agents section ends (after unreal specialists).

- [ ] **Step 2: Append Ebitengine agent entries**

Add after the Unreal engine agents section:

```yaml
  # Tier: Engine - Ebitengine
  - name: ebiten-specialist
    spec: CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-specialist.md
    last_spec: ""
    last_spec_result: ""
    category: engine

  - name: ebiten-kage-specialist
    spec: CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-kage-specialist.md
    last_spec: ""
    last_spec_result: ""
    category: engine
```

- [ ] **Step 3: Commit catalog.yaml modification**

```bash
git add "CCGS Skill Testing Framework/catalog.yaml"
git commit -m "feat: add Ebitengine agents to catalog.yaml"
```

---

### Task 10: Add Ebitengine Tier to coordination-rules.md

**Files:**
- Modify: `.claude/docs/coordination-rules.md`

- [ ] **Step 1: Read coordination-rules.md**

Verify no existing Ebitengine references.

- [ ] **Step 2: Add Ebitengine tier documentation**

The coordination-rules.md currently doesn't have an explicit engine tier list section. Add a new section after the Parallel Task Protocol section:

```markdown
## Engine Specialist Tiers

This project supports multiple game engines. Each engine has a primary specialist and sub-specialists:

### Godot Engine
- Primary: `godot-specialist`
- Sub-specialists: `godot-gdscript-specialist`, `godot-csharp-specialist`, `godot-shader-specialist`, `godot-gdextension-specialist`

### Unity Engine
- Primary: `unity-specialist`
- Sub-specialists: `unity-ui-specialist`, `unity-shader-specialist`, `unity-dots-specialist`, `unity-addressables-specialist`

### Unreal Engine
- Primary: `unreal-specialist`
- Sub-specialists: `ue-gas-specialist`, `ue-replication-specialist`, `ue-umg-specialist`, `ue-blueprint-specialist`

### Ebitengine (Go 2D)
- Primary: `ebiten-specialist`
- Sub-specialists: `ebiten-kage-specialist` (Kage shaders)

**Routing:** When code review or implementation involves engine-specific APIs, spawn the appropriate engine specialist based on project configuration.
```

- [ ] **Step 3: Commit coordination-rules.md modification**

```bash
git add .claude/docs/coordination-rules.md
git commit -m "docs: add Ebitengine engine specialist tier"
```

---

## Phase 5: Validation

### Task 11: Validate Implementation

**Files:**
- None (validation only)

- [ ] **Step 1: Verify all files created**

```bash
ls -la .claude/agents/ebiten-specialist.md
ls -la .claude/agents/ebiten-kage-specialist.md
ls -la "CCGS Skill Testing Framework/agents/engine/ebiten/"
ls -la docs/engine-reference/ebiten/
```

Expected: All 7 new files exist.

- [ ] **Step 2: Verify catalog.yaml entries**

```bash
grep -A 5 "ebiten-specialist" "CCGS Skill Testing Framework/catalog.yaml"
```

Expected: Two entries found (ebiten-specialist, ebiten-kage-specialist).

- [ ] **Step 3: Verify setup-engine modification**

```bash
grep "Ebitengine" .claude/skills/setup-engine/SKILL.md
```

Expected: Ebitengine appears in engine options.

- [ ] **Step 4: Manual invocation test**

Invoke ebiten-specialist with a test query to verify agent responds correctly:

```
Ask: "How do I set up a basic Ebitengine Game struct with Update and Draw?"
```

Expected: Agent provides pattern guide, references VERSION.md, asks before writing.

- [ ] **Step 5: Commit any validation fixes**

If validation reveals issues, fix and commit:

```bash
git add -A
git commit -m "fix: validation corrections for Ebitengine support"
```

---

## Summary

| Phase | Tasks | Files |
|-------|-------|-------|
| 1 | 2 | 2 agent definitions |
| 2 | 2 | 2 test specs |
| 3 | 3 | 3 reference docs |
| 4 | 3 | 3 modifications |
| 5 | 1 | Validation only |

**Total:** 11 tasks, 7 new files, 3 modifications

---

## Self-Review

**1. Spec coverage:**
- ✓ Agent hierarchy defined (Task 1, Task 2)
- ✓ Test specs defined (Task 3, Task 4)
- ✓ VERSION.md defined (Task 5)
- ✓ game-loop.md defined (Task 6)
- ✓ rendering.md defined (Task 7)
- ✓ setup-engine integration defined (Task 8)
- ✓ catalog.yaml defined (Task 9)
- ✓ coordination-rules defined (Task 10)
- ✓ Validation defined (Task 11)

**2. Placeholder scan:**
- No TBD/TODO found
- All code blocks contain complete content
- All file paths are exact
- All commit messages specified

**3. Type consistency:**
- Agent names consistent throughout (ebiten-specialist, ebiten-kage-specialist)
- File paths consistent throughout
- VERSION.md values match spec (v2.9.9)