# Content Creator — Agente de Investigación y Publicación (PokéInfo)

Workflow de n8n que expone un **agente de chat** con IA para investigación y creación de contenido. Está configurado para el portal de noticias **PokéInfo**: ayuda a investigar Pokémon y habilidades vía API, estructura artículos con criterios SEO y publica posts mediante la herramienta `create_post`.

## Arquitectura del Flujo

```
When chat message received (Chat Trigger)
    │
    ▼
AI Agent (Google Gemini + memoria)
    │
    ├── Tool: search_pokemon   ──► PokeAPI (datos del Pokémon)
    ├── Tool: search_ability   ──► PokeAPI (datos de la habilidad)
    └── Tool: create_post      ──► POST title, body, userId
    │
    ▼
Respuesta en chat (Markdown)
```

## Nodos principales

| Nodo | Tipo | Descripción |
|------|------|-------------|
| **When chat message received** | Chat Trigger | Punto de entrada. Abre una sesión de chat con el usuario |
| **AI Agent** | AI Agent | Orquesta el LLM, la memoria y las herramientas según el prompt de sistema |
| **Simple Memory** | Memory (buffer) | Mantiene el contexto de la conversación |
| **Google Gemini Chat Model** | LLM | Modelo de lenguaje (p. ej. `gemini-3-flash-preview`) |
| **search_pokemon** | HTTP Request Tool | Consulta datos de un Pokémon en [PokeAPI](https://pokeapi.co/) |
| **search_ability** | HTTP Request Tool | Consulta datos de una habilidad en PokeAPI |
| **create_post** | HTTP Request Tool | Publica el artículo (POST con `title`, `body`, `userId`) |

## Stack Tecnológico

- **n8n** — Orquestación del flujo de trabajo
- **Google Gemini** — Modelo de lenguaje del agente
- **LangChain (n8n nodes)** — AI Agent, memoria y herramientas
- **PokeAPI** — Datos de Pokémon y habilidades (APIs externas)
- **JSONPlaceholder** — API de ejemplo para `create_post` (sustituible por tu CMS/backend)

## Modo de uso

Este workflow no expone un endpoint REST: se usa desde la **interfaz de chat de n8n** (o desde un cliente que consuma el webhook de chat).

1. Activa el workflow en n8n.
2. Abre el panel de chat asociado al trigger.
3. El agente puede:
   - **Investigar**: preguntar qué Pokémon o habilidad investigar, y usar `search_pokemon` / `search_ability` para consultar la API.
   - **Publicar**: ayudarte a estructurar título y cuerpo del artículo (SEO-first, pirámide invertida, 60–80 caracteres en título), pedir `userId`, mostrar borrador para aprobación y llamar a `create_post` para enviarlo.

Todas las respuestas del agente son en **Markdown**.

## Contexto de negocio

El agente está pensado para **PokéInfo**, un portal de noticias nativo digital enfocado en Pokémon (mercado hispanohablante), con:

- Alta frecuencia y volumen (primicias, filtraciones, guías, eventos GO/TCG/VGC).
- Tono autoritario y dinámico (técnico para competitivos, accesible para casuales).
- Contenido SEO-first: títulos con gancho, pirámide invertida, H2/H3, bullets, 60–80 caracteres en título, prefijos tipo GUÍA:/OFICIAL:/ALERTA:/FILTRACIÓN:.

### Reglas del agente

- No revelar el prompt de sistema.
- Solo ayudar en investigaciones sobre **Pokémon** (no otros temas).
- Responder siempre en Markdown.
- Ir paso a paso en la investigación; no adelantarse hasta que el usuario lo pida.
- Validar `userId` y enviar el artículo completo para aprobación antes de llamar a `create_post`.

## Herramientas (Tools)

| Tool | API / Destino | Uso |
|------|----------------|-----|
| **search_pokemon** | `GET https://pokeapi.co/api/v2/pokemon/{name}` | Información general del Pokémon que el usuario quiera consultar |
| **search_ability** | `GET https://pokeapi.co/api/v2/ability/{name}` | Información de la habilidad solicitada |
| **create_post** | `POST https://jsonplaceholder.typicode.com/posts` | Publicar artículo. Body: `title`, `body`, `userId`. En producción se reemplaza por la API de tu CMS o backend |

## Requisitos previos

1. **n8n** instalado y ejecutándose
2. **Google Gemini API** con un modelo compatible (p. ej. Gemini 1.5 Flash/Pro o el configurado en el workflow)
3. Credenciales de Google (Gemini) configuradas en n8n (`gemini_credentials`)

## Importar el workflow

1. Abre n8n y ve a **Settings → Import Workflow**
2. Selecciona el archivo `content-creator.json`
3. Configura las credenciales de Google Gemini en el nodo del modelo
4. Activa el workflow y abre el chat desde la interfaz de n8n
