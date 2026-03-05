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
- Embedding model = profile.embedding

Current defaults:
- model: Sweaterdog/Andy-4:latest
- code_model: qwen2.5-coder:7b
- embedding: embeddinggemma

Please read these links directly and base recommendations on them:
- https://ollama.com/Sweaterdog/Andy-4
- https://ollama.com/search?q=andy+minecraft
- https://ollama.com/search?q=instruct
- https://ollama.com/search?q=coder

Task:
1) Recommend 3 profile sets for my hardware:
   - recommended (balanced default)
   - fallback_light (faster / lower memory)
   - fallback_quality (better quality / heavier)
2) For each set, provide:
   - exact Ollama tag for model
   - exact Ollama tag for code_model
   - exact Ollama tag for embedding
   - short reason with expected resource impact
3) Avoid recommendations likely unstable for my VRAM/RAM.
4) Keep embeddinggemma unless you have a strong reason to change it.
5) Output exact `ollama pull` commands for all recommended tags.
6) Output final JSON ready to paste into both `profiles/bot_1.json` and `profiles/bot_2.json`.
```

## 3) Apply The Result

Pick one recommended set and update:

- `profiles/bot_1.json`
- `profiles/bot_2.json`

Replace only these fields unless you intentionally change more:

- `model.model`
- `code_model.model`
- `embedding.model`
