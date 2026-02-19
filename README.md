# n8n AI Automation Portfolio

Colección de workflows de n8n enfocados en automatización con inteligencia artificial. Cada proyecto incluye el archivo exportado listo para importar y documentación detallada.

## Proyectos

| Proyecto | Descripción | Stack |
|----------|-------------|-------|
| [Feelings Analysis](./feelings-analysis/) | Agente que analiza sentimiento de comentarios de clientes, clasifica al equipo responsable y genera planes de acción para feedback negativo | n8n, Ollama, gemma2:2b, LangChain |

## Stack General

- **[n8n](https://n8n.io/)** — Plataforma de automatización de workflows
- **[Ollama](https://ollama.com/)** — Servidor local de modelos de lenguaje
- **[LangChain](https://js.langchain.com/)** — Framework para cadenas LLM integrado en n8n

## Cómo usar

1. Clona el repositorio:
   ```bash
   git clone https://github.com/DLeon24/n8n-ai-automation-portfolio.git
   ```
2. Abre n8n y ve a **Settings → Import Workflow**
3. Selecciona el archivo `.json` del proyecto que quieras importar
4. Configura las credenciales necesarias según el README de cada proyecto
5. Activa el workflow
