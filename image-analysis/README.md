# Image Analysis — Análisis de Pedidos con Visión por IA

Workflow de n8n que analiza imágenes de pedidos de restaurantes mediante un modelo de visión (Google Gemini). Contrasta la foto del cliente con el reclamo y la orden original, clasifica el estado del pedido y calcula el valor de devolución según la política de Rappi.

## Arquitectura del Flujo

```
Webhook (POST)
    │  body: image (URL), claim, order
    ▼
HTTP Request (descarga imagen desde URL)
    │
    ▼
Image Analysis (LLM con visión) ──► returnValue + orderStatus
    │
    ▼
Respond to Webhook
    { returnValue, orderStatus }
```

## Nodos principales

| Nodo | Tipo | Descripción |
|------|------|-------------|
| **Webhook - Image Analysis** | Webhook (POST) | Punto de entrada. Recibe `body.image` (URL), `body.claim` (reclamo) y `body.order` (pedido original) |
| **HTTP Request** | HTTP Request | Descarga la imagen desde la URL indicada en `body.image` |
| **Image Analysis** | LLM Chain (visión) | Analiza imagen + reclamo + pedido; clasifica estado y calcula valor de devolución |
| **SOP - Image Analysis** | Output Parser | Estructura la salida en `returnValue` (decimal) y `orderStatus` (texto) |
| **Respond to Webhook** | Respond to Webhook | Devuelve la respuesta JSON al cliente de la API |

## Stack Tecnológico

- **n8n** — Orquestación del flujo de trabajo
- **Google Gemini** — Modelo de lenguaje con visión para análisis de imágenes
- **LangChain (n8n nodes)** — Cadena LLM con output parser estructurado

## Endpoint

```
POST /webhook/image/analysis
```

### Request

```json
{
    "claim": "no me llego la gaseosa",
    "order": {
        "products": [
            {
                "name": "Hamburguesa",
                "price": 20,
                "currency": "USD",
                "quantity": 1
            },
            {
                "name": "Papas a la francesa",
                "price": 5,
                "currency": "USD",
                "quantity": 1
            },
            {
                "name": "Gaseosa",
                "price": 3,
                "currency": "USD",
                "quantity": 1
            }
        ]
    },
    "image": "https://media.istockphoto.com/id/153835617/es/foto/hamburguesa-con-queso-y-papas-fritas.jpg?s=612x612&w=0&k=20&c=7Bzi7wjS4WKkOZL7BUNXXD0J8sPCRCnIhqDi-VqX2xI="
}
```

### Response

```json
{
    "returnValue": 3,
    "orderStatus": "pedido incompleto"
}
```

## Contexto de negocio

El workflow está pensado para el **marketplace de Rappi (vertical restaurantes)**. El agente conoce:

- El flujo de última milla (restaurante → repartidor → cliente)
- La política de devoluciones de Rappi

### Clasificación del pedido (`orderStatus`)

| Estado | Descripción |
|--------|-------------|
| **pedido errado** | Llegó algo distinto a lo solicitado |
| **pedido incompleto** | Faltan productos del pedido |
| **pedido completo** | Llegó todo correctamente (sin lugar a devolución) |
| **pedido deteriorado** | Productos llegaron en mal estado |

### Política de devolución (valor a devolver)

| Situación | Criterio |
|-----------|----------|
| Pedido incompleto | Valor de lo que **no llegó** |
| Pedido errado | **100%** del valor del pedido |
| Pedido deteriorado | **40%** del valor de los productos deteriorados |
| Pedido no llegó | **100%** del valor del pedido |

El agente usa los precios del insumo `order` para calcular `returnValue` en dólares (2 decimales).

## Requisitos previos

1. **n8n** instalado y ejecutándose
2. **Google Gemini API** con acceso a un modelo con visión (p. ej. Gemini 1.5 Flash/Pro)
3. Credenciales de Google (Gemini) configuradas en n8n (`gemini_credentials`)

## Importar el workflow

1. Abre n8n y ve a **Settings → Import Workflow**
2. Selecciona el archivo `image-analysis.json`
3. Configura las credenciales de Google Gemini en el nodo del modelo
4. Activa el workflow
