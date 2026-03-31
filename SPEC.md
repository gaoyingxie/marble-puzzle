# 弹珠解谜 (Marble Puzzle) — SPEC

## 1. Concept & Vision

玩家在画布上**画线**，弹珠从起点滚下，物理引擎驱动弹珠沿自己画的线滑行、弹跳，最终落入目标杯即过关。核心乐趣是"我随手画的线居然真的能到"——意外惊喜 + 物理模拟的可信度。风格：蓝图 / 工程图纸感，冷静优雅。

## 2. Design Language

- **Aesthetic**: 工程蓝图风 — 深色背景 + 细网格 + 发光线条，像在设计图纸上实验
- **Color Palette**:
  - Background: `#0d1117`
  - Grid: `#1e2936` (细格), `#2a3a4d` (粗格)
  - Player Line: `#38bdf8` (cyan glow)
  - Marble: `#f97316` (orange) with radial gradient
  - Target Cup: `#22c55e` (green glow)
  - Obstacle: `#64748b` (slate)
  - UI Text: `#e2e8f0`
  - Accent: `#fbbf24` (gold, for score/success)
- **Typography**: `JetBrains Mono` (monospace, fits blueprint theme), fallback: `monospace`
- **Motion**:
  - Marble: realistic physics via Matter.js, subtle motion blur trail
  - Line drawing: instant stroke, no animation
  - Win: cup pulses + particles burst
  - Level transition: fade 300ms

## 3. Layout & Structure

```
┌─────────────────────────────────────────┐
│  Level X         ★ Stars    [Reset]    │  ← Top bar (40px)
├─────────────────────────────────────────┤
│                                         │
│         ← Game Canvas (full) →          │
│                                         │
│  [marble start]                        │
│                                         │
│         obstacles, lines, cup           │
│                                         │
├─────────────────────────────────────────┤
│  ✏️ Draw  |  ▶ Play  |  🗑️ Clear       │  ← Bottom controls (50px)
└─────────────────────────────────────────┘
```

- **States**: DRAW → PLAY → WIN / LOSE → next level or retry
- Responsive: canvas scales to fit viewport, min-width 320px

## 4. Features & Interactions

### Drawing
- Mouse/touch drag draws line segments
- Minimum segment length: 10px (ignore tiny strokes)
- Line thickness: 4px, rounded caps
- Lines are static physics bodies (marble rolls/bounces off them)

### Physics
- Matter.js handles all physics (gravity, collision, friction, restitution)
- Marble: circle body, density 0.001, restitution 0.4, friction 0.05
- Lines: static bodies with friction 0.3
- Gravity: 1.0 (standard)

### Win/Lose
- **Win**: marble center enters cup hitbox → show success overlay → auto-advance to next level after 1.5s
- **Lose**: marble falls off screen (y > canvas height + 50px) → show fail overlay → tap/button to retry
- **Star rating**: 3 stars if marble wins; stars displayed on win overlay

### Controls
- **Draw mode**: mouse down → draw, mouse up → finish segment
- **Play button**: switch to physics simulation, disable drawing
- **Reset button**: stop simulation, restore marble to start position (keep lines)
- **Clear button**: erase all drawn lines, reset to draw mode

### Levels
- 5 built-in levels with increasing difficulty:
  1. Straight drop, one redirect needed
  2. Obstacle between start and cup
  3. Two redirect points needed
  4. Narrow passage
  5. Multiple obstacles, precise line needed
- Level data stored in `levels.js`

## 5. Component Inventory

| Component | Description |
|-----------|-------------|
| `Canvas` | Main game area, handles draw + render |
| `Marble` | Orange circle, Matter.js body |
| `TargetCup` | Green glowing cup shape (CSS/canvas drawn) |
| `Obstacle` | Slate gray static blocks |
| `LineSegment` | Player-drawn cyan line, becomes physics body |
| `PlayBtn` | Cyan outlined button |
| `ResetBtn` | Ghost button |
| `ClearBtn` | Ghost button |
| `WinOverlay` | Centered modal with stars + "Next Level" |
| `LoseOverlay` | Centered modal with "Retry" |

## 6. Technical Approach

- **Single HTML file** (self-contained, easy to deploy to GitHub Pages)
- **Matter.js** via CDN: `https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.19.0/matter.min.js`
- **Rendering**: HTML5 Canvas 2D (custom render loop, not Matter.Render)
- **File structure**:
  ```
  marble-puzzle/
  ├── index.html      # All-in-one: HTML + CSS + JS
  ├── SPEC.md
  └── README.md
  ```
- **GitHub Pages**: serve from `main` branch, `index.html` at root
- **Physics step**: `Matter.Engine.update(engine, delta)` in `requestAnimationFrame` loop
- **Collision detection**: Matter.js events, detect marble vs cup sensor body
