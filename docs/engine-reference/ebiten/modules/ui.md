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

---

## Widget Testing

For automated EbitenUI widget testing, invoke the `using-autoebiten` skill. Autoui provides:

- **Widget querying by coordinates, attributes, or XPath** — Locate widgets without hardcoded positions
- **Widget method invocation** — Click, SetText, and other actions via CLI or testkit
- **Integration with testkit tests** — Combine widget interaction with tick-based assertions

See the skill's autoui reference docs:
- `autoui-commands.md` — CLI commands for widget interaction
- `autoui-reference.md` — XML format, ae tags, XPath queries
