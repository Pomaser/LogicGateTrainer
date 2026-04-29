# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Browser-based logic gate training game. Single file: `index.html` — no build step, no dependencies, open directly via `file://` in a browser.

## Verifying changes

```bash
# Syntax check the embedded JS
node --input-type=module <<'EOF'
import { readFileSync } from 'fs';
const html = readFileSync('index.html', 'utf8');
const m = html.match(/<script>([\s\S]*?)<\/script>/);
new Function(m[1]);
console.log('OK');
EOF

# Check that no level evaluates to true with all-false inputs
node <<'EOF'
# (paste the level-check script from the session)
EOF
```

## Architecture

Everything lives in one `<script>` block, organized in this order:

| Section | Role |
|---|---|
| Constants | Canvas size, gate geometry (`GATE_W`, `CURSOR_R`, `SPINNER_GAP`…), color palette (`C.*`) |
| `LEVELS[]` | 10 pre-defined level configs: `circuit`, `timeLimit`, `cursorPause`, `cm` (complexity multiplier) |
| `evaluateCircuit()` | Evaluates a gate DAG. Propagates `null` for any undetermined input — gates with an unset input return `null` (not `true`/`false`). |
| Layout engine | `computeLayout()` assigns column-based X positions and stacked Y positions with a fixed `GATE_GAP` between gates. Gate height is derived from `CURSOR_R` and `SPINNER_GAP` so spinner rings never overlap. |
| Renderer | `renderFrame()` → `drawTimeBar`, `drawStatusLine`, `drawWires`, `drawGates`, `drawUserPins`, overlay. Wires are colored green/red/gray based on determined/undetermined value. |
| Game state machine | Phases: `PLAYING` → `LEVEL_COMPLETE` → (auto-advance after 2.5 s) → `PLAYING` / `GAME_OVER` |
| Input handler | Keys `1`/`T` = True, `0`/`F` = False. Sets current input and immediately advances cursor. Any game key during `LEVEL_COMPLETE` skips the countdown. |

## Key data shapes

```js
// Circuit gate (in topological order — sources before consumers)
{ id: string, type: 'AND'|'OR', inputs: [{ source: 'user'|gateId, negated: bool }] }

// User pin (derived by computeLayout, cursor iterates these)
{ gateId, pinIndex, x, y, negated }

// inputMap key
`${gateId}:${pinIndex}`   // e.g. "g0:1"
```

## Level completion rules

A level completes only when **both** conditions hold:
1. All user pins have an explicit value in `inputMap` (no `?` remaining).
2. `evaluateCircuit()` returns `output === true`.

## Scoring

```
levelScore = floor(1000 × cm × (timeRemaining / timeLimit) × cyclePenalty)
cyclePenalty: 0 cycles → 1.0 | 1 → 0.8 | 2 → 0.6 | ≥3 → 0.4
```

## Gate rendering (IEC format)

- AND: rectangle with `&` in the **upper-right corner**
- OR: rectangle with `≥1` in the **upper-right corner**
- Negated inputs: open circle (bubble) drawn on the wire near the gate input
- Active input shown with a countdown arc spinner (shrinks clockwise from 12 o'clock)

## Adding or modifying levels

Add an entry to `LEVELS[]`. Verify:
- Gates are listed in **topological order** (all source gates before the gates that consume them).
- `outputGateId` matches the last gate's `id`.
- With all inputs `false`, the circuit output is `false` (run the level-check script above).
- At least one input assignment produces output `true`.
