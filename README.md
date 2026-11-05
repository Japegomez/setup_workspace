# Tutorial: configurar un workspace agéntico desde cero

Guía paso a paso para crear un **workspace y un repositorio nuevos** preparados para desarrollar con agentes de IA. Cada herramienta tiene instrucciones para **cualquier IDE/agente** (Claude Code, Codex, Copilot, Windsurf, OpenCode…) y una sección concreta para **Cursor**.

Los pasos siguen un orden lógico **de menor a mayor importancia** en el desarrollo de software: primero lo opcional y de consulta (elegir modelo, gateways, acceso web), después lo que ahorra tokens y amplía el agente, luego revisión automática y escaneo de vulnerabilidades, y al final lo que más impacto tiene en la calidad del software: las rules del proyecto (`AGENTS.md`) y el desarrollo guiado por especificación (SDD).

> Basado en la *Guía de uso de IA en el desarrollo* (secciones 3.5, 5, 6, 7, 8 y 12). Trata cada herramienta de terceros como una dependencia: revisa permisos, licencia y qué datos envía antes de instalarla.

## Índice

0. [Requisitos y nuevo repositorio](#0-requisitos-y-nuevo-repositorio)
1. [Comparar modelos](#1-comparar-modelos)
2. [OmniRoute: gateway multi-proveedor](#2-omniroute-gateway-multi-proveedor)
3. [Agent Reach: acceso a web y canales](#3-agent-reach-acceso-a-web-y-canales)
4. [Plugins](#4-plugins)
5. [Skills de este repositorio](#5-skills-de-este-repositorio)
6. [Caveman: salida más corta](#6-caveman-salida-más-corta)
7. [RTK y Context Mode: menos ruido en el contexto](#7-rtk-y-context-mode-menos-ruido-en-el-contexto)
8. [Graphify: mapa del codebase](#8-graphify-mapa-del-codebase)
9. [CodeRabbit: revisión automática](#9-coderabbit-revisión-automática)
10. [Escaneo de vulnerabilidades](#10-escaneo-de-vulnerabilidades)
11. [`AGENTS.md`: las rules del proyecto](#11-agentsmd-las-rules-del-proyecto)
12. [Desarrollo guiado por especificación (SDD)](#12-desarrollo-guiado-por-especificación-sdd)
13. [Checklist final](#13-checklist-final)

---

## 0. Requisitos y nuevo repositorio

Varias herramientas dependen de **Git**, **Node.js 22.13+** (Caveman, OmniRoute, Context Mode, `npx skills`) y **Python + uv** (Graphify).

```bash
# macOS / Linux
brew install git node gh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

```powershell
# Windows
winget install Git.Git OpenJS.NodeJS.LTS GitHub.cli
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Instalación manual (webs oficiales)

Si no usas `brew` / `winget` o el instalador por script, descarga cada herramienta desde su web oficial, instálala y **cierra y vuelve a abrir la terminal** (o reinicia el IDE) para que el `PATH` se actualice.

| Herramienta | Web oficial | Qué instalar |
|---|---|---|
| **Git** | [https://git-scm.com/downloads](https://git-scm.com/downloads) | Instalador de tu SO (Windows: Git for Windows) |
| **Node.js** | [https://nodejs.org](https://nodejs.org) | Versión **LTS** (22.13+ recomendada para Caveman / `npx skills`) |
| **GitHub CLI (`gh`)** | [https://cli.github.com](https://cli.github.com) | Instalador o paquete de tu SO |
| **uv** (Python) | [https://docs.astral.sh/uv/getting-started/installation/](https://docs.astral.sh/uv/getting-started/installation/) | Método oficial para tu plataforma (standalone, pip, etc.) |

Comprueba que quedaron en el PATH:

```bash
git --version
node --version
gh --version
uv --version
```

En Windows, si `uv` o `gh` no se reconocen tras instalar, cierra todas las ventanas de PowerShell/Cursor y ábrelas de nuevo; a veces hace falta un cierre completo de sesión.

Crea el workspace y el repositorio:

```bash
mkdir mi-proyecto && cd mi-proyecto
git init -b main
mkdir -p docs/specs
touch AGENTS.md README.md .gitignore
gh repo create mi-proyecto --private --source . --remote origin   # o el equivalente en GitLab
```

Estructura objetivo al terminar el tutorial:

```text
mi-proyecto/
├── AGENTS.md                 # rules del proyecto (paso 11)
├── .coderabbit.yaml          # reglas de revisión (paso 9)
├── .github/workflows/        # TruffleHog en CI (paso 10)
├── .cursor/rules/            # solo Cursor: graphify.mdc, context-mode.mdc
├── docs/specs/               # paquete SDD (paso 12)
│   ├── PRD.md
│   ├── TRD.md
│   ├── appflow.md
│   ├── ui-ux.md
│   ├── backend-schema.md
│   └── implementation-plan.md
└── graphify-out/             # grafo del codebase (paso 8)
```

**Cursor:** abre la carpeta con **File → Open Folder** (o `cursor .`). Cursor guarda su configuración de proyecto en `.cursor/` (rules, `mcp.json`, `hooks.json`); la global vive en `~/.cursor/`.

---

## 1. Comparar modelos

No se instala nada, pero conviene elegir modelo antes de empezar. Los mejores modelos cambian cada semana: consulta las comparativas a menudo en lugar de memorizar un ranking.

| Sitio | Enfoque | Úsalo para |
|---|---|---|
| [Artificial Analysis](https://artificialanalysis.ai/models) | Índice de inteligencia, velocidad y precio por proveedor | Separar familias (ligero / medio / frontera) y ver la latencia real del endpoint |
| [LLM Stats](https://llm-stats.com/) | Leaderboard de 300+ modelos con ejes Reasoning, Coding, Agent, contexto, velocidad y precio | Filtrar por **coding** o **agent**, comparar open-source y propietario, ver coste $/M |
| [Arena](https://arena.ai/leaderboard) | Ranking por votación humana a ciegas (antes LMArena) | Ver la preferencia real de usuarios en la categoría de **código** |

Cómo decidir: prioriza las columnas de coding y agent frente a un índice genérico, compara el precio por **tarea típica** (un modelo barato con cinco reintentos sale caro) y desconfía de benchmarks sin versión ni fecha. La mejor comparativa es probarlos en tu stack con tus propias tareas.

**Cualquier IDE:** el selector de modelo de cada harness (Claude Code `/model`, Codex `--model`, Copilot en el chat).

**Cursor:** selector de modelo del chat. Activa solo los modelos que uses en **Settings → Models**. Sube el *effort/thinking* para bugs difíciles o planes ambiguos; si el mismo modelo falla dos veces seguidas, cambia de modelo en lugar de seguir subiendo effort.

---

## 2. OmniRoute: gateway multi-proveedor

[OmniRoute](https://github.com/diegosouzapw/OmniRoute) es un gateway open-source (MIT) que expone **un único endpoint compatible con OpenAI** y enruta a cientos de proveedores, muchos con tier gratuito. Hace *fallback* automático cuando se agota una cuota y tiene estrategias de enrutado por coste, latencia o capacidad. Es opcional: útil cuando el coste frena la adopción.

**Instalación (cualquier sistema):**

```bash
npm install -g omniroute
omniroute            # arranca dashboard + API en http://localhost:20128
```

Con Docker:

```bash
docker run -d --name omniroute --restart unless-stopped --stop-timeout 40 \
  -e OMNIROUTE_MEMORY_MB=8192 --memory=10g \
  -p 127.0.0.1:20128:20128 -v omniroute-data:/app/data diegosouzapw/omniroute:latest
```

Abre el dashboard, conecta proveedores, crea una API key y verifica:

```bash
curl http://localhost:20128/v1/models -H "Authorization: Bearer TU_KEY"
```

**Cualquier IDE:** configura el cliente con base URL `http://localhost:20128/v1` y tu key (Claude Code, Codex, Cline, OpenCode y Copilot aceptan endpoints OpenAI-compatibles).

**Cursor:** **Settings → Models → API Keys**, activa *OpenAI API Key*, pega la key de OmniRoute y activa *Override OpenAI Base URL* con `http://localhost:20128/v1`. Cursor hace estas peticiones desde sus servidores: si `localhost` no responde, expón OmniRoute con un túnel (cloudflared, ngrok) protegido por la API key.

> Un tier gratuito sigue siendo un tercero: no envíes código ni datos confidenciales por proveedores cuya retención no hayas revisado.

---

## 3. Agent Reach: acceso a web y canales

[Agent Reach](https://github.com/Panniantong/Agent-Reach) da al agente acceso a web, GitHub, YouTube, RSS y otros canales. La comprobación por defecto **no cambia el sistema**: `agent-reach install` solo revisa el entorno. `--system` instala dependencias y registra la skill, y hay que pedirlo aparte. Los canales con login (Twitter, Reddit, Xiaohongshu…) tampoco se instalan solos: el agente pregunta cuáles quieres.

**Cualquier IDE / Cursor:** pega esto en el chat del agente y él hace el resto:

```text
帮我安装 Agent Reach：https://raw.githubusercontent.com/Panniantong/agent-reach/main/docs/install.md
```

Para actualizar:

```text
帮我更新 Agent Reach：https://raw.githubusercontent.com/Panniantong/agent-reach/main/docs/update.md
```

Comprobar el estado de cada canal:

```bash
agent-reach doctor
```

> Instala la CLI desde el repositorio oficial, no desde PyPI: el paquete homónimo de PyPI no es este proyecto. Desinstalar: `agent-reach uninstall` (borra también tokens y cookies de `~/.agent-reach/`).

---

## 4. Plugins

Un plugin empaqueta skills, rules y servidores MCP listos para usar. El empaquetado es propio de cada IDE (el marketplace de Cursor no es el de Claude Code ni el de Codex), aunque el MCP suele reutilizarse entre entornos.

**Cualquier IDE:**

- **Claude Code:** `/plugin marketplace add <owner/repo>` y después `/plugin install <plugin>`.
- **Otros (Codex, Copilot, Windsurf…):** instala el servidor MCP equivalente en su configuración (`mcp.json` o similar) desde la documentación de cada integración.

**Cursor:** en el chat ejecuta:

```text
/add-plugin asana context-mode context7-plugin continual-learning cursor-team-kit devtools-for-agents docs-canvas gamma github gitlab google-calendar google-docs google-sheets google-slides higgsfield posthog pr-review-canvas sentry supabase teaching thermos
```

Comprueba en **Settings → Plugins** y **Settings → MCP** que estén activos. Instala solo los que vayas a usar: cada MCP añade herramientas al contexto.

| Plugin | Qué hace |
|---|---|
| **asana** | Tareas, proyectos y comentarios de Asana. |
| **context-mode** | Reduce el contexto que llega al agente: filtra la salida de herramientas, guarda memoria de la sesión y enruta llamadas (paso 7). |
| **context7-plugin** | Documentación actual de librerías, sacada del repositorio y no del entrenamiento del modelo. |
| **continual-learning** | Aprende preferencias y hechos del workspace a partir de las conversaciones. |
| **cursor-team-kit** | Flujos de CI, revisión y entrega. |
| **devtools-for-agents** | Chrome DevTools para depurar páginas, red y rendimiento. |
| **docs-canvas** | Abre la documentación como un canvas navegable. |
| **gamma** | Genera presentaciones, documentos y páginas. |
| **github** / **gitlab** | Repositorios, issues, pull/merge requests y CI. |
| **google-calendar / docs / sheets / slides** | Agenda, documentos, hojas de cálculo y presentaciones de Google. |
| **higgsfield** | Genera imágenes y vídeo. |
| **posthog** | Analítica, feature flags, experimentos y errores. |
| **pr-review-canvas** | Muestra el diff de un PR agrupado por importancia. |
| **sentry** | Issues y stack traces de Sentry. |
| **supabase** | Proyectos, tablas y consultas de Supabase. |
| **teaching** | Planes de práctica y retrospectivas de aprendizaje. |
| **thermos** | Revisión estricta de una rama: bugs, seguridad y calidad. |

---

## 5. Skills de este repositorio

Una skill es una carpeta con un `SKILL.md` que enseña al agente a ejecutar una tarea concreta (TDD, revisión de código, commits, diseño de UI…). Este repositorio contiene en `skills/` la colección recomendada.

**Cualquier IDE** (el CLI [`skills`](https://skills.sh/) detecta los agentes instalados):

```bash
npx skills add Japegomez/cursor_skills          # instala la colección
npx skills add Japegomez/cursor_skills --list   # ver qué skills incluye antes de instalar
```

**Cursor:** instala de forma global para el agente de Cursor:

```bash
npx skills add Japegomez/cursor_skills -g -a cursor
```

Las skills quedan disponibles en el chat como `/nombre-skill` o se activan solas cuando la tarea encaja con su descripción.

Desde la raíz del proyecto, [Impeccable](https://github.com/pbakaus/impeccable) añade un lenguaje de diseño (`/impeccable audit`, `/polish`, `/critique`…) para que el agente diseñe interfaces con más criterio:

```bash
npx impeccable install
```

---

## 6. Caveman: salida más corta

[Caveman](https://github.com/JuliusBrussee/caveman) recorta tokens de salida: respuestas en voz telegráfica (respuesta primero, sin relleno, código y errores literales), salidas de herramientas comprimidas y, opcionalmente, selección de modelo. Ahorra en torno a un 65 % frente a no dar instrucciones y un 35 % extra con `/ultracave` frente a "be concise".

**Cualquier IDE** (instalador unificado, Node.js 22.13+; detecta tus agentes y muestra qué cambia antes de aplicarlo):

```bash
npx @caveman-ai/cli@latest
```

```powershell
# Windows, alternativa
irm https://raw.githubusercontent.com/JuliusBrussee/caveman/v4.0.1/install.ps1 | iex
```

Solo la skill de voz corta, sin proxy:

```bash
npx skills add JuliusBrussee/caveman -g
```

**Cursor:** instala skills, agentes Cavecrew y el hook de sesión solo para Cursor:

```bash
npx -y github:JuliusBrussee/caveman -- --only cursor
```

Uso en el chat: `/caveman` (día a día), `/ultracave` (revisiones masivas), `/caveman-review`, `/caveman-compress`, `/caveman-stats`. Para desactivar: `stop caveman`.

> Si pagas por petición (p. ej. GitHub Copilot), acortar respuestas no ahorra dinero; solo ganas velocidad de lectura.

---

## 7. RTK y Context Mode: menos ruido en el contexto

Las dos herramientas cubren extremos distintos del mismo problema. RTK recorta la salida de los **comandos de terminal** antes de que entre al contexto; Context Mode evita que la salida de **herramientas MCP** y el historial lleguen crudos al modelo.

### RTK

[RTK](https://github.com/rtk-ai/rtk) es un proxy de la terminal (un binario Rust). Cuando el agente lanza `git`, `ls`, tests o logs, un hook reescribe el comando a `rtk …`, que filtra, agrupa, trunca y deduplica la salida. Promete un 60–90 % menos de tokens en comandos habituales.

```bash
brew install rtk                                              # macOS / Linux
winget install rtk-ai.rtk                                     # Windows
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
rtk --version && rtk gain    # gain debe mostrar el panel de ahorro
```

Existe otro proyecto llamado `rtk` en crates.io: si `rtk gain` da "command not found", tienes el paquete equivocado.

**Cualquier IDE:** inicia el hook de tu agente.

```bash
rtk init -g                     # Claude Code / Copilot
rtk init -g --codex             # Codex
rtk init -g --gemini            # Gemini CLI
rtk init -g --agent windsurf    # Windsurf
rtk init --agent cline          # Cline / Roo Code
rtk init -g --opencode          # OpenCode
```

**Cursor:**

```bash
rtk init -g --agent cursor      # registra un hook preToolUse en hooks.json
```

### Context Mode

[Context Mode](https://github.com/mksglu/context-mode) es un servidor MCP que guarda los datos crudos en un sandbox y solo deja pasar el resultado útil (`ctx_execute`, `ctx_search`…), recuerda la sesión cuando el chat se compacta y obliga a analizar con un pequeño programa en lugar de volcar datos.

**Cualquier IDE:** instala el CLI global y regístralo como servidor MCP en la configuración de tu agente (comando `context-mode`); el README tiene configuraciones para Claude Code, Codex, OpenCode, Copilot, Kiro, Zed y otros.

```bash
npm install -g context-mode
```

**Cursor (opción A, plugin):** `/add-plugin context-mode` cuando aparezca en el Marketplace. Hasta entonces, plugin local + CLI global:

```powershell
# Windows (Cursor no sigue symlinks en Windows, por eso robocopy)
git clone https://github.com/mksglu/context-mode.git
cd context-mode
robocopy . "$env:USERPROFILE\.cursor\plugins\local\context-mode" /MIR `
  /XD node_modules .git build web tests scripts .vscode `
  /XF *.log .gitignore *.bundle.mjs.map
npm install -g context-mode
```

```bash
# macOS / Linux
git clone https://github.com/mksglu/context-mode.git
ln -s "$PWD/context-mode" ~/.cursor/plugins/local/context-mode
npm install -g context-mode
```

**Cursor (opción B, manual):** tras `npm install -g context-mode`, crea `.cursor/mcp.json` (o `~/.cursor/mcp.json` para global):

```json
{
  "mcpServers": {
    "context-mode": { "command": "context-mode" }
  }
}
```

Usa una sola de las dos opciones: si combinas plugin y `hooks.json` manual, los hooks se disparan dos veces. Reinicia Cursor, comprueba en **Settings → MCP** que `context-mode` esté conectado y escribe en el chat `ctx doctor` o `ctx stats`.

---

## 8. Graphify: mapa del codebase

[Graphify](https://github.com/Graphify-Labs/graphify) convierte el proyecto (código, docs, PDFs, imágenes) en un **grafo de conocimiento** local bajo `graphify-out/`: `graph.json`, `GRAPH_REPORT.md` y un HTML interactivo. El agente consulta el grafo (`query`, `path`, `explain`) en lugar de leer carpetas enteras, y abre solo los archivos que el grafo cita. Para código, la mayor parte del grafo sale de AST local, sin LLM.

**Instalación (cualquier sistema).** El paquete oficial de PyPI es `graphifyy` (doble y); el comando es `graphify`:

```bash
uv tool install graphifyy     # recomendado; si no aparece el comando: uv tool update-shell
# alternativas: pipx install graphifyy  |  pip install graphifyy
```

**Cualquier IDE:**

```bash
graphify install                       # detecta tu asistente y registra la skill
graphify install --project             # instala la skill dentro del repo (versionable)
graphify claude install                # Claude Code: hook que obliga a consultar el grafo
graphify codex install                 # Codex: escribe la instrucción en AGENTS.md
```

**Cursor:**

```bash
graphify install --platform cursor     # skill + regla .cursor/rules/graphify.mdc (alwaysApply)
```

Genera el grafo en el chat, desde la raíz del proyecto, y consúltalo:

```text
/graphify .
/graphify query "flujo de autenticación"
/graphify path "Login" "Database"
```

Mantenlo al día: `graphify update .` tras cambios de código (incremental, sin coste de API), `/graphify . --watch` mientras trabajas y un job de CI tras cada merge a `main`. Política típica: versionar `graphify-out/graph.json` y `GRAPH_REPORT.md`, e ignorar las cachés `.graphify_*`. Docs: [Graphify + Cursor](https://docs.graphify.com/integrations/cursor).

---

## 9. CodeRabbit: revisión automática

[CodeRabbit](https://www.coderabbit.ai/) revisa cada PR con IA: resume el cambio, señala bugs y riesgos, sugiere diffs y sigue las reglas del equipo. Hace el primer pase mecánico; la aprobación final sigue siendo humana.

**En el repositorio (cualquier IDE):**

1. Instala la app de CodeRabbit en GitHub o GitLab desde [coderabbit.ai](https://www.coderabbit.ai/) y dale acceso al repositorio.
2. Añade un `.coderabbit.yaml` en la raíz con reglas del equipo, por ejemplo: rechazar `any` en TypeScript, exigir paginación en listados o validar inputs en la frontera. Limita los comentarios a severidad media-alta para evitar fatiga.

**CLI (revisar antes de abrir el PR):**

```bash
curl -fsSL https://cli.coderabbit.ai/install.sh | sh     # macOS / Linux
brew install coderabbit                                   # alternativa
```

```powershell
irm https://cli.coderabbit.ai/install.ps1 | iex           # Windows
```

```bash
cr auth login                  # cr es el alias de coderabbit; --region eu para la región UE
cr review                      # cambios trackeados
cr review --uncommitted        # solo cambios locales sin commit
cr review --agent              # salida NDJSON para que la consuma un agente
coderabbit skills              # instala las skills de CodeRabbit en tus agentes
```

**Cursor:** instala la extensión de CodeRabbit desde el panel de extensiones (compatible con VS Code, Cursor y Windsurf) para revisar el diff desde el editor, o pide al agente que ejecute `cr review --agent` y corrija los hallazgos antes de abrir el PR.

---

## 10. Escaneo de vulnerabilidades

El código generado con IA introduce los mismos fallos que el humano (inyecciones, autenticación rota, secretos en claro) y algunos propios, como APIs "seguras" alucinadas. La revisión de PR (paso 9) no basta: monta **secret scanning automático** y, encima, **revisión con skills** cuando el cambio toca superficie sensible.

**Camino recomendado (y el único que documenta este tutorial):**

| Momento | Herramienta | Rol |
|---|---|---|
| **Local (pre-commit)** | [gitleaks](https://github.com/gitleaks/gitleaks) | Bloquea el commit en segundos si hay secreto en el working tree |
| **CI (cada PR y push a `main`)** | [TruffleHog](https://github.com/trufflesecurity/trufflehog) | Red de seguridad si alguien se salta el hook; verifica si el secreto **sigue vivo** (`--results=verified,unknown`) |
| **Antes de liberar** (opcional) | [Strix](https://github.com/usestrix/strix) | Pentest con PoC; **Docker debe estar corriendo** |

gitleaks es el filtro rápido del día a día; TruffleHog es la red en el PR. No pongas las dos en el mismo job de CI: solapan hallazgos y generan ruido.

### 1. Local: gitleaks

```bash
# macOS / Linux
brew install gitleaks
```

```powershell
# Windows: descarga el binario desde
# https://github.com/gitleaks/gitleaks/releases
```

```bash
gitleaks dir .          # working tree actual
gitleaks git -v         # historial completo (auditoría puntual)
```

Engancha el pre-commit copiando el `pre-commit.py` de ejemplo de gitleaks a `.git/hooks/pre-commit`, o usa el framework [`pre-commit`](https://pre-commit.com/) con el hook oficial de gitleaks. Así ningún commit con secreto sale del portátil.

### 2. CI: TruffleHog

Crea `.github/workflows/secret-scan.yml`:

```yaml
name: secret-scan
on:
  push:
    branches: [main]
  pull_request:
jobs:
  trufflehog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          extra_args: --results=verified,unknown
```

Escanea los commits del PR / push. Si aparece un secreto, considéralo comprometido: rótalo y revócalo. Borrarlo del último commit no basta; sigue en el historial. Reescribe o limpia el historial solo con política de equipo y, después, vuelve a pasar el job de CI.

> gitleaks ya solo recibe parches de seguridad; su autor continúa el trabajo en [Betterleaks](https://github.com/betterleaks/betterleaks). El CLI de gitleaks sigue siendo válido para el pre-commit.

### 3. Skills: cuándo usar cada una

Se instalan con la colección del repo (paso 5). No sustituyen a gitleaks/TruffleHog: cubren lógica, auth y flujos que un escáner de secretos no ve.

| Skill | Úsala cuando… | No la uses cuando… |
|---|---|---|
| **`security-review`** | Vas a abrir o fusionar un PR; quieres un pase rápido del **diff / rama / carpeta** con severidad y fix concreto | Necesitas un informe formal, artefactos o una auditoría end-to-end del repo |
| **`security-audit`** | Pides explícitamente una **auditoría completa**, pen-test defensivo o entregables (`architecture.md`, hallazgos, plan de validación) | Solo quieres un consejo puntual o un “¿esto es seguro?” sobre un archivo: en ese caso usa modo guía o `security-review` |

Regla práctica: **cada PR → `security-review`**; **cambio grande en auth/pagos/datos o release → `security-audit` en modo completo**.

### 4. Proceso paso a paso (por PR)

1. Desarrolla en la rama. Antes de cada commit, el pre-commit de **gitleaks** debe estar en verde (`gitleaks dir .` si lo lanzas a mano).
2. Abre el PR. El workflow de **TruffleHog** debe pasar; si falla, rota el secreto y limpia el historial antes de seguir.
3. En el chat del agente, lanza la revisión del cambio:

```text
/security-review revisa los cambios de esta rama antes del merge
```

   Si el alcance es una carpeta concreta: `/security-review src/auth/`.
4. El agente debe: acotar el scope → auditar dependencias → buscar secretos/exposición → trazar flujos de datos → clasificar por severidad → proponer parches **sin aplicarlos** hasta que tú apruebes.
5. Corrige CRITICAL/HIGH primero; vuelve a pasar `/security-review` (o pide re-scan del mismo scope) hasta que no queden hallazgos bloqueantes.
6. Solo si el PR toca autenticación, pagos, PII o una superficie nueva de ataque, o si preparas un release, lanza la auditoría completa:

```text
/security-audit audita el módulo de autenticación (modo completo, con informe)
```

7. Fusiona cuando: gitleaks local OK, TruffleHog CI verde, `security-review` sin CRITICAL/HIGH abiertos y, si aplica, el informe de `security-audit` revisado.

### 5. Pentesting opcional: Strix

[Strix](https://github.com/usestrix/strix) orquesta agentes de IA que atacan tu app como un red team y entregan cada hallazgo con prueba de concepto. Úsalo **antes de liberar** cambios en auth, pagos o datos (staging), no en cada PR.

**Requisito: Docker tiene que estar instalado y en marcha** antes de lanzar Strix. Si el daemon no corre, la CLI falla al crear contenedores.

```bash
# 1) Comprueba que Docker responde
docker info
# Si ves "Cannot connect to the Docker daemon", arráncalo primero.
```

```bash
# macOS: abre Docker Desktop (o)
open -a Docker

# Linux (systemd)
sudo systemctl start docker
sudo systemctl enable docker   # opcional: al arrancar el sistema

# Windows: inicia Docker Desktop desde el menú Inicio, o
Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"
```

Espera a que el icono de Docker Desktop diga *Running* (o `docker info` sin error) y entonces instala/ejecuta Strix:

```bash
curl -sSL https://strix.ai/install | bash        # o: pipx install strix-agent / uv tool install strix-agent
export STRIX_LLM="openrouter/z-ai/glm-5.3"       # proveedor/modelo
export LLM_API_KEY="tu-api-key"
strix --target ./mi-proyecto                     # código local
strix -n --target https://staging.mi-app.com     # app desplegada, modo no interactivo (CI)
```

En Windows, ejecútalo desde WSL **con Docker Desktop integrado** (Settings → Resources → WSL integration). Puedes apuntar `STRIX_LLM` a OmniRoute (paso 2) para las pasadas de reconocimiento masivo y reservar un modelo frontera para validar.

> Solo contra tus entornos de desarrollo o staging. Nunca contra producción ni contra sistemas de terceros sin autorización escrita.

### Política mínima

- Ningún PR se fusiona con el job `secret-scan` en rojo.
- Todo PR pasa por `/security-review` antes del merge.
- Auth, pagos o datos personales: además, `/security-audit` en modo completo.
- Antes de liberar esa superficie a staging/prod: Strix con Docker en marcha.

---

## 11. `AGENTS.md`: las rules del proyecto

[`AGENTS.md`](https://agents.md) es un formato abierto: un README para agentes en la raíz del repositorio. Lo leen Cursor, Codex, Copilot, Claude Code, Windsurf y otros, así que es **la fuente de verdad** de cómo trabajar en el proyecto, independiente del IDE.

Cada línea se paga en tokens en casi todos los chats: mantenlo en **una pantalla**. Incluye stack y comandos canónicos, convenciones, carpetas prohibidas, estilo de commits/PR, prohibiciones y punteros a skills. Los tutoriales y checklists largos van a skills.

Sample oficial de [agentsmd/agents.md](https://github.com/agentsmd/agents.md):

```markdown
# Sample AGENTS.md file

## Dev environment tips
- Use `pnpm dlx turbo run where <project_name>` to jump to a package instead of scanning with `ls`.
- Run `pnpm install --filter <project_name>` to add the package to your workspace so Vite, ESLint, and TypeScript can see it.
- Use `pnpm create vite@latest <project_name> -- --template react-ts` to spin up a new React + Vite package with TypeScript checks ready.
- Check the name field inside each package's package.json to confirm the right name—skip the top-level one.

## Testing instructions
- Find the CI plan in the .github/workflows folder.
- Run `pnpm turbo run test --filter <project_name>` to run every check defined for that package.
- From the package root you can just call `pnpm test`. The commit should pass all tests before you merge.
- To focus on one step, add the Vitest pattern: `pnpm vitest run -t "<test name>"`.
- Fix any test or type errors until the whole suite is green.
- After moving files or changing imports, run `pnpm lint --filter <project_name>` to be sure ESLint and TypeScript rules still pass.
- Add or update tests for the code you change, even if nobody asked.

## PR instructions
- Title format: [<project_name>] <Title>
- Always run `pnpm lint` and `pnpm test` before committing.
```

Bloques que conviene añadir para las herramientas de este tutorial:

```markdown
## Mapa del codebase (Graphify)
- Si existe `graphify-out/graph.json`, antes de leer carpetas amplias usa `graphify query "<pregunta>"` (o `path` / `explain`) y abre solo los archivos citados.
- Tras cambios de código, ejecuta `graphify update .`.

## Especificación
- La fuente de verdad está en `docs/specs/` (PRD, TRD, appflow, UI/UX, esquema de backend, plan).
- No inventes tablas, endpoints ni pantallas que no estén en la especificación; si falta algo, pregunta.
- Tareas de 3+ archivos: primero plan aprobado, después código.

## Prohibiciones
- Nunca secretos ni PII en prompts. Nunca `git push --force` ni despliegues sin aprobación.
```

**Cualquier IDE:** basta con el archivo en la raíz (y opcionalmente uno por subcarpeta en monorepos; gana el más cercano al archivo editado).

**Cursor:** lee `AGENTS.md` automáticamente. Las rules en `.cursor/rules/*.mdc` (las que crean Graphify o Context Mode) son un complemento con activación por archivo o `alwaysApply`; no dupliques en ellas lo que ya está en `AGENTS.md`.

---

## 12. Desarrollo guiado por especificación (SDD)

Lo que más impacto tiene en la calidad: **la especificación es la fuente de verdad, no el código**. En lugar de pedir código con una instrucción abierta, das al agente un paquete de documentos versionados y él implementa por tareas. El chat es efímero; la especificación es memoria que se revisa en PR y se reutiliza como contexto preciso.

### Los documentos

Guárdalos en `docs/specs/`:

| Documento | Contenido |
|---|---|
| **PRD** (*Product Requirements Document*) | Requisitos de producto: problema, usuarios, alcance, no-objetivos y criterios de aceptación. Responde al *qué* y al *por qué*. |
| **TRD** (*Technical Requirements Document*) | Requisitos técnicos: restricciones de stack, no funcionales (latencia, seguridad, privacidad), integraciones y límites operativos. |
| **Appflow** | Flujo de la aplicación: pantallas o pasos, transiciones, estados y caminos felices/error. Une producto y experiencia. |
| **Diseño UI/UX** | Interfaz y experiencia: wireframes, componentes y reglas de interacción. |
| **Esquema de backend** | Modelo de datos, contratos de API, eventos y límites de consistencia. Evita inventar tablas o endpoints al implementar. |
| **Plan de implementación** | Orden de trabajo, archivos a tocar, riesgos, dependencias y cortes por tarea. Se aprueba antes de codificar (ver *Plan vs Implement*, abajo). |

### El ciclo

1. **Constitución:** reglas no negociables del proyecto, en `AGENTS.md` (paso 11).
2. **Especificar (qué):** PRD, más Appflow y UI/UX cuando hay superficie de usuario. Sin código.
3. **Planificar (cómo):** TRD, esquema de backend y plan de implementación.
4. **Tareas:** el plan troceado en unidades pequeñas y verificables; cada una cabe en un chat.
5. **Implementar y verificar:** una tarea por sesión, con tests; cada diff se revisa contra el paquete, no contra el gusto del revisor.

**Plan vs Implement.** Usa el modo Plan para que el agente investigue en solo lectura y proponga pasos, archivos, tests y riesgos; revisa el plan y solo después cambia a implementación, paso a paso. Corregir un plan cuesta minutos; corregir código, horas. Exígelo en tareas de 3+ archivos, refactors y migraciones.

- **Cualquier IDE:** modo Plan de Claude Code (`Shift+Tab`), Codex o el equivalente; si no existe, pide "no escribas código, dame pasos, archivos, tests y riesgos".
- **Cursor:** selector de modo del chat (**Plan** / **Agent**). Entre tarea y tarea, chat nuevo o `/handoff` para no arrastrar ruido.

### Flujos y alternativas

**Flujo de Matt Pocock (punto de partida recomendado).** [mattpocock/skills](https://github.com/mattpocock/skills) ya es SDD con skills pequeñas:

```bash
npx skills add mattpocock/skills
```

Después ejecuta `/setup-matt-pocock-skills` una vez por repositorio (gestor de trabajo, etiquetas, rutas de specs y glosario).

| Fase | Skill | Resultado |
|---|---|---|
| Alinear | `/grill-me` o `/grill-with-docs` | Alcance cerrado, lenguaje ubicuo (`GLOSSARY.md`) y ADR |
| Especificar | `/to-spec` | Especificación publicada en el gestor de trabajo |
| Trocear | `/to-tickets` (o `/wayfinder` si hay incertidumbre) | Tareas con dependencias, cada una cabe en una sesión |
| Implementar | `/implement` + `/tdd` | Código y tests por tarea, con `/code-review` antes del commit |
| Entregar | `/pr` + `/retro` | PR con evidencia y mejoras al entorno del agente |

**Alternativas:**

| Herramienta | Qué es | Cuándo encaja | Instalación |
|---|---|---|---|
| [GitHub Spec Kit](https://github.com/github/spec-kit) | Proceso guiado: `/speckit.specify`, `/speckit.plan`, `/speckit.tasks`, `/speckit.implement` | Proyecto nuevo, equipo pequeño, camino marcado | `uvx --from git+https://github.com/github/spec-kit.git specify init mi-proyecto` (elige el agente al iniciar) |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | Ligero; cada cambio es una carpeta (`proposal.md`, `specs/`, `design.md`, `tasks.md`) con specs delta | Repositorio con historia (*brownfield*) | `npm install -g @fission-ai/openspec@latest` y `openspec init` |
| [superpowers](https://github.com/obra/superpowers) | Skills + metodología: brainstorming, worktrees, plan en tareas de 2–5 min, TDD obligatorio, revisión por severidad | Disciplina de ejecución (tests, revisiones, ramas) | Claude Code: marketplace de plugins; otros: `npx skills add obra/superpowers` |
| [agent-skills](https://github.com/addyosmani/agent-skills) | Biblioteca de skills de ingeniería de producción con criterio senior | Equipos con proceso propio que quieren subir el listón | `npx skills add addyosmani/agent-skills` |
| [BMAD-Method](https://github.com/bmad-code-org/BMAD-METHOD) | Metodología por fases con agentes por rol (producto, arquitectura, UX, dev, QA) | Muchas partes interesadas o entorno regulado | `npx bmad-method install` |
| Amazon Kiro | IDE con specs en notación EARS y hooks | Equipos en AWS que aceptan atarse al IDE | Descarga del IDE |

Criterio: empezar ya sin ceremonia, flujo Matt; brownfield, OpenSpec; proyecto nuevo, Spec Kit; disciplina con TDD y worktrees, superpowers; criterio senior sin cambiar proceso, agent-skills; trazabilidad y roles, BMAD. En todos los casos la especificación se versiona como código y se revisa en PR.

---

## 13. Checklist final

- [ ] Repositorio creado con `AGENTS.md`, `docs/specs/` y `.gitignore`.
- [ ] Modelo elegido tras consultar Artificial Analysis, LLM Stats y Arena.
- [ ] (Opcional) OmniRoute corriendo y configurado como base URL.
- [ ] Agent Reach instalado; `agent-reach doctor` en verde.
- [ ] Plugins necesarios activos en **Settings → Plugins / MCP**.
- [ ] Skills instaladas con `npx skills add Japegomez/cursor_skills`.
- [ ] Caveman, RTK (`rtk gain`) y Context Mode (`ctx doctor`) funcionando.
- [ ] Grafo generado con `/graphify .` y regla de Graphify activa.
- [ ] CodeRabbit instalado en el repositorio y `.coderabbit.yaml` creado.
- [ ] gitleaks en pre-commit + TruffleHog en CI; `/security-review` en cada PR (y `/security-audit` si toca auth/pagos/datos).
- [ ] (Opcional) Docker en marcha y Strix probado contra staging antes de liberar auth/pagos/datos.
- [ ] `AGENTS.md` de una pantalla con comandos, convenciones y prohibiciones.
- [ ] PRD, TRD, Appflow, UI/UX, esquema de backend y plan de implementación en `docs/specs/` antes de escribir código.
