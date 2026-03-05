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

2. Start (pick one):

   If your system has a supported GPU, prefer the matching GPU startup option for faster Ollama inference.
   This requires host-side GPU setup first (OS drivers + Docker GPU container support, for example NVIDIA Container Toolkit or AMD/ROCm container setup), which is not covered in this repo.
  
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

## Access

- Minecraft server: `localhost:${MC_SERVER_PORT}` (default `localhost:55916`)
- Mindcraft UI: `http://localhost:${MINDCRAFT_UI_PORT}` (default [http://localhost:18080](http://localhost:18080))
- Dynmap: `http://localhost:${DYNMAP_HOST_PORT}` (default [http://localhost:8123](http://localhost:8123))

## First Start Is Slow (Expected)

The first startup can take several minutes because:

- Ollama downloads model files into `ollama-data` (often multiple GB)
- Minecraft creates initial server/world data in `minecraft-data`
- Docker builds the bots image the first time

You can watch progress with:

```shell
docker compose logs -f
```

You can check Docker disk usage with:

```shell
docker system df -v
```

## Optional `.env` changes

Most users can keep `.env` unchanged.

- Host ports (if defaults are already in use):
  - `MC_SERVER_PORT` (default `55916`)
  - `MINDCRAFT_UI_PORT` (default `18080`)
  - `DYNMAP_HOST_PORT` (default `8123`)
- Linux only, if your user is not `1000:1000`:
  - set `BOT_UID` and `BOT_GID` to your actual ids (`id -u`, `id -g`)
- If you want another upstream source/branch for image build:
  - set `MINDCRAFT_REPO`
  - set `MINDCRAFT_REF`
- Bot startup objective prompt:
  - set `INIT_MESSAGE`
- If you use non-Ollama providers, add API keys in `.env`:
  - key names: `https://raw.githubusercontent.com/mindcraft-bots/mindcraft/refs/heads/develop/keys.example.json`
  - syntax example: `OPENROUTER_API_KEY=sk-or-v1-...`

## Grant Minecraft Admin

Run once after startup:

```shell
docker compose exec minecraft rcon-cli op <your_minecraft_username>
```

If you only want a user to spectate (without giving admin), this is enough:

```shell
docker compose exec minecraft rcon-cli gamemode spectator <your_minecraft_username>
```

## Watch Bots In-Game (Minecraft Beginner Steps)

After the server is running, this is a simple way to observe a bot from inside Minecraft Java:

1. Become admin/operator (see section above), or have an admin run:
   ```text
   /gamemode spectator <your_minecraft_username>
   ```
2. Join the server at `localhost:${MC_SERVER_PORT}` (default `localhost:55916`).
3. Switch to spectator mode:
   ```text
   /gamemode spectator
   ```
4. Teleport to bot 1:
   ```text
   /tp bot_1
   ```
5. Quickly left-click `bot_1` (attack button) to start spectating that entity.
   - Minecraft term: this is called spectating an entity (also known as "mob view" in spectator mode docs).
6. Press `F5` repeatedly to cycle camera view modes.
7. Press `Left Shift` (default dismount key) to stop spectating and return to free-fly spectator mode.

Permissions note for non-admin users:
- On Java Edition, `/gamemode` requires permission level 2.
- Non-admin players cannot run `/gamemode spectator` unless an admin grants the needed permission (for example via operator level or a permissions plugin).

## Customize Bots

- Edit profiles in `profiles/` (currently `bot_1.json`, `bot_2.json`).
- Add/remove profiles by updating `SETTINGS_JSON.profiles` in `docker-compose.yml`.
- For hardware-specific model selection help, see [`MODEL_SELECTION_PROMPT_GUIDE.md`](MODEL_SELECTION_PROMPT_GUIDE.md).
- Upstream model/profile format reference:
  - `https://github.com/mindcraft-bots/mindcraft?tab=readme-ov-file#model-specifications`
  - `https://github.com/mindcraft-bots/mindcraft?tab=readme-ov-file#model-customization`

For non-Ollama providers:
- change profile `model` fields to target your provider
- add matching API key(s) to `.env`
- optionally remove `ollama` / `ollama-pull` services and related `depends_on` if you do not want local Ollama

## Settings Override

Mindcraft default settings:
- `https://raw.githubusercontent.com/mindcraft-bots/mindcraft/refs/heads/develop/settings.js`
- `https://github.com/mindcraft-bots/mindcraft/blob/develop/settings.js`

This repo overrides selected values via `SETTINGS_JSON` in `docker-compose.yml`.
Extend that JSON object to override more settings (for example `max_messages`, `num_examples`, `init_message`).

### In-Game Chat Verbosity

If bots are too chatty in Minecraft chat, adjust these `SETTINGS_JSON` keys in `docker-compose.yml`:

- `chat_ingame`: enable/disable bot response messages in Minecraft chat
- `narrate_behavior`: enable/disable automatic action narration (for example "Picking up item!")
- `chat_bot_messages`: enable/disable bot-to-bot public chat messages

Another related setting in the same settings file:

- `only_chat_with`: limits who bots listen/respond to for general chat

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
