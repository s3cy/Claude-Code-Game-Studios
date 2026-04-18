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
