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
