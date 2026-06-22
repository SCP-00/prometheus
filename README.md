<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=200&section=header&text=Prometheus&fontSize=50&fontColor=fff&animation=fadeIn&desc=Local%20AI%20Coding%20Agent%20%7C%20Ollama%20%7C%20Offline&descSize=16&descAlignY=65"/>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Ollama-000?logo=ollama&logoColor=white"/>
  <img src="https://img.shields.io/badge/MCP-000?logo=protocol&logoColor=white"/>
  <img src="https://img.shields.io/badge/RTX%203050-76B900?logo=nvidia&logoColor=white"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow"/>
</p>

---

## 🔥 Overview

**Prometheus** is a local-first AI coding agent powered by **Ollama**, built on top of MiMo Code (Xiaomi's fork of OpenCode). It runs entirely offline on consumer hardware — no API keys, no cloud dependency.

> Optimized for **RTX 3050 6GB** — 96K context, 21 tok/s, 5.4 GB VRAM usage.

---

## ✨ Features

### Core
- **Local LLM Orchestration**: Qwen 3.5 9B (orchestrator) + Qwen 3.5 4B (sub-agent) + Gemma4 (vision)
- **96K Context Window**: Full project awareness for large codebases
- **Persistent Memory**: SQLite FTS5 with 4 layers (Global, Project, Session, History)
- **Goal Verification**: Independent judge agent for task validation
- **Dynamic Workflow**: JavaScript sandbox for custom agent workflows

### Tools & Integration
- **MCP Server**: Model Context Protocol for browser automation, file system, and external tools
- **13 Compose Skills**: TDD, debug, brainstorm, plan, execute, parallel, review, etc.
- **Hardware Auto-Detection**: Automatic GPU/VRAM/RAM/CPU profiling
- **Voice Input**: TenVAD + MiMo ASR + sox integration
- **LSP Integration**: Real-time code analysis and completion

### Performance (RTX 3050 6GB)
| Model | VRAM | Tok/s | Role |
|:------|:----:|:-----:|:-----|
| Qwen 3.5 9B | 5.1 GB | 21.4 | Orchestrator (96K ctx) |
| Qwen 3.5 4B | 3.0 GB | 47.4 | Sub-agent (32K ctx) |
| Gemma4 E2B | 1.1 GB | 38.3 | Vision (8K ctx) |

---

## 🚀 Quick Start

```bash
# Prerequisites: Ollama installed and running

# Clone and install
bun install

# Start Prometheus
prometheus

# Or with specific profile
prometheus --profile balanced
prometheus --agent plan      # Read-only mode
prometheus --agent compose   # Skill-driven workflow
```

### Configuration
```jsonc
{
  "model": "ollama/qwen3.5-9b-code",      // 5.4GB VRAM, 96K ctx
  "small_model": "ollama/qwen3.5-4b-32k",  // 3.0GB VRAM, 32K ctx
  "default_agent": "build",
  "disabled_providers": ["mimo", "openai", "anthropic", "gemini"]
}
```

---

## 📁 Project Structure

```
prometheus/
├── scripts/            # Utility scripts (auto-config, build, detect)
├── ollama/             # Modelfiles for each model variant
├── mcp-browser-use/    # MCP server integration
├── knowledge.md        # System knowledge base
└── PLAN.md            # Development roadmap
```

---

## 📜 License

MIT — Based on [MiMo Code](https://github.com/XiaomiMiMo/MiMo-Code) (MIT) and [OpenCode](https://github.com/anomalyco/opencode).

---

<p align="center">
  <i>Local AI. No cloud. No API keys. Just code.</i>
</p>

<div align="center">
  <a href="https://github.com/SCP-00">SCP-00</a> •
  <a href="https://linkedin.com/in/buendia001">LinkedIn</a>
</div>
