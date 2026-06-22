# PROMETHEUS v6 — PLAN MAESTRO

> **Versión:** 6.0 — 21-Jun-2026
> **Hardware:** RTX 3050 6GB | i5-13420H | 16GB RAM | NVMe 1TB
> **Stack:** MiMo Code + Ollama + Qwen3.5 4B + Qwen3.5 9B + Gemma4
> **Filosofía:** *MiMo es la plataforma. Prometheus es la personalidad.*

---

## ⚡ — PERFIL DE HARDWARE

### Modelos Quantizados (RTX 3050 6GB)

| Modelo | Base | VRAM | Velocidad | Contexto | Capas GPU |
|:-------|:-----|:----:|:---------:|:--------:|:---------:|
| **prometheus-orchestrator** | Qwen3.5 4B IQ4_XS | 5.5 GB | **48 tok/s** | **192K** | 99/99 (100%) |
| **prometheus-orchestrator-extended** | Qwen3.5 4B IQ4_XS | 5.5 GB | **26.7 tok/s** | **262K** | 25/36 (69%) |
| **prometheus-supervisor** | Qwen3.5 9B Q4_K_M | 5.6 GB | **27 tok/s** | **32K** | 35/35 (100%) |
| **gemma4:e2b** | Gemma4 E2B Q4_K_M | 2.0 GB | **38 tok/s** | **8K** | 15/15 (100%) |

### Model Swap Benchmarks

| Operación | Tiempo | VRAM | Carga/Descarga |
|:----------|:------:|:----:|:--------------:|
| Cargar 4B (desde idle) | ~2.2s | 5,434 MB | — |
| Cargar 9B (desde idle) | ~2.5s | 5,580 MB | — |
| Swap 4B ↔ 9B | ~5-6s | ~20 MB (post-unload) | Sin fugas ✅ |
| Swap 4B → Gemma4 | ~4s | 2,048 MB | Swap obligatorio |
| Swap Gemma4 → 4B | ~4s | 5,434 MB | |

**⚠️ REALIDAD DE VRAM (6 GB):**
- 4B (5.5 GB) + 9B (5.6 GB) = 11.1 GB > 6 GB ❌ → Swap obligatorio
- 4B (5.5 GB) + Gemma4 (2.0 GB) = 7.5 GB > 6 GB ❌ → Swap obligatorio
- Gemma4 NUNCA cabe con ningún otro modelo. Siempre requiere su propio swap.
- **Conclusión:** CERO paralelismo real. Todo es cola con model swap (~10s cada uno).

### Límite de Contexto Seguro (4B)

| Contexto | VRAM | Velocidad | Estado |
|:--------:|:----:|:---------:|:------|
| 32K | 3.2 GB | 48 tok/s | ✅ Cómodo |
| 128K | 4.6 GB | 48 tok/s | ✅ |
| **192K** | **5.5 GB** | **48 tok/s** | **✅ Safe (89%)** |
| 224K | 5.9 GB | 48 tok/s | ⚠️ Límite (96%) |
| 256K | OOM | — | ❌ Crasha |

---

## 📈 — HISTORIAL DE EVALUACIONES

### Benchmark 1: Headless (65/100) — 20-Jun-2026

| Categoría | Peso | Score | Problema |
|:----------|:----:|:-----:|:---------|
| Autonomía | 20% | 55 | 5 "continue" del usuario |
| Herramientas | 15% | 30 | Solo cat + ls |
| Paralelismo | 15% | 20 | 100% secuencial |
| Web Research | 12% | 30 | 1 URL, metodología pobre |
| Code Analysis | 12% | 40 | Bugs reales + JAWS_SECRET alucinado |
| Arquitectura | 10% | 25 | 1 fase vaga |
| Reporte | 8% | 30 | Tablas rotas, mezcla idiomas |
| Eficiencia | 8% | 40 | Tiempo ok, recursos desperdiciados |
| **Base Score** | | **35.0** | |
| **Error Bonus** | | **+30** | |
| **Final** | | **65** | Competente |

**Problemas:** Usó `ollama run` en vez de `@code-reviewer`, Playwright MCP nunca usado, reporte corrupto, bug alucinado (JAWS_SECRET), 5 intervenciones del usuario.

### Benchmark 2: TUI Interactive (~80/100) — 21-Jun-2026

| Categoría | Peso | Score | Señal |
|:----------|:----:|:-----:|:------|
| Autonomía | 20% | 60 | Single-prompt, no multi-step |
| Herramientas | 15% | 40 | read + glob, sin Playwright MCP |
| Paralelismo | 15% | 30 | Menciona sub-agentes, checkpoint lineal |
| Structured Output | 12% | 85 | `<output>` JSON válido |
| Web Research | 12% | 75 | "No existe" correcto, metodología vaga |
| Code Analysis | 10% | 75 | 13 bugs reales, 0 falsos positivos |
| Reporte | 8% | 80 | Bien estructurado |
| Eficiencia | 8% | 50 | Sin evidencia de paralelismo real |
| **Base Score** | | **59.6** (sin Arquitectura) | |
| **Error Bonus** | | **+20** | |
| **Final** | | **~80** | Muy bueno |

**Mejoras vs B1:** Reporte limpio, 0 alucinaciones, menciona @code-reviewer, web research correcta.

### Benchmark 3: Headless via mimo-headless.sh — 21-Jun-2026

| Resultado | Señal |
|:----------|:------|
| EXIT code | 0 |
| Prompt recibido | ✅ (char-by-char fix funcionó) |
| @code-reviewer iniciado | ✅ "Code-Reviewer Task" en log |
| @web-researcher iniciado | ✅ "Web-Researcher Task" en log |
| Playwright MCP | ✅ "1 MCP active" en status bar |
| Benchmark completado | ❌ Timeout 240s (necesita >300s) |

---

## 🗺️ — ROADMAP

### ✅ Completado

- [x] Benchmarks de 4B: 192K safe, 224K límite, 262K con 25 capas GPU
- [x] Model swap 4B↔9B: ~5-6s, 0 fugas VRAM
- [x] Modelos: orchestrator (4B-192K), orchestrator-extended (4B-262K), supervisor (9B-32K)
- [x] Template Jinja fix (parche GGUF para error 400)
- [x] Symlinks de config: ~/.config/mimocode, prometheus-eval-test/.mimocode, ~/.mimocode/skills
- [x] Wrapper /usr/local/bin/mimo con cleanup de VRAM
- [x] Skills: prometheus-master, explore, find, ask, next-steps
- [x] Fix: `question: allow` en permission (auto-acepta trust prompt)
- [x] Fix: mimo-headless.sh char-by-char UTF-8 safe
- [x] Fix: Symlink eval-test recreado (elimina error 401)
- [x] Fix: MIMOCODE_HOME embebido en mimo-headless.sh
- [x] Benchmark 1 superado (65), Benchmark 2 superado (~80)
- [x] F1.1: write_todos forzado en SKILL.md
- [x] F1.3: Context freshness check documentado
- [x] F2.1-F2.2-F2.4: Web-researcher prompts actualizados
- [x] F3.1-F3.2-F3.3: Auto-verificación + 2 pasadas en prompts 9B + SKILL.md
- [x] F4.2: <output> JSON en TODOS los prompts (9/9 agentes)

### 🟡 Fase 1 — Orquestación Multi-Step

| # | Tarea | Esfuerzo | Criterio de éxito |
|:-:|:------|:--------:|:-------------------|
| F1.1 | Forzar write_todos al inicio de tareas 2+ pasos | 15 min | Orquestador usa write_todos automáticamente |
| F1.2 | Documentar cuándo spawnear cada sub-agente | 30 min | SKILL.md tiene decision tree con señales concretas |
| F1.3 | Implementar context freshness check | 30 min | Si 50K+ tokens → spawn sub-agente con contexto fresco |
| F1.4 | Benchmark multi-step (5 pasos, headless) | 1 hr | Checkpoint muestra fases separadas, sin timeout |

### 🟡 Fase 2 — Web Research con Playwright MCP

| # | Tarea | Esfuerzo | Criterio de éxito |
|:-:|:------|:--------:|:-------------------|
| F2.1 | Forzar browser_navigate + browser_snapshot en web-researcher | ✅ | En prompt: ORDEN DE OPERACIONES con PASOS 1-5 |
| F2.2 | Multi-source verification en web-researcher | ✅ | Prompt: "repite PASOS 1-4 con SEGUNDA fuente" |
| F2.3 | Test: web research con Playwright MCP | 30 min | `browser_navigate` en tool usage log |
| F2.4 | Gravity Index en web-researcher | ✅ | Added to prompt + tool_allowlist (gravity_index) |

### 🟡 Fase 3 — Calidad de Análisis

| # | Tarea | Esfuerzo | Criterio de éxito |
|:-:|:------|:--------:|:-------------------|
| F3.1 | Reforzar anti-hallucination en prompts 9B | ✅ | 7 reglas anti-hallucination en code-reviewer |
| F3.2 | Auto-verificación post-review | ✅ | "DESPUES de escribir tabla, LEE CADA ARCHIVO de nuevo" |
| F3.3 | Self-correction cycle: 2 pasadas mínimo | ✅ | SKILL.md: "Las pasadas 1 y 2 son OBLIGATORIAS" |
| F3.4 | Code analysis benchmark | 1 hr | Score >= 85 en Code Analysis |

### 🟡 Fase 4 — Reportes y Output

| # | Tarea | Esfuerzo | Criterio de éxito |
|:-:|:------|:--------:|:-------------------|
| F4.1 | Template de reporte | ✅ Ya en SKILL.md |
| F4.2 | <output> JSON siempre presente | ✅ | Los 9 agentes tienen <output> JSON en prompt |
| F4.3 | Report quality benchmark | 30 min | Score >= 90 en Reporte |

### 🟢 Fase 5 — Benchmark Final

| # | Tarea | Esfuerzo | Score Target |
|:-:|:------|:--------:|:------------|
| F5.1 | Benchmark multi-step en TUI (5 pasos) | 1.5 hr | >= 85 |
| F5.2 | Benchmark headless (mimo-headless.sh, timeout 360s) | 2 hr | >= 80 |
| F5.3 | Benchmark final completo | 2 hr | >= 90 |

---

## 🔧 — CONFIGURACIÓN DEL SISTEMA

### Ollama Optimizations

```bash
export OLLAMA_FLASH_ATTENTION=1
export OLLAMA_KV_CACHE_TYPE=q4_0
```

Systemd override:
```ini
[Service]
Environment=OLLAMA_FLASH_ATTENTION=1
Environment=OLLAMA_KV_CACHE_TYPE=q4_0
```

### Wrapper `/usr/local/bin/mimo`

```
mimo (sin args)   → TUI interactiva
mimo serve/debug  → pasa directo al binary
mimo run "prompt" → mimo-headless.sh (bypass bug de 'mimo run')
mimo "prompt"     → mimo-headless.sh
```

### Symlinks

| Ubicación | Apunta a |
|:----------|:---------|
| `~/.config/mimocode/mimocode.jsonc` | `PROMETHEUS/.mimocode/mimocode.jsonc` |
| `prometheus-eval-test/.mimocode/` | `PROMETHEUS/.mimocode/` |
| `~/.mimocode/skills` | `PROMETHEUS/.mimocode/skills` |

### Fixes Aplicados

| Fix | Archivo | Problema |
|:----|:--------|:---------|
| `question: allow` | `mimocode.jsonc` | Trust prompt bloqueaba headless mode |
| Char-by-char UTF-8 safe | `mimo-headless.sh` | Pipe stdin no funcionaba con TUI de MiMo |
| Symlink eval-test | `prometheus-eval-test/.mimocode/` | Error 401 "Invalid API Key" |
| Jinja template (GGUF) | `prometheus-orchestrator` | Error 400 con system message en posición ≠ 0 |

---

## ⚠️ — RIESGOS Y MITIGACIONES

| Riesgo | Prob. | Impacto | Mitigación |
|:-------|:-----:|:-------:|:-----------|
| 4B saturado en sesiones largas | Alta | Medio | Spawn sub-agentes con contexto fresco (>50K tokens) |
| Playwright MCP no se usa | Media | Alto | Forzar en prompt + verificar en log |
| Sub-agentes 9B no spawnen | Media | Alto | Verificar con log (visto: se spawnen) |
| VRAM insuficiente | Alta | Bajo | Documentado: 1 modelo a la vez. Siempre swap. |
| Jinja fix se pierde | Media | Medio | Documentado + check-templates.sh |
| Benchmark timeout | Alta | Medio | Aumentar a 360s. Si sigue, reducir scope. |
| Reportes sin <output> JSON | Media | Alto | Checklist obligatorio en SKILL.md |

---

## 📁 — ESTRUCTURA DEL PROYECTO

```
PROMETHEUS/
├── PLAN.md                        ← Este archivo
├── knowledge.md                   ← Base de conocimiento técnico
├── .mimocode/
│   ├── mimocode.jsonc             ← Config CANONICAL
│   └── skills/
│       ├── prometheus-master/SKILL.md
│       ├── explore/SKILL.md
│       ├── find/SKILL.md
│       ├── ask/SKILL.md
│       └── next-steps/SKILL.md
├── scripts/
│   ├── mimo-headless.sh
│   ├── check-templates.sh
│   └── ...
├── Hephaestus/
│   └── PLAN.md                    ← Roadmap detallado
└── knowledge.md
```
