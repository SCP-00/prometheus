# KNOWLEDGE BASE — Prometheus

> **Propósito:** Base de conocimiento técnico. Contiene fixes documentados,
> procedimientos de verificación, y todos los workarounds necesarios para
> mantener el sistema funcionando.

---

## 🔧 — FIX 1: Jinja Template (Error 400)

**Problema:** Qwen3.5-4B lanza `raise_exception('System message must be at the beginning.')`
cuando MiMo envía un system message en posición ≠ 0. Causa error HTTP 400.

**Afecta:** `prometheus-orchestrator` (4B-192K), `prometheus-orchestrator-extended` (4B-262K)
**No afecta:** `prometheus-supervisor` (9B-32K), `gemma4:e2b`

### Verificar

```bash
ollama show prometheus-orchestrator --template | grep 'raise_exception'
# Output → NO fixeado
# No output → Fixeado
```

### Re-aplicar (si se pierde)

```bash
# 1. Ver el script Python completo al final de este archivo
# 2. pip install gguf numpy
# 3. Ejecutar script → reemplaza 66 bytes: raise_exception → comentario
# 4. Verificar
ollama show prometheus-orchestrator --template | grep 'raise_exception' && echo "❌" || echo "✅"
```

---

## 🔧 — FIX 2: question:allow

**Problema:** MiMo TUI se bloquea en "Quick safety check" (trust prompt) en modo headless.

**Solución en `mimocode.jsonc`:**
```jsonc
"permission": {
    "question": "allow",
    "edit": { "PROMETHEUS/**": "allow" },
    "external_directory": { ... }
}
```

**Verificar:**
```bash
python3 -c "import json; c=json.load(open('PROMETHEUS/.mimocode/mimocode.jsonc')); print(c['permission'].get('question','❌'))"
# Debe mostrar: allow
```

---

## 🔧 — FIX 3: mimo-headless.sh Char-by-Char (UTF-8 Safe)

**Problema:** `printf '%s' "$PROMPT" | tmux send-keys -` no funciona con el input handler
del TUI de MiMo. El TUI muestra "The user's message is empty".

**Solución en `mimo-headless.sh`:**
```bash
while IFS= read -rn1 char; do
  tmux send-keys -t "$SESSION" "$char"
  sleep 0.015
done < <(printf '%s' "$PROMPT")
sleep 0.3
tmux send-keys -t "$SESSION" Enter
```

**Ventajas:** UTF-8 safe (soporta emoji, acentos), 67 chars/s, el TUI procesa cada tecla como input real.

---

## 🔧 — FIX 4: Symlink Eval-Test

**Problema:** Symlink `prometheus-eval-test/.mimocode/` perdido → error 401 "Invalid API Key".

**Solución:**
```bash
mkdir -p /home/buendia001/Desktop/prometheus-eval-test/.mimocode
ln -sf /home/buendia001/Projects/PROMETHEUS/.mimocode/mimocode.jsonc \
       /home/buendia001/Desktop/prometheus-eval-test/.mimocode/mimocode.jsonc
ln -sf /home/buendia001/Projects/PROMETHEUS/.mimocode/skills \
       /home/buendia001/Desktop/prometheus-eval-test/.mimocode/skills
```

---

## 🧪 — PROCEDIMIENTOS DE VERIFICACIÓN

### Verificar sub-agente spawneado

```bash
grep -E 'sub-agent|spawn|code-reviewer|Code-Reviewer|9B.*swap' /tmp/mimo_tui_*.log
# Señales de SÍ: "Code-Reviewer Task", "Using model prometheus-supervisor"
# Señales de NO: output genérico sin estructura, sin pausa de swap
```

### Verificar Playwright MCP conectado

```bash
grep -E 'MCP|playwright|browser.*active|1 MCP' /tmp/mimo_tui_*.log
# Debe mostrar: "1 MCP active"
```

### Verificar JSON de configuración

```bash
python3 -c "import json; json.load(open('PROMETHEUS/.mimocode/mimocode.jsonc')); print('✅')"
```

### Verificar skills accesibles

```bash
ls PROMETHEUS/.mimocode/skills/*/SKILL.md
# Debe listar: ask, explore, find, next-steps, prometheus-master
```

### Smoke test (benchmark rápido)

```bash
MIMOCODE_HOME=/home/buendia001/Projects/PROMETHEUS/.mimocode \
  timeout 120 ./PROMETHEUS/scripts/mimo-headless.sh "responde solo: ok"
# Esperado: EXIT 0, output contiene "ok"
```

---

## 📊 — LÍMITES DE VRAM

| Combinación | VRAM Total | ¿Cabe en 6GB? |
|:------------|:----------:|:-------------:|
| 4B solo | 5.5 GB | ✅ |
| 9B solo | 5.6 GB | ✅ |
| Gemma4 solo | 2.0 GB | ✅ |
| 4B + 9B | 11.1 GB | ❌ Swap obligatorio |
| 4B + Gemma4 | 7.5 GB | ❌ Swap obligatorio |
| 9B + Gemma4 | 7.6 GB | ❌ Swap obligatorio |

**Regla:** 1 modelo a la vez. Siempre. Swap ~10s.

### Costo de Swaps

| Operación | Costo | Cuándo |
|:----------|:-----:|:-------|
| 4B (default) | 0s | Siempre cargado |
| 9B → code review | ~10s | Después de cambios |
| 9B → debugger | ~10s | Errores de runtime |
| 9B → architect | ~10s | Decisiones de diseño |
| Gemma4 → web research | ~10s | Solo si requiere screenshots |

---

## 📝 — HISTORIAL

| Fecha | Cambio |
|:------|:-------|
| 20-Jun-2026 | Creación (Jinja fix documentado) |
| 21-Jun-2026 | Fix question:allow, char-by-char, symlink |
| 21-Jun-2026 | Reestructuración: fixes + verificación + VRAM limits |

---

## JINJA2 TEMPLATE FIX — Script de Parche

> ⚠️ **Problema recurrente.** Cada vez que se regenera el modelo, el parche se pierde.
> Este parche es para Qwen3.5-4B (IQ4_XS) solamente. El 9B no tiene este problema.

### Verificar

```bash
ollama show prometheus-orchestrator --template | grep 'raise_exception.*System message must be at the beginning'
# Output → NO fixeado. No output → Fixeado.
```

### Script Python

```python
import gguf
import numpy as np
import hashlib
import json
import shutil
import os

BLOB_PATH = "/usr/share/ollama/.ollama/models/blobs/sha256-43a1c472942a537ebd96bb3340fece099b64e8ed161dcbf6ee2ca5ee2b1975a6"
OLD_DIGEST = "sha256:43a1c472942a537ebd96bb3340fece099b64e8ed161dcbf6ee2ca5ee2b1975a6"
TEMPLATE_OLD = b"{{- raise_exception('System message must be at the beginning.')"
TEMPLATE_NEW  = b"{#- system message unexpected pos - already handled above -#}  "
MANIFEST_PATH = "/usr/share/ollama/.ollama/models/manifests/registry.ollama.ai/library/prometheus-orchestrator/latest"

reader = gguf.GGUFReader(BLOB_PATH, "r+")
field = reader.fields["tokenizer.chat_template"]
template_part = None
for part in field.parts:
    if part.dtype == np.uint8 and part.size > 100:
        template_part = part
        break

old_bytes = bytes(template_part)
idx = old_bytes.find(TEMPLATE_OLD)
if idx < 0:
    print("ERROR: raise_exception no encontrado (¿ya parchado?)")
    reader.close()
    exit(1)

new_bytes = old_bytes[:idx] + TEMPLATE_NEW + old_bytes[idx + len(TEMPLATE_OLD):]
template_part[:] = np.frombuffer(new_bytes, dtype=np.uint8).reshape(template_part.shape)
print("OK: raise_exception eliminado" if b"raise_exception" not in bytes(template_part) else "ERROR: aún presente")
reader.close()

new_digest = "sha256:" + hashlib.sha256(open(BLOB_PATH, "rb").read()).hexdigest()
new_blob_path = f"/usr/share/ollama/.ollama/models/blobs/{new_digest.replace(':', '-')}"
shutil.copy2(BLOB_PATH, new_blob_path)
print(f"Nuevo blob: {new_blob_path}")

with open(MANIFEST_PATH) as f:
    manifest = json.load(f)
for layer in manifest["layers"]:
    if layer["digest"] == OLD_DIGEST:
        layer["digest"] = new_digest
        layer["size"] = os.path.getsize(new_blob_path)

config_digest = manifest["config"]["digest"]
config_path = f"/usr/share/ollama/.ollama/models/blobs/{config_digest.replace(':', '-')}"
with open(config_path) as f:
    config = json.load(f)
for layer in config.get("diff_ids", []):
    if layer.get("mediaType") == "application/vnd.ollama.image.model" and "digest" in layer:
        layer["digest"] = new_digest
        break

new_config_digest = "sha256:" + hashlib.sha256(json.dumps(config, separators=(",", ":")).encode()).hexdigest()
with open(f"/usr/share/ollama/.ollama/models/blobs/{new_config_digest.replace(':', '-')}", "w") as f:
    json.dump(config, f)
manifest["config"]["digest"] = new_config_digest
manifest["config"]["size"] = os.path.getsize(
    f"/usr/share/ollama/.ollama/models/blobs/{new_config_digest.replace(':', '-')}"
)

with open(MANIFEST_PATH, "w") as f:
    json.dump(manifest, f, indent=2)
print(f"Manifiesto actualizado. Digest: {new_digest}")
```

> **⚠️ Regla de los 66 bytes:** TEMPLATE_NEW debe tener exactamente 66 bytes.
> Si la longitud cambia, los offsets del GGUF se corrompen.
