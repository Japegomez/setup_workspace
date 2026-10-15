# cursor_skills

Skills de agente para Cursor. En otro equipo, instala las skills desde este repositorio y el resto con los pasos de abajo.

## Skills

```bash
npx skills add Japegomez/cursor_skills
```

## Plugins

En el chat de Cursor, ejecuta este comando:

```text
/add-plugin asana context7-plugin continual-learning cursor-team-kit devtools-for-agents docs-canvas gamma github gitlab google-calendar google-docs google-sheets google-slides higgsfield posthog pr-review-canvas sentry supabase teaching thermos
```

## RTK en Cursor

Elige un método de instalación y después inicia el hook de Cursor.

```bash
brew install rtk                                              # macOS / Linux
winget install rtk-ai.rtk                                     # Windows
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
rtk init -g --agent cursor      # Cursor
```
