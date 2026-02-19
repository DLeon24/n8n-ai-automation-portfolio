# Análisis de Sentimiento — Agente de Feedbacks de Clientes

Workflow de n8n que analiza comentarios de clientes mediante agentes de IA encadenados. Clasifica el sentimiento, identifica el equipo de trabajo responsable y, en caso de comentarios negativos, genera un plan de acción correctivo.

## Arquitectura del Flujo

```
Webhook (POST)
    │
    ▼
Edit Fields (contexto de la empresa)
    │
    ▼
Feeling Analysis (LLM) ──► positivo / negativo / neutral
    │
    ▼
Team Classification (LLM) ──► Venta | Servicio al cliente | Calidad | Garantía | Logística
    │
    ▼
 ┌──If (¿negativo?)──┐
 │ SÍ                 │ NO
 ▼                    ▼
Action Plan (LLM)   Respuesta directa
 │                   { feeling, workTeam }
 ▼
Respuesta completa
{ feeling, workTeam, actionPlan }
```

## Nodos principales

| Nodo | Tipo | Descripción |
|------|------|-------------|
| **Webhook - Feeling Analysis** | Webhook (POST) | Punto de entrada. Recibe el comentario del cliente en `body.message` |
| **Edit Fields** | Set | Inyecta el contexto de la empresa y la definición de los equipos de trabajo |
| **Feeling Analysis** | LLM Chain | Clasifica el sentimiento del comentario: `positivo`, `negativo` o `neutral` |
| **Team Classification** | LLM Chain | Asigna el comentario al equipo de trabajo correspondiente |
| **If** | Condicional | Evalúa si el sentimiento es `negativo` para activar el plan de acción |
| **Action Plan** | LLM Chain | Genera un plan correctivo con descripción del problema, tareas y KPIs |
| **Respond - Webhook** | Respond to Webhook | Devuelve la respuesta JSON al cliente de la API |

## Stack Tecnológico

- **n8n** — Orquestación del flujo de trabajo
- **Ollama** — Servidor local de modelos de lenguaje
- **gemma2:2b** — Modelo LLM utilizado en los tres agentes
- **LangChain (n8n nodes)** — Cadenas LLM con output parsers estructurados

## Endpoint

```
POST /webhook/feeling/analysis
```

### Request

```json
{
  "message": "El producto llegó dañado y nadie me ha dado respuesta en 3 días."
}
```

### Response — Sentimiento negativo

```json
{
  "feeling": "negativo",
  "workTeam": "Servicio al cliente",
  "actionPlan": "Plan de acción detallado con tareas y KPIs..."
}
```

### Response — Sentimiento positivo o neutral

```json
{
  "feeling": "positivo",
  "workTeam": "Logistica"
}
```

## Contexto de negocio

El workflow está configurado para una **empresa de venta de productos tecnológicos por internet con cobertura en Perú**. Los equipos de trabajo reconocidos son:

| Equipo | Alcance |
|--------|---------|
| **Venta** | Proceso comercial antes y durante la compra |
| **Servicio al cliente** | Atención postventa |
| **Calidad de producto** | Auditoría y estandarización del producto |
| **Garantía** | Cambios o devoluciones por defectos de fábrica |
| **Logística** | Gestión de envíos a clientes |

## Plan de acción (solo sentimientos negativos)

Cuando el sentimiento detectado es **negativo**, el agente `Action Plan` genera un informe que contiene:

- Descripción del problema
- Tareas a realizar
- KPIs de seguimiento operativos y tácticos

## Requisitos previos

1. **n8n** instalado y ejecutándose
2. **Ollama** corriendo localmente con el modelo `gemma2:2b` descargado:
   ```bash
   ollama pull gemma2:2b
   ```
3. Credenciales de Ollama configuradas en n8n (`ollama_credentials`)

## Importar el workflow

1. Abre n8n y ve a **Settings → Import Workflow**
2. Selecciona el archivo `feelings-analysis.json`
3. Configura las credenciales de Ollama en los nodos del modelo
4. Activa el workflow
