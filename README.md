# cursor_skills

Skills de agente para Cursor. En otro equipo, instala las skills desde este repositorio y el resto con los pasos de abajo.

## Skills

```bash
npx skills add Japegomez/cursor_skills
```

Desde la raíz del proyecto, [Impeccable](https://github.com/pbakaus/impeccable) instala su lenguaje de diseño para que el agente diseñe interfaces con más criterio:

```bash
npx impeccable install
```


## Plugins

En el chat de Cursor, ejecuta este comando:

```text
/add-plugin asana context-mode context7-plugin continual-learning cursor-team-kit devtools-for-agents docs-canvas gamma github gitlab google-calendar google-docs google-sheets google-slides higgsfield posthog pr-review-canvas sentry supabase teaching thermos
```

- **asana** — tareas, proyectos y comentarios de Asana.
- **context-mode** — reduce el contexto que llega al agente: filtra la salida de herramientas, guarda memoria de la sesión y enruta llamadas.
- **context7-plugin** — documentación actual de librerías, sacada del repositorio y no del entrenamiento del modelo.
- **continual-learning** — aprende preferencias y hechos del workspace a partir de las conversaciones.
- **cursor-team-kit** — flujos de CI, revisión y entrega.
- **devtools-for-agents** — Chrome DevTools para depurar páginas, red y rendimiento.
- **docs-canvas** — abre la documentación como un canvas navegable.
- **gamma** — genera presentaciones, documentos y páginas.
- **github** — repositorios, issues, pull requests y Actions.
- **gitlab** — el equivalente para GitLab.
- **google-calendar** — busca eventos y crea reuniones.
- **google-docs** — lee y edita documentos.
- **google-sheets** — lee y escribe hojas de cálculo.
- **google-slides** — crea y edita presentaciones.
- **higgsfield** — genera imágenes y vídeo.
- **posthog** — analítica, feature flags, experimentos y errores.
- **pr-review-canvas** — muestra el diff de un PR agrupado por importancia.
- **sentry** — issues y stack traces de Sentry.
- **supabase** — proyectos, tablas y consultas de Supabase.
- **teaching** — planes de práctica y retrospectivas de aprendizaje.
- **thermos** — revisión estricta de una rama: bugs, seguridad y calidad.

## RTK en Cursor

RTK es un proxy de la terminal. Cuando el agente lanza un comando (`git`, `ls`, tests, logs), el hook lo reescribe a `rtk …` y devuelve un resumen en lugar de la salida completa. El agente conserva la señal y gasta menos contexto.

Elige un método de instalación y después inicia el hook de Cursor.

```bash
brew install rtk                                              # macOS / Linux
winget install rtk-ai.rtk                                     # Windows
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
rtk init -g --agent cursor      # Cursor
```

## Agent Reach

Da al agente acceso a web, GitHub, YouTube, RSS y otros canales. La comprobación por defecto no cambia el sistema. `--system` instala dependencias que falten y hay que pedirlo aparte. Los canales opcionales (Twitter, Reddit, Xiaohongshu, etc.) tampoco se instalan solos.

Pegar en el agente de IA:

```bash
en帮我安装 Agent Reach：https://raw.githubusercontent.com/Panniantong/agent-reach/main/docs/install.md
```

## Graphify

[Graphify](https://github.com/Graphify-Labs/graphify) convierte un proyecto (código, docs, PDFs, imágenes) en un **grafo de conocimiento** local: relaciones entre archivos/conceptos, informe y `graph.html`. El agente consulta el grafo antes de explorar el repo a ciegas.

```bash
uv tool install graphifyy          # paquete PyPI oficial (doble y); el CLI es `graphify`
graphify install --platform cursor # skill + regla .cursor/rules/graphify.mdc
```

En el chat de Cursor, en la raíz del proyecto:

```text
/graphify .
```

Docs: [Graphify + Cursor](https://docs.graphify.com/integrations/cursor).

## Caveman

[Caveman](https://github.com/JuliusBrussee/caveman) reduce tokens: respuestas más cortas, salidas de herramientas comprimidas y (opcional) selección de modelo. En Cursor instala skills, agentes Cavecrew y un hook de sesión.

```bash
# Opción completa (recomendado; Node.js 22.13+)
npx -y github:JuliusBrussee/caveman -- --only cursor

# Solo la skill de voz corta
npx skills add JuliusBrussee/caveman -g
```

Windows (PowerShell), instalador unificado:

```powershell
irm https://raw.githubusercontent.com/JuliusBrussee/caveman/v4.0.1/install.ps1 | iex
```

En el chat: `/caveman`. Para desactivar: `stop caveman`.

## Context Mode

[Context Mode](https://github.com/mksglu/context-mode) evita que la salida enorme de herramientas llene el contexto: sandbox (`ctx_execute`, etc.), memoria de sesión y hooks de enrutado. Ahorra tokens manteniendo la señal.

Hasta que esté en el Marketplace de Cursor, plugin local + CLI global:

```powershell
# Windows
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

Reinicia Cursor. Comprueba en **Settings → MCP** que `context-mode` esté conectado y en el chat: `ctx doctor` o `ctx stats`.

También puedes añadirlo con `/add-plugin context-mode` cuando el Marketplace lo liste (ya aparece en la lista de plugins de arriba).
