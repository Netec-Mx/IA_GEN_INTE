---LAB_START---
LAB_ID: 11-00-01
---MARKDOWN---
# Instrumentar una cadena de LangChain con LangSmith para identificar cuellos de botella y errores en el flujo de pensamiento de un agente.

## 1. Metadatos

| Campo            | Detalle                                                                 |
|------------------|-------------------------------------------------------------------------|
| **Duración**     | 40 minutos                                                              |
| **Complejidad**  | Media                                                                   |
| **Nivel Bloom**  | Crear                                                                   |
| **Costo estimado** | < $0.10 USD (GPT-4o-mini con dataset de 10 preguntas)                |

---

## 2. Descripción General

En este laboratorio construirás un agente ReAct con LangChain que usa tres herramientas personalizadas, lo instrumentarás con LangSmith para capturar trazas completas y analizarás el dashboard para identificar cuellos de botella de latencia y errores de razonamiento. Aplicarás directamente los conceptos de **evaluación de consistencia** de la lección 11.1: crearás un dataset de referencia en LangSmith, ejecutarás evaluadores semánticos con un modelo juez (a `temperature=0`), calcularás la tasa de acuerdo entre respuestas y compararás dos versiones del agente para detectar regresiones o mejoras de calidad.

---

## 3. Objetivos de Aprendizaje

- [ ] Configurar LangSmith como plataforma de observabilidad habilitando tracing automático (variables de entorno) y manual (`@traceable`).
- [ ] Identificar cuellos de botella de latencia y errores lógicos en el flujo de pensamiento de un agente ReAct usando el dashboard de trazas de LangSmith.
- [ ] Diseñar y ejecutar evaluaciones de consistencia semántica con LangSmith Evaluators midiendo la tasa de acuerdo contra un dataset de referencia.
- [ ] Comparar dos versiones del agente (prompt original vs. prompt mejorado) para detectar regresiones de rendimiento documentando hallazgos cuantitativos.

---

## 4. Prerrequisitos

### Conocimiento
- Python intermedio: decoradores, manejo de excepciones, variables de entorno con `python-dotenv`.
- Familiaridad básica con LangChain: chains, agents, tools (curso IA Generativa Essentials).
- Conceptos de evaluación de consistencia: tasa de acuerdo, modelo juez, pruebas de re-ejecución (Lección 11.1).

### Acceso y Cuentas
- Cuenta activa en [smith.langchain.com](https://smith.langchain.com) (tier gratuito es suficiente).
- OpenAI API key con créditos disponibles (se usará `gpt-4o-mini`).
- Proyecto LangSmith creado previamente (o se creará en el Paso 1).

---

## 5. Entorno del Laboratorio

### Hardware Recomendado

| Recurso      | Mínimo              | Recomendado          |
|--------------|---------------------|----------------------|
| RAM          | 8 GB                | 16 GB                |
| CPU          | 4 núcleos           | 8 núcleos            |
| Disco        | 2 GB libres         | 5 GB libres          |
| Internet     | 10 Mbps             | 25 Mbps              |

### Software Requerido

| Paquete                  | Versión          |
|--------------------------|------------------|
| Python                   | 3.11.x           |
| langchain                | 0.2.x            |
| langchain-openai         | 0.1.x            |
| langsmith                | 0.1.x            |
| openai                   | 1.35.x           |
| python-dotenv            | 1.0.x            |
| pytest                   | 8.x              |

### Comandos de Configuración del Entorno

```bash
# 1. Crear y activar entorno virtual
python -m venv venv_lab11
# En macOS/Linux:
source venv_lab11/bin/activate
# En Windows:
venv_lab11\Scripts\activate

# 2. Instalar dependencias
pip install \
  "langchain==0.2.16" \
  "langchain-openai==0.1.23" \
  "langchain-community==0.2.16" \
  "langsmith==0.1.98" \
  "openai==1.35.14" \
  "python-dotenv==1.0.1" \
  "pytest==8.2.2"

# 3. Verificar instalaciones
python -c "import langchain; print('LangChain:', langchain.__version__)"
python -c "import langsmith; print('LangSmith:', langsmith.__version__)"
```

### Estructura de Archivos del Proyecto

```
lab11/
├── .env                    # API keys (NO subir a Git)
├── .gitignore
├── requirements.txt
├── agent_v1.py             # Agente con prompt original (Paso 2)
├── agent_v2.py             # Agente con prompt mejorado (Paso 5)
├── consistency_eval.py     # Evaluación de consistencia (Paso 4)
└── test_regression.py      # Pruebas de regresión con pytest (Paso 6)
```

Crear el archivo `.gitignore` antes de cualquier commit:

```bash
cat > .gitignore << 'EOF'
.env
__pycache__/
*.pyc
venv_lab11/
.pytest_cache/
*.egg-info/
EOF
```

Crear el archivo `.env`:

```bash
cat > .env << 'EOF'
OPENAI_API_KEY=sk-...tu-clave-aqui...
LANGCHAIN_API_KEY=ls__...tu-clave-langsmith...
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=lab11-agente-react
LANGCHAIN_ENDPOINT=https://api.smith.langchain.com
EOF
```

> ⚠️ **Seguridad:** Nunca incluyas valores reales de API keys en capturas de pantalla, commits o chats de soporte. Verifica que `.env` aparezca en `.gitignore` antes de continuar.

---

## 6. Pasos del Laboratorio

---

### Paso 1: Configurar el Proyecto en LangSmith y Verificar la Conexión

**Objetivo:** Crear el proyecto en LangSmith y confirmar que la conexión desde Python funciona correctamente antes de escribir código del agente.

#### Instrucciones

1. Abre un navegador y ve a [smith.langchain.com](https://smith.langchain.com). Inicia sesión con tu cuenta.

2. En el panel izquierdo, haz clic en **"Projects"** → **"+ New Project"**. Nombra el proyecto `lab11-agente-react` y haz clic en **"Create"**.

3. Ve a **Settings** → **API Keys** → **"Create API Key"**. Copia la clave generada (formato `ls__...`) y pégala en el campo `LANGCHAIN_API_KEY` de tu archivo `.env`.

4. Crea el archivo `verify_connection.py` con el siguiente contenido:

```python
# verify_connection.py
"""
Verifica la conexión con LangSmith y lista los proyectos disponibles.
"""
import os
from dotenv import load_dotenv
from langsmith import Client

load_dotenv()

def verificar_conexion():
    """Verifica que las credenciales de LangSmith son válidas."""
    api_key = os.getenv("LANGCHAIN_API_KEY")
    if not api_key:
        raise ValueError("LANGCHAIN_API_KEY no está configurada en .env")
    
    client = Client(api_key=api_key)
    
    # Listar proyectos para confirmar autenticación
    proyectos = list(client.list_projects())
    nombres = [p.name for p in proyectos]
    
    print("✅ Conexión con LangSmith exitosa")
    print(f"   Proyectos disponibles: {nombres}")
    
    proyecto_objetivo = os.getenv("LANGCHAIN_PROJECT", "lab11-agente-react")
    if proyecto_objetivo in nombres:
        print(f"✅ Proyecto '{proyecto_objetivo}' encontrado y listo")
    else:
        print(f"⚠️  Proyecto '{proyecto_objetivo}' no encontrado. Verifica el nombre en .env")
    
    return client

if __name__ == "__main__":
    verificar_conexion()
```

5. Ejecuta el script:

```bash
python verify_connection.py
```

#### Salida Esperada

```
✅ Conexión con LangSmith exitosa
   Proyectos disponibles: ['lab11-agente-react']
✅ Proyecto 'lab11-agente-react' encontrado y listo
```

#### Verificación

En el dashboard de LangSmith, el proyecto `lab11-agente-react` debe aparecer en la lista de proyectos con estado activo. Si ves el proyecto pero sin runs aún, es correcto: los runs aparecerán a partir del Paso 2.

---

### Paso 2: Construir el Agente ReAct con Tres Herramientas (Sin Tracing Inicial)

**Objetivo:** Implementar el agente base con tres herramientas personalizadas y ejecutarlo sin tracing para establecer una línea base de comportamiento.

#### Instrucciones

1. Crea el archivo `agent_v1.py` con el siguiente contenido completo:

```python
# agent_v1.py
"""
Agente ReAct v1 con tres herramientas personalizadas.
Versión con prompt original (línea base para comparación).
"""
import os
import math
import time
from typing import Optional
from dotenv import load_dotenv

from langchain.agents import AgentExecutor, create_react_agent
from langchain.tools import tool
from langchain_openai import ChatOpenAI
from langchain import hub

load_dotenv()

# ─────────────────────────────────────────────
# HERRAMIENTA 1: Búsqueda simulada
# ─────────────────────────────────────────────
@tool
def busqueda_web(query: str) -> str:
    """
    Realiza una búsqueda web simulada sobre temas de tecnología e IA.
    Usa esta herramienta cuando necesites información factual sobre 
    conceptos de inteligencia artificial, machine learning o tecnología.
    
    Args:
        query: La consulta de búsqueda en lenguaje natural.
    
    Returns:
        Resultado simulado de la búsqueda web.
    """
    # Simulación de búsqueda con base de conocimiento estática
    base_conocimiento = {
        "machine learning": (
            "Machine learning es una rama de la IA que permite a los sistemas "
            "aprender de datos sin ser explícitamente programados. Incluye "
            "aprendizaje supervisado, no supervisado y por refuerzo."
        ),
        "deep learning": (
            "Deep learning usa redes neuronales profundas con múltiples capas "
            "para aprender representaciones jerárquicas de los datos. Es un "
            "subconjunto de machine learning especialmente efectivo para imagen, "
            "texto y audio."
        ),
        "transformer": (
            "La arquitectura Transformer, introducida en 2017 por Vaswani et al., "
            "usa mecanismos de atención para procesar secuencias en paralelo. "
            "Es la base de modelos como BERT, GPT y T5."
        ),
        "rag": (
            "Retrieval-Augmented Generation (RAG) combina recuperación de documentos "
            "con generación de texto. El modelo consulta una base de conocimiento "
            "externa antes de generar su respuesta, reduciendo alucinaciones."
        ),
        "embeddings": (
            "Los embeddings son representaciones vectoriales densas de texto. "
            "Capturan el significado semántico: textos similares tienen vectores "
            "cercanos en el espacio de alta dimensión."
        ),
    }
    
    # Búsqueda por coincidencia parcial (simulación simplificada)
    query_lower = query.lower()
    for clave, resultado in base_conocimiento.items():
        if clave in query_lower:
            # Simular latencia de red
            time.sleep(0.1)
            return f"[Resultado de búsqueda para '{query}']: {resultado}"
    
    return (
        f"[Búsqueda para '{query}']: No se encontraron resultados específicos. "
        "Intenta con términos más precisos como 'machine learning', 'deep learning', "
        "'transformer', 'rag' o 'embeddings'."
    )


# ─────────────────────────────────────────────
# HERRAMIENTA 2: Calculadora
# ─────────────────────────────────────────────
@tool
def calculadora(expresion: str) -> str:
    """
    Evalúa expresiones matemáticas de forma segura.
    Usa esta herramienta para cálculos numéricos, conversiones o 
    estimaciones matemáticas. Soporta operaciones básicas (+, -, *, /), 
    potencias (**), raíces (sqrt) y logaritmos (log).
    
    Args:
        expresion: Expresión matemática como string. 
                   Ejemplos: '2 ** 10', 'math.sqrt(144)', '1024 / 8'
    
    Returns:
        El resultado numérico de la expresión o un mensaje de error.
    """
    # Contexto seguro: solo funciones matemáticas permitidas
    contexto_seguro = {
        "__builtins__": {},
        "math": math,
        "sqrt": math.sqrt,
        "log": math.log,
        "log2": math.log2,
        "log10": math.log10,
        "pi": math.pi,
        "e": math.e,
        "abs": abs,
        "round": round,
        "pow": pow,
    }
    
    try:
        resultado = eval(expresion, contexto_seguro)  # noqa: S307
        return f"Resultado de '{expresion}' = {resultado}"
    except ZeroDivisionError:
        return f"Error: División por cero en '{expresion}'"
    except (SyntaxError, NameError) as e:
        return f"Error de sintaxis en '{expresion}': {e}"
    except Exception as e:
        return f"Error al evaluar '{expresion}': {e}"


# ─────────────────────────────────────────────
# HERRAMIENTA 3: Consulta a base de conocimiento
# ─────────────────────────────────────────────
@tool
def consulta_base_conocimiento(tema: str) -> str:
    """
    Consulta la base de conocimiento interna del sistema sobre 
    benchmarks, métricas y estadísticas de modelos de IA.
    Usa esta herramienta cuando necesites datos cuantitativos sobre 
    rendimiento de modelos, tamaños de datasets o fechas de lanzamiento.
    
    Args:
        tema: El tema a consultar. Ejemplos: 'GPT-4', 'BERT', 'ImageNet'.
    
    Returns:
        Información estructurada de la base de conocimiento interna.
    """
    base_interna = {
        "gpt-4": {
            "parámetros": "~1.8 trillones (estimado, no confirmado oficialmente)",
            "lanzamiento": "Marzo 2023",
            "contexto_max": "128,000 tokens (GPT-4 Turbo)",
            "benchmark_mmlu": "86.4%",
        },
        "gpt-3.5-turbo": {
            "parámetros": "~175 billones (estimado)",
            "lanzamiento": "Noviembre 2022 (ChatGPT)",
            "contexto_max": "16,385 tokens",
            "benchmark_mmlu": "70.0%",
        },
        "bert": {
            "parámetros": "110M (base) / 340M (large)",
            "lanzamiento": "Octubre 2018",
            "arquitectura": "Transformer encoder bidireccional",
            "benchmark_glue": "80.4 (BERT-large)",
        },
        "imagenet": {
            "tipo": "Dataset de clasificación de imágenes",
            "tamaño": "14 millones de imágenes, 1000 clases",
            "top1_sota": "~91% (modelos modernos)",
            "año_creación": "2009",
        },
        "gpt-4o-mini": {
            "lanzamiento": "Julio 2024",
            "contexto_max": "128,000 tokens",
            "costo_input": "$0.15 / 1M tokens",
            "costo_output": "$0.60 / 1M tokens",
            "benchmark_mmlu": "82.0%",
        },
    }
    
    tema_lower = tema.lower().replace("-", "").replace(" ", "")
    
    for clave, datos in base_interna.items():
        clave_normalizada = clave.replace("-", "").replace(" ", "")
        if clave_normalizada in tema_lower or tema_lower in clave_normalizada:
            # Formatear respuesta
            lineas = [f"[Base de conocimiento - {clave.upper()}]"]
            for k, v in datos.items():
                lineas.append(f"  • {k}: {v}")
            return "\n".join(lineas)
    
    temas_disponibles = list(base_interna.keys())
    return (
        f"[Base de conocimiento]: No hay datos sobre '{tema}'. "
        f"Temas disponibles: {temas_disponibles}"
    )


# ─────────────────────────────────────────────
# CONSTRUCCIÓN DEL AGENTE
# ─────────────────────────────────────────────
def crear_agente_v1(verbose: bool = True) -> AgentExecutor:
    """
    Crea el agente ReAct v1 con prompt original.
    
    Returns:
        AgentExecutor listo para usar.
    """
    llm = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0,
        openai_api_key=os.getenv("OPENAI_API_KEY"),
    )
    
    herramientas = [busqueda_web, calculadora, consulta_base_conocimiento]
    
    # Usar el prompt ReAct estándar de LangChain Hub
    prompt = hub.pull("hwchase17/react")
    
    agente = create_react_agent(llm, herramientas, prompt)
    
    ejecutor = AgentExecutor(
        agent=agente,
        tools=herramientas,
        verbose=verbose,
        max_iterations=5,
        handle_parsing_errors=True,
        return_intermediate_steps=True,
    )
    
    return ejecutor


# ─────────────────────────────────────────────
# EJECUCIÓN DE PRUEBA (sin tracing)
# ─────────────────────────────────────────────
if __name__ == "__main__":
    print("=" * 60)
    print("AGENTE ReAct v1 — Línea Base (sin tracing LangSmith)")
    print("=" * 60)
    
    agente = crear_agente_v1(verbose=True)
    
    preguntas_prueba = [
        "¿Cuántos parámetros tiene GPT-4 y qué arquitectura usa?",
        "Si un modelo tiene 175 billones de parámetros y cada parámetro "
        "ocupa 4 bytes, ¿cuántos GB de memoria necesita?",
        "¿Cuál es la diferencia entre RAG y fine-tuning?",
    ]
    
    for i, pregunta in enumerate(preguntas_prueba, 1):
        print(f"\n[Pregunta {i}]: {pregunta}")
        print("-" * 40)
        inicio = time.time()
        resultado = agente.invoke({"input": pregunta})
        duracion = time.time() - inicio
        print(f"\n[Respuesta]: {resultado['output']}")
        print(f"[Latencia]: {duracion:.2f}s")
```

2. Ejecuta el agente en modo línea base (asegúrate de que `LANGCHAIN_TRACING_V2` esté en `false` temporalmente para esta prueba inicial):

```bash
# Deshabilitar tracing temporalmente para la línea base
LANGCHAIN_TRACING_V2=false python agent_v1.py
```

#### Salida Esperada

```
============================================================
AGENTE ReAct v1 — Línea Base (sin tracing LangSmith)
============================================================

[Pregunta 1]: ¿Cuántos parámetros tiene GPT-4 y qué arquitectura usa?
----------------------------------------

> Entering new AgentExecutor chain...
Thought: Necesito consultar la base de conocimiento sobre GPT-4...
Action: consulta_base_conocimiento
Action Input: GPT-4
Observation: [Base de conocimiento - GPT-4]
  • parámetros: ~1.8 trillones (estimado, no confirmado oficialmente)
  ...
Final Answer: GPT-4 tiene aproximadamente 1.8 trillones de parámetros...

[Respuesta]: GPT-4 tiene aproximadamente 1.8 trillones de parámetros...
[Latencia]: 3.45s
```

#### Verificación

Confirma que las tres preguntas producen respuestas coherentes y que el agente usa las herramientas correctas (no debe usar `calculadora` para la pregunta 1, ni `busqueda_web` cuando la respuesta está en la base de conocimiento).

---

### Paso 3: Habilitar LangSmith Tracing y Analizar las Trazas

**Objetivo:** Activar el tracing automático y manual con `@traceable`, ejecutar el agente con tracing activo y analizar el dashboard de LangSmith para identificar latencia y errores.

#### Instrucciones

1. Verifica que tu `.env` tenga `LANGCHAIN_TRACING_V2=true` (debe estar habilitado para este paso).

2. Crea el archivo `agent_v1_traced.py` que extiende el agente con decoradores `@traceable` para captura manual adicional:

```python
# agent_v1_traced.py
"""
Agente ReAct v1 con tracing completo:
- Automático: via variables de entorno LANGCHAIN_TRACING_V2=true
- Manual: via decoradores @traceable en funciones críticas
"""
import os
import time
from dotenv import load_dotenv
from langsmith import traceable, Client
from agent_v1 import crear_agente_v1

load_dotenv()

# ─────────────────────────────────────────────
# FUNCIONES INSTRUMENTADAS CON @traceable
# ─────────────────────────────────────────────

@traceable(
    name="preprocesar_pregunta",
    tags=["preprocessing", "v1"],
    metadata={"version": "1.0", "etapa": "input"}
)
def preprocesar_pregunta(pregunta: str) -> str:
    """
    Preprocesa la pregunta del usuario antes de enviarla al agente.
    Instrumentada con @traceable para capturar latencia de esta etapa.
    """
    # Normalización básica
    pregunta_limpia = pregunta.strip()
    
    # Simular validación (en producción podría incluir detección de idioma,
    # clasificación de intención, etc.)
    time.sleep(0.05)  # Simular procesamiento
    
    return pregunta_limpia


@traceable(
    name="postprocesar_respuesta",
    tags=["postprocessing", "v1"],
    metadata={"version": "1.0", "etapa": "output"}
)
def postprocesar_respuesta(respuesta: str, pregunta_original: str) -> dict:
    """
    Postprocesa la respuesta del agente.
    Instrumentada con @traceable para capturar latencia de esta etapa.
    """
    # Simular enriquecimiento de respuesta
    time.sleep(0.03)
    
    return {
        "respuesta": respuesta,
        "longitud_tokens_estimada": len(respuesta.split()),
        "pregunta_original": pregunta_original,
        "timestamp": time.time(),
    }


@traceable(
    name="pipeline_completo_v1",
    tags=["agent", "v1", "react"],
    metadata={"modelo": "gpt-4o-mini", "version_agente": "1.0"}
)
def ejecutar_pipeline_v1(pregunta: str) -> dict:
    """
    Pipeline completo: preprocesamiento → agente → postprocesamiento.
    El decorador @traceable captura esta función como una traza raíz,
    y las sub-llamadas al agente aparecerán como trazas hijas.
    """
    agente = crear_agente_v1(verbose=False)
    
    # Etapa 1: Preprocesamiento (traza hija)
    pregunta_procesada = preprocesar_pregunta(pregunta)
    
    # Etapa 2: Ejecución del agente (traza hija automática de LangChain)
    inicio_agente = time.time()
    resultado_agente = agente.invoke({"input": pregunta_procesada})
    latencia_agente = time.time() - inicio_agente
    
    # Etapa 3: Postprocesamiento (traza hija)
    resultado_final = postprocesar_respuesta(
        resultado_agente["output"],
        pregunta
    )
    resultado_final["latencia_agente_segundos"] = latencia_agente
    resultado_final["pasos_intermedios"] = len(
        resultado_agente.get("intermediate_steps", [])
    )
    
    return resultado_final


# ─────────────────────────────────────────────
# DATASET DE PREGUNTAS PARA TRACING
# ─────────────────────────────────────────────
PREGUNTAS_EVALUACION = [
    "¿Cuál es la diferencia entre machine learning y deep learning?",
    "Si proceso 1 millón de tokens con GPT-4o-mini, ¿cuánto me costará en USD?",
    "¿Qué es RAG y por qué reduce las alucinaciones?",
    "¿Cuántos GB de VRAM necesito para cargar un modelo de 7 billones de parámetros en float16?",
    "Explica qué son los embeddings y para qué se usan en sistemas de IA.",
]


if __name__ == "__main__":
    print("=" * 60)
    print("AGENTE ReAct v1 — Con Tracing LangSmith ACTIVO")
    print(f"Proyecto: {os.getenv('LANGCHAIN_PROJECT')}")
    print("=" * 60)
    
    resultados = []
    
    for i, pregunta in enumerate(PREGUNTAS_EVALUACION, 1):
        print(f"\n[{i}/{len(PREGUNTAS_EVALUACION)}] Procesando: {pregunta[:60]}...")
        
        inicio_total = time.time()
        try:
            resultado = ejecutar_pipeline_v1(pregunta)
            duracion_total = time.time() - inicio_total
            
            print(f"  ✅ Completado en {duracion_total:.2f}s "
                  f"(agente: {resultado['latencia_agente_segundos']:.2f}s, "
                  f"pasos: {resultado['pasos_intermedios']})")
            resultados.append({
                "pregunta": pregunta,
                "respuesta": resultado["respuesta"],
                "duracion": duracion_total,
                "exito": True,
            })
        except Exception as e:
            duracion_total = time.time() - inicio_total
            print(f"  ❌ Error en {duracion_total:.2f}s: {e}")
            resultados.append({
                "pregunta": pregunta,
                "respuesta": None,
                "duracion": duracion_total,
                "exito": False,
                "error": str(e),
            })
    
    # Resumen
    exitosos = sum(1 for r in resultados if r["exito"])
    latencia_promedio = sum(r["duracion"] for r in resultados) / len(resultados)
    
    print("\n" + "=" * 60)
    print("RESUMEN DE EJECUCIÓN")
    print("=" * 60)
    print(f"  Preguntas procesadas: {len(resultados)}")
    print(f"  Exitosas: {exitosos}/{len(resultados)}")
    print(f"  Latencia promedio: {latencia_promedio:.2f}s")
    print(f"\n  🔍 Revisa las trazas en: https://smith.langchain.com")
    print(f"     Proyecto: {os.getenv('LANGCHAIN_PROJECT')}")
```

3. Ejecuta el agente con tracing activo:

```bash
python agent_v1_traced.py
```

4. Abre el dashboard de LangSmith en [smith.langchain.com](https://smith.langchain.com), navega a tu proyecto `lab11-agente-react` y observa las trazas generadas.

5. En el dashboard, para cada traza, identifica y anota en tu cuaderno:
   - El paso con **mayor latencia** (normalmente la llamada al LLM).
   - Si algún paso del agente usó una herramienta incorrecta.
   - El número de iteraciones del ciclo ReAct (Thought → Action → Observation).
   - Cualquier error de parsing del agente.

#### Salida Esperada

```
============================================================
AGENTE ReAct v1 — Con Tracing LangSmith ACTIVO
Proyecto: lab11-agente-react
============================================================

[1/5] Procesando: ¿Cuál es la diferencia entre machine learning y de...
  ✅ Completado en 4.23s (agente: 4.15s, pasos: 2)

[2/5] Procesando: Si proceso 1 millón de tokens con GPT-4o-mini, ¿cu...
  ✅ Completado en 5.87s (agente: 5.79s, pasos: 3)
...

RESUMEN DE EJECUCIÓN
============================================================
  Preguntas procesadas: 5
  Exitosas: 5/5
  Latencia promedio: 4.89s

  🔍 Revisa las trazas en: https://smith.langchain.com
     Proyecto: lab11-agente-react
```

#### Verificación

En el dashboard de LangSmith debes ver:
- 5 trazas raíz con nombre `pipeline_completo_v1`.
- Cada traza con sub-trazas: `preprocesar_pregunta`, la cadena del agente LangChain, y `postprocesar_respuesta`.
- La traza del agente LangChain debe mostrar el desglose por herramienta y llamada al LLM.
- El panel de **Latency** debe mostrar qué porcentaje del tiempo total corresponde a cada sub-traza.

---

### Paso 4: Crear Dataset y Ejecutar Evaluación de Consistencia

**Objetivo:** Crear un dataset de referencia en LangSmith con 10 preguntas y respuestas esperadas, luego ejecutar evaluadores de consistencia semántica usando el modelo juez (a `temperature=0`).

#### Instrucciones

1. Crea el archivo `consistency_eval.py`:

```python
# consistency_eval.py
"""
Evaluación de consistencia del agente v1 usando LangSmith.

Implementa:
1. Creación de dataset de referencia en LangSmith (10 preguntas)
2. Ejecución del agente sobre el dataset
3. Evaluación de consistencia semántica con modelo juez (temperature=0)
4. Cálculo de tasa de acuerdo y reporte de resultados
"""
import os
import time
from itertools import combinations
from dotenv import load_dotenv
from langsmith import Client, traceable
from langsmith.evaluation import evaluate, LangChainStringEvaluator
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage

load_dotenv()

# ─────────────────────────────────────────────
# DATASET DE REFERENCIA (10 preguntas)
# ─────────────────────────────────────────────
DATASET_REFERENCIA = [
    {
        "pregunta": "¿Cuál es la diferencia entre machine learning y deep learning?",
        "respuesta_esperada": (
            "Machine learning es el campo general donde los sistemas aprenden "
            "de datos. Deep learning es un subconjunto de machine learning que "
            "usa redes neuronales profundas con múltiples capas ocultas."
        ),
    },
    {
        "pregunta": "¿Qué es RAG en el contexto de IA generativa?",
        "respuesta_esperada": (
            "RAG (Retrieval-Augmented Generation) combina recuperación de "
            "documentos con generación de texto. El modelo consulta una base "
            "de conocimiento externa para reducir alucinaciones."
        ),
    },
    {
        "pregunta": "¿Qué son los embeddings en IA?",
        "respuesta_esperada": (
            "Los embeddings son representaciones vectoriales densas de texto "
            "que capturan significado semántico. Textos similares tienen "
            "vectores cercanos en el espacio vectorial."
        ),
    },
    {
        "pregunta": "¿Cuánto cuesta procesar 1 millón de tokens de entrada con GPT-4o-mini?",
        "respuesta_esperada": (
            "El costo de GPT-4o-mini es $0.15 por millón de tokens de entrada, "
            "por lo que 1 millón de tokens de entrada cuesta $0.15 USD."
        ),
    },
    {
        "pregunta": "¿Cuándo fue lanzado BERT y qué arquitectura usa?",
        "respuesta_esperada": (
            "BERT fue lanzado en octubre de 2018 por Google. Usa una "
            "arquitectura Transformer encoder bidireccional."
        ),
    },
    {
        "pregunta": "¿Qué es la arquitectura Transformer?",
        "respuesta_esperada": (
            "La arquitectura Transformer, introducida en 2017, usa mecanismos "
            "de atención para procesar secuencias en paralelo. Es la base de "
            "modelos modernos como BERT y GPT."
        ),
    },
    {
        "pregunta": "¿Cuántos parámetros tiene BERT base?",
        "respuesta_esperada": (
            "BERT base tiene 110 millones de parámetros. "
            "BERT large tiene 340 millones de parámetros."
        ),
    },
    {
        "pregunta": "Si tengo 175 billones de parámetros en float32 (4 bytes cada uno), ¿cuántos GB son?",
        "respuesta_esperada": (
            "175 billones de parámetros × 4 bytes = 700 billones de bytes = "
            "700 GB aproximadamente."
        ),
    },
    {
        "pregunta": "¿Cuál es el score MMLU de GPT-4o-mini?",
        "respuesta_esperada": (
            "GPT-4o-mini tiene un score MMLU de 82.0%."
        ),
    },
    {
        "pregunta": "¿Cuántos tokens de contexto soporta GPT-4 Turbo?",
        "respuesta_esperada": (
            "GPT-4 Turbo soporta hasta 128,000 tokens de contexto máximo."
        ),
    },
]


# ─────────────────────────────────────────────
# FUNCIÓN: Crear o recuperar dataset en LangSmith
# ─────────────────────────────────────────────
def crear_dataset_langsmith(client: Client, nombre: str) -> str:
    """
    Crea el dataset de referencia en LangSmith.
    Si ya existe, lo retorna sin duplicar.
    
    Returns:
        dataset_id: ID del dataset en LangSmith.
    """
    # Verificar si el dataset ya existe
    datasets_existentes = list(client.list_datasets(dataset_name=nombre))
    
    if datasets_existentes:
        dataset_id = str(datasets_existentes[0].id)
        print(f"  ℹ️  Dataset '{nombre}' ya existe (ID: {dataset_id[:8]}...)")
        return dataset_id
    
    # Crear nuevo dataset
    dataset = client.create_dataset(
        dataset_name=nombre,
        description=(
            "Dataset de referencia para evaluación de consistencia del "
            "agente ReAct Lab11. Contiene 10 preguntas sobre IA con "
            "respuestas esperadas."
        ),
    )
    
    # Agregar ejemplos al dataset
    for item in DATASET_REFERENCIA:
        client.create_example(
            inputs={"input": item["pregunta"]},
            outputs={"output": item["respuesta_esperada"]},
            dataset_id=dataset.id,
        )
    
    print(f"  ✅ Dataset '{nombre}' creado con {len(DATASET_REFERENCIA)} ejemplos")
    return str(dataset.id)


# ─────────────────────────────────────────────
# FUNCIÓN: Modelo juez para consistencia semántica
# ─────────────────────────────────────────────
def crear_juez_semantico() -> callable:
    """
    Crea un modelo juez determinista (temperature=0) para evaluar
    si dos respuestas son semánticamente equivalentes.
    
    Implementa el patrón de la Lección 11.1: usar temperature=0
    para garantizar reproducibilidad del árbitro.
    """
    llm_juez = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0,  # CRÍTICO: determinista para reproducibilidad
        openai_api_key=os.getenv("OPENAI_API_KEY"),
    )
    
    def evaluar_consistencia(
        respuesta_generada: str,
        respuesta_esperada: str,
        pregunta: str = ""
    ) -> dict:
        """
        Evalúa si la respuesta generada es semánticamente equivalente
        a la respuesta esperada.
        
        Returns:
            dict con 'es_equivalente' (bool), 'score' (0.0-1.0) y 'razonamiento'.
        """
        prompt = f"""Eres un evaluador experto en IA. Tu tarea es determinar si 
una respuesta generada es semánticamente equivalente y factualmente correcta 
respecto a la respuesta de referencia.

Pregunta: {pregunta}

Respuesta de referencia: {respuesta_esperada}

Respuesta generada: {respuesta_generada}

Evalúa en dos dimensiones:
1. ¿La respuesta generada contiene la información factual clave de la referencia?
2. ¿La respuesta generada NO contradice ningún hecho de la referencia?

Responde en este formato exacto:
EQUIVALENTE: [SI/NO]
SCORE: [0.0 a 1.0, donde 1.0 = perfectamente equivalente]
RAZONAMIENTO: [Una oración explicando tu decisión]"""
        
        respuesta_juez = llm_juez.invoke([HumanMessage(content=prompt)])
        contenido = respuesta_juez.content.strip()
        
        # Parsear respuesta del juez
        lineas = {
            linea.split(":")[0].strip(): ":".join(linea.split(":")[1:]).strip()
            for linea in contenido.split("\n")
            if ":" in linea
        }
        
        es_equivalente = lineas.get("EQUIVALENTE", "NO").upper() == "SI"
        
        try:
            score = float(lineas.get("SCORE", "0.0"))
            score = max(0.0, min(1.0, score))  # Clamp entre 0 y 1
        except ValueError:
            score = 1.0 if es_equivalente else 0.0
        
        razonamiento = lineas.get("RAZONAMIENTO", "Sin razonamiento disponible")
        
        return {
            "es_equivalente": es_equivalente,
            "score": score,
            "razonamiento": razonamiento,
        }
    
    return evaluar_consistencia


# ─────────────────────────────────────────────
# FUNCIÓN: Calcular tasa de acuerdo (Lección 11.1)
# ─────────────────────────────────────────────
def calcular_tasa_acuerdo(respuestas: list[str], juez: callable) -> float:
    """
    Calcula la tasa de acuerdo entre múltiples respuestas
    generadas para la misma consulta.
    
    Implementación directa del concepto de la Lección 11.1.
    
    Args:
        respuestas: Lista de respuestas generadas para la misma consulta.
        juez: Función que compara dos respuestas y retorna dict con 'es_equivalente'.
    
    Returns:
        Tasa de acuerdo entre 0.0 y 1.0.
    """
    pares = list(combinations(respuestas, 2))
    if not pares:
        return 1.0
    
    acuerdos = sum(
        1 for a, b in pares
        if juez(a, b)["es_equivalente"]
    )
    return acuerdos / len(pares)


# ─────────────────────────────────────────────
# FUNCIÓN PRINCIPAL: Ejecutar evaluación completa
# ─────────────────────────────────────────────
@traceable(
    name="evaluacion_consistencia_v1",
    tags=["evaluation", "consistency", "v1"],
    metadata={"tipo": "consistencia_semantica", "umbral": 0.80}
)
def ejecutar_evaluacion_completa(version_agente: str = "v1") -> dict:
    """
    Ejecuta la evaluación completa de consistencia del agente.
    
    Returns:
        Diccionario con métricas de evaluación.
    """
    from agent_v1 import crear_agente_v1
    
    client = Client(api_key=os.getenv("LANGCHAIN_API_KEY"))
    juez = crear_juez_semantico()
    agente = crear_agente_v1(verbose=False)
    
    UMBRAL_MINIMO = 0.80  # 80% de tasa de acuerdo mínima (Lección 11.1)
    
    print(f"\n{'='*60}")
    print(f"EVALUACIÓN DE CONSISTENCIA — Agente {version_agente}")
    print(f"{'='*60}")
    
    resultados_evaluacion = []
    
    for i, item in enumerate(DATASET_REFERENCIA, 1):
        pregunta = item["pregunta"]
        respuesta_esperada = item["respuesta_esperada"]
        
        print(f"\n[{i:02d}/10] {pregunta[:55]}...")
        
        # Generar 3 respuestas del agente para prueba de re-ejecución
        respuestas_generadas = []
        for intento in range(3):
            try:
                resultado = agente.invoke({"input": pregunta})
                respuestas_generadas.append(resultado["output"])
                time.sleep(0.5)  # Rate limiting cortés
            except Exception as e:
                print(f"         ⚠️  Error en intento {intento+1}: {e}")
                respuestas_generadas.append(f"[ERROR: {e}]")
        
        # Evaluar cada respuesta contra la referencia
        scores_vs_referencia = []
        for resp in respuestas_generadas:
            if not resp.startswith("[ERROR"):
                eval_resultado = juez(resp, respuesta_esperada, pregunta)
                scores_vs_referencia.append(eval_resultado["score"])
        
        score_promedio = (
            sum(scores_vs_referencia) / len(scores_vs_referencia)
            if scores_vs_referencia else 0.0
        )
        
        # Calcular tasa de acuerdo entre las 3 respuestas (Lección 11.1)
        respuestas_validas = [r for r in respuestas_generadas if not r.startswith("[ERROR")]
        tasa_acuerdo = calcular_tasa_acuerdo(respuestas_validas, juez)
        
        estado = "✅" if tasa_acuerdo >= UMBRAL_MINIMO else "⚠️ "
        print(f"         {estado} Score vs referencia: {score_promedio:.2f} | "
              f"Tasa acuerdo: {tasa_acuerdo:.0%}")
        
        resultados_evaluacion.append({
            "pregunta": pregunta,
            "score_promedio": score_promedio,
            "tasa_acuerdo": tasa_acuerdo,
            "pasa_umbral": tasa_acuerdo >= UMBRAL_MINIMO,
            "respuestas_generadas": respuestas_generadas,
        })
    
    # Métricas globales
    score_global = sum(r["score_promedio"] for r in resultados_evaluacion) / len(resultados_evaluacion)
    tasa_acuerdo_global = sum(r["tasa_acuerdo"] for r in resultados_evaluacion) / len(resultados_evaluacion)
    preguntas_que_pasan = sum(1 for r in resultados_evaluacion if r["pasa_umbral"])
    
    print(f"\n{'='*60}")
    print("MÉTRICAS GLOBALES")
    print(f"{'='*60}")
    print(f"  Score promedio vs referencia: {score_global:.2f}")
    print(f"  Tasa de acuerdo global:       {tasa_acuerdo_global:.0%}")
    print(f"  Preguntas sobre umbral:        {preguntas_que_pasan}/{len(DATASET_REFERENCIA)}")
    
    if tasa_acuerdo_global >= UMBRAL_MINIMO:
        print(f"\n  ✅ Sistema CONSISTENTE (≥ {UMBRAL_MINIMO:.0%})")
    else:
        print(f"\n  ❌ ALERTA: Consistencia por debajo del umbral ({UMBRAL_MINIMO:.0%})")
    
    return {
        "version_agente": version_agente,
        "score_global": score_global,
        "tasa_acuerdo_global": tasa_acuerdo_global,
        "preguntas_que_pasan": preguntas_que_pasan,
        "total_preguntas": len(DATASET_REFERENCIA),
        "pasa_umbral_global": tasa_acuerdo_global >= UMBRAL_MINIMO,
        "detalles": resultados_evaluacion,
    }


if __name__ == "__main__":
    metricas = ejecutar_evaluacion_completa(version_agente="v1")
```

2. Ejecuta la evaluación de consistencia:

```bash
python consistency_eval.py
```

> ⏱️ **Nota de tiempo:** Este script realiza ~30 llamadas al LLM (3 respuestas × 10 preguntas + llamadas al juez). Puede tardar entre 5-8 minutos. Es normal.

#### Salida Esperada

```
============================================================
EVALUACIÓN DE CONSISTENCIA — Agente v1
============================================================

[01/10] ¿Cuál es la diferencia entre machine learning y de...
         ✅ Score vs referencia: 0.87 | Tasa acuerdo: 100%

[02/10] ¿Qué es RAG en el contexto de IA generativa?...
         ✅ Score vs referencia: 0.91 | Tasa acuerdo: 100%
...
[08/10] Si tengo 175 billones de parámetros en float32...
         ⚠️  Score vs referencia: 0.62 | Tasa acuerdo: 67%

============================================================
MÉTRICAS GLOBALES
============================================================
  Score promedio vs referencia: 0.83
  Tasa de acuerdo global:       89%
  Preguntas sobre umbral:        8/10

  ✅ Sistema CONSISTENTE (≥ 80%)
```

#### Verificación

En LangSmith, ve a tu proyecto y filtra por el tag `evaluation`. Debes ver la traza `evaluacion_consistencia_v1` con todas las sub-llamadas al agente y al juez. Los scores deben aparecer como metadatos de la traza.

---

### Paso 5: Crear el Agente v2 con Prompt Mejorado y Comparar

**Objetivo:** Implementar una segunda versión del agente con un prompt de sistema mejorado y comparar sus métricas contra la v1 para detectar regresiones o mejoras.

#### Instrucciones

1. Crea el archivo `agent_v2.py`:

```python
# agent_v2.py
"""
Agente ReAct v2 con prompt mejorado.
Mejoras respecto a v1:
- Prompt de sistema con instrucciones más específicas sobre uso de herramientas
- Instrucción explícita de usar calculadora para operaciones numéricas
- Indicación de citar la fuente (herramienta usada) en la respuesta final
"""
import os
from dotenv import load_dotenv
from langchain.agents import AgentExecutor, create_react_agent
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate

# Importar las mismas herramientas del agente v1
from agent_v1 import busqueda_web, calculadora, consulta_base_conocimiento

load_dotenv()

# ─────────────────────────────────────────────
# PROMPT MEJORADO (v2)
# Diferencias clave respecto al prompt estándar de Hub:
# 1. Instrucción explícita de prioridad de herramientas
# 2. Formato de respuesta final más estructurado
# 3. Instrucción de verificar cálculos numéricos
# ─────────────────────────────────────────────
PROMPT_MEJORADO_V2 = PromptTemplate.from_template("""Eres un asistente experto en inteligencia artificial y machine learning. 
Tienes acceso a herramientas especializadas que DEBES usar estratégicamente:

REGLAS DE USO DE HERRAMIENTAS:
1. Para preguntas sobre conceptos de IA (ML, DL, transformers, RAG, embeddings): usa `busqueda_web`
2. Para datos cuantitativos de modelos (parámetros, benchmarks, costos, fechas): usa `consulta_base_conocimiento`  
3. Para CUALQUIER cálculo numérico (multiplicaciones, divisiones, conversiones): usa `calculadora` SIEMPRE
4. Puedes combinar múltiples herramientas en una misma respuesta
5. Si una herramienta no devuelve datos útiles, intenta con otra

FORMATO DE RESPUESTA FINAL:
- Sé preciso y conciso
- Si usaste datos numéricos de una herramienta, muestra el cálculo
- No inventes datos; si no tienes la información, dilo claramente

Tienes acceso a las siguientes herramientas:
{tools}

Usa el siguiente formato:

Question: la pregunta de entrada que debes responder
Thought: siempre debes pensar qué hacer
Action: la acción a tomar, debe ser una de [{tool_names}]
Action Input: el input para la acción
Observation: el resultado de la acción
... (este ciclo Thought/Action/Action Input/Observation puede repetirse N veces)
Thought: Ahora sé la respuesta final
Final Answer: la respuesta final a la pregunta original

Begin!

Question: {input}
Thought:{agent_scratchpad}""")


def crear_agente_v2(verbose: bool = True) -> AgentExecutor:
    """
    Crea el agente ReAct v2 con prompt mejorado.
    
    Returns:
        AgentExecutor con prompt optimizado.
    """
    llm = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0,
        openai_api_key=os.getenv("OPENAI_API_KEY"),
    )
    
    herramientas = [busqueda_web, calculadora, consulta_base_conocimiento]
    
    agente = create_react_agent(llm, herramientas, PROMPT_MEJORADO_V2)
    
    ejecutor = AgentExecutor(
        agent=agente,
        tools=herramientas,
        verbose=verbose,
        max_iterations=6,  # +1 iteración respecto a v1 para cálculos complejos
        handle_parsing_errors=True,
        return_intermediate_steps=True,
    )
    
    return ejecutor


if __name__ == "__main__":
    print("Agente v2 cargado correctamente. Usa crear_agente_v2() para instanciarlo.")
```

2. Crea el script de comparación `comparar_versiones.py`:

```python
# comparar_versiones.py
"""
Compara el rendimiento de los agentes v1 y v2.
Ejecuta ambos sobre el mismo dataset y compara:
- Tasa de acuerdo (consistencia)
- Score promedio vs referencia
- Latencia promedio
- Número de pasos del agente
"""
import os
import time
import json
from dotenv import load_dotenv
from langsmith import traceable
from consistency_eval import (
    DATASET_REFERENCIA,
    crear_juez_semantico,
    calcular_tasa_acuerdo,
)
from agent_v1 import crear_agente_v1
from agent_v2 import crear_agente_v2

load_dotenv()

UMBRAL_MINIMO = 0.80
PREGUNTAS_COMPARACION = DATASET_REFERENCIA[:5]  # 5 preguntas para comparación rápida


@traceable(
    name="comparacion_v1_vs_v2",
    tags=["comparison", "regression-test"],
    metadata={"tipo": "A/B test agentes", "umbral": UMBRAL_MINIMO}
)
def comparar_agentes() -> dict:
    """
    Ejecuta ambos agentes sobre el mismo subconjunto del dataset
    y compara sus métricas de consistencia y calidad.
    """
    juez = crear_juez_semantico()
    
    resultados = {"v1": [], "v2": []}
    
    for version, crear_fn in [("v1", crear_agente_v1), ("v2", crear_agente_v2)]:
        agente = crear_fn(verbose=False)
        print(f"\n{'─'*50}")
        print(f"Evaluando Agente {version.upper()}...")
        print(f"{'─'*50}")
        
        for item in PREGUNTAS_COMPARACION:
            pregunta = item["pregunta"]
            respuesta_esperada = item["respuesta_esperada"]
            
            # Generar 2 respuestas para tasa de acuerdo
            respuestas = []
            latencias = []
            pasos_list = []
            
            for _ in range(2):
                inicio = time.time()
                try:
                    resultado = agente.invoke({"input": pregunta})
                    latencia = time.time() - inicio
                    respuestas.append(resultado["output"])
                    latencias.append(latencia)
                    pasos_list.append(len(resultado.get("intermediate_steps", [])))
                    time.sleep(0.3)
                except Exception as e:
                    respuestas.append(f"[ERROR: {e}]")
                    latencias.append(0)
                    pasos_list.append(0)
            
            # Evaluar contra referencia
            respuestas_validas = [r for r in respuestas if not r.startswith("[ERROR")]
            
            score_ref = 0.0
            if respuestas_validas:
                eval_r = juez(respuestas_validas[0], respuesta_esperada, pregunta)
                score_ref = eval_r["score"]
            
            tasa_acuerdo = calcular_tasa_acuerdo(respuestas_validas, juez)
            
            resultados[version].append({
                "pregunta": pregunta[:50] + "...",
                "score_referencia": score_ref,
                "tasa_acuerdo": tasa_acuerdo,
                "latencia_promedio": sum(latencias) / len(latencias) if latencias else 0,
                "pasos_promedio": sum(pasos_list) / len(pasos_list) if pasos_list else 0,
            })
            
            print(f"  [{version.upper()}] {pregunta[:45]}...")
            print(f"         Score: {score_ref:.2f} | Acuerdo: {tasa_acuerdo:.0%} | "
                  f"Latencia: {sum(latencias)/len(latencias):.1f}s | "
                  f"Pasos: {sum(pasos_list)/len(pasos_list):.1f}")
    
    # ─────────────────────────────────────────
    # REPORTE COMPARATIVO
    # ─────────────────────────────────────────
    def promedio(lista_dicts, campo):
        valores = [d[campo] for d in lista_dicts]
        return sum(valores) / len(valores) if valores else 0
    
    metricas_v1 = {
        "score_promedio": promedio(resultados["v1"], "score_referencia"),
        "tasa_acuerdo_promedio": promedio(resultados["v1"], "tasa_acuerdo"),
        "latencia_promedio": promedio(resultados["v1"], "latencia_promedio"),
        "pasos_promedio": promedio(resultados["v1"], "pasos_promedio"),
    }
    
    metricas_v2 = {
        "score_promedio": promedio(resultados["v2"], "score_referencia"),
        "tasa_acuerdo_promedio": promedio(resultados["v2"], "tasa_acuerdo"),
        "latencia_promedio": promedio(resultados["v2"], "latencia_promedio"),
        "pasos_promedio": promedio(resultados["v2"], "pasos_promedio"),
    }
    
    print(f"\n{'='*60}")
    print("REPORTE COMPARATIVO v1 vs v2")
    print(f"{'='*60}")
    print(f"{'Métrica':<35} {'v1':>8} {'v2':>8} {'Delta':>8}")
    print(f"{'─'*60}")
    
    for metrica in ["score_promedio", "tasa_acuerdo_promedio", "latencia_promedio", "pasos_promedio"]:
        val_v1 = metricas_v1[metrica]
        val_v2 = metricas_v2[metrica]
        delta = val_v2 - val_v1
        
        # Indicar si el delta es una mejora o regresión
        # Para score y tasa_acuerdo: delta positivo = mejora ✅
        # Para latencia y pasos: delta negativo = mejora ✅
        if metrica in ["score_promedio", "tasa_acuerdo_promedio"]:
            indicador = "✅" if delta >= 0 else "❌"
        else:
            indicador = "✅" if delta <= 0 else "⚠️ "
        
        print(f"{metrica:<35} {val_v1:>8.3f} {val_v2:>8.3f} {delta:>+7.3f} {indicador}")
    
    # Detección de regresión
    hay_regresion = (
        metricas_v2["score_promedio"] < metricas_v1["score_promedio"] - 0.05 or
        metricas_v2["tasa_acuerdo_promedio"] < metricas_v1["tasa_acuerdo_promedio"] - 0.05
    )
    
    print(f"\n{'─'*60}")
    if hay_regresion:
        print("❌ REGRESIÓN DETECTADA: v2 tiene peor calidad que v1 en >5%")
        print("   Acción recomendada: revisar cambios en el prompt de v2")
    else:
        print("✅ Sin regresión detectada: v2 mantiene o mejora la calidad de v1")
    
    return {
        "metricas_v1": metricas_v1,
        "metricas_v2": metricas_v2,
        "hay_regresion": hay_regresion,
        "detalles": resultados,
    }


if __name__ == "__main__":
    reporte = comparar_agentes()
    
    # Guardar reporte en JSON para uso en test_regression.py
    with open("reporte_comparacion.json", "w", encoding="utf-8") as f:
        # Serializar solo las métricas (sin detalles para brevedad)
        json.dump({
            "metricas_v1": reporte["metricas_v1"],
            "metricas_v2": reporte["metricas_v2"],
            "hay_regresion": reporte["hay_regresion"],
        }, f, indent=2, ensure_ascii=False)
    
    print(f"\n  📄 Reporte guardado en: reporte_comparacion.json")
```

3. Ejecuta la comparación:

```bash
python comparar_versiones.py
```

#### Salida Esperada

```
──────────────────────────────────────────────────
Evaluando Agente V1...
──────────────────────────────────────────────────
  [V1] ¿Cuál es la diferencia entre machine learni...
         Score: 0.88 | Acuerdo: 100% | Latencia: 3.2s | Pasos: 2.0
...

============================================================
REPORTE COMPARATIVO v1 vs v2
============================================================
Métrica                             v1       v2    Delta
────────────────────────────────────────────────────────────
score_promedio                   0.850    0.920   +0.070 ✅
tasa_acuerdo_promedio            0.880    0.920   +0.040 ✅
latencia_promedio                3.450    3.780   +0.330 ⚠️ 
pasos_promedio                   2.200    2.600   +0.400 ⚠️ 

──────────────────────────────────────────────────────────────
✅ Sin regresión detectada: v2 mantiene o mejora la calidad de v1
```

#### Verificación

En LangSmith, filtra las trazas por el tag `comparison`. Debes ver la traza `comparacion_v1_vs_v2` con todas las sub-ejecuciones de ambos agentes. Usa el panel de **Compare Runs** de LangSmith para visualizar side-by-side las diferencias de latencia entre v1 y v2.

---

### Paso 6: Pruebas de Regresión Automatizadas con Pytest

**Objetivo:** Crear pruebas automatizadas con pytest que validen que las métricas del agente no caen por debajo de umbrales definidos, usando el reporte generado en el Paso 5.

#### Instrucciones

1. Crea el archivo `test_regression.py`:

```python
# test_regression.py
"""
Pruebas de regresión automatizadas para el agente ReAct.
Valida que las métricas de calidad no caigan por debajo de umbrales.

Ejecutar con: pytest test_regression.py -v
"""
import json
import os
import pytest
from dotenv import load_dotenv

load_dotenv()

# ─────────────────────────────────────────────
# UMBRALES DE CALIDAD (definidos por el equipo)
# ─────────────────────────────────────────────
UMBRAL_SCORE_MINIMO = 0.70        # Score mínimo vs referencia
UMBRAL_TASA_ACUERDO_MINIMA = 0.75 # Tasa de acuerdo mínima
UMBRAL_LATENCIA_MAXIMA = 10.0     # Segundos máximos por pregunta
UMBRAL_MEJORA_V2_SCORE = -0.05    # v2 no puede ser >5% peor que v1


# ─────────────────────────────────────────────
# FIXTURE: Cargar reporte de comparación
# ─────────────────────────────────────────────
@pytest.fixture(scope="session")
def reporte():
    """Carga el reporte JSON generado por comparar_versiones.py."""
    ruta_reporte = "reporte_comparacion.json"
    if not os.path.exists(ruta_reporte):
        pytest.skip(
            f"Archivo '{ruta_reporte}' no encontrado. "
            "Ejecuta 'python comparar_versiones.py' primero."
        )
    with open(ruta_reporte, encoding="utf-8") as f:
        return json.load(f)


# ─────────────────────────────────────────────
# TESTS DE UMBRAL PARA AGENTE v1
# ─────────────────────────────────────────────
class TestAgentev1Umbrales:
    """Valida que el agente v1 cumple los umbrales mínimos de calidad."""
    
    def test_score_promedio_v1_sobre_umbral(self, reporte):
        """El score promedio de v1 debe ser >= UMBRAL_SCORE_MINIMO."""
        score = reporte["metricas_v1"]["score_promedio"]
        assert score >= UMBRAL_SCORE_MINIMO, (
            f"Score v1 ({score:.3f}) por debajo del umbral ({UMBRAL_SCORE_MINIMO}). "
            "Revisar calidad del agente base."
        )
    
    def test_tasa_acuerdo_v1_sobre_umbral(self, reporte):
        """La tasa de acuerdo de v1 debe ser >= UMBRAL_TASA_ACUERDO_MINIMA."""
        tasa = reporte["metricas_v1"]["tasa_acuerdo_promedio"]
        assert tasa >= UMBRAL_TASA_ACUERDO_MINIMA, (
            f"Tasa de acuerdo v1 ({tasa:.3f}) por debajo del umbral "
            f"({UMBRAL_TASA_ACUERDO_MINIMA}). "
            "El agente v1 es inconsistente."
        )
    
    def test_latencia_v1_bajo_maximo(self, reporte):
        """La latencia promedio de v1 debe ser < UMBRAL_LATENCIA_MAXIMA."""
        latencia = reporte["metricas_v1"]["latencia_promedio"]
        assert latencia < UMBRAL_LATENCIA_MAXIMA, (
            f"Latencia v1 ({latencia:.2f}s) supera el máximo "
            f"({UMBRAL_LATENCIA_MAXIMA}s). "
            "Posible cuello de botella en herramientas o LLM."
        )


# ─────────────────────────────────────────────
# TESTS DE REGRESIÓN v1 → v2
# ─────────────────────────────────────────────
class TestRegresionV2vsV1:
    """Valida que v2 no introduce regresiones respecto a v1."""
    
    def test_v2_no_regresa_score(self, reporte):
        """v2 no debe tener un score >5% inferior a v1."""
        delta_score = (
            reporte["metricas_v2"]["score_promedio"] -
            reporte["metricas_v1"]["score_promedio"]
        )
        assert delta_score >= UMBRAL_MEJORA_V2_SCORE, (
            f"REGRESIÓN DETECTADA: v2 score es {delta_score:.3f} respecto a v1 "
            f"(umbral mínimo: {UMBRAL_MEJORA_V2_SCORE}). "
            "El prompt mejorado empeoró la calidad."
        )
    
    def test_v2_no_regresa_consistencia(self, reporte):
        """v2 no debe tener una tasa de acuerdo >5% inferior a v1."""
        delta_acuerdo = (
            reporte["metricas_v2"]["tasa_acuerdo_promedio"] -
            reporte["metricas_v1"]["tasa_acuerdo_promedio"]
        )
        assert delta_acuerdo >= UMBRAL_MEJORA_V2_SCORE, (
            f"REGRESIÓN DE CONSISTENCIA: v2 tasa acuerdo es {delta_acuerdo:.3f} "
            f"respecto a v1 (umbral: {UMBRAL_MEJORA_V2_SCORE}). "
            "El prompt mejorado introdujo inconsistencia."
        )
    
    def test_sin_regresion_reportada(self, reporte):
        """El flag hay_regresion del reporte debe ser False."""
        assert not reporte["hay_regresion"], (
            "El reporte de comparación indica regresión. "
            "Revisar los cambios en el prompt de v2."
        )


# ─────────────────────────────────────────────
# TEST DE INTEGRACIÓN: Variables de entorno
# ─────────────────────────────────────────────
class TestConfiguracion:
    """Valida que el entorno está correctamente configurado."""
    
    def test_openai_api_key_configurada(self):
        """OPENAI_API_KEY debe estar configurada."""
        assert os.getenv("OPENAI_API_KEY"), (
            "OPENAI_API_KEY no está configurada. "
            "Verifica tu archivo .env"
        )
    
    def test_langchain_api_key_configurada(self):
        """LANGCHAIN_API_KEY debe estar configurada."""
        assert os.getenv("LANGCHAIN_API_KEY"), (
            "LANGCHAIN_API_KEY no está configurada. "
            "Verifica tu archivo .env"
        )
    
    def test_langchain_tracing_habilitado(self):
        """LANGCHAIN_TRACING_V2 debe estar en 'true'."""
        valor = os.getenv("LANGCHAIN_TRACING_V2", "false").lower()
        assert valor == "true", (
            f"LANGCHAIN_TRACING_V2='{valor}'. "
            "Debe ser 'true' para que LangSmith capture las trazas."
        )
    
    def test_proyecto_langsmith_configurado(self):
        """LANGCHAIN_PROJECT debe estar configurado."""
        proyecto = os.getenv("LANGCHAIN_PROJECT")
        assert proyecto, (
            "LANGCHAIN_PROJECT no está configurado. "
            "Verifica tu archivo .env"
        )
```

2. Ejecuta las pruebas de regresión:

```bash
pytest test_regression.py -v --tb=short
```

#### Salida Esperada

```
========================= test session starts ==========================
platform linux -- Python 3.11.x, pytest-8.2.2
collected 9 items

test_regression.py::TestConfiguracion::test_openai_api_key_configurada PASSED
test_regression.py::TestConfiguracion::test_langchain_api_key_configurada PASSED
test_regression.py::TestConfiguracion::test_langchain_tracing_habilitado PASSED
test_regression.py::TestConfiguracion::test_proyecto_langsmith_configurado PASSED
test_regression.py::TestAgentev1Umbrales::test_score_promedio_v1_sobre_umbral PASSED
test_regression.py::TestAgentev1Umbrales::test_tasa_acuerdo_v1_sobre_umbral PASSED
test_regression.py::TestAgentev1Umbrales::test_latencia_v1_bajo_maximo PASSED
test_regression.py::TestRegresionV2vsV1::test_v2_no_regresa_score PASSED
test_regression.py::TestRegresionV2vsV1::test_v2_no_regresa_consistencia PASSED
test_regression.py::TestRegresionV2vsV1::test_sin_regresion_reportada PASSED

========================== 9 passed in 0.42s ===========================
```

#### Verificación

Todos los tests deben pasar (`9 passed`). Si alguno falla, el mensaje de error indicará exactamente qué umbral fue violado y qué acción tomar.

---

## 7. Validación y Pruebas

### Lista de Verificación Final

Ejecuta esta secuencia de comandos para validar que todo el laboratorio está completo:

```bash
# 1. Verificar conexión con LangSmith
python verify_connection.py

# 2. Verificar que el agente v1 funciona
LANGCHAIN_TRACING_V2=false python -c "
from agent_v1 import crear_agente_v1
a = crear_agente_v1(verbose=False)
r = a.invoke({'input': '¿Qué es un embedding?'})
print('✅ Agente v1 OK:', r['output'][:80])
"

# 3. Verificar que el agente v2 funciona
python -c "
from agent_v2 import crear_agente_v2
a = crear_agente_v2(verbose=False)
r = a.invoke({'input': '¿Qué es un embedding?'})
print('✅ Agente v2 OK:', r['output'][:80])
"

# 4. Ejecutar pruebas de regresión (requiere reporte_comparacion.json)
pytest test_regression.py -v --tb=short

# 5. Verificar que las trazas aparecen en LangSmith
python -c "
import os
from dotenv import load_dotenv
from langsmith import Client
load_dotenv()
client = Client(api_key=os.getenv('LANGCHAIN_API_KEY'))
runs = list(client.list_runs(
    project_name=os.getenv('LANGCHAIN_PROJECT'),
    limit=5
))
print(f'✅ Trazas en LangSmith: {len(runs)} runs recientes')
for r in runs:
    print(f'   - {r.name} ({r.status})')
"
```

### Criterios de Éxito

| Criterio | Cómo verificar |
|----------|---------------|
| LangSmith conectado | `verify_connection.py` muestra ✅ |
| Trazas visibles en dashboard | ≥ 5 runs en proyecto `lab11-agente-react` |
| Agente usa las 3 herramientas | Revisar `intermediate_steps` en trazas |
| Dataset creado en LangSmith | Visible en sección "Datasets" del proyecto |
| Evaluación de consistencia ejecut
