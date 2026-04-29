# Logic Gate Trainer

A browser-based game for practising binary logic gates. No build step, no dependencies — open `index.html` directly in any browser via `file://`.

## How to play

- The cursor cycles through input pins automatically. Set the current pin's value before it advances.
- **1** / **T** — set input to True
- **0** / **F** — set input to False
- Complete a level by setting all inputs so the circuit output equals **1**.
- On Game Over: **Enter** / **Space** restarts, **S** saves a PNG screenshot.

## Levels

20 levels of increasing difficulty:

| Levels | Content |
|--------|---------|
| 1–3 | Single AND / OR gate, up to 3 inputs, negations |
| 4–6 | 2 gates in series / parallel, negated inputs |
| 7–10 | 3–5 AND/OR gates, growing negation count |
| 11–12 | Introduction to NAND and NOR (1 gate) |
| 13 | NAND with 3 inputs + negation |
| 14–15 | 2–3 NAND/NOR combinations |
| 16–18 | Mixed AND/OR/NAND/NOR (2–3 gates) |
| 19–20 | 4-gate complex mixed circuits |

## Scoring

```
score = floor(1000 × cm × (timeRemaining / timeLimit) × cyclePenalty)
```

`cm` is a per-level complexity multiplier (1.0 – 8.5).  
`cyclePenalty`: 0 cycles = 1.0 · 1 = 0.8 · 2 = 0.6 · ≥3 = 0.4

## Gate symbols (IEC 60617)

| Gate | Symbol |
|------|--------|
| AND  | `&` in upper-right corner |
| OR   | `≥1` in upper-right corner |
| NAND | `&` + negation bubble on output |
| NOR  | `≥1` + negation bubble on output |

Negated inputs are shown as a small open circle on the input wire.
