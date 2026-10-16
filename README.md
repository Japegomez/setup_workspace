# cursor_skills

Skills de agente para Cursor. En otro equipo, instala las skills desde este repositorio y el resto con los pasos de abajo.

## Skills

```bash
npx skills add Japegomez/cursor_skills
```

Incluye `archify` (diagramas de arquitectura, flujos, secuencias y ciclos de vida) y `find-skills` (buscar skills instalables).

## Plugins

En el chat de Cursor, ejecuta este comando:

```text
/add-plugin asana context-mode https://github.com/mksglu/context-mode context7-plugin continual-learning cursor-team-kit devtools-for-agents docs-canvas gamma github gitlab google-calendar google-docs google-sheets google-slides higgsfield posthog pr-review-canvas sentry supabase teaching thermos
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

En esta máquina Python está protegido por PEP 668, así que se usa un entorno virtual:

```bash
python3 -m venv ~/.agent-reach-venv
source ~/.agent-reach-venv/bin/activate
pip install https://github.com/Panniantong/agent-reach/archive/main.zip
agent-reach install --env=auto
```

Si `pipx` está disponible:

```bash
pipx install https://github.com/Panniantong/agent-reach/archive/main.zip
agent-reach install --env=auto
```

Después de aprobar cambios de sistema:

```bash
agent-reach install --env=auto --system
agent-reach doctor
```
