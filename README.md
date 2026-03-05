# mindcraft-test

Slim runtime repo for local `mindcraft` experiments.

This repo keeps only local runtime artifacts (compose, profiles, minecraft config, docs) and builds the bot image directly from upstream `mindcraft`.

## Included migration scope

- Files changed on `mindcraft` branch `Wuodan` in range `200edcd7..HEAD`:
  - `docker-compose.yml`
  - `minecraft-config/bukkit.yml`
  - `minecraft-config/ops.json`
  - `profiles/bot_or_1.json` to `profiles/bot_or_5.json`
  - `OPENROUTER_FREE_MODELS.md`
  - `OLLAMA_LENOVO_LEGION_PRO_5_ANALYSIS.md`
- Adaptation: bot image is built from upstream `mindcraft` `develop` (configurable).

## Use

1. Create your local `.env` file:
   - `cp .env.example .env`
2. Set host user mapping in `.env`:
   - `BOT_UID=<your uid>` (for example `1000`)
   - `BOT_GID=<your gid>` (for example `1000`)
3. Add API keys to `.env` as needed:
   - Upstream key names are listed in:
     - `https://raw.githubusercontent.com/mindcraft-bots/mindcraft/refs/heads/develop/keys.example.json`
   - Syntax example:
     - `OPENROUTER_API_KEY=sk-or-v1-...`
     - `OPENAI_API_KEY=...`
4. Create host bind-mount directories as your user:
   - `mkdir -p bots/bot_or_1`
5. Optional upstream branch override in `.env`:
   - `MINDCRAFT_REF=develop`
6. Start with one of these modes:
   - No GPU / CPU only:
     - `docker compose up --build`
   - NVIDIA GPU:
     - `docker compose -f docker-compose.yml -f docker-compose.nvidia.yml up --build`
   - AMD GPU (ROCm-capable host):
     - `docker compose -f docker-compose.yml -f docker-compose.amd.yml up --build`

## Notes

- `bots/` is mounted into the container for generated action code/runtime state.
- API keys are read from container environment variables via `.env` (`env_file` in compose), not from `keys.json`.
- Upstream default settings come from:
  - `https://raw.githubusercontent.com/mindcraft-bots/mindcraft/refs/heads/develop/settings.js`
- This repo overrides selected settings via `SETTINGS_JSON` in `docker-compose.yml`.
- You can extend overrides by adding fields to that JSON object, for example:
  - `"max_messages": 25`
  - `"num_examples": 4`
  - `"chat_bot_messages": false`
