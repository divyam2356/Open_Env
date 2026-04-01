# ✅ Final Submission Checklist

Before you hit "Submit", make sure:

- [ ] **Hugging Face Space** is live.
- [ ] Your **URL** responds to a `reset()` command.
- [ ] `openenv validate` passes with no errors.
- [ ] **Dockerfile** works (test it with `docker build`).
- [ ] **inference.py** is in the root folder and runs in under 20 minutes.
- [ ] Your **README.md** explains how to use your environment.

### 🐧 Machine Limits
Your environment must run smoothly on:
* 2 CPUs
* 8 GB of RAM

### 🔑 Keys Needed
Make sure your script can use these environment variables:
* `OPENAI_API_KEY` (or the equivalent API key needed)
* `API_BASE_URL` (The API endpoint for the LLM)
* `MODEL_NAME` (The model identifier)
* `HF_TOKEN` (Your Hugging Face token)
