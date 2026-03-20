# RHesus LIFE v1.0

Conway's Game of Life for M5Cardputer — with real-time audio synthesis, dynamic rule switching, long-lived cell entities, neon color palettes, zoom camera, and visual effects.

The name references the Rh (Rhesus) blood antigen: something present in life, invisible, that defines behavior and compatibility.

---

## How it differs from other Conway implementations

Most Game of Life implementations faithfully reproduce John Horton Conway's original cellular automaton (1970): a grid, two states (alive/dead), B3/S23 rules, and basic visual output. RHesus LIFE adds several layers that set it apart in both gameplay and technical design.

**Dynamic rules: PANIC mode.** The classic Conway always runs on B3/S23. RHesus LIFE lets you switch rules in real time. Activating PANIC applies an alternative bitmask ruleset (`BIRTH_PANIC = 0b011001100`, `SURV_PANIC = 0b100101110`), producing explosive growth patterns impossible under standard Conway — without resetting the simulation.

**Cells with memory: GOD and DEMON modes.** Standard implementations treat all cells as equivalent. RHesus LIFE introduces two types of marked cells, each with their own life counter. GOD cells are born with 12 extra cycles and survive under extended conditions (S2345). DEMON cells carry 24 cycles and survive under even denser conditions (S23456). Both types coexist on the same grid, rendered in distinct color variants derived from the active palette, and compete with normal cells under their own survival rules. No standard Conway implementation contemplates qualitatively different cell types sharing the same grid.

**Generative audio synchronized to the simulation.** Almost all Conway implementations are silent or play background music. RHesus LIFE generates sound procedurally from simulation events. Births and deaths produce tones whose frequency and duration depend on activity volume. Eight distinct synthesis engines are available, each with a different mapping from simulation data to sound parameters: Bioacoustic, Electronic, Chaotic, Melodic, Pulses, Spatial, Resonant, and Glitchcore. Zone Audio mode restricts audio analysis to the current viewport instead of the full grid.

**Camera with zoom and panning.** The original Conway works at 1:1 scale. RHesus LIFE implements a virtual camera with zoom from 1x to 8x (pixel scaling) and free panning across the 240x135 cell grid. The differential rendering system (`drawGridDiff`) only repaints cells that changed state, optimizing performance on constrained hardware.

**Written natively for embedded hardware.** Many microcontroller ports are direct translations of desktop code. RHesus LIFE uses the M5Cardputer native display API with direct `fillRect` calls, implements audio throttling to avoid interference with the simulation loop (`SOUND_THROTTLE_MS = 55ms`), uses separate `uint8_t` arrays for state, GOD counters, and DEMON counters to minimize RAM, and draws text with a custom 5x7 pixel bitmap font without external typography libraries.

---

## Technical Specification

| Parameter | Value |
|---|---|
| Platform | M5Cardputer (ESP32-S3) |
| Framework | Arduino / M5Stack |
| Main library | M5Cardputer.h |
| Display resolution | 240 x 135 px (landscape) |
| Grid size | 240 x 135 cells |
| Grid topology | Toroidal (edges wrap around) |
| Estimated RAM (grids) | 4 x 32,400 bytes, approx. 130 KB |
| Default rules | B3/S23 (classic Conway) |
| PANIC rules | B2/B3/B6/B7 / S2/S3/S5/S7/S8 |
| Zoom range | 1x to 8x |
| Color palettes | 8 neon palettes (green, cyan, lime, teal, magenta, yellow, orange, gray) |
| Audio engines | 8, synthesized in real time |
| Serial baud rate | 9600 |
| Version | 2.7 |

---
HAVE FUN
---

## Controls

| Key | Action |
|---|---|
| BtnA / G0 | Nuke — reset world, advance color palette |
| = | Zoom in (up to 8x) |
| - | Zoom out (down to 1x) |
| , | Move camera left |
| / | Move camera right |
| ; | Move camera up |
| . | Move camera down |
| DEL | Next color palette (8 neon options) |
| 1-9 | Volume (1 = min, 9 = max) |
| 0 | Cycle sound engine (8 types) |
| S | Mute / unmute |
| P | Toggle PANIC mode |
| G | Invoke GOD mode |
| D | Invoke DEMON mode |
| I | Hold to show live info overlay |
| H | Hold to show help overlay |
| Z | Zone audio on / off |
| W | Screen brightness (cycles 25 to 255) |
| Enter | Close open overlays |

---

## Special Modes

**NUKE.** Fully resets the world with a new random population. Deactivates PANIC, GOD, and DEMON. Advances the color palette. Plays a visual scan-line effect and a descending tone sequence.

**PANIC mode.** Replaces the automaton rules with a chaotic bitmask that produces rapid growth and irregular patterns. Periodic glitch effects appear on screen while active. Simulation speed increases (8ms delay instead of 16ms). Activating shows an animated PANIC sequence with a descending jingle; deactivating shows a glitch effect and the text NORMAL.

**GOD mode.** Injects up to 80 divine cells into low-density positions in the current viewport. GOD cells render in a brighter variant of the active color, survive under extended conditions (S2345), and carry an internal 12-cycle counter that decrements on each death event.

**DEMON mode.** Injects up to 60 demonic cells into medium-density positions (2 to 4 neighbors). DEMON cells render in a darker variant of the active color, survive under wide conditions (S23456), and carry a 24-cycle counter. GOD and DEMON cells can coexist in the same simulation, producing unique color interactions and behavioral interference.

---

## Info Overlay (hold I)

A panel in the upper-left corner shows:

| Field | Description |
|---|---|
| GEN | Current generation number |
| POP | Live cells in viewport |
| ZOOM | Active zoom level |
| CAM | Camera X/Y position |
| BORN | Cells born in last cycle |
| DEAD | Cells died in last cycle |
| SND | Active audio engine, or --- if muted |
| MODE | Current state: normal / !PANIC! / ~GOD~ / [DEMON] |

Two activity bars show birth rate (active color) and death rate (gray).

---

## Audio Engines

| No. | Name | Behavior |
|---|---|---|
| 0 | Bioacoustic | Organic mid-range tones proportional to activity |
| 1 | Electronic | Short high-frequency bursts, digital synthesizer style |
| 2 | Chaotic | Pure random noise, no scale or structure |
| 3 | Melodic | Pentatonic scale mapped to population density |
| 4 | Pulses | Short rhythmic pulses, high response speed |
| 5 | Spatial | Long sustained tones, ambient feel |
| 6 | Resonant | Minor scale, long notes with resonance |
| 7 | Glitchcore | Modular arithmetic frequencies, unpredictable output |

Zone Audio (`Z`) limits the birth/death counters fed to the audio engine to the visible viewport only, so sound reflects what you see rather than the entire grid.

---

## Code Architecture

```
RHesusLife.ino
├── Color palette (neonPalette[8])
├── State grids
│   ├── grid[240][135]        current state
│   ├── newgrid[240][135]     next state (double buffer)
│   ├── godgrid[240][135]     GOD life counter per cell
│   └── demongrid[240][135]   DEMON life counter per cell
├── Audio engine (8 types, throttled at 55ms)
├── 5x7 pixel bitmap font (no external library)
├── Visual effects (glitch, nuke, panic, god, demon, splash screen)
├── Camera system (camX, camY, res, viewW, viewH)
├── Differential renderer (drawGridDiff)
└── Main loop with keyboard handling
```

The grid topology is toroidal: cells at the right edge have the left-edge cells as neighbors, and the same applies vertically. This eliminates border artifacts and allows patterns to wrap across the display.


---

## License

MIT License — free to use, modify, and distribute with attribution.
