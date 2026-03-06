# Model Selection Prompt Guide (For Other Hardware)

Your sample profiles in `profiles/` are tuned for one specific machine. If your hardware is different, use this guide to ask ChatGPT/Copilot/Gemini for model recommendations tailored to your PC.

## 1) Gather Hardware Info

Collect these three values:

- CPU model (for example `AMD Ryzen 7 5800X`)
- GPU model + VRAM (for example `NVIDIA RTX 4060 8GB`)
- System RAM (for example `32GB`)

### Command Snippets To Get These Values

Windows (PowerShell):

```powershell
# CPU model
(Get-CimInstance Win32_Processor).Name

# GPU model + VRAM (GB)
Get-CimInstance Win32_VideoController |
  Select-Object Name, @{Name='VRAM_GB';Expression={[math]::Round($_.AdapterRAM / 1GB, 1)}}

# System RAM (GB)
[math]::Round((Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory / 1GB, 1)
```

RedHat/Fedora (bash):

```bash
# CPU model
lscpu | awk -F: '/Model name/ {gsub(/^[ \t]+/, "", $2); print $2}'

# GPU model
lspci | grep -Ei 'vga|3d|display'

# System RAM
free -h | awk '/Mem:/ {print $2}'

# NVIDIA VRAM (if NVIDIA drivers/tools are installed)
nvidia-smi --query-gpu=name,memory.total --format=csv,noheader

# AMD VRAM (if ROCm tools are installed)
rocm-smi --showproductname --showmeminfo vram
```

Debian/Ubuntu (bash):

```bash
# CPU model
lscpu | awk -F: '/Model name/ {gsub(/^[ \t]+/, "", $2); print $2}'

# GPU model
lspci | grep -Ei 'vga|3d|display'

# System RAM
free -h | awk '/Mem:/ {print $2}'

# NVIDIA VRAM (if NVIDIA drivers/tools are installed)
nvidia-smi --query-gpu=name,memory.total --format=csv,noheader

# AMD VRAM (if ROCm tools are installed)
rocm-smi --showproductname --showmeminfo vram
```

If `lscpu` or `lspci` are missing:

- Debian/Ubuntu: `sudo apt install util-linux pciutils`
- RedHat/Fedora: `sudo dnf install util-linux pciutils`

## 2) Use This Prompt Template

Replace placeholders and paste into your browser AI:

```text
I am setting up Mindcraft Minecraft bots with local Ollama models.

My hardware:
- CPU: <CPU_MODEL>
- GPU: <GPU_MODEL_AND_VRAM>
- RAM: <SYSTEM_RAM_GB>

Important runtime context:
- Minecraft server runs on the same machine.
- 2 Mindcraft bots run in parallel.
- Ollama also runs on the same machine.
- So recommendations must leave enough CPU/GPU/RAM headroom for Minecraft + bots + Ollama together.

Profiles use:
- Main model = profile.model
- Coding/tool model = profile.code_model

Current defaults:
- model: sweaterdog/andy-4:micro-q5_k_m
- code_model: qwen2.5-coder:7b

Please read these links directly and base recommendations on them:
- https://ollama.com/Sweaterdog/Andy-4
- https://ollama.com/search?q=andy+minecraft
- https://ollama.com/search?q=instruct
- https://ollama.com/search?q=coder

Task:
Recommend exactly two Ollama model tags for my hardware:
1) one tag for `model` (main bot model)
2) one tag for `code_model` (coding/tool model)

Keep recommendations practical for my VRAM/RAM and this multi-service setup.
```

## 3) Apply The Result

Update:

- `profiles/bot_1.json`
- `profiles/bot_2.json`
- `docker-compose.yml` (`ollama-pull` service, `OLLAMA_MODELS: "..."`)

Replace only these model names:

- `model.model`
- `code_model.model`

Keep `embeddinggemma` unchanged unless you intentionally want a different embedding model.
