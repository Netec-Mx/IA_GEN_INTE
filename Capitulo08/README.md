<div align="center">

# 🧪 Laboratorio 8

## Agente con Function Calling para tareas matemáticas, consulta de datos y herramientas controladas

![Nivel](https://img.shields.io/badge/Nivel-Intermedio%20Alto-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)
![API](https://img.shields.io/badge/API-OpenAI%20Function%20Calling-111111?style=flat-square)

</div>

> [!IMPORTANT]
> En este laboratorio construirás un agente controlado y observable. El objetivo no es solo “llamar funciones”, sino entender cómo un modelo decide qué herramienta usar, cómo se ejecutan esas herramientas, cómo se controla el riesgo y cómo se valida el comportamiento del agente.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Agentes con herramientas</td>
<td width="25%"><strong>⏱️ Duración</strong><br>58 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar, evaluar y crear</td>
<td width="25%"><strong>📦 Entregable</strong><br>Agente CLI + herramientas + pruebas</td>
</tr>
</table>

---

## 🧭 Sección 1. Información general de la práctica

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| **Nombre** | Agente con Function Calling para tareas matemáticas y consulta de datos |
| **Duración estimada** | 58 minutos |
| **Complejidad** | Alta |
| **Nivel Bloom** | Aplicar, analizar, evaluar y crear |
| **Costo estimado** | Bajo, aproximadamente $0.10–$0.20 USD según pruebas y reintentos |
| **Modalidad** | Individual o parejas |
| **Entregables** | Código Python, datasets CSV, logs, reporte opcional y matriz de evaluación |

---

### 📌 Descripción general

En este laboratorio construirás un agente conversacional en Python llamado `math_data_agent.py`. El agente usará **Function Calling** para decidir dinámicamente cuándo invocar herramientas externas y cómo combinar sus resultados para responder una consulta.

A diferencia de un workflow tradicional, donde tú programas una secuencia fija de pasos, aquí registrarás herramientas y permitirás que el modelo decida cuál usar, con qué argumentos y en qué orden. Esto te permitirá observar el patrón agentico:

```text
Observar → Decidir herramienta → Ejecutar acción → Integrar resultado → Responder
```

El agente contará con cinco herramientas:

| Herramienta | Propósito |
|---|---|
| `calculate_expression` | Evalúa expresiones matemáticas de forma segura usando `ast`, sin `eval()` |
| `get_currency_exchange` | Consulta tipos de cambio con API pública y fallback local |
| `query_statistics` | Calcula estadísticas sobre datasets CSV locales |
| `search_wikipedia_summary` | Recupera resúmenes desde Wikipedia con fallback local |
| `create_calculation_report` | Genera reportes Markdown con confirmación humana |

Además, implementarás:

- JSON Schema para describir herramientas.
- Dispatcher de funciones.
- Bucle agentico con límite de iteraciones.
- Manejo de múltiples `tool_calls` en una misma iteración.
- Ejecución paralela controlada para herramientas seguras.
- Human-in-the-Loop para acciones con efecto secundario.
- Logging estructurado de llamadas a herramientas.
- Suite de pruebas con trazabilidad de herramientas usadas.
- Comparación final entre agente y workflow.

---

### 🎯 Objetivos de aprendizaje

Al finalizar este laboratorio, podrás:

1. Diseñar herramientas invocables por un modelo mediante JSON Schema.
2. Implementar un bucle agentico basado en Function Calling.
3. Diferenciar un agente de un workflow mediante comportamiento observable.
4. Ejecutar herramientas de forma segura y controlada.
5. Proteger acciones con efecto secundario usando confirmación humana.
6. Implementar evaluación matemática segura sin `eval()`.
7. Consultar fuentes externas con fallback para entornos de aula.
8. Ejecutar y validar múltiples herramientas solicitadas por el modelo.
9. Registrar trazabilidad de herramientas usadas por consulta.
10. Evaluar si el agente respondió correctamente y si eligió herramientas adecuadas.

---

### ✅ Prerrequisitos

#### 3.1 Conocimientos previos

Antes de iniciar, asegúrate de conocer:

- Uso básico de Python.
- Creación de entornos virtuales.
- Lectura y escritura de archivos.
- Fundamentos de JSON y JSON Schema.
- Conceptos básicos de APIs HTTP.
- Uso básico de `pandas`.
- Conceptos del módulo: agentes, workflows, herramientas y Human-in-the-Loop.

#### 3.2 Laboratorios recomendados antes de este

| Laboratorio previo | Motivo |
|---|---|
| Laboratorio 2 | Uso de endpoints y validación de solicitudes |
| Laboratorio 3 | Cliente LLM robusto, respuestas estructuradas y reintentos |
| Laboratorio 4 | Trazabilidad, reportes y seguridad en automatizaciones |

---

### 💻 Hardware y software requerido

#### Hardware

| Recurso | Mínimo | Recomendado |
|---|---:|---:|
| CPU | 2 núcleos | 4 núcleos |
| RAM | 8 GB | 16 GB |
| Disco libre | 500 MB | 1 GB |
| Internet | 10 Mbps | 25 Mbps |

#### Software

| Software | Versión recomendada | Uso |
|---|---|---|
| Windows | 10/11 | Sistema operativo |
| Visual Studio Code | Actual | Editor de código |
| Git Bash | Actual | Terminal del laboratorio |
| Python | 3.11+ | Runtime principal |
| OpenAI SDK | `>=1.90,<2` | Function Calling |
| pandas | `>=2.2,<3` | Consulta de datasets |
| requests | `>=2.32,<3` | APIs externas |
| wikipedia-api | `>=0.8,<1` | Consulta de Wikipedia |
| python-dotenv | `>=1.0,<2` | Variables de entorno |
| tenacity | `>=8.5,<10` | Reintentos con backoff |

---

### 🛡️ Consideraciones importantes para estudiantes

> [!WARNING]
> Este laboratorio usa una API de pago. Aunque el costo estimado es bajo, valida que tengas límites de gasto configurados en tu proveedor.

> [!CAUTION]
> No escribas tu API key dentro del código. Usa siempre el archivo `.env` y verifica que esté incluido en `.gitignore`.

> [!NOTE]
> Las APIs públicas como Frankfurter o Wikipedia pueden fallar por red, disponibilidad o restricciones temporales. Por eso el laboratorio incluye fallbacks locales para que puedas continuar aunque alguna API externa no responda.

> [!TIP]
> Si un resultado cambia con el tiempo, como tipos de cambio, documenta la fecha y fuente. No trates esos valores como constantes.

> [!NOTE]
> Este laboratorio no usa ChromaDB. Si en tu terminal aparecen mensajes como `Failed to send telemetry event ClientStartEvent` o `ClientCreateCollectionEvent`, normalmente provienen de otra práctica, librería o ambiente reutilizado con ChromaDB. Trátalos como advertencias de telemetría no críticas si el proceso principal sigue funcionando; no indican que el agente, los CSV o Function Calling hayan fallado.

---

### 🧠 Diferencia conceptual: agente vs. workflow

Antes de programar, observa la diferencia principal:

| Caso | Workflow | Agente |
|---|---|---|
| Conversión de moneda | El código siempre llama a la herramienta de moneda | El modelo decide si necesita moneda |
| Estadísticas | El flujo decide dataset, columna y operación | El modelo elige herramienta y argumentos |
| Reporte | El sistema siempre exporta | El modelo solo exporta si el usuario lo pide |
| Error externo | El flujo suele romperse si un paso falla | El agente puede explicar el error y sugerir alternativa |
| Secuencia | Fija | Dinámica |

En este laboratorio no programarás una ruta fija como:

```text
1. Calcular
2. Consultar moneda
3. Consultar CSV
4. Responder
```

En su lugar, registrarás herramientas y permitirás que el modelo decida si las necesita.

---

### 🗂️ Estructura final del proyecto

Al finalizar tendrás esta estructura:

```text
lab-08-agente-function-calling/
├── .env
├── .gitignore
├── requirements.txt
├── tools_schema.py
├── tools_impl.py
├── math_data_agent.py
├── test_tools.py
├── test_agent_queries.py
├── matriz_evaluacion.md
├── data/
│   ├── ventas.csv
│   └── temperaturas.csv
├── logs/
│   └── agent.log
└── reports/
    └── reporte_YYYYMMDD_HHMMSS.md
```

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local

## 🎯 Objetivo de la tarea

Crear el directorio del laboratorio, abrirlo en VS Code, configurar el entorno virtual, instalar dependencias y proteger archivos sensibles.

### ✅ Paso 1.1. Crear carpeta del laboratorio

**📝 Descripción del paso:**

Ejecuta estos comandos en Git Bash. Con ellos crearás la carpeta del laboratorio y entrarás en ella para que todos los archivos, scripts, datos, logs y reportes se generen en una sola ubicación controlada.

Ejecuta en **Git Bash**:

```bash
mkdir -p ~/labs-ia-gen/lab-08-agente-function-calling
cd ~/labs-ia-gen/lab-08-agente-function-calling
```

**✅ Validación del paso:**

```bash
pwd
```

**📌 Resultado esperado:**

La ruta debe terminar en:

```text
/labs-ia-gen/lab-08-agente-function-calling
```

---

### ✅ Paso 1.2. Abrir el proyecto en Visual Studio Code

**📝 Descripción del paso:**

Ejecuta este comando desde la carpeta `lab-08-agente-function-calling`. VS Code se abrirá usando esa carpeta como proyecto, para que puedas crear y editar los archivos del laboratorio desde el explorador lateral.

```bash
code .
```

**✅ Validación del paso:**

Confirma que VS Code muestra una carpeta vacía llamada:

```text
lab-08-agente-function-calling
```

---

### ✅ Paso 1.3. Crear y activar entorno virtual

**📝 Descripción del paso:**

Ejecuta estos comandos en Git Bash dentro de la carpeta del laboratorio. Primero crearás la carpeta `.venv/` con un entorno virtual local y después lo activarás para que las instalaciones de paquetes se apliquen solo a esta práctica.

```bash
python -m venv .venv
source .venv/Scripts/activate
```

**✅ Validación del paso:**

```bash
python --version
which python
```

**📌 Resultado esperado:**

La ruta de Python debe incluir `.venv`.

---

### ✅ Paso 1.4. Crear `requirements.txt`

**📝 Descripción del paso:**

Crea el archivo `requirements.txt` en la raíz del proyecto. Este archivo lista las librerías que instalarás para usar Function Calling, hacer solicitudes HTTP, leer CSV, consultar Wikipedia, cargar variables de entorno y aplicar reintentos.

```bash
cat > requirements.txt << 'EOF'
openai>=1.90,<2
requests>=2.32,<3
pandas>=2.2,<3
wikipedia-api>=0.8,<1
python-dotenv>=1.0,<2
tenacity>=8.5,<10
EOF
```

**✅ Validación del paso:**

```bash
cat requirements.txt
```

---

### ✅ Paso 1.5. Instalar dependencias

**📝 Descripción del paso:**

Ejecuta estos comandos en Git Bash con el entorno virtual activado. El primer comando actualiza `pip` y el segundo instala todas las dependencias declaradas en `requirements.txt`.

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**✅ Validación del paso:**

```bash
python -c "import openai, requests, pandas, wikipediaapi, dotenv, tenacity; print('Dependencias instaladas correctamente')"
```

**📌 Resultado esperado:**

```text
Dependencias instaladas correctamente
```

---

### ✅ Paso 1.6. Crear `.env`

**📝 Descripción del paso:**

Crea el archivo `.env` en la raíz del proyecto. En este archivo guardarás la API key, el modelo que usará el agente y la bandera para permitir fallbacks locales cuando una API externa no responda.

```bash
cat > .env << 'EOF'
OPENAI_API_KEY=pega_aqui_tu_api_key
OPENAI_MODEL=gpt-4o-mini
USE_OFFLINE_FALLBACKS=true
EOF
```

**✅ Validación del paso:**

```bash
cat .env
```

> [!IMPORTANT]
> Antes de continuar, reemplaza `pega_aqui_tu_api_key` por tu API key real.

---

### ✅ Paso 1.7. Crear `.gitignore`

**📝 Descripción del paso:**

Crea el archivo `.gitignore` en la raíz del proyecto. Este archivo evita que se suban credenciales, el entorno virtual, logs, reportes y datasets generados si después decides versionar el proyecto con Git.

```bash
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
logs/
reports/
data/*.csv
EOF
```

**✅ Validación del paso:**

```bash
grep ".env" .gitignore
```

**📌 Resultado esperado:**

```text
.env
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20un%20laboratorio%20con%20agentes%20necesita%20variables%20de%20entorno%2C%20logs%2C%20l%C3%ADmites%20de%20costo%20y%20un%20archivo%20.gitignore%20bien%20configurado.)

---

# 🧩 Tarea 2. Crear datasets locales de prueba

## 🎯 Objetivo de la tarea

Crear datasets CSV pequeños para que el agente pueda responder preguntas usando datos estructurados locales.

---

### ✅ Paso 2.1. Crear carpetas `data`, `logs` y `reports`

**📝 Descripción del paso:**

Ejecuta este comando en Git Bash desde la raíz del proyecto. Crearás tres carpetas: `data/` para datasets CSV, `logs/` para trazabilidad del agente y `reports/` para archivos Markdown generados por herramientas con efecto secundario.

```bash
mkdir -p data logs reports
```

**✅ Validación del paso:**

```bash
ls -la
```

**📌 Resultado esperado:**

Debes ver:

```text
data/
logs/
reports/
```

---

### ✅ Paso 2.2. Crear datasets CSV

**📝 Descripción del paso:**

Crea el archivo `crear_datasets.py` en la raíz del proyecto y ejecútalo. El script generará dos archivos CSV dentro de `data/`: `ventas.csv` y `temperaturas.csv`, que después serán consultados por la herramienta `query_statistics`.

```bash
cat > crear_datasets.py << 'EOF'
import os
import pandas as pd

os.makedirs("data", exist_ok=True)

ventas = pd.DataFrame({
    "mes": ["Ene", "Feb", "Mar", "Abr", "May", "Jun"],
    "ventas": [15200, 18400, 21300, 17800, 23100, 25600],
    "costos": [9100, 10200, 11800, 9900, 12400, 13200],
    "unidades": [152, 184, 213, 178, 231, 256]
})

ventas.to_csv("data/ventas.csv", index=False)

temperaturas = pd.DataFrame({
    "ciudad": ["Madrid", "Barcelona", "Sevilla", "Bilbao", "Valencia"],
    "temp_max": [28.5, 26.3, 35.2, 22.1, 30.8],
    "temp_min": [14.2, 16.1, 20.3, 11.5, 17.9],
    "humedad": [45, 68, 32, 78, 55]
})

temperaturas.to_csv("data/temperaturas.csv", index=False)

print("Datasets creados correctamente:")
print("- data/ventas.csv")
print("- data/temperaturas.csv")
EOF
```
```bash
python crear_datasets.py
```

**✅ Validación del paso:**

```bash
ls -la data
```
```bash
python -c "import pandas as pd; print(pd.read_csv('data/ventas.csv').head()); print(pd.read_csv('data/temperaturas.csv').head())"
```

**📌 Resultado esperado:**

Debes ver dos archivos CSV y una vista previa de ambos datasets.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20un%20agente%20con%20herramientas%20puede%20consultar%20datos%20locales%20y%20APIs%20externas%20en%20una%20misma%20conversaci%C3%B3n%2C%20y%20qu%C3%A9%20riesgos%20existen%20si%20las%20herramientas%20no%20validan%20sus%20argumentos.)

---

# 🧩 Tarea 3. Definir esquemas JSON Schema de herramientas

## 🎯 Objetivo de la tarea

Crear `tools_schema.py` con la descripción formal de las herramientas que el modelo podrá invocar.

---

### ✅ Paso 3.1. Crear `tools_schema.py`

**📝 Descripción del paso:**

Crea el archivo `tools_schema.py` en la raíz del proyecto. En este archivo definirás los JSON Schema que describen al modelo qué herramientas existen, para qué sirven y qué argumentos acepta cada una.

```bash
cat > tools_schema.py << 'EOF'
"""
Esquemas JSON Schema para las herramientas del agente.
Cada herramienta define nombre, descripción y parámetros esperados.
"""

TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "calculate_expression",
            "description": (
                "Evalúa expresiones matemáticas de forma segura. "
                "Soporta +, -, *, /, **, %, abs() y round(). "
                "No usa eval(). Rechaza expresiones peligrosas o demasiado costosas."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "Expresión matemática. Ejemplos: '2 + 3 * 4', '(100 - 20) / 4'."
                    }
                },
                "required": ["expression"],
                "additionalProperties": False
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "get_currency_exchange",
            "description": (
                "Obtiene tipo de cambio entre dos monedas usando una API pública. "
                "Si la API no responde, puede usar una tasa local de referencia marcada como fallback."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "base": {
                        "type": "string",
                        "description": "Código ISO 4217 de moneda base. Ejemplo: USD, EUR, MXN.",
                        "minLength": 3,
                        "maxLength": 3
                    },
                    "target": {
                        "type": "string",
                        "description": "Código ISO 4217 de moneda destino. Ejemplo: USD, EUR, MXN.",
                        "minLength": 3,
                        "maxLength": 3
                    }
                },
                "required": ["base", "target"],
                "additionalProperties": False
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "query_statistics",
            "description": (
                "Calcula estadísticas sobre datasets CSV locales. "
                "Datasets disponibles: ventas y temperaturas. "
                "Operaciones: mean, median, std, max, min, sum, count."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "dataset_name": {
                        "type": "string",
                        "enum": ["ventas", "temperaturas"],
                        "description": "Nombre del dataset sin extensión."
                    },
                    "operation": {
                        "type": "string",
                        "enum": ["mean", "median", "std", "max", "min", "sum", "count"],
                        "description": "Operación estadística a ejecutar."
                    },
                    "column": {
                        "type": "string",
                        "description": "Columna numérica sobre la cual aplicar la operación."
                    }
                },
                "required": ["dataset_name", "operation", "column"],
                "additionalProperties": False
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_wikipedia_summary",
            "description": (
                "Obtiene un resumen introductorio de Wikipedia. "
                "Útil para definiciones y contexto factual general. "
                "Si no hay conexión, puede usar resúmenes locales de referencia."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "Tema a buscar. Ejemplo: 'Inteligencia artificial', 'Pi', 'ReAct'."
                    },
                    "language": {
                        "type": "string",
                        "enum": ["es", "en"],
                        "default": "es",
                        "description": "Idioma de búsqueda."
                    }
                },
                "required": ["query"],
                "additionalProperties": False
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "create_calculation_report",
            "description": (
                "Herramienta con efecto secundario. Genera y guarda un reporte Markdown en disco. "
                "Solo debe usarse si el usuario pide explícitamente guardar, exportar o generar un reporte. "
                "Requiere confirmación humana antes de ejecutarse."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "title": {
                        "type": "string",
                        "description": "Título del reporte."
                    },
                    "calculations": {
                        "type": "array",
                        "description": "Lista de cálculos o resultados a incluir.",
                        "items": {
                            "type": "object",
                            "properties": {
                                "description": {
                                    "type": "string",
                                    "description": "Descripción del cálculo o resultado."
                                },
                                "result": {
                                    "type": "string",
                                    "description": "Resultado como texto."
                                }
                            },
                            "required": ["description", "result"],
                            "additionalProperties": False
                        }
                    }
                },
                "required": ["title", "calculations"],
                "additionalProperties": False
            }
        }
    }
]
EOF
```

**✅ Validación del paso:**

```bash
python -c "from tools_schema import TOOLS; print(f'{len(TOOLS)} herramientas definidas'); print([t['function']['name'] for t in TOOLS])"
```

**📌 Resultado esperado:**

```text
5 herramientas definidas
['calculate_expression', 'get_currency_exchange', 'query_statistics', 'search_wikipedia_summary', 'create_calculation_report']
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20c%C3%B3mo%20influye%20la%20descripci%C3%B3n%20de%20una%20herramienta%20en%20la%20decisi%C3%B3n%20del%20modelo%20al%20usar%20Function%20Calling.%20Dame%20ejemplos%20de%20buenas%20y%20malas%20descripciones.)

---

# 🧩 Tarea 4. Implementar herramientas seguras

## 🎯 Objetivo de la tarea

Crear `tools_impl.py` con las funciones reales que ejecutará el agente.

---

### ✅ Paso 4.1. Crear `tools_impl.py`

**📝 Descripción del paso:**

Crea el archivo `tools_impl.py` en la raíz del proyecto. En este archivo escribirás la implementación real de las funciones que el agente podrá ejecutar cuando el modelo solicite una herramienta. Implementas cinco herramientas con validación y controles:

- Matemáticas sin `eval()`.
- Límite de tamaño y potencia.
- Tipo de cambio con fallback.
- Estadísticas con `sum` y `count`.
- Wikipedia con fallback local.
- Reporte Markdown con sanitización básica.

```bash
cat > tools_impl.py << 'EOF'
"""
Implementación de herramientas para el agente con Function Calling.
Incluye validación de argumentos, fallbacks y controles de seguridad.
"""

from __future__ import annotations

import ast
import logging
import operator
import os
import re
from datetime import datetime
from typing import Any

import pandas as pd
import requests
import wikipediaapi
from tenacity import retry, retry_if_exception_type, stop_after_attempt, wait_exponential

logger = logging.getLogger(__name__)

USE_OFFLINE_FALLBACKS = os.getenv("USE_OFFLINE_FALLBACKS", "true").lower() in {"1", "true", "yes", "si", "sí"}

# -------------------------------------------------------------------
# Herramienta 1: evaluación matemática segura
# -------------------------------------------------------------------

MAX_EXPRESSION_LENGTH = 120
MAX_ABS_NUMBER = 1_000_000
MAX_POWER_EXPONENT = 10

_OPERADORES_PERMITIDOS = {
    ast.Add: operator.add,
    ast.Sub: operator.sub,
    ast.Mult: operator.mul,
    ast.Div: operator.truediv,
    ast.Pow: operator.pow,
    ast.Mod: operator.mod,
    ast.USub: operator.neg,
    ast.UAdd: operator.pos,
}

_FUNCIONES_PERMITIDAS = {
    "abs": abs,
    "round": round,
}


def _validar_numero(valor: Any) -> float:
    if not isinstance(valor, (int, float)):
        raise ValueError(f"Solo se permiten números. Valor recibido: {valor!r}")
    if abs(valor) > MAX_ABS_NUMBER:
        raise ValueError(f"Número demasiado grande. Límite absoluto: {MAX_ABS_NUMBER}")
    return valor


def _evaluar_nodo(nodo: ast.AST) -> float:
    if isinstance(nodo, ast.Constant):
        return _validar_numero(nodo.value)

    if isinstance(nodo, ast.BinOp):
        tipo_op = type(nodo.op)
        if tipo_op not in _OPERADORES_PERMITIDOS:
            raise ValueError(f"Operador no permitido: {tipo_op.__name__}")

        izquierdo = _evaluar_nodo(nodo.left)
        derecho = _evaluar_nodo(nodo.right)

        if isinstance(nodo.op, ast.Pow) and abs(derecho) > MAX_POWER_EXPONENT:
            raise ValueError(f"Exponente demasiado grande. Límite: {MAX_POWER_EXPONENT}")

        resultado = _OPERADORES_PERMITIDOS[tipo_op](izquierdo, derecho)
        return _validar_numero(resultado)

    if isinstance(nodo, ast.UnaryOp):
        tipo_op = type(nodo.op)
        if tipo_op not in _OPERADORES_PERMITIDOS:
            raise ValueError(f"Operador unario no permitido: {tipo_op.__name__}")
        return _validar_numero(_OPERADORES_PERMITIDOS[tipo_op](_evaluar_nodo(nodo.operand)))

    if isinstance(nodo, ast.Call):
        if not isinstance(nodo.func, ast.Name) or nodo.func.id not in _FUNCIONES_PERMITIDAS:
            raise ValueError("Solo se permiten las funciones abs() y round()")
        args = [_evaluar_nodo(arg) for arg in nodo.args]
        resultado = _FUNCIONES_PERMITIDAS[nodo.func.id](*args)
        return _validar_numero(resultado)

    raise ValueError(f"Expresión no soportada: {type(nodo).__name__}")


def calculate_expression(expression: str) -> dict:
    """Evalúa una expresión matemática segura usando AST."""
    logger.info("[TOOL] calculate_expression | expression=%s", expression)

    if not isinstance(expression, str) or not expression.strip():
        return {"error": "La expresión no puede estar vacía."}

    expression = expression.strip()

    if len(expression) > MAX_EXPRESSION_LENGTH:
        return {"error": f"Expresión demasiado larga. Máximo {MAX_EXPRESSION_LENGTH} caracteres.", "expression": expression}

    try:
        arbol = ast.parse(expression, mode="eval")
        resultado = _evaluar_nodo(arbol.body)
        return {
            "expression": expression,
            "result": float(resultado),
            "description": f"{expression} = {resultado}"
        }
    except ZeroDivisionError:
        return {"error": "División por cero no permitida.", "expression": expression}
    except SyntaxError as exc:
        return {"error": f"Sintaxis inválida: {exc}", "expression": expression}
    except Exception as exc:
        return {"error": f"Expresión inválida o no permitida: {exc}", "expression": expression}


# -------------------------------------------------------------------
# Herramienta 2: tipo de cambio
# -------------------------------------------------------------------

FRANKFURTER_PRIMARY_URL = "https://api.frankfurter.dev/v1/latest"
FRANKFURTER_LEGACY_URL = "https://api.frankfurter.app/latest"

FALLBACK_RATES = {
    ("USD", "EUR"): 0.92,
    ("EUR", "USD"): 1.09,
    ("USD", "MXN"): 18.00,
    ("MXN", "USD"): 0.055,
    ("USD", "GBP"): 0.79,
    ("USD", "BRL"): 5.20,
    ("USD", "COP"): 4000.00,
}


def _validar_moneda(code: str) -> str:
    code = str(code).upper().strip()
    if not re.fullmatch(r"[A-Z]{3}", code):
        raise ValueError(f"Código de moneda inválido: {code!r}")
    return code


@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10),
    retry=retry_if_exception_type((requests.RequestException, TimeoutError)),
    reraise=True,
)
def _request_exchange_rate(url: str, base: str, target: str) -> dict:
    response = requests.get(url, params={"from": base, "to": target}, timeout=10)
    response.raise_for_status()
    return response.json()


def get_currency_exchange(base: str, target: str) -> dict:
    """Obtiene tipo de cambio actual con fallback local."""
    try:
        base = _validar_moneda(base)
        target = _validar_moneda(target)
    except ValueError as exc:
        return {"error": str(exc)}

    logger.info("[TOOL] get_currency_exchange | %s -> %s", base, target)

    if base == target:
        return {
            "base": base,
            "target": target,
            "rate": 1.0,
            "date": datetime.now().strftime("%Y-%m-%d"),
            "source": "identity",
            "description": f"1 {base} = 1 {target}"
        }

    for url in (FRANKFURTER_PRIMARY_URL, FRANKFURTER_LEGACY_URL):
        try:
            data = _request_exchange_rate(url, base, target)
            rates = data.get("rates", {})
            if target not in rates:
                continue
            rate = float(rates[target])
            return {
                "base": base,
                "target": target,
                "rate": rate,
                "date": data.get("date", "N/A"),
                "source": url,
                "description": f"1 {base} = {rate:.4f} {target}"
            }
        except Exception as exc:
            logger.warning("[TOOL] exchange endpoint failed | url=%s | error=%s", url, exc)

    if USE_OFFLINE_FALLBACKS and (base, target) in FALLBACK_RATES:
        rate = FALLBACK_RATES[(base, target)]
        return {
            "base": base,
            "target": target,
            "rate": rate,
            "date": "fallback-reference",
            "source": "offline_fallback",
            "warning": "Tasa local de referencia. Verifica una fuente oficial para decisiones reales.",
            "description": f"1 {base} ≈ {rate:.4f} {target}"
        }

    return {"error": f"No se pudo obtener tipo de cambio para {base}->{target}."}


# -------------------------------------------------------------------
# Herramienta 3: estadísticas sobre CSV
# -------------------------------------------------------------------

_DATASETS_CACHE: dict[str, pd.DataFrame] = {}


def _cargar_dataset(nombre: str) -> pd.DataFrame:
    nombre = str(nombre).strip().lower()
    if nombre not in {"ventas", "temperaturas"}:
        raise ValueError("Dataset no permitido. Usa: ventas o temperaturas.")

    if nombre not in _DATASETS_CACHE:
        ruta = os.path.join("data", f"{nombre}.csv")
        if not os.path.exists(ruta):
            raise FileNotFoundError(f"Dataset no encontrado: {ruta}")
        _DATASETS_CACHE[nombre] = pd.read_csv(ruta)

    return _DATASETS_CACHE[nombre]


def query_statistics(dataset_name: str, operation: str, column: str) -> dict:
    """Calcula estadísticas sobre datasets CSV locales."""
    operation = str(operation).strip().lower()
    column = str(column).strip()

    operaciones = {"mean", "median", "std", "max", "min", "sum", "count"}
    if operation not in operaciones:
        return {"error": f"Operación no permitida: {operation}. Usa: {sorted(operaciones)}"}

    try:
        df = _cargar_dataset(dataset_name)

        if column not in df.columns:
            columnas_numericas = [c for c in df.columns if pd.api.types.is_numeric_dtype(df[c])]
            return {
                "error": f"La columna '{column}' no existe en el dataset '{dataset_name}'.",
                "available_columns": list(df.columns),
                "numeric_columns": columnas_numericas
            }

        if not pd.api.types.is_numeric_dtype(df[column]):
            return {
                "error": f"La columna '{column}' no es numérica.",
                "available_columns": list(df.columns)
            }

        serie = df[column].dropna()
        if operation == "count":
            result = int(serie.count())
        else:
            result = float(getattr(serie, operation)())

        return {
            "dataset": dataset_name,
            "column": column,
            "operation": operation,
            "result": result,
            "n_rows": int(serie.count()),
            "description": f"{operation}({dataset_name}.{column}) = {result}"
        }
    except Exception as exc:
        return {"error": f"Error al calcular estadística: {exc}"}


# -------------------------------------------------------------------
# Herramienta 4: Wikipedia con fallback
# -------------------------------------------------------------------

_WIKI_CLIENTS: dict[str, wikipediaapi.Wikipedia] = {}

LOCAL_WIKI_SUMMARIES = {
    "pi": {
        "title": "Pi",
        "summary": "Pi es una constante matemática que representa la relación entre la longitud de una circunferencia y su diámetro. Su valor aproximado es 3.14159265.",
        "url": "offline://pi"
    },
    "inteligencia artificial": {
        "title": "Inteligencia artificial",
        "summary": "La inteligencia artificial es un campo de la informática que busca crear sistemas capaces de realizar tareas que normalmente requieren inteligencia humana, como razonamiento, aprendizaje y percepción.",
        "url": "offline://inteligencia-artificial"
    },
    "react": {
        "title": "ReAct",
        "summary": "ReAct es un patrón de agentes que combina razonamiento y acción, permitiendo que un modelo decida cuándo usar herramientas y cómo integrar sus resultados.",
        "url": "offline://react"
    }
}


def _get_wiki_client(language: str) -> wikipediaapi.Wikipedia:
    language = language if language in {"es", "en"} else "es"
    if language not in _WIKI_CLIENTS:
        _WIKI_CLIENTS[language] = wikipediaapi.Wikipedia(
            language=language,
            user_agent="genai-intermedio-lab08/1.0 (training; contact: instructor)"
        )
    return _WIKI_CLIENTS[language]


def search_wikipedia_summary(query: str, language: str = "es") -> dict:
    """Busca resumen en Wikipedia o fallback local."""
    query = str(query).strip()
    language = language if language in {"es", "en"} else "es"

    if not query:
        return {"error": "La consulta de Wikipedia no puede estar vacía."}

    logger.info("[TOOL] search_wikipedia_summary | query=%s | language=%s", query, language)

    try:
        wiki = _get_wiki_client(language)
        page = wiki.page(query)
        if page.exists():
            summary = page.summary[:700]
            if len(page.summary) > 700:
                summary += "..."
            return {
                "title": page.title,
                "summary": summary,
                "url": page.fullurl,
                "language": language,
                "source": "wikipedia"
            }
    except Exception as exc:
        logger.warning("[TOOL] Wikipedia failed | error=%s", exc)

    key = query.lower()
    if USE_OFFLINE_FALLBACKS:
        for local_key, value in LOCAL_WIKI_SUMMARIES.items():
            if local_key in key or key in local_key:
                return {
                    **value,
                    "language": language,
                    "source": "offline_fallback",
                    "warning": "Resumen local de referencia. Verifica Wikipedia para información actual."
                }

    return {"error": f"No se encontró información para '{query}'."}


# -------------------------------------------------------------------
# Herramienta 5: reporte Markdown con sanitización
# -------------------------------------------------------------------


def _safe_markdown_text(text: Any, max_len: int = 500) -> str:
    text = str(text).replace("\x00", "").strip()
    text = re.sub(r"[\r\n]{3,}", "\n\n", text)
    return text[:max_len]


def create_calculation_report(title: str, calculations: list[dict]) -> dict:
    """Genera un reporte Markdown. Requiere confirmación desde el agente."""
    title = _safe_markdown_text(title, max_len=120) or "Reporte de cálculos"

    if not isinstance(calculations, list) or not calculations:
        return {"error": "Debes proporcionar al menos un cálculo para el reporte."}

    os.makedirs("reports", exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    file_path = os.path.join("reports", f"reporte_{timestamp}.md")

    lines = [
        f"# {title}",
        "",
        f"**Generado:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}",
        f"**Total de resultados:** {len(calculations)}",
        "",
        "---",
        "",
        "## Resultados",
        ""
    ]

    for index, item in enumerate(calculations, start=1):
        description = _safe_markdown_text(item.get("description", "Sin descripción"), max_len=300)
        result = _safe_markdown_text(item.get("result", "N/A"), max_len=300)
        lines.append(f"### {index}. {description}")
        lines.append("")
        lines.append(f"**Resultado:** `{result}`")
        lines.append("")

    lines.append("---")
    lines.append("")
    lines.append("*Reporte generado por el agente del Laboratorio 8.*")
    lines.append("")

    with open(file_path, "w", encoding="utf-8") as file:
        file.write("\n".join(lines))

    return {
        "status": "success",
        "file_path": file_path,
        "title": title,
        "calculations_count": len(calculations),
        "message": f"Reporte guardado en {file_path}"
    }


TOOL_FUNCTIONS = {
    "calculate_expression": calculate_expression,
    "get_currency_exchange": get_currency_exchange,
    "query_statistics": query_statistics,
    "search_wikipedia_summary": search_wikipedia_summary,
    "create_calculation_report": create_calculation_report,
}
EOF
```

**✅ Validación del paso:**

```bash
python -m py_compile tools_impl.py
```

**📌 Resultado esperado:**

El comando no debe mostrar errores.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20una%20herramienta%20matem%C3%A1tica%20puede%20ser%20insegura%20aunque%20no%20use%20eval%28%29%2C%20y%20qu%C3%A9%20controles%20debo%20aplicar%20para%20evitar%20c%C3%A1lculos%20costosos%20o%20entradas%20maliciosas.)

---

# 🧩 Tarea 5. Probar herramientas sin agente

## 🎯 Objetivo de la tarea

Validar cada herramienta de forma aislada antes de conectarla al modelo.

---

### ✅ Paso 5.1. Crear `test_tools.py`

**📝 Descripción del paso:**

Crea el archivo `test_tools.py` en la raíz del proyecto y ejecútalo. Este script valida las herramientas de forma aislada, antes de conectarlas al agente y al modelo, para detectar errores de implementación sin depender de Function Calling.

```bash
cat > test_tools.py << 'EOF'
from tools_schema import TOOLS
from tools_impl import (
    TOOL_FUNCTIONS,
    calculate_expression,
    get_currency_exchange,
    query_statistics,
    search_wikipedia_summary,
)


def check(condition: bool, message: str) -> None:
    if condition:
        print(f"✅ {message}")
    else:
        raise AssertionError(message)


print("=== Validación de herramientas ===")

schema_names = {tool["function"]["name"] for tool in TOOLS}
impl_names = set(TOOL_FUNCTIONS.keys())
check(schema_names == impl_names, "Los schemas coinciden con las funciones implementadas")

math_ok = calculate_expression("(100 - 20) / 4 + 2 ** 3")
check("result" in math_ok and math_ok["result"] == 28.0, "Cálculo matemático válido")

math_danger = calculate_expression('__import__("os").system("dir")')
check("error" in math_danger, "Expresión peligrosa rechazada")

math_expensive = calculate_expression("999999 ** 999")
check("error" in math_expensive, "Cálculo costoso rechazado")

stats_sum = query_statistics("ventas", "sum", "ventas")
check("result" in stats_sum and stats_sum["result"] > 0, "Estadística sum sobre ventas")

stats_count = query_statistics("temperaturas", "count", "temp_max")
check("result" in stats_count and stats_count["result"] == 5, "Estadística count sobre temperaturas")

exchange = get_currency_exchange("USD", "MXN")
check("rate" in exchange or "error" in exchange, "Tipo de cambio responde con tasa o error controlado")

wiki = search_wikipedia_summary("Pi", "es")
check("summary" in wiki or "error" in wiki, "Wikipedia responde con resumen o error controlado")

print("\nTodas las herramientas fueron validadas correctamente.")
EOF
```
```bash
python test_tools.py
```

**📌 Resultado esperado:**

```text
Todas las herramientas fueron validadas correctamente.
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20debo%20probar%20las%20herramientas%20de%20un%20agente%20de%20forma%20aislada%20antes%20de%20conectarlas%20al%20modelo.)

---

# 🧩 Tarea 6. Implementar el agente con Function Calling

## 🎯 Objetivo de la tarea

Construir `math_data_agent.py` con el bucle agentico, dispatcher, trazabilidad, límite de iteraciones y confirmación humana.

---

### ✅ Paso 6.1. Crear `math_data_agent.py`

**📝 Descripción del paso:**

Crea el archivo `math_data_agent.py` en la raíz del proyecto. Este será el archivo principal del laboratorio y contendrá el bucle agentico, el cliente de OpenAI, el dispatcher de herramientas, la trazabilidad y la confirmación humana. Implementas el agente completo con:

- cliente OpenAI,
- mensajes `system` y `user`,
- `tools`,
- ejecución de tool calls,
- resultados con rol `tool`,
- límite de iteraciones,
- trazabilidad,
- ejecución paralela para herramientas seguras,
- confirmación humana para reportes.


```bash
cat > math_data_agent.py << 'EOF'
"""
Agente conversacional con Function Calling.
Implementa herramientas, trazabilidad, límite de iteraciones y Human-in-the-Loop.
"""

from __future__ import annotations

import json
import logging
import os
import sys
from concurrent.futures import ThreadPoolExecutor, as_completed
from dataclasses import dataclass, field
from typing import Any

from dotenv import load_dotenv
from openai import APIConnectionError, APIStatusError, APITimeoutError, OpenAI, RateLimitError
from tenacity import retry, retry_if_exception_type, stop_after_attempt, wait_exponential

from tools_impl import TOOL_FUNCTIONS
from tools_schema import TOOLS

load_dotenv()

os.makedirs("logs", exist_ok=True)

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
    handlers=[
        logging.FileHandler("logs/agent.log", encoding="utf-8"),
        logging.StreamHandler(sys.stdout),
    ],
)

logger = logging.getLogger("math_data_agent")

MODEL = os.getenv("OPENAI_MODEL", "gpt-4o-mini")
MAX_ITERATIONS = 10
TOOLS_REQUIRING_CONFIRMATION = {"create_calculation_report"}

SYSTEM_PROMPT = """Eres un agente asistente especializado en análisis matemático y consulta de datos.

Tienes acceso a herramientas para:
- calcular expresiones matemáticas seguras,
- consultar tipos de cambio,
- calcular estadísticas sobre CSV locales,
- buscar resúmenes en Wikipedia,
- crear reportes Markdown cuando el usuario lo pida explícitamente.

Instrucciones:
- Usa herramientas solo cuando aporten precisión o datos necesarios.
- Si necesitas varios datos independientes, puedes solicitarlos en la misma iteración.
- Explica los pasos principales de forma clara.
- Si una herramienta devuelve error, informa el problema y sugiere alternativa.
- No inventes resultados numéricos si pueden calcularse con una herramienta.
- No uses create_calculation_report salvo que el usuario pida guardar, exportar o generar un reporte.
- Responde siempre en español.
"""


@dataclass
class AgentRunResult:
    answer: str
    tools_used: list[str] = field(default_factory=list)
    iterations: int = 0
    tool_trace: list[dict] = field(default_factory=list)


def _parse_tool_arguments(raw_arguments: str) -> dict:
    try:
        return json.loads(raw_arguments or "{}")
    except json.JSONDecodeError as exc:
        raise ValueError(f"Argumentos JSON inválidos: {exc}") from exc


def _request_human_confirmation(tool_name: str, arguments: dict) -> bool:
    print("\n" + "=" * 70)
    print("⚠️  CONFIRMACIÓN REQUERIDA")
    print(f"El agente quiere ejecutar la herramienta: {tool_name}")
    print("Argumentos:")
    print(json.dumps(arguments, indent=2, ensure_ascii=False))
    print("Esta acción puede escribir archivos en disco.")
    print("=" * 70)

    while True:
        response = input("¿Confirmas la ejecución? [s/n]: ").strip().lower()
        if response in {"s", "si", "sí", "y", "yes"}:
            logger.info("[HITL] Usuario confirmó herramienta %s", tool_name)
            return True
        if response in {"n", "no"}:
            logger.info("[HITL] Usuario canceló herramienta %s", tool_name)
            return False
        print("Responde con 's' o 'n'.")


def _execute_tool_call(tool_call: Any, require_confirmation: bool = True) -> tuple[str, str, dict]:
    tool_name = tool_call.function.name

    try:
        arguments = _parse_tool_arguments(tool_call.function.arguments)
    except ValueError as exc:
        result = {"error": str(exc)}
        return tool_call.id, tool_name, result

    logger.info("[TOOL_CALL] id=%s | name=%s | args=%s", tool_call.id, tool_name, arguments)

    if tool_name not in TOOL_FUNCTIONS:
        return tool_call.id, tool_name, {"error": f"Herramienta no registrada: {tool_name}"}

    if require_confirmation and tool_name in TOOLS_REQUIRING_CONFIRMATION:
        if not _request_human_confirmation(tool_name, arguments):
            return tool_call.id, tool_name, {
                "status": "cancelled",
                "message": "El usuario canceló la ejecución de la herramienta."
            }

    try:
        result = TOOL_FUNCTIONS[tool_name](**arguments)
        logger.info("[TOOL_RESULT] id=%s | name=%s | result=%s", tool_call.id, tool_name, str(result)[:300])
        return tool_call.id, tool_name, result
    except TypeError as exc:
        return tool_call.id, tool_name, {"error": f"Argumentos incorrectos para {tool_name}: {exc}"}
    except Exception as exc:
        logger.exception("[TOOL_ERROR] id=%s | name=%s", tool_call.id, tool_name)
        return tool_call.id, tool_name, {"error": f"Error inesperado en {tool_name}: {exc}"}


def _execute_tool_calls(tool_calls: list[Any]) -> list[tuple[str, str, dict]]:
    """Ejecuta tool calls. Las herramientas con confirmación se ejecutan secuencialmente."""
    results: list[tuple[str, str, dict]] = []

    confirmation_calls = [tc for tc in tool_calls if tc.function.name in TOOLS_REQUIRING_CONFIRMATION]
    safe_calls = [tc for tc in tool_calls if tc.function.name not in TOOLS_REQUIRING_CONFIRMATION]

    if safe_calls:
        with ThreadPoolExecutor(max_workers=min(4, len(safe_calls))) as executor:
            futures = [executor.submit(_execute_tool_call, tool_call, False) for tool_call in safe_calls]
            for future in as_completed(futures):
                results.append(future.result())

    for tool_call in confirmation_calls:
        results.append(_execute_tool_call(tool_call, True))

    return results


@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=30),
    retry=retry_if_exception_type((RateLimitError, APIConnectionError, APITimeoutError, APIStatusError)),
    reraise=True,
)
def _call_llm(client: OpenAI, messages: list[dict]) -> Any:
    return client.chat.completions.create(
        model=MODEL,
        messages=messages,
        tools=TOOLS,
        tool_choice="auto",
        temperature=0.1,
        max_tokens=1600,
    )


def run_agent(user_query: str, client: OpenAI) -> AgentRunResult:
    messages: list[Any] = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": user_query},
    ]

    tools_used: list[str] = []
    tool_trace: list[dict] = []

    logger.info("[AGENT] Nueva consulta: %s", user_query)

    for iteration in range(1, MAX_ITERATIONS + 1):
        logger.info("[AGENT] Iteración %d/%d", iteration, MAX_ITERATIONS)

        response = _call_llm(client, messages)
        assistant_message = response.choices[0].message
        finish_reason = response.choices[0].finish_reason

        messages.append(assistant_message)

        tool_calls = assistant_message.tool_calls or []
        logger.info("[AGENT] finish_reason=%s | tool_calls=%d", finish_reason, len(tool_calls))

        if not tool_calls:
            answer = assistant_message.content or "El agente terminó sin texto final."
            return AgentRunResult(
                answer=answer,
                tools_used=tools_used,
                iterations=iteration,
                tool_trace=tool_trace,
            )

        execution_results = _execute_tool_calls(tool_calls)

        for tool_call_id, tool_name, result in execution_results:
            tools_used.append(tool_name)
            tool_trace.append({
                "tool_call_id": tool_call_id,
                "tool_name": tool_name,
                "result_preview": str(result)[:300],
            })
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call_id,
                "content": json.dumps(result, ensure_ascii=False, default=str),
            })

    raise RuntimeError(f"El agente excedió el límite de {MAX_ITERATIONS} iteraciones.")


def run_agent_text(user_query: str, client: OpenAI) -> str:
    return run_agent(user_query, client).answer


def main() -> None:
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key or api_key.startswith("pega_aqui"):
        print("ERROR: configura OPENAI_API_KEY en el archivo .env")
        raise SystemExit(1)

    client = OpenAI(api_key=api_key)

    print("\n" + "=" * 72)
    print("🤖 Math & Data Agent — Function Calling")
    print(f"Modelo: {MODEL}")
    print(f"Herramientas: {len(TOOLS)}")
    print(f"Máximo de iteraciones: {MAX_ITERATIONS}")
    print("Comandos: salir | exit | quit")
    print("=" * 72)

    while True:
        try:
            query = input("\nTú: ").strip()
        except (EOFError, KeyboardInterrupt):
            print("\nHasta luego.")
            break

        if not query:
            continue

        if query.lower() in {"salir", "exit", "quit"}:
            print("Hasta luego.")
            break

        try:
            result = run_agent(query, client)
            print("\nAgente:")
            print(result.answer)
            print("\nTrazabilidad:")
            print(f"- Iteraciones: {result.iterations}")
            print(f"- Herramientas usadas: {result.tools_used if result.tools_used else 'ninguna'}")
        except Exception as exc:
            logger.exception("Error ejecutando agente")
            print(f"Error: {exc}")


if __name__ == "__main__":
    main()
EOF
```

**✅ Validación del paso:**

```bash
python -m py_compile math_data_agent.py
```
```bash
python math_data_agent.py
```

Escribe:

```text
salir
```

**📌 Resultado esperado:**

El agente debe iniciar y salir sin errores.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20el%20ciclo%20completo%20de%20Function%20Calling%3A%20mensaje%20del%20usuario%2C%20tool_calls%20del%20asistente%2C%20ejecuci%C3%B3n%20de%20herramienta%2C%20mensaje%20tool%20y%20respuesta%20final.)

---

# 🧩 Tarea 7. Ejecutar consultas interactivas

## 🎯 Objetivo de la tarea

Probar manualmente que el agente decide cuándo usar herramientas.

---

### ✅ Paso 7.1. Ejecutar el agente

**📝 Descripción del paso:**

Ejecuta el archivo principal `math_data_agent.py` desde Git Bash. Esto abre la consola interactiva del agente para que puedas escribir consultas en lenguaje natural y observar qué herramientas decide usar.

```bash
python math_data_agent.py
```

### ✅ Paso 7.2. Consulta matemática simple

**📝 Descripción del paso:**

Con el agente abierto en la terminal, escribe la siguiente consulta como entrada del usuario. No debes crear ni editar archivos en este paso; solo probarás si el agente decide usar la herramienta matemática.

Escribe:

```text
¿Cuánto es (100 - 20) / 4 + 2 ** 3?
```

**📌 Resultado esperado:**

El agente debe usar `calculate_expression` y responder que el resultado es `28`.

---

### ✅ Paso 7.3. Consulta de datos locales

**📝 Descripción del paso:**

Mantén abierto el agente en la terminal y escribe la siguiente consulta. El agente deberá leer los CSV que generaste en la carpeta `data/` mediante la herramienta `query_statistics`.

Escribe:

```text
Del dataset de ventas, calcula la suma total de ventas y el promedio de costos. Explica el resultado.
```

**📌 Resultado esperado:**

El agente debe usar `query_statistics`, posiblemente más de una vez, y devolver suma de ventas y promedio de costos.

---

### ✅ Paso 7.4. Consulta con herramientas combinadas

**📝 Descripción del paso:**

Mantén abierto `math_data_agent.py` y escribe la consulta indicada. En este paso no editas código; validas que el agente combine datos locales, tipo de cambio y cálculo matemático en una misma respuesta.

Escribe:

```text
Convierte la suma total de ventas del dataset ventas de USD a MXN usando el tipo de cambio actual. Explica los pasos.
```

**📌 Resultado esperado:**

El agente debe usar:

- `query_statistics`
- `get_currency_exchange`
- `calculate_expression`

---

### ✅ Paso 7.5. Consulta con Wikipedia y cálculo

**📝 Descripción del paso:**

Con el agente en ejecución, escribe la consulta indicada. El agente deberá consultar información contextual con `search_wikipedia_summary` y calcular el área con `calculate_expression`.

Escribe:

```text
Busca qué es Pi y calcula el área de un círculo con radio 7.5 usando Pi = 3.14159265.
```

**📌 Resultado esperado:**

El agente debe usar:

- `search_wikipedia_summary`
- `calculate_expression`

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20las%20herramientas%20con%20efectos%20secundarios%20deben%20requerir%20confirmaci%C3%B3n%20humana%20en%20sistemas%20agenticos.)

---


# 🧩 Tarea 8. Probar Human-in-the-Loop

## 🎯 Objetivo de la tarea

Validar que el agente solicita confirmación antes de escribir un archivo.

---

### ✅ Paso 8.1. Solicitar reporte

**📝 Descripción del paso:**

Ejecuta `math_data_agent.py` desde Git Bash y escribe la consulta indicada en la terminal interactiva. El agente debe calcular el área y después intentar usar `create_calculation_report`. Como esa herramienta escribe un archivo Markdown dentro de `reports/`, debe pedir confirmación antes de continuar.

Ejecuta el agente:

```bash
python math_data_agent.py
```

Escribe:

```text
Calcula el área de un círculo con radio 5 usando Pi = 3.14159. Luego guarda un reporte con ese cálculo.
```

### Validación 1 — Cancelar

Cuando pregunte:

```text
¿Confirmas la ejecución? [s/n]:
```

Responde:

```text
n
```

**📌 Resultado esperado:**

El agente debe informar que el cálculo se realizó, pero que el reporte no fue guardado porque cancelaste la operación.

---

### Validación 2 — Confirmar

Repite la consulta y responde:

```text
s
```

**✅ Validación del paso:**

```bash
ls -la reports
```

**📌 Resultado esperado:**

Debe existir un archivo parecido a:

```text
reporte_YYYYMMDD_HHMMSS.md
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20una%20prueba%20de%20agente%20debe%20validar%20las%20herramientas%20usadas%20y%20no%20solo%20el%20texto%20final%20de%20la%20respuesta.)

---

# 🧩 Tarea 9. Crear suite de pruebas con trazabilidad

## 🎯 Objetivo de la tarea

Validar no solo la respuesta final, sino también las herramientas que el agente decidió usar.

---

### ✅ Paso 9.1. Crear `test_agent_queries.py`

**📝 Descripción del paso:**

Crea el archivo `test_agent_queries.py` en la raíz del proyecto. Esta prueba valida comportamiento real: respuesta generada y herramientas utilizadas. Esto evita falsos positivos donde el agente responde sin usar las herramientas necesarias.


```bash
cat > test_agent_queries.py << 'EOF'
import os
from dotenv import load_dotenv
from openai import OpenAI

from math_data_agent import run_agent

load_dotenv()

TEST_QUERIES = [
    {
        "id": 1,
        "name": "Matemática simple",
        "query": "Calcula (100 - 20) / 4 + 2 ** 3.",
        "expected_tools": {"calculate_expression"},
        "must_contain": ["28"],
    },
    {
        "id": 2,
        "name": "Tipo de cambio y cálculo",
        "query": "Si tengo 5000 USD, conviértelos a EUR y calcula el 15% de ese monto en EUR.",
        "expected_tools": {"get_currency_exchange", "calculate_expression"},
        "must_contain": ["EUR"],
    },
    {
        "id": 3,
        "name": "Estadísticas y cálculo",
        "query": "Del dataset ventas calcula la media de ventas, la media de costos y el margen promedio como porcentaje.",
        "expected_tools": {"query_statistics", "calculate_expression"},
        "must_contain": ["margen"],
    },
    {
        "id": 4,
        "name": "Wikipedia y cálculo",
        "query": "Busca qué es Pi y calcula el área de un círculo de radio 7.5 usando Pi = 3.14159265.",
        "expected_tools": {"search_wikipedia_summary", "calculate_expression"},
        "must_contain": ["área", "Pi"],
    },
    {
        "id": 5,
        "name": "Datos, moneda y cálculo",
        "query": "Suma las ventas del dataset ventas y convierte ese total de USD a MXN.",
        "expected_tools": {"query_statistics", "get_currency_exchange", "calculate_expression"},
        "must_contain": ["MXN"],
    },
]


def main() -> int:
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key or api_key.startswith("pega_aqui"):
        print("Configura OPENAI_API_KEY en .env antes de ejecutar pruebas.")
        return 1

    client = OpenAI(api_key=api_key)

    print("\n=== Suite de pruebas del agente ===")

    passed = 0
    rows = []

    for test in TEST_QUERIES:
        print(f"\n[{test['id']}] {test['name']}")
        result = run_agent(test["query"], client)

        tools_used = set(result.tools_used)
        expected_tools = test["expected_tools"]
        missing_tools = expected_tools - tools_used
        contains_expected = any(term.lower() in result.answer.lower() for term in test["must_contain"])
        ok = not missing_tools and contains_expected and len(result.answer.strip()) > 40

        if ok:
            passed += 1
            status = "PASS"
        else:
            status = "PARTIAL"

        print(f"Estado: {status}")
        print(f"Herramientas esperadas: {sorted(expected_tools)}")
        print(f"Herramientas usadas:    {result.tools_used}")
        print(f"Iteraciones: {result.iterations}")
        print(f"Respuesta: {result.answer[:250]}...")

        rows.append({
            "id": test["id"],
            "consulta": test["name"],
            "esperadas": ", ".join(sorted(expected_tools)),
            "usadas": ", ".join(result.tools_used),
            "estado": status,
        })

    with open("matriz_evaluacion.md", "w", encoding="utf-8") as file:
        file.write("# Matriz de evaluación — Laboratorio 8\n\n")
        file.write("| ID | Consulta | Herramientas esperadas | Herramientas usadas | Estado | Observaciones |\n")
        file.write("|---|---|---|---|---|---|\n")
        for row in rows:
            file.write(
                f"| {row['id']} | {row['consulta']} | {row['esperadas']} | {row['usadas']} | {row['estado']} |  |\n"
            )

    print("\n=== Resumen ===")
    print(f"Pruebas exitosas: {passed}/{len(TEST_QUERIES)}")
    print("Matriz generada: matriz_evaluacion.md")

    return 0 if passed >= 4 else 1


if __name__ == "__main__":
    raise SystemExit(main())
EOF
```

### ✅ Paso 9.2. Ejecutar pruebas

**📝 Descripción del paso:**

Ejecuta el archivo `test_agent_queries.py` desde Git Bash. Este script enviará varias consultas al agente y generará `matriz_evaluacion.md` con las herramientas esperadas y las herramientas realmente usadas.

```bash
python test_agent_queries.py
```

**📌 Resultado esperado:**

Debes obtener al menos:

```text
Pruebas exitosas: 4/5
Matriz generada: matriz_evaluacion.md
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20una%20prueba%20de%20agente%20debe%20validar%20las%20herramientas%20usadas%20y%20no%20solo%20el%20texto%20final%20de%20la%20respuesta.)

---

# 🧩 Tarea 10. Comparar agente vs. workflow

## 🎯 Objetivo de la tarea

Documentar cuándo conviene usar un agente y cuándo un workflow estructurado.

---

### ✅ Paso 10.1. Abrir matriz de evaluación

**📝 Descripción del paso:**

Abre o visualiza el archivo `matriz_evaluacion.md`, generado por la suite de pruebas. Puedes verlo rápidamente desde Git Bash con `cat` o abrirlo en VS Code para editarlo con más comodidad.

```bash
cat matriz_evaluacion.md
```

### ✅ Paso 10.2. Completar observaciones

**📝 Descripción del paso:**

Abre `matriz_evaluacion.md` en VS Code y edita únicamente la columna **Observaciones**. En esta parte documentarás si el agente eligió bien las herramientas, si hubo sobreuso o si el caso pudo resolverse con un workflow fijo.

Edita `matriz_evaluacion.md` en VS Code y completa la columna **Observaciones**.

Usa esta guía:

| Pregunta | Criterio |
|---|---|
| ¿La herramienta elegida fue correcta? | Evalúa selección dinámica |
| ¿El agente usó demasiadas herramientas? | Detecta sobreuso |
| ¿Pudo resolverse con workflow fijo? | Compara enfoques |
| ¿Hubo errores externos? | Evalúa resiliencia |
| ¿La respuesta fue clara? | Evalúa utilidad final |

---

### ✅ Paso 10.3. Agregar conclusión final

**📝 Descripción del paso:**

Edita el archivo `matriz_evaluacion.md` en VS Code y agrega la conclusión al final del documento. Esta conclusión funciona como cierre técnico de la comparación entre agente y workflow.

Agrega al final de `matriz_evaluacion.md`:

```markdown
## Conclusión

En este laboratorio observé que un agente es útil cuando la consulta puede requerir diferentes herramientas y el orden no siempre es predecible. Sin embargo, un workflow puede ser preferible cuando el proceso es repetitivo, regulado o requiere máxima determinismo.
```

**📌 Resultado esperado:**

Debes tener una matriz documentada con evidencia de herramientas esperadas, herramientas usadas y observaciones.


---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar esta tarea en ChatGPT](https://chatgpt.com/?q=Ay%C3%BAdame%20a%20comparar%20cu%C3%A1ndo%20conviene%20un%20agente%20y%20cu%C3%A1ndo%20conviene%20un%20workflow%20para%20tareas%20empresariales%20con%20IA%20Generativa.)

---

# ✅ Validación final del laboratorio

Ejecuta esta secuencia completa:

```bash
python -m py_compile tools_schema.py
python -m py_compile tools_impl.py
python -m py_compile math_data_agent.py
python -m py_compile test_tools.py
python -m py_compile test_agent_queries.py
python test_tools.py
python test_agent_queries.py
ls -la logs
ls -la data
```

## Resultado esperado

Debes confirmar:

| Validación | Resultado esperado |
|---|---|
| Compilación de scripts | Sin errores |
| Herramientas aisladas | Todas pasan |
| Pruebas del agente | Al menos 4/5 exitosas |
| Logs | `logs/agent.log` creado |
| Matriz | `matriz_evaluacion.md` creada |
| Reporte opcional | Solo si confirmaste Human-in-the-Loop |

---

# 📦 Entregables del laboratorio

Entrega estos archivos:

| Archivo | Obligatorio | Descripción |
|---|---|---|
| `tools_schema.py` | Sí | Esquemas JSON Schema de herramientas |
| `tools_impl.py` | Sí | Implementación de herramientas |
| `math_data_agent.py` | Sí | Agente principal |
| `test_tools.py` | Sí | Pruebas unitarias de herramientas |
| `test_agent_queries.py` | Sí | Pruebas con trazabilidad |
| `matriz_evaluacion.md` | Sí | Evaluación agente vs workflow |
| `reports/*.md` | Opcional | Reporte generado si confirmaste HITL |

No entregues:

```text
.env
.venv/
logs/
data/*.csv si contienen datos sensibles
```

---

# 📊 Criterios de evaluación

| Criterio | Peso |
|---|---:|
| Proyecto local y entorno correctamente configurados | 10% |
| Schemas JSON definidos correctamente | 10% |
| Herramientas implementadas con validaciones | 15% |
| Evaluación matemática segura sin `eval()` | 10% |
| Agente con bucle Function Calling funcional | 20% |
| Human-in-the-Loop implementado correctamente | 10% |
| Pruebas con trazabilidad de herramientas usadas | 15% |
| Matriz agente vs workflow documentada | 10% |

---

# ⚠️ Errores comunes y solución

## Error 1 — `OPENAI_API_KEY` no encontrada

### Síntoma

```text
ERROR: configura OPENAI_API_KEY en el archivo .env
```

### Solución

Edita `.env`:

```env
OPENAI_API_KEY=tu_api_key_real
OPENAI_MODEL=gpt-4o-mini
USE_OFFLINE_FALLBACKS=true
```

Luego ejecuta:

```bash
python math_data_agent.py
```

---

## Error 2 — El modelo no usa la herramienta esperada

### Posibles causas

- La consulta es ambigua.
- El modelo cree que puede responder sin herramienta.
- La descripción del schema no es suficientemente clara.

### Solución

Reformula la consulta:

```text
Usa el dataset ventas para calcular la suma de la columna ventas y después convierte ese resultado a MXN.
```

---

## Error 3 — Frankfurter o Wikipedia no responden

### Síntoma

```text
No se pudo obtener tipo de cambio
```

### Solución

Verifica que `.env` tenga:

```env
USE_OFFLINE_FALLBACKS=true
```

El laboratorio podrá usar tasas o resúmenes locales de referencia.

---

## Error 4 — Cálculo rechazado

### Síntoma

```text
Expresión inválida o no permitida
```

### Causa

La herramienta rechaza expresiones peligrosas o costosas, por ejemplo:

```python
999999 ** 999
__import__("os").system("dir")
```

### Solución

Usa expresiones matemáticas simples:

```text
(100 - 20) / 4 + 2 ** 3
```

---

## Error 5 — La prueba marca `PARTIAL`

### Causa

La respuesta puede ser correcta, pero el modelo no usó todas las herramientas esperadas.

### Solución

Revisa:

```bash
cat logs/agent.log
cat matriz_evaluacion.md
```

Evalúa si faltó una herramienta o si la consulta podía resolverse de otra manera.

---

# 🧹 Limpieza del entorno

Cuando termines, puedes limpiar archivos generados:

```bash
rm -rf reports/
rm -f logs/agent.log
rm -f crear_datasets.py
```

Para salir del entorno virtual:

```bash
deactivate
```

Para eliminar el entorno completo:

```bash
rm -rf .venv/
```

Antes de subir a Git:

```bash
git status
grep -r "sk-" . --include="*.py" --include="*.md" --include="*.txt" 2>/dev/null || echo "Sin API keys detectadas"
```

---

# 🏁 Resultado final esperado

Al finalizar, habrás construido un agente que:

- Recibe consultas en lenguaje natural.
- Decide qué herramientas necesita.
- Ejecuta herramientas seguras.
- Usa datos locales y fuentes externas.
- Controla acciones con efecto secundario.
- Registra trazabilidad.
- Devuelve respuestas en español.
- Permite evaluar si el comportamiento fue agentico o si bastaba un workflow.

---

# 📚 Cierre conceptual

En este laboratorio construiste un agente práctico con Function Calling. El valor principal no está solo en conectar funciones, sino en diseñar controles alrededor del agente:

- límites de iteración,
- validación de herramientas,
- manejo de errores,
- fallbacks,
- Human-in-the-Loop,
- trazabilidad,
- evaluación de herramientas usadas.

Ese conjunto de controles es lo que convierte una demostración de agente en una base más cercana a una solución empresarial.
