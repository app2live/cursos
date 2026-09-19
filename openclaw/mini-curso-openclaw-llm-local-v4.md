# Mini-curso avanzado: OpenClaw + LLMs Local/Cloud
## Arquitectura práctica de agentes, routing de modelos e infraestructura híbrida

**Perfil objetivo:** usuario técnico que ya utiliza Claude, Claude Code, Claude Cowork, agentes, n8n, Docker, PostgreSQL/pgvector y Ollama. El curso evita fundamentos y se centra en los conocimientos que realmente amplían ese stack.

**Objetivo:** pasar de saber utilizar agentes y modelos a poder diseñar, desplegar, evaluar y operar un sistema de agentes donde OpenClaw actúe como plano de control y los LLM locales/cloud se utilicen según calidad, coste, latencia, privacidad y disponibilidad.

**Nivel:** intermedio/avanzado.

**Laboratorio de referencia:** ThinkPad T15 Core i7, 48 GB RAM, Intel Iris Xe integrada y sin GPU dedicada. El hardware se utiliza deliberadamente como parte del aprendizaje.

---

# Tabla de contenidos

## Bloque A — OpenClaw como plano de control
1. [Qué problema resuelve OpenClaw](#1-qué-problema-resuelve-openclaw)
2. [Arquitectura interna y ciclo de ejecución](#2-arquitectura-interna-y-ciclo-de-ejecución)
3. [Canales, sesiones, identidad y routing](#3-canales-sesiones-identidad-y-routing)
4. [Skills, tools, MCP y agentes](#4-skills-tools-mcp-y-agentes)
5. [Memoria y contexto](#5-memoria-y-contexto)
6. [Multi-agent y agent routing](#6-multi-agent-y-agent-routing)
7. [Automatización: OpenClaw, n8n, cron y webhooks](#7-automatización-openclaw-n8n-cron-y-webhooks)
8. [Seguridad y operación self-hosted](#8-seguridad-y-operación-self-hosted)

## Bloque B — LLMs locales y cloud
9. [Cómo funciona realmente la inferencia local](#9-cómo-funciona-realmente-la-inferencia-local)
10. [Ollama como runtime local](#10-ollama-como-runtime-local)
11. [Cuantización, contexto, memoria y rendimiento](#11-cuantización-contexto-memoria-y-rendimiento)
12. [Evaluación y benchmarking de modelos](#12-evaluación-y-benchmarking-de-modelos)
13. [Model routing: elegir el modelo adecuado](#13-model-routing-elegir-el-modelo-adecuado)
14. [RAG: cuándo sí, cuándo no](#14-rag-cuándo-sí-cuándo-no)
15. [Arquitectura híbrida local/cloud](#15-arquitectura-híbrida-localcloud)
16. [Observabilidad, costes y resiliencia](#16-observabilidad-costes-y-resiliencia)

## Proyecto final
17. [Diseña tu propio AI Control Plane](#17-proyecto-final-diseña-tu-propio-ai-control-plane)

## Laboratorio
- [5 ejercicios fundamentales](#5-ejercicios-fundamentales)
- [7 ejercicios avanzados](#7-ejercicios-avanzados)
- [Criterios de evaluación](#criterios-de-evaluación)
- [Recursos gratuitos](#recursos-gratuitos)

---

# 1. Qué problema resuelve OpenClaw

## 1.1 De chatbot a sistema operativo de agentes
OpenClaw no debe estudiarse como "otro chatbot". Su interés está en disponer de una capa persistente que recibe eventos, mantiene sesiones, decide qué agente/modelo utilizar, accede a herramientas y devuelve acciones o respuestas.

**Idea clave:** Claude Code es principalmente un agente de coding; OpenClaw es un plano de control para agentes persistentes, canales, sesiones, herramientas y automatizaciones.

## 1.2 Qué aporta respecto a usar directamente Claude
Compara canal, sesión persistente, memoria, herramientas, skills, múltiples agentes y automatización.

## 1.3 OpenClaw frente a n8n
n8n destaca en workflows deterministas, integraciones, triggers y transformaciones. OpenClaw aporta interacción agentiva, contexto y decisiones dinámicas. La arquitectura interesante suele ser:
`Canal → OpenClaw → agente → n8n/API/tool → sistema externo`.

## 1.4 OpenClaw frente a MCP
MCP responde principalmente a "cómo expongo herramientas/contexto a un agente"; OpenClaw resuelve una capa más amplia: canales, sesiones, agentes, herramientas y operación.

**Recurso:** https://docs.openclaw.ai/

---

# 2. Arquitectura interna y ciclo de ejecución

## 2.1 Gateway como centro de control
Estudia el Gateway como punto central de entrada, routing, sesiones, agentes, herramientas y configuración.

## 2.2 Workspace y configuración
Distingue configuración del Gateway, workspace, skills, instrucciones, memoria y secretos.

## 2.3 Ciclo de una petición
Modela:
`evento → canal → sesión → agente → contexto → modelo → tools → resultado → respuesta`

Para cada etapa pregunta qué estado necesita, qué puede fallar y qué debe persistir.

## 2.4 El modelo no es el agente completo
Un LLM es inferencia. El comportamiento agentivo aparece al combinar modelo, contexto, instrucciones, herramientas, memoria, bucle de ejecución, permisos y estado.

---

# 3. Canales, sesiones, identidad y routing

## 3.1 Canales de entrada
Clasifica canales en mensajería, interfaz web, eventos y APIs.

## 3.2 Sesiones
Estudia identificación, aislamiento, continuidad, contexto, concurrencia y expiración.

## 3.3 Identidad y permisos
Separa quién habla, qué agente atiende, qué herramientas puede usar y qué operaciones requieren aprobación.

## 3.4 Routing por contexto
Un mismo sistema puede decidir:
`personal → agente personal`
`técnico → agente técnico`
`research → agente research`
`trivial → modelo local`.

---

# 4. Skills, tools, MCP y agentes

## 4.1 Tool calling
Entiende:
`LLM → decisión de tool → ejecución → resultado → nuevo contexto → LLM`.

## 4.2 Skills
Una skill encapsula conocimiento, instrucciones o procedimientos reutilizables. Aprende cuándo merece una skill y cuándo basta un prompt.

## 4.3 MCP
Compara tool nativa, skill, MCP, webhook/API y workflow n8n.

## 4.4 Diseño de herramientas
Define inputs explícitos, outputs estructurados, validación, permisos mínimos y errores recuperables.

## 4.5 Agent vs workflow
Workflow: sabes de antemano qué pasos ocurren. Agent: necesitas que el sistema decida qué pasos ejecutar. Los sistemas buenos combinan ambos.

---

# 5. Memoria y contexto

## 5.1 Context window ≠ memoria
Un contexto largo permite ver información durante una ejecución; memoria conserva información relevante entre ejecuciones.

## 5.2 Tipos de memoria
Sesión, usuario, operacional, documental y estado de workflow.

## 5.3 Memoria estructurada vs vectorial
Preferencias y estado suelen ser estructurados; documentos pueden requerir retrieval; conocimiento semántico puede beneficiarse de vectores.

## 5.4 Estrategia para tu stack
Compara memoria nativa del agente con PostgreSQL/pgvector. Pregunta qué información necesita recuperación semántica y cuál acceso determinista.

---

# 6. Multi-agent y agent routing

**Sección central: es uno de los huecos más interesantes frente a Claude Code/Cowork.**

## 6.1 Agente especializado vs generalista
Diseña agentes coding, research, administrativo y personal. Evalúa cuándo dividirlos.

## 6.2 Routing entre agentes
Define reglas por canal, usuario, tarea, sensibilidad, coste y complejidad.

## 6.3 Handoff
Estudia qué contexto transferir, qué resultado exigir, cómo evitar ciclos y cómo controlar permisos.

## 6.4 Arquitectura jerárquica
Compara:
`router → especialistas`
`agente principal → subagentes`
`workflow → agente → workflow`.

## 6.5 Fallos de multi-agent
Más agentes implican más latencia, coste, contexto duplicado y dificultad de debugging. Justifica cada división.

---

# 7. Automatización: OpenClaw, n8n, cron y webhooks

## 7.1 Cron
Úsalo para tareas temporales simples y recurrentes.

## 7.2 Webhooks
Útiles cuando un evento externo inicia una ejecución.

## 7.3 n8n como capa determinista
Mantén n8n para integraciones, transformaciones, trazabilidad y procesos reproducibles.

## 7.4 OpenClaw como capa agentiva
Úsalo cuando hay ambigüedad, conversación, decisión, memoria o selección dinámica de herramientas.

## 7.5 Arquitectura combinada
Diseña tanto:
`Telegram → OpenClaw → decisión → n8n → APIs/DB`
como:
`Webhook → n8n → OpenClaw → decisión → n8n`.

---

# 8. Seguridad y operación self-hosted

## 8.1 Mínimo privilegio
Un agente con shell, navegador, correo y filesystem tiene una superficie de ataque mucho mayor que un chatbot.

## 8.2 Secretos
Nunca incrustes claves en prompts, repositorios, skills o imágenes Docker.

## 8.3 Exposición de red
Distingue localhost, LAN, VPN, reverse proxy y acceso público. No expongas administración directamente a Internet sin una estrategia de autenticación y red.

## 8.4 Threat model
Analiza prompt injection, tool abuse, fuga de secretos, acceso indebido, skills maliciosas y documentos no confiables.

## 8.5 Backups y recuperación
Determina qué debes recuperar: configuración, workspace, skills, memoria, bases de datos y configuración externa.

**Recurso:** https://docs.openclaw.ai/

---

# 9. Cómo funciona realmente la inferencia local

## 9.1 Parámetros y memoria
La memoria real depende de pesos, precisión, KV cache, contexto, runtime y overhead.

## 9.2 Prefill y decode
Prefill procesa el prompt; decode genera tokens. Un prompt enorme puede penalizar la latencia aunque la generación sea rápida.

## 9.3 RAM, VRAM y CPU
Analiza qué sucede cuando todo cabe en VRAM, cuando parte se descarga a RAM y cuando se ejecuta principalmente en CPU.

## 9.4 Contexto y KV cache
Un contexto mayor puede aumentar mucho el consumo de memoria. No confundas modelo grande con contexto grande.

## 9.5 Por qué un modelo local peor puede ser mejor
El valor puede estar en coste marginal, privacidad, disponibilidad, latencia predecible, volumen y control.

---

# 10. Tu ThinkPad T15 como laboratorio de LLMs

Esta edición usa como máquina de referencia tu **ThinkPad T15: Core i7, 48 GB de RAM e Intel Iris Xe integrada, sin GPU dedicada**. El objetivo no es conseguir la máxima velocidad, sino aprender experimentalmente dónde está el límite práctico de la inferencia local y cuándo conviene escalar a cloud.

## 10.1 Qué hace bien el T15
Es una máquina muy válida para OpenClaw, Docker/WSL2, Ollama, n8n, PostgreSQL/pgvector, RAG, agentes y benchmarking CPU/RAM.

## 10.2 Qué no perseguiremos
No intentaremos convertirlo en una workstation de inferencia: modelos frontier grandes, 70B como uso cotidiano o alta concurrencia quedan fuera del objetivo.

## 10.3 Rangos iniciales
- 3–4B: cómodos.
- 7–8B Q4: objetivo principal.
- 12–14B Q4: experimento de límite práctico.
- Modelos mayores: solo como experimento.

## 10.4 Ollama + OpenClaw
Configura el proveedor local y determina qué tareas asume y qué tareas permanecen en cloud.

**Recursos:** https://docs.ollama.com/ · https://docs.ollama.com/integrations/openclaw

---

# 11. Cuantización, contexto, memoria y rendimiento

## 11.1 Q4, Q5, Q8
La cuantización reduce la precisión de los pesos. Trade-off aproximado:
`memoria ↓ ↔ calidad potencial ↓`.

## 11.2 Benchmark Q4 vs Q8
Mide calidad, tokens/s, tiempo hasta primer token, RAM/VRAM y estabilidad.

## 11.3 Contexto como recurso
Prueba varios tamaños de contexto y mide cuándo empieza a degradarse el rendimiento.

## 11.4 Hardware-aware model selection
Construye:
`modelo × cuantización × contexto × hardware × tokens/s × calidad`.

---

# 12. Evaluación y benchmarking de modelos

## 12.1 De impresiones a medición
"Me parece mejor" no es benchmark. Usa tareas reales de coding, extracción, clasificación, resumen, razonamiento y tool calling.

## 12.2 Dataset personal
Construye 20-50 prompts representativos con contexto y criterios de evaluación.

## 12.3 Métricas
Calidad, latencia, tokens/s, coste, errores y uso de herramientas.

## 12.4 Evaluación automática + humana
Combina tests, reglas, comparación estructurada y evaluación humana. No delegues toda la evaluación a otro LLM sin validar el evaluador.

## 12.5 Modelo ganador ≠ modelo más potente
El óptimo maximiza utilidad bajo restricciones reales.

---

# 13. Model routing: elegir el modelo adecuado

## 13.1 Routing por complejidad
Ejemplo:
`simple → local pequeño`
`resumen → local`
`coding complejo → Claude`
`research complejo → cloud`
`datos sensibles → local`.

## 13.2 Routing por coste
Define presupuesto por tarea o agente.

## 13.3 Routing por privacidad
Clasifica información pública, interna y confidencial.

## 13.4 Routing por disponibilidad
Diseña:
`modelo principal → fallback → segundo fallback`.

## 13.5 LiteLLM como capa opcional
Estudia LiteLLM cuando necesites interfaz unificada, routing, retries, fallbacks y coste. No lo añadas solo por moda.

**Recurso:** https://docs.litellm.ai/

---

# 14. RAG: cuándo sí, cuándo no

## 14.1 Primero prueba contexto directo
Compara:
`documento → contexto → modelo`
frente a:
`documentos → retrieval → contexto → modelo`.

## 14.2 Embeddings
Comprende embedding model, dimensión, similitud, chunking y metadata.

## 14.3 pgvector
Como ya conoces PostgreSQL/pgvector, céntrate en chunking, filtros, búsqueda híbrida, reranking y evaluación de retrieval.

## 14.4 RAG vs contexto largo
Usa el mismo corpus y compara calidad, latencia y complejidad.

## 14.5 Cuándo evitar RAG
Si el corpus cabe en contexto y retrieval no mejora precisión, la complejidad puede no compensar.

---

# 15. Arquitectura híbrida local/cloud

## 15.1 Arquitectura de referencia

```text
Canales → OpenClaw → Router → Agentes
                         │
                  ┌──────┴──────┐
                  ▼             ▼
               Ollama         Cloud
               local          Claude/API
                  │             │
                  └──────┬──────┘
                         ▼
                    Tools / n8n
                         │
                    PostgreSQL
```

## 15.2 Local-first
Prueba local cuando la tarea sea sencilla, sensible o de alto volumen.

## 15.3 Cloud escalation
Escala cuando el local falla o la calidad/contexto requerido es superior.

## 15.4 Routing vs fallback
Routing decide antes; fallback cambia cuando el primero falla o no está disponible.

## 15.5 Coste real
Incluye API, electricidad, hardware, mantenimiento y tiempo de operación. "Local = gratis" es una simplificación.

---

# 16. Observabilidad, costes y resiliencia

## 16.1 Qué registrar
Agente, modelo, tokens, duración, herramientas, resultado, errores y coste estimado.

## 16.2 Trazabilidad
Debes poder explicar por qué un agente respondió algo: modelo, instrucciones relevantes, tools y resultados recuperados.

## 16.3 Control de coste
Define presupuestos por agente, usuario, tarea y proveedor.

## 16.4 Resiliencia
Simula API caída, Ollama ocupado, timeout, tool failure, respuesta inválida y contexto excesivo.

## 16.5 Observabilidad como requisito
En sistemas agentivos, depurar sin trazabilidad es especialmente difícil.

---

# 17. Proyecto final: diseña tu propio AI Control Plane

## 17.1 Objetivo
Combina OpenClaw, Ollama, un modelo cloud, n8n, PostgreSQL/pgvector solo donde aporte valor, herramientas, memoria, routing y seguridad.

## 17.2 Caso de uso
Construye un asistente técnico personal que reciba una petición desde un canal y decida si:
1. resolver con local;
2. usar una herramienta;
3. delegar a n8n;
4. escalar a cloud;
5. recuperar memoria/documentación.

## 17.3 Restricciones
Debe minimizar coste, evitar enviar datos sensibles a cloud, tener fallback, registrar ejecuciones, limitar permisos y ser recuperable.

## 17.4 Entregables
1. Diagrama.
2. Matriz agente/modelo.
3. Política de routing.
4. Threat model.
5. Benchmark.
6. Coste mensual.
7. Backup/restore.
8. Decisiones y trade-offs.

## 17.5 Pregunta final
Cada componente debe justificar su existencia. Si está porque "es una herramienta de moda", elimínalo.

---

# 5 ejercicios fundamentales

## Ejercicio 1 — OpenClaw como control plane
Instala OpenClaw en un entorno de prueba y documenta el recorrido de una petición desde canal hasta respuesta.

## Ejercicio 2 — Primer benchmark del T15
Compara un 7–8B Q4 y un 12–14B Q4 con 10 tareas reales. Mide calidad, latencia, tokens/s y RAM.

## Ejercicio 3 — Tool + workflow
Crea una herramienta que delegue una tarea determinista a n8n y define cuándo debe utilizarla el agente.

## Ejercicio 4 — Local vs cloud
Ejecuta las mismas tareas con tu modelo local y Claude/cloud. Compara calidad, latencia y coste.

## Ejercicio 5 — RAG con criterio
Compara contexto directo frente a RAG sobre el mismo corpus y decide si la complejidad está justificada.

---

# 8 ejercicios avanzados

## Avanzado 1 — Multi-agent router
Crea agentes coding, research y general y enruta automáticamente las peticiones.

## Avanzado 2 — Model router local/cloud
Implementa routing por complejidad y fallback cuando falle el modelo local.

## Avanzado 3 — Benchmark Q4 vs Q8
Compara dos cuantizaciones del mismo modelo manteniendo constantes prompt, contexto y temperatura.

## Avanzado 4 — Límite práctico del T15
Prueba 8B, 14B y un modelo mayor cuantizado. Determina cuándo la latencia deja de ser aceptable y si el aumento de tamaño mejora realmente tus tareas.

## Avanzado 5 — OpenClaw + n8n + pgvector
Construye un flujo completo y justifica qué parte debe ser determinista y cuál agentiva.

## Avanzado 6 — Seguridad adversarial
Prueba prompt injection, tool abuse y fuga de secretos; aplica mitigaciones y repite.

## Avanzado 7 — Resiliencia
Simula Ollama/API caídos, timeout, tool failure y respuestas inválidas.

## Avanzado 8 — AI Control Plane completo
Integra OpenClaw, multi-agent, model routing, Ollama, cloud, n8n, memoria, observabilidad, seguridad y fallback.

---

# Matriz de experimentos del T15

| Experimento | Modelo | Cuantización | Objetivo |
|---|---|---|---|
| A | 7–8B | Q4 | línea base |
| B | 7–8B | Q8 | calidad vs memoria |
| C | 12–14B | Q4 | tamaño vs velocidad |
| D | 12–14B | Q4 | efecto del contexto |
| E | mayor | Q4 | límite práctico |
| F | Claude | cloud | referencia de calidad/coste |
| G | local + OpenClaw | según modelo | tool calling |
| H | local + n8n | según modelo | arquitectura end-to-end |

**Regla:** cambia una variable cada vez cuando quieras atribuir una mejora o degradación.

# Criterios de evaluación

Puntúa de 0 a 5:

| Criterio | Pregunta |
|---|---|
| Utilidad | ¿Resuelve un problema real? |
| Simplicidad | ¿Hay componentes innecesarios? |
| Calidad | ¿Es suficientemente bueno? |
| Coste | ¿Está controlado? |
| Latencia | ¿Es suficientemente rápido? |
| Privacidad | ¿Los datos van al lugar correcto? |
| Seguridad | ¿Los permisos están limitados? |
| Resiliencia | ¿Qué pasa cuando falla algo? |
| Observabilidad | ¿Puedes saber qué ocurrió? |
| Mantenibilidad | ¿Podrás mantenerlo en 6 meses? |

**Regla de oro:** una arquitectura más compleja solo gana si el beneficio medido justifica la complejidad.

---

# Recursos gratuitos

## OpenClaw
- https://docs.openclaw.ai/
- https://docs.openclaw.ai/start/getting-started
- https://docs.openclaw.ai/cli

## Ollama
- https://docs.ollama.com/
- https://docs.ollama.com/capabilities/embeddings
- https://docs.ollama.com/integrations/openclaw

## Inferencia
- https://docs.vllm.ai/en/latest/serving/openai_compatible_server/
- https://huggingface.co/docs/transformers/main/en/quantization/overview

## Routing
- https://docs.litellm.ai/
- https://docs.litellm.ai/docs/routing

## RAG
- https://github.com/pgvector/pgvector
- https://cookbook.openai.com/

## Automatización
- https://docs.n8n.io/

---

# Qué NO estudiar en este curso

No merece dedicar tiempo específico a:
- introducción a IA generativa;
- qué es un prompt;
- tutoriales básicos de ChatGPT/Claude;
- Claude Code;
- Claude Cowork;
- Lovable básico;
- n8n básico;
- Ollama básico;
- instalar muchas interfaces gráficas.

El objetivo es subir de:
**"sé utilizar agentes y modelos"**
a:
**"sé diseñar y operar la arquitectura que los conecta."**

---

# Resultado esperado

Al terminar deberías poder responder y demostrar:

1. ¿Cuándo usar OpenClaw y cuándo n8n?
2. ¿Cuándo merece la pena MCP?
3. ¿Cuándo separar agentes?
4. ¿Cómo enrutar entre agentes y modelos?
5. ¿Qué tareas deben ir a Ollama?
6. ¿Cuándo pagar por Claude?
7. ¿Qué impacto tiene Q4 frente a Q8?
8. ¿Cuándo RAG mejora realmente el sistema?
9. ¿Cómo proteger un agente con herramientas?
10. ¿Cómo sobrevivir a fallos?
11. ¿Cuánto cuesta cada arquitectura?
12. ¿Cómo demostrar con datos que una arquitectura es mejor?

Si puedes contestar estas preguntas con un laboratorio propio, has aprendido bastante más que simplemente usar OpenClaw.
