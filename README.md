# CHATBOT-ATHON

Una aplicación web de chatbot minimalista y limpia que utiliza una API de LLM. Construida con Next.js, TypeScript y Tailwind CSS.

## Funcionalidades

- Interfaz de chat con desplazamiento automático e indicador de escritura
- Ruta de API en el backend que llama a cualquier modelo compatible con OpenAI
- Prompt de sistema fijo (configurable mediante variable de entorno)
- Memoria de conversación a corto plazo por sesión de navegador (en memoria, últimos 20 mensajes)
- Ensamblado eficiente de prompts — el prompt de sistema se envía una vez y el historial se recorta
- Registro de uso por solicitud (tokens de prompt/completado/total)
- Manejo básico de errores con mensajes visibles para el usuario

## Estructura del proyecto

```
app/
  api/chat/route.ts   — Manejador POST: valida entrada, llama al LLM, registra uso
  layout.tsx          — Layout raíz
  page.tsx            — Renderiza ChatInterface
components/
  ChatInterface.tsx   — UI del chat (mensajes, entrada, carga, errores)
lib/
  session-store.ts    — Map en memoria del historial de mensajes por ID de sesión
  llm-client.ts       — Cliente OpenAI + configuración
  prompt-builder.ts   — Ensambla [sistema, ...historial, usuario] por solicitud
types/
  chat.ts             — Tipos TypeScript compartidos
```

## Inicio rápido

```bash
# 1. Copia y completa las variables de entorno
cp .env.local.example .env.local
# edita .env.local — establece OPENAI_API_KEY como mínimo

# 2. Instala dependencias
npm install

# 3. Inicia el servidor de desarrollo
npm run dev
```

Abre [http://localhost:3000](http://localhost:3000).

## Variables de entorno

| Variable               | Requerida | Por defecto          | Descripción                                           |
|------------------------|-----------|----------------------|-------------------------------------------------------|
| `OPENAI_API_KEY`       | Sí        | —                    | Clave de API para OpenAI o proveedor compatible       |
| `OPENAI_BASE_URL`      | No        | Por defecto OpenAI   | Alternativa para APIs compatibles (e.g. Ollama)       |
| `OPENAI_MODEL`         | No        | `gpt-4o-mini`        | Identificador del modelo                              |
| `MAX_COMPLETION_TOKENS`| No        | `1024`               | Máximo de tokens en cada respuesta del asistente      |
| `SYSTEM_PROMPT`        | No        | Ver `prompt-builder` | Reemplaza el prompt de sistema fijo                   |

## Diseño de eficiencia de tokens

- El prompt de sistema es **fijo** — misma cadena en cada llamada, lo que permite caché de prompts si el proveedor lo soporta.
- El historial se **recorta** a los últimos 20 mensajes (10 turnos) antes de ensamblar cada solicitud.
- El contexto dinámico (historial + nuevo mensaje) se coloca **después** del prompt de sistema estático, para que los prefijos cacheados sigan siendo válidos.
- `max_tokens` limita la longitud de la respuesta.

## Extensión

- **Cambiar el proveedor del modelo**: establece `OPENAI_BASE_URL` para apuntar a cualquier endpoint compatible con OpenAI (Ollama, Together, Groq, etc.).
- **Cambiar la personalidad**: edita `SYSTEM_PROMPT` en `.env.local` o directamente en `lib/prompt-builder.ts`.
- **Persistir el historial**: reemplaza el `Map` en `lib/session-store.ts` con Redis o una base de datos.
- **Respuestas en streaming**: cambia `llm.chat.completions.create` para usar `stream: true` y devuelve un `ReadableStream` desde el manejador de ruta.
