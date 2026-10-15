# cursor_skills

Skills de agente para Cursor. En otro equipo, instala las skills desde este repositorio y el resto con los pasos de abajo.

## Skills

```bash
npx skills add Japegomez/cursor_skills
```

## Plugins

Los plugins se instalan con la skill `/add-plugin`. En el chat de Cursor, ejecútala, pega este repositorio y elige estos nombres:

https://github.com/cursor/plugins

- continual-learning
- cursor-team-kit
- docs-canvas
- gamma
- github
- google-calendar
- google-docs
- google-sheets
- google-slides
- pr-review-canvas
- teaching
- thermos

Estos otros se instalan por nombre:

```text
/add-plugin asana
/add-plugin context7-plugin
/add-plugin devtools-for-agents
/add-plugin gitlab
/add-plugin higgsfield
/add-plugin posthog
/add-plugin sentry
/add-plugin supabase
```

## RTK en Cursor

Elige un método de instalación y después inicia el hook de Cursor.

```bash
brew install rtk                                              # macOS / Linux
winget install rtk-ai.rtk                                     # Windows
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
rtk init -g --agent cursor      # Cursor
```
