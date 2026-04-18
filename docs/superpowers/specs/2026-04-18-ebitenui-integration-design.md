# EbitenUI Integration — Design Specification

> **Created**: 2026-04-18
> **Status**: Approved for implementation
> **Scope**: Add EbitenUI library support as sub-specialist under ebiten-specialist

## Overview

Add EbitenUI (github.com/ebitenui/ebitenui) support to Claude Code Game Studios as a sub-specialist under ebiten-specialist. EbitenUI is a retained-mode widget library for Ebitengine, providing buttons, lists, layouts, and theme systems.

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Specialist type | Sub-specialist under ebiten-specialist | Follows Unity/Unreal pattern (unity-ui-specialist, ue-umg-specialist) |
| Styling/theming | Delegate to art-director | Visual direction is art domain; UI specialist implements technical theme config |
| Input handling | UI specialist owns UI input layering | EbitenUI's UIHovered/Layer system is library-specific, not engine-level |
| No input specialist | N/A | Input APIs stay with ebiten-specialist; gameplay input with gameplay-programmer; accessibility with accessibility-specialist |
| Engine reference | New ui.md module | Cross-referenced with game-loop.md for integration patterns |

## Agent Architecture

### Hierarchy

```
ebiten-specialist → ebiten-ui-specialist (widgets, layouts, themes, input layering)
                   → ebiten-kage-specialist (Kage shaders)
```

### Agent: ebiten-ui-specialist

**Domain:**
- EbitenUI widget hierarchy (Container, Button, List, ScrollContainer, etc.)
- Layout systems (AnchorLayout, GridLayout, RowLayout)
- Theme configuration and application
- Input layer integration (UIHovered, Layer blocking)
- Game loop integration (Update/Draw placement)

**Delegates to:**
- `art-director` for visual styling and theme direction

**Does NOT own:**
- General Go code quality (excluded, same as ebiten-specialist)
- Engine-level input APIs (ebiten-specialist)
- Gameplay input logic (gameplay-programmer)
- Accessibility input patterns (accessibility-specialist)

**Coordination flow:**
- UI widget/layout questions → handled directly
- UI visual style questions → consult art-director first
- UI blocking gameplay → provide UIHovered/Layer patterns
- Integration with Game interface → reference game-loop.md and ui.md

## Engine Reference Documentation

### Module: ui.md

**Location:** `docs/engine-reference/ebiten/modules/ui.md`

**Structure:**

```markdown
# EbitenUI — UI Widget Library

> Library: github.com/ebitenui/ebitenui
> Version: [current at implementation]
> Scope: Third-party library

## Integration with Game Interface
- Update/Draw placement pattern
- UIHovered flag usage
- Input layer blocking (FullScreen + BlockLower)

## Widget Hierarchy
- Container (root, AddChild)
- Core widgets (Button, Checkbox, List, ScrollContainer, TextInput, TextArea)

## Layout Systems
- AnchorLayout (edge-relative positioning)
- GridLayout (row/column)
- RowLayout (horizontal/vertical stacking)

## Theme System
- Theme configuration and application

## Event Handling
- Widget events (ClickedEvent, PressedEvent)
- Deferred event system
```

**Cross-reference:**
- `game-loop.md` → references `ui.md` for integration pattern
- `ui.md` → references `game-loop.md` for Update/Draw context

## Test Specifications

### Location

`CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md`

### Test Cases

| Case | Input | Expected |
|------|-------|----------|
| 1: In-domain | "How do I create a main menu with buttons?" | Widget hierarchy guide, layout pattern, ask before write |
| 2: Wrong-library | "Use Fyne for this UI" | Redirect to EbitenUI, explain difference |
| 3: Input blocking | "Modal dialog blocks gameplay clicks" | UIHovered + Layer pattern with FullScreen/BlockLower |
| 4: Styling delegation | "Design a theme for medieval RPG UI" | Delegate to art-director, implement after art direction |
| 5: Integration placement | "Where does ui.Update() go?" | Reference game-loop.md, show integration order |

### Protocol Compliance

- Stays within EbitenUI domain (widgets, layouts, input layering)
- Redirects styling/theming to art-director
- References engine reference docs (ui.md, game-loop.md)
- Does NOT provide general Go code quality advice
- Follows collaborative protocol (ask → draft → approve)

## Integration Points

### coordination-rules.md

Add `ebitenui` to ebiten tier:

```markdown
- Sub-specialists: `ebiten-ui-specialist`, `ebiten-kage-specialist`
```

### catalog.yaml

Add entry:

```yaml
- name: ebiten-ui-specialist
  type: agent
  category: engine
  tier: ebiten
  spec: CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md
```

### game-loop.md cross-reference

Add section:

```markdown
## EbitenUI Integration

When using EbitenUI (github.com/ebitenui/ebitenui), the UI must be integrated
into the Game interface:

```go
func (g *Game) Update() error {
    g.ui.Update()  // UI first - processes input
    if !input.UIHovered {
        // gameplay input only when not over UI
    }
    return nil
}

func (g *Game) Draw(screen *ebiten.Image) {
    g.drawGameWorld(screen)  // game world first
    g.ui.Draw(screen)        // UI on top
}
```

See `ui.md` for widget/layout patterns.
```

## Implementation Phases

### Phase 1: Agent Definition

Create: `.claude/agents/ebiten-ui-specialist.md`

### Phase 2: Test Spec

Create: `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md`

### Phase 3: Engine Reference

Create: `docs/engine-reference/ebiten/modules/ui.md`

### Phase 4: Integration Files

Modify:
- `.claude/docs/coordination-rules.md` (add sub-specialist)
- `CCGS Skill Testing Framework/catalog.yaml` (add entry)

### Phase 5: Cross-reference

Modify: `docs/engine-reference/ebiten/modules/game-loop.md` (add EbitenUI integration section)

## File Summary

| Type | Count | Files |
|------|-------|-------|
| Create | 3 | 1 agent + 1 test spec + 1 engine reference |
| Modify | 3 | coordination-rules.md, catalog.yaml, game-loop.md |

## Exclusions

- NOT included: General Go code quality, idiomatic Go patterns
- Focus only: EbitenUI-specific widgets, layouts, themes, input layer integration

## Source Reference

- EbitenUI source: `~/Desktop/source/ebitenui`
- Key files: `ui.go`, `widget/*.go`, `input/*.go`, `themes/*.go`