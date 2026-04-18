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
