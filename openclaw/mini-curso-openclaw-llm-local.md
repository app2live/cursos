# Mini-curso: OpenClaw + LLMs Local/Cloud

**Perfil objetivo:** ya usas Claude, Claude Code y agentes en producción, tienes Ollama instalado en Windows. Este curso no repite fundamentos de IA generativa — va directo a los dos huecos: orquestación con OpenClaw y modelos ejecutándose en tu propia máquina.

**Formato:** 10 apartados (5 OpenClaw + 5 LLM local/cloud), 3-5 subapartados cada uno, enlaces gratuitos, 10 ejercicios (5 básicos + 5 avanzados).

---

## Tabla de contenidos

**Bloque A — OpenClaw**
1. [Fundamentos de OpenClaw](#1-fundamentos-de-openclaw)
2. [Empezar en modo hosted](#2-empezar-en-modo-hosted)
3. [Migrar a self-hosted](#3-migrar-a-self-hosted)
4. [Skills y automatización](#4-skills-y-automatización)
5. [Seguridad y límites](#5-seguridad-y-límites)

**Bloque B — LLMs Local/Cloud**
6. [Fundamentos de LLMs locales](#6-fundamentos-de-llms-locales)
7. [Ollama en Windows](#7-ollama-en-windows)
8. [Interfaces sobre Ollama](#8-interfaces-sobre-ollama)
9. [RAG local](#9-rag-local)
10. [Arquitectura híbrida local + cloud](#10-arquitectura-híbrida-local--cloud)

**[Ejercicios básicos](#ejercicios-básicos)** · **[Ejercicios avanzados](#ejercicios-avanzados)** · **[Crítica del curso](#crítica-constructiva-del-curso)**

---

## 1. Fundamentos de OpenClaw

### 1.1 Qué es y por qué existe
OpenClaw es un gateway self-hosted que conecta apps de mensajería (WhatsApp, Telegram, Discord, Slack, iMessage...) con un agente de IA siempre disponible. Nació como Clawdbot, pasó por Moltbot y hoy es un proyecto de la OpenClaw Foundation.
🔗 [docs.openclaw.ai](https://docs.openclaw.ai/) — overview oficial

### 1.2 Arquitectura: Gateway, agente, skills
Un único proceso Gateway enruta mensajes entrantes, gestiona sesiones, ejecuta herramientas y devuelve respuestas. El agente corre sobre un core ("Pi") con acceso a browser, exec, web search y skills.
🔗 [OpenClaw Skill Reference](https://docs.openclaw.ai/skill.md) — ficheros clave del workspace (AGENTS.md, SOUL.md, TOOLS.md, MEMORY.md)

### 1.3 OpenClaw vs n8n vs MCP
n8n orquesta flujos deterministas (webhooks, ramas condicionales). OpenClaw añade la capa de razonamiento con LLM sobre esos flujos — no lo sustituye. MCP estandariza cómo un agente llama herramientas; OpenClaw estandariza el agente completo (canal + memoria + skills).
🔗 [Comparativa OpenClaw + n8n en un caso real](https://generect.com/blog/openclaw-ai-agent/)

### 1.4 Casos de uso reales
Asistente personal por WhatsApp, triage de leads, notas de llamadas con escritura automática a CRM, research diario programado.
🔗 [Personal assistant setup](https://docs.openclaw.ai/start/openclaw)

---

## 2. Empezar en modo hosted

### 2.1 Opciones hosted sin infraestructura
Para probar sin montar servidor: Gclaw (Gcore, con GPU incluida) o despliegue de un click en Koyeb.
🔗 [Gclaw docs](https://gcore.com/docs/gclaw.md) · 🔗 [Deploy en Koyeb](https://koyeb.com/deploy/openclaw)

### 2.2 Deploy en Koyeb paso a paso
Desplegar el servicio, abrir `https://tu-servicio.koyeb.com/overview`, introducir el `OPENCLAW_GATEWAY_TOKEN`, aprobar el dispositivo desde la consola.
🔗 [Guía completa Koyeb](https://koyeb.com/deploy/openclaw)

### 2.3 Primer chat sin configurar canales
La vía más rápida para el primer contacto es el Control UI, sin enlazar ningún canal de mensajería todavía.
🔗 [Dashboard docs](https://docs.openclaw.ai/web/dashboard)

### 2.4 Límites del modo hosted
Dependes de la infraestructura de un tercero, memoria y datos no quedan 100% bajo tu control, coste recurrente desde ~22€/mes según proveedor. Sirve para validar el caso de uso, no para producción con datos sensibles.

---

## 3. Migrar a self-hosted

### 3.1 Requisitos (Windows)
En Windows se recomienda WSL2. La instalación guiada por CLI configura Gateway, canales, skills y workspace en un único flujo.
🔗 [Onboarding CLI](https://docs.openclaw.ai/start/wizard.md)

### 3.2 Instalación y primer arranque
```bash
npm install -g openclaw
openclaw onboard
```
🔗 [Getting started](https://docs.openclaw.ai/start/getting-started)

### 3.3 Configurar canales
Enlazar WhatsApp, Telegram o Discord vía `openclaw configure` / `openclaw agents add <name>`.
🔗 [Setup por canal](https://docs.openclaw.ai/)

### 3.4 Workspace y archivos clave
`~/.openclaw/openclaw.json` (config) y `~/.openclaw/workspace/` con `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `MEMORY.md`, `skills/`.

---

## 4. Skills y automatización

### 4.1 ClawHub y comandos básicos
```bash
openclaw skills search "calendar"
openclaw skills install <slug>
openclaw skills list --verbose
```
🔗 [CLI skills reference](https://docs.openclaw.ai/zh-CN/cli/skills) (referencia de comandos, funciona igual en cualquier idioma de interfaz)

### 4.2 Instalar una skill de terceros
Ejemplo real: búsqueda web en tiempo real vía Linkup.
🔗 [Integración Linkup + OpenClaw](https://docs.linkup.so/pages/integrations/openclaw/openclaw.md)

### 4.3 Crear tu propio SKILL.md
Un SKILL.md define cuándo se activa la skill y qué comandos expone — mismo patrón que usas ya en Claude Code.

### 4.4 Cron, webhooks y hooks
Automatizaciones programadas (research diario, resumen matutino) sin depender de n8n para la parte de razonamiento.
🔗 [Tools, skills, cron, webhooks](https://docs.openclaw.ai/)

---

## 5. Seguridad y límites

### 5.1 Allowlists y tokens
El Gateway requiere autorizar cada dispositivo que intenta conectarse; los tokens controlan quién puede operar el agente.

### 5.2 Sandbox de ejecución
El agente opera en un entorno aislado para no tocar directamente archivos personales o sistemas de producción.

### 5.3 Vulnerabilidad RCE conocida — actuar ya
Existe una RCE crítica en versiones anteriores a v2026.1.29. Si vas a instalar, parte directamente de v2026.2.21 o posterior y sigue la guía de hardening.
🔗 [Security guide](https://openclaw-ai.net/en)

### 5.4 Cuándo NO usar OpenClaw
Datos regulados sin revisar dónde corre el Gateway, necesidad de auditoría formal, o si tu caso de uso es un flujo 100% determinista (ahí n8n solo es más barato y predecible).

---

## 6. Fundamentos de LLMs locales

### 6.1 Por qué local: privacidad, coste, offline
Cero dependencia de red, sin coste por token, control total del dato — a cambio de calidad inferior a un modelo frontier y de gestionar tú el hardware.

### 6.2 Cuantización y GGUF explicado
Un modelo cuantizado a 4-bit (Q4) reduce memoria necesaria a costa de precisión; un 70B en Q4 puede correr en 32 GB de RAM sin GPU dedicada, con calidad reducida pero usable.
🔗 [Comparativa de 20 herramientas locales 2026](https://www.iunera.com/kraken/enterprise-ai/top-20-tools-to-run-llms-locally-in-2026-ollama-anythingllm-open-webui-lm-studio-vllm-and-every-real-alternative-compared/)

### 6.3 Requisitos de hardware
GPU con 8+ GB VRAM recomendado para velocidad usable; sin GPU, Ollama cae a CPU (funcional, lento). Revisa `ollama ps` para ver cuántas capas fueron a GPU vs CPU.
🔗 [Ollama Setup Guide 2026](https://www.mayhemcode.com/2026/06/ollama-setup-guide-2026-install-gpu.html)

### 6.4 Modelos recomendados 2026
Llama 3.3, Qwen 2.5/3, DeepSeek, Phi-4, Mistral. Para agentes que llaman herramientas: Qwen2.5 7B/14B o Llama 3.1 8B con buen seguimiento de instrucciones.
🔗 [Ollama model library](https://ollama.com/library)

---

## 7. Ollama en Windows

### 7.1 Instalación oficial
Descarga desde `ollama.com/download/windows`, instalador ~5 MB, no requiere admin. Abre terminal **nuevo** tras instalar (el PATH no se actualiza en el que ya tenías abierto).
🔗 [Guía oficial paso a paso](https://ai-ollama.github.io/install-windows.html)

### 7.2 Verificar GPU
NVIDIA vía CUDA, AMD vía DirectML — detección automática. Si `gpu layers` marca 0, la GPU no se está usando.
🔗 [GPU Acceleration guide](https://ai-ollama.github.io/)

### 7.3 Gestión de modelos
```bash
ollama pull llama3.1:8b
ollama pull mxbai-embed-large   # para RAG, ver apartado 9
ollama list
```

### 7.4 API REST local
Ollama expone API en `http://localhost:11434`, compatible en gran parte con el formato OpenAI — así puedes apuntar tus propios scripts sin cambiar mucho código.
🔗 [Ollama API docs](https://ollama.com/download/windows)

---

## 8. Interfaces sobre Ollama

### 8.1 LM Studio — explorar y comparar modelos rápido
GUI para descargar y probar modelos GGUF de Hugging Face directamente, sin pasar por Ollama.
🔗 [lmstudio.ai](https://lmstudio.ai/)

### 8.2 Open WebUI — interfaz tipo ChatGPT sobre Ollama
Multi-usuario, historial de conversación, RAG documental, se despliega con Docker.
🔗 [docs.openwebui.com](https://docs.openwebui.com/)

### 8.3 AnythingLLM — RAG + agentes sobre tus documentos
Combina inferencia local con ingestión de documentos y base de conocimiento; soporta Ollama, LM Studio y proveedores cloud como backend.
🔗 [AnythingLLM GitHub](https://github.com/Mintplex-Labs/anything-llm)

### 8.4 Cuándo usar cada una
LM Studio para explorar modelo nuevo; Open WebUI si quieres una interfaz de uso diario tipo ChatGPT; AnythingLLM si el objetivo es preguntar sobre tus propios documentos de trabajo.

---

## 9. RAG local

### 9.1 Qué es RAG y por qué combinarlo con local
Recuperar fragmentos relevantes de tus documentos antes de generar la respuesta — necesario porque un modelo local de 7-8B tiene memoria y razonamiento limitados comparado con Claude.

### 9.2 Embeddings y vector store
`mxbai-embed-large` o `nomic-embed-text` son los embedders estándar en el ecosistema Ollama para indexar tus documentos.
🔗 [Guía práctica de RAG local con Ollama](https://blog.stephenturner.us/p/gui-local-llm-rag)

### 9.3 Montar RAG con AnythingLLM
Instalación, conexión a Ollama como backend, ingestión de PDFs/markdown, primera consulta sobre tu propia base documental.
🔗 [AnythingLLM vs Open WebUI para RAG](https://localaimaster.com/blog/anythingllm-vs-open-webui)

### 9.4 Limitaciones frente a Claude con contexto largo
Con Claude puedes meter documentos enteros en el contexto sin indexar nada. RAG local solo compensa cuando el corpus es demasiado grande para contexto directo, o el requisito es privacidad/offline — no lo montes "porque sí".

---

## 10. Arquitectura híbrida: local + cloud

### 10.1 Cuándo usar local vs cloud
Local: datos sensibles, volumen alto de llamadas triviales, sin conexión. Cloud (Claude): tareas que requieren razonamiento fuerte, contexto largo, o donde la calidad de salida es lo que vendes.

### 10.2 Failover entre proveedores
OpenClaw soporta configuración de "providers, model configuration, failover, and local model services" — puedes definir un modelo local como opción primaria y una API cloud como fallback si falla o tarda.
🔗 [Providers y failover](https://docs.openclaw.ai/)

### 10.3 Integrar Ollama como fallback en OpenClaw
Configurar en `openclaw.json` un modelo local (Ollama, endpoint `localhost:11434`) junto a tu proveedor cloud actual, con reglas de cuándo usar cada uno.

### 10.4 Decisión práctica: qué llevar a Claude y qué a local
Regla simple: si la tarea es repetitiva, de bajo riesgo y alto volumen → local. Si es puntual, crítica o compleja → Claude/Claude Code, como ya haces hoy.

---

## Ejercicios básicos

1. Desplegar OpenClaw en modo hosted (Koyeb) y conectar un canal de Telegram.
2. Instalar Ollama en Windows, descargar `llama3.1:8b`, comparar 3 prompts iguales contra Claude y anotar diferencias de calidad.
3. Instalar Open WebUI conectado a tu Ollama local y subir un PDF para hacer RAG básico sobre él.
4. Crear un `SKILL.md` simple para OpenClaw (ejemplo: recordatorio diario a una hora fija).
5. Configurar el allowlist/token de un OpenClaw self-hosted y verificar que un dispositivo no autorizado no puede conectarse.

## Ejercicios avanzados

1. Migrar tu instancia OpenClaw de hosted a self-hosted vía Docker/WSL2, con persistencia del workspace.
2. Montar un cron job en OpenClaw que dispare cada mañana una skill de research web + resumen entregado por Telegram.
3. Construir un pipeline RAG completo con AnythingLLM sobre documentos reales de tu trabajo y comparar calidad de respuesta contra pegar los mismos documentos en un chat de Claude.
4. Configurar failover real: agente que use Ollama local por defecto y haga fallback automático a Claude API si el modelo local falla o supera un umbral de latencia.
5. Benchmark de cuantizaciones (Q4 vs Q8) del mismo modelo 8B en tu hardware: medir tokens/segundo y evaluar de forma ciega la calidad de 10 respuestas.

---

## Crítica constructiva del curso

Cosas que haría distinto si fuera solo para mí:

- **Orden invertido posible.** Podrías empezar por el bloque LLM local (6-10) y dejar OpenClaw para el final, porque el apartado 10 conecta ambos — OpenClaw termina siendo el orquestador que decide cuándo usar tu modelo local. El orden actual (OpenClaw primero) funciona igual, es cuestión de gusto.
- **Bloque 2+3 se solapa.** Hosted y self-hosted podrían fusionarse en un apartado y liberar hueco para "multi-agent routing", una capacidad real de OpenClaw que aquí no se cubre y que sí es relevante si ya trabajas con varios agentes.
- **Bloque RAG (9) es el más prescindible para ti.** Dado que ya usas Claude con contexto largo a diario, montar RAG local solo te aporta valor si hay un requisito real de offline/privacidad. Si no lo hay, es la sección con menor ROI de las diez — la dejaría pero no la priorizaría.
- **Apartado 5 (seguridad) debería ir más arriba, no al final.** La RCE crítica mencionada es un dato operativo, no una curiosidad — si vas a instalar self-hosted, léela antes del apartado 3, no después.
- **Falta explícitamente "coste real".** No hay apartado sobre cuánto cuesta cada opción (hosted OpenClaw ~22€/mes, cloud APIs, electricidad de tener GPU encendida 24/7 para local) — para decidir arquitectura híbrida (bloque 10) ese dato pesa tanto como la latencia.

Si quieres, reordeno el curso aplicando estos cambios en vez de mantener la versión actual.