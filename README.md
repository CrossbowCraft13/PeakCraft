<img width="1379" height="934" alt="image" src="https://github.com/user-attachments/assets/8233861b-8924-4006-9f26-f92746ed466c" />


<div align="center">

# ⛏️ PeakCraft

### A local, self-training AI that speedruns Minecraft — legally, pixel by pixel — until it beats itself.

> *Like the Trackmania champion AIs that grind toward the world record, PeakCraft's agent
> **"Champion"** races the same seed over and over — watching the screen like a human, pressing
> keys like a human, learning from every run until its personal best stops improving.*

**100% local & private · fully automatic · every knob configurable**

</div>

---

## The Pitch

Point PeakCraft at any Minecraft seed. It launches the game, plays a full Any% speedrun — mining,
bartering with piglins, throwing eyes of ender, triangulating the stronghold, bed-bombing the
dragon — records the attempts you choose, and then does it again. And again. Each run is an
episode, each finish time is a reward signal, and a population of self-mutating policies compete
until a **Champion** emerges with the fastest time.

It uses the most advanced techniques the pros use — eye-throw triangulation, stronghold ring math,
bastion bartering, F3 microlensing, portal linking, one-cycle dragon fights — and trains itself on
top of them.

## The Rules (the part that makes this hard — and legal)

Most Minecraft bots cheat by being inside the game: Baritone and Mineflayer inject pathfinding and
movement into the game itself. PeakCraft refuses that.

**It plays the game. It never runs the game.**

- **Human channels only** — screen pixels in, keyboard & mouse out. No memory reading, no mods, no
  game APIs, no touching save files mid-run.
- **Human parity** — inputs are clamped and scheduled to human-feasible rates. No sub-frame
  inhuman aiming, no 30 cps clicking, no mid-run render-distance tricks to despawn a pearl. If a
  human can't theoretically do it, Champion can't.
- **F3 is fair game** — it opens the debug overlay when it needs coordinates (exactly what a human
  does) and reads it with local OCR.
- **Legal runs** — real-time, glitchless, whole-run recording, no save-scumming. External seed
  analysis is allowed *before* a run (as pros do for Set Seed and practice); during a run it only
  knows what it sees and what it computes — and for Random Seed, no seed-derived information may
  be used mid-run, period.

The control layer is the **law-enforcement boundary**: illegal inputs are structurally impossible,
not just discouraged.

## How it works

```
┌──────────────────────────────────────────────────────────────────┐
│  DESKTOP CONTROL PANEL — launch MC · config · live view · PB     │
│  chart · recordings · model manager · logs · "leave it running"  │
└───────────────────────────────┬──────────────────────────────────┘
┌─ CAPTURE ─────────────────────▼──────────────────────────────────┐
│ Windows.Graphics.Capture on the MC window (60fps) · DXGI fallback │
└───────────────┬──────────────────────────────────────────────────┘
┌─ PERCEPTION ─▼───────────────────────────────────────────────────┐
│ YOLO-class detector (items, mobs, blocks) · PaddleOCR on F3      │
│ (coords, facing, entity counts) · optional local VLM · spatial   │
│ memory map built purely from what it sees                        │
└───────────────┬──────────────────────────────────────────────────┘
┌─ POLICY ──────▼──────────────────────────────────────────────────┐
│ Macro (task selection) + micro (movement/aim/combat) policy,     │
│ over deterministic pro-strat priors: eye math, ring math, portal │
│ linking, bed timing, barter management                           │
└───────────────┬──────────────────────────────────────────────────┘
┌─ CONTROL ─────▼──────────────────────────────────────────────────┐
│ SendInput (hardware scan codes) — clamped to human-feasible      │
│ input rates, turn speeds, and key cadence                        │
└───────────────┬──────────────────────────────────────────────────┘
┌─ TRAINING ────▼──────────────────────────────────────────────────┐
│ Episode = attempt · reward = −time to dragon death · offline RL  │
│ + population-based evolution · champion = fastest PB · human     │
│ teleop demos bootstrap imitation learning                        │
└───────────────┬──────────────────────────────────────────────────┘
┌─ RECORDING ───▼──────────────────────────────────────────────────┐
│ Your choice: last N attempts · PBs only · milestones · specific  │
│ runs · lossless → compressed archive with full audit metadata    │
└──────────────────────────────────────────────────────────────────┘
```

## Features

- 🎯 **Any seed, any version** — pin 1.16.1 (the classic RSG category) or modern 1.21.x; every
  game rule and difficulty is a setting.
- 🧠 **Self-improving** — reward = finish time; population-based training promotes the fastest
  policy; PBs and improvement curves tracked per seed.
- 📺 **Selective recording** — record the last run, every personal best, milestone moments, or any
  attempt ID you choose, with lossless intermediate capture and a compressed archive.
- 🏁 **Pro techniques built in** — eye-throw triangulation, stronghold ring math, F3 microlensing,
  bastion bartering, portal linking, bed-bomb dragon kills.
- 🛡️ **Legally constrained by construction** — the input layer physically cannot exceed human
  capabilities.
- 🏠 **Private by design** — all models, training, and recordings local; zero cloud; telemetry off.
- 🖥️ **Leave it running** — watchdog, crash recovery, disk quotas, unattended overnight sessions.
- 🎛️ **Everything is a setting** — seed, difficulty, game rules, input profile, model choices,
  training hyperparameters, recording policy. Nothing hard-coded.

## Tech stack (planned)

| Layer | Choice |
|---|---|
| Language | Python 3.11+ (PyTorch, ONNX Runtime) |
| Capture | Windows.Graphics.Capture (DXGI fallback) |
| Input | `SendInput` with hardware scan codes |
| Perception | YOLO-class detector · PaddleOCR (F3) · optional Qwen2.5-VL / MiniCPM-V |
| Policy | Transformer policy (VPT/STEVE-1 lineage) + DreamerV3-class world model, trained locally |
| UI | PySide6 desktop control panel |
| Models | Local registry, auto-download from Hugging Face with checksums |

## Roadmap

- [ ] **M1 — Skeleton:** desktop window, config + schema, pinned MC launcher, capture→input loop,
      human-teleop "ghost mode" for demo collection
- [ ] **M2 — Perception:** detectors, F3 OCR, spatial memory map
- [ ] **M3 — Scripted pro-strat baseline:** the full legal route, beat easy seeds
- [ ] **M4 — Learning loop:** RL fine-tunes on top of priors + demos; PB curve improves
- [ ] **M5 — Champion:** population training, milestone recording, long unattended runs, PB vs
      human-WR leaderboards

## FAQ

**Is this a TAS?** No. TASes use frame-perfect inputs a human can't perform. Champion's inputs are
clamped to human-feasible rates — any run it produces is human-plausible.

**Does it use mods?** Never. No Baritone, no Mineflayer, no debug mods. It sees pixels and presses
keys, like you do.

**Can it beat the human world record?** That's the entire point. It starts with pro routing
knowledge, then improves with a training budget a human could never match. The current human WRs
(~6:40 in 1.16.1 RSG) are the benchmark.

**Does it need an internet connection?** Only once, optionally, to download local models. After
that it runs fully offline.

**What hardware?** A consumer GPU helps (the policy/world model train locally); the perception
pipeline runs at 60fps+ even on modest hardware thanks to small, fast local models.

## Status

🚧 **Pre-alpha.** This repository currently contains the project's [core memory](CORE_MEMORY.md)
and this page. The roadmap above is being built out in order — M1 first.

## License

TBD. (Nothing here yet to license — the constitution is [CORE_MEMORY.md](CORE_MEMORY.md).)
