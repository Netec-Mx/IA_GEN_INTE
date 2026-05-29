# Desarrollar un script en Python que evalúe y compare automáticamente el costo estimado de procesar un dataset específico entre OpenAI, Gemini y Anthropic

## 1. Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 45 minutos |
| **Complejidad** | Media |
| **Nivel Bloom** | Aplicar |
| **Lab ID** | 01-00-01 |
| **Módulo** | 1.1 — Modelos Comerciales vs. Open Source |

---

## 2. Descripción General

En este lab construirás un script Python modular que calcula y compara el **costo estimado** de procesar un dataset de 20 prompts contra seis modelos de tres proveedores líderes: OpenAI (GPT-4o, GPT-4o-mini), Google (Gemini 1.5 Pro, Gemini 1.5 Flash) y Anthropic (Claude 3.5 Sonnet, Claude 3 Haiku). Aplicarás la librería `tiktoken` para contar tokens en modelos OpenAI, la API de Gemini para conteo nativo y una estimación por caracteres para Claude. El script enviará una muestra de 5 prompts a cada API para medir latencia real, extrapolará el costo total para el dataset completo y generará un reporte tabular exportado a CSV. Este ejercicio traduce directamente los conceptos de la Lección 1.1 —donde aprendiste las diferencias entre modelos comerciales y sus trade-offs de costo, rendimiento y ventana de contexto— en una herramienta de análisis cuantitativo reutilizable.

---

## 3. Objetivos de Aprendizaje

- [ ] Implementar un script Python modular que calcule el costo estimado de procesar un dataset de prompts contra GPT-4o, GPT-4o-mini, Gemini 1.5 Pro, Gemini 1.5 Flash, Claude 3.5 Sonnet y Claude 3 Haiku.
- [ ] Aplicar `tiktoken` y métodos de conteo específicos de cada proveedor para estimar tokens de entrada y salida con precisión.
- [ ] Medir latencia promedio real enviando una muestra de 5 prompts a cada API y calcular el costo extrapolado para el dataset completo.
- [ ] Generar un reporte tabular comparativo en consola y exportarlo a `cost_comparison_report.csv` usando `pandas`.

---

## 4. Prerrequisitos

### Conocimientos

- Python intermedio: funciones, manejo de archivos, diccionarios, listas, f-strings.
- Haber completado el curso **IA GEN ESS** o equivalente.
- Comprensión básica del concepto de **tokenización** y **pago por token** (cubierto en Lección 1.1).

### Acceso y Credenciales

| Recurso | Estado requerido |
|---|---|
| Cuenta OpenAI Platform con crédito activo | ✅ Activa |
| Cuenta Google AI Studio con API habilitada | ✅ Activa |
| Cuenta Anthropic Console con crédito activo | ✅ Activa |
| Variables de entorno configuradas | `OPENAI_API_KEY`, `GOOGLE_API_KEY`, `ANTHROPIC_API_KEY` |

> ⚠️ **Control de costos:** Este lab consume créditos reales. La muestra de 5 prompts por proveedor (15 llamadas totales) tiene un costo estimado **inferior a $0.05 USD** con los modelos y prompts del dataset incluido. Configura límites de gasto en las consolas de cada proveedor antes de iniciar.

---

## 5. Entorno del Lab

### Hardware Mínimo

| Componente | Mínimo | Recomendado |
|---|---|---|
| RAM | 8 GB | 16 GB |
| Almacenamiento libre | 2 GB | 5 GB |
| Conexión a Internet | 10 Mbps | 25 Mbps |
| CPU | 4 núcleos | 8 núcleos |

### Software Requerido

| Paquete | Versión |
|---|---|
| Python | 3.11.x |
| openai | 1.35.x |
| google-generativeai | 0.7.x |
| anthropic | 0.28.x |
| tiktoken | 0.7.x |
| pandas | 2.2.x |
| python-dotenv | 1.0.x |

### Configuración del Entorno

**Paso de preparación — Crear entorno virtual e instalar dependencias:**

```bash
# 1. Crear directorio del lab
mkdir lab-01-00-01 && cd lab-01-00-01

# 2. Crear entorno virtual aislado
python -m venv .venv

# 3. Activar el entorno (Linux/macOS)
source .venv/bin/activate

# 3. Activar el entorno (Windows PowerShell)
# .venv\Scripts\Activate.ps1

# 4. Actualizar pip
pip install --upgrade pip

# 5. Instalar dependencias
pip install openai==1.35.3 google-generativeai==0.7.2 anthropic==0.28.0 \
            tiktoken==0.7.0 pandas==2.2.2 python-dotenv==1.0.1
```

**Crear el archivo `.env` con las API keys:**

```bash
# Crear archivo .env (NO lo subas a git)
cat > .env << 'EOF'
OPENAI_API_KEY=sk-...tu_clave_aqui...
GOOGLE_API_KEY=AIza...tu_clave_aqui...
ANTHROPIC_API_KEY=sk-ant-...tu_clave_aqui...
EOF
```

**Crear el archivo `.gitignore`:**

```bash
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
cost_comparison_report.csv
EOF
```

> 🔒 **Seguridad:** Verifica que `.env` aparezca en `.gitignore` antes de continuar. **Nunca** escribas API keys directamente en el código fuente.

---

## 6. Instrucciones Paso a Paso

---

### Paso 1 — Crear el Dataset de Prompts

**Objetivo:** Preparar el archivo JSON con los 20 prompts de complejidad variada que servirá como entrada del análisis.

#### Instrucciones

1. Dentro del directorio `lab-01-00-01`, crea el archivo `prompts_dataset.json`:

```bash
cat > prompts_dataset.json << 'EOF'
{
  "dataset_name": "cost_analysis_dataset_v1",
  "description": "20 prompts de complejidad variada para análisis de costo entre proveedores",
  "prompts": [
    {"id": 1, "category": "corto", "text": "¿Cuál es la capital de Francia?"},
    {"id": 2, "category": "corto", "text": "Traduce 'hello world' al español."},
    {"id": 3, "category": "corto", "text": "¿Cuánto es 15% de 200?"},
    {"id": 4, "category": "corto", "text": "Dame un sinónimo de 'rápido'."},
    {"id": 5, "category": "corto", "text": "¿En qué año se fundó OpenAI?"},
    {"id": 6, "category": "mediano", "text": "Explica en 3 oraciones qué es el machine learning y cómo se diferencia del deep learning."},
    {"id": 7, "category": "mediano", "text": "Escribe un correo profesional de 100 palabras solicitando una reunión para revisar el avance de un proyecto de software."},
    {"id": 8, "category": "mediano", "text": "Describe las ventajas y desventajas de usar microservicios versus una arquitectura monolítica en una startup de tecnología."},
    {"id": 9, "category": "mediano", "text": "Genera 5 ideas de nombres para una aplicación móvil de gestión de tareas orientada a desarrolladores de software."},
    {"id": 10, "category": "mediano", "text": "Explica el concepto de tokenización en modelos de lenguaje grande y por qué es relevante para el costo de uso de las APIs."},
    {"id": 11, "category": "mediano", "text": "¿Cuáles son las mejores prácticas para manejar errores en una API REST construida con FastAPI? Menciona al menos 4 prácticas concretas."},
    {"id": 12, "category": "mediano", "text": "Compara brevemente los modelos GPT-4o, Gemini 1.5 Pro y Claude 3.5 Sonnet en términos de ventana de contexto y casos de uso recomendados."},
    {"id": 13, "category": "largo", "text": "Actúa como arquitecto de soluciones de IA. Un cliente del sector salud quiere implementar un chatbot para responder preguntas frecuentes de pacientes sobre sus citas médicas. Los datos son altamente sensibles. Describe una arquitectura completa que incluya: selección de modelo (justificando si usar comercial u open source), estrategia de almacenamiento de datos, medidas de seguridad y privacidad, estimación de costos mensuales asumiendo 10,000 consultas diarias, y consideraciones de cumplimiento normativo (HIPAA o equivalente latinoamericano)."},
    {"id": 14, "category": "largo", "text": "Escribe un tutorial paso a paso de 400 palabras explicando cómo implementar Retrieval-Augmented Generation (RAG) con Python, LangChain y una base de datos vectorial. Incluye fragmentos de código comentados para cada etapa: carga de documentos, generación de embeddings, almacenamiento en vector store, recuperación de contexto relevante y generación de respuesta final."},
    {"id": 15, "category": "largo", "text": "Analiza el siguiente escenario de negocio y proporciona un plan de implementación detallado: Una empresa de e-commerce con 500,000 productos quiere implementar un sistema de recomendaciones personalizadas usando IA Generativa. El sistema debe procesar el historial de compras de cada usuario, generar descripciones personalizadas de productos recomendados y responder preguntas en lenguaje natural sobre los productos. Incluye: arquitectura técnica, stack tecnológico recomendado, fases de implementación con estimaciones de tiempo, KPIs para medir el éxito y riesgos principales."},
    {"id": 16, "category": "largo", "text": "Genera un documento de especificación técnica completo para una API REST de análisis de sentimientos. El documento debe incluir: descripción del servicio, endpoints con métodos HTTP, parámetros de request y response en formato JSON, códigos de error posibles, ejemplos de uso con curl, consideraciones de autenticación con JWT, límites de rate limiting y SLA de disponibilidad del 99.9%. Usa formato estructurado con secciones claramente definidas."},
    {"id": 17, "category": "largo", "text": "Como experto en MLOps, describe el pipeline completo para llevar un modelo de clasificación de texto desde el experimento local hasta producción en Kubernetes. Incluye: gestión de experimentos con MLflow, empaquetado del modelo con Docker, CI/CD con GitHub Actions, despliegue en Kubernetes con estrategia de rolling update, monitoreo de drift con Evidently AI, y procedimiento de rollback ante degradación de métricas. Proporciona fragmentos de código o configuración YAML donde sea relevante."},
    {"id": 18, "category": "largo", "text": "Escribe un análisis comparativo exhaustivo de las estrategias de fine-tuning disponibles para modelos de lenguaje grande: full fine-tuning, LoRA, QLoRA, y prompt tuning. Para cada estrategia explica: fundamento técnico, requisitos de hardware, tamaño de dataset recomendado, costo estimado en GPU-horas, casos de uso óptimos y limitaciones. Concluye con una matriz de decisión que ayude a elegir la estrategia correcta según el caso de uso."},
    {"id": 19, "category": "largo", "text": "Desarrolla un plan de seguridad completo para una aplicación de IA Generativa expuesta públicamente. El plan debe cubrir: defensa contra prompt injection (con ejemplos de ataques y contramedidas), gestión segura de API keys y secretos con HashiCorp Vault, validación y sanitización de inputs con Pydantic, límites de tokens por usuario para prevenir abusos, logging y auditoría de todas las interacciones, y procedimiento de respuesta ante incidentes de seguridad. Incluye código Python de ejemplo para las contramedidas más críticas."},
    {"id": 20, "category": "largo", "text": "Como consultor de transformación digital, elabora un roadmap de 12 meses para que una empresa de servicios financieros tradicional adopte IA Generativa de forma responsable. El roadmap debe incluir: fase de evaluación y casos de uso prioritarios (meses 1-2), fase de pilotos controlados con métricas de éxito (meses 3-5), fase de escalamiento con gobierno de datos (meses 6-9), y fase de optimización y expansión (meses 10-12). Para cada fase especifica entregables concretos, roles involucrados, presupuesto aproximado y criterios de go/no-go para avanzar a la siguiente fase."}
  ]
}
EOF
```

2. Verifica que el archivo se creó correctamente:

```bash
python -c "import json; data=json.load(open('prompts_dataset.json')); print(f'Dataset cargado: {len(data[\"prompts\"])} prompts')"
```

#### Salida Esperada

```
Dataset cargado: 20 prompts
```

#### Verificación

Confirma que el archivo `prompts_dataset.json` existe y tiene exactamente 20 prompts con categorías `corto`, `mediano` y `largo`.

---

### Paso 2 — Crear el Módulo de Configuración de Precios

**Objetivo:** Definir el diccionario de precios por token de cada modelo, incluyendo las métricas de ventana de contexto para el reporte comparativo.

#### Instrucciones

1. Crea el archivo `config.py`:

```python
# config.py
# Módulo de configuración: precios por token y metadatos de modelos
# Precios en USD por 1,000,000 de tokens (actualizado a junio 2025)
# Fuentes: platform.openai.com/docs/pricing, ai.google.dev, anthropic.com/pricing

MODEL_CONFIG = {
    # ── OpenAI ──────────────────────────────────────────────────────────────
    "gpt-4o": {
        "provider": "OpenAI",
        "display_name": "GPT-4o",
        "input_price_per_1m": 5.00,       # USD por 1M tokens de input
        "output_price_per_1m": 15.00,     # USD por 1M tokens de output
        "context_window_tokens": 128_000,
        "tokenizer": "tiktoken",
        "tiktoken_encoding": "cl100k_base",
    },
    "gpt-4o-mini": {
        "provider": "OpenAI",
        "display_name": "GPT-4o-mini",
        "input_price_per_1m": 0.15,
        "output_price_per_1m": 0.60,
        "context_window_tokens": 128_000,
        "tokenizer": "tiktoken",
        "tiktoken_encoding": "cl100k_base",
    },
    # ── Google ───────────────────────────────────────────────────────────────
    "gemini-1.5-pro": {
        "provider": "Google",
        "display_name": "Gemini 1.5 Pro",
        "input_price_per_1m": 3.50,
        "output_price_per_1m": 10.50,
        "context_window_tokens": 1_000_000,
        "tokenizer": "gemini_api",
    },
    "gemini-1.5-flash": {
        "provider": "Google",
        "display_name": "Gemini 1.5 Flash",
        "input_price_per_1m": 0.075,
        "output_price_per_1m": 0.30,
        "context_window_tokens": 1_000_000,
        "tokenizer": "gemini_api",
    },
    # ── Anthropic ────────────────────────────────────────────────────────────
    "claude-3-5-sonnet-20240620": {
        "provider": "Anthropic",
        "display_name": "Claude 3.5 Sonnet",
        "input_price_per_1m": 3.00,
        "output_price_per_1m": 15.00,
        "context_window_tokens": 200_000,
        "tokenizer": "char_estimate",
        "chars_per_token": 4.0,           # Estimación: ~4 caracteres por token
    },
    "claude-3-haiku-20240307": {
        "provider": "Anthropic",
        "display_name": "Claude 3 Haiku",
        "input_price_per_1m": 0.25,
        "output_price_per_1m": 1.25,
        "context_window_tokens": 200_000,
        "tokenizer": "char_estimate",
        "chars_per_token": 4.0,
    },
}

# Número de prompts en el dataset completo
DATASET_SIZE = 20

# Número de prompts en la muestra para llamadas reales a la API
SAMPLE_SIZE = 5

# Índices de los prompts seleccionados para la muestra (0-based)
# Se seleccionan prompts corto, mediano y largo para representatividad
SAMPLE_INDICES = [0, 5, 9, 13, 18]  # IDs 1, 6, 10, 14, 19

# Nombre del archivo de reporte de salida
OUTPUT_CSV = "cost_comparison_report.csv"
```

#### Salida Esperada

No hay salida en este paso. El archivo `config.py` se crea sin errores.

#### Verificación

```bash
python -c "from config import MODEL_CONFIG; print(f'Modelos configurados: {list(MODEL_CONFIG.keys())}')"
```

Salida esperada:
```
Modelos configurados: ['gpt-4o', 'gpt-4o-mini', 'gemini-1.5-pro', 'gemini-1.5-flash', 'claude-3-5-sonnet-20240620', 'claude-3-haiku-20240307']
```

---

### Paso 3 — Crear el Módulo de Tokenización

**Objetivo:** Implementar las tres estrategias de conteo de tokens: `tiktoken` para OpenAI, API de Gemini para Google y estimación por caracteres para Anthropic.

#### Instrucciones

1. Crea el archivo `tokenizer.py`:

```python
# tokenizer.py
# Módulo de tokenización: cuenta tokens de input según el proveedor

import tiktoken
import google.generativeai as genai
from config import MODEL_CONFIG


def count_tokens_tiktoken(text: str, encoding_name: str = "cl100k_base") -> int:
    """
    Cuenta tokens usando tiktoken (para modelos OpenAI).
    
    Args:
        text: Texto a tokenizar.
        encoding_name: Nombre del encoding de tiktoken.
    
    Returns:
        Número de tokens.
    """
    encoding = tiktoken.get_encoding(encoding_name)
    return len(encoding.encode(text))


def count_tokens_gemini(text: str, model_name: str = "gemini-1.5-flash") -> int:
    """
    Cuenta tokens usando la API de Gemini (conteo nativo).
    
    Args:
        text: Texto a tokenizar.
        model_name: Nombre del modelo Gemini para el conteo.
    
    Returns:
        Número de tokens según el tokenizador de Gemini.
    """
    model = genai.GenerativeModel(model_name)
    result = model.count_tokens(text)
    return result.total_tokens


def count_tokens_char_estimate(text: str, chars_per_token: float = 4.0) -> int:
    """
    Estima tokens para Claude basándose en el número de caracteres.
    
    Anthropic no expone un tokenizador público standalone. La estimación
    de ~4 caracteres por token es una aproximación conservadora documentada
    en la guía oficial de Anthropic.
    
    Args:
        text: Texto a estimar.
        chars_per_token: Ratio de caracteres por token.
    
    Returns:
        Estimación del número de tokens.
    """
    return max(1, int(len(text) / chars_per_token))


def count_input_tokens(text: str, model_key: str) -> int:
    """
    Dispatcher que selecciona el método de tokenización correcto según el modelo.
    
    Args:
        text: Texto del prompt de entrada.
        model_key: Clave del modelo en MODEL_CONFIG.
    
    Returns:
        Número (o estimación) de tokens de input.
    """
    config = MODEL_CONFIG[model_key]
    tokenizer_type = config["tokenizer"]

    if tokenizer_type == "tiktoken":
        return count_tokens_tiktoken(text, config["tiktoken_encoding"])
    elif tokenizer_type == "gemini_api":
        # Usamos flash para el conteo (más económico) ya que el tokenizador
        # es compartido entre variantes de Gemini 1.5
        return count_tokens_gemini(text, model_name="gemini-1.5-flash")
    elif tokenizer_type == "char_estimate":
        return count_tokens_char_estimate(text, config["chars_per_token"])
    else:
        raise ValueError(f"Tipo de tokenizador desconocido: {tokenizer_type}")
```

#### Salida Esperada

No hay salida directa. El módulo se importará en pasos posteriores.

#### Verificación

```bash
python -c "
from tokenizer import count_tokens_tiktoken, count_tokens_char_estimate
texto = 'Hola, esto es una prueba de tokenización.'
print(f'tiktoken: {count_tokens_tiktoken(texto)} tokens')
print(f'char_estimate: {count_tokens_char_estimate(texto)} tokens')
"
```

Salida esperada (valores aproximados):
```
tiktoken: 12 tokens
char_estimate: 10 tokens
```

---

### Paso 4 — Crear el Módulo de Ejecución con Medición de Latencia

**Objetivo:** Implementar las funciones que envían prompts reales a cada API, miden la latencia con `time.perf_counter()` y retornan el conteo de tokens de output.

#### Instrucciones

1. Crea el archivo `api_executor.py`:

```python
# api_executor.py
# Módulo de ejecución: envía prompts a las APIs y mide latencia real

import time
import os
from dataclasses import dataclass

from openai import OpenAI
import google.generativeai as genai
import anthropic
from dotenv import load_dotenv

load_dotenv()

# ── Inicialización de clientes ────────────────────────────────────────────────
openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
genai.configure(api_key=os.getenv("GOOGLE_API_KEY"))
anthropic_client = anthropic.Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))


@dataclass
class APICallResult:
    """Resultado de una llamada individual a la API."""
    model_key: str
    prompt_id: int
    input_tokens: int
    output_tokens: int
    latency_seconds: float
    success: bool
    error_message: str = ""


def call_openai(prompt: str, model_key: str, prompt_id: int) -> APICallResult:
    """
    Envía un prompt a la API de OpenAI y mide la latencia.
    
    Args:
        prompt: Texto del prompt.
        model_key: Clave del modelo (ej. 'gpt-4o-mini').
        prompt_id: ID del prompt en el dataset.
    
    Returns:
        APICallResult con métricas de la llamada.
    """
    start = time.perf_counter()
    try:
        response = openai_client.chat.completions.create(
            model=model_key,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=512,   # Limitar output para controlar costos del lab
        )
        latency = time.perf_counter() - start
        
        return APICallResult(
            model_key=model_key,
            prompt_id=prompt_id,
            input_tokens=response.usage.prompt_tokens,
            output_tokens=response.usage.completion_tokens,
            latency_seconds=round(latency, 3),
            success=True,
        )
    except Exception as e:
        latency = time.perf_counter() - start
        return APICallResult(
            model_key=model_key,
            prompt_id=prompt_id,
            input_tokens=0,
            output_tokens=0,
            latency_seconds=round(latency, 3),
            success=False,
            error_message=str(e),
        )


def call_gemini(prompt: str, model_key: str, prompt_id: int) -> APICallResult:
    """
    Envía un prompt a la API de Gemini y mide la latencia.
    
    Args:
        prompt: Texto del prompt.
        model_key: Clave del modelo (ej. 'gemini-1.5-flash').
        prompt_id: ID del prompt en el dataset.
    
    Returns:
        APICallResult con métricas de la llamada.
    """
    start = time.perf_counter()
    try:
        model = genai.GenerativeModel(
            model_key,
            generation_config=genai.types.GenerationConfig(max_output_tokens=512),
        )
        response = model.generate_content(prompt)
        latency = time.perf_counter() - start

        # Gemini retorna conteo de tokens en usage_metadata
        input_tokens = response.usage_metadata.prompt_token_count
        output_tokens = response.usage_metadata.candidates_token_count

        return APICallResult(
            model_key=model_key,
            prompt_id=prompt_id,
            input_tokens=input_tokens,
            output_tokens=output_tokens,
            latency_seconds=round(latency, 3),
            success=True,
        )
    except Exception as e:
        latency = time.perf_counter() - start
        return APICallResult(
            model_key=model_key,
            prompt_id=prompt_id,
            input_tokens=0,
            output_tokens=0,
            latency_seconds=round(latency, 3),
            success=False,
            error_message=str(e),
        )


def call_anthropic(prompt: str, model_key: str, prompt_id: int) -> APICallResult:
    """
    Envía un prompt a la API de Anthropic y mide la latencia.
    
    Args:
        prompt: Texto del prompt.
        model_key: Clave del modelo (ej. 'claude-3-haiku-20240307').
        prompt_id: ID del prompt en el dataset.
    
    Returns:
        APICallResult con métricas de la llamada.
    """
    start = time.perf_counter()
    try:
        response = anthropic_client.messages.create(
            model=model_key,
            max_tokens=512,
            messages=[{"role": "user", "content": prompt}],
        )
        latency = time.perf_counter() - start

        return APICallResult(
            model_key=model_key,
            prompt_id=prompt_id,
            input_tokens=response.usage.input_tokens,
            output_tokens=response.usage.output_tokens,
            latency_seconds=round(latency, 3),
            success=True,
        )
    except Exception as e:
        latency = time.perf_counter() - start
        return APICallResult(
            model_key=model_key,
            prompt_id=prompt_id,
            input_tokens=0,
            output_tokens=0,
            latency_seconds=round(latency, 3),
            success=False,
            error_message=str(e),
        )


# Mapa de funciones de llamada por proveedor
PROVIDER_CALL_MAP = {
    "OpenAI": call_openai,
    "Google": call_gemini,
    "Anthropic": call_anthropic,
}
```

#### Salida Esperada

No hay salida directa. El módulo se importará en el paso siguiente.

#### Verificación

```bash
python -c "from api_executor import PROVIDER_CALL_MAP; print('Proveedores disponibles:', list(PROVIDER_CALL_MAP.keys()))"
```

Salida esperada:
```
Proveedores disponibles: ['OpenAI', 'Google', 'Anthropic']
```

---

### Paso 5 — Crear el Módulo de Análisis de Costos

**Objetivo:** Implementar la lógica que calcula el costo estimado por modelo usando los resultados de la muestra y extrapola al dataset completo.

#### Instrucciones

1. Crea el archivo `cost_analyzer.py`:

```python
# cost_analyzer.py
# Módulo de análisis: calcula costo estimado y extrapola al dataset completo

from dataclasses import dataclass, field
from typing import List
from api_executor import APICallResult
from config import MODEL_CONFIG, DATASET_SIZE, SAMPLE_SIZE


@dataclass
class ModelCostSummary:
    """Resumen de costos y métricas para un modelo específico."""
    model_key: str
    display_name: str
    provider: str
    context_window_tokens: int
    
    # Métricas de la muestra
    sample_calls: int = 0
    sample_input_tokens: int = 0
    sample_output_tokens: int = 0
    sample_cost_usd: float = 0.0
    avg_latency_seconds: float = 0.0
    successful_calls: int = 0
    
    # Métricas extrapoladas al dataset completo
    estimated_total_input_tokens: int = 0
    estimated_total_output_tokens: int = 0
    estimated_total_cost_usd: float = 0.0
    
    # Ratio de tokens de output vs input (para extrapolación)
    avg_output_input_ratio: float = 0.0


def calculate_cost(input_tokens: int, output_tokens: int, model_key: str) -> float:
    """
    Calcula el costo en USD para una cantidad de tokens dada.
    
    Args:
        input_tokens: Número de tokens de entrada.
        output_tokens: Número de tokens de salida.
        model_key: Clave del modelo en MODEL_CONFIG.
    
    Returns:
        Costo en USD.
    """
    config = MODEL_CONFIG[model_key]
    input_cost = (input_tokens / 1_000_000) * config["input_price_per_1m"]
    output_cost = (output_tokens / 1_000_000) * config["output_price_per_1m"]
    return round(input_cost + output_cost, 8)


def analyze_model_results(
    model_key: str,
    results: List[APICallResult],
    all_input_tokens: List[int],
) -> ModelCostSummary:
    """
    Analiza los resultados de la muestra y extrapola al dataset completo.
    
    Args:
        model_key: Clave del modelo.
        results: Lista de APICallResult de la muestra.
        all_input_tokens: Lista con el conteo de tokens de input de TODOS
                          los 20 prompts del dataset (calculado sin llamadas a API).
    
    Returns:
        ModelCostSummary con todas las métricas calculadas.
    """
    config = MODEL_CONFIG[model_key]
    successful = [r for r in results if r.success]

    summary = ModelCostSummary(
        model_key=model_key,
        display_name=config["display_name"],
        provider=config["provider"],
        context_window_tokens=config["context_window_tokens"],
        sample_calls=len(results),
        successful_calls=len(successful),
    )

    if not successful:
        return summary

    # ── Métricas de la muestra ────────────────────────────────────────────────
    summary.sample_input_tokens = sum(r.input_tokens for r in successful)
    summary.sample_output_tokens = sum(r.output_tokens for r in successful)
    summary.sample_cost_usd = calculate_cost(
        summary.sample_input_tokens,
        summary.sample_output_tokens,
        model_key,
    )
    summary.avg_latency_seconds = round(
        sum(r.latency_seconds for r in successful) / len(successful), 3
    )
    
    # Ratio output/input para extrapolación
    if summary.sample_input_tokens > 0:
        summary.avg_output_input_ratio = round(
            summary.sample_output_tokens / summary.sample_input_tokens, 4
        )

    # ── Extrapolación al dataset completo ────────────────────────────────────
    # Usamos el conteo real de tokens de todos los prompts para el input
    # y el ratio observado de output/input para estimar el output
    summary.estimated_total_input_tokens = sum(all_input_tokens)
    summary.estimated_total_output_tokens = int(
        summary.estimated_total_input_tokens * summary.avg_output_input_ratio
    )
    summary.estimated_total_cost_usd = calculate_cost(
        summary.estimated_total_input_tokens,
        summary.estimated_total_output_tokens,
        model_key,
    )

    return summary
```

#### Salida Esperada

No hay salida directa. El módulo se importará en el script principal.

#### Verificación

```bash
python -c "from cost_analyzer import calculate_cost; print(f'Costo 1M tokens GPT-4o: \${calculate_cost(500000, 500000, \"gpt-4o\"):.4f} USD')"
```

Salida esperada:
```
Costo 1M tokens GPT-4o: $10.0000 USD
```

---

### Paso 6 — Crear el Módulo de Reporte

**Objetivo:** Implementar la función que genera la tabla comparativa en consola y exporta los resultados a CSV usando `pandas`.

#### Instrucciones

1. Crea el archivo `reporter.py`:

```python
# reporter.py
# Módulo de reporte: genera tabla comparativa y exporta a CSV

import pandas as pd
from typing import List
from cost_analyzer import ModelCostSummary
from config import OUTPUT_CSV


def generate_report(summaries: List[ModelCostSummary]) -> pd.DataFrame:
    """
    Genera un DataFrame con los resultados comparativos de todos los modelos.
    
    Args:
        summaries: Lista de ModelCostSummary con los resultados de cada modelo.
    
    Returns:
        DataFrame de pandas con el reporte completo.
    """
    rows = []
    for s in summaries:
        rows.append({
            "Proveedor": s.provider,
            "Modelo": s.display_name,
            "Ventana Contexto (tokens)": f"{s.context_window_tokens:,}",
            "Latencia Prom. (s)": f"{s.avg_latency_seconds:.3f}",
            "Tokens Input (muestra)": s.sample_input_tokens,
            "Tokens Output (muestra)": s.sample_output_tokens,
            "Costo Muestra (USD)": f"${s.sample_cost_usd:.6f}",
            "Tokens Input (dataset)": s.estimated_total_input_tokens,
            "Tokens Output (dataset)": s.estimated_total_output_tokens,
            "Costo Dataset (USD)": f"${s.estimated_total_cost_usd:.6f}",
            "Llamadas Exitosas": f"{s.successful_calls}/{s.sample_calls}",
        })

    df = pd.DataFrame(rows)
    return df


def print_console_report(df: pd.DataFrame) -> None:
    """
    Imprime el reporte formateado en la consola.
    
    Args:
        df: DataFrame con los resultados del análisis.
    """
    print("\n" + "═" * 100)
    print("  REPORTE COMPARATIVO DE COSTOS — ANÁLISIS MULTI-PROVEEDOR")
    print("═" * 100)
    
    # Configurar pandas para mostrar todas las columnas
    pd.set_option("display.max_columns", None)
    pd.set_option("display.width", 120)
    pd.set_option("display.max_colwidth", 25)
    
    print(df.to_string(index=False))
    print("═" * 100)
    
    # Identificar el modelo más económico para el dataset completo
    # Extraer valores numéricos para comparación
    df_numeric = df.copy()
    df_numeric["_costo_num"] = df["Costo Dataset (USD)"].str.replace("$", "").astype(float)
    
    min_idx = df_numeric["_costo_num"].idxmin()
    max_idx = df_numeric["_costo_num"].idxmax()
    
    print(f"\n  ✅ Modelo más económico (dataset completo): "
          f"{df.loc[min_idx, 'Modelo']} ({df.loc[min_idx, 'Costo Dataset (USD)']})")
    print(f"  💰 Modelo más costoso (dataset completo):   "
          f"{df.loc[max_idx, 'Modelo']} ({df.loc[max_idx, 'Costo Dataset (USD)']})")
    
    # Modelo con menor latencia
    df_numeric["_latencia_num"] = df["Latencia Prom. (s)"].astype(float)
    min_lat_idx = df_numeric["_latencia_num"].idxmin()
    print(f"  ⚡ Modelo con menor latencia:               "
          f"{df.loc[min_lat_idx, 'Modelo']} ({df.loc[min_lat_idx, 'Latencia Prom. (s)']}s)")
    print("═" * 100 + "\n")


def export_to_csv(df: pd.DataFrame, filepath: str = OUTPUT_CSV) -> None:
    """
    Exporta el DataFrame a un archivo CSV.
    
    Args:
        df: DataFrame con los resultados.
        filepath: Ruta del archivo CSV de salida.
    """
    df.to_csv(filepath, index=False, encoding="utf-8-sig")
    print(f"  📄 Reporte exportado exitosamente: {filepath}")
```

#### Salida Esperada

No hay salida directa en este paso.

#### Verificación

```bash
python -c "from reporter import generate_report, print_console_report; print('Módulo reporter importado correctamente')"
```

---

### Paso 7 — Crear el Script Principal de Orquestación

**Objetivo:** Integrar todos los módulos en un script principal (`main.py`) que ejecuta el pipeline completo: carga del dataset → tokenización → llamadas a APIs → análisis → reporte.

#### Instrucciones

1. Crea el archivo `main.py`:

```python
# main.py
# Script principal: orquesta el pipeline completo de análisis de costos

import json
import os
import sys
from dotenv import load_dotenv

import google.generativeai as genai

from config import MODEL_CONFIG, SAMPLE_INDICES, SAMPLE_SIZE, DATASET_SIZE
from tokenizer import count_input_tokens
from api_executor import PROVIDER_CALL_MAP, APICallResult
from cost_analyzer import analyze_model_results, ModelCostSummary
from reporter import generate_report, print_console_report, export_to_csv


def load_dataset(filepath: str = "prompts_dataset.json") -> list:
    """Carga el dataset de prompts desde el archivo JSON."""
    with open(filepath, "r", encoding="utf-8") as f:
        data = json.load(f)
    return data["prompts"]


def validate_environment() -> bool:
    """
    Verifica que todas las variables de entorno requeridas estén configuradas.
    
    Returns:
        True si el entorno es válido, False en caso contrario.
    """
    required_keys = ["OPENAI_API_KEY", "GOOGLE_API_KEY", "ANTHROPIC_API_KEY"]
    missing = [k for k in required_keys if not os.getenv(k)]
    
    if missing:
        print(f"❌ ERROR: Faltan las siguientes variables de entorno: {missing}")
        print("   Asegúrate de que el archivo .env esté configurado correctamente.")
        return False
    
    print("✅ Variables de entorno verificadas correctamente.")
    return True


def tokenize_full_dataset(prompts: list) -> dict:
    """
    Cuenta los tokens de input para TODOS los prompts del dataset,
    sin realizar llamadas costosas a las APIs de generación.
    
    Args:
        prompts: Lista completa de prompts del dataset.
    
    Returns:
        Diccionario {model_key: [lista de conteos de tokens por prompt]}.
    """
    print("\n📊 Tokenizando dataset completo (sin llamadas a API de generación)...")
    token_counts = {model_key: [] for model_key in MODEL_CONFIG}

    for i, prompt_data in enumerate(prompts):
        text = prompt_data["text"]
        for model_key in MODEL_CONFIG:
            count = count_input_tokens(text, model_key)
            token_counts[model_key].append(count)
        
        if (i + 1) % 5 == 0:
            print(f"   Procesados {i + 1}/{len(prompts)} prompts...")

    print(f"   ✅ Tokenización completa: {len(prompts)} prompts × {len(MODEL_CONFIG)} modelos")
    return token_counts


def run_api_sample(prompts: list) -> dict:
    """
    Envía la muestra de 5 prompts a cada API y mide la latencia.
    
    Args:
        prompts: Lista completa de prompts del dataset.
    
    Returns:
        Diccionario {model_key: [lista de APICallResult]}.
    """
    sample_prompts = [prompts[i] for i in SAMPLE_INDICES]
    results = {model_key: [] for model_key in MODEL_CONFIG}

    print(f"\n🚀 Ejecutando muestra de {SAMPLE_SIZE} prompts en cada API...")
    print(f"   Prompts seleccionados: IDs {[p['id'] for p in sample_prompts]}")
    print(f"   Total de llamadas a API: {SAMPLE_SIZE * len(MODEL_CONFIG)}\n")

    for model_key, config in MODEL_CONFIG.items():
        provider = config["provider"]
        call_fn = PROVIDER_CALL_MAP[provider]
        
        print(f"   ▶ Procesando {config['display_name']} ({provider})...")
        
        for prompt_data in sample_prompts:
            result = call_fn(
                prompt=prompt_data["text"],
                model_key=model_key,
                prompt_id=prompt_data["id"],
            )
            results[model_key].append(result)
            
            status = "✓" if result.success else "✗"
            print(f"     {status} Prompt ID {result.prompt_id}: "
                  f"{result.input_tokens} in / {result.output_tokens} out / "
                  f"{result.latency_seconds:.3f}s"
                  + (f" [ERROR: {result.error_message[:50]}]" if not result.success else ""))

    return results


def main():
    """Función principal que orquesta el pipeline completo."""
    print("\n" + "═" * 60)
    print("  LAB 01-00-01: Análisis Comparativo de Costos LLM")
    print("═" * 60)

    # ── 1. Cargar variables de entorno ────────────────────────────────────────
    load_dotenv()
    
    # ── 2. Validar entorno ────────────────────────────────────────────────────
    if not validate_environment():
        sys.exit(1)
    
    # Configurar cliente de Gemini con la API key cargada
    genai.configure(api_key=os.getenv("GOOGLE_API_KEY"))

    # ── 3. Cargar dataset ─────────────────────────────────────────────────────
    print("\n📂 Cargando dataset de prompts...")
    prompts = load_dataset()
    print(f"   ✅ Dataset cargado: {len(prompts)} prompts")
    
    # Mostrar distribución por categoría
    categories = {}
    for p in prompts:
        categories[p["category"]] = categories.get(p["category"], 0) + 1
    print(f"   Distribución: {categories}")

    # ── 4. Tokenizar dataset completo (sin llamadas de generación) ────────────
    token_counts = tokenize_full_dataset(prompts)

    # ── 5. Ejecutar muestra en APIs reales ────────────────────────────────────
    api_results = run_api_sample(prompts)

    # ── 6. Analizar resultados y calcular costos ──────────────────────────────
    print("\n📈 Calculando costos y extrapolando al dataset completo...")
    summaries = []
    
    for model_key in MODEL_CONFIG:
        summary = analyze_model_results(
            model_key=model_key,
            results=api_results[model_key],
            all_input_tokens=token_counts[model_key],
        )
        summaries.append(summary)
        print(f"   ✅ {summary.display_name}: "
              f"costo dataset estimado = ${summary.estimated_total_cost_usd:.6f} USD")

    # ── 7. Generar y mostrar reporte ──────────────────────────────────────────
    df_report = generate_report(summaries)
    print_console_report(df_report)

    # ── 8. Exportar a CSV ─────────────────────────────────────────────────────
    export_to_csv(df_report)
    
    print("\n✅ Lab completado exitosamente.\n")


if __name__ == "__main__":
    main()
```

#### Salida Esperada

No hay salida en este paso (solo la creación del archivo).

#### Verificación

```bash
python -c "import ast; ast.parse(open('main.py').read()); print('main.py: sintaxis válida')"
```

---

### Paso 8 — Ejecutar el Pipeline Completo

**Objetivo:** Ejecutar `main.py` y verificar que el reporte comparativo se genera correctamente y se exporta a CSV.

#### Instrucciones

1. Asegúrate de que el entorno virtual esté activado y que el archivo `.env` esté configurado.

2. Ejecuta el script principal:

```bash
python main.py
```

3. Observa la salida en consola. Deberás ver el progreso de tokenización, las llamadas a las APIs con sus métricas individuales, y finalmente el reporte comparativo.

#### Salida Esperada (ejemplo representativo)

```
════════════════════════════════════════════════════════════
  LAB 01-00-01: Análisis Comparativo de Costos LLM
════════════════════════════════════════════════════════════
✅ Variables de entorno verificadas correctamente.

📂 Cargando dataset de prompts...
   ✅ Dataset cargado: 20 prompts
   Distribución: {'corto': 5, 'mediano': 7, 'largo': 8}

📊 Tokenizando dataset completo (sin llamadas a API de generación)...
   Procesados 5/20 prompts...
   Procesados 10/20 prompts...
   Procesados 15/20 prompts...
   Procesados 20/20 prompts...
   ✅ Tokenización completa: 20 prompts × 6 modelos

🚀 Ejecutando muestra de 5 prompts en cada API...
   Prompts seleccionados: IDs [1, 6, 10, 14, 19]
   Total de llamadas a API: 30

   ▶ Procesando GPT-4o (OpenAI)...
     ✓ Prompt ID 1: 8 in / 12 out / 0.843s
     ✓ Prompt ID 6: 31 in / 68 out / 1.204s
     ...
   ▶ Procesando GPT-4o-mini (OpenAI)...
     ...
   ▶ Procesando Gemini 1.5 Pro (Google)...
     ...
   ▶ Procesando Gemini 1.5 Flash (Google)...
     ...
   ▶ Procesando Claude 3.5 Sonnet (Anthropic)...
     ...
   ▶ Procesando Claude 3 Haiku (Anthropic)...
     ...

📈 Calculando costos y extrapolando al dataset completo...
   ✅ GPT-4o: costo dataset estimado = $0.002341 USD
   ✅ GPT-4o-mini: costo dataset estimado = $0.000071 USD
   ✅ Gemini 1.5 Pro: costo dataset estimado = $0.001638 USD
   ✅ Gemini 1.5 Flash: costo dataset estimado = $0.000035 USD
   ✅ Claude 3.5 Sonnet: costo dataset estimado = $0.001124 USD
   ✅ Claude 3 Haiku: costo dataset estimado = $0.000094 USD

════════════════════════════════════════════════════════════════════════════════════════════════════════
  REPORTE COMPARATIVO DE COSTOS — ANÁLISIS MULTI-PROVEEDOR
════════════════════════════════════════════════════════════════════════════════════════════════════════
 Proveedor          Modelo  Ventana Contexto (tokens)  Latencia Prom. (s)  ...  Costo Dataset (USD)
  OpenAI           GPT-4o                    128,000               1.124  ...          $0.002341
  OpenAI      GPT-4o-mini                    128,000               0.612  ...          $0.000071
  Google    Gemini 1.5 Pro                 1,000,000               2.341  ...          $0.001638
  Google   Gemini 1.5 Flash                1,000,000               0.891  ...          $0.000035
Anthropic  Claude 3.5 Sonnet               200,000               1.567  ...          $0.001124
Anthropic    Claude 3 Haiku                200,000               0.734  ...          $0.000094
════════════════════════════════════════════════════════════════════════════════════════════════════════

  ✅ Modelo más económico (dataset completo): Gemini 1.5 Flash ($0.000035)
  💰 Modelo más costoso (dataset completo):   GPT-4o ($0.002341)
  ⚡ Modelo con menor latencia:               GPT-4o-mini (0.612s)
════════════════════════════════════════════════════════════════════════════════════════════════════════

  📄 Reporte exportado exitosamente: cost_comparison_report.csv

✅ Lab completado exitosamente.
```

> **Nota:** Los valores exactos de tokens, latencia y costos variarán según las respuestas generadas por cada API en el momento de ejecución. Los valores anteriores son representativos.

#### Verificación

```bash
# Verificar que el CSV fue generado
ls -lh cost_comparison_report.csv

# Verificar el contenido del CSV
python -c "import pandas as pd; df = pd.read_csv('cost_comparison_report.csv'); print(df[['Modelo','Costo Dataset (USD)','Latencia Prom. (s)']].to_string())"
```

---

## 7. Validación y Pruebas

Ejecuta las siguientes verificaciones para confirmar que el lab se completó correctamente:

```bash
# ── Verificación 1: Todos los archivos del proyecto existen ──────────────────
python -c "
import os
required = ['config.py','tokenizer.py','api_executor.py',
            'cost_analyzer.py','reporter.py','main.py',
            'prompts_dataset.json','cost_comparison_report.csv','.env','.gitignore']
for f in required:
    status = '✅' if os.path.exists(f) else '❌'
    print(f'{status} {f}')
"

# ── Verificación 2: El CSV tiene exactamente 6 filas (una por modelo) ────────
python -c "
import pandas as pd
df = pd.read_csv('cost_comparison_report.csv')
assert len(df) == 6, f'Se esperaban 6 filas, se encontraron {len(df)}'
print(f'✅ CSV contiene {len(df)} modelos correctamente')
print(f'✅ Columnas: {list(df.columns)}')
"

# ── Verificación 3: Los tres proveedores están representados ─────────────────
python -c "
import pandas as pd
df = pd.read_csv('cost_comparison_report.csv')
providers = set(df['Proveedor'].unique())
expected = {'OpenAI', 'Google', 'Anthropic'}
assert providers == expected, f'Proveedores incorrectos: {providers}'
print(f'✅ Proveedores presentes: {providers}')
"

# ── Verificación 4: El .env NO está siendo trackeado por git ─────────────────
python -c "
with open('.gitignore') as f:
    content = f.read()
assert '.env' in content, '.env no está en .gitignore'
print('✅ .env está correctamente en .gitignore')
"

# ── Verificación 5: No hay API keys hardcodeadas en el código ────────────────
python -c "
import re, glob
pattern = re.compile(r'(sk-[a-zA-Z0-9]{20,}|AIza[a-zA-Z0-9_-]{35}|sk-ant-[a-zA-Z0-9-]{50,})')
for filepath in glob.glob('*.py'):
    with open(filepath) as f:
        content = f.read()
    matches = pattern.findall(content)
    if matches:
        print(f'❌ ALERTA: Posible API key en {filepath}')
    else:
        print(f'✅ {filepath}: sin credenciales hardcodeadas')
"
```

---

## 8. Resolución de Problemas

### Problema 1: `google.api_core.exceptions.ResourceExhausted: 429 Quota exceeded`

**Síntoma:** Al ejecutar el script, las llamadas a Gemini fallan con el error `429 Quota exceeded` o `RESOURCE_EXHAUSTED`. Las llamadas a OpenAI y Anthropic funcionan correctamente.

**Causa:** La API de Google AI Studio tiene límites de tasa por minuto (RPM) en el plan gratuito. Gemini 1.5 Pro tiene un límite de 2 RPM en el tier gratuito. Al enviar 5 prompts consecutivos a Gemini 1.5 Pro y luego 5 a Gemini 1.5 Flash sin pausa, se supera el límite.

**Solución:** Agrega un delay entre llamadas a la API de Google. Modifica la función `run_api_sample` en `main.py` para añadir una pausa cuando el proveedor sea Google:

```python
# En main.py, dentro del bucle de api_results, agrega después de call_fn():
import time

for prompt_data in sample_prompts:
    result = call_fn(
        prompt=prompt_data["text"],
        model_key=model_key,
        prompt_id=prompt_data["id"],
    )
    results[model_key].append(result)
    
    # Pausa para respetar rate limits de Google AI Studio (plan gratuito)
    if provider == "Google":
        time.sleep(3)  # 3 segundos entre llamadas = ~20 RPM máximo
    
    # ... resto del código de logging
```

Alternativamente, si tienes una cuenta de Google Cloud con Vertex AI habilitado, el límite de tasa es significativamente más alto.

---

### Problema 2: `ModuleNotFoundError: No module named 'tiktoken'` o resultados de tokenización inconsistentes

**Síntoma:** El script falla al importar `tiktoken` con `ModuleNotFoundError`, o los conteos de tokens para modelos OpenAI difieren significativamente de los valores reportados por la API en `response.usage.prompt_tokens`.

**Causa A (ModuleNotFoundError):** El entorno virtual no está activado o `tiktoken` no fue instalado en el entorno correcto. Es común cuando se tienen múltiples entornos Python en el sistema.

**Causa B (conteos inconsistentes):** Se está usando el encoding `cl100k_base` para contar tokens del prompt, pero la API de OpenAI incluye tokens adicionales del formato de mensaje del sistema (`role`, `content`, separadores de chat). El conteo de tiktoken sobre el texto plano subestima ligeramente el conteo real.

**Solución A:**
```bash
# Verificar qué Python y pip están activos
which python  # Linux/macOS
where python  # Windows

# Confirmar que el entorno virtual está activo (debe mostrar (.venv) en el prompt)
# Si no está activo:
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\Activate.ps1  # Windows

# Reinstalar en el entorno correcto
pip install tiktoken==0.7.0
python -c "import tiktoken; print('tiktoken OK:', tiktoken.__version__)"
```

**Solución B:** Para el propósito de este lab (estimación de costos), una diferencia del 3-5% entre el conteo de tiktoken y el conteo real de la API es aceptable. Si necesitas precisión exacta, usa el conteo de tokens devuelto por `response.usage.prompt_tokens` en lugar del calculado con tiktoken. El módulo `api_executor.py` ya usa los valores reales de la API para la muestra; el conteo con tiktoken se usa únicamente para la extrapolación del dataset completo.

---

## 9. Limpieza del Entorno

Una vez completado el lab, ejecuta los siguientes pasos para limpiar los recursos:

```bash
# 1. Desactivar el entorno virtual
deactivate

# 2. (Opcional) Eliminar el entorno virtual para liberar espacio en disco
# ADVERTENCIA: esto eliminará todas las dependencias instaladas
# rm -rf .venv  # Linux/macOS
# Remove-Item -Recurse -Force .venv  # Windows PowerShell

# 3. Verificar que el archivo .env NO está en el historial de git
# (si inicializaste un repositorio git en este directorio)
git status  # .env NO debe aparecer en la lista de archivos trackeados

# 4. El archivo cost_comparison_report.csv está en .gitignore
# Puedes conservarlo para referencia o eliminarlo:
# rm cost_comparison_report.csv
```

> 💡 **Recomendación:** Conserva el directorio `lab-01-00-01` con el archivo `cost_comparison_report.csv` generado. Lo utilizarás como referencia en los labs posteriores del módulo para justificar la selección de modelos en tus arquitecturas.

---

## 10. Resumen

En este lab implementaste un pipeline completo de análisis de costos multi-proveedor que traduce los conceptos teóricos de la Lección 1.1 en una herramienta cuantitativa real. Los puntos clave que aplicaste:

| Concepto de Lección 1.1 | Implementación en el Lab |
|---|---|
| Modelos comerciales con pago por token | Módulo `config.py` con precios por 1M tokens |
| GPT-4o vs GPT-4o-mini (costo vs. rendimiento) | Comparación directa en el reporte CSV |
| Gemini 1.5 Pro con ventana de 1M tokens | Columna "Ventana Contexto" en el reporte |
| Claude con enfoque en seguridad | Incluido como proveedor Anthropic en el análisis |
| Selección de modelo según caso de uso | Identificación automática del modelo óptimo por costo y latencia |
| Abstracción de la capa del modelo | Arquitectura modular que permite agregar nuevos modelos editando solo `config.py` |

### Estructura Final del Proyecto

```
lab-01-00-01/
├── .env                          # API keys (en .gitignore)
├── .gitignore                    # Excluye .env y .venv
├── config.py                     # Precios y metadatos de modelos
├── tokenizer.py                  # Módulo de tokenización multi-proveedor
├── api_executor.py               # Llamadas a APIs con medición de latencia
├── cost_analyzer.py              # Cálculo y extrapolación de costos
├── reporter.py                   # Generación de tabla y exportación CSV
├── main.py                       # Orquestador del pipeline completo
├── prompts_dataset.json          # Dataset de 20 prompts
└── cost_comparison_report.csv    # Reporte generado (en .gitignore)
```

### Recursos Adicionales

- [OpenAI Pricing — Precios actualizados por modelo](https://openai.com/pricing)
- [Google AI Studio Pricing — Precios de Gemini](https://ai.google.dev/pricing)
- [Anthropic Pricing — Precios de Claude](https://www.anthropic.com/pricing)
- [tiktoken — Documentación oficial](https://github.com/openai/tiktoken)
- [Anthropic Token Counting Guide](https://docs.anthropic.com/en/docs/build-with-claude/token-counting)
- [LMSYS Chatbot Arena — Comparativa de rendimiento](https://chat.lmsys.org)

---
