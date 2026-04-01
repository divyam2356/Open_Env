# 📜 Technical Rules

### ⚙️ Main Tasks
Your absolute goal is to:
* 🏗️ **Create** the environment the AI can play in.
* 📈 **Define** tasks that get harder as they go.
* 📏 **Write** "Graders" (automatic scoring) to check if the AI succeeded.
* 💰 **Reward** logic that helps the AI learn.
* 📦 **Package** everything with the OpenEnv spec.

### 1. OpenEnv Compliance
Your code must pass the `openenv validate` test. This means:
* Use **Pydantic** models to define what the AI can see (Observation) and do (Action).
* Your `step()` function must return: `observation`, `reward`, `done`, and `info`.

### 2. Three Difficulty Levels
You need to create at least **3 tasks** for the AI:
* 🟢 **Easy** (Simple objective)
* 🟡 **Medium** (A bit more complex)
* 🔴 **Hard** (Really tests the AI)
* *Each task needs a "Grader" (a script that gives a score from 0 to 1).*

### 3. Rewards
The AI needs "crumbs" to follow. 
* Don't just give a score at the very end.
* Give **partial rewards** as the AI gets closer to the goal.
* Penalize bad behavior (like getting stuck in a loop).

### 4. Baseline AI
You must include an `inference.py` script in the root directory.
* This script runs a model against your environment to prove it works.
* You **must** use the `OpenAI API client` to run the model.
* It must read API credentials from environment variables (e.g., `OPENAI_API_KEY`).
* It must produce a reproducible baseline score on all 3 of your tasks.
* **Important:** It must output specific strings to stdout: `[START]`, `[STEP]`, and `[END]` lines in an exact format.

---
*Next: See [SCORING.md](SCORING.md) to see how you are judged.*
