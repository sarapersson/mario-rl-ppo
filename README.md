# Mario RL PPO — Local Training Suite

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](
https://colab.research.google.com/github/sarapersson/mario-rl-ppo/blob/main/Mario_RL_PPO_Local_TrainingSuite.ipynb)

Train a PPO agent (Stable-Baselines3 + PyTorch) to play **Super Mario Bros** in a fully local, token-free Colab workflow.  
Observations use **84×84 grayscale** stacked frames; default action set is **SIMPLE_MOVEMENT**; default reward mode matches the project spec (**spec**).

> **Note:** The `/content/mario_rl_local/` folder is generated at runtime and is **not** versioned.
> Large artifacts are excluded by `.gitignore`:
> `models/`, `videos/`, `logs/`, `plots/`, `*.mp4`, `*.zip`, etc.

---

## Table of Contents
- [Overview](#overview)
- [Key Features](#key-features)
- [Repo Layout](#repo-layout)
- [Requirements](#requirements)
- [Quick Start (Colab)](#quick-start-colab)
- [Training Recipes](#training-recipes)
- [Configuration](#configuration)
- [Evaluation & Reporting](#evaluation--reporting)
- [Exporting Artifacts](#exporting-artifacts)
- [Troubleshooting](#troubleshooting)
- [Reproducibility](#reproducibility)
- [Notes on Reward Modes](#notes-on-reward-modes)
- [Roadmap](#roadmap)
- [Acknowledgments](#acknowledgments)

---

## Overview

This project implements a **PPO** reinforcement learning agent that learns to progress through **Super Mario Bros**. The agent receives only pixel observations and learns via reward feedback.

- **Goal:** learn forward progress and avoid dying.
- **Data:** generated on-the-fly from the environment (no static dataset).
- **Observation preprocessing:** grayscale, resize to 84×84, stack frames.
- **Action sets:** `SIMPLE_MOVEMENT` (default, 7 actions) or `RIGHT_ONLY` (5 actions).
- **Default reward mode ("spec"):** +1 when moving forward within a step, −15 on death, 0 otherwise.

---

## Key Features

- **Local-only workflow:** no Drive, no tokens; everything under `/content/mario_rl_local`.
- **Two reward modes:**  
  - `spec` (default) matches assignment spec.  
  - `shaped` is optional for faster early learning.
- **Robustness patches:** JoypadSpace seed/options handling, nes-py overflow hotfix, NumPy pin.
- **Auto-resume training:** warmup + configurable blocks; quick test and full runs.
- **Evaluation:** aggregate reward (J) and reward+distance (J2).
- **Video export:** trained agent (F1) and random baseline (F2).
- **Report generator:** 1–2 pages summarizing data prep, model, metrics, and improvements (G).
- **Token-free GitHub packer:** produce a clean ZIP for manual upload (U-LOCAL-PACK).

---

## Repo Layout

**Root**
- `Mario_RL_PPO_Local_TrainingSuite.ipynb` # main Colab notebook
- `requirements.txt`
- `.gitignore`
- `README.md`

**Folder `reports/`**
- `report_local.md`
- `report_local.html` (optional)

Artifacts created during runs (ignored by git unless you add them):

## Runtime Artifacts (not versioned)

**Folder:** `/content/mario_rl_local/`

- **logs/**
  - `<run-id>/`
    - `*.monitor.csv`
    - `progress.txt` (optional)

- **models/**
  - `<run-id>/`
    - `best_model.zip` (from EvalCallback)
    - `ckpt_*.zip` (checkpoints)
  - **exports/**
    - `<timestamp>_*.zip` (exported models)
    - `<timestamp>_*.meta.json`

- **videos/**
  - `play_*.mp4` (trained agent)
  - `random_*.mp4` (random baseline)

- **plots/**
  - `*.png` (optional)

- **reports/**
  - `report_local.md`
  - `report_local.html`

---

## Requirements

- Google Colab (CPU or GPU). GPU recommended (T4 or better).
- Python libs are installed by the notebook:
  - `stable-baselines3`, `gymnasium==0.29.1`, `gym==0.26.2`, `shimmy`
  - `nes-py==8.2.1`, `gym-super-mario-bros==7.4.0`
  - `numpy==1.26.4` (pinned to avoid ABI issues)
  - `pillow`, `imageio`, `imageio-ffmpeg`, `moviepy`, `markdown`, `pandas`, `matplotlib`

---

## Quick Start (Colab)

1. **Open the notebook** (Colab badge at top).
2. Run **A1** (pin NumPy) → **Runtime → Restart runtime**.
3. Run **A2–A3** to install and verify dependencies.
4. Run **B-LOCAL** to set up local folders under `/content/mario_rl_local/`.
5. Choose:
   - **T** (Quick test): short warmup + short main pass.
   - **D** (Full training with blocks): warmup + multiple blocks.
6. Inspect curves (**E**), evaluate (**J/J2**), and generate the report (**G**).
7. Package for GitHub (**U-LOCAL-PACK**) and upload via GitHub web UI.

---

## Training Recipes

- **Quick Test (T):** a small warmup (e.g., 3k steps) + one short block (e.g., 8k steps).
  - Good for smoke tests and for generating initial artifacts.
- **Full Training (D):** warmup + N blocks with auto-resume and safe restarts.
  - Increase `num_blocks` and `timesteps_per_block` as needed (GPU recommended).

**Tip:** Start with `RIGHT_ONLY` for early forward motion, then switch to `SIMPLE_MOVEMENT` for richer behavior.  
Default here is `SIMPLE_MOVEMENT` to match the final spec.

---

## Configuration

In section **C1** of the notebook:
- `action_set = 'SIMPLE_MOVEMENT'` or `'RIGHT_ONLY'` (default: `SIMPLE_MOVEMENT`)
- `frame_stack = 2` (stacked frames)
- `action_repeat = 2` (repeat actions to reduce jitter)
- `frame_skip = 4` (warmup skip; main pass often uses 2)
- Global `REWARD_MODE` (in section C): `"spec"` (default) or `"shaped"`

Model architecture:
- Compact CNN feature extractor (`SmallCNNFeatures`)
- PPO policy with `net_arch=[128, 128]`
- Default PPO hyperparameters tuned for stability and speed in Colab

---

## Evaluation & Reporting

- **E:** plots reward per episode with EMA(20).  
- **J:** evaluates N episodes and reports mean/std reward.  
- **J2:** evaluates N episodes and reports reward + max `x_pos` (how far Mario progressed).  
- **G:** generates a short report (1–2 pages) covering:
  - Data preparation (84×84 grayscale, stacking, skip/repeat)
  - Model selection (PPO) and motivation
  - Metrics: episode rewards, last-20 mean, best reward, distance
  - Improvement suggestions

Reports are saved to `/content/mario_rl_local/reports/`.

---

## Exporting Artifacts

- **F1:** 60s video of the trained agent (MP4).
- **F2:** 15s random baseline video for comparison.
- **S-LOCAL:** full ZIP of `/content/mario_rl_local` (large).
- **S-LOCAL-LITE:** a small ZIP with latest export, video, plots, and reports.
- **U-LOCAL-PACK:** GitHub-ready ZIP containing:
  - Notebook, report(s), requirements.txt, .gitignore, README.

Upload the contents of the ZIP to GitHub via the web UI (no tokens required).

---

## Troubleshooting

**NumPy ABI or Tensorboard errors**  
- Always run **A1** first and pin `numpy==1.26.4`, then **restart runtime**.

**`JoypadSpace.reset(...) got unexpected keyword argument 'seed'`**  
- The notebook patches JoypadSpace to drop unsupported kwargs and wraps vector envs to strip `seed/options`.

**`OverflowError` in nes-py `_rom.py`**  
- A small overflow hotfix is applied in section C. Make sure A2 is run after restart.

**`AssertionError: The number of environments ... set_env ...`**  
- When loading a model, the notebook uses `PPO.load(path, env=...)` to match `n_envs`. Avoid switching `n_envs` with `set_env` directly.

**`VecVideoRecorder` requires `render_mode='rgb_array'`**  
- The video cells build envs with `render_mode='rgb_array'`. If you change code, keep that setting for video export.

**Worker crashes (EOFError) in SubprocVecEnv**  
- The training loop auto-restarts envs and can fall back to `DummyVecEnv`. If instability persists, reduce `num_envs`.

---

## Reproducibility

- Seeds are set for `numpy`, `random`, and `torch`.
- Model exports include a small `.meta.json` capturing:
  - `action_set`, `frame_stack`, `action_repeat`, `reward_mode`
- Use the same reward mode and action set when resuming from exports.

---

## Notes on Reward Modes

- **spec (default):**  
  +1 when forward progress occurs within a step; −15 on death; 0 otherwise.  
  This aligns with the assignment’s project specification.

- **shaped (optional):**  
  Dense progress shaping (+dx), small per-step penalty, checkpoint bonuses, and death penalty.  
  Useful for faster early learning; consider switching back to `spec` for final evaluation.

---

## Roadmap

- Optional curriculum from `RIGHT_ONLY` to `SIMPLE_MOVEMENT`.
- Entropy scheduling: 0.02 early → 0.005 later.
- Slightly larger CNN (e.g., +64 filters) if learning plateaus.
- Tune `frame_skip` and `action_repeat` for your runtime.
- Longer training runs on GPU for level completion attempts.

---

## Acknowledgments

- [Stable-Baselines3](https://github.com/DLR-RM/stable-baselines3)  
- [gym-super-mario-bros](https://github.com/Kautenja/gym-super-mario-bros)  
- [Gymnasium](https://github.com/Farama-Foundation/Gymnasium) and [Gym](https://github.com/openai/gym)  
- [nes-py](https://github.com/Kautenja/nes-py)
