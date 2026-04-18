# EbitenUI Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add EbitenUI (github.com/ebitenui/ebitenui) library support as a sub-specialist under ebiten-specialist.

**Architecture:** Single sub-specialist (`ebiten-ui-specialist`) handles EbitenUI widgets, layouts, themes, and input layering. Delegates styling/theming to art-director. Integrates with existing Ebitengine support via coordination-rules.md and catalog.yaml updates. Cross-reference ui.md with game-loop.md.

**Tech Stack:** EbitenUI v0.x (third-party Go library), Ebitengine v2.9.9, agent definitions in YAML frontmatter + Markdown, test specs in Markdown, engine reference in Markdown.

---

## File Structure

| File | Purpose |
|------|---------|
| `.claude/agents/ebiten-ui-specialist.md` | Agent definition with frontmatter, domain, collaboration protocol |
| `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md` | Test spec with 5 cases + protocol compliance assertions |
| `docs/engine-reference/ebiten/modules/ui.md` | EbitenUI API reference: integration, widgets, layouts, themes, events |
| `.claude/docs/coordination-rules.md` | Add `ebiten-ui-specialist` to ebiten tier sub-specialists list |
| `CCGS Skill Testing Framework/catalog.yaml` | Add agent entry for ebiten-ui-specialist |
| `docs/engine-reference/ebiten/modules/game-loop.md` | Add EbitenUI integration section with code example |

---

### Task 1: Agent Definition — ebiten-ui-specialist

**Files:**
- Create: `.claude/agents/ebiten-ui-specialist.md`

- [ ] **Step 1: Create the agent definition file**

Write complete agent definition with frontmatter, domain, collaboration protocol, core responsibilities, EbitenUI best practices, delegation map, version awareness, and coordination sections.

```markdown
---
name: ebiten-ui-specialist
description: "The EbitenUI specialist owns all EbitenUI widget implementation: Container hierarchy, layout systems (Anchor/Grid/Row), theme configuration, input layer integration (UIHovered, Layer blocking), and Game interface integration. Delegates visual styling to art-director. Focuses on EbitenUI patterns only — not general Go code quality."
tools: Read, Glob, Grep, Write, Edit, Bash, Task
model: sonnet
maxTurns: 20
---
You are the EbitenUI Specialist for a game project using Ebitengine with EbitenUI (github.com/ebitenui/ebitenui). You own everything related to EbitenUI widget implementation.

## Collaboration Protocol

**You are a collaborative implementer, not an autonomous code generator.** The user approves all architectural decisions and file changes.

### Implementation Workflow

Before writing any code:

1. **Read the design document:**
   - Identify what UI structure is specified
   - Note any layout or widget requirements
   - Flag potential EbitenUI complexity issues

2. **Ask UI architecture questions:**
   - "Should this use AnchorLayout (edge-relative) or GridLayout (row/column)?"
   - "How many widget instances will be needed for this screen?"
   - "The design specifies [modal dialog]. Should it use FullScreen input layer blocking?"
   - "This requires visual styling. Should I consult art-director first?"

3. **Propose UI architecture before implementing:**
   - Show Container hierarchy, widget composition, layout flow
   - Explain WHY you're recommending this layout system
   - Highlight trade-offs: "AnchorLayout is simpler for single-center elements" vs "GridLayout scales better for lists"
   - Ask: "Does this match your expectations? Any changes before I write the code?"

4. **Implement with transparency:**
   - If widget behavior differs from spec, STOP and ask
   - If rules/hooks flag issues, fix them and explain what was wrong
   - If visual styling is needed without art direction, delegate to art-director

5. **Get approval before writing files:**
   - Show the code or a detailed summary
   - Explicitly ask: "May I write this to [filepath(s)]?"
   - For multi-file changes, list all affected files
   - Wait for "yes" before using Write/Edit tools

6. **Offer next steps:**
   - "Should I add event handlers for the button clicks?"
   - "This is ready for art-director review for theme styling"
   - "I notice [potential optimization]. Should I refactor?"

### Collaborative Mindset

- Clarify widget intent before assuming — layouts are hard to change later
- Propose widget hierarchy, don't just implement
- Explain layout trade-offs transparently
- Delegate styling decisions to art-director — visual direction is their domain
- Tests prove it works — offer integration tests with Game loop

## Core Responsibilities
- Guide UI hierarchy implementation (Container, widgets, AddChild pattern)
- Review layout system choices (AnchorLayout, GridLayout, RowLayout, StackedLayout)
- Advise on theme configuration and application
- Coordinate input layer blocking patterns (UIHovered, FullScreen + BlockLower)
- Ensure proper UI.Update()/UI.Draw() integration with Game interface
- Delegate visual styling and theme direction to art-director

## EbitenUI Best Practices to Enforce

### Game Interface Integration
- **Update order:** UI.Update() called before gameplay input processing
- **Draw order:** UI.Draw() called after game world rendering (UI on top)
- **UIHovered check:** Check `!input.UIHovered` before processing gameplay clicks
- **One UI instance:** Exactly one `ebitenui.UI` per application

```go
func (g *Game) Update() error {
    g.ui.Update()  // UI processes input first
    
    if inpututil.IsMouseButtonJustPressed(ebiten.MouseButtonLeft) && !input.UIHovered {
        // Gameplay input only when NOT over UI
        g.handleGameplayClick()
    }
    
    return nil
}

func (g *Game) Draw(screen *ebiten.Image) {
    g.drawGameWorld(screen)  // Game world first
    g.ui.Draw(screen)        // UI overlay on top
}
```

### Container Hierarchy
- Root container uses AnchorLayout or RowLayout for screen positioning
- Child containers use appropriate layouts for their content
- AddChild returns RemoveChildFunc for dynamic widget management
- ContainerOpts.BackgroundImage enables hover tracking (TrackHover)

### Layout Systems
- **AnchorLayout:** Edge-relative positioning (center, top-left, bottom-right)
- **GridLayout:** Row/column arrangement with configurable spacing
- **RowLayout:** Horizontal or vertical stacking
- **StackedLayout:** Layered widgets (only one visible at a time)
- Choose layout based on: number of elements, dynamic vs static, screen positioning

### Widget Construction Pattern
- Functional options pattern: `widget.NewButton(widget.ButtonOpts.Text(...), ...)`
- Create widgets once, store references for event handling
- Don't create widgets in Update/Draw — create in initialization
- Use widget references to update state (text labels, progress bars)

### Input Layer Blocking
- `input.Layer` with `FullScreen: true, BlockLower: true` for modal dialogs
- `input.UIHovered` automatically set when cursor over TrackHover-enabled widgets
- BackgroundImage on Container enables hover tracking by default
- Disable with `widget.WidgetOpts.TrackHover(false)` if needed

### Theme System
- Theme configuration: colors, fonts, images per widget state
- Apply theme via `UI.PrimaryTheme` or `widget.WidgetOpts.SetTheme()`
- **Delegate theme design to art-director** — implement technical config after visual direction
- ThemeBasicDark and ThemeBasicLight provided as defaults

### Event Handling
- Button: `ClickedEvent.AddHandler(func(args interface{}) { ... })`
- Checkbox: `StateChangedEvent` for toggle state changes
- List: `SelectedEvent` for entry selection
- Deferred event execution: `event.ExecuteDeferred()` called by UI.Update()

## Common Pitfalls to Flag
- Creating widgets every frame (memory leak, performance hit)
- Not calling UI.Update() before gameplay input (input not processed)
- Not calling UI.Draw() after game world (UI renders behind game)
- Heavy computation in widget event handlers (blocks UI responsiveness)
- Not checking UIHovered for gameplay clicks (clicks pass through UI)
- Missing BackgroundImage for hover tracking (UIHovered not set)
- Applying theme without art-director sign-off (visual inconsistency)

## Delegation Map

**Reports to**: `ebiten-specialist` (via primary engine specialist hierarchy)

**Delegates to**:
- `art-director` for visual styling, theme color/font choices, UI aesthetic direction

**Coordinates with**:
- `ebiten-specialist` for Game interface integration (Update/Draw placement)
- `ui-programmer` for general UI patterns (not EbitenUI-specific)
- `ux-designer` for interaction design and accessibility
- `accessibility-specialist` for keyboard navigation and screen reader support

## What This Agent Must NOT Do
- Make visual styling decisions (delegate to art-director)
- Provide general Go code quality advice (outside scope — focus on EbitenUI patterns)
- Implement gameplay logic in UI event handlers (dispatch to game systems)
- Create multiple UI instances (one UI per application)
- Skip UI.Update() or UI.Draw() integration checks

## Sub-Specialist Status

This agent is a sub-specialist under `ebiten-specialist`. It does not spawn further sub-specialists.

## Version Awareness

**CRITICAL**: EbitenUI is a third-party library that evolves independently of Ebitengine. Before suggesting EbitenUI API code, you MUST:

1. Check `docs/engine-reference/ebiten/modules/ui.md` for current widget/layout patterns
2. Check `docs/engine-reference/ebiten/modules/game-loop.md` for integration patterns
3. Use WebSearch if suggesting widgets not in the reference docs (new widgets may exist)

When in doubt, prefer the API documented in the reference files over training data.

## When Consulted

Always involve this agent when:
- Designing UI screens for an Ebitengine project
- Choosing between layout systems for widget positioning
- Setting up modal dialogs with input blocking
- Integrating UI with the Game interface Update/Draw cycle
- Implementing button/checkbox/list event handlers
- Configuring themes (after art-director provides visual direction)
- Debugging UI input blocking or hover detection
```

- [ ] **Step 2: Verify file was created correctly**

Run: `head -10 .claude/agents/ebiten-ui-specialist.md`
Expected: Shows frontmatter with `name: ebiten-ui-specialist`

- [ ] **Step 3: Commit**

```bash
git add .claude/agents/ebiten-ui-specialist.md
git commit -m "feat: add ebiten-ui-specialist agent definition"
```

---

### Task 2: Test Specification

**Files:**
- Create: `CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md`

- [ ] **Step 1: Create the test spec file**

Write complete test spec with static assertions, 5 test cases with expected behavior and assertions, protocol compliance section, and coverage notes.

```markdown
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
```

- [ ] **Step 2: Verify file was created correctly**

Run: `head -20 "CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md"`
Expected: Shows agent summary and static assertions

- [ ] **Step 3: Commit**

```bash
git add "CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md"
git commit -m "feat: add ebiten-ui-specialist test specification"
```

---

### Task 3: Engine Reference — ui.md

**Files:**
- Create: `docs/engine-reference/ebiten/modules/ui.md`

- [ ] **Step 1: Create the ui.md module**

Write complete engine reference for EbitenUI with integration patterns, widget hierarchy, layout systems, theme system, event handling, and references.

```markdown
# EbitenUI — UI Widget Library

## Library Information

| Field | Value |
|-------|-------|
| **Library** | github.com/ebitenui/ebitenui |
| **Scope** | Third-party library (not Ebitengine core) |
| **Purpose** | Retained-mode widget system for Ebitengine games |
| **Source** | `~/Desktop/source/ebitenui` |

---

## Integration with Game Interface

EbitenUI requires integration with Ebitengine's Game interface. The UI must be updated and drawn in the correct order.

### Update/Draw Placement

```go
type Game struct {
    ui *ebitenui.UI
}

func (g *Game) Update() error {
    // 1. UI processes input first
    g.ui.Update()
    
    // 2. Check UIHovered before gameplay input
    if inpututil.IsMouseButtonJustPressed(ebiten.MouseButtonLeft) && !input.UIHovered {
        // Gameplay input only when NOT over UI
        g.handleGameplayClick()
    }
    
    // 3. Update game state
    g.updateGameplay()
    
    return nil
}

func (g *Game) Draw(screen *ebiten.Image) {
    // 1. Draw game world first
    g.drawGameWorld(screen)
    
    // 2. Draw UI on top (overlay)
    g.ui.Draw(screen)
}

func (g *Game) Layout(outsideW, outsideH int) (int, int) {
    // Return window size for dynamic UI sizing
    return outsideW, outsideH
}
```

### UIHovered Flag

The `input.UIHovered` boolean indicates whether the cursor is over any UI widget:

```go
import "github.com/ebitenui/ebitenui/input"

if input.UIHovered {
    // Cursor is over UI — don't process gameplay clicks
} else {
    // Cursor is over game world — allow gameplay interaction
}
```

**Enabling hover tracking:** Widgets with `BackgroundImage` automatically track hover via `TrackHover: true`.

**Disabling hover tracking:** Use `widget.WidgetOpts.TrackHover(false)` for non-interactive elements.

### Input Layer Blocking

For modal dialogs that block all gameplay input:

```go
import "github.com/ebitenui/ebitenui/input"

// Create modal window
modalWindow := widget.NewWindow(
    widget.WindowOpts.Contents(modalContainer),
    widget.WindowOpts.Modal(),  // Built-in modal behavior
    widget.WindowOpts.Location(rect),
)

// Manual input layer (if not using Window)
layer := &input.Layer{
    DebugLabel: "modal",
    EventTypes: input.LayerEventTypeAll,
    BlockLower: true,   // Block events to lower layers
    FullScreen: true,   // Cover entire screen
    RectFunc:   nil,    // Not needed for FullScreen
}
input.AddLayer(layer)
```

---

## Widget Hierarchy

### Container

Root of all widget trees. Holds child widgets and applies layout.

```go
rootContainer := widget.NewContainer(
    widget.ContainerOpts.Layout(widget.NewAnchorLayout()),
    widget.ContainerOpts.BackgroundImage(image),  // Enables hover tracking
)

// Add child widgets
button := widget.NewButton(...)
rootContainer.AddChild(button)

// Remove child widgets
removeFunc := rootContainer.AddChild(tempWidget)
removeFunc()  // Removes tempWidget
```

**Container options:**
- `ContainerOpts.Layout(layout)` — Layout system for children
- `ContainerOpts.BackgroundImage(nineSlice)` — Background image (enables hover)
- `ContainerOpts.AutoDisableChildren()` — Disable children when container disabled

### Core Widgets

#### Button

```go
button := widget.NewButton(
    widget.ButtonOpts.Text("Start Game", fontFace, color),
    widget.ButtonOpts.Image(&widget.ButtonImage{
        Idle:     idleImage,   // NineSlice for button background
        Hover:    hoverImage,
        Pressed:  pressedImage,
    }),
    widget.ButtonOpts.TextPadding(widget.NewInsetsSimple(10)),
)

// Handle click
button.ClickedEvent.AddHandler(func(args interface{}) {
    // Button was clicked
})
```

#### Checkbox

```go
checkbox := widget.NewCheckbox(
    widget.CheckboxOpts.Text("Enable Sound", fontFace, color),
    widget.CheckboxOpts.Image(&widget.CheckboxImage{
        Checked:   checkedImage,
        Unchecked: uncheckedImage,
    }),
)

// Handle state change
checkbox.StateChangedEvent.AddHandler(func(args interface{}) {
    if checkbox.State() == widget.CheckboxChecked {
        // Enable sound
    }
})
```

#### List

```go
list := widget.NewList(
    widget.ListOpts.Entries([]interface{}{"Option A", "Option B", "Option C"}),
    widget.ListOpts.EntryLabelFunc(func(e interface{}) string {
        return e.(string)
    }),
    widget.ListOpts.ScrollContainerOpts(
        widget.ScrollContainerOpts.Image(scrollImage),
    ),
)

// Handle selection
list.SelectedEvent.AddHandler(func(args interface{}) {
    selectedEntry := list.SelectedEntry()
})
```

#### ScrollContainer

```go
scroll := widget.NewScrollContainer(
    widget.ScrollContainerOpts.Content(contentContainer),
    widget.ScrollContainerOpts.Image(&widget.ScrollContainerImage{
        Idle: trackImage,
    }),
)
```

#### TextInput

```go
textInput := widget.NewTextInput(
    widget.TextInputOpts.WidgetOpts(
        widget.WidgetOpts.MinSize(200, 30),
    ),
    widget.TextInputOpts.Image(&widget.TextInputImage{
        Idle: inputImage,
    }),
    widget.TextInputOpts.Placeholder("Enter name..."),
)

// Get text
text := textInput.GetText()
```

#### TextArea (Multi-line)

```go
textArea := widget.NewTextArea(
    widget.TextAreaOpts.Text("Initial content", fontFace, color),
    widget.TextAreaOpts.ScrollContainerOpts(...),
)

// Get content
content := textArea.GetText()
```

#### ProgressBar

```go
progress := widget.NewProgressBar(
    widget.ProgressBarOpts.Values(0, 100, 50),  // min, max, current
    widget.ProgressBarOpts.Images(trackImage, fillImage),
    widget.ProgressBarOpts.WidgetOpts(
        widget.WidgetOpts.MinSize(200, 20),
    ),
)

// Update progress
progress.SetCurrent(75)
```

---

## Layout Systems

### AnchorLayout

Positions widgets relative to container edges. Best for single elements or simple positioning.

```go
layout := widget.NewAnchorLayout(
    widget.AnchorLayoutOpts.Padding(widget.NewInsetsSimple(10)),
)

// Widget positioning via LayoutData
button := widget.NewButton(
    widget.ButtonOpts.WidgetOpts(
        widget.WidgetOpts.LayoutData(widget.AnchorLayoutData{
            HorizontalPosition: widget.AnchorLayoutPositionCenter,
            VerticalPosition:   widget.AnchorLayoutPositionCenter,
            StretchHorizontal:  false,
            StretchVertical:    false,
        }),
    ),
)
```

**Positions:** Start (left/top), Center, End (right/bottom)

### GridLayout

Arranges widgets in rows and columns. Best for grids, tables, inventory screens.

```go
layout := widget.NewGridLayout(
    widget.GridLayoutOpts.Columns(3),
    widget.GridLayoutOpts.Spacing(10, 10),
    widget.GridLayoutOpts.Padding(widget.NewInsetsSimple(5)),
)

// Widget positioning via LayoutData
widget.NewButton(
    widget.ButtonOpts.WidgetOpts(
        widget.WidgetOpts.LayoutData(widget.GridLayoutData{
            HorizontalPosition: widget.GridLayoutPositionCenter,
            VerticalPosition:   widget.GridLayoutPositionCenter,
        }),
    ),
)
```

### RowLayout

Stacks widgets horizontally or vertically. Best for button bars, toolbars, menu lists.

```go
layout := widget.NewRowLayout(
    widget.RowLayoutOpts.Direction(widget.DirectionHorizontal),
    widget.RowLayoutOpts.Spacing(10),
    widget.RowLayoutOpts.Padding(widget.NewInsetsSimple(5)),
)
```

**Directions:** Horizontal, Vertical

### StackedLayout

Layers widgets with only one visible at a time. Best for tab books, flipbooks.

```go
layout := widget.NewStackedLayout()

// Only one child visible at a time
flipbook := widget.NewFlipBook(
    widget.FlipBookOpts.Layout(layout),
)
flipbook.Show(widgetA)  // Shows widgetA, hides others
```

---

## Theme System

### Theme Configuration

Themes define colors, fonts, and images for widget states.

```go
theme := widget.NewTheme(
    widget.ThemeOpts.ButtonImage(&widget.ButtonImage{
        Idle:    idleNineSlice,
        Hover:   hoverNineSlice,
        Pressed: pressedNineSlice,
    }),
    widget.ThemeOpts.ButtonTextColor(&widget.ButtonTextColor{
        Idle:   color.White,
        Hover:  color.RGBA{255, 255, 0, 255},
        Pressed: color.RGBA{200, 200, 0, 255},
    }),
)

// Apply to UI
ui := &ebitenui.UI{
    Container:      rootContainer,
    PrimaryTheme:   theme,
}
```

### Built-in Themes

- `themes.ThemeBasicDark` — Dark background, light text
- `themes.ThemeBasicLight` — Light background, dark text

```go
import "github.com/ebitenui/ebitenui/themes"

ui.PrimaryTheme = themes.ThemeBasicDark()
```

---

## Event Handling

### Widget Events

All widgets fire events for user interaction:

```go
// Button click
button.ClickedEvent.AddHandler(func(args interface{}) {
    fmt.Println("Button clicked")
})

// Checkbox toggle
checkbox.StateChangedEvent.AddHandler(func(args interface{}) {
    fmt.Println("Checkbox toggled")
})

// List selection
list.SelectedEvent.AddHandler(func(args interface{}) {
    fmt.Println("List entry selected")
})

// Text input change
textInput.TextChangedEvent.AddHandler(func(args interface{}) {
    newText := textInput.GetText()
})
```

### Deferred Event System

Events are processed via deferred execution. UI.Update() calls `event.ExecuteDeferred()` automatically.

```go
// Manual deferred action (rarely needed)
event.AddDeferredAction(func() {
    // Runs after current frame's events processed
})
```

---

## Common Patterns

### Main Menu Screen

```go
func createMainMenu(ui *ebitenui.UI, fontFace text.Face) {
    rootContainer := widget.NewContainer(
        widget.ContainerOpts.Layout(widget.NewAnchorLayout()),
    )
    
    buttonContainer := widget.NewContainer(
        widget.ContainerOpts.Layout(widget.NewRowLayout(
            widget.RowLayoutOpts.Direction(widget.DirectionVertical),
            widget.RowLayoutOpts.Spacing(20),
        )),
        widget.ContainerOpts.WidgetOpts(
            widget.WidgetOpts.LayoutData(widget.AnchorLayoutData{
                HorizontalPosition: widget.AnchorLayoutPositionCenter,
                VerticalPosition:   widget.AnchorLayoutPositionCenter,
            }),
        ),
    )
    
    startBtn := widget.NewButton(
        widget.ButtonOpts.Text("Start", fontFace, color.White),
        widget.ButtonOpts.ClickedHandler(func(args interface{}) {
            // Start game
        }),
    )
    buttonContainer.AddChild(startBtn)
    
    quitBtn := widget.NewButton(
        widget.ButtonOpts.Text("Quit", fontFace, color.White),
        widget.ButtonOpts.ClickedHandler(func(args interface{}) {
            // Quit game
        }),
    )
    buttonContainer.AddChild(quitBtn)
    
    rootContainer.AddChild(buttonContainer)
    ui.Container = rootContainer
}
```

### HUD Overlay

```go
func createHUD(ui *ebitenui.UI) {
    // HUD at top of screen, doesn't block gameplay
    hudContainer := widget.NewContainer(
        widget.ContainerOpts.Layout(widget.NewAnchorLayout()),
        widget.ContainerOpts.WidgetOpts(
            widget.WidgetOpts.LayoutData(widget.AnchorLayoutData{
                VerticalPosition:   widget.AnchorLayoutPositionStart,
                StretchHorizontal:  true,
            }),
            widget.WidgetOpts.TrackHover(false),  // Don't block gameplay
        ),
    )
    
    healthBar := widget.NewProgressBar(...)
    hudContainer.AddChild(healthBar)
    
    ui.Container = hudContainer
}
```

---

## Common Pitfalls

| Pitfall | Consequence | Fix |
|---------|-------------|-----|
| Creating widgets in Update/Draw | Memory leak, performance hit | Create once in initialization |
| Not calling UI.Update() | Input not processed | Call at start of Game.Update() |
| Not calling UI.Draw() after game world | UI behind game | Call at end of Game.Draw() |
| Not checking UIHovered | Clicks pass through UI | Check `!input.UIHovered` for gameplay |
| Heavy computation in event handler | UI freezes | Dispatch to game system, don't block |
| Missing BackgroundImage | Hover not tracked | Add BackgroundImage or TrackHover(true) |
| Multiple UI instances | Input conflicts | Exactly one UI per application |

---

## Reference

- Library: https://github.com/ebitenui/ebitenui
- Docs: https://ebitenui.github.io
- pkg.go.dev: https://pkg.go.dev/github.com/ebitenui/ebitenui
- Examples: `~/Desktop/source/ebitenui/_examples/`
- Integration: See `game-loop.md` for Game interface patterns
```

- [ ] **Step 2: Verify file was created correctly**

Run: `head -30 docs/engine-reference/ebiten/modules/ui.md`
Expected: Shows library information table and integration section

- [ ] **Step 3: Commit**

```bash
git add docs/engine-reference/ebiten/modules/ui.md
git commit -m "docs: add EbitenUI engine reference module"
```

---

### Task 4: Integration — coordination-rules.md

**Files:**
- Modify: `.claude/docs/coordination-rules.md:91-93`

- [ ] **Step 1: Update Ebitengine sub-specialists list**

Locate line 91-93 showing:
```markdown
### Ebitengine (Go 2D)
- Primary: `ebiten-specialist`
- Sub-specialists: `ebiten-kage-specialist` (Kage shaders)
```

Change to:
```markdown
### Ebitengine (Go 2D)
- Primary: `ebiten-specialist`
- Sub-specialists: `ebiten-ui-specialist`, `ebiten-kage-specialist` (Kage shaders)
```

- [ ] **Step 2: Verify modification**

Run: `grep -A2 "Ebitengine (Go 2D)" .claude/docs/coordination-rules.md`
Expected: Shows both sub-specialists (ebiten-ui-specialist, ebiten-kage-specialist)

- [ ] **Step 3: Commit**

```bash
git add .claude/docs/coordination-rules.md
git commit -m "docs: add ebiten-ui-specialist to coordination rules"
```

---

### Task 5: Integration — catalog.yaml

**Files:**
- Modify: `CCGS Skill Testing Framework/catalog.yaml:1042-1046`

- [ ] **Step 1: Add catalog entry after ebiten-kage-specialist**

Locate lines 1042-1046 showing:
```yaml
  - name: ebiten-kage-specialist
    spec: CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-kage-specialist.md
    last_spec: ""
    last_spec_result: ""
    category: engine
```

Insert new entry after line 1046:
```yaml
  - name: ebiten-kage-specialist
    spec: CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-kage-specialist.md
    last_spec: ""
    last_spec_result: ""
    category: engine

  - name: ebiten-ui-specialist
    spec: CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md
    last_spec: ""
    last_spec_result: ""
    category: engine
```

- [ ] **Step 2: Verify modification**

Run: `grep -A5 "ebiten-ui-specialist" "CCGS Skill Testing Framework/catalog.yaml"`
Expected: Shows entry with spec path and category: engine

- [ ] **Step 3: Commit**

```bash
git add "CCGS Skill Testing Framework/catalog.yaml"
git commit -m "feat: add ebiten-ui-specialist to catalog.yaml"
```

---

### Task 6: Cross-reference — game-loop.md

**Files:**
- Modify: `docs/engine-reference/ebiten/modules/game-loop.md:235` (append section)

- [ ] **Step 1: Add EbitenUI integration section at end**

Append after the Reference section (line 235):

```markdown
---

## EbitenUI Integration

When using EbitenUI (github.com/ebitenui/ebitenui) for UI widgets, integrate the UI into the Game interface:

```go
import "github.com/ebitenui/ebitenui"
import "github.com/ebitenui/ebitenui/input"

type Game struct {
    ui *ebitenui.UI
}

func (g *Game) Update() error {
    // UI first - processes input and events
    g.ui.Update()
    
    // Check if cursor is over UI before gameplay input
    if !input.UIHovered {
        // Gameplay input only when NOT over UI
        if inpututil.IsMouseButtonJustPressed(ebiten.MouseButtonLeft) {
            g.handleGameplayClick()
        }
    }
    
    return nil
}

func (g *Game) Draw(screen *ebiten.Image) {
    // Game world first (background)
    g.drawGameWorld(screen)
    
    // UI on top (overlay)
    g.ui.Draw(screen)
}
```

**Key points:**
- Call `ui.Update()` at the **start** of Update() — UI processes input first
- Check `input.UIHovered` before gameplay clicks — prevents click-through
- Call `ui.Draw()` at the **end** of Draw() — UI renders on top of game world
- Exactly one `ebitenui.UI` instance per application

**Modal dialogs:**
For modal UI that blocks all gameplay input, use input Layer blocking:

```go
layer := &input.Layer{
    DebugLabel: "modal",
    EventTypes: input.LayerEventTypeAll,
    BlockLower: true,
    FullScreen: true,
}
```

See `ui.md` for complete widget and layout documentation.
```

- [ ] **Step 2: Verify modification**

Run: `tail -30 docs/engine-reference/ebiten/modules/game-loop.md`
Expected: Shows new EbitenUI Integration section

- [ ] **Step 3: Commit**

```bash
git add docs/engine-reference/ebiten/modules/game-loop.md
git commit -m "docs: add EbitenUI integration section to game-loop.md"
```

---

### Task 7: Validation — Git Status

**Files:**
- Verify: Git working tree

- [ ] **Step 1: Check git status**

Run: `git status --short`
Expected: No uncommitted changes (all files committed)

- [ ] **Step 2: Check commit count**

Run: `git log --oneline | head -10`
Expected: Shows 6 new commits for this implementation

- [ ] **Step 3: Verify all files exist**

Run: `ls -la .claude/agents/ebiten-ui-specialist.md "CCGS Skill Testing Framework/agents/engine/ebiten/ebiten-ui-specialist.md" docs/engine-reference/ebiten/modules/ui.md`
Expected: All three new files exist

---

## Self-Review

**1. Spec coverage:**
- Agent hierarchy (sub-specialist under ebiten-specialist) → Task 1, Task 4
- Delegates styling to art-director → Task 1 (agent definition section)
- Test spec with 5 cases → Task 2
- Engine reference ui.md → Task 3
- Integration files (coordination-rules, catalog) → Task 4, Task 5
- Cross-reference game-loop.md → Task 6
- All requirements covered ✅

**2. Placeholder scan:**
- No TBD, TODO, or "implement later"
- No "add appropriate error handling"
- No "similar to Task N" without full content
- All code steps show complete code
- All file paths are exact
- No placeholders found ✅

**3. Type consistency:**
- `ebitenui.UI` used consistently across tasks
- `input.UIHovered` used consistently
- Widget names (Button, Checkbox, List) consistent
- Layout names (AnchorLayout, GridLayout, RowLayout) consistent
- All types match across tasks ✅