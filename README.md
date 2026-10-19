# soma-onboard — Build Your Own AI Companion

> A single script that walks you through creating a personalized AI companion with persistent identity, long-term memory, and optional local image/music generation — all running on your hardware.

---

## What This Does

Run one script. Answer a few questions. Get a complete, ready-to-use AI companion profile that works with **Hermes Agent** (Nous Research).

**You get:**

| Component | What It Is |
|-----------|------------|
| **SOUL.md** | Your companion's global identity — personality, voice, values, boundaries, praise/reinforcement system. Loaded automatically by Hermes as **System Prompt Slot #1**. |
| **AGENTS.md** | Project-specific rules — paths, conventions, tool usage. Auto-loaded when you `cd` into the project folder. |
| **RAG/** | 8 curated knowledge folders (Music, TV, Movies, Games, AI, User Profile, Creative Writing, Technical Docs) to combat hallucination. You populate these over time. |
| **TOOLS/** | Optional local generation binaries: `sd-cli` (Stable Diffusion) and/or `acestep.cpp` (ACE-Step music). |
| **generated_images/** | Default output folder for images. |
| **generated_music/** | Default output folder for music (created when you opt in to ACE-Step). |
| **scripts/** | `backup.sh`, `sync_to_remote.sh`, `test_identity.py` — maintenance helpers. |

---

## Quick Start

```bash
# 1. Get the script
curl -O https://raw.githubusercontent.com/your-repo/soma-onboard/main/soma_onboard.py
# or copy soma_onboard.py from wherever you found it

# 2. Run it (interactive — walks you through everything)
python3 soma_onboard.py --install-all --profile-dir ~/my-companion

# 3. Deploy to Hermes
cp ~/my-companion/SOUL.md ~/.hermes/SOUL.md
cp ~/my-companion/AGENTS.md /path/to/your/project/AGENTS.md

# 4. Restart Hermes (/reset in chat or restart the process)
# 5. Verify
python3 ~/my-companion/scripts/test_identity.py
```

**Non-interactive (all defaults, Soma's production config):**
```bash
python3 soma_onboard.py --non-interactive --install-all --profile-dir ~/my-companion
```

**Just the guides (no profile creation):**
```bash
python3 soma_onboard.py --show-guides
```

---

## What the Interactive Questionnaire Covers

11 questions — press **Enter** to accept sensible defaults, or customize:

| # | Question | Example Default |
|---|----------|-----------------|
| 1 | Agent name | `Aria, Nexus, Companion` |
| 2 | Your name | `Alex, Jordan, Captain` |
| 3 | How they address you | `Captain, Boss, Alex` |
| 4 | What they must NEVER call you | `Good Boy, Slave, Pet` |
| 5 | Forbidden praise variants | `Good Girl yourself...` |
| 6 | Role/archetype | `Witty companion, loyal aide, creative partner` |
| 7 | Voice/tone | `Warm and playful, sharp and direct, dry humor` |
| 8 | Content filters | `None — fully uncensored, or Light — skip extreme only` |
| 9 | Hard boundaries | `No child content, no non-consensual real-person, no illegal acts` |
| 10 | Physical description (for images) | *shows default; press Enter to customize* |
| 11 | **Anything else?** Physical quirks, personality details, catchphrases... |

All defaults are generic — no identifying info baked in.

---

## Local Generation Tools (Optional)

### `--install-sdcli` → Stable Diffusion (images)
- Builds **stable-diffusion.cpp** (`sd-cli`) from source
- Runs on your GPU (CUDA) or CPU
- **Hardware survey runs first** — tells you exactly what your GPU can handle:
  - **4–6 GB VRAM** → SD 1.5 (512×512)
  - **6–8 GB VRAM** → SDXL/Pony with CPU offload
  - **8–12 GB VRAM** → SDXL/Pony with quantized KV cache (q4_0)
  - **12+ GB VRAM** → Full SDXL/Pony, headroom for multiple models
- **Guide included**: SD 1.5 vs SDXL vs Pony comparison, LoRA explanation, **why you need a Character LoRA for consistent images**, free Civitai training walkthrough (15–30 images → 10–30 min → `.safetensors`)

### `--install-acestep` → ACE-Step (music)
- Builds **acestep.cpp** from source
- Text-to-music: lyrics + genre/tags → 30–60 second song with vocals
- **4 GB+ VRAM** (8 GB recommended for longer clips)
- Creates `generated_music/` output folder automatically

### `--install-all` → Both

> **No CUDA?** Tools build CPU-only (slow but works). Consider a cloud GPU for generation.

---

## The Hermes Architecture (Why Two Files?)

| File | Location | Scope | Purpose |
|------|----------|-------|---------|
| **SOUL.md** | `~/.hermes/SOUL.md` | Global — all sessions, all projects | **WHO** — identity, personality, hard rules (Slot #1 in system prompt) |
| **AGENTS.md** | Project root folder | Per-directory — auto-switches on `cd` | **WHAT** — paths, conventions, tool usage, project rules |

This separation keeps your companion's *identity* consistent everywhere while letting *project rules* change per folder.

---

## Directory Structure

```
my-companion/
├── SOUL.md                 # → ~/.hermes/SOUL.md (global identity)
├── AGENTS.md               # → project root (project rules)
├── RAG/                    # Knowledge base (you populate)
│   ├── Music/
│   ├── TV_Shows/
│   ├── Movies/
│   ├── Video_Games/
│   ├── AI_Development/
│   ├── User_Profile/
│   ├── Creative_Writing/
│   └── Technical_Docs/
├── TOOLS/                  # Local binaries (if installed)
│   ├── sd-cli              # Stable Diffusion CLI
│   └── acestep.cpp         # ACE-Step music CLI
├── generated_images/       # Default image output
├── generated_music/        # Default music output (if ACE-Step installed)
├── Workspace/              # Your coding projects
├── Personality/            # Core docs (wardrobe, manual, etc.)
├── SKILLS/                 # Hermes skills
└── scripts/                # backup.sh, sync_to_remote.sh, test_identity.py
```

---

## Character Consistency — The LoRA Requirement

**Consistent character images REQUIRE a trained Character LoRA.**

Without it, every generation is a different person who merely *resembles* your description — hair shifts, face drifts, tattoos appear/disappear.

### Free Character LoRA on Civitai

1. **Create free account** → civitai.com
2. **Gather 15–30 reference images** (consistent angle/lighting/outfit for strict consistency, or varied poses for flexibility)
3. **Create → Train LoRA → Character**
4. **Upload images**, add unique trigger word (e.g., `aria_char`)
5. **Start training** — 10–30 min on free tier queue
6. **Download** `.safetensors` when done
7. **Place** in your model's LoRA folder
8. **Use** `<lora:aria_char:1>` in prompts

**Pro tips:** Unique trigger word (not in base vocab), caption with Booru tags (WD14 tagger), rank 32–64, 10–20 epochs, test at 0.8–1.0 weight.

---

## Maintenance

```bash
# Backup profile (excludes generated_images/, TOOLS/, logs)
./scripts/backup.sh [~/backups]

# Sync to remote host (BlueBox pattern)
./scripts/sync_to_remote.sh bluebox /mnt/storage/my-companion

# Verify structure after changes
python3 scripts/test_identity.py

# Update SOUL.md → copy to ~/.hermes/SOUL.md → restart Hermes
# Update AGENTS.md → copy to project root (auto-loaded on cd)
```

---

## Requirements

| Runtime | Needed For |
|---------|------------|
| Python 3.8+ | Running the script |
| git, cmake, g++ | Building sd-cli / acestep.cpp |
| CUDA toolkit (nvcc) | GPU acceleration (optional but recommended) |
| psutil (optional) | More accurate hardware survey |
| Hermes Agent | Actually running the companion |

**stdlib only** — the script itself has zero external dependencies.

---

## Why This Exists

Building a persistent, personalized AI companion with Hermes involves:

- Writing a proper SOUL.md (global identity, Slot #1)
- Writing AGENTS.md (project rules, directory-scoped)
- Setting up RAG folders for hallucination resistance
- Configuring local generation tools with correct output paths
- Understanding hardware limits for model selection
- Learning Character LoRA training for consistent images

**This script does all of it in one run.** You answer questions, it generates the complete structure with guides, best practices, and working defaults.

---

## License

MIT — use it, modify it, share it.

---

## Related

- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — The agent framework
- [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) — Local SD inference
- [ACE-Step](https://github.com/ace-step/ace-step) — Local music generation
- [Civitai](https://civitai.com) — Free LoRA training & model hosting
