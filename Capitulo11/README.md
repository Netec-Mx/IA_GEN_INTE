<div align="center">

# 🧪 Laboratorio 11

## Observabilidad, evaluación y regresión de agentes con LangSmith

![Nivel](https://img.shields.io/badge/Nivel-Intermedio%20Alto-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)
![Observabilidad](https://img.shields.io/badge/Observabilidad-LangSmith-DB2777?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio vas a construir, instrumentar y evaluar un agente con herramientas usando **LangChain** y **LangSmith**. El objetivo no es ver el “pensamiento interno” del modelo, sino analizar **trazas de ejecución**, llamadas a herramientas, latencia, errores, outputs y métricas de calidad. No uses datos sensibles, credenciales en código, tickets reales ni información privada dentro de las consultas de prueba.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Observabilidad y regresión de agentes</td>
<td width="25%"><strong>⏱️ Duración</strong><br>40 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar, evaluar y crear</td>
<td width="25%"><strong>📦 Entregable</strong><br>Trazas + evaluación + reporte</td>
</tr>
</table>

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a construir un agente con herramientas, observarlo con LangSmith y evaluarlo de forma repetible. Trabajarás con dos versiones del agente: **v1**, una versión base con instrucciones generales, y **v2**, una versión mejorada con política explícita de uso de herramientas.

Después ejecutarás ambas versiones contra preguntas de referencia, revisarás trazas, medirás calidad semántica, latencia, consistencia, uso de herramientas y generarás reportes para decidir si la versión v2 debe aceptarse o rechazarse por regresión.

La práctica conserva el enfoque del laboratorio original: usa variables `LANGSMITH_*`, evita `eval()`, implementa calculadora segura con `ast`, separa métricas de exactitud, consistencia y uso de herramientas, usa cache para reducir costos y deja trazabilidad clara en LangSmith.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Configurar LangSmith para capturar trazas de una aplicación GenAI.
2. Instrumentar funciones críticas con `@traceable`.
3. Crear un agente con herramientas usando LangChain moderno.
4. Implementar herramientas seguras sin usar `eval()`.
5. Analizar latencia, errores y herramientas usadas en el dashboard de LangSmith.
6. Crear un dataset de referencia para evaluación.
7. Ejecutar evaluaciones locales con cache.
8. Ejecutar evaluaciones con `evaluate()` de LangSmith.
9. Medir exactitud semántica, consistencia, latencia y uso correcto de herramientas.
10. Comparar dos versiones del agente para detectar mejoras o regresiones.
11. Generar un reporte profesional con hallazgos y decisión de aceptación.
12. Automatizar pruebas de regresión con Pytest.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Tener conocimientos intermedios de Python.
2. Saber usar Git Bash en Windows.
3. Saber crear y activar entornos virtuales.
4. Saber trabajar con archivos `.env`.
5. Conocer el concepto de agentes y herramientas.
6. Conocer evaluación con dataset de referencia.
7. Tener Visual Studio Code instalado.
8. Tener Python 3.11 o 3.12 instalado.
9. Tener una cuenta de LangSmith.
10. Tener una API key de LangSmith.
11. Tener una API key de OpenAI.
12. Tener acceso a internet.

> [!NOTE]
> Este laboratorio usa API porque su objetivo es observar y evaluar ejecuciones reales. Para controlar costos, usarás muestras pequeñas, cache local y repeticiones limitadas.

---

### 💻 Hardware

| Recurso | Requisito mínimo | Recomendado |
|---|---:|---:|
| Equipo | Laptop o PC con Windows | Laptop o PC con Windows 11 |
| CPU | 2 núcleos | 4 núcleos o más |
| RAM | 8 GB | 16 GB |
| Almacenamiento libre | 1 GB | 2 GB |
| GPU | No requerida | No requerida |
| Internet | 10 Mbps | 25 Mbps o superior |

---

### 🧰 Software

| Software / Paquete | Uso |
|---|---|
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Python 3.11 o 3.12 | Runtime del laboratorio |
| `langchain` | Construcción de agentes |
| `langchain-openai` | Integración OpenAI con LangChain |
| `langsmith` | Observabilidad y evaluación |
| `openai` | Modelo agente y modelo juez |
| `python-dotenv` | Variables de entorno |
| `pydantic` | Validación de datos |
| `pytest` | Pruebas de regresión |
| `tabulate` | Formato de tablas |

---

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 40 minutos |
| Complejidad | Alta |
| Nivel de Bloom | Aplicar, analizar, evaluar y crear |
| Modalidad | Individual o guiada |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Framework | LangChain |
| Observabilidad | LangSmith |
| Modelo agente | Configurable con `.env` |
| Modelo juez | Configurable con `.env` |
| Costo estimado | $0.10–$0.50 USD según repeticiones, muestra y modelo juez |
| Entregable principal | `reports/langsmith_observability_report.md` |
| Entregables secundarios | `comparison_report.json`, cache local, trazas y pruebas |

---

## 🛡️ Consideraciones para estudiantes

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>No subas `.env`.</td>
<td><strong>💸 Costo</strong><br>Usa muestras pequeñas y cache.</td>
<td><strong>📊 Observabilidad</strong><br>Analiza trazas, no pensamiento interno.</td>
</tr>
</table>

1. No escribas API keys dentro del código.
2. No entregues `.env`; entrega solo `.env.example` si se requiere.
3. No uses datos reales de clientes ni información confidencial.
4. LangSmith puede registrar inputs y outputs; usa datos sintéticos.
5. No uses `eval()` para herramientas matemáticas.
6. No presentes estimaciones de modelos como datos oficiales.
7. Ejecuta primero pruebas pequeñas antes de lanzar evaluaciones completas.
8. Evita usar `--refresh-cache` si no necesitas regenerar resultados.
9. La revisión humana sigue siendo necesaria aunque existan métricas automáticas.
10. Si v2 mejora calidad pero aumenta latencia, documenta la decisión.

---

### 🏗️ Arquitectura del laboratorio

```text
Usuario
  ↓
run_traced_agents.py
  ↓
preprocesar_pregunta()  → traza manual @traceable
  ↓
Agente LangChain v1/v2  → trazas automáticas LangSmith
  ↓
Tools:
  - buscar_conocimiento
  - calcular_seguro
  - consultar_modelo
  ↓
postprocesar_respuesta() → traza manual @traceable
  ↓
LangSmith Dashboard
  ↓
Evaluación con evaluate()
  ↓
Reporte final Markdown + JSON
```

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear la carpeta del laboratorio, abrirla en Visual Studio Code, configurar el entorno virtual, instalar dependencias y preparar archivos base seguros.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear una carpeta nueva para este laboratorio. Esta carpeta será la raíz del proyecto y ahí guardarás todos los scripts Python, archivos de configuración, cache, reportes y pruebas. Ejecuta estos comandos desde Git Bash; no necesitas crear archivos manualmente en este paso.

**⚙️ Contenido del paso:**

```bash
mkdir -p ~/labs-ia-gen/lab-11-langsmith-observabilidad
cd ~/labs-ia-gen/lab-11-langsmith-observabilidad
```

**✅ Validación del paso:**

```bash
pwd
```

**📌 Resultado esperado:**

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-11-langsmith-observabilidad
```

---

### ✅ Paso 2. Abre el proyecto en Visual Studio Code

**📝 Descripción del paso:**  
Vas a abrir en Visual Studio Code la carpeta `lab-11-langsmith-observabilidad`. A partir de este momento, todos los archivos nuevos deben crearse dentro de esta carpeta para que las rutas relativas funcionen correctamente.

**⚙️ Contenido del paso:**

```bash
code .
```

Si `code .` no funciona, abre VS Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-11-langsmith-observabilidad
```

**✅ Validación del paso:**  
Confirma que VS Code muestre la carpeta `lab-11-langsmith-observabilidad`.

**📌 Resultado esperado:**  
El proyecto está abierto en Visual Studio Code.

---

### ✅ Paso 3. Crea y activa el entorno virtual

**📝 Descripción del paso:**  
Vas a crear un entorno virtual llamado `.venv` dentro de la carpeta del laboratorio y después lo activarás. Este entorno evita mezclar dependencias de LangChain, LangSmith y OpenAI con otros proyectos Python.

**⚙️ Contenido del paso:**

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
La ruta de Python debe apuntar a `.venv/Scripts/python`.

---

### ✅ Paso 4. Crea `requirements.txt`

**📝 Descripción del paso:**  
Vas a crear el archivo `requirements.txt` en la raíz del proyecto. Este archivo declara las librerías necesarias para crear agentes, instrumentar trazas, ejecutar evaluaciones, usar OpenAI, cargar variables de entorno y correr pruebas de regresión.

**⚙️ Contenido del paso:**

```bash
cat > requirements.txt <<'EOF'
langchain>=1.0,<2
langchain-openai>=1.0,<2
langsmith>=0.4,<1
openai>=1.90,<2
python-dotenv>=1.0,<2
pydantic>=2.10,<3
pytest>=8,<9
tabulate>=0.9,<1
EOF
```

**✅ Validación del paso:**

```bash
cat requirements.txt
```

**📌 Resultado esperado:**  
El archivo contiene las dependencias principales del laboratorio.

---

### ✅ Paso 5. Instala dependencias

**📝 Descripción del paso:**  
Vas a instalar las dependencias dentro del entorno virtual activo. Antes de ejecutar, confirma que Git Bash muestre `(.venv)` al inicio de la línea.

**⚙️ Contenido del paso:**

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**✅ Validación del paso:**

```bash
python -c "import langchain, langsmith, openai; print('✅ Dependencias principales OK')"
```

**📌 Resultado esperado:**

```text
✅ Dependencias principales OK
```

---

### ✅ Paso 6. Crea `.gitignore`

**📝 Descripción del paso:**  
Vas a crear `.gitignore` en la raíz del proyecto. Este archivo evita subir credenciales, entorno virtual, cache, reportes, logs y archivos temporales a un repositorio.

**⚙️ Contenido del paso:**

```bash
cat > .gitignore <<'EOF'
.env
.venv/
__pycache__/
*.pyc
.pytest_cache/
cache/
reports/
*.log
reporte_comparacion.json
EOF
```

**✅ Validación del paso:**

```bash
grep -q '^.env$' .gitignore && echo '✅ .env protegido en .gitignore'
```

**📌 Resultado esperado:**

```text
✅ .env protegido en .gitignore
```

---

### ✅ Paso 7. Crea `.env.example` y `.env`

**📝 Descripción del paso:**  
Vas a crear dos archivos en la raíz del proyecto. `.env.example` funciona como plantilla segura para documentar variables; `.env` contiene tus claves reales y no debe compartirse. Después de crearlo, abre `.env` en VS Code y sustituye los valores de ejemplo por tus claves.

**⚙️ Contenido del paso:**

- Debes ingresar a [LangChainDocs](https://docs.langchain.com/langsmith/observability)
- Crear una cuenta **Free**
- Usar el proveedor de nube **Recomendado AWS**
- Ir a **Tracing**
- Crear un proyecto con el valor de la variable **LANGSMITH_PROJECT**
- Crear una **API KEY**

```bash
cat > .env.example <<'EOF'
OPENAI_API_KEY=sk-tu_clave_openai
LANGSMITH_API_KEY=ls__tu_clave_langsmith
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=tu_langsmith_endpoint
LANGSMITH_PROJECT=lab-11-langsmith-observabilidad
OPENAI_MODEL_AGENT=gpt-4o-mini
OPENAI_MODEL_JUDGE=gpt-4o-mini
SAMPLE_SIZE=5
REPETITIONS=2
EOF

cp .env.example .env
```

**🔧 Qué debes cambiar:**  
Abre `.env` y sustituye:

```text
OPENAI_API_KEY=sk-tu_clave_openai
LANGSMITH_API_KEY=ls__tu_clave_langsmith
LANGSMITH_ENDPOINT=tu_langsmith_endpoint
```

por tus claves reales.

**✅ Validación del paso:**

```bash
cat .env.example
```

**📌 Resultado esperado:**  
Existe `.env.example`, existe `.env` y `.env` está protegido por `.gitignore`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 2. Validar entorno, credenciales y configuración de LangSmith

## 🎯 Objetivo de la tarea

Crear un script de validación para confirmar que las credenciales, el proyecto de LangSmith y el modo de tracing están configurados antes de ejecutar agentes.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `verify_langsmith.py`

**📝 Descripción del paso:**  
Vas a crear el archivo `verify_langsmith.py` en la raíz del proyecto. Este script cargará `.env`, revisará `LANGSMITH_API_KEY`, `LANGSMITH_PROJECT` y `LANGSMITH_TRACING`, y probará conexión con LangSmith usando el cliente oficial.

**⚙️ Contenido del paso:**

```bash
cat > verify_langsmith.py <<'PY'
"""
Verifica configuración básica de LangSmith para el laboratorio 11.
"""
from __future__ import annotations

import os
from dotenv import load_dotenv
from langsmith import Client

load_dotenv()


def main() -> None:
    api_key = os.getenv("LANGSMITH_API_KEY")
    project = os.getenv("LANGSMITH_PROJECT")
    tracing = os.getenv("LANGSMITH_TRACING", "false")

    if not api_key:
        raise RuntimeError("Falta LANGSMITH_API_KEY en .env")
    if not project:
        raise RuntimeError("Falta LANGSMITH_PROJECT en .env")

    client = Client(api_key=api_key)
    projects = list(client.list_projects())
    names = [p.name for p in projects]

    print("✅ Conexión con LangSmith exitosa")
    print(f"Tracing configurado: {tracing}")
    print(f"Proyecto objetivo: {project}")

    if project in names:
        print(f"✅ Proyecto encontrado: {project}")
    else:
        print(f"ℹ️ Proyecto '{project}' aún no existe o no aparece en la lista.")
        print("   LangSmith puede crearlo automáticamente al recibir las primeras trazas.")


if __name__ == "__main__":
    main()
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile verify_langsmith.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Ejecuta la verificación

**📝 Descripción del paso:**  
Vas a ejecutar `verify_langsmith.py` desde Git Bash. Esta validación debe completarse antes de correr agentes, porque si LangSmith no está configurado no podrás observar trazas ni subir evaluaciones.

**⚙️ Contenido del paso:**

```bash
python verify_langsmith.py
```

**✅ Validación del paso:**  
En LangSmith, confirma que puedes entrar a la consola y ver tu área de trabajo. Las trazas aparecerán después de ejecutar el agente.

**📌 Resultado esperado:**

```text
✅ Conexión con LangSmith exitosa
Tracing configurado: true
Proyecto objetivo: lab-11-langsmith-observabilidad
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 3. Crear herramientas seguras del agente

## 🎯 Objetivo de la tarea

Crear herramientas reutilizables y seguras para que el agente pueda buscar conocimiento, calcular expresiones y consultar datos operativos de modelos sin inventar información.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `agent_tools.py`

**📝 Descripción del paso:**  
Vas a crear el archivo `agent_tools.py` en la raíz del proyecto. Este archivo contiene las herramientas que el agente podrá invocar. Incluye una calculadora segura basada en `ast`, una búsqueda simulada de conocimiento y una consulta cautelosa sobre modelos. No uses `eval()` en esta práctica.

**⚙️ Contenido del paso:**

```bash
cat > agent_tools.py <<'PY'
"""
Herramientas seguras para el agente del laboratorio 11.
"""
from __future__ import annotations

import ast
import math
import operator
from typing import Any

from langchain.tools import tool

MAX_EXPR_LEN = 120
MAX_ABS_NUMBER = 1_000_000_000
MAX_POWER_EXPONENT = 12

_ALLOWED_BINOPS = {
    ast.Add: operator.add,
    ast.Sub: operator.sub,
    ast.Mult: operator.mul,
    ast.Div: operator.truediv,
    ast.Mod: operator.mod,
    ast.Pow: operator.pow,
}
_ALLOWED_UNARYOPS = {ast.UAdd: operator.pos, ast.USub: operator.neg}
_ALLOWED_FUNCS = {"sqrt": math.sqrt, "log": math.log, "log2": math.log2, "log10": math.log10, "abs": abs, "round": round}
_ALLOWED_NAMES = {"pi": math.pi, "e": math.e}


def _check_number(value: float | int) -> float | int:
    if abs(value) > MAX_ABS_NUMBER:
        raise ValueError(f"Número demasiado grande: {value}")
    return value


def _eval_node(node: ast.AST) -> float | int:
    if isinstance(node, ast.Constant):
        if isinstance(node.value, (int, float)):
            return _check_number(node.value)
        raise ValueError("Solo se permiten constantes numéricas")
    if isinstance(node, ast.Name):
        if node.id in _ALLOWED_NAMES:
            return _ALLOWED_NAMES[node.id]
        raise ValueError(f"Nombre no permitido: {node.id}")
    if isinstance(node, ast.UnaryOp):
        op_type = type(node.op)
        if op_type not in _ALLOWED_UNARYOPS:
            raise ValueError(f"Operador unario no permitido: {op_type.__name__}")
        return _check_number(_ALLOWED_UNARYOPS[op_type](_eval_node(node.operand)))
    if isinstance(node, ast.BinOp):
        op_type = type(node.op)
        if op_type not in _ALLOWED_BINOPS:
            raise ValueError(f"Operador no permitido: {op_type.__name__}")
        left = _eval_node(node.left)
        right = _eval_node(node.right)
        if op_type is ast.Pow and abs(right) > MAX_POWER_EXPONENT:
            raise ValueError(f"Exponente demasiado grande: {right}")
        return _check_number(_ALLOWED_BINOPS[op_type](left, right))
    if isinstance(node, ast.Call):
        if not isinstance(node.func, ast.Name):
            raise ValueError("Solo se permiten funciones simples por nombre")
        func_name = node.func.id
        if func_name not in _ALLOWED_FUNCS:
            raise ValueError(f"Función no permitida: {func_name}")
        args = [_eval_node(arg) for arg in node.args]
        return _check_number(_ALLOWED_FUNCS[func_name](*args))
    raise ValueError(f"Expresión no soportada: {type(node).__name__}")


@tool
def calcular_seguro(expresion: str) -> str:
    """Evalúa una expresión matemática segura. Úsala para cualquier cálculo numérico."""
    if not expresion or len(expresion) > MAX_EXPR_LEN:
        return f"Error: expresión vacía o demasiado larga. Máximo {MAX_EXPR_LEN} caracteres."
    try:
        tree = ast.parse(expresion.strip(), mode="eval")
        result = _eval_node(tree.body)
        return f"Resultado de {expresion} = {result}"
    except ZeroDivisionError:
        return "Error: división por cero."
    except Exception as exc:
        return f"Error al calcular la expresión: {exc}"


@tool
def buscar_conocimiento(query: str) -> str:
    """Busca información conceptual simulada sobre IA, RAG, embeddings, transformers y evaluación."""
    kb = {
        "machine learning": "Machine learning es un campo de IA donde los sistemas aprenden patrones a partir de datos.",
        "deep learning": "Deep learning es un subconjunto de machine learning basado en redes neuronales profundas.",
        "transformer": "Transformer es una arquitectura basada en atención, introducida en 2017, base de muchos modelos modernos.",
        "rag": "RAG combina recuperación de documentos con generación para responder con contexto externo y reducir alucinaciones.",
        "embeddings": "Los embeddings representan texto como vectores densos que capturan similitud semántica.",
        "langsmith": "LangSmith permite observar, depurar, evaluar y monitorear aplicaciones LLM mediante trazas y experimentos.",
        "evaluación": "La evaluación de sistemas GenAI combina datasets de referencia, evaluadores automáticos y revisión humana.",
    }
    q = query.lower()
    for key, value in kb.items():
        if key in q:
            return f"[buscar_conocimiento] {value}"
    return "[buscar_conocimiento] No encontré un resultado específico. Intenta con: RAG, embeddings, transformer, LangSmith o evaluación."


@tool
def consultar_modelo(tema: str) -> str:
    """Consulta datos operativos cautelosos sobre modelos. Úsala para costos, fechas o contexto cuando estén en la base interna."""
    base = {
        "gpt-4o-mini": {"contexto": "128k tokens, sujeto a disponibilidad del proveedor", "costo": "verifica el precio vigente antes de usarlo en producción", "nota": "los precios y capacidades pueden cambiar"},
        "bert": {"lanzamiento": "2018", "arquitectura": "Transformer encoder bidireccional", "parametros": "BERT base: 110M; BERT large: 340M"},
        "gpt-4": {"parametros": "no divulgados oficialmente", "nota": "evita presentar estimaciones como hechos confirmados"},
    }
    normalized = tema.lower().replace(" ", "").replace("-", "")
    for key, data in base.items():
        key_norm = key.lower().replace(" ", "").replace("-", "")
        if key_norm in normalized or normalized in key_norm:
            lines = [f"[consultar_modelo] {key}"]
            lines += [f"- {k}: {v}" for k, v in data.items()]
            return "\n".join(lines)
    return "[consultar_modelo] No hay datos confirmados para ese tema en la base interna."


TOOLS = [buscar_conocimiento, calcular_seguro, consultar_modelo]
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile agent_tools.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Valida herramientas de forma aislada

**📝 Descripción del paso:**  
Vas a probar las herramientas sin agente. Esto permite confirmar que funcionan antes de conectarlas al modelo y facilita detectar errores de implementación.

**⚙️ Contenido del paso:**

```bash
python - <<'PY'
from agent_tools import calcular_seguro, buscar_conocimiento, consultar_modelo
print(calcular_seguro.invoke({"expresion": "175 * 4"}))
print(calcular_seguro.invoke({"expresion": "999999999 ** 999999999"}))
print(buscar_conocimiento.invoke({"query": "qué es RAG"}))
print(consultar_modelo.invoke({"tema": "BERT"}))
PY
```

**✅ Validación del paso:**  
Debe mostrarse una respuesta correcta para `175 * 4`, un bloqueo para el exponente enorme y respuestas de conocimiento.

**📌 Resultado esperado:**  
Las herramientas responden de forma controlada y segura.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 4. Crear agentes v1 y v2

## 🎯 Objetivo de la tarea

Crear dos versiones comparables del agente para evaluar si una mejora de prompt y política de herramientas reduce errores o genera regresiones.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `agent_factory.py`

**📝 Descripción del paso:**  
Vas a crear el archivo `agent_factory.py` en la raíz del proyecto. Este archivo construye `create_agent_v1()` y `create_agent_v2()`, define los prompts del sistema y agrega funciones para extraer la respuesta final y contar herramientas usadas.

**⚙️ Contenido del paso:**

```bash
cat > agent_factory.py <<'PY'
"""
Factory de agentes para el laboratorio 11.
"""
from __future__ import annotations

import os
from typing import Any

from dotenv import load_dotenv
from langchain.agents import create_agent

from agent_tools import TOOLS

load_dotenv()

MODEL_AGENT = os.getenv("OPENAI_MODEL_AGENT", "gpt-4o-mini")

SYSTEM_PROMPT_V1 = """Eres un asistente técnico de IA. Responde en español de forma clara.
Usa herramientas cuando necesites información o cálculos.
No inventes datos que no estén disponibles."""

SYSTEM_PROMPT_V2 = """Eres un asistente técnico de IA especializado en respuestas verificables.

Política de herramientas:
1. Usa buscar_conocimiento para conceptos como RAG, embeddings, deep learning, transformers y evaluación.
2. Usa consultar_modelo para datos de modelos, contexto, costos, fechas o benchmarks.
3. Usa calcular_seguro para cualquier operación numérica.
4. Si un dato no está confirmado oficialmente, dilo explícitamente.
5. En la respuesta final, menciona brevemente qué fuente/herramienta usaste cuando sea relevante.
6. Sé conciso, preciso y evita estimaciones no verificadas.
"""


def create_agent_v1() -> Any:
    return create_agent(model=f"openai:{MODEL_AGENT}", tools=TOOLS, system_prompt=SYSTEM_PROMPT_V1)


def create_agent_v2() -> Any:
    return create_agent(model=f"openai:{MODEL_AGENT}", tools=TOOLS, system_prompt=SYSTEM_PROMPT_V2)


def extract_text(result: Any) -> str:
    if isinstance(result, dict) and "messages" in result:
        messages = result["messages"]
        for msg in reversed(messages):
            content = getattr(msg, "content", None)
            if content:
                return content if isinstance(content, str) else str(content)
    return str(result)


def count_tool_messages(result: Any) -> int:
    if not isinstance(result, dict) or "messages" not in result:
        return 0
    return sum(1 for msg in result["messages"] if getattr(msg, "type", "") == "tool")
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile agent_factory.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Crea y ejecuta una prueba rápida del agente

**📝 Descripción del paso:**  
Vas a crear `smoke_agent.py` para validar que el agente puede inicializarse, responder una pregunta y usar herramientas.

**⚙️ Contenido del paso:**

```bash
cat > smoke_agent.py <<'PY'
from agent_factory import create_agent_v1, extract_text, count_tool_messages

agent = create_agent_v1()
result = agent.invoke({"messages": [{"role": "user", "content": "¿Qué es RAG?"}]})
print("Respuesta:", extract_text(result))
print("Tools usadas:", count_tool_messages(result))
PY
```
```bash
python smoke_agent.py
```

**✅ Validación del paso:**  
El agente debe responder en español y debe usar al menos una herramienta para la pregunta conceptual.

**📌 Resultado esperado:**  
`smoke_agent.py` confirma que el agente está funcional.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 5. Agregar trazas manuales con LangSmith

## 🎯 Objetivo de la tarea

Instrumentar un pipeline de ejecución para capturar preprocesamiento, agente, postprocesamiento, latencia, herramientas usadas y resultados en LangSmith.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `run_traced_agents.py`

**📝 Descripción del paso:**  
Vas a crear `run_traced_agents.py` en la raíz del proyecto. Este script ejecuta preguntas contra los agentes v1 y v2, usa `@traceable` para etapas manuales y guarda una copia local de resultados en `cache/traced_runs.json`.

**⚙️ Contenido del paso:**

```bash
cat > run_traced_agents.py <<'PY'
"""
Ejecuta agentes v1/v2 con tracing automático y manual en LangSmith.
"""
from __future__ import annotations

import json
import time
from pathlib import Path
from typing import Any

from dotenv import load_dotenv
from langsmith import traceable
from agent_factory import create_agent_v1, create_agent_v2, extract_text, count_tool_messages

load_dotenv()

QUESTIONS = [
    "¿Cuál es la diferencia entre machine learning y deep learning?",
    "¿Qué es RAG y por qué ayuda a reducir alucinaciones?",
    "Si tengo 175 billones de parámetros en float32, ¿cuántos GB son?",
    "¿Qué son los embeddings y para qué se usan?",
    "¿Qué datos confirmados tienes sobre GPT-4?",
]

@traceable(name="preprocesar_pregunta", tags=["preprocessing"])
def preprocesar_pregunta(question: str) -> str:
    return question.strip()

@traceable(name="postprocesar_respuesta", tags=["postprocessing"])
def postprocesar_respuesta(answer: str, tool_count: int, latency: float) -> dict[str, Any]:
    return {"answer": answer, "answer_words": len(answer.split()), "tool_count": tool_count, "latency_seconds": round(latency, 4)}

@traceable(name="pipeline_agente", tags=["agent", "observability"])
def run_pipeline(question: str, version: str = "v1") -> dict[str, Any]:
    agent = create_agent_v1() if version == "v1" else create_agent_v2()
    clean_question = preprocesar_pregunta(question)
    start = time.perf_counter()
    result = agent.invoke({"messages": [{"role": "user", "content": clean_question}]})
    latency = time.perf_counter() - start
    answer = extract_text(result)
    tool_count = count_tool_messages(result)
    final = postprocesar_respuesta(answer, tool_count, latency)
    final.update({"question": question, "version": version})
    return final


def main() -> None:
    Path("cache").mkdir(exist_ok=True)
    all_results = []
    for version in ["v1", "v2"]:
        print(f"\n=== Ejecutando agente {version.upper()} con tracing ===")
        for idx, question in enumerate(QUESTIONS, 1):
            print(f"[{idx}/{len(QUESTIONS)}] {question[:70]}...")
            try:
                output = run_pipeline(question, version=version)
                print(f"  ✅ {output['latency_seconds']}s | tools: {output['tool_count']}")
                all_results.append(output)
            except Exception as exc:
                print(f"  ❌ Error: {exc}")
                all_results.append({"question": question, "version": version, "error": str(exc)})
    with open("cache/traced_runs.json", "w", encoding="utf-8") as f:
        json.dump(all_results, f, ensure_ascii=False, indent=2)
    print("\n✅ Ejecución completada. Revisa LangSmith y cache/traced_runs.json")

if __name__ == "__main__":
    main()
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile run_traced_agents.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Ejecuta las trazas

**📝 Descripción del paso:**  
Vas a ejecutar `run_traced_agents.py`. Este paso enviará consultas al modelo, por lo que puede consumir API. Al finalizar, revisa LangSmith para confirmar que aparezcan runs raíz y sub-runs.

**⚙️ Contenido del paso:**

```bash
python run_traced_agents.py
```

**✅ Validación del paso:**  
En LangSmith debes observar `pipeline_agente`, `preprocesar_pregunta`, `postprocesar_respuesta`, llamadas del agente, uso de herramientas y latencia por etapa.

**📌 Resultado esperado:**  
Las ejecuciones quedan trazadas en LangSmith y guardadas localmente en `cache/traced_runs.json`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 6. Crear dataset de referencia y evaluadores

## 🎯 Objetivo de la tarea

Definir un dataset pequeño y construir un flujo de evaluación local y en LangSmith para medir exactitud semántica, latencia, consistencia y uso de herramientas.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `eval_dataset.py`

**📝 Descripción del paso:**  
Vas a crear `eval_dataset.py` en la raíz del proyecto. Este archivo contiene las preguntas de referencia, la respuesta esperada y la herramienta que idealmente debería usar el agente.

**⚙️ Contenido del paso:**

```bash
cat > eval_dataset.py <<'PY'
DATASET_REFERENCIA = [
    {"id": "q1", "input": "¿Cuál es la diferencia entre machine learning y deep learning?", "reference": "Machine learning es el campo general donde sistemas aprenden de datos; deep learning es un subconjunto que usa redes neuronales profundas.", "expected_tool": "buscar_conocimiento"},
    {"id": "q2", "input": "¿Qué es RAG en IA generativa?", "reference": "RAG combina recuperación de documentos con generación de texto para responder usando contexto externo y reducir alucinaciones.", "expected_tool": "buscar_conocimiento"},
    {"id": "q3", "input": "Si tengo 175 billones de parámetros en float32, ¿cuántos GB son?", "reference": "175 billones de parámetros por 4 bytes equivalen aproximadamente a 700 GB.", "expected_tool": "calcular_seguro"},
    {"id": "q4", "input": "¿Qué son los embeddings?", "reference": "Los embeddings son vectores densos que representan texto y capturan similitud semántica.", "expected_tool": "buscar_conocimiento"},
    {"id": "q5", "input": "¿Qué datos confirmados tienes sobre GPT-4?", "reference": "Los parámetros de GPT-4 no han sido divulgados oficialmente; no deben presentarse estimaciones como hechos confirmados.", "expected_tool": "consultar_modelo"},
]
PY
```

**✅ Validación del paso:**

```bash
python - <<'PY'
from eval_dataset import DATASET_REFERENCIA
assert len(DATASET_REFERENCIA) == 5
assert all('input' in row and 'reference' in row and 'expected_tool' in row for row in DATASET_REFERENCIA)
print('✅ Dataset de referencia válido')
PY
```

**📌 Resultado esperado:**

```text
✅ Dataset de referencia válido
```

---

### ✅ Paso 2. Crea `evaluate_agents.py`

**📝 Descripción del paso:**  
Vas a crear `evaluate_agents.py`. Este archivo implementa el target evaluable, evaluadores de score semántico, latencia, uso de herramientas, evaluación local con cache y evaluación formal con `evaluate()` de LangSmith.

**⚙️ Contenido del paso:**

```bash
cat > evaluate_agents.py <<'PY'
"""
Evaluación de agentes v1/v2 con cache y evaluadores compatibles con LangSmith.
"""
from __future__ import annotations

import argparse
import json
import os
import time
from pathlib import Path
from typing import Any

from dotenv import load_dotenv
from langsmith import Client, traceable
from langsmith.evaluation import evaluate
from openai import OpenAI

from agent_factory import create_agent_v1, create_agent_v2, extract_text, count_tool_messages
from eval_dataset import DATASET_REFERENCIA

load_dotenv()

CACHE_DIR = Path("cache")
CACHE_DIR.mkdir(exist_ok=True)
REPORTS_DIR = Path("reports")
REPORTS_DIR.mkdir(exist_ok=True)

MODEL_JUDGE = os.getenv("OPENAI_MODEL_JUDGE", "gpt-4o-mini")
client_openai = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))


@traceable(name="target_agent", tags=["target"])
def target_agent(inputs: dict[str, Any], version: str = "v1") -> dict[str, Any]:
    agent = create_agent_v1() if version == "v1" else create_agent_v2()
    question = inputs["input"]
    start = time.perf_counter()
    result = agent.invoke({"messages": [{"role": "user", "content": question}]})
    latency = time.perf_counter() - start
    answer = extract_text(result)
    tools_used_count = count_tool_messages(result)
    return {
        "output": answer,
        "latency_seconds": latency,
        "tool_count": tools_used_count,
        "raw": str(result)[:3000],
    }


def semantic_score(outputs: dict, reference_outputs: dict, inputs: dict) -> dict:
    """Evaluador LLM-as-a-judge para exactitud semántica."""
    prediction = outputs.get("output", "")
    reference = reference_outputs.get("reference", "")
    question = inputs.get("input", "")
    prompt = f"""Evalúa si la respuesta del agente es semánticamente correcta contra la referencia.

Pregunta: {question}
Referencia: {reference}
Respuesta del agente: {prediction}

Devuelve SOLO JSON válido con esta forma:
{{"score": 0.0, "reasoning": "explicación breve"}}
La escala es 0.0 a 1.0."""
    try:
        response = client_openai.chat.completions.create(
            model=MODEL_JUDGE,
            messages=[
                {"role": "system", "content": "Eres un evaluador estricto y breve."},
                {"role": "user", "content": prompt},
            ],
            temperature=0,
            response_format={"type": "json_object"},
        )
        data = json.loads(response.choices[0].message.content)
        score = max(0.0, min(1.0, float(data.get("score", 0))))
        return {"key": "semantic_score", "score": score, "comment": data.get("reasoning", "")}
    except Exception as exc:
        return {"key": "semantic_score", "score": 0.0, "comment": f"Error juez: {exc}"}


def latency_score(outputs: dict, reference_outputs: dict, inputs: dict) -> dict:
    """Evaluador determinista de latencia."""
    latency = float(outputs.get("latency_seconds", 999))
    score = 1.0 if latency <= 8 else 0.5 if latency <= 15 else 0.0
    return {"key": "latency_score", "score": score, "comment": f"latency={latency:.2f}s"}


def tool_usage_score(outputs: dict, reference_outputs: dict, inputs: dict) -> dict:
    """Evaluador simple de uso de herramientas."""
    tool_count = int(outputs.get("tool_count", 0))
    score = 1.0 if tool_count >= 1 else 0.0
    return {"key": "tool_usage_score", "score": score, "comment": f"tool_count={tool_count}"}


def load_or_create_dataset(dataset_name: str) -> str:
    """Crea dataset en LangSmith si no existe."""
    client = Client(api_key=os.getenv("LANGSMITH_API_KEY"))
    existing = list(client.list_datasets(dataset_name=dataset_name))
    if existing:
        return dataset_name
    dataset = client.create_dataset(dataset_name=dataset_name, description="Dataset Lab 11 observabilidad y evaluación")
    for row in DATASET_REFERENCIA:
        client.create_example(
            inputs={"input": row["input"]},
            outputs={"reference": row["reference"], "expected_tool": row["expected_tool"]},
            dataset_id=dataset.id,
        )
    return dataset_name


def run_langsmith_evaluate(version: str, sample_size: int, upload_results: bool = True) -> Any:
    dataset_name = load_or_create_dataset("lab-11-agent-eval-dataset")
    def app(inputs: dict) -> dict:
        return target_agent(inputs, version=version)
    return evaluate(
        app,
        data=dataset_name,
        evaluators=[semantic_score, latency_score, tool_usage_score],
        experiment_prefix=f"lab11-{version}",
        max_concurrency=1,
        upload_results=upload_results,
        num_repetitions=1,
    )


def run_local_cached(version: str, sample_size: int, repetitions: int, refresh_cache: bool = False) -> dict:
    cache_path = CACHE_DIR / f"eval_{version}_s{sample_size}_r{repetitions}.json"
    if cache_path.exists() and not refresh_cache:
        with open(cache_path, encoding="utf-8") as f:
            return json.load(f)
    rows = DATASET_REFERENCIA[:sample_size]
    outputs = []
    for row in rows:
        for rep in range(repetitions):
            print(f"[{version}] {row['id']} rep {rep+1}/{repetitions}")
            result = target_agent({"input": row["input"]}, version=version)
            sem = semantic_score(result, {"reference": row["reference"]}, {"input": row["input"]})
            lat = latency_score(result, {}, {})
            tool = tool_usage_score(result, {}, {})
            outputs.append({
                "id": row["id"],
                "input": row["input"],
                "reference": row["reference"],
                "version": version,
                "repetition": rep + 1,
                "output": result["output"],
                "latency_seconds": result["latency_seconds"],
                "tool_count": result["tool_count"],
                "semantic_score": sem["score"],
                "semantic_comment": sem.get("comment", ""),
                "latency_score": lat["score"],
                "tool_usage_score": tool["score"],
            })
            time.sleep(0.3)
    summary = summarize(outputs)
    payload = {"version": version, "summary": summary, "rows": outputs}
    with open(cache_path, "w", encoding="utf-8") as f:
        json.dump(payload, f, ensure_ascii=False, indent=2)
    return payload


def summarize(rows: list[dict]) -> dict:
    if not rows:
        return {}
    return {
        "semantic_score_avg": sum(r["semantic_score"] for r in rows) / len(rows),
        "latency_avg": sum(r["latency_seconds"] for r in rows) / len(rows),
        "tool_usage_score_avg": sum(r["tool_usage_score"] for r in rows) / len(rows),
        "n": len(rows),
    }


def consistency_rate(rows: list[dict]) -> float:
    """Mide estabilidad por pregunta."""
    by_id: dict[str, list[float]] = {}
    for r in rows:
        by_id.setdefault(r["id"], []).append(float(r["semantic_score"]))
    if not by_id:
        return 0.0
    consistent = 0
    for scores in by_id.values():
        if max(scores) - min(scores) <= 0.25:
            consistent += 1
    return consistent / len(by_id)


def main() -> None:
    parser = argparse.ArgumentParser()
    parser.add_argument("--sample-size", type=int, default=int(os.getenv("SAMPLE_SIZE", "5")))
    parser.add_argument("--repetitions", type=int, default=int(os.getenv("REPETITIONS", "2")))
    parser.add_argument("--refresh-cache", action="store_true")
    parser.add_argument("--langsmith-evaluate", action="store_true", help="Ejecuta evaluate() y sube resultados a LangSmith")
    parser.add_argument("--local-only", action="store_true", help="Solo ejecuta cache local")
    args = parser.parse_args()
    if args.langsmith_evaluate and not args.local_only:
        print("Ejecutando evaluate() en LangSmith para v1 y v2...")
        run_langsmith_evaluate("v1", args.sample_size, upload_results=True)
        run_langsmith_evaluate("v2", args.sample_size, upload_results=True)
    v1 = run_local_cached("v1", args.sample_size, args.repetitions, args.refresh_cache)
    v2 = run_local_cached("v2", args.sample_size, args.repetitions, args.refresh_cache)
    v1["summary"]["consistency_rate"] = consistency_rate(v1["rows"])
    v2["summary"]["consistency_rate"] = consistency_rate(v2["rows"])
    comparison = {
        "v1": v1["summary"],
        "v2": v2["summary"],
        "delta": {
            "semantic_score_avg": v2["summary"]["semantic_score_avg"] - v1["summary"]["semantic_score_avg"],
            "latency_avg": v2["summary"]["latency_avg"] - v1["summary"]["latency_avg"],
            "tool_usage_score_avg": v2["summary"]["tool_usage_score_avg"] - v1["summary"]["tool_usage_score_avg"],
            "consistency_rate": v2["summary"]["consistency_rate"] - v1["summary"]["consistency_rate"],
        },
    }
    with open("reports/comparison_report.json", "w", encoding="utf-8") as f:
        json.dump(comparison, f, ensure_ascii=False, indent=2)

    print("\n=== Comparación v1 vs v2 ===")
    print(json.dumps(comparison, indent=2, ensure_ascii=False))
    print("\n✅ Reporte JSON: reports/comparison_report.json")


if __name__ == "__main__":
    main()
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile evaluate_agents.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 3. Ejecuta evaluación económica con cache

**📝 Descripción del paso:**  
Vas a ejecutar una evaluación local con muestra pequeña y cache. Esta ruta genera `reports/comparison_report.json` sin subir resultados formales a LangSmith.

**⚙️ Contenido del paso:**

```bash
python evaluate_agents.py --sample-size 5 --repetitions 2 --local-only
```

**✅ Validación del paso:**

```bash
ls -la reports/comparison_report.json
```

**📌 Resultado esperado:**  
Se genera `reports/comparison_report.json`.

---

### ✅ Paso 4. Ejecuta evaluación formal con LangSmith

**📝 Descripción del paso:**  
Vas a ejecutar `evaluate()` de LangSmith. Este paso crea un experimento en LangSmith y conserva también la comparación local en `reports/comparison_report.json`.

**⚙️ Contenido del paso:**

```bash
python evaluate_agents.py --sample-size 5 --repetitions 1 --langsmith-evaluate
```

**✅ Validación del paso:**  
En LangSmith debes ver experimentos con prefijo `lab11-v1` y `lab11-v2`.

**📌 Resultado esperado:**  
Los resultados aparecen en LangSmith y el reporte local se actualiza.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 7. Generar reporte profesional

## 🎯 Objetivo de la tarea

Convertir las métricas y hallazgos de evaluación en un reporte Markdown legible para revisar calidad, regresión, latencia y decisión de aceptación.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `generate_report.py`

**📝 Descripción del paso:**  
Vas a crear `generate_report.py` en la raíz del proyecto. Este script leerá `reports/comparison_report.json`, calculará si existe regresión y generará `reports/langsmith_observability_report.md`.

**⚙️ Contenido del paso:**

```bash
cat > generate_report.py <<'PY'
from __future__ import annotations

import json
from datetime import datetime
from pathlib import Path

REPORTS_DIR = Path("reports")
REPORTS_DIR.mkdir(exist_ok=True)

def fmt(x: float) -> str:
    return f"{x:.4f}"

def main() -> None:
    comparison_path = REPORTS_DIR / "comparison_report.json"
    if not comparison_path.exists():
        raise RuntimeError("Falta reports/comparison_report.json. Ejecuta evaluate_agents.py primero.")
    with open(comparison_path, encoding="utf-8") as f:
        data = json.load(f)
    v1, v2, delta = data["v1"], data["v2"], data["delta"]
    regression = delta["semantic_score_avg"] < -0.05 or delta["consistency_rate"] < -0.05
    decision = "Rechazar v2" if regression else "Aceptar v2 con monitoreo"
    md = f"""# Reporte de Observabilidad y Evaluación — Laboratorio 11

**Fecha:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}  
**Sistema evaluado:** Agente LangChain con herramientas  
**Plataforma de observabilidad:** LangSmith  
**Decisión:** **{decision}**

---

## 1. Resumen ejecutivo

Este reporte compara dos versiones de un agente instrumentado con LangSmith. Se evaluaron métricas de exactitud semántica, consistencia, latencia y uso de herramientas.

---

## 2. Métricas comparativas

| Métrica | v1 | v2 | Delta | Interpretación |
|---|---:|---:|---:|---|
| Score semántico promedio | {fmt(v1['semantic_score_avg'])} | {fmt(v2['semantic_score_avg'])} | {fmt(delta['semantic_score_avg'])} | {'Mejora' if delta['semantic_score_avg'] >= 0 else 'Posible regresión'} |
| Tasa de consistencia | {fmt(v1['consistency_rate'])} | {fmt(v2['consistency_rate'])} | {fmt(delta['consistency_rate'])} | {'Mejora/estable' if delta['consistency_rate'] >= -0.05 else 'Regresión'} |
| Latencia promedio | {fmt(v1['latency_avg'])}s | {fmt(v2['latency_avg'])}s | {fmt(delta['latency_avg'])}s | {'Más rápida/estable' if delta['latency_avg'] <= 0.5 else 'Más lenta'} |
| Uso correcto de herramientas | {fmt(v1['tool_usage_score_avg'])} | {fmt(v2['tool_usage_score_avg'])} | {fmt(delta['tool_usage_score_avg'])} | {'Mejora/estable' if delta['tool_usage_score_avg'] >= -0.05 else 'Regresión'} |

---

## 3. Matriz de observabilidad manual

| Run | Pregunta | Herramienta esperada | Herramienta usada | Latencia | Error | Observación |
|---|---|---|---|---:|---|---|
| 1 | Diferencia ML vs DL | buscar_conocimiento |  |  |  |  |
| 2 | RAG | buscar_conocimiento |  |  |  |  |
| 3 | Parámetros float32 | calcular_seguro |  |  |  |  |
| 4 | Embeddings | buscar_conocimiento |  |  |  |  |
| 5 | Datos GPT-4 | consultar_modelo |  |  |  |  |

---

## 4. Decisión

**Decisión recomendada:** {decision}

- Regresión detectada: **{regression}**
- Delta de score semántico: **{fmt(delta['semantic_score_avg'])}**
- Delta de consistencia: **{fmt(delta['consistency_rate'])}**
- Delta de latencia: **{fmt(delta['latency_avg'])}s**
"""
    output_path = REPORTS_DIR / "langsmith_observability_report.md"
    output_path.write_text(md, encoding="utf-8")
    print(f"✅ Reporte generado: {output_path}")

if __name__ == "__main__":
    main()
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile generate_report.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Ejecuta el generador de reporte

**📝 Descripción del paso:**  
Vas a ejecutar `generate_report.py`. El script leerá el JSON comparativo y generará un reporte Markdown profesional que puedes abrir en VS Code.

**⚙️ Contenido del paso:**

```bash
python generate_report.py
```

**✅ Validación del paso:**

```bash
ls -la reports/langsmith_observability_report.md
```

**📌 Resultado esperado:**  
Se genera `reports/langsmith_observability_report.md`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%207%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 8. Crear pruebas de regresión con Pytest

## 🎯 Objetivo de la tarea

Automatizar umbrales mínimos de calidad para evitar aceptar cambios que degraden score semántico, consistencia, uso de herramientas o latencia.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `test_regression.py`

**📝 Descripción del paso:**  
Vas a crear `test_regression.py` en la raíz del proyecto. Este archivo revisa `reports/comparison_report.json` y falla si la calidad cae por debajo de umbrales mínimos.

**⚙️ Contenido del paso:**

```bash
cat > test_regression.py <<'PY'
import json
from pathlib import Path

REPORT = Path("reports/comparison_report.json")
UMBRAL_SEMANTIC_SCORE = 0.70
UMBRAL_TOOL_USAGE = 0.70
UMBRAL_REGRESSION = -0.05
UMBRAL_LATENCY_SECONDS = 15.0

def load_report():
    if not REPORT.exists():
        raise AssertionError("Falta reports/comparison_report.json. Ejecuta evaluate_agents.py primero.")
    return json.loads(REPORT.read_text(encoding="utf-8"))

def test_v1_semantic_score_minimo():
    data = load_report()
    assert data["v1"]["semantic_score_avg"] >= UMBRAL_SEMANTIC_SCORE

def test_v2_semantic_score_minimo():
    data = load_report()
    assert data["v2"]["semantic_score_avg"] >= UMBRAL_SEMANTIC_SCORE

def test_v2_no_regresa_score():
    data = load_report()
    assert data["delta"]["semantic_score_avg"] >= UMBRAL_REGRESSION

def test_v2_no_regresa_consistencia():
    data = load_report()
    assert data["delta"]["consistency_rate"] >= UMBRAL_REGRESSION

def test_tool_usage_aceptable():
    data = load_report()
    assert data["v2"]["tool_usage_score_avg"] >= UMBRAL_TOOL_USAGE

def test_latencia_aceptable():
    data = load_report()
    assert data["v2"]["latency_avg"] <= UMBRAL_LATENCY_SECONDS
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile test_regression.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Ejecuta pruebas de regresión

**📝 Descripción del paso:**  
Vas a ejecutar Pytest para validar los umbrales. Si alguna prueba falla, revisa el reporte y las trazas antes de aceptar la versión v2.

**⚙️ Contenido del paso:**

- Si la prueba genera **Fallas** es normal ya que los umbrales son muy estrictos.
- Ajusta los umbrales del archivo **test_regression.py** para que la prueba pase.
- Puedes usar los ejemplos de abajo
  - UMBRAL_SEMANTIC_SCORE = 0.50
  - UMBRAL_TOOL_USAGE = 0.70
  - UMBRAL_REGRESSION = -0.25
  - UMBRAL_LATENCY_SECONDS = 15.0

```bash
pytest -v --tb=short
```

**✅ Validación del paso:**  
Pytest debe reportar las pruebas ejecutadas.

**📌 Resultado esperado:**

```text
6 passed
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%208%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 9. Ejecutar validación integral

## 🎯 Objetivo de la tarea

Comprobar que todos los componentes del laboratorio funcionan de extremo a extremo y que existen los artefactos principales.

---

## 🛠️ Pasos

### ✅ Paso 1. Ejecuta la secuencia integral

**📝 Descripción del paso:**  
Vas a ejecutar una secuencia completa desde Git Bash. Esta validación revisa conexión con LangSmith, agente, trazas, evaluación, reporte y pruebas.

**⚙️ Contenido del paso:**

```bash
python verify_langsmith.py
python smoke_agent.py
python run_traced_agents.py
python evaluate_agents.py --sample-size 5 --repetitions 2 --local-only
python generate_report.py
pytest -v --tb=short
```

**✅ Validación del paso:**  
Cada comando debe finalizar sin errores críticos.

**📌 Resultado esperado:**  
El laboratorio completo ejecuta correctamente.

---

### ✅ Paso 2. Revisa archivos generados

**📝 Descripción del paso:**  
Vas a listar `cache/` y `reports/` para confirmar que existen los artefactos de evidencia. Estos archivos son útiles para entregar o revisar resultados sin repetir llamadas a la API.

**⚙️ Contenido del paso:**

```bash
ls -la cache/
ls -la reports/
```

**✅ Validación del paso:**  
Debes tener al menos:

```text
cache/traced_runs.json
cache/eval_v1_s5_r2.json
cache/eval_v2_s5_r2.json
reports/comparison_report.json
reports/langsmith_observability_report.md
```

**📌 Resultado esperado:**  
Las evidencias principales existen y están disponibles.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 9 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%209%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🧩 Tarea 10. Revisar trazas y documentar hallazgos en LangSmith

## 🎯 Objetivo de la tarea

Convertir la observabilidad en conclusiones accionables mediante revisión manual de trazas, herramientas, latencia, errores y regresión.

---

## 🛠️ Pasos

### ✅ Paso 1. Abre el proyecto en LangSmith

**📝 Descripción del paso:**  
Vas a entrar a LangSmith desde el navegador y abrir el proyecto definido en `LANGSMITH_PROJECT`. Ahí revisarás las trazas generadas por `run_traced_agents.py` y por las evaluaciones.

**⚙️ Contenido del paso:**

```text
LangSmith > Projects > lab-11-langsmith-observabilidad
```

**✅ Validación del paso:**  
Debes ver runs recientes relacionados con el laboratorio.

**📌 Resultado esperado:**  
El proyecto muestra trazas de ejecución.

---

### ✅ Paso 2. Revisa una traza `pipeline_agente`

**📝 Descripción del paso:**  
Vas a abrir una traza raíz llamada `pipeline_agente`. Dentro de la traza revisa el input, output, llamadas al modelo, herramientas usadas, sub-runs manuales y latencia por paso.

**⚙️ Contenido del paso:**

Revisa estos elementos en la interfaz de LangSmith:

```text
input del usuario
output final
llamadas al modelo
herramientas usadas
latencia por paso
errores o reintentos
```

**✅ Validación del paso:**  
Confirma que las herramientas aparecen como pasos separados dentro de la traza.

**📌 Resultado esperado:**  
Puedes explicar qué ocurrió durante la ejecución.

---

### ✅ Paso 3. Completa la matriz de observabilidad

**📝 Descripción del paso:**  
Vas a abrir `reports/langsmith_observability_report.md` en VS Code y completar la matriz de observabilidad manual con base en lo que viste en LangSmith. No inventes resultados; usa la evidencia real de las trazas.

**⚙️ Contenido del paso:**

```bash
code reports/langsmith_observability_report.md
```

**✅ Validación del paso:**  
La matriz debe tener observaciones para las preguntas revisadas.

**📌 Resultado esperado:**  
El reporte combina métricas automáticas y revisión humana.

---

### ✅ Paso 4. Usa checklist de observabilidad

**📝 Descripción del paso:**  
Vas a responder este checklist para cerrar el análisis manual. Puedes colocarlo al final del reporte Markdown o usarlo como notas de entrega.

**⚙️ Contenido del paso:**

| Pregunta | Sí/No | Evidencia |
|---|---|---|
| ¿El agente usó la herramienta esperada? |  |  |
| ¿La llamada al modelo domina la latencia? |  |  |
| ¿Hubo errores de herramienta? |  |  |
| ¿Hubo datos no confirmados presentados como hechos? |  |  |
| ¿v2 mejora la calidad de v1? |  |  |
| ¿v2 aumenta demasiado la latencia? |  |  |

**✅ Validación del paso:**  
El checklist debe estar completo con evidencia observada.

**📌 Resultado esperado:**  
Tienes una conclusión accionable basada en datos y trazas.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 10 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2010%20del%20Laboratorio%2011%20sobre%20observabilidad%2C%20evaluaci%C3%B3n%20y%20regresi%C3%B3n%20de%20agentes%20con%20LangSmith.%20Ay%C3%BAdame%20a%20resumir%20el%20objetivo%2C%20archivos%20creados%2C%20comandos%20ejecutados%2C%20validaciones%20y%20resultado%20esperado.)

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Variables de entorno configuradas de forma segura.
4. Conexión validada con LangSmith.
5. Herramientas seguras implementadas sin `eval()`.
6. Agente v1 funcional.
7. Agente v2 funcional con política mejorada.
8. Trazas visibles en LangSmith.
9. Dataset de referencia para evaluación.
10. Evaluación local con cache.
11. Evaluación formal opcional con `evaluate()`.
12. Reporte JSON de comparación.
13. Reporte Markdown profesional.
14. Pruebas Pytest de regresión.
15. Matriz de observabilidad completada manualmente.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación | Evidencia esperada |
|---|---:|---|
| Proyecto local funcional | 10% | `.venv`, dependencias y `.env.example` |
| Conexión con LangSmith | 10% | `verify_langsmith.py` exitoso |
| Herramientas seguras | 15% | `agent_tools.py` sin `eval()` y con validaciones |
| Agentes v1/v2 funcionales | 15% | `smoke_agent.py` y trazas visibles |
| Evaluación ejecutada | 20% | `reports/comparison_report.json` |
| Reporte profesional | 15% | `reports/langsmith_observability_report.md` |
| Pruebas de regresión | 10% | `pytest` exitoso |
| Análisis manual | 5% | Matriz de observabilidad completada |
| Total | 100% |  |

---

# ⚠️ Errores comunes que debes evitar

1. Subir `.env` a un repositorio.
2. Ejecutar evaluaciones grandes sin cache.
3. Usar `eval()` en herramientas matemáticas.
4. Aceptar v2 sin revisar regresiones.
5. Confiar solo en score semántico sin revisar trazas.
6. Presentar datos no confirmados como hechos.
7. Ignorar errores de herramienta en LangSmith.
8. Ejecutar `--refresh-cache` innecesariamente.
9. No revisar latencia por etapa.
10. No documentar la decisión final.

---

# 🧯 Solución de problemas

## Problema 1. No aparecen trazas en LangSmith

**Causa probable:**  
Falta `LANGSMITH_TRACING=true`, la API key no es correcta o el proyecto no está configurado.

**Solución:**

```bash
grep LANGSMITH .env
python verify_langsmith.py
python run_traced_agents.py
```

---

## Problema 2. `create_agent` falla o no reconoce OpenAI

**Causa probable:**  
Versiones incompatibles o falta `langchain-openai`.

**Solución:**

```bash
pip show langchain langchain-openai langsmith
pip install --upgrade "langchain>=1.0,<2" "langchain-openai>=1.0,<2" "langsmith>=0.4,<1"
```

---

## Problema 3. El costo sube por muchas ejecuciones

**Causa probable:**  
Se ejecutan muchas repeticiones, se usa muestra grande o se fuerza cache.

**Solución:**

```bash
python evaluate_agents.py --sample-size 3 --repetitions 1 --local-only
```

Evita `--refresh-cache` salvo que realmente necesites regenerar resultados.

---

## Problema 4. Las pruebas fallan por score bajo

**Causa probable:**  
El agente no usó herramienta, la herramienta no tenía el dato, el prompt permitió inventar o el juez fue severo.

**Solución:**

```bash
cat reports/comparison_report.json
code reports/langsmith_observability_report.md
```

Revisa en LangSmith los casos con menor score.

---

## Problema 5. `reports/comparison_report.json` no existe

**Causa probable:**  
No ejecutaste `evaluate_agents.py` antes de generar el reporte o correr Pytest.

**Solución:**

```bash
python evaluate_agents.py --sample-size 5 --repetitions 2 --local-only
python generate_report.py
pytest -v --tb=short
```

---

# 🧹 Limpieza del entorno

Puedes limpiar resultados generados si necesitas reiniciar evidencias:

```bash
rm -rf cache/ reports/ .pytest_cache/
find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true
find . -name "*.pyc" -delete 2>/dev/null || true
```

Para desactivar el entorno virtual:

```bash
deactivate
```

Antes de compartir el proyecto, valida que `.env` esté protegido:

```bash
grep -q '^.env$' .gitignore && echo '✅ .env protegido'
git status
```

---

# 📚 Resumen conceptual

En este laboratorio instrumentaste un agente con LangSmith, capturaste trazas automáticas y manuales, implementaste herramientas seguras, comparaste dos versiones del agente y aplicaste pruebas de regresión con umbrales de calidad.

| Pregunta | Evidencia |
|---|---|
| ¿Qué pasó durante la ejecución? | Trazas LangSmith |
| ¿Qué herramienta se usó? | Tool calls y mensajes de herramienta |
| ¿Dónde estuvo la mayor latencia? | Duración por run y sub-run |
| ¿La respuesta fue semánticamente correcta? | Score semántico y revisión humana |
| ¿El sistema fue consistente? | Repeticiones y consistencia |
| ¿La nueva versión mejora o degrada? | Deltas v1 vs v2 y pruebas Pytest |

Este laboratorio transforma la evaluación de agentes en un proceso observable, medible, repetible y defendible técnicamente.
