# Short Report — Mario PPO (Local Training Suite)

**Author:** Sara Persson  
**Date:** 2025-11-16 21:39

## Data preparation
Observations are derived directly from the environment (no static dataset). Each frame is converted to **84x84 grayscale** and we apply **frame stacking** (stack=2) to expose short-term temporal context.  
We also use **frame skip** and **action repeat** (repeat=2) to reduce computational load and improve sample efficiency.  
Reward mode is **spec**:
- **spec (default):** +1 only when forward progress occurs within a step, and -15 on death; 0 otherwise.
- **shaped (optional):** adds dense progress shaping and small step penalties.

## Model choice and motivation
We use **PPO (Proximal Policy Optimization)** from Stable-Baselines3 with a compact CNN feature extractor.
PPO is chosen because it is:
- **Stable and robust** for on-policy training in discrete-control environments like NES Mario.
- **Well-supported** with reliable implementations, monitoring, and callbacks.
- **Sample-efficient enough** for short Colab runs while still converging with modest tuning.

## Performance and evaluation
- **Episodes:** 3  
- **Total timesteps (from logs):** 350  
- **Mean reward (last 20 episodes):** 46.33  
- **Best single-episode reward:** 120.00

We additionally provide two evaluation utilities:
- **J:** aggregate rewards over N episodes.
- **J2:** reports both reward and maximum `x_pos` (how far Mario progressed).

## Improvement suggestions
If performance is below target, we suggest:
1. **Longer training** (increase warmup and block timesteps) and ensure GPU runtime.
2. **Curriculum**: start with `RIGHT_ONLY`, then switch to `SIMPLE_MOVEMENT` once forward progress is consistent.
3. **Entropy schedule**: higher entropy early, then lower (0.02 -> 0.005) to solidify behaviors.
4. **Reward mode swap**: try `shaped` to speed early learning, then revert to `spec` for alignment with project spec.
5. **Model capacity**: modestly larger CNN (e.g., +64 filters) if learning plateaus.
6. **Frame skip/repeat**: tune (e.g., skip 2–4, repeat 2) to balance fidelity and speed.

## Reproducibility and artifacts
- Notebook: **Mario_RL_PPO_Local_TrainingSuite.ipynb**
- Logs and monitor CSVs under `/content/mario_rl_local/logs`
- Exported models under `/content/mario_rl_local/models/exports`
- Videos under `/content/mario_rl_local/videos`
