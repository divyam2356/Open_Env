# 🔍 Analysis: `nvidia.ipynb` vs Hackathon Requirements

## What the Notebook Does

Your notebook implements **"Thermal-Ops"** — a Data Center Cooling Management simulation where an AI agent controls fan speeds, chiller temperatures, and workload migration across 3 server racks to keep temperatures in the safe zone (20–25°C) while minimising energy costs.

It uses **GRPO (Group Relative Policy Optimization)** via TRL to fine-tune `Qwen/Qwen2.5-0.5B-Instruct` on this environment using tool-calling format.

### Components in the Notebook

| Component | What it does |
|-----------|-------------|
| `ThermalOpsEnv` class | Simulates thermodynamics: heat generation, fan cooling, chiller effects, ambient temp, broken fans |
| System Prompt | Instructs the model to use `set_fan_speed`, `adjust_chiller`, `migrate_workload`, `wait` tools |
| `reward_func` | GRPO reward: env reward + tool-name bonuses + `<tool_call>` tag bonuses + JSON validity bonuses |
| Dataset | 100 scenario prompts (nominal, fan failure, heatwave, budget alert, etc.) |
| GRPO Training | Fine-tunes Qwen2.5-0.5B with `GRPOTrainer` + `environment_factory` |
| Evaluation | Runs the fine-tuned model through 10 steps of the env |

---

## ✅ What's Good (Aligned with Hackathon)

| Requirement | Status | Notes |
|---|---|---|
| **Real-world task** (30% weight) | ✅ Excellent | Data center thermal management is a genuine, high-value real-world problem. This is a strong domain choice. |
| **Meaningful reward function** | ✅ Good foundation | Energy cost penalty + overheat penalty + baseline drift penalty. Provides continuous signal, not binary. |
| **Randomised episodes** | ✅ Good | Random ambient temps, rack temps, loads, fan RPMs, chiller setpoints, broken fans (30% chance). |
| **Multiple tools/actions** | ✅ Good | 4 actions: `set_fan_speed`, `adjust_chiller`, `migrate_workload`, `wait` |
| **Thermodynamics simulation** | ✅ Good | Heat generation, fan cooling curves, chiller effects, ambient bleed-in are modeled. |

---

## ❌ Critical Gaps — What's Missing for the Hackathon

### 1. 🚫 No OpenEnv Spec Compliance (Required)

Your `ThermalOpsEnv` is a **standalone Python class**, NOT an OpenEnv environment. The hackathon requires:

```
❌ No Pydantic models (Action, Observation, Reward)
❌ No openenv.yaml
❌ No HTTP API (FastAPI server with /reset, /step, /state endpoints)
❌ No client.py (HTTPEnvClient subclass)
❌ No typed models.py
❌ openenv validate will FAIL — instant disqualification
```

**What's needed:**
```
src/envs/thermal_ops/
├── models.py           ← Pydantic: ThermalAction, ThermalObservation, ThermalState
├── client.py           ← HTTPEnvClient subclass
├── openenv.yaml        ← Metadata + task definitions
└── server/
    ├── environment.py  ← Your ThermalOpsEnv adapted to Environment base class
    ├── app.py          ← FastAPI server
    └── Dockerfile      ← Container definition
```

### 2. 🚫 No Defined Tasks with Graders (Required — 25% weight)

The hackathon requires **minimum 3 tasks** with programmatic graders that score 0.0–1.0. Your notebook has no task definitions or graders at all.

**Needed:**
- **Easy task**: "Keep all racks below 25°C for 10 steps" (no broken fans, mild ambient)
- **Medium task**: "Manage cooling with 1 broken fan under moderate load"
- **Hard task**: "Handle heatwave (30°C ambient) + 2 broken fans + power spike, minimize energy"

Each needs a **grader function** returning a float score 0.0–1.0.

### 3. 🚫 No Dockerfile (Required — Instant DQ)

The pre-submission checklist requires `docker build && docker run` to work. No Dockerfile exists.

### 4. 🚫 No HF Space Deployment (Required — Instant DQ)

Must deploy as a containerized HF Space tagged with `openenv`. The automated validation pings your `/reset` endpoint.

### 5. 🚫 No `inference.py` in Required Format (Required — Instant DQ)

The hackathon mandates a specific `inference.py` format with:
- `[START]`, `[STEP]`, `[END]` stdout format
- OpenAI client usage (not local model inference)
- Reading `API_BASE_URL`, `MODEL_NAME`, `HF_TOKEN` from env vars
- Must complete in < 20 minutes on 2 vCPU / 8GB RAM

Your notebook runs the model locally with `transformers` — this won't work for submission.

### 6. 🚫 No `state()` Endpoint

The OpenEnv spec requires `state()` → returns current environment state. Your env doesn't implement this.

### 7. 🚫 No README (Required — 15% weight)

Must include: environment description, action/observation spaces, task descriptions, setup instructions, baseline scores.

---

## ⚠️ Training Issues Observed in the Notebook

Looking at the training logs, GRPO training **collapsed** around step 8:

| Steps | Behaviour |
|-------|-----------|
| 1–4 | Model produces valid `<tool_call>` JSON — rewards ~5–35 |
| 5–7 | Model starts producing gibberish, mixed languages, nonsense JSON |
| 8+ | **Model degenerates to outputting only backticks** (`` ` ` ` ` ` ``) |
| 8–78 | Reward variance → ~0.002, `[WARNING] Very low reward variance!` on every step |

The model effectively **died during training** — GRPO collapsed due to:
- The 0.5B model is too small for this task complexity
- Reward variance was too low for GRPO to compute meaningful advantages
- The random noise fix (±0.1) was insufficient

The evaluation output confirms this — the model just **hallucinated long paragraphs about RPMs** rather than producing tool calls.

> [!CAUTION]
> The GRPO training is separate from the hackathon submission. The hackathon wants you to build the **environment** (the gym), not train a model. The inference script uses an external LLM via API, not your fine-tuned model.

---

## 🎯 What Needs to Happen — Concrete Plan

### Priority 1: Convert to OpenEnv Spec (MUST DO)

1. Create Pydantic models for `ThermalAction`, `ThermalObservation`, `ThermalState`
2. Adapt `ThermalOpsEnv` to inherit from OpenEnv's `Environment` base class
3. Implement `reset()`, `step(action)`, `state()` with proper return types
4. Create `openenv.yaml` with metadata
5. Build FastAPI server (`app.py`) exposing `/reset`, `/step`, `/state`

### Priority 2: Define 3 Tasks + Graders (MUST DO)

| Task | Difficulty | Scenario | Grading |
|------|-----------|----------|---------|
| `stable_cooling` | Easy | Normal conditions, no failures | Score based on % of steps all racks stayed in [20, 25°C] |
| `fan_failure_recovery` | Medium | 1 fan broken, elevated ambient | Score based on temp control + energy efficiency |
| `crisis_management` | Hard | 2 broken fans, 30°C ambient, power spikes | Score based on avoiding critical temps + minimizing energy |

### Priority 3: Containerize + Deploy (MUST DO)

1. Create `Dockerfile` that builds and starts the FastAPI server
2. Deploy to HF Spaces
3. Verify `/reset` returns 200

### Priority 4: Write `inference.py` (MUST DO)

Adapt the sample inference script to work with your ThermalOps environment using the OpenAI client.

### Priority 5: Documentation (MUST DO)

Write a comprehensive `README.md`.

---

## Summary

| Category | Current Status | Hackathon Ready? |
|----------|---------------|-----------------|
| Domain/Concept | ✅ Excellent (Data Center Cooling) | ✅ |
| OpenEnv Spec | ❌ Not implemented | ❌ **DQ risk** |
| Tasks + Graders | ❌ Not implemented | ❌ **DQ risk** |
| Dockerfile | ❌ Missing | ❌ **DQ risk** |
| HF Space | ❌ Not deployed | ❌ **DQ risk** |
| inference.py | ❌ Wrong format | ❌ **DQ risk** |
| README | ❌ Missing | ❌ |
| Reward Design | ✅ Good foundation | ✅ |
| Training (GRPO) | ⚠️ Collapsed | N/A (not required) |

**Bottom line:** Your **domain choice is excellent** and the thermodynamics simulation is well-designed. But the notebook is a GRPO training experiment, not an OpenEnv submission. We need to restructure everything into the OpenEnv spec format, add tasks/graders, containerize, and deploy. The core physics logic is reusable — the wrapping is what's missing.

Shall I start building the full OpenEnv-compliant project?
