# mindcraft-test

Small user-focused demo setup for `mindcraft`:
- Minecraft server in Docker (Paper + Dynmap)
- Ollama in Docker for local LLM inference
- 2 bot profiles running through upstream `mindcraft`

## Quickstart

1. Copy env template:

```bash
cp .env.example .env
```

2. Start (pick one):
- CPU only:
  - `docker compose up -d --build`
- NVIDIA GPU:
  - `docker compose -f docker-compose.yml -f docker-compose.nvidia.yml up -d --build`
- AMD GPU (ROCm host):
  - `docker compose -f docker-compose.yml -f docker-compose.amd.yml up -d --build`

The tracked `bots/bot_or_1` and `bots/bot_or_2` folders already exist (`.keep` files), so no manual folder creation is needed.

## Optional `.env` changes

Most users can keep `.env` unchanged.

- Linux only, if your user is not `1000:1000`:
  - set `BOT_UID` and `BOT_GID` to your actual ids (`id -u`, `id -g`)
- If you want another upstream source/branch for image build:
  - set `MINDCRAFT_REPO`
  - set `MINDCRAFT_REF`
- If you use non-Ollama providers, add API keys in `.env`:
  - key names: `https://raw.githubusercontent.com/mindcraft-bots/mindcraft/refs/heads/develop/keys.example.json`
  - syntax example: `OPENROUTER_API_KEY=sk-or-v1-...`

## Customize Bots

- Edit profiles in `profiles/` (currently `bot_or_1.json`, `bot_or_2.json`).
- Add/remove profiles by updating `SETTINGS_JSON.profiles` in `docker-compose.yml`.
- Upstream model/profile format reference:
  - `https://github.com/mindcraft-bots/mindcraft?tab=readme-ov-file#model-specifications`
  - `https://github.com/mindcraft-bots/mindcraft?tab=readme-ov-file#model-customization`

For non-Ollama providers:
- change profile `model` fields to target your provider
- add matching API key(s) to `.env`
- optionally remove `ollama` / `ollama-pull` services and related `depends_on` if you do not want local Ollama

## Settings Override

Upstream default settings:
- `https://raw.githubusercontent.com/mindcraft-bots/mindcraft/refs/heads/develop/settings.js`

This repo overrides selected values via `SETTINGS_JSON` in `docker-compose.yml`.
Extend that JSON object to override more settings (for example `max_messages`, `num_examples`, `init_message`).

## Cleanup

```bash
# Stop and remove project containers + network
docker compose down

# Remove local project image
docker image rm mindcraft-bot:local

# Optional: clean Docker build cache
docker builder prune -f
```

To truly remove everything:

```bash
# Also remove project volumes.
# WARNING: this deletes Minecraft world data AND Ollama model data.
# Next startup will need to re-download models (can take a long time).
docker compose down -v
```
