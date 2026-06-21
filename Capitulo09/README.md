<div align="center">

# 🧪 Laboratorio 9

## Servidor MCP seguro para interacción con sistema de archivos

![Nivel](https://img.shields.io/badge/Nivel-Intermedio%20Alto-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)
![Protocolo](https://img.shields.io/badge/Protocolo-MCP-DB2777?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio vas a construir un servidor **Model Context Protocol (MCP)** para exponer herramientas de sistema de archivos dentro de un directorio controlado. El objetivo no es dar acceso libre al sistema operativo, sino crear una capa segura, observable y reutilizable para listar, leer, escribir, buscar y auditar archivos dentro de un sandbox.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Servidor MCP seguro</td>
<td width="25%"><strong>⏱️ Duración</strong><br>50 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar, evaluar y crear</td>
<td width="25%"><strong>📦 Entregable</strong><br>Servidor MCP + cliente + sandbox + logs</td>
</tr>
</table>

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a construir un servidor MCP en Python usando **FastMCP**. El servidor permitirá que un cliente MCP descubra y utilice herramientas para interactuar con archivos dentro de un directorio controlado llamado `sandbox/`.

El servidor expondrá capacidades para:

1. **Listar archivos** dentro del sandbox.
2. **Leer archivos permitidos** por extensión y tamaño.
3. **Escribir archivos únicamente** dentro de `sandbox/output/`.
4. **Buscar texto** usando expresiones regulares controladas.
5. **Consultar metadatos** de archivos permitidos.
6. **Exponer un resource MCP** con el árbol del sandbox.
7. **Exponer prompts MCP reutilizables** para análisis de directorios y búsqueda de patrones.

También construirás un cliente de prueba llamado `test_mcp_client.py`. Este cliente iniciará el servidor MCP por transporte `stdio`, hará handshake, descubrirá tools, resources y prompts, ejecutará pruebas funcionales y validará controles de seguridad.

La práctica está diseñada para que comprendas la diferencia entre integrar herramientas directamente en un agente y exponer capacidades mediante un protocolo estándar. Con MCP, el servidor se convierte en una capa reutilizable que puede ser consumida por distintos clientes o hosts compatibles.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Preparar un proyecto Python local en Windows con VS Code y Git Bash.
2. Instalar y validar el SDK MCP oficial para Python.
3. Crear un sandbox controlado para operaciones de sistema de archivos.
4. Implementar un servidor MCP usando FastMCP.
5. Exponer tools, resources y prompts desde un servidor MCP.
6. Aplicar controles de seguridad de mínimo privilegio.
7. Bloquear intentos de path traversal usando `Path.resolve()` y `relative_to()`.
8. Restringir lectura y escritura por extensión, tamaño y ubicación.
9. Crear un cliente MCP de prueba usando transporte `stdio`.
10. Ejecutar pruebas funcionales y pruebas de seguridad automatizadas.
11. Analizar logs de auditoría para confirmar que el servidor falla de forma segura.
12. Comparar MCP contra function calling directo en términos de reutilización, seguridad y complejidad.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Haber completado o comprendido el laboratorio anterior sobre agentes y function calling.
2. Tener conocimientos básicos de Python.
3. Saber usar `pathlib`, funciones y manejo de excepciones.
4. Comprender la diferencia conceptual entre tools, resources y prompts.
5. Saber ejecutar comandos básicos desde Git Bash.
6. Tener instalado Visual Studio Code.
7. Tener instalado Python 3.11 o superior.
8. Tener conexión a internet para instalar dependencias.
9. Comprender que esta práctica no requiere API key ni llamadas a modelos.
10. Entender que un servidor MCP con transporte `stdio` no debe imprimir logs en `stdout`.

---

### 💻 Hardware

| Recurso | Requisito mínimo | Recomendado |
|---|---:|---:|
| Equipo | Laptop o PC con Windows | Laptop o PC con Windows |
| Sistema operativo | Windows 10 | Windows 11 |
| Procesador | 2 núcleos | 4 núcleos o más |
| Memoria RAM | 8 GB | 16 GB |
| Almacenamiento libre | 500 MB | 1 GB |
| GPU | No requerida | No requerida |
| Internet | Solo para instalar paquetes | 10 Mbps o superior |

---

### 🧰 Software

| Software / Paquete | Uso |
|---|---|
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Python 3.11 o superior | Runtime del servidor y cliente |
| pip | Instalación de dependencias |
| `mcp` | SDK oficial MCP para Python |
| `aiofiles` | Lectura y escritura asíncrona, disponible para extensiones posteriores |
| `python-dotenv` | Carga de variables de entorno |
| `pytest` | Validación opcional |
| `pathlib` | Validación segura de rutas |
| `logging` | Auditoría del servidor |

---

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 50 minutos |
| Complejidad | Intermedia - Alta |
| Nivel de Bloom | Aplicar, analizar, evaluar y crear |
| Modalidad | Individual o equipos de 2 personas |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Protocolo | Model Context Protocol |
| Framework MCP | FastMCP |
| Transporte | `stdio` |
| Costo API | $0 USD |
| Entregable principal | Servidor MCP seguro |
| Entregable secundario | Cliente MCP de prueba, logs y evaluación de seguridad |

---

## 🛡️ Consideraciones para estudiantes

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>El servidor opera solo dentro de un sandbox.</td>
<td><strong>🧪 Alcance</strong><br>No se usa API key ni modelo LLM en el flujo base.</td>
<td><strong>📜 Auditoría</strong><br>Los accesos se registran en logs.</td>
</tr>
</table>

1. No permitas rutas absolutas del sistema operativo.
2. No permitas que las herramientas lean fuera de `sandbox/`.
3. No permitas escritura fuera de `sandbox/output/`.
4. No uses `print()` para depurar el servidor MCP cuando uses transporte `stdio`.
5. Usa `logging` hacia archivo o `stderr`.
6. Valida rutas con `Path.resolve()` y `relative_to()`.
7. Restringe extensiones permitidas.
8. Limita tamaño de archivos y contenido.
9. Trata los errores controlados como parte esperada de las pruebas de seguridad.
10. No confundas un error bloqueado con una falla del laboratorio: si una ruta peligrosa se bloquea, el servidor funcionó correctamente.

> [!NOTE]
> Este laboratorio no usa ChromaDB. Si en tu terminal aparecen mensajes como `Failed to send telemetry event ClientStartEvent` o `ClientCreateCollectionEvent`, normalmente provienen de otra práctica, librería o ambiente reutilizado con ChromaDB. No corresponden al servidor MCP de este laboratorio.

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear la carpeta del laboratorio, abrirla en Visual Studio Code, crear el entorno virtual e instalar las dependencias necesarias para trabajar con MCP.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear una carpeta nueva para este laboratorio. Esta carpeta será la raíz del proyecto y ahí guardarás el servidor MCP, el cliente de prueba, la configuración, el sandbox, los logs y la evaluación de seguridad. Ejecuta estos comandos desde Git Bash; no necesitas crear archivos manualmente en este paso.

**⚙️ Contenido del paso:**

```bash
mkdir -p ~/labs-ia-gen/lab-09-mcp-filesystem
cd ~/labs-ia-gen/lab-09-mcp-filesystem
```

**✅ Validación del paso:**

```bash
pwd
```

**📌 Resultado esperado:**

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-09-mcp-filesystem
```

---

### ✅ Paso 2. Abre el proyecto en Visual Studio Code

**📝 Descripción del paso:**  
Vas a abrir en Visual Studio Code la carpeta `lab-09-mcp-filesystem` que acabas de crear. A partir de este punto, todos los archivos nuevos del laboratorio deben crearse dentro de esta carpeta.

**⚙️ Contenido del paso:**

```bash
code .
```

Si `code .` no funciona, abre VS Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-09-mcp-filesystem
```

**✅ Validación del paso:**  
Confirma que VS Code muestre la carpeta `lab-09-mcp-filesystem`.

**📌 Resultado esperado:**  
El proyecto está abierto en Visual Studio Code.

---

### ✅ Paso 3. Crea y activa el entorno virtual

**📝 Descripción del paso:**  
Vas a crear un entorno virtual llamado `.venv` dentro de la carpeta del laboratorio y después lo vas a activar. Este entorno permite instalar las librerías de MCP sin afectar otros proyectos de Python.

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
La ruta de Python apunta a `.venv/Scripts/python`.

---

### ✅ Paso 4. Crea `requirements.txt`

**📝 Descripción del paso:**  
Vas a crear el archivo `requirements.txt` en la raíz del proyecto. Este archivo contendrá la lista de paquetes Python necesarios para construir el servidor y cliente MCP.

**⚙️ Contenido del paso:**

```bash
cat > requirements.txt << 'REQ'
mcp>=1.9,<2
aiofiles>=24,<25
python-dotenv>=1.0,<2
pytest>=8,<9
REQ
```

**✅ Validación del paso:**

```bash
cat requirements.txt
```

**📌 Resultado esperado:**  
El archivo contiene las dependencias del laboratorio.

---

### ✅ Paso 5. Instala dependencias

**📝 Descripción del paso:**  
Vas a instalar dentro del entorno virtual activo todas las dependencias declaradas en `requirements.txt`. Antes de ejecutar este paso, confirma que Git Bash muestre `(.venv)` al inicio de la línea.

**⚙️ Contenido del paso:**

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**✅ Validación del paso:**

```bash
python - << 'PY'
import mcp
print("✅ MCP SDK instalado correctamente")
PY
```

**📌 Resultado esperado:**

```text
✅ MCP SDK instalado correctamente
```

---

### ✅ Paso 6. Valida la instalación del paquete MCP

**📝 Descripción del paso:**  
Vas a comprobar que `pip` reconoce el paquete `mcp` instalado en el entorno virtual. Este paso no crea archivos; solo valida instalación y versión.

**⚙️ Contenido del paso:**

```bash
pip show mcp
```

**✅ Validación del paso:**  
Debes ver información del paquete `mcp`, como nombre, versión y ubicación.

**📌 Resultado esperado:**  
El SDK MCP está instalado y disponible para los scripts del laboratorio.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20problema%20resuelve%20MCP%20y%20cu%C3%A1l%20es%20la%20diferencia%20entre%20Host%2C%20Client%20y%20Server%20en%20una%20arquitectura%20MCP.%20Tambi%C3%A9n%20expl%C3%ADcame%20por%20qu%C3%A9%20prepar%C3%A9%20un%20proyecto%20local%20con%20VS%20Code%2C%20Git%20Bash%2C%20entorno%20virtual%20y%20requirements.txt.)

---

# 🧩 Tarea 2. Crear sandbox, archivos de prueba y configuración

## 🎯 Objetivo de la tarea

Preparar un directorio controlado donde el servidor MCP podrá operar de forma segura, junto con archivos de prueba y configuración local.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la estructura del sandbox

**📝 Descripción del paso:**  
Vas a crear las carpetas base del laboratorio. `sandbox/docs/` contendrá archivos de documentación, `sandbox/data/` contendrá datos de prueba, `sandbox/output/` será el único directorio autorizado para escritura y `logs/` guardará la auditoría del servidor MCP.

**⚙️ Contenido del paso:**

```bash
mkdir -p sandbox/docs sandbox/data sandbox/output logs
```

**✅ Validación del paso:**

```bash
ls -la
```

**📌 Resultado esperado:**  
Debes ver las carpetas `sandbox/` y `logs/`.

---

### ✅ Paso 2. Crea `.gitignore`

**📝 Descripción del paso:**  
Vas a crear el archivo `.gitignore` en la raíz del proyecto. Este archivo evita subir a Git el entorno virtual, archivos de log, archivos compilados y salidas generadas por el servidor MCP.

**⚙️ Contenido del paso:**

```bash
cat > .gitignore << 'GITIGNORE'
.env
.venv/
__pycache__/
*.pyc
logs/
mcp_access.log
sandbox/output/
GITIGNORE
```

**✅ Validación del paso:**

```bash
cat .gitignore
```

**📌 Resultado esperado:**  
`.env`, `.venv/`, `logs/` y `sandbox/output/` están excluidos.

---

### ✅ Paso 3. Crea `.env`

**📝 Descripción del paso:**  
Vas a crear el archivo `.env` en la raíz del proyecto. Este archivo define el directorio sandbox, el tamaño máximo permitido para archivos y la ruta del log de auditoría.

**⚙️ Contenido del paso:**

```bash
cat > .env << 'ENV'
MCP_SANDBOX_DIR=./sandbox
MCP_MAX_FILE_SIZE_BYTES=1048576
MCP_LOG_FILE=logs/mcp_access.log
ENV
```

**✅ Validación del paso:**

```bash
cat .env
```

**📌 Resultado esperado:**  
El archivo `.env` contiene las variables de configuración del servidor MCP.

---

### ✅ Paso 4. Crea `sandbox/docs/readme.md`

**📝 Descripción del paso:**  
Vas a crear un archivo Markdown de prueba dentro de `sandbox/docs/`. Este archivo servirá para validar lectura, búsqueda de texto y metadatos.

**⚙️ Contenido del paso:**

```bash
cat > sandbox/docs/readme.md << 'MD'
# Documentación del Proyecto

Este archivo pertenece al sandbox del Laboratorio 9.
Contiene información de prueba para validar lectura, búsqueda y metadatos.

## Secciones
- Introducción
- Configuración
- Uso
MD
```

**✅ Validación del paso:**

```bash
cat sandbox/docs/readme.md
```

**📌 Resultado esperado:**  
El archivo `readme.md` existe y contiene texto de prueba.

---

### ✅ Paso 5. Crea `sandbox/data/sample.csv`

**📝 Descripción del paso:**  
Vas a crear un archivo CSV de prueba dentro de `sandbox/data/`. Este archivo permite validar que el servidor puede leer archivos permitidos con extensión `.csv`.

**⚙️ Contenido del paso:**

```bash
cat > sandbox/data/sample.csv << 'CSV'
id,nombre,valor
1,alpha,100
2,beta,200
3,gamma,300
CSV
```

**✅ Validación del paso:**

```bash
cat sandbox/data/sample.csv
```

**📌 Resultado esperado:**  
El archivo `sample.csv` existe dentro de `sandbox/data/`.

---

### ✅ Paso 6. Crea `sandbox/docs/config.py`

**📝 Descripción del paso:**  
Vas a crear un archivo Python de prueba dentro de `sandbox/docs/`. Este archivo se usará para probar búsquedas con patrones como `DEBUG`, `config` o `TIMEOUT`.

**⚙️ Contenido del paso:**

```bash
cat > sandbox/docs/config.py << 'PY'
# Configuración de ejemplo
DEBUG = True
MAX_RETRIES = 3
TIMEOUT = 30
PY
```

**✅ Validación del paso:**

```bash
find sandbox -maxdepth 3 -type f -print
```

**📌 Resultado esperado:**  
Debes ver estos archivos:

```text
sandbox/docs/readme.md
sandbox/docs/config.py
sandbox/data/sample.csv
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20un%20servidor%20MCP%20de%20sistema%20de%20archivos%20debe%20usar%20un%20sandbox%20y%20no%20permitir%20rutas%20absolutas%20del%20sistema%20operativo.%20Incluye%20riesgos%20de%20path%20traversal%20y%20m%C3%ADnimo%20privilegio.)

---

# 🧩 Tarea 3. Crear el servidor MCP y definir políticas de seguridad

## 🎯 Objetivo de la tarea

Crear el archivo principal `filesystem_mcp_server.py` y definir configuración, límites, logging, sandbox y objeto FastMCP.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `filesystem_mcp_server.py`

**📝 Descripción del paso:**  
Vas a crear el archivo principal del servidor MCP en la raíz del proyecto. Este archivo contendrá la configuración del servidor, las funciones de seguridad, las herramientas MCP, el resource, los prompts y el punto de entrada.

**⚙️ Contenido del paso:**

```bash
touch filesystem_mcp_server.py
```

**✅ Validación del paso:**

```bash
ls -la filesystem_mcp_server.py
```

**📌 Resultado esperado:**  
El archivo `filesystem_mcp_server.py` existe en la raíz del proyecto.

---

### ✅ Paso 2. Agrega la configuración inicial del servidor

**📝 Descripción del paso:**  
Vas a abrir `filesystem_mcp_server.py` en Visual Studio Code y pegar este bloque desde la primera línea. Este bloque carga variables desde `.env`, resuelve el sandbox, define extensiones permitidas, configura límites y prepara logging. No uses `print()` para depurar este servidor porque `stdout` pertenece al protocolo MCP cuando usas transporte `stdio`.

**⚙️ Contenido del paso:**

```python
# filesystem_mcp_server.py
"""
Servidor MCP seguro para operaciones controladas de sistema de archivos.
Lab 09-00-01 — Model Context Protocol

Regla importante:
- En transporte stdio, stdout pertenece al protocolo.
- Usa logging hacia archivo o stderr, nunca print() para depurar el servidor.
"""

from __future__ import annotations

import json
import logging
import mimetypes
import os
import re
import stat
from datetime import datetime
from pathlib import Path
from typing import Any

from dotenv import load_dotenv
from mcp.server.fastmcp import FastMCP

load_dotenv()

RAW_SANDBOX_DIR = os.getenv("MCP_SANDBOX_DIR", "./sandbox")
SANDBOX_DIR = Path(RAW_SANDBOX_DIR).resolve()
MAX_FILE_SIZE = int(os.getenv("MCP_MAX_FILE_SIZE_BYTES", "1048576"))
LOG_FILE = os.getenv("MCP_LOG_FILE", "logs/mcp_access.log")

ALLOWED_EXTENSIONS = {".txt", ".md", ".py", ".json", ".csv"}
MAX_REGEX_LENGTH = 120
MAX_SEARCH_RESULTS = 50

Path(LOG_FILE).parent.mkdir(parents=True, exist_ok=True)

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.FileHandler(LOG_FILE, encoding="utf-8"),
        logging.StreamHandler(),
    ],
)

access_log = logging.getLogger("mcp_access")
mcp = FastMCP("secure-filesystem-mcp-server")
```

**✅ Validación del paso:**

```bash
python -m py_compile filesystem_mcp_server.py && echo "✅ Sintaxis inicial OK"
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20MCP%20no%20reemplaza%20los%20controles%20de%20autorizaci%C3%B3n%20y%20por%20qu%C3%A9%20cada%20herramienta%20debe%20validar%20sus%20propios%20permisos.%20Incluye%20el%20riesgo%20de%20usar%20stdout%20con%20transporte%20stdio.)

---

# 🧩 Tarea 4. Implementar utilidades de seguridad

## 🎯 Objetivo de la tarea

Agregar funciones reutilizables para validar rutas, extensiones, tamaños y contenido antes de permitir operaciones de archivo.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega funciones de seguridad en `filesystem_mcp_server.py`

**📝 Descripción del paso:**  
Vas a abrir `filesystem_mcp_server.py` y agregar este bloque después de la línea `mcp = FastMCP("secure-filesystem-mcp-server")`. No reemplaces la configuración anterior; solo agrega estas funciones debajo. Estas utilidades serán usadas por todas las herramientas MCP.

**⚙️ Contenido del paso:**

```python
def validate_path(raw_path: str, must_exist: bool = True) -> Path:
    """
    Resuelve una ruta relativa al sandbox y valida que no escape de él.
    Bloquea path traversal, rutas absolutas y rutas inexistentes cuando aplica.
    """
    if raw_path is None:
        raise ValueError("La ruta no puede ser None.")

    raw_path = str(raw_path).strip()
    if raw_path == "":
        raw_path = "."

    candidate = (SANDBOX_DIR / raw_path).resolve()

    try:
        candidate.relative_to(SANDBOX_DIR)
    except ValueError as exc:
        access_log.warning(
            "PATH_TRAVERSAL_ATTEMPT | raw='%s' | resolved='%s' | sandbox='%s'",
            raw_path,
            candidate,
            SANDBOX_DIR,
        )
        raise ValueError(
            f"Acceso denegado: la ruta '{raw_path}' intenta salir del sandbox."
        ) from exc

    if must_exist and not candidate.exists():
        access_log.info("PATH_NOT_FOUND | path='%s'", candidate)
        raise FileNotFoundError(f"No existe la ruta: {raw_path}")

    access_log.info("PATH_VALIDATED | raw='%s' | resolved='%s'", raw_path, candidate)
    return candidate


def validate_extension(file_path: Path) -> None:
    """Valida que la extensión del archivo esté permitida."""
    ext = file_path.suffix.lower()
    if ext not in ALLOWED_EXTENSIONS:
        access_log.warning("EXTENSION_DENIED | file='%s' | ext='%s'", file_path, ext)
        raise ValueError(
            f"Extensión '{ext}' no permitida. Permitidas: {', '.join(sorted(ALLOWED_EXTENSIONS))}"
        )


def validate_file_size(file_path: Path) -> None:
    """Valida que el tamaño de un archivo existente esté dentro del límite."""
    size = file_path.stat().st_size
    if size > MAX_FILE_SIZE:
        raise ValueError(
            f"Archivo demasiado grande: {size} bytes. Máximo permitido: {MAX_FILE_SIZE} bytes."
        )


def validate_content_size(content: str) -> None:
    """Valida que el contenido a escribir no exceda el tamaño máximo."""
    size = len(content.encode("utf-8"))
    if size > MAX_FILE_SIZE:
        raise ValueError(
            f"Contenido demasiado grande: {size} bytes. Máximo permitido: {MAX_FILE_SIZE} bytes."
        )


def human_size(size_bytes: int) -> str:
    """Convierte bytes a una representación legible."""
    value = float(size_bytes)
    for unit in ("B", "KB", "MB", "GB"):
        if value < 1024:
            return f"{value:.1f} {unit}"
        value /= 1024
    return f"{value:.1f} TB"
```

**✅ Validación del paso:**

```bash
python - << 'PY'
from filesystem_mcp_server import validate_path

print(validate_path("docs/readme.md"))

try:
    validate_path("../../../etc/passwd")
except Exception as exc:
    print("✅ Bloqueado:", exc)
PY
```

**📌 Resultado esperado:**  
La ruta válida se resuelve dentro del sandbox y el intento de path traversal queda bloqueado.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20c%C3%B3mo%20Path.resolve%28%29%20y%20Path.relative_to%28%29%20ayudan%20a%20prevenir%20path%20traversal%20en%20Python.%20Usa%20ejemplos%20de%20rutas%20v%C3%A1lidas%20y%20rutas%20maliciosas.)

---

# 🧩 Tarea 5. Implementar herramientas MCP seguras

## 🎯 Objetivo de la tarea

Exponer cinco herramientas MCP seguras: listar archivos, leer archivos, escribir archivos, buscar texto y consultar metadatos.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega herramientas MCP en `filesystem_mcp_server.py`

**📝 Descripción del paso:**  
Vas a abrir `filesystem_mcp_server.py` y agregar este bloque después de las funciones de seguridad de la tarea anterior. Este bloque registra las tools que el cliente MCP podrá descubrir y ejecutar. Cada tool valida ruta, extensión, tamaño o ubicación antes de operar.

**⚙️ Contenido del paso:**

```python
@mcp.tool()
def list_files(directory: str = ".") -> str:
    """Lista archivos y subdirectorios dentro de un directorio del sandbox."""
    dir_path = validate_path(directory)
    if not dir_path.is_dir():
        raise ValueError(f"'{directory}' no es un directorio.")

    entries = []
    for entry in sorted(dir_path.iterdir()):
        rel = entry.relative_to(SANDBOX_DIR)
        icon = "📁" if entry.is_dir() else "📄"
        entries.append(f"{icon} {rel}")

    access_log.info("LIST_FILES | directory='%s' | count=%d", directory, len(entries))
    if not entries:
        return f"El directorio '{directory}' está vacío."
    return f"Contenido de '{directory}' ({len(entries)} entradas):\n" + "\n".join(entries)


@mcp.tool()
def read_file(filepath: str) -> str:
    """Lee archivos de texto permitidos dentro del sandbox."""
    file_path = validate_path(filepath)
    if not file_path.is_file():
        raise ValueError(f"'{filepath}' no es un archivo regular.")

    validate_extension(file_path)
    validate_file_size(file_path)

    content = file_path.read_text(encoding="utf-8", errors="replace")
    access_log.info("READ_FILE | filepath='%s' | size=%d", filepath, file_path.stat().st_size)
    return f"📄 Contenido de '{filepath}' ({file_path.stat().st_size} bytes):\n\n{content}"


@mcp.tool()
def write_file(filepath: str, content: str, overwrite_confirmed: bool = False) -> str:
    """
    Escribe un archivo dentro de sandbox/output/.
    Bloquea escritura fuera de output, extensiones no permitidas y contenido demasiado grande.
    """
    if not filepath.startswith("output/") and not filepath.startswith("output\\"):
        raise ValueError(
            "Solo se permite escribir dentro de 'output/'. "
            f"Ruta recibida: '{filepath}'"
        )

    file_path = validate_path(filepath, must_exist=False)
    validate_extension(file_path)
    validate_content_size(content)

    if file_path.exists() and not overwrite_confirmed:
        access_log.warning("WRITE_BLOCKED_OVERWRITE | filepath='%s'", filepath)
        return (
            f"⚠️ El archivo '{filepath}' ya existe. "
            "Para sobreescribirlo, llama de nuevo con overwrite_confirmed=true."
        )

    file_path.parent.mkdir(parents=True, exist_ok=True)
    file_path.write_text(content, encoding="utf-8")

    access_log.info(
        "WRITE_FILE | filepath='%s' | size=%d | overwrite=%s",
        filepath,
        len(content.encode("utf-8")),
        overwrite_confirmed,
    )
    return f"✅ Archivo '{filepath}' escrito correctamente."


@mcp.tool()
def search_in_files(pattern: str, directory: str = ".") -> str:
    """
    Busca un patrón regex en archivos permitidos dentro del sandbox.
    Limita longitud del regex y número de resultados.
    """
    if len(pattern) > MAX_REGEX_LENGTH:
        raise ValueError(
            f"El patrón es demasiado largo: {len(pattern)} caracteres. Máximo: {MAX_REGEX_LENGTH}."
        )

    try:
        compiled = re.compile(pattern, re.IGNORECASE)
    except re.error as exc:
        raise ValueError(f"Regex inválida: {exc}") from exc

    dir_path = validate_path(directory)
    if not dir_path.is_dir():
        raise ValueError(f"'{directory}' no es un directorio.")

    matches: list[str] = []
    scanned = 0

    for file_path in sorted(dir_path.rglob("*")):
        if not file_path.is_file():
            continue
        if file_path.suffix.lower() not in ALLOWED_EXTENSIONS:
            continue
        if file_path.stat().st_size > MAX_FILE_SIZE:
            continue

        scanned += 1
        lines = file_path.read_text(encoding="utf-8", errors="replace").splitlines()
        for line_number, line in enumerate(lines, start=1):
            if compiled.search(line):
                rel = file_path.relative_to(SANDBOX_DIR)
                matches.append(f"📄 {rel}:{line_number} → {line}")
                if len(matches) >= MAX_SEARCH_RESULTS:
                    break
        if len(matches) >= MAX_SEARCH_RESULTS:
            break

    access_log.info(
        "SEARCH_IN_FILES | pattern='%s' | directory='%s' | scanned=%d | matches=%d",
        pattern,
        directory,
        scanned,
        len(matches),
    )

    if not matches:
        return f"🔍 Sin coincidencias para '{pattern}' en '{directory}'. Archivos analizados: {scanned}."

    return (
        f"🔍 Coincidencias para '{pattern}' en '{directory}' "
        f"({len(matches)} resultados, {scanned} archivos analizados):\n"
        + "\n".join(matches)
    )


@mcp.tool()
def get_file_metadata(filepath: str) -> str:
    """Retorna metadatos de un archivo dentro del sandbox."""
    file_path = validate_path(filepath)
    if not file_path.is_file():
        raise ValueError(f"'{filepath}' no es un archivo regular.")

    stat_info = file_path.stat()
    mime_type, _ = mimetypes.guess_type(str(file_path))
    metadata: dict[str, Any] = {
        "filepath": filepath,
        "size_bytes": stat_info.st_size,
        "size_human": human_size(stat_info.st_size),
        "mime_type": mime_type or "application/octet-stream",
        "modified_at": datetime.fromtimestamp(stat_info.st_mtime).isoformat(),
        "permissions": oct(stat.S_IMODE(stat_info.st_mode)),
        "extension": file_path.suffix.lower(),
        "is_allowed_extension": file_path.suffix.lower() in ALLOWED_EXTENSIONS,
    }

    access_log.info("GET_FILE_METADATA | filepath='%s' | size=%d", filepath, stat_info.st_size)
    return "📊 Metadata:\n" + json.dumps(metadata, indent=2, ensure_ascii=False)
```

**✅ Validación del paso:**

```bash
python -m py_compile filesystem_mcp_server.py && echo "✅ Tools compiladas"
```

**📌 Resultado esperado:**  
El archivo compila sin errores y las herramientas quedan registradas en el servidor.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Revisa%20conceptualmente%20herramientas%20MCP%20para%20listar%2C%20leer%2C%20escribir%2C%20buscar%20y%20consultar%20metadatos.%20Expl%C3%ADcame%20qu%C3%A9%20controles%20de%20seguridad%20deben%20aplicarse%20antes%20de%20leer%20o%20escribir%20archivos.)

---

# 🧩 Tarea 6. Implementar Resource y Prompts MCP

## 🎯 Objetivo de la tarea

Exponer un resource MCP con el árbol del sandbox y dos prompts reutilizables para análisis de directorios y búsqueda de patrones.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega resource y prompts en `filesystem_mcp_server.py`

**📝 Descripción del paso:**  
Vas a abrir `filesystem_mcp_server.py` y agregar este bloque después de las herramientas MCP. El resource permite consultar el árbol del sandbox y los prompts ofrecen instrucciones reutilizables que un cliente compatible puede descubrir.

**⚙️ Contenido del paso:**

```python
def build_tree(path: Path, prefix: str = "") -> list[str]:
    """Construye una representación textual del árbol de directorios."""
    lines: list[str] = []
    entries = sorted(path.iterdir())

    for index, entry in enumerate(entries):
        connector = "└── " if index == len(entries) - 1 else "├── "
        icon = "📁" if entry.is_dir() else "📄"
        lines.append(f"{prefix}{connector}{icon} {entry.name}")
        if entry.is_dir():
            extension = "    " if index == len(entries) - 1 else "│   "
            lines.extend(build_tree(entry, prefix + extension))

    return lines


@mcp.resource("file://sandbox/tree")
def sandbox_tree() -> str:
    """Expone el árbol del sandbox como recurso MCP."""
    access_log.info("READ_RESOURCE | uri='file://sandbox/tree'")
    lines = [f"📁 SANDBOX: {SANDBOX_DIR}"]
    lines.extend(build_tree(SANDBOX_DIR))
    return "\n".join(lines)


@mcp.prompt()
def analizar_directorio(directorio: str = ".") -> str:
    """Prompt reutilizable para analizar un directorio del sandbox."""
    return (
        f"Analiza el directorio '{directorio}' dentro del sandbox MCP. "
        "Primero lista su contenido, después revisa los archivos relevantes, "
        "consulta metadatos cuando sea útil y entrega un resumen con: "
        "tipos de archivo, propósito probable, riesgos y recomendaciones de organización."
    )


@mcp.prompt()
def buscar_y_reportar(patron: str, directorio: str = ".") -> str:
    """Prompt reutilizable para buscar un patrón y reportar resultados."""
    return (
        f"Busca el patrón '{patron}' en el directorio '{directorio}' usando las herramientas MCP. "
        "Entrega un reporte con número de coincidencias, archivos afectados, líneas relevantes "
        "y conclusiones. No escribas archivos a menos que el usuario lo confirme."
    )
```

**✅ Validación del paso:**

```bash
python -m py_compile filesystem_mcp_server.py && echo "✅ Resource y prompts compilados"
```

**📌 Resultado esperado:**  
El servidor compila y ahora contiene tools, un resource y dos prompts.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20cu%C3%A1l%20es%20la%20diferencia%20pr%C3%A1ctica%20entre%20una%20tool%2C%20un%20resource%20y%20un%20prompt%20dentro%20de%20MCP.%20Usa%20ejemplos%20del%20laboratorio%20de%20sistema%20de%20archivos.)

---

# 🧩 Tarea 7. Agregar punto de entrada del servidor MCP

## 🎯 Objetivo de la tarea

Agregar la función `main()` para iniciar el servidor MCP usando transporte `stdio`.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega `main()` al final de `filesystem_mcp_server.py`

**📝 Descripción del paso:**  
Vas a abrir `filesystem_mcp_server.py` y agregar este bloque al final del archivo. Esta función valida que el sandbox exista, asegura la carpeta `output/`, registra el inicio del servidor y ejecuta MCP usando transporte `stdio`.

**⚙️ Contenido del paso:**

```python
def main() -> None:
    """Inicia el servidor MCP usando transporte stdio."""
    if not SANDBOX_DIR.exists():
        raise RuntimeError(f"No existe el sandbox: {SANDBOX_DIR}")

    (SANDBOX_DIR / "output").mkdir(parents=True, exist_ok=True)

    access_log.info(
        "SERVER_START | sandbox='%s' | max_file_size=%d | allowed_extensions=%s",
        SANDBOX_DIR,
        MAX_FILE_SIZE,
        sorted(ALLOWED_EXTENSIONS),
    )

    mcp.run(transport="stdio")


if __name__ == "__main__":
    main()
```

**✅ Validación del paso:**

```bash
python -m py_compile filesystem_mcp_server.py && echo "✅ Servidor listo"
```

**📌 Resultado esperado:**  
El servidor compila. No lo ejecutes manualmente por largo tiempo porque quedará esperando mensajes MCP por `stdio`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20por%20qu%C3%A9%20un%20servidor%20MCP%20con%20transporte%20stdio%20se%20queda%20esperando%20entrada%20y%20no%20muestra%20una%20interfaz%20interactiva%20normal.%20Tambi%C3%A9n%20expl%C3%ADcame%20por%20qu%C3%A9%20no%20debo%20usar%20print%20para%20depurar%20el%20servidor.)

---

# 🧩 Tarea 8. Crear cliente MCP de prueba

## 🎯 Objetivo de la tarea

Crear un cliente que lance el servidor como subproceso, haga handshake MCP, descubra capacidades y ejecute pruebas funcionales y de seguridad.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `test_mcp_client.py`

**📝 Descripción del paso:**  
Vas a crear el archivo `test_mcp_client.py` en la raíz del proyecto. Este archivo será el cliente de prueba que iniciará el servidor MCP como subproceso por `stdio`.

**⚙️ Contenido del paso:**

```bash
touch test_mcp_client.py
```

**✅ Validación del paso:**

```bash
ls -la test_mcp_client.py
```

**📌 Resultado esperado:**  
El archivo `test_mcp_client.py` existe.

---

### ✅ Paso 2. Agrega el cliente de prueba

**📝 Descripción del paso:**  
Vas a abrir `test_mcp_client.py` en Visual Studio Code y pegar este código completo. El cliente validará handshake, tools, resource, prompts, lectura, escritura controlada, búsqueda, metadatos y bloqueos de seguridad.

**⚙️ Contenido del paso:**

```python
# test_mcp_client.py
"""
Cliente de prueba para el servidor MCP filesystem.
Ejecuta descubrimiento de capacidades, pruebas funcionales y pruebas de seguridad.
"""

from __future__ import annotations

import asyncio
import sys
from pathlib import Path

from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

SERVER_SCRIPT = Path(__file__).parent / "filesystem_mcp_server.py"


def get_text(result) -> str:
    """Extrae texto de una respuesta MCP de forma tolerante."""
    if not getattr(result, "content", None):
        return ""
    first = result.content[0]
    return getattr(first, "text", str(first))


def show(title: str) -> None:
    print("\n" + "=" * 70)
    print(title)
    print("=" * 70)


def mark(ok: bool, label: str, detail: str = "") -> bool:
    icon = "✅" if ok else "❌"
    print(f"{icon} {label}")
    if detail:
        print("   " + detail[:400].replace("\n", "\n   "))
    return ok


async def main() -> int:
    results = {"passed": 0, "failed": 0}

    server_params = StdioServerParameters(
        command=sys.executable,
        args=[str(SERVER_SCRIPT)],
        env=None,
    )

    show("Fase 1 — Inicialización MCP")

    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            results["passed"] += mark(True, "Handshake MCP completado")

            show("Fase 2 — Descubrimiento de capacidades")

            tools = await session.list_tools()
            tool_names = {tool.name for tool in tools.tools}
            expected = {
                "list_files",
                "read_file",
                "write_file",
                "search_in_files",
                "get_file_metadata",
            }
            ok = expected.issubset(tool_names)
            results["passed" if ok else "failed"] += mark(
                ok,
                "Tools registradas",
                f"Encontradas: {sorted(tool_names)}",
            )

            resources = await session.list_resources()
            resource_uris = [str(resource.uri) for resource in resources.resources]
            ok = "file://sandbox/tree" in resource_uris
            results["passed" if ok else "failed"] += mark(
                ok,
                "Resource registrado",
                f"URIs: {resource_uris}",
            )

            prompts = await session.list_prompts()
            prompt_names = {prompt.name for prompt in prompts.prompts}
            ok = {"analizar_directorio", "buscar_y_reportar"}.issubset(prompt_names)
            results["passed" if ok else "failed"] += mark(
                ok,
                "Prompts registrados",
                f"Prompts: {sorted(prompt_names)}",
            )

            show("Fase 3 — Pruebas funcionales")

            checks = [
                (
                    "list_files raíz",
                    "list_files",
                    {"directory": "."},
                    lambda text: "docs" in text and "data" in text,
                ),
                (
                    "read_file readme",
                    "read_file",
                    {"filepath": "docs/readme.md"},
                    lambda text: "Documentación" in text or "sandbox" in text,
                ),
                (
                    "write_file output",
                    "write_file",
                    {
                        "filepath": "output/test_output.txt",
                        "content": "Archivo generado por test_mcp_client.py",
                        "overwrite_confirmed": False,
                    },
                    lambda text: "escrito" in text.lower() or "existe" in text.lower(),
                ),
                (
                    "search_in_files",
                    "search_in_files",
                    {"pattern": "DEBUG|config|id", "directory": "."},
                    lambda text: "coincid" in text.lower() or "sin coincidencias" in text.lower(),
                ),
                (
                    "get_file_metadata",
                    "get_file_metadata",
                    {"filepath": "data/sample.csv"},
                    lambda text: "size_bytes" in text and "mime_type" in text,
                ),
            ]

            for label, tool_name, args, validator in checks:
                response = await session.call_tool(tool_name, args)
                text = get_text(response)
                ok = validator(text)
                results["passed" if ok else "failed"] += mark(ok, label, text)

            show("Fase 4 — Pruebas de seguridad")

            malicious_reads = [
                "../../../etc/passwd",
                "../../.env",
                "/etc/hosts",
                "docs/../../../../tmp/evil.txt",
            ]

            for path in malicious_reads:
                response = await session.call_tool("read_file", {"filepath": path})
                text = get_text(response)
                ok = "denegado" in text.lower() or "error" in text.lower()
                results["passed" if ok else "failed"] += mark(
                    ok,
                    f"Bloqueo de path traversal: {path}",
                    text,
                )

            response = await session.call_tool(
                "write_file",
                {
                    "filepath": "docs/no_autorizado.md",
                    "content": "No debería escribirse aquí",
                    "overwrite_confirmed": True,
                },
            )
            text = get_text(response)
            ok = "output" in text.lower() or "error" in text.lower()
            results["passed" if ok else "failed"] += mark(
                ok,
                "Bloqueo de escritura fuera de output/",
                text,
            )

            response = await session.call_tool(
                "write_file",
                {
                    "filepath": "output/no_autorizado.exe",
                    "content": "binario falso",
                    "overwrite_confirmed": True,
                },
            )
            text = get_text(response)
            ok = "extensión" in text.lower() or "extension" in text.lower() or "error" in text.lower()
            results["passed" if ok else "failed"] += mark(
                ok,
                "Bloqueo de extensión no permitida",
                text,
            )

            response = await session.call_tool(
                "search_in_files",
                {"pattern": "(" * 10, "directory": "."},
            )
            text = get_text(response)
            ok = "regex" in text.lower() or "error" in text.lower()
            results["passed" if ok else "failed"] += mark(
                ok,
                "Error controlado con regex inválida",
                text,
            )

            show("Fase 5 — Resource MCP")

            resource = await session.read_resource("file://sandbox/tree")
            resource_text = ""
            if resource.contents:
                first = resource.contents[0]
                resource_text = getattr(first, "text", str(first))
            ok = "SANDBOX" in resource_text or "docs" in resource_text
            results["passed" if ok else "failed"] += mark(
                ok,
                "Lectura de file://sandbox/tree",
                resource_text,
            )

    show("Resumen")
    total = results["passed"] + results["failed"]
    print(f"✅ Pasadas: {results['passed']}")
    print(f"❌ Fallidas: {results['failed']}")
    print(f"📊 Total: {total}")

    return 0 if results["failed"] == 0 else 1


if __name__ == "__main__":
    raise SystemExit(asyncio.run(main()))
```

**✅ Validación del paso:**

```bash
python -m py_compile test_mcp_client.py && echo "✅ Cliente compilado"
```

**📌 Resultado esperado:**  
El cliente compila sin errores.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20c%C3%B3mo%20un%20cliente%20MCP%20descubre%20tools%2C%20resources%20y%20prompts%20antes%20de%20invocar%20una%20herramienta.%20Incluye%20handshake%2C%20list_tools%2C%20list_resources%20y%20list_prompts.)

---

# 🧩 Tarea 9. Ejecutar pruebas funcionales y de seguridad

## 🎯 Objetivo de la tarea

Validar que el servidor MCP funciona correctamente y bloquea operaciones peligrosas.

---

## 🛠️ Pasos

### ✅ Paso 1. Ejecuta el cliente de prueba

**📝 Descripción del paso:**  
Vas a ejecutar `test_mcp_client.py` desde Git Bash con el entorno virtual activo. No ejecutes directamente `filesystem_mcp_server.py`; el cliente lo iniciará automáticamente como subproceso MCP por `stdio`.

**⚙️ Contenido del paso:**

```bash
python test_mcp_client.py
```

**✅ Validación del paso:**  
La salida debe mostrar fases de inicialización, descubrimiento, pruebas funcionales, pruebas de seguridad y resource MCP.

**📌 Resultado esperado:**  
Las pruebas funcionales deben pasar y las pruebas maliciosas deben quedar bloqueadas con errores controlados.

---

### ✅ Paso 2. Verifica el log de auditoría

**📝 Descripción del paso:**  
Vas a comprobar que el servidor generó `logs/mcp_access.log`. Este archivo sirve como evidencia de auditoría de operaciones permitidas y bloqueadas.

**⚙️ Contenido del paso:**

```bash
ls -lh logs/mcp_access.log
```

**✅ Validación del paso:**  
Debe existir el archivo `logs/mcp_access.log`.

**📌 Resultado esperado:**  
El log fue creado correctamente.

---

### ✅ Paso 3. Busca eventos de seguridad

**📝 Descripción del paso:**  
Vas a buscar dentro del log eventos relacionados con path traversal, extensiones no permitidas y escrituras bloqueadas. Estos eventos son evidencia de que el servidor falla de forma segura.

**⚙️ Contenido del paso:**

```bash
grep -E "PATH_TRAVERSAL_ATTEMPT|EXTENSION_DENIED|WRITE_BLOCKED" logs/mcp_access.log || true
```

**✅ Validación del paso:**  
Debes ver eventos relacionados con intentos bloqueados o el comando debe finalizar sin romper la terminal.

**📌 Resultado esperado:**  
El servidor registra eventos de seguridad.

---

### ✅ Paso 4. Valida escritura controlada y path traversal

**📝 Descripción del paso:**  
Vas a ejecutar validaciones adicionales desde Git Bash. La primera confirma que la herramienta escribió un archivo dentro de `sandbox/output/`. La segunda confirma que el intento de path traversal quedó registrado.

**⚙️ Contenido del paso:**

```bash
[ -f sandbox/output/test_output.txt ] && echo "✅ Archivo de salida creado" || echo "❌ No se creó archivo"
grep -q "PATH_TRAVERSAL_ATTEMPT" logs/mcp_access.log && echo "✅ Traversal registrado" || echo "⚠️ Revisa log"
```

**✅ Validación del paso:**  
Debes ver mensajes de confirmación.

**📌 Resultado esperado:**  
La escritura controlada funciona y los intentos peligrosos se registran.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 9 en ChatGPT](https://chatgpt.com/?q=Ay%C3%BAdame%20a%20interpretar%20un%20log%20MCP%20y%20dime%20qu%C3%A9%20eventos%20muestran%20que%20el%20servidor%20bloque%C3%B3%20accesos%20peligrosos%20como%20path%20traversal%2C%20extensiones%20no%20permitidas%20o%20escrituras%20fuera%20de%20output.)

---

# 🧩 Tarea 10. Analizar resultados y comparar MCP contra function calling directo

## 🎯 Objetivo de la tarea

Documentar los resultados de seguridad y comparar cuándo conviene usar MCP frente a herramientas integradas directamente en un agente.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `evaluacion_seguridad.md`

**📝 Descripción del paso:**  
Vas a crear el archivo `evaluacion_seguridad.md` en la raíz del proyecto. Este archivo será la evidencia final donde documentarás pruebas permitidas, bloqueadas, errores controlados y comparación entre MCP y function calling directo.

**⚙️ Contenido del paso:**

```bash
cat > evaluacion_seguridad.md << 'MD'
# Evaluación de Seguridad — Laboratorio 9 MCP

| Prueba | Resultado esperado | Resultado obtenido | Estado |
|---|---|---|---|
| Leer `docs/readme.md` | Permitido |  |  |
| Leer `../../../etc/passwd` | Bloqueado |  |  |
| Leer `../../.env` | Bloqueado |  |  |
| Escribir en `docs/` | Bloqueado |  |  |
| Escribir en `output/` | Permitido |  |  |
| Escribir `.exe` en `output/` | Bloqueado |  |  |
| Buscar regex inválida | Error controlado |  |  |
| Leer `file://sandbox/tree` | Permitido |  |  |

## Comparación MCP vs Function Calling directo

| Criterio | Function Calling directo | MCP |
|---|---|---|
| Reutilización entre hosts |  |  |
| Separación cliente/servidor |  |  |
| Descubrimiento de capacidades |  |  |
| Seguridad por servidor |  |  |
| Complejidad inicial |  |  |

## Conclusión

Escribe aquí tu conclusión sobre cuándo usarías MCP y cuándo usarías function calling directo.
MD
```

**✅ Validación del paso:**

```bash
ls -la evaluacion_seguridad.md
```

**📌 Resultado esperado:**  
El archivo existe y contiene la matriz de evaluación.

---

### ✅ Paso 2. Completa la evaluación manualmente

**📝 Descripción del paso:**  
Vas a abrir `evaluacion_seguridad.md` en VS Code y completar las columnas vacías usando la salida de `python test_mcp_client.py` y los eventos de `logs/mcp_access.log`. No inventes resultados; registra si cada prueba fue permitida, bloqueada o generó un error controlado.

**⚙️ Contenido del paso:**

```bash
code evaluacion_seguridad.md
```

**✅ Validación del paso:**  
La matriz debe tener resultados y estados completados.

**📌 Resultado esperado:**  
Tienes evidencia escrita de que el servidor funciona y falla de forma segura.

---

### ✅ Paso 3. Revisa la evaluación desde Git Bash

**📝 Descripción del paso:**  
Vas a imprimir el archivo final para confirmar que la evidencia quedó guardada. Este paso no modifica archivos; solo revisa el contenido.

**⚙️ Contenido del paso:**

```bash
cat evaluacion_seguridad.md
```

**✅ Validación del paso:**  
Debe mostrarse la matriz de seguridad y la comparación MCP vs function calling directo.

**📌 Resultado esperado:**  
El laboratorio cuenta con una evidencia final documentada.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 10 en ChatGPT](https://chatgpt.com/?q=Con%20base%20en%20una%20matriz%20de%20seguridad%20de%20un%20servidor%20MCP%2C%20ay%C3%BAdame%20a%20redactar%20una%20conclusi%C3%B3n%20breve%20sobre%20cu%C3%A1ndo%20conviene%20MCP%20frente%20a%20function%20calling%20directo.)

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Dependencias MCP instaladas.
4. Archivo `.env` con configuración local del servidor.
5. Archivo `.gitignore` protegiendo entorno, logs y salidas.
6. Sandbox con carpetas `docs`, `data` y `output`.
7. Archivo `filesystem_mcp_server.py` implementado.
8. Servidor MCP con cinco tools funcionales.
9. Resource MCP `file://sandbox/tree`.
10. Prompts MCP `analizar_directorio` y `buscar_y_reportar`.
11. Cliente de prueba `test_mcp_client.py`.
12. Pruebas funcionales ejecutadas.
13. Pruebas de seguridad ejecutadas.
14. Log `logs/mcp_access.log` generado.
15. Archivo `evaluacion_seguridad.md` documentado.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del ambiente local | 10% |
| Configuración segura del sandbox | 10% |
| Validación de rutas y path traversal | 15% |
| Implementación de tools MCP | 20% |
| Implementación de resource y prompts | 10% |
| Cliente MCP de prueba funcional | 15% |
| Logging y auditoría | 10% |
| Evaluación MCP vs function calling directo | 10% |
| Total | 100% |

---

# ⚠️ Errores comunes que debes evitar

1. Ejecutar el servidor directamente y esperar una interfaz interactiva.
2. Usar `print()` dentro del servidor MCP con transporte `stdio`.
3. Permitir rutas absolutas como `/etc/hosts` o `C:\\Windows\\...`.
4. Permitir `../` sin validar con `Path.resolve()` y `relative_to()`.
5. Escribir archivos fuera de `sandbox/output/`.
6. Permitir extensiones no controladas como `.exe`.
7. No revisar `logs/mcp_access.log`.
8. Confundir un bloqueo de seguridad con una falla de la práctica.
9. Olvidar activar `.venv` antes de ejecutar el cliente.
10. Subir `.env`, `.venv/`, logs o salidas generadas a un repositorio.

---

# 🧯 Solución de problemas

## Problema 1. El cliente se queda colgado

**Causa probable:**  
El servidor está escribiendo texto en `stdout`, contaminando el canal del protocolo MCP.

**Solución:**

```bash
grep -n "print(" filesystem_mcp_server.py
```

Si encuentras `print()`, elimínalo o cámbialo por logging hacia archivo o `stderr`.

---

## Problema 2. `ImportError` con `FastMCP`

**Causa probable:**  
El SDK MCP no está instalado o la versión no contiene `FastMCP`.

**Solución:**

```bash
pip show mcp
pip install --upgrade "mcp>=1.9,<2"
python - << 'PY'
from mcp.server.fastmcp import FastMCP
print("✅ FastMCP disponible")
PY
```

---

## Problema 3. `ModuleNotFoundError: No module named 'mcp'`

**Causa probable:**  
El entorno virtual no está activo o no instalaste dependencias.

**Solución:**

```bash
source .venv/Scripts/activate
pip install -r requirements.txt
```

---

## Problema 4. La prueba de `.env` falla

**Causa probable:**  
`.gitignore` no contiene `.env` como línea independiente.

**Solución:**

```bash
echo ".env" >> .gitignore
grep -q "^\\.env$" .gitignore && echo "✅ .env protegido"
```

---

## Problema 5. `PATH_TRAVERSAL_ATTEMPT` no aparece en el log

**Causa probable:**  
No ejecutaste las pruebas de seguridad o el log está en otra ruta.

**Solución:**

```bash
python test_mcp_client.py
cat logs/mcp_access.log | grep PATH_TRAVERSAL_ATTEMPT
```

---

# 🧹 Limpieza del entorno

Ejecuta estos comandos si deseas limpiar archivos generados:

```bash
rm -f sandbox/output/test_output.txt
rm -f logs/mcp_access.log
find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true
find . -name "*.pyc" -delete 2>/dev/null || true
```

Para desactivar el entorno virtual:

```bash
deactivate
```

Para eliminar el entorno virtual completo:

```bash
rm -rf .venv/
```

Antes de compartir el proyecto, valida que no haya archivos sensibles:

```bash
git status
grep -r "sk-" . --include="*.py" --include="*.md" --include="*.txt" 2>/dev/null || echo "Sin API keys detectadas"
```

---

# 📚 Resumen conceptual

En este laboratorio construiste un servidor MCP seguro y reutilizable. La arquitectura final separa responsabilidades:

| Capa | Tecnología | Función |
|---|---|---|
| Protocolo | MCP | Estandarizar comunicación entre cliente y servidor |
| Servidor | FastMCP | Exponer tools, resources y prompts |
| Seguridad | `pathlib`, listas blancas y límites | Restringir rutas, extensiones, tamaños y escritura |
| Auditoría | `logging` | Registrar operaciones permitidas y bloqueadas |
| Cliente | `ClientSession` + `stdio_client` | Descubrir capacidades y ejecutar pruebas |

La clave del diseño está en que MCP estandariza la forma de exponer capacidades, pero no reemplaza la seguridad del servidor. Por eso validaste rutas, restringiste escritura a `sandbox/output/`, limitaste extensiones, controlaste tamaño de archivos y registraste eventos de seguridad. Este patrón permite conectar modelos y clientes compatibles con sistemas externos sin perder trazabilidad ni control operativo.
