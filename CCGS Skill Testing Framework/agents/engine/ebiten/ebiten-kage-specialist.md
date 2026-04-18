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
