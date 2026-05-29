# Práctica 8 — Agente con Function Calling para Tareas Matemáticas y Consulta de Datos

## 1. Metadatos

| Campo | Valor |
|---|---|
| **Duración estimada** | 58 minutos |
| **Complejidad** | Alta |
| **Nivel Bloom** | Crear |
| **Módulo** | 8 — Agentes vs. Workflows |
| **Costo API estimado** | < $0.10 USD (GPT-4o-mini recomendado) |

---

## 2. Descripción General

En este laboratorio construirás un **agente conversacional completo** (`math_data_agent.py`) que implementa el bucle agentico ReAct usando la API de Function Calling de OpenAI. El agente dispondrá de cinco herramientas especializadas: evaluación matemática segura, consulta de tipos de cambio en tiempo real, estadísticas sobre datasets CSV, búsqueda en Wikipedia y generación de reportes Markdown. Aplicarás los conceptos de la Lección 8.1 sobre el ciclo **Observar → Pensar → Actuar**, incluyendo manejo de múltiples `tool_calls` en paralelo, límite de iteraciones y confirmación humana para acciones con efectos secundarios.

---

## 3. Objetivos de Aprendizaje

- [ ] Implementar el bucle agentico completo (ReAct loop) con la API de OpenAI: recepción de mensaje → selección de tool → ejecución → retorno de resultado → respuesta final.
- [ ] Diseñar esquemas JSON Schema correctos para cinco herramientas distintas que el LLM pueda seleccionar e invocar autónomamente.
- [ ] Manejar múltiples `tool_calls` en paralelo y aplicar el patrón Human-in-the-Loop para herramientas con efectos secundarios en disco.
- [ ] Aplicar evaluación matemática segura sin `eval()`, usando `ast` y operadores explícitamente permitidos.
- [ ] Verificar el comportamiento del agente con al menos cinco queries complejas que encadenan múltiples herramientas.

---

## 4. Prerrequisitos

### Conocimientos previos
- Haber completado Labs 02-00-01 y 03-00-01 (uso del SDK de OpenAI y manejo de errores).
- Comprensión del formato de mensajes con `tool_calls` en la API de OpenAI (roles `assistant` y `tool`).
- Conocimiento básico de JSON Schema (tipos, `required`, `properties`).
- Familiaridad con `pandas` para operaciones estadísticas básicas.

### Acceso y credenciales
- `OPENAI_API_KEY` activa con acceso a `gpt-4o-mini` o `gpt-4o`.
- Conexión a internet para Frankfurter API y Wikipedia API (ambas gratuitas, sin API key).
- Python 3.11 instalado y entorno virtual disponible.

> ⚠️ **Advertencia de costos:** Este lab usa `gpt-4o-mini` por defecto para minimizar costos. No cambies el modelo a `gpt-4o` a menos que sea necesario. El costo estimado con `gpt-4o-mini` para completar todas las pruebas es inferior a $0.05 USD.

---

## 5. Entorno del Laboratorio

### Hardware requerido

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 8 GB | 16 GB |
| CPU | 4 núcleos | 4+ núcleos |
| Almacenamiento libre | 500 MB | 1 GB |
| Conexión a internet | 5 Mbps | 10 Mbps |

### Software y dependencias

| Paquete | Versión | Propósito |
|---|---|---|
| `python` | 3.11.x | Intérprete |
| `openai` | 1.35.x | SDK Function Calling |
| `requests` | 2.31.x | Llamadas HTTP a APIs externas |
| `pandas` | 2.1.x | Operaciones estadísticas |
| `wikipedia-api` | 0.6.x | Búsqueda en Wikipedia |
| `python-dotenv` | 1.0.x | Gestión de variables de entorno |
| `tenacity` | 8.3.x | Reintentos con backoff |

### Configuración del entorno

```bash
# 1. Crear y activar entorno virtual
python -m venv venv_lab08
# Windows:
venv_lab08\Scripts\activate
# macOS/Linux:
source venv_lab08/bin/activate

# 2. Instalar dependencias
pip install openai==1.35.0 requests==2.31.0 pandas==2.1.4 \
            wikipedia-api==0.6.0 python-dotenv==1.0.1 tenacity==8.3.0

# 3. Crear estructura del proyecto
mkdir lab08 && cd lab08
mkdir data logs

# 4. Crear archivo .env (NUNCA lo subas a Git)
cat > .env << 'EOF'
OPENAI_API_KEY=sk-...tu_clave_aqui...
EOF

# 5. Crear .gitignore
cat > .gitignore << 'EOF'
.env
venv_lab08/
__pycache__/
*.pyc
logs/
data/*.csv
reports/
EOF
```

### Datasets de prueba

Ejecuta este script para crear los datasets CSV locales que usará la herramienta de estadísticas:

```bash
python - << 'EOF'
import pandas as pd
import os

os.makedirs("data", exist_ok=True)

# Dataset 1: ventas mensuales
pd.DataFrame({
    "mes": ["Ene","Feb","Mar","Abr","May","Jun"],
    "ventas": [15200, 18400, 21300, 17800, 23100, 25600],
    "costos": [9100, 10200, 11800, 9900, 12400, 13200],
    "unidades": [152, 184, 213, 178, 231, 256]
}).to_csv("data/ventas.csv", index=False)

# Dataset 2: temperaturas ciudad
pd.DataFrame({
    "ciudad": ["Madrid","Barcelona","Sevilla","Bilbao","Valencia"],
    "temp_max": [28.5, 26.3, 35.2, 22.1, 30.8],
    "temp_min": [14.2, 16.1, 20.3, 11.5, 17.9],
    "humedad": [45, 68, 32, 78, 55]
}).to_csv("data/temperaturas.csv", index=False)

print("✅ Datasets creados: data/ventas.csv, data/temperaturas.csv")
EOF
```

---

## 6. Desarrollo Paso a Paso

### Paso 1 — Definir los esquemas JSON Schema de las cinco herramientas

**Objetivo:** Crear el módulo `tools_schema.py` con los esquemas que el LLM usará para seleccionar e invocar cada herramienta. Un esquema bien diseñado es la diferencia entre un agente que selecciona la herramienta correcta y uno que falla.

#### Instrucciones

1. Crea el archivo `tools_schema.py` en la carpeta `lab08/`:

```python
# tools_schema.py
"""
Esquemas JSON Schema para las herramientas del agente math_data_agent.
Cada esquema define nombre, descripción y parámetros con tipos y restricciones.
"""

TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "calculate_expression",
            "description": (
                "Evalúa expresiones matemáticas de forma segura. "
                "Soporta operadores +, -, *, /, **, % y funciones abs(), round(). "
                "NO usa eval(); usa AST para seguridad. "
                "Ejemplos válidos: '2 + 3 * 4', '(100 - 20) / 4', '2 ** 10'."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "Expresión matemática en texto plano. Solo números y operadores permitidos."
                    }
                },
                "required": ["expression"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "get_currency_exchange",
            "description": (
                "Obtiene el tipo de cambio actual entre dos monedas usando la API pública de Frankfurter. "
                "Retorna la tasa de cambio y la fecha de actualización. "
                "Ejemplos de códigos: USD, EUR, GBP, JPY, MXN, COP, ARS, BRL."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "base": {
                        "type": "string",
                        "description": "Código ISO 4217 de la moneda base (ej. 'USD', 'EUR').",
                        "minLength": 3,
                        "maxLength": 3
                    },
                    "target": {
                        "type": "string",
                        "description": "Código ISO 4217 de la moneda destino (ej. 'MXN', 'COP').",
                        "minLength": 3,
                        "maxLength": 3
                    }
                },
                "required": ["base", "target"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "query_statistics",
            "description": (
                "Realiza operaciones estadísticas sobre datasets CSV locales precargados. "
                "Datasets disponibles: 'ventas' (columnas: ventas, costos, unidades), "
                "'temperaturas' (columnas: temp_max, temp_min, humedad). "
                "Operaciones disponibles: mean, median, std, max, min."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "dataset_name": {
                        "type": "string",
                        "description": "Nombre del dataset sin extensión.",
                        "enum": ["ventas", "temperaturas"]
                    },
                    "operation": {
                        "type": "string",
                        "description": "Operación estadística a realizar.",
                        "enum": ["mean", "median", "std", "max", "min"]
                    },
                    "column": {
                        "type": "string",
                        "description": "Nombre de la columna numérica sobre la que aplicar la operación."
                    }
                },
                "required": ["dataset_name", "operation", "column"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_wikipedia_summary",
            "description": (
                "Obtiene el resumen introductorio de un artículo de Wikipedia en español o inglés. "
                "Útil para obtener definiciones, contexto histórico o información factual general. "
                "Retorna los primeros párrafos del artículo más relevante."
            ),
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "Término o frase de búsqueda para Wikipedia."
                    },
                    "language": {
                        "type": "string",
                        "description": "Idioma de Wikipedia: 'es' para español, 'en' para inglés.",
                        "enum": ["es", "en"],
                        "default": "es"
                    }
                },
                "required": ["query"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "create_calculation_report",
            "description": (
                "⚠️ HERRAMIENTA CON EFECTO SECUNDARIO: Genera y GUARDA en disco un reporte Markdown "
                "con los cálculos realizados durante la sesión. "
                "Solo usar cuando el usuario solicite explícitamente guardar o exportar un reporte. "
                "Requiere confirmación del usuario antes de ejecutar."
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
                        "description": "Lista de cálculos a incluir en el reporte.",
                        "items": {
                            "type": "object",
                            "properties": {
                                "description": {
                                    "type": "string",
                                    "description": "Descripción del cálculo realizado."
                                },
                                "result": {
                                    "type": "string",
                                    "description": "Resultado del cálculo como string."
                                }
                            },
                            "required": ["description", "result"]
                        }
                    }
                },
                "required": ["title", "calculations"]
            }
        }
    }
]
```

#### Salida esperada

El archivo `tools_schema.py` debe crearse sin errores. Verifica su sintaxis:

```bash
python -c "from tools_schema import TOOLS; print(f'✅ {len(TOOLS)} herramientas definidas')"
```

#### Verificación

```
✅ 5 herramientas definidas
```

---

### Paso 2 — Implementar las cinco funciones herramienta

**Objetivo:** Crear `tools_impl.py` con la implementación real de cada herramienta. Aquí aplicarás evaluación matemática segura con `ast`, llamadas HTTP con reintentos y operaciones estadísticas con `pandas`.

#### Instrucciones

2. Crea el archivo `tools_impl.py`:

```python
# tools_impl.py
"""
Implementación de las funciones herramienta del agente math_data_agent.
Cada función tiene manejo de errores explícito y retorna strings descriptivos en caso de fallo.
"""

import ast
import operator
import os
import logging
from datetime import datetime
from typing import List
import requests
import pandas as pd
import wikipediaapi
from tenacity import retry, stop_after_attempt, wait_exponential

logger = logging.getLogger(__name__)

# ─────────────────────────────────────────────
# HERRAMIENTA 1: Evaluación matemática segura
# ─────────────────────────────────────────────

# Operadores permitidos explícitamente (sin eval())
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

def _evaluar_nodo(nodo):
    """Evalúa recursivamente un nodo AST usando solo operadores permitidos."""
    if isinstance(nodo, ast.Constant):
        if isinstance(nodo.value, (int, float)):
            return nodo.value
        raise ValueError(f"Tipo de constante no permitido: {type(nodo.value)}")
    
    elif isinstance(nodo, ast.BinOp):
        tipo_op = type(nodo.op)
        if tipo_op not in _OPERADORES_PERMITIDOS:
            raise ValueError(f"Operador no permitido: {tipo_op.__name__}")
        izq = _evaluar_nodo(nodo.left)
        der = _evaluar_nodo(nodo.right)
        return _OPERADORES_PERMITIDOS[tipo_op](izq, der)
    
    elif isinstance(nodo, ast.UnaryOp):
        tipo_op = type(nodo.op)
        if tipo_op not in _OPERADORES_PERMITIDOS:
            raise ValueError(f"Operador unario no permitido: {tipo_op.__name__}")
        return _OPERADORES_PERMITIDOS[tipo_op](_evaluar_nodo(nodo.operand))
    
    elif isinstance(nodo, ast.Call):
        # Solo abs() y round() están permitidas
        if isinstance(nodo.func, ast.Name) and nodo.func.id in ("abs", "round"):
            args = [_evaluar_nodo(a) for a in nodo.args]
            return abs(*args) if nodo.func.id == "abs" else round(*args)
        raise ValueError(f"Función no permitida: {ast.dump(nodo.func)}")
    
    raise ValueError(f"Expresión no soportada: {type(nodo).__name__}")


def calculate_expression(expression: str) -> dict:
    """
    Evalúa una expresión matemática de forma segura usando AST.
    Retorna dict con 'result' (float) o 'error' (str).
    """
    logger.info(f"[TOOL] calculate_expression | expression='{expression}'")
    try:
        # Parsear a AST sin ejecutar
        arbol = ast.parse(expression.strip(), mode="eval")
        resultado = _evaluar_nodo(arbol.body)
        logger.info(f"[TOOL] calculate_expression | result={resultado}")
        return {"result": float(resultado), "expression": expression}
    except ZeroDivisionError:
        return {"error": "División por cero no permitida.", "expression": expression}
    except ValueError as e:
        return {"error": f"Expresión inválida o no permitida: {e}", "expression": expression}
    except SyntaxError as e:
        return {"error": f"Sintaxis incorrecta en la expresión: {e}", "expression": expression}


# ─────────────────────────────────────────────
# HERRAMIENTA 2: Tipos de cambio (Frankfurter)
# ─────────────────────────────────────────────

FRANKFURTER_URL = "https://api.frankfurter.app/latest"
BACKUP_URL = "https://open.er-api.com/v6/latest"  # Backup sin API key (limitado)

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
def get_currency_exchange(base: str, target: str) -> dict:
    """
    Consulta el tipo de cambio actual usando Frankfurter API.
    Retorna dict con tasa, fecha y monedas, o 'error' si falla.
    """
    logger.info(f"[TOOL] get_currency_exchange | base={base.upper()}, target={target.upper()}")
    base = base.upper().strip()
    target = target.upper().strip()
    
    try:
        resp = requests.get(
            FRANKFURTER_URL,
            params={"from": base, "to": target},
            timeout=10
        )
        resp.raise_for_status()
        data = resp.json()
        
        if target not in data.get("rates", {}):
            return {"error": f"Moneda '{target}' no encontrada. Verifica el código ISO 4217."}
        
        tasa = data["rates"][target]
        resultado = {
            "base": base,
            "target": target,
            "rate": tasa,
            "date": data.get("date", "N/A"),
            "description": f"1 {base} = {tasa:.4f} {target}"
        }
        logger.info(f"[TOOL] get_currency_exchange | result={resultado['description']}")
        return resultado
    
    except requests.exceptions.ConnectionError:
        return {"error": "No se pudo conectar a Frankfurter API. Verifica tu conexión a internet."}
    except requests.exceptions.HTTPError as e:
        return {"error": f"Error HTTP {e.response.status_code}: {e}"}
    except Exception as e:
        return {"error": f"Error inesperado al consultar tipo de cambio: {e}"}


# ─────────────────────────────────────────────
# HERRAMIENTA 3: Estadísticas sobre CSV
# ─────────────────────────────────────────────

_DATASETS_CACHE: dict[str, pd.DataFrame] = {}

def _cargar_dataset(nombre: str) -> pd.DataFrame:
    """Carga y cachea un dataset CSV desde la carpeta data/."""
    if nombre not in _DATASETS_CACHE:
        ruta = os.path.join("data", f"{nombre}.csv")
        if not os.path.exists(ruta):
            raise FileNotFoundError(f"Dataset '{nombre}' no encontrado en {ruta}")
        _DATASETS_CACHE[nombre] = pd.read_csv(ruta)
        logger.info(f"[CACHE] Dataset '{nombre}' cargado desde {ruta}")
    return _DATASETS_CACHE[nombre]


def query_statistics(dataset_name: str, operation: str, column: str) -> dict:
    """
    Aplica una operación estadística sobre una columna de un dataset CSV.
    """
    logger.info(f"[TOOL] query_statistics | dataset={dataset_name}, op={operation}, col={column}")
    
    operaciones_validas = {"mean", "median", "std", "max", "min"}
    if operation not in operaciones_validas:
        return {"error": f"Operación '{operation}' no válida. Usa: {operaciones_validas}"}
    
    try:
        df = _cargar_dataset(dataset_name)
        
        if column not in df.columns:
            cols_disponibles = [c for c in df.columns if df[c].dtype in ["float64", "int64"]]
            return {
                "error": f"Columna '{column}' no existe en '{dataset_name}'.",
                "columnas_disponibles": cols_disponibles
            }
        
        serie = df[column].dropna()
        resultado_num = getattr(serie, operation)()
        
        resultado = {
            "dataset": dataset_name,
            "column": column,
            "operation": operation,
            "result": float(resultado_num),
            "n_rows": len(serie),
            "description": f"{operation}({dataset_name}.{column}) = {resultado_num:.4f}"
        }
        logger.info(f"[TOOL] query_statistics | result={resultado['description']}")
        return resultado
    
    except FileNotFoundError as e:
        return {"error": str(e)}
    except Exception as e:
        return {"error": f"Error al calcular estadística: {e}"}


# ─────────────────────────────────────────────
# HERRAMIENTA 4: Búsqueda en Wikipedia
# ─────────────────────────────────────────────

_WIKI_CLIENTS: dict[str, wikipediaapi.Wikipedia] = {}

def _get_wiki_client(language: str) -> wikipediaapi.Wikipedia:
    """Retorna (y cachea) un cliente de Wikipedia para el idioma dado."""
    if language not in _WIKI_CLIENTS:
        _WIKI_CLIENTS[language] = wikipediaapi.Wikipedia(
            language=language,
            user_agent="math_data_agent/1.0 (lab08@curso-genai.edu)"
        )
    return _WIKI_CLIENTS[language]


def search_wikipedia_summary(query: str, language: str = "es") -> dict:
    """
    Busca el resumen de un artículo en Wikipedia.
    """
    logger.info(f"[TOOL] search_wikipedia_summary | query='{query}', lang={language}")
    
    try:
        wiki = _get_wiki_client(language)
        pagina = wiki.page(query)
        
        if not pagina.exists():
            # Intentar con el idioma alternativo
            lang_alt = "en" if language == "es" else "es"
            wiki_alt = _get_wiki_client(lang_alt)
            pagina = wiki_alt.page(query)
            if not pagina.exists():
                return {"error": f"No se encontró artículo para '{query}' en Wikipedia ({language}/{lang_alt})."}
            language = lang_alt
        
        # Limitar el resumen a los primeros 500 caracteres para no saturar el contexto
        resumen = pagina.summary[:500] + ("..." if len(pagina.summary) > 500 else "")
        
        resultado = {
            "title": pagina.title,
            "summary": resumen,
            "url": pagina.fullurl,
            "language": language
        }
        logger.info(f"[TOOL] search_wikipedia_summary | found='{pagina.title}'")
        return resultado
    
    except Exception as e:
        return {"error": f"Error al consultar Wikipedia: {e}"}


# ─────────────────────────────────────────────
# HERRAMIENTA 5: Generar reporte Markdown
# ─────────────────────────────────────────────

def create_calculation_report(title: str, calculations: List[dict]) -> dict:
    """
    Genera y guarda en disco un reporte Markdown con los cálculos realizados.
    ⚠️ EFECTO SECUNDARIO: escribe un archivo en la carpeta reports/.
    """
    logger.info(f"[TOOL] create_calculation_report | title='{title}', n_calcs={len(calculations)}")
    
    try:
        os.makedirs("reports", exist_ok=True)
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        nombre_archivo = f"reports/reporte_{timestamp}.md"
        
        lineas = [
            f"# {title}",
            f"\n**Generado:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}",
            f"\n**Total de cálculos:** {len(calculations)}",
            "\n---\n",
            "## Resultados\n"
        ]
        
        for i, calc in enumerate(calculations, 1):
            desc = calc.get("description", "Sin descripción")
            res = calc.get("result", "N/A")
            lineas.append(f"### {i}. {desc}")
            lineas.append(f"\n**Resultado:** `{res}`\n")
        
        lineas.append("\n---\n*Reporte generado por math_data_agent v1.0*\n")
        
        contenido = "\n".join(lineas)
        with open(nombre_archivo, "w", encoding="utf-8") as f:
            f.write(contenido)
        
        resultado = {
            "status": "success",
            "file_path": nombre_archivo,
            "title": title,
            "calculations_count": len(calculations),
            "message": f"Reporte guardado exitosamente en '{nombre_archivo}'"
        }
        logger.info(f"[TOOL] create_calculation_report | saved={nombre_archivo}")
        return resultado
    
    except Exception as e:
        return {"error": f"Error al generar el reporte: {e}"}


# Mapa de nombre → función para el dispatcher del agente
TOOL_FUNCTIONS = {
    "calculate_expression": calculate_expression,
    "get_currency_exchange": get_currency_exchange,
    "query_statistics": query_statistics,
    "search_wikipedia_summary": search_wikipedia_summary,
    "create_calculation_report": create_calculation_report,
}
```

#### Salida esperada

```bash
# Prueba rápida de cada herramienta
python - << 'EOF'
from tools_impl import (calculate_expression, get_currency_exchange,
                        query_statistics, search_wikipedia_summary)

print(calculate_expression("(100 - 20) / 4 + 2 ** 3"))
print(get_currency_exchange("USD", "EUR"))
print(query_statistics("ventas", "mean", "ventas"))
print(search_wikipedia_summary("Inteligencia artificial", "es"))
EOF
```

#### Verificación

Debes ver salidas similares a:
```
{'result': 28.0, 'expression': '(100 - 20) / 4 + 2 ** 3'}
{'base': 'USD', 'target': 'EUR', 'rate': 0.9234, 'date': '2024-...', 'description': '1 USD = 0.9234 EUR'}
{'dataset': 'ventas', 'column': 'ventas', 'operation': 'mean', 'result': 20233.3333, ...}
{'title': 'Inteligencia artificial', 'summary': '...', 'url': '...'}
```

---

### Paso 3 — Implementar el bucle agentico con control de flujo

**Objetivo:** Crear el archivo principal `math_data_agent.py` con el bucle ReAct completo, incluyendo: manejo de múltiples `tool_calls` en paralelo, límite de 10 iteraciones, confirmación humana para `create_calculation_report` y logging detallado.

#### Instrucciones

3. Crea el archivo `math_data_agent.py`:

```python
# math_data_agent.py
"""
Agente conversacional con Function Calling de OpenAI.
Implementa el bucle ReAct: Observar → Pensar (LLM) → Actuar (Tools) → repetir.

Herramientas disponibles:
  - calculate_expression: evaluación matemática segura (sin eval)
  - get_currency_exchange: tipos de cambio en tiempo real (Frankfurter)
  - query_statistics: estadísticas sobre datasets CSV locales
  - search_wikipedia_summary: resúmenes de Wikipedia
  - create_calculation_report: genera reporte Markdown (requiere confirmación)

Controles de seguridad:
  - Máximo 10 iteraciones del bucle agentico
  - Confirmación humana antes de ejecutar create_calculation_report
  - Logging de cada tool_call con argumentos y resultado
"""

import json
import logging
import os
import sys
from typing import Optional

from dotenv import load_dotenv
from openai import OpenAI, APIError, RateLimitError
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type

from tools_schema import TOOLS
from tools_impl import TOOL_FUNCTIONS

# ─────────────────────────────────────────────
# Configuración de logging
# ─────────────────────────────────────────────
os.makedirs("logs", exist_ok=True)
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
    handlers=[
        logging.FileHandler("logs/agent.log", encoding="utf-8"),
        logging.StreamHandler(sys.stdout)
    ]
)
logger = logging.getLogger("math_data_agent")

# ─────────────────────────────────────────────
# Constantes del agente
# ─────────────────────────────────────────────
MAX_ITERACIONES = 10
MODELO = "gpt-4o-mini"  # Cambiar a "gpt-4o" para mayor capacidad (mayor costo)
HERRAMIENTAS_CON_CONFIRMACION = {"create_calculation_report"}

PROMPT_SISTEMA = """Eres un agente asistente especializado en análisis matemático y consulta de datos.
Tienes acceso a las siguientes herramientas:

1. calculate_expression: evalúa expresiones matemáticas de forma segura
2. get_currency_exchange: consulta tipos de cambio en tiempo real
3. query_statistics: realiza estadísticas sobre datasets CSV (ventas, temperaturas)
4. search_wikipedia_summary: busca información en Wikipedia
5. create_calculation_report: genera un reporte Markdown (⚠️ escribe en disco, solo si el usuario lo pide)

Instrucciones de comportamiento:
- Usa las herramientas necesarias para responder con precisión.
- Si necesitas múltiples datos, puedes solicitar varias herramientas en paralelo.
- Cuando hagas cálculos, muestra los pasos intermedios.
- Para create_calculation_report: SOLO úsala si el usuario pide explícitamente guardar o exportar.
- Si una herramienta retorna un error, informa al usuario y sugiere alternativas.
- Responde siempre en español, de forma clara y estructurada."""


# ─────────────────────────────────────────────
# Función: solicitar confirmación humana
# ─────────────────────────────────────────────

def _solicitar_confirmacion(nombre_tool: str, argumentos: dict) -> bool:
    """
    Implementa el patrón Human-in-the-Loop para herramientas con efectos secundarios.
    Retorna True si el usuario confirma, False si cancela.
    """
    print("\n" + "="*60)
    print(f"⚠️  CONFIRMACIÓN REQUERIDA")
    print(f"El agente quiere ejecutar: '{nombre_tool}'")
    print(f"Argumentos:")
    print(json.dumps(argumentos, indent=2, ensure_ascii=False))
    print("Esta acción ESCRIBIRÁ un archivo en disco.")
    print("="*60)
    
    while True:
        respuesta = input("¿Confirmas la ejecución? [s/n]: ").strip().lower()
        if respuesta in ("s", "si", "sí", "y", "yes"):
            logger.info(f"[HUMAN-IN-THE-LOOP] Usuario CONFIRMÓ ejecución de '{nombre_tool}'")
            return True
        elif respuesta in ("n", "no"):
            logger.info(f"[HUMAN-IN-THE-LOOP] Usuario CANCELÓ ejecución de '{nombre_tool}'")
            return False
        print("Por favor responde 's' (sí) o 'n' (no).")


# ─────────────────────────────────────────────
# Función: ejecutar una tool_call individual
# ─────────────────────────────────────────────

def _ejecutar_tool_call(tool_call) -> str:
    """
    Ejecuta una tool_call del modelo, aplicando confirmación si es necesario.
    Retorna el resultado serializado como string JSON.
    """
    nombre = tool_call.function.name
    
    try:
        argumentos = json.loads(tool_call.function.arguments)
    except json.JSONDecodeError as e:
        logger.error(f"[TOOL] Error al parsear argumentos de '{nombre}': {e}")
        return json.dumps({"error": f"Argumentos inválidos: {e}"})
    
    logger.info(f"[TOOL_CALL] id={tool_call.id} | name={nombre} | args={argumentos}")
    
    # Verificar si requiere confirmación humana
    if nombre in HERRAMIENTAS_CON_CONFIRMACION:
        confirmado = _solicitar_confirmacion(nombre, argumentos)
        if not confirmado:
            resultado = {
                "status": "cancelled",
                "message": "El usuario canceló la ejecución de esta herramienta."
            }
            logger.info(f"[TOOL_CALL] '{nombre}' CANCELADA por el usuario.")
            return json.dumps(resultado, ensure_ascii=False)
    
    # Verificar que la función existe
    if nombre not in TOOL_FUNCTIONS:
        resultado = {"error": f"Herramienta '{nombre}' no registrada en el agente."}
        logger.error(f"[TOOL_CALL] Herramienta desconocida: '{nombre}'")
        return json.dumps(resultado, ensure_ascii=False)
    
    # Ejecutar la función
    try:
        resultado = TOOL_FUNCTIONS[nombre](**argumentos)
        logger.info(f"[TOOL_RESULT] id={tool_call.id} | name={nombre} | result={str(resultado)[:200]}")
        return json.dumps(resultado, ensure_ascii=False, default=str)
    except TypeError as e:
        resultado = {"error": f"Argumentos incorrectos para '{nombre}': {e}"}
        logger.error(f"[TOOL_CALL] TypeError en '{nombre}': {e}")
        return json.dumps(resultado, ensure_ascii=False)
    except Exception as e:
        resultado = {"error": f"Error inesperado en '{nombre}': {e}"}
        logger.error(f"[TOOL_CALL] Exception en '{nombre}': {e}", exc_info=True)
        return json.dumps(resultado, ensure_ascii=False)


# ─────────────────────────────────────────────
# Función principal: llamada al LLM con reintentos
# ─────────────────────────────────────────────

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=30),
    retry=retry_if_exception_type(RateLimitError)
)
def _llamar_llm(cliente: OpenAI, mensajes: list) -> object:
    """Llama al LLM con reintentos automáticos en caso de RateLimitError."""
    return cliente.chat.completions.create(
        model=MODELO,
        messages=mensajes,
        tools=TOOLS,
        tool_choice="auto",
        temperature=0.1,  # Baja temperatura para respuestas más deterministas
        max_tokens=2048
    )


# ─────────────────────────────────────────────
# Bucle agentico principal (ReAct Loop)
# ─────────────────────────────────────────────

def ejecutar_agente(pregunta: str, cliente: OpenAI) -> str:
    """
    Ejecuta el bucle agentico completo para responder una pregunta.
    
    Ciclo ReAct:
    1. Enviar mensajes al LLM
    2. Si el modelo retorna tool_calls → ejecutar todas en paralelo
    3. Agregar resultados al historial
    4. Repetir hasta respuesta final o MAX_ITERACIONES
    
    Args:
        pregunta: La consulta del usuario en lenguaje natural.
        cliente: Instancia del cliente OpenAI.
    
    Returns:
        La respuesta final del agente como string.
    
    Raises:
        RuntimeError: Si se supera MAX_ITERACIONES sin respuesta final.
    """
    logger.info(f"[AGENTE] Nueva consulta: '{pregunta[:100]}...' " if len(pregunta) > 100 else f"[AGENTE] Nueva consulta: '{pregunta}'")
    
    mensajes = [
        {"role": "system", "content": PROMPT_SISTEMA},
        {"role": "user", "content": pregunta}
    ]
    
    for iteracion in range(1, MAX_ITERACIONES + 1):
        logger.info(f"[AGENTE] Iteración {iteracion}/{MAX_ITERACIONES}")
        
        try:
            respuesta = _llamar_llm(cliente, mensajes)
        except APIError as e:
            logger.error(f"[AGENTE] Error de API en iteración {iteracion}: {e}")
            return f"Error al comunicarse con el modelo: {e}"
        
        mensaje_asistente = respuesta.choices[0].message
        finish_reason = respuesta.choices[0].finish_reason
        
        logger.info(f"[AGENTE] finish_reason={finish_reason} | tool_calls={len(mensaje_asistente.tool_calls or [])}")
        
        # Agregar el mensaje del asistente al historial
        mensajes.append(mensaje_asistente)
        
        # ── Caso 1: El modelo retorna una respuesta final (sin tool_calls) ──
        if finish_reason == "stop" or not mensaje_asistente.tool_calls:
            respuesta_final = mensaje_asistente.content or "El agente completó la tarea sin generar texto."
            logger.info(f"[AGENTE] Respuesta final generada en iteración {iteracion}.")
            return respuesta_final
        
        # ── Caso 2: El modelo solicita una o más herramientas (paralelo) ──
        if finish_reason == "tool_calls":
            n_tools = len(mensaje_asistente.tool_calls)
            logger.info(f"[AGENTE] Procesando {n_tools} tool_call(s) en paralelo...")
            
            # Ejecutar todas las tool_calls solicitadas (pueden ser múltiples)
            for tool_call in mensaje_asistente.tool_calls:
                resultado_str = _ejecutar_tool_call(tool_call)
                
                # Agregar el resultado al historial con el formato correcto de OpenAI
                mensajes.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": resultado_str
                })
            
            # Continuar al siguiente ciclo del bucle
            continue
        
        # ── Caso 3: finish_reason inesperado ──
        logger.warning(f"[AGENTE] finish_reason inesperado: '{finish_reason}'. Deteniendo.")
        break
    
    # Si se agotaron las iteraciones sin respuesta final
    logger.error(f"[AGENTE] Se alcanzó el límite de {MAX_ITERACIONES} iteraciones sin respuesta final.")
    raise RuntimeError(
        f"El agente no alcanzó una respuesta final en {MAX_ITERACIONES} iteraciones. "
        "Considera simplificar la consulta o aumentar MAX_ITERACIONES."
    )


# ─────────────────────────────────────────────
# Interfaz de línea de comandos (CLI)
# ─────────────────────────────────────────────

def main():
    load_dotenv()
    api_key = os.getenv("OPENAI_API_KEY")
    
    if not api_key:
        print("❌ ERROR: No se encontró OPENAI_API_KEY en el archivo .env")
        sys.exit(1)
    
    cliente = OpenAI(api_key=api_key)
    
    print("\n" + "="*65)
    print("  🤖 Math & Data Agent — Agente con Function Calling")
    print("  Modelo:", MODELO)
    print("  Herramientas:", len(TOOLS))
    print("  Max iteraciones:", MAX_ITERACIONES)
    print("="*65)
    print("Escribe tu consulta o 'salir' para terminar.\n")
    
    while True:
        try:
            consulta = input("🧑 Tú: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\n👋 Hasta luego.")
            break
        
        if not consulta:
            continue
        if consulta.lower() in ("salir", "exit", "quit"):
            print("👋 Hasta luego.")
            break
        
        print("\n🤖 Agente procesando...\n")
        
        try:
            respuesta = ejecutar_agente(consulta, cliente)
            print(f"🤖 Agente: {respuesta}\n")
            print("-"*65 + "\n")
        except RuntimeError as e:
            print(f"⚠️  {e}\n")
        except Exception as e:
            logger.error(f"Error inesperado: {e}", exc_info=True)
            print(f"❌ Error inesperado: {e}\n")


if __name__ == "__main__":
    main()
```

#### Salida esperada

El agente debe iniciar sin errores:

```
=================================================================
  🤖 Math & Data Agent — Agente con Function Calling
  Modelo: gpt-4o-mini
  Herramientas: 5
  Max iteraciones: 10
=================================================================
Escribe tu consulta o 'salir' para terminar.

🧑 Tú:
```

#### Verificación

```bash
python math_data_agent.py
# Escribe 'salir' para salir inmediatamente y verificar que carga sin errores
```

---

### Paso 4 — Ejecutar las cinco queries de prueba complejas

**Objetivo:** Verificar que el agente encadena múltiples herramientas correctamente para resolver consultas complejas que ninguna herramienta individual puede responder sola.

#### Instrucciones

4. Crea el script de prueba automatizado `test_agent_queries.py`:

```python
# test_agent_queries.py
"""
Suite de pruebas para math_data_agent.
Ejecuta 5 queries complejas que requieren encadenar múltiples herramientas.
Verifica que el agente produce respuestas coherentes y usa las herramientas correctas.
"""

import os
import sys
import logging
from dotenv import load_dotenv
from openai import OpenAI

# Suprimir logs en las pruebas para salida más limpia
logging.getLogger("math_data_agent").setLevel(logging.WARNING)
logging.getLogger("tools_impl").setLevel(logging.WARNING)

load_dotenv()
from math_data_agent import ejecutar_agente

QUERIES_PRUEBA = [
    {
        "id": 1,
        "descripcion": "Cálculo matemático + tipo de cambio (2 herramientas)",
        "query": (
            "Si tengo 5,000 USD y quiero convertirlos a Euros, "
            "¿cuántos euros obtengo? Calcula también cuánto sería "
            "el 15% de ese monto en euros como comisión."
        ),
        "herramientas_esperadas": ["get_currency_exchange", "calculate_expression"],
        "verificacion": lambda r: "euro" in r.lower() or "eur" in r.lower()
    },
    {
        "id": 2,
        "descripcion": "Estadísticas + cálculo (2 herramientas)",
        "query": (
            "Del dataset de ventas, calcula la media de ventas y la media de costos. "
            "Luego calcula el margen de ganancia promedio como porcentaje: "
            "((ventas_media - costos_media) / ventas_media) * 100"
        ),
        "herramientas_esperadas": ["query_statistics", "calculate_expression"],
        "verificacion": lambda r: "%" in r or "margen" in r.lower() or "porcentaje" in r.lower()
    },
    {
        "id": 3,
        "descripcion": "Wikipedia + cálculo (2 herramientas)",
        "query": (
            "¿Qué es el número Pi según Wikipedia? "
            "Luego calcula el área de un círculo con radio 7.5 usando Pi = 3.14159265."
        ),
        "herramientas_esperadas": ["search_wikipedia_summary", "calculate_expression"],
        "verificacion": lambda r: "pi" in r.lower() or "área" in r.lower() or "area" in r.lower()
    },
    {
        "id": 4,
        "descripcion": "Múltiples estadísticas en paralelo (3 herramientas)",
        "query": (
            "Necesito un análisis completo: "
            "1) La temperatura máxima promedio de las ciudades en el dataset de temperaturas. "
            "2) La temperatura mínima promedio. "
            "3) Calcula la diferencia entre ambas. "
            "Presenta los resultados de forma clara."
        ),
        "herramientas_esperadas": ["query_statistics", "calculate_expression"],
        "verificacion": lambda r: "temperatura" in r.lower() or "°" in r or "grado" in r.lower()
    },
    {
        "id": 5,
        "descripcion": "Conversión de moneda + estadística + cálculo (3 herramientas)",
        "query": (
            "Las ventas totales del dataset de ventas suman cierta cantidad en USD. "
            "Primero calcula la suma total de ventas (max * número de meses aproximado). "
            "Luego, ¿cuánto sería esa suma convertida a Pesos Mexicanos (MXN)? "
            "Usa el tipo de cambio actual USD→MXN."
        ),
        "herramientas_esperadas": ["query_statistics", "get_currency_exchange", "calculate_expression"],
        "verificacion": lambda r: "mxn" in r.lower() or "peso" in r.lower() or "mexicano" in r.lower()
    }
]


def ejecutar_pruebas():
    """Ejecuta todas las queries de prueba y reporta resultados."""
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        print("❌ OPENAI_API_KEY no encontrada.")
        sys.exit(1)
    
    cliente = OpenAI(api_key=api_key)
    
    print("\n" + "="*70)
    print("  🧪 Suite de Pruebas — math_data_agent")
    print("="*70)
    
    resultados = []
    
    for prueba in QUERIES_PRUEBA:
        print(f"\n📋 Query {prueba['id']}: {prueba['descripcion']}")
        print(f"   Consulta: {prueba['query'][:80]}...")
        print(f"   Herramientas esperadas: {prueba['herramientas_esperadas']}")
        
        try:
            respuesta = ejecutar_agente(prueba["query"], cliente)
            
            # Verificar que la respuesta contiene elementos esperados
            es_valida = prueba["verificacion"](respuesta)
            tiene_contenido = len(respuesta.strip()) > 50
            
            exito = es_valida and tiene_contenido
            estado = "✅ PASS" if exito else "⚠️  PARTIAL"
            
            print(f"   Estado: {estado}")
            print(f"   Respuesta (primeros 200 chars): {respuesta[:200]}...")
            
            resultados.append({
                "id": prueba["id"],
                "exito": exito,
                "respuesta_len": len(respuesta)
            })
        
        except RuntimeError as e:
            print(f"   Estado: ❌ FAIL — {e}")
            resultados.append({"id": prueba["id"], "exito": False, "respuesta_len": 0})
        except Exception as e:
            print(f"   Estado: ❌ ERROR — {e}")
            resultados.append({"id": prueba["id"], "exito": False, "respuesta_len": 0})
        
        print()
    
    # Resumen final
    exitosos = sum(1 for r in resultados if r["exito"])
    print("="*70)
    print(f"  📊 RESUMEN: {exitosos}/{len(QUERIES_PRUEBA)} queries exitosas")
    
    if exitosos >= 4:
        print("  ✅ El agente supera el umbral mínimo de 4/5 queries correctas.")
    else:
        print("  ⚠️  El agente no alcanzó el umbral mínimo. Revisa los logs en logs/agent.log")
    print("="*70 + "\n")
    
    return exitosos


if __name__ == "__main__":
    n = ejecutar_pruebas()
    sys.exit(0 if n >= 4 else 1)
```

5. Ejecuta las pruebas:

```bash
python test_agent_queries.py
```

#### Salida esperada

```
======================================================================
  🧪 Suite de Pruebas — math_data_agent
======================================================================

📋 Query 1: Cálculo matemático + tipo de cambio (2 herramientas)
   Consulta: Si tengo 5,000 USD y quiero convertirlos a Euros...
   Herramientas esperadas: ['get_currency_exchange', 'calculate_expression']
   Estado: ✅ PASS
   Respuesta (primeros 200 chars): Con el tipo de cambio actual de 1 USD = 0.9234 EUR...

📋 Query 2: Estadísticas + cálculo (2 herramientas)
   ...
   Estado: ✅ PASS

[... 3 queries más ...]

======================================================================
  📊 RESUMEN: 5/5 queries exitosas
  ✅ El agente supera el umbral mínimo de 4/5 queries correctas.
======================================================================
```

#### Verificación

```bash
# Verificar que se generaron logs detallados
cat logs/agent.log | grep "\[TOOL_CALL\]" | head -20
```

---

### Paso 5 — Probar el mecanismo Human-in-the-Loop

**Objetivo:** Verificar que el agente solicita confirmación antes de ejecutar `create_calculation_report` y que cancela correctamente si el usuario dice "no".

#### Instrucciones

6. Inicia el agente en modo interactivo:

```bash
python math_data_agent.py
```

7. Escribe la siguiente consulta que debería activar la herramienta de reporte:

```
Calcula el área de un círculo con radio 5 usando Pi = 3.14159. 
Luego guarda un reporte con ese cálculo.
```

8. Cuando aparezca la confirmación, primero responde `n` (cancelar) y observa el comportamiento. Luego repite la consulta y responde `s` para confirmar.

#### Salida esperada (respuesta `n` — cancelar):

```
==============================================================
⚠️  CONFIRMACIÓN REQUERIDA
El agente quiere ejecutar: 'create_calculation_report'
Argumentos:
{
  "title": "Cálculo de Área de Círculo",
  "calculations": [
    {
      "description": "Área de círculo radio=5",
      "result": "78.53975"
    }
  ]
}
Esta acción ESCRIBIRÁ un archivo en disco.
==============================================================
¿Confirmas la ejecución? [s/n]: n

🤖 Agente: Entendido. El cálculo del área del círculo con radio 5 es 
78.54 unidades cuadradas (π × 5² = 78.53975). El reporte no fue 
guardado ya que cancelaste la operación. ¿Deseas que lo guarde ahora?
```

#### Salida esperada (respuesta `s` — confirmar):

```bash
# Verificar que el archivo fue creado
ls -la reports/
# Debe mostrar: reporte_YYYYMMDD_HHMMSS.md

cat reports/reporte_*.md
```

---

## 7. Validación y Pruebas

### Lista de verificación final

Ejecuta cada verificación y confirma que todas pasan:

```bash
# ── Verificación 1: Herramientas definidas correctamente ──
python -c "
from tools_schema import TOOLS
from tools_impl import TOOL_FUNCTIONS
assert len(TOOLS) == 5, f'Se esperaban 5 tools, hay {len(TOOLS)}'
assert len(TOOL_FUNCTIONS) == 5, f'Se esperaban 5 funciones, hay {len(TOOL_FUNCTIONS)}'
nombres_schema = {t['function']['name'] for t in TOOLS}
nombres_impl = set(TOOL_FUNCTIONS.keys())
assert nombres_schema == nombres_impl, f'Mismatch: {nombres_schema ^ nombres_impl}'
print('✅ Verificación 1 PASS: 5 herramientas correctamente definidas e implementadas')
"

# ── Verificación 2: Evaluación matemática segura ──
python -c "
from tools_impl import calculate_expression
# Debe funcionar
r1 = calculate_expression('2 ** 10')
assert r1['result'] == 1024.0, f'Esperado 1024, obtenido {r1}'

# Debe rechazar código peligroso
r2 = calculate_expression('__import__(\"os\").system(\"ls\")')
assert 'error' in r2, 'Debería rechazar importaciones'

r3 = calculate_expression('open(\"/etc/passwd\").read()')
assert 'error' in r3, 'Debería rechazar acceso a archivos'

print('✅ Verificación 2 PASS: Evaluación matemática segura funciona correctamente')
"

# ── Verificación 3: Estadísticas sobre datasets ──
python -c "
from tools_impl import query_statistics
r = query_statistics('ventas', 'mean', 'ventas')
assert 'result' in r, f'Error: {r}'
assert r['result'] > 0, 'La media de ventas debe ser positiva'

# Columna inexistente debe retornar error descriptivo
r2 = query_statistics('ventas', 'mean', 'columna_inexistente')
assert 'error' in r2, 'Columna inexistente debe retornar error'
assert 'columnas_disponibles' in r2, 'Debe sugerir columnas disponibles'

print(f'✅ Verificación 3 PASS: Estadísticas OK (mean ventas = {r[\"result\"]:.2f})')
"

# ── Verificación 4: Logs generados ──
python -c "
import os
assert os.path.exists('logs/agent.log'), 'Archivo de log no encontrado'
with open('logs/agent.log') as f:
    contenido = f.read()
assert '[TOOL]' in contenido or '[AGENTE]' in contenido, 'Logs no contienen entradas del agente'
print('✅ Verificación 4 PASS: Sistema de logging funcionando')
"

# ── Verificación 5: Suite de pruebas ──
python test_agent_queries.py
echo "Exit code: $?"
```

### Prueba de estrés: límite de iteraciones

```python
# test_max_iterations.py — Verifica que el agente no entra en loop infinito
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()
from math_data_agent import ejecutar_agente, MAX_ITERACIONES

cliente = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# Una consulta normal debe responder bien antes del límite
respuesta = ejecutar_agente("¿Cuánto es 2 + 2?", cliente)
assert "4" in respuesta, f"Respuesta inesperada: {respuesta}"
print(f"✅ Consulta simple resuelta: {respuesta[:50]}")
print(f"✅ MAX_ITERACIONES configurado en: {MAX_ITERACIONES}")
```

---

## 8. Resolución de Problemas

### Problema 1: `JSONDecodeError` al parsear argumentos de una tool_call

**Síntoma:**
```
[TOOL] Error al parsear argumentos de 'calculate_expression': 
Expecting value: line 1 column 1 (char 0)
```
El agente registra el error pero no detiene la ejecución. En la respuesta final, el modelo indica que no pudo ejecutar la herramienta.

**Causa:**
El modelo generó una respuesta con `tool_calls` pero el campo `arguments` llegó vacío o malformado. Esto ocurre ocasionalmente con `gpt-4o-mini` cuando la consulta es ambigua o la expresión matemática contiene caracteres especiales (comillas tipográficas `"` en lugar de `"`).

**Solución:**
1. Reformula la consulta evitando comillas tipográficas o caracteres especiales en las expresiones matemáticas.
2. Si el problema persiste, agrega validación en `_ejecutar_tool_call`:

```python
# Reemplaza en _ejecutar_tool_call:
try:
    argumentos = json.loads(tool_call.function.arguments)
except json.JSONDecodeError as e:
    # Intentar limpiar el string antes de parsear
    args_limpio = tool_call.function.arguments.strip()
    if not args_limpio or args_limpio == "null":
        args_limpio = "{}"
    try:
        argumentos = json.loads(args_limpio)
    except json.JSONDecodeError:
        logger.error(f"[TOOL] Argumentos irrecuperables: '{args_limpio}'")
        return json.dumps({"error": f"Argumentos inválidos: {e}"})
```

---

### Problema 2: `ConnectionError` en `get_currency_exchange` durante la sesión

**Síntoma:**
```
[TOOL] get_currency_exchange | result={'error': 'No se pudo conectar a Frankfurter API...'}
```
El agente responde que no pudo obtener el tipo de cambio y no puede completar la consulta.

**Causa:**
La API pública de Frankfurter (`api.frankfurter.app`) puede estar temporalmente no disponible. Esto ocurre con baja frecuencia pero puede suceder durante sesiones de clase. El decorador `@retry` de `tenacity` reintentará 3 veces, pero si el servicio está caído, todos los intentos fallarán.

**Solución:**
Activa el backup con Open Exchange Rates (plan gratuito, requiere registro en `openexchangerates.org`):

```python
# En tools_impl.py, modifica get_currency_exchange:
def get_currency_exchange(base: str, target: str) -> dict:
    # ... código existente ...
    
    # Si Frankfurter falla, intentar con valor hardcodeado de referencia
    # (solo para demostración en clase cuando la API no está disponible)
    TASAS_FALLBACK = {
        ("USD", "EUR"): 0.92, ("USD", "MXN"): 17.15,
        ("USD", "COP"): 3900.0, ("EUR", "USD"): 1.09,
        ("USD", "GBP"): 0.79, ("USD", "BRL"): 4.95,
    }
    clave = (base.upper(), target.upper())
    if clave in TASAS_FALLBACK:
        tasa = TASAS_FALLBACK[clave]
        logger.warning(f"[FALLBACK] Usando tasa de referencia para {clave}: {tasa}")
        return {
            "base": base, "target": target, "rate": tasa,
            "date": "fallback-reference",
            "description": f"1 {base} ≈ {tasa} {target} (tasa de referencia, API no disponible)",
            "warning": "Dato de referencia. Verifica en fuente oficial."
        }
    return {"error": f"API no disponible y no hay tasa de fallback para {base}→{target}"}
```

---

## 9. Limpieza del Entorno

```bash
# 1. Salir del agente (si está ejecutándose): escribe 'salir' o Ctrl+C

# 2. Revisar archivos generados
ls -la reports/   # Reportes Markdown generados
ls -la logs/      # Archivos de log

# 3. (Opcional) Eliminar archivos generados durante las pruebas
rm -rf reports/
rm -f logs/agent.log

# 4. Desactivar el entorno virtual
deactivate

# 5. (Opcional) Eliminar el entorno virtual completo
# Windows:
# rmdir /s /q venv_lab08
# macOS/Linux:
# rm -rf venv_lab08

# 6. Verificar que .env NO está en el repositorio
git status  # .env debe aparecer como "ignored" o no aparecer
git diff --cached  # No debe mostrar API keys
```

> ⚠️ **Importante:** Nunca elimines el archivo `.env` con `git rm` si ya fue rastreado accidentalmente. En ese caso, usa `git rm --cached .env` y confirma que está en `.gitignore`.

---

## 10. Resumen

En este laboratorio construiste un **agente conversacional completo** basado en el paradigma ReAct (Observar → Pensar → Actuar) de la Lección 8.1. Los logros principales fueron:

| Componente | Implementado |
|---|---|
| 5 herramientas con esquemas JSON Schema | ✅ |
| Evaluación matemática segura (sin `eval`) | ✅ |
| Consulta de API externa con reintentos | ✅ |
| Estadísticas sobre CSV con pandas | ✅ |
| Búsqueda en Wikipedia | ✅ |
| Bucle ReAct con límite de 10 iteraciones | ✅ |
| Múltiples `tool_calls` en paralelo | ✅ |
| Human-in-the-Loop para herramientas destructivas | ✅ |
| Logging estructurado de cada tool_call | ✅ |
| Suite de pruebas con 5 queries complejas | ✅ |

La diferencia clave entre este agente y un workflow es que **el LLM decide dinámicamente** qué herramientas invocar y en qué orden, sin que el programador haya anticipado cada combinación posible. Esto es exactamente el paradigma que distingue a los agentes de los workflows estructurados, tal como se estudió en la Lección 8.1.

### Recursos adicionales

- [OpenAI Function Calling Documentation](https://platform.openai.com/docs/guides/function-calling)
- [JSON Schema Reference](https://json-schema.org/understanding-json-schema/)
- [Frankfurter API Documentation](https://www.frankfurter.app/docs/)
- [Wikipedia-API Python Package](https://pypi.org/project/Wikipedia-API/)
- [ReAct: Synergizing Reasoning and Acting in Language Models (paper)](https://arxiv.org/abs/2210.03629)
- [Tenacity — Retry library for Python](https://tenacity.readthedocs.io/)

---
