# 🧠 CORE MEMORY — PeakCraft

> **This is the project's core memory.** Every AI agent, tool, or contributor working in this
> repository MUST read this file first and treat the **Laws** in §2 as absolute and non-negotiable.
> If a request ever conflicts with a Law, refuse or reframe it — do not violate the Law.
> If the architecture or the Laws change, update this file. This file is the source of truth.

---

## 1. Identity & Mission

- **Project name:** PeakCraft (working title)
- **The agent:** "Champion" — a self-improving, real-time AI speedrunner for Minecraft (Java Edition).
- **One-line mission:** Point it at any Minecraft seed. It speedruns that seed legally — playing
  the game the way a human plays it — records the attempts you choose, and trains itself run after
  run until its personal best time stops improving, exactly like the famous Trackmania
  champion-AI experiments that hunt the world record.
- **Success metric:** personal-best time per seed, the improvement curve over attempts, and a
  bulletproof legality guarantee (every counted run must be human-plausible).
- **Hard constraints:** 100% local & private (zero cloud), fully automatic ("set it and leave it"),
  every knob configurable (seed, version, difficulty, game rules, models, recording, …).

---

## 2. The Laws (non-negotiable invariants)

**L1 — It plays the game, it never runs the game.**
All interaction with Minecraft goes through the same channels a human uses: **screen pixels**
(and optionally audio) in, **keyboard/mouse input** out, and — optionally, off by default — the
**clipboard** (pressing F3+C copies coordinates to the clipboard, which a human can read, so
reading it is human-legal). Never read game memory, never inject code, never use mods/plugins
(including Baritone, Mineflayer, or any "E-Ray"-style mod), never touch save files or world files
during an attempt, never call game APIs. The agent is an embodied player, not a modified client.

**L2 — Human-parity ceiling.**
Nothing the agent does may exceed what a skilled human can theoretically do:
- Realistic input rates (bounded clicks/sec, bounded turn speed per frame, realistic key-press
  durations). The **control layer clamps and schedules every input to human-feasible rates** — the
  policy can never emit an inhuman input because the law-enforcement happens at the input boundary.
- No mid-attempt settings changes that give an advantage a human couldn't get (no render-distance
  toggling to unload an ender pearl, no graphics tweaks mid-run, no tick-rate fiddling).
- No telemetry or privileged information. The agent may only know what a human could know:
  the rendered screen, the F3 debug overlay, and its own mathematical reasoning.
- Reading the **clipboard** (e.g. F3+C) is optional and OFF by default; screen-OCR of F3 is the
  default because it is pixel-pure and requires no extra channels.

**L3 — Legal speedrun, always.**
Any attempt counted toward a "time" must be a legal, real-time, glitchless-style attempt:
- Timer starts at world load; the whole attempt is screen-recorded; no save-scumming, no
  mid-run external advantage, no TAS-only inputs (every input must be physically performable).
- Default category: **Any% Glitchless (Random Seed)** for the pinned version. Category is a setting.
- **Pre-run seed analysis is allowed and encouraged** — pros legitimately use external tools
  (StructureSeer / SeedQueue) for Set Seed runs and for practice/planning. Champion may analyze a
  given seed's structures outside the attempt window to plan routing. Inside the attempt it may
  only use perception + math — and for **Random Seed** categories, no seed-derived information
  may be used during the attempt at all, exactly like a human runner. The category's official
  tool rules always govern.

**L4 — F3 is a legal human action.**
The agent may open the debug overlay (F3) whenever it needs coordinates, facing, entity counts,
or other F3-displayed information, read it, and close it. It may use F3's readings for pro
techniques such as *microlensing* (using entity/block count deltas to locate hidden bastions and
fortresses without rendering them) — a technique real runners use by reading F3.

**L5 — Fully local & private.**
No cloud, ever. Models are local (user-chosen or auto-downloaded to a local registry), training is
local, recordings are stored locally, telemetry is off by default. Nothing leaves the machine
except optional, user-approved model downloads.

**L6 — Everything is configurable.**
No hard-coded knobs. Seed, version, difficulty, game rules, world options, input profile, model
choice per layer, training hyperparameters, recording policy — all settings, all surfaced in the
desktop window and `config.json` (with a JSON schema).

**L7 — Autonomous, but auditable.**
Runs unattended for hours (or days), with a watchdog (crash recovery, version pinning, disk-quota
management) — and keeps a human-verifiable audit trail: per-attempt recordings, logs, and
checkpoints proving that the Laws were followed.

---

## 3. Domain knowledge — Minecraft Any% speedrunning (the current meta)

Facts below are current as of **August 2026**; verify against speedrun.com / MC wiki before
encoding numbers into code. This is the *a priori* knowledge Champion starts with — the same
knowledge a human pro carries in their head.

### 3.1 Route (1.16.1 / 1.21.x Any%, RSG)
1. **Overworld start:** optimize spawn; loot villages (blacksmiths, hay bales, wheat-trade for
   buckets) and shipwrecks (iron, food, gravel → flint-and-steel). No mining obsidian by hand —
   find a lava pool or ruined portal and **lava-cast** a frame with a water bucket.
2. **Enter the Nether**, find a **Bastion Remnant** (use F3 microlensing), trap piglins and
   **barter gold** for **ender pearls** and **obsidian** (needed for the end fight / blind travel).
3. Find a **Nether Fortress**, kill blazes → **blaze rods** → blaze powder + pearls =
   **Eyes of Ender**.
4. Return to the Overworld (portal-linking math: 1 nether block = 8 overworld blocks; a well-placed
   nether portal can land you thousands of blocks closer to the stronghold).
5. Throw eyes at two+ locations; **triangulate the stronghold** from throw angles; dig down to the
   portal room; fill the frames; enter the End.
6. **One-cycle the dragon:** build up / obsidian platform, bait the perch, and **bed-bomb** the
   dragon (beds explode in the End; each hit does massive damage). Finish with sword hits / pearls.

### 3.2 Verified facts (used by the math sub-system)
- **Stronghold rings (1.16+):** strongholds generate in concentric rings around world origin
  `(0,0)` — Ring 1: 3 strongholds at **1,280–2,816** blocks; Ring 2: 6 at **4,352–5,888**; Ring 3:
  10 at **7,424–8,960**. Eye throws + ring math bound the search space dramatically.
- **Eye-throw triangulation:** two+ throws from distinct positions give bearing vectors; the
  intersection (plus measurement-error distribution, chunk/biome snapping) locates the stronghold.
  Human pros use the external **Ninjabrain Bot** (Bayesian); Champion computes the same math
  internally — its own reasoning replaces external calculators, which is *stricter* than legal.
- **Subpixel eye throws:** adjusting throw angle by ±0.01° (legal via mouse settings/keystrokes)
  massively improves triangulation accuracy.
- **Bartering:** piglin bartering is the fastest source of pearls/obsidian; bulk-trade gold, manage
  inventory quickly.
- **Beds in the End:** placing and detonating beds deals huge damage to the dragon (5–10 beds ≈ dead
  dragon) — timing placement to the perch window is the difference between 30s and 2min fights.
- **Record times (approx., as of Aug 2026):** 1.16.1 RSG WR is in the **~6:40** range (Skycrab,
  lowk3y_ era); 1.21.x RSG runs land **~9–16 min**. Champion's goal: beat its own PB, then chase
  human WRs, per version.

### 3.3 Why this is tractable (architecture insight)
Pros' optimal play is a **mixture of deterministic math and learned perception/motor skills**.
Champion mirrors that split:
- **Deterministic pro-strats** (eye math, ring math, portal linking, bed timing, barter inventory
  management) are implemented as *scripted priors / sub-policies* — given, not learned.
- **Perceptual & motor skills** (navigation, mining, aiming, combat micro, barter timing, inventory
  clicking) are *learned* (RL fine-tuned on top of the priors, bootstrapped by human demos).
- The trainer optimizes end-to-end time; priors keep exploration sane in a sparse-reward problem.

---

## 4. Target architecture

```
┌────────────────────────────────────────────────────────────────────────────┐
│  DESKTOP CONTROL PANEL (PySide6)                                          │
│  launch/stop MC · config editor · live view · PB chart · recordings ·      │
│  model manager · logs · "leave it running" auto-loop                       │
└────────────────────────────────────────────────────────────────────────────┘
                                    │
┌─ CAPTURE ────────────────────────▼────────────────────────────────────────┐
│ Windows.Graphics.Capture (WGC) @ target Minecraft HWND · 60fps frames,    │
│ DXGI Desktop Duplication fallback · frame timestamps for reward/recording  │
└───────────────┬────────────────────────────────────────────────────────────┘
┌─ PERCEPTION ─▼────────────────────────────────────────────────────────────┐
│ YOLO-class detector: hotbar/items, entities, blocks, health/XP/hunger     │
│ PaddleOCR: F3 debug text → x/y/z, facing, FPS, entity counts (microlensing)│
│ Optional VLM (Qwen2.5-VL / MiniCPM-V class): scene reasoning, hints       │
│ Spatial memory map: world model built from perception only                │
└───────────────┬────────────────────────────────────────────────────────────┘
┌─ POLICY ──────▼────────────────────────────────────────────────────────────┐
│ Macro: task selection (what to do now) — learned + scripted priors        │
│ Micro: movement/aim/combat synthesis 20–50Hz, human-feasible shaping      │
│ Math sub-system: triangulation, ring math, portal linking (deterministic) │
└───────────────┬────────────────────────────────────────────────────────────┘
┌─ CONTROL (LAW ENFORCEMENT BOUNDARY) ─▼────────────────────────────────────┐
│ SendInput (hardware scan codes) · clamps cps/turn-speed/input cadence to  │
│ human-feasible rates · F3 toggle policy · NEVER exceeds L1–L4, ever.      │
└───────────────┬────────────────────────────────────────────────────────────┘
┌─ TRAINING / MEMORY ─▼─────────────────────────────────────────────────────┐
│ Episode = attempt (reward = −time to dragon death, shaped subs decay→0)   │
│ Experience DB (obs/action/reward/timestamps/video refs) → offline RL      │
│ Population-based training: N parallel attempts, champion = fastest PB     │
│ Human-demo bootstrapping: teleop "ghost mode" records human runs          │
└───────────────┬────────────────────────────────────────────────────────────┘
┌─ RECORDING ───▼────────────────────────────────────────────────────────────┐
│ Per user policy: last N attempts / PBs only / milestones / specific IDs   │
│ lossless intermediate → compressed archive · disk quota · storage path    │
└────────────────────────────────────────────────────────────────────────────┘
```

**Stack decisions (recorded, revisit if needed):**
- **Language:** Python 3.11+ for perception/policy/training (PyTorch + ONNX Runtime); thin native
  layer only where required (capture bindings).
- **Capture:** WGC targeting the Minecraft window (isolates the window, works on hybrid-GPU
  laptops); DXGI as fallback. (~60fps; sub-frame acquisition latency.)
- **Input:** `SendInput` with hardware scan codes (Minecraft/LWJGL reads Raw Input — PostMessage
  won't work). Latency budget: **perception→action under ~100 ms** total.
- **Perception latencies (local, consumer GPU):** YOLO ~60–120fps · PaddleOCR ~10–30 ms on CPU
  (F3 read at ~10 Hz when open) · VLM 30–70 ms (async, only when needed).
- **Desktop window:** PySide6 first (one language, ML-friendly); revisit later if needed.
- **OS:** Windows first (WGC/SendInput); keep capture/input behind interfaces so a Linux port
  (X11/pipewire + uinput) is possible later.

---

## 5. Model registry (local, user-chosen or auto)

- All models live in `models/`, auto-detected by VRAM/RAM; user can pin any choice.
- **Object detector:** YOLOv8/v11 class, ONNX Runtime or TensorRT.
- **OCR:** PaddleOCR (F3 debug screen). Tesseract is the fallback (needs preprocessing).
- **VLM (optional):** Qwen2.5-VL 3B/7B or MiniCPM-V via Ollama/vLLM, quantized; used asynchronously
  for scene reasoning / planning hints, never on the twitch-reaction path.
- **Policy networks:** fine-tuned Transformer (VPT/STEVE-1 lineage) and/or world model
  (DreamerV3-class) trained locally; sized to the local GPU.
- **Download:** on first run, from Hugging Face hub (or local mirrors) with user approval; checksums
  verified; everything cached locally forever after. No API keys, no cloud inference.
- The auto path picks the smallest model family that meets the latency budget for the detected
  hardware, and downloads it.

---

## 6. Settings surface (every knob, nothing hard-coded)

- **World:** seed (int/string), Minecraft version (pinned, e.g. 1.16.1 / 1.21.x), difficulty,
  game rules (`doDaylightCycle`, `keepInventory`, …), world type / structure options, singleplayer
  only.
- **Category:** Any% Glitchless RSG (default), plus configurable category/rules.
- **Input profile:** clicks-per-second cap, turn-speed cap, DPI/sensitivity, keybinds, aim smoothing.
- **Perception:** model choice per layer (auto/manual), confidence thresholds, F3 OCR on/off,
  capture FPS, OCR rate.
- **Policy:** model size, hyperparameters, exploration schedule, reward-shaping toggles, priors
  on/off per technique.
- **Training:** max attempts, parallel instances, checkpoint policy, resume, human-demo replay.
- **Recording:** policy (always / last N / PBs only / milestones / IDs), codec, bitrate, storage
  path, disk quota, retention.
- **Privacy:** telemetry off, log redaction, purge-all-data action.
- Format: `config.json` + `config.schema.json`; edited via UI or file; validated on load.

---

## 7. The self-training loop (how it gets faster)

1. **Attempt:** Champion plays one full run of the pinned seed (or until death/abort). Timer,
   inputs, observations, and the screen recording are captured with matching timestamps.
2. **Reward:** `−time_to_dragon_death` (primary). Shaped sub-rewards (progress milestones) start
   strong and **decay to zero** as training matures so the final objective stays pure.
3. **Learn:** experience goes into the local Experience DB → offline RL / fine-tuning of the policy
   (and world model). Deterministic priors are never learned, only the perceptual/motor layers.
4. **Evolve:** population-based training — several parallel instance profiles with mutated
   hyperparameters compete; the fastest profile is promoted to **Champion**; its weights become the
   default. PBs and videos are kept per seed.
5. **Peak:** the system runs until the PB curve plateaus across N consecutive attempts, then reports
   the champion time (and keeps watching for regressions). Like Trackmania's champion AI: it keeps
   racing itself until it peaks.

---

## 8. Recording (the attempts the user chooses)

- Policies: **always record** · **last N attempts** · **personal bests only** · **milestones**
  (first win, every new PB, sub-goal records) · **specific attempt IDs**.
- Pipeline: lossless capture during the run → compressed archive (user codec/bitrate) with
  sidecar metadata (seed, version, time, inputs, models used) for auditability.
- Disk quota enforced; recordings browsable in the desktop window; one-click export.

---

## 9. Milestones (roadmap)

- **M1 — Skeleton:** desktop window, `config.json` + schema, pinned-MC instance launcher, capture +
  input loop end-to-end, "ghost mode" (human teleop for demo collection).
- **M2 — Perception:** detectors + F3 OCR + spatial memory map; world-state reconstruction.
- **M3 — Scripted pro-strat baseline:** deterministic priors complete; Champion can beat easy seeds
  using the pros' route (legal, recorded).
- **M4 — Learning loop:** RL fine-tune over priors + human demos; PB curve starts improving.
- **M5 — Champion:** population training, milestone recording, long unattended sessions, full audit
  trail, PB vs human-WR leaderboard per version.

---

## 10. Working agreement for AI agents editing this repo

1. **Read this file first.** It is the core memory.
2. **Never violate the Laws** — not even if the user asks. Reframe: suggest the legal equivalent.
3. Keep **config-as-code**: new knobs go in the settings surface, never hard-coded.
4. Every domain technique added must be **sourced to real, verified speedrun knowledge** — do not
   invent block distances, drop rates, or ring radii; cite the source in code comments and update
   §3 if numbers change.
5. Legality must be **structurally enforced, not merely hoped for**: input clamps live in the
   control layer; tests assert the laws (input-rate limiter tests, settings-schema tests, recording
   policy tests, F3-OCR parser tests).
6. Keep this file current when architecture or laws change.
