# Mindcraft-Test

Small user-focused demo setup for [Mindcraft](https://github.com/mindcraft-bots/mindcraft):
- Minecraft server in Docker ([PaperMC](https://papermc.io/) + [Dynmap plugin](https://www.curseforge.com/minecraft/bukkit-plugins/dynmap))
- [Ollama](https://github.com/ollama/ollama) in [Docker](https://www.docker.com/) for local LLM inference
- 2 bot profiles running through upstream `mindcraft`

## Prerequisites

- [Git](https://git-scm.com/) installed
- [Docker](https://www.docker.com/) installed and running
- Optional: Minecraft Java client `1.21.6` if you want to join the server and watch bots in-game

## Getting Started

1. Clone this repo:

   ```shell
   git clone https://github.com/Wuodan/mindcraft-test.git
   ```

2. Enter the project folder:

   ```shell
   cd mindcraft-test
   ```

3. Continue with Quickstart below.

## Quickstart

1. Copy env template:
   
   ```shell
   cp .env.example .env
   ```
   
   > **Hotfix**  
   > While [Mindcraft issue #728](https://github.com/mindcraft-bots/mindcraft/issues/728) is still open, use this in the `.env` file:
   > ```text
   > MINDCRAFT_REPO=https://github.com/Wuodan/mindcraft.git
   > MINDCRAFT_REF=fix/728-docker-apply-patches
   > ```

2. Start (pick one):
  
   - CPU only:  
     ```shell
     docker compose up -d
     ```
   - NVIDIA GPU:  
     ```shell
     docker compose -f docker-compose.yml -f docker-compose.nvidia.yml up -d
     ```
   - AMD GPU:  
     ```shell
     docker compose -f docker-compose.yml -f docker-compose.amd.yml up -d
     ```

## Optional `.env` changes

Most users can keep `.env` unchanged.

- Minecraft admin usernames:
  - set `OPS` (comma-separated), for example `OPS=DeKuen` or `OPS=DeKuen,OtherUser`
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

```shell
# Stop and remove project containers + network
docker compose down

# Remove local project image
docker image rm mindcraft-bot:local

# Optional: clean Docker build cache
docker builder prune -f
```

Remove data of bots:
```shell
rm -rf bots/*/*
```

Remove Minecraft server data (the world):
```shell
# WARNING: this deletes Minecraft world dataa.
docker compose down -v minecraft
```

Remove Ollama models:

```shell
# WARNING: this deletes Ollama model data.
# Next startup will need to re-download models (can take a long time).
docker compose down -v ollama
```
