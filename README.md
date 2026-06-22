<p align="center">
  <img src="images/neteclogo (2).png" alt="Netec logo" width="300"/>
</p>

<h1 align="center">Inteligencia Artificial Generativa Intermediate</h1>

<p align="center">
  <strong>Plataforma de laboratorios prácticos guiados</strong><br/>
  Desarrollo, integración, evaluación y despliegue de soluciones empresariales con IA Generativa.
</p>

<p align="center">
  <a href="#-ruta-de-aprendizaje"><strong>Ruta de aprendizaje</strong></a> ·
  <a href="#-menú-de-laboratorios"><strong>Menú de laboratorios</strong></a> ·
  <a href="#-resumen-ejecutivo"><strong>Resumen ejecutivo</strong></a> ·
  <a href="#-contacto-y-más-información"><strong>Contacto</strong></a>
</p>

---

## Bienvenida

Te damos la bienvenida a la **plataforma de laboratorios** del curso **Inteligencia Artificial Generativa Intermediate**. En este espacio desarrollarás prácticas guiadas para construir soluciones reales con IA Generativa, desde el consumo de APIs y respuestas estructuradas hasta arquitecturas con RAG, agentes, MCP, evaluación, observabilidad, seguridad, contenedores y despliegue.

Este curso está diseñado para **desarrolladores** que necesitan llevar sus conocimientos más allá del uso básico de interfaces de chat. A lo largo de los laboratorios construirás componentes reutilizables y sistemas completos con enfoque profesional, validación técnica y buenas prácticas de ingeniería.

> **Objetivo de la plataforma:** que puedas avanzar laboratorio por laboratorio con instrucciones claras, comandos listos para ejecutar, validaciones por tarea y entregables concretos.

---

## Ruta de aprendizaje

La ruta está organizada de forma progresiva. Cada laboratorio refuerza una capacidad práctica y prepara el terreno para el siguiente.

| Bloque | Capítulos | Enfoque | Resultado esperado |
|---|---:|---|---|
| **Fundamentos aplicados** | 1–3 | Costos, selección de modelos, API, validación y clientes asíncronos | Scripts y clientes robustos para consumir modelos generativos |
| **Calidad y seguridad** | 4 | Auditoría automática de código con LLM | Pipeline inicial de revisión técnica y seguridad |
| **RAG y memoria** | 5–6 | Ingesta documental, chunking, ChromaDB y memoria semántica | Aplicaciones con recuperación de contexto persistente |
| **Optimización y agentes** | 7–9 | Fine-tuning dataset, function calling y MCP | Agentes con herramientas y servidores seguros |
| **Evaluación y observabilidad** | 10–11 | Golden Dataset, métricas, LangSmith y regresión | Evaluación objetiva y trazabilidad de sistemas GenAI |
| **Despliegue y capstone** | 12–13 | Docker, secretos, seguridad, UI, RAG, agentes y documentación | Sistema GenAI completo listo para demostrar |

---

## Recomendaciones antes de comenzar

- Trabaja los laboratorios en orden, especialmente desde el Capítulo 5 en adelante.
- Usa **Visual Studio Code** y una terminal compatible como **Git Bash** en Windows.
- Crea un entorno virtual por laboratorio para evitar conflictos de dependencias.
- Nunca publiques archivos `.env`, API keys, tokens o secretos en repositorios.
- Valida cada tarea antes de continuar; las prácticas están diseñadas para construir sobre resultados previos.

---

## Menú de laboratorios

### Capítulo 1 — Costos, modelos y criterios de selección

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Evaluar y comparar costos entre OpenAI, Gemini y Anthropic](Capitulo01/README.md) |
| **Descripción** | Desarrollarás un script en Python que estima y compara automáticamente el costo de procesar un dataset específico con distintos proveedores de modelos generativos. |
| **Resultado** | Comparador técnico de costo, tokens y recomendación por caso de uso. |
| **Duración estimada** | 45 min |

---

### Capítulo 2 — Router de modelos con FastAPI

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Crear un Router de Modelos con FastAPI](Capitulo02/README.md) |
| **Descripción** | Construirás el boilerplate de una arquitectura en FastAPI que clasifica prompts por complejidad y enruta la carga de trabajo hacia modelos adecuados. |
| **Resultado** | API base con endpoint de chat, clasificador y lógica de enrutamiento. |
| **Duración estimada** | 45 min |

---

### Capítulo 3 — Cliente LLM asíncrono y respuestas estructuradas

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Implementar un cliente Python asíncrono con Pydantic](Capitulo03/README.md) |
| **Descripción** | Implementarás un cliente asíncrono que fuerza respuestas estructuradas, valida salidas con Pydantic y maneja reintentos exponenciales. |
| **Resultado** | Cliente robusto para llamadas LLM con validación, retries y control de errores. |
| **Duración estimada** | 50 min |

---

### Capítulo 4 — Auditoría automática de seguridad con LLM

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Configurar un pipeline de revisión automática de seguridad](Capitulo04/README.md) |
| **Descripción** | Configurarás un pipeline que utiliza un LLM para auditar commits o fragmentos de código Python e identificar riesgos técnicos. |
| **Resultado** | Flujo automatizado de revisión con clasificación de severidad y reporte. |
| **Duración estimada** | 40 min |

---

### Capítulo 5 — RAG con ingesta y semantic chunking

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Construir un pipeline de ingesta con Semantic Chunking](Capitulo05/README.md) |
| **Descripción** | Crearás un pipeline de ingesta para documentos técnicos, aplicando segmentación semántica y preparación de datos para recuperación aumentada. |
| **Resultado** | Base documental procesada, chunks validados y estructura lista para búsqueda semántica. |
| **Duración estimada** | 58 min |

---

### Capítulo 6 — Memoria semántica con ChromaDB

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Integrar ChromaDB para memoria y recuperación de contexto](Capitulo06/README.md) |
| **Descripción** | Desarrollarás una aplicación que persiste conversaciones, genera embeddings y recupera contexto relevante mediante búsquedas semánticas. |
| **Resultado** | Aplicación con memoria semántica persistente y consultas contextualizadas. |
| **Duración estimada** | 50 min |

---

### Capítulo 7 — Dataset validado para fine-tuning

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Preparar un dataset validado para fine-tuning](Capitulo07/README.md) |
| **Descripción** | Crearás un script de preprocesamiento que transforma una base de preguntas y respuestas en un dataset limpio, validado y listo para entrenamiento o ajuste. |
| **Resultado** | Dataset estructurado, validado y documentado para procesos de fine-tuning. |
| **Duración estimada** | 50 min |

---

### Capítulo 8 — Agente con Function Calling

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Implementar un agente con Function Calling](Capitulo08/README.md) |
| **Descripción** | Construirás un agente que invoca funciones locales para resolver tareas matemáticas o consultar datos mediante herramientas controladas. |
| **Resultado** | Agente funcional con herramientas, validación de argumentos y respuestas estructuradas. |
| **Duración estimada** | 58 min |

---

### Capítulo 9 — Servidor MCP seguro en Python

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Desarrollar un servidor MCP seguro para sistema de archivos](Capitulo09/README.md) |
| **Descripción** | Implementarás un servidor MCP que expone herramientas, recursos y prompts para interactuar de forma segura con un sandbox de archivos. |
| **Resultado** | Servidor MCP con sandbox, auditoría, validaciones de seguridad y cliente de prueba. |
| **Duración estimada** | 50 min |

---

### Capítulo 10 — Evaluación con Golden Dataset

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Evaluar la fidelidad de respuestas contra un Golden Dataset](Capitulo10/README.md) |
| **Descripción** | Crearás un framework de evaluación para comparar respuestas de un chatbot contra referencias usando métricas léxicas y evaluación semántica. |
| **Resultado** | Reporte de calidad con métricas, análisis de divergencias y resultados exportables. |
| **Duración estimada** | 40 min |

---

### Capítulo 11 — Observabilidad y evaluación con LangSmith

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Instrumentar un agente con LangSmith](Capitulo11/README.md) |
| **Descripción** | Instrumentarás un agente para analizar trazas, latencia, uso de herramientas, consistencia y regresiones entre versiones. |
| **Resultado** | Sistema observado con trazas, evaluación, comparación v1/v2 y pruebas de regresión. |
| **Duración estimada** | 40 min |

---

### Capítulo 12 — Docker, secretos y seguridad para GenAI

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Crear un Dockerfile seguro para una solución GenAI](Capitulo12/README.md) |
| **Descripción** | Contenerizarás una aplicación GenAI con Docker multi-stage, usuario no-root, gestión de secretos, controles de seguridad y Docker Compose. |
| **Resultado** | Imagen segura, Compose funcional, controles de seguridad y documentación de despliegue. |
| **Duración estimada** | 50 min |

---

### Capítulo 13 — Proyecto final integrador

| Campo | Detalle |
|---|---|
| **Laboratorio** | [Desarrollar un sistema GenAI completo con API, RAG, agente, evaluación, UI y Docker](Capitulo13/README.md) |
| **Descripción** | Construirás un proyecto final guiado que integra API segura, RAG híbrido, agente con herramientas, UI minimalista, evaluación, contenedores y documentación técnica. |
| **Resultado** | Sistema GenAI Capstone listo para demostración técnica y entrega final. |
| **Duración estimada** | 179 min |

---

## Resumen ejecutivo

| Capítulo | Laboratorio | Duración | Producto principal |
|---:|---|---:|---|
| 1 | Costos entre proveedores | 45 min | Comparador de costos |
| 2 | Router de modelos | 45 min | API de enrutamiento |
| 3 | Cliente asíncrono estructurado | 50 min | Cliente LLM robusto |
| 4 | Auditoría de seguridad | 40 min | Pipeline de revisión |
| 5 | Semantic Chunking | 58 min | Pipeline de ingesta |
| 6 | Memoria semántica | 50 min | App con ChromaDB |
| 7 | Dataset para fine-tuning | 50 min | Dataset validado |
| 8 | Function Calling | 58 min | Agente con herramientas |
| 9 | Servidor MCP seguro | 50 min | Servidor MCP + cliente |
| 10 | Golden Dataset | 40 min | Evaluador de fidelidad |
| 11 | LangSmith | 40 min | Observabilidad y regresión |
| 12 | Docker y seguridad | 50 min | Imagen y Compose seguros |
| 13 | Capstone GenAI | 179 min | Sistema completo |

---

## Contacto y más información

Si tienes alguna pregunta o necesitas más detalles, no dudes en [contactarnos](mailto:soporte@netec.com). También puedes encontrar más recursos en nuestra [página](https://netec.com).

---

<p align="center">
  <strong>¡Gracias por visitar nuestra plataforma!</strong><br/>
  No olvides revisar todos los laboratorios y comenzar tu viaje de aprendizaje hoy mismo.
</p>
