# Ebitengine Support — Design Specification

> **Created**: 2026-04-18
> **Status**: Approved for implementation
> **Scope**: Full parity with existing engine support (Godot/Unity/Unreal)

## Overview

Add Ebitengine (Go 2D game engine) support to Claude Code Game Studios with full parity with existing engine specialists. Ebitengine differs from visual-editor engines (Godot/Unity/Unreal) by being code-centric, Go-only, and 2D-focused.

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Specialist granularity | Primary + Shader (2 agents) | User specified: focus on Ebitengine-specific patterns, not general Go quality. Kage shaders are a distinct language requiring dedicated specialist. |
| setup-engine integration | Minimal (option only) | User selected: add Ebitengine as menu option without full decision matrix. Assumes user already knows they want Go + 2D. |
| Engine reference modules | Game Loop + Rendering (core only) | Essential patterns only. Audio/Input covered by primary specialist. Shader module covered by Kage specialist. |
| Implementation order | Top-down (agents → specs → docs → integration) | Agents are god nodes in architecture. Everything else references them. |

## Architecture

### Agent Hierarchy

```
technical-director → lead-programmer → ebiten-specialist → ebiten-kage-specialist
```

Matches existing Godot/Unity/Unreal three-tier structure.

### Agent: ebiten-specialist (Primary)

**Domain:** Ebitengine-specific patterns, game loop architecture, rendering pipeline, input handling, API usage.

**Delegates to:** ebiten-kage-specialist for Kage shader code.

**Does NOT own:** General Go code quality, idiomatic Go patterns (user specified exclusion).

**Key responsibilities:**
- Guide Game interface implementation (Update/Draw/Layout)
- Review rendering patterns (ebiten.Image, DrawImageOptions, GeoM)
- Advise on fixed timestep vs delta time handling
- Coordinate with gameplay-programmer for game logic architecture

### Agent: ebiten-kage-specialist (Sub-Specialist)

**Domain:** Kage shader language, uniform handling, vertex/fragment functions.

**Reports to:** ebiten-specialist via lead-programmer.

**Key responsibilities:**
- Kage syntax and compilation
- Performance patterns (avoid branching, optimize uniform usage)
- Shader debugging and error diagnosis

## Engine Reference Documentation

### VERSION.md

| Field | Value |
|-------|-------|
| **Engine Version** | Ebitengine v2.9.9 |
| **Source Reference** | `~/Desktop/source/ebiten` (local checkout) |
| **LLM Knowledge Cutoff** | May 2025 |
| **Risk Level** | LOW — v2.9.x patterns stable within training data |

### Module: game-loop.md

**Core pattern (from run.go):**

```go
type Game interface {
    Update() error          // Fixed TPS (60 default)
    Draw(screen *Image)     // Every frame (monitor refresh)
    Layout(outsideW, outsideH int) (screenW, screenH int)
}
```

**Key behaviors documented:**
- Fixed timestep: Update at 1/TPS intervals
- Draw frequency: monitor-dependent (FPS ≠ TPS)
- Layout: called before first Update
- First frame guarantee: Update before Draw

**Common pitfalls:**
- Heavy computation in Draw
- Manual delta time measurement
- Blocking in Update

### Module: rendering.md

**Core types (from image.go):**

```go
type Image struct { ... }  // Premultiplied RGBA

type DrawImageOptions struct {
    GeoM       GeoM        // Transform matrix
    ColorScale ColorScale  // RGBA scale (premultiplied)
    Blend      Blend       // Blend mode
    Filter     Filter      // Nearest or Linear
}
```

**Key patterns documented:**
- Sub-images for sprite sheets
- Offscreen rendering
- Shader rendering (DrawRectShader, DrawTrianglesShader)

**Common pitfalls:**
- Creating Images every frame
- Not disposing external images
- Using deprecated ColorM (v2.5+)

## Test Specifications

### Location Pattern

Follow existing structure in `CCGS Skill Testing Framework/agents/engine/ebiten/`.

### ebiten-specialist Test Cases

| Case | Input | Expected |
|------|-------|----------|
| 1: In-domain | Update/Draw loop structure question | Pattern guide, no raw code |
| 2: Wrong-engine | Godot Node2D request | Redirect with Ebitengine equivalent |
| 3: Version risk | v2.8+ API usage | Check VERSION.md, flag if unverified |
| 4: Shader delegation | Kage shader request | Delegate to kage-specialist |
| 5: Context pass | Offscreen rendering pattern | Reference game-loop.md, explain GeoM scaling |

### ebiten-kage-specialist Test Cases

| Case | Input | Expected |
|------|-------|----------|
| 1: In-domain | Color inversion shader | Provide Kage code, ask before write |
| 2: Wrong-shader | GLSL/Godot shader request | Redirect with Kage equivalent |
| 3: Performance | Loop in fragment shader | Flag risk, suggest alternatives |
| 4: Uniforms | Array of lights | Explain uniform syntax, note limits |
| 5: Error diagnosis | Compilation error | Diagnose common Kage errors |

## Integration Points

### setup-engine Skill (Minimal Mode)

Add Ebitengine to engine selection menu:

```
Options: Godot / Unity / Unreal Engine 5 / Ebitengine (Go) / Explore further
```

On selection:
1. Skip tradeoff matrix (user knows they want Go + 2D)
2. Version lookup: WebSearch or local checkout
3. CLAUDE.md update with Ebitengine template
4. technical-preferences.md routing section populated

### CLAUDE.md Template (Ebitengine)

```markdown
- **Engine**: Ebitengine v2.9.9
- **Language**: Go
- **Build System**: Go build (go build ./cmd/game)
- **Asset Pipeline**: EmbedFS or ebitenutil.NewImageFromFile
```

### technical-preferences.md Routing

```markdown
## Engine Specialists
- **Primary**: ebiten-specialist
- **Shader Specialist**: ebiten-kage-specialist
- **UI Specialist**: ebiten-specialist (code-centric, no dedicated UI)

### File Extension Routing
| .go files | ebiten-specialist |
| .kage files | ebiten-kage-specialist |
```

### coordination-rules.md

Add `ebiten` to engine tier:

```markdown
- **Engine specialists**: godot + subs, unity + subs, unreal + subs, ebiten + ebiten-kage
```

### catalog.yaml

```yaml
- name: ebiten-specialist
  type: agent
  category: engine
  tier: ebiten
  spec: CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-specialist.md

- name: ebiten-kage-specialist
  type: agent
  category: engine
  tier: ebiten
  spec: CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-kage-specialist.md
```

## Implementation Phases

### Phase 1: Specialist Agents

Create:
- `.claude/agents/ebiten-specialist.md`
- `.claude/agents/ebiten-kage-specialist.md`

### Phase 2: Test Specs

Create:
- `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-specialist.md`
- `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-kage-specialist.md`

### Phase 3: Engine Reference Docs

Create:
- `docs/engine-reference/ebiten/VERSION.md`
- `docs/engine-reference/ebiten/modules/game-loop.md`
- `docs/engine-reference/ebiten/modules/rendering.md`

### Phase 4: Integration Files

Modify:
- `.claude/skills/setup-engine/SKILL.md` (add Ebitengine option)
- `CCGS Skill Testing Framework/catalog.yaml` (add entries)
- `.claude/docs/coordination-rules.md` (add tier)

### Phase 5: Validation

- `/skill-test spec ebiten-specialist`
- `/skill-test spec ebiten-kage-specialist`
- Manual invocation test

## File Summary

| Type | Count | Files |
|------|-------|-------|
| Create | 7 | 2 agents + 2 specs + 3 docs |
| Modify | 3 | setup-engine, catalog.yaml, coordination-rules |

## Exclusions (Per User Direction)

- **NOT included:** General Go code quality, idiomatic Go patterns, Go style enforcement
- **Focus only:** Ebitengine-specific APIs, game loop patterns, rendering, Kage shaders

## Risk Notes

- Ebitengine evolves rapidly (~monthly releases)
- v2.9.9 is within LLM training data range (LOW risk)
- Future versions may require VERSION.md refresh via `/setup-engine refresh`