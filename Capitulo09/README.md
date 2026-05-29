# Desarrollar un Servidor MCP en Python que permita a un agente interactuar de forma segura con un sistema de archivos.

## 1. Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 50 minutos |
| **Complejidad** | Alta (Hard) |
| **Nivel Bloom** | Crear |
| **Módulo** | 9 — Model Context Protocol (MCP) |
| **Costo estimado en APIs** | < $0.10 USD (llamadas mínimas a Claude o GPT-4o durante las pruebas) |

---

## 2. Descripción General

En este laboratorio implementarás un servidor **Model Context Protocol (MCP)** completo en Python que expone herramientas de sistema de archivos con controles de seguridad explícitos. Siguiendo la arquitectura cliente-servidor definida por Anthropic, el servidor correrá en modo **stdio** y aplicará el principio de mínimo privilegio: todas las operaciones quedan confinadas a un directorio sandbox y cualquier intento de path traversal es rechazado y registrado. Al finalizar, contarás con un servidor reutilizable que cualquier host compatible con MCP (Claude Desktop, LangGraph, aplicación FastAPI) puede consumir sin modificaciones.

---

## 3. Objetivos de Aprendizaje

- [ ] Implementar un servidor MCP en Python usando el SDK oficial que exponga las tres primitivas del protocolo: **Tools**, **Resources** y **Prompts**.
- [ ] Aplicar controles de seguridad de mínimo privilegio: sandbox de directorio, lista blanca de extensiones, rechazo de path traversal y registro de accesos en `mcp_access.log`.
- [ ] Diseñar y exponer cinco herramientas MCP con validaciones robustas: `list_files`, `read_file`, `write_file`, `search_in_files` y `get_file_metadata`.
- [ ] Construir un cliente de prueba (`test_mcp_client.py`) que conecte al servidor via stdio y ejecute una secuencia automatizada de operaciones.
- [ ] Verificar el comportamiento de seguridad del servidor mediante pruebas de intentos de path traversal y operaciones no permitidas.

---

## 4. Prerrequisitos

### Conocimiento previo
- Haber completado **Lab 08-00-01** (conceptos de agentes y herramientas con LangChain/OpenAI).
- Comprensión del modelo cliente-servidor y comunicación por stdin/stdout.
- Manejo de `pathlib.Path` y `os.path` para rutas seguras en Python.
- Familiaridad con `asyncio` y funciones `async/await` en Python 3.11.
- Comprensión básica de JSON-RPC 2.0 (cubierta en la Lección 9.1).

### Acceso y credenciales
- Cuenta activa en **Anthropic** (Claude API) **o** **OpenAI** (GPT-4o) con API key configurada — solo se necesita una.
- Límite de gasto mensual configurado en la consola del proveedor (máximo $5 USD para todo el curso).
- Acceso a terminal con permisos para instalar paquetes Python (`pip install`).

> ⚠️ **Nota sobre el SDK de MCP:** El SDK oficial de Python para MCP (`mcp`) es relativamente nuevo y su API pública puede cambiar entre versiones menores. Este lab fue validado con `mcp>=1.0.0`. El instructor debe verificar la versión disponible en PyPI la semana antes del lab (`pip index versions mcp`) y ajustar las instrucciones si la API pública cambió significativamente. Como alternativa de respaldo, se incluye una nota al final del lab para implementar el protocolo manualmente sobre stdio con JSON.

---

## 5. Entorno del Laboratorio

### Requisitos de hardware

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 4 GB libres | 8 GB libres |
| Almacenamiento | 500 MB libres | 1 GB libres |
| CPU | 2 núcleos | 4 núcleos |
| Red | 10 Mbps | 20 Mbps |

### Requisitos de software

| Componente | Versión requerida |
|---|---|
| Python | 3.11.x |
| pip | 23.x o superior |
| mcp (SDK oficial) | ≥ 1.0.0 |
| aiofiles | ≥ 23.2.0 |
| anthropic SDK | 0.28.x (opcional, para cliente de prueba) |
| openai SDK | 1.35.x (alternativa para cliente de prueba) |
| python-dotenv | ≥ 1.0.0 |

### Configuración del entorno

**Paso A — Crear y activar el entorno virtual:**

```bash
# Desde el directorio raíz del curso
python -m venv venv_lab09
# Windows
venv_lab09\Scripts\activate
# macOS / Linux
source venv_lab09/bin/activate
```

**Paso B — Instalar dependencias:**

```bash
pip install "mcp>=1.0.0" aiofiles python-dotenv anthropic openai
```

**Paso C — Verificar la instalación del SDK de MCP:**

```bash
python -c "import mcp; print(mcp.__version__)"
```

> Si el comando anterior falla, consulta la sección de Troubleshooting al final del lab.

**Paso D — Crear la estructura de directorios del proyecto:**

```bash
mkdir -p lab09/sandbox/output
mkdir -p lab09/sandbox/docs
mkdir -p lab09/sandbox/data
touch lab09/sandbox/docs/readme.md
touch lab09/sandbox/data/sample.csv
touch lab09/.env
touch lab09/.gitignore
```

**Paso E — Configurar `.gitignore` y `.env`:**

```bash
# Contenido de lab09/.gitignore
cat > lab09/.gitignore << 'EOF'
.env
__pycache__/
*.pyc
mcp_access.log
venv_lab09/
EOF
```

```bash
# lab09/.env — reemplaza con tu API key real
cat > lab09/.env << 'EOF'
# Usa solo UNA de las siguientes según tu proveedor
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxx
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxx

# Configuración del sandbox MCP
MCP_SANDBOX_DIR=./sandbox
MCP_MAX_FILE_SIZE_BYTES=1048576
MCP_LOG_FILE=mcp_access.log
EOF
```

> 🔒 **Seguridad:** Verifica que `.env` esté en `.gitignore` **antes** de ejecutar cualquier `git add`. El instructor revisará esto al inicio de la sesión.

**Paso F — Crear archivos de prueba en el sandbox:**

```bash
# Archivo de texto de prueba
cat > lab09/sandbox/docs/readme.md << 'EOF'
# Documentación del Proyecto

Este es un archivo de documentación de prueba para el Lab 09.
Contiene información sobre el sistema de archivos sandbox.

## Secciones
- Introducción
- Configuración
- Uso
EOF

# CSV de prueba
cat > lab09/sandbox/data/sample.csv << 'EOF'
id,nombre,valor
1,alpha,100
2,beta,200
3,gamma,300
EOF

# Script Python de prueba
cat > lab09/sandbox/docs/config.py << 'EOF'
# Configuración de ejemplo
DEBUG = True
MAX_RETRIES = 3
TIMEOUT = 30
EOF
```

---

## 6. Pasos del Laboratorio

### Paso 1 — Implementar el servidor MCP con herramientas de sistema de archivos

**Objetivo:** Crear `filesystem_mcp_server.py` con las cinco herramientas MCP, controles de seguridad y registro de accesos.

**Instrucciones:**

1. Navega al directorio `lab09/`:

```bash
cd lab09
```

2. Crea el archivo `filesystem_mcp_server.py` con el siguiente contenido completo:

```python
# filesystem_mcp_server.py
# Servidor MCP para acceso seguro al sistema de archivos
# Lab 09-00-01 — Model Context Protocol

import asyncio
import logging
import mimetypes
import os
import re
import stat
from datetime import datetime
from pathlib import Path
from typing import Any

import aiofiles
from dotenv import load_dotenv
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import (
    EmbeddedResource,
    GetPromptResult,
    ListResourcesResult,
    Prompt,
    PromptArgument,
    PromptMessage,
    ReadResourceResult,
    Resource,
    TextContent,
    TextResourceContents,
    Tool,
)

# ─────────────────────────────────────────────
# Configuración inicial
# ─────────────────────────────────────────────
load_dotenv()

# Directorio sandbox: todas las operaciones deben ocurrir dentro de él
_RAW_SANDBOX = os.getenv("MCP_SANDBOX_DIR", "./sandbox")
SANDBOX_DIR = Path(_RAW_SANDBOX).resolve()
MAX_FILE_SIZE = int(os.getenv("MCP_MAX_FILE_SIZE_BYTES", "1048576"))  # 1 MB
LOG_FILE = os.getenv("MCP_LOG_FILE", "mcp_access.log")

# Extensiones de archivo permitidas para lectura
ALLOWED_EXTENSIONS = {".txt", ".md", ".py", ".json", ".csv"}

# Configuración del logger de accesos
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.FileHandler(LOG_FILE, encoding="utf-8"),
        logging.StreamHandler(),  # También a stderr para depuración
    ],
)
access_log = logging.getLogger("mcp_access")

# ─────────────────────────────────────────────
# Utilidades de seguridad
# ─────────────────────────────────────────────

def _validate_path(raw_path: str, must_exist: bool = True) -> Path:
    """
    Resuelve y valida que la ruta esté dentro del SANDBOX_DIR.
    Lanza ValueError con mensaje descriptivo si la validación falla.
    Registra todos los intentos en el log de accesos.
    """
    try:
        # Construir la ruta candidata
        candidate = (SANDBOX_DIR / raw_path).resolve()
    except Exception as exc:
        access_log.warning("PATH_INVALID | raw='%s' | error=%s", raw_path, exc)
        raise ValueError(f"Ruta inválida: '{raw_path}'. Detalle: {exc}") from exc

    # Detectar path traversal: la ruta resuelta debe comenzar con SANDBOX_DIR
    try:
        candidate.relative_to(SANDBOX_DIR)
    except ValueError:
        access_log.warning(
            "PATH_TRAVERSAL_ATTEMPT | raw='%s' | resolved='%s' | sandbox='%s'",
            raw_path,
            candidate,
            SANDBOX_DIR,
        )
        raise ValueError(
            f"Acceso denegado: la ruta '{raw_path}' intenta salir del sandbox "
            f"'{SANDBOX_DIR}'. Operación rechazada por política de seguridad."
        )

    if must_exist and not candidate.exists():
        access_log.info("PATH_NOT_FOUND | resolved='%s'", candidate)
        raise FileNotFoundError(f"El archivo o directorio no existe: '{candidate}'")

    access_log.info("PATH_VALIDATED | resolved='%s'", candidate)
    return candidate


def _validate_extension(filepath: Path) -> None:
    """Verifica que la extensión del archivo esté en la lista blanca."""
    ext = filepath.suffix.lower()
    if ext not in ALLOWED_EXTENSIONS:
        access_log.warning(
            "EXTENSION_DENIED | file='%s' | ext='%s' | allowed=%s",
            filepath,
            ext,
            ALLOWED_EXTENSIONS,
        )
        raise ValueError(
            f"Extensión '{ext}' no permitida. Extensiones válidas: "
            f"{', '.join(sorted(ALLOWED_EXTENSIONS))}"
        )


# ─────────────────────────────────────────────
# Creación del servidor MCP
# ─────────────────────────────────────────────
app = Server("filesystem-mcp-server")


# ─────────────────────────────────────────────
# TOOLS — Herramientas del servidor
# ─────────────────────────────────────────────

@app.list_tools()
async def list_tools() -> list[Tool]:
    """Declara las cinco herramientas disponibles en este servidor."""
    return [
        Tool(
            name="list_files",
            description=(
                "Lista todos los archivos y subdirectorios dentro de un directorio "
                "del sandbox. Solo acepta rutas dentro del directorio sandbox configurado."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "directory": {
                        "type": "string",
                        "description": (
                            "Ruta del directorio a listar, relativa al sandbox. "
                            "Usa '.' para el directorio raíz del sandbox."
                        ),
                    }
                },
                "required": ["directory"],
            },
        ),
        Tool(
            name="read_file",
            description=(
                "Lee el contenido de un archivo de texto. Solo permite extensiones: "
                ".txt, .md, .py, .json, .csv. Tamaño máximo: 1 MB."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "filepath": {
                        "type": "string",
                        "description": "Ruta del archivo relativa al sandbox.",
                    }
                },
                "required": ["filepath"],
            },
        ),
        Tool(
            name="write_file",
            description=(
                "Escribe contenido en un archivo dentro del directorio sandbox/output/. "
                "Si el archivo ya existe, se sobreescribe (el agente debe confirmar antes)."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "filepath": {
                        "type": "string",
                        "description": (
                            "Ruta del archivo destino, relativa al sandbox. "
                            "Debe estar dentro del subdirectorio 'output/'."
                        ),
                    },
                    "content": {
                        "type": "string",
                        "description": "Contenido a escribir en el archivo.",
                    },
                    "overwrite_confirmed": {
                        "type": "boolean",
                        "description": (
                            "Debe ser true si el archivo ya existe y el usuario "
                            "confirmó la sobreescritura. Por defecto: false."
                        ),
                        "default": False,
                    },
                },
                "required": ["filepath", "content"],
            },
        ),
        Tool(
            name="search_in_files",
            description=(
                "Busca un patrón regex en todos los archivos permitidos de un directorio. "
                "Retorna lista de coincidencias con nombre de archivo y número de línea."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "pattern": {
                        "type": "string",
                        "description": "Expresión regular a buscar.",
                    },
                    "directory": {
                        "type": "string",
                        "description": (
                            "Directorio donde buscar, relativo al sandbox. "
                            "Usa '.' para buscar en todo el sandbox."
                        ),
                    },
                },
                "required": ["pattern", "directory"],
            },
        ),
        Tool(
            name="get_file_metadata",
            description=(
                "Retorna metadata de un archivo: tamaño en bytes, fecha de modificación, "
                "tipo MIME y permisos básicos."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "filepath": {
                        "type": "string",
                        "description": "Ruta del archivo relativa al sandbox.",
                    }
                },
                "required": ["filepath"],
            },
        ),
    ]


@app.call_tool()
async def call_tool(name: str, arguments: dict[str, Any]) -> list[TextContent]:
    """Dispatcher principal: enruta la llamada a la herramienta correcta."""
    access_log.info("TOOL_CALL | tool='%s' | args=%s", name, arguments)

    try:
        if name == "list_files":
            return await _tool_list_files(arguments)
        elif name == "read_file":
            return await _tool_read_file(arguments)
        elif name == "write_file":
            return await _tool_write_file(arguments)
        elif name == "search_in_files":
            return await _tool_search_in_files(arguments)
        elif name == "get_file_metadata":
            return await _tool_get_file_metadata(arguments)
        else:
            raise ValueError(f"Herramienta desconocida: '{name}'")
    except (ValueError, FileNotFoundError, PermissionError) as exc:
        access_log.error("TOOL_ERROR | tool='%s' | error=%s", name, exc)
        return [TextContent(type="text", text=f"❌ Error: {exc}")]
    except Exception as exc:
        access_log.error("TOOL_UNEXPECTED_ERROR | tool='%s' | error=%s", name, exc)
        return [TextContent(type="text", text=f"❌ Error inesperado: {exc}")]


# ─── Implementaciones de herramientas ───────

async def _tool_list_files(args: dict) -> list[TextContent]:
    """list_files: lista archivos en un directorio del sandbox."""
    directory = args.get("directory", ".")
    dir_path = _validate_path(directory)

    if not dir_path.is_dir():
        raise ValueError(f"'{directory}' no es un directorio.")

    entries = []
    for entry in sorted(dir_path.iterdir()):
        rel = entry.relative_to(SANDBOX_DIR)
        tipo = "📁" if entry.is_dir() else "📄"
        entries.append(f"{tipo} {rel}")

    if not entries:
        result = f"El directorio '{directory}' está vacío."
    else:
        result = f"Contenido de '{directory}' ({len(entries)} entradas):\n" + "\n".join(entries)

    access_log.info("LIST_FILES | dir='%s' | count=%d", directory, len(entries))
    return [TextContent(type="text", text=result)]


async def _tool_read_file(args: dict) -> list[TextContent]:
    """read_file: lee el contenido de un archivo con validaciones."""
    filepath = args.get("filepath", "")
    file_path = _validate_path(filepath)

    if not file_path.is_file():
        raise ValueError(f"'{filepath}' no es un archivo regular.")

    _validate_extension(file_path)

    file_size = file_path.stat().st_size
    if file_size > MAX_FILE_SIZE:
        raise ValueError(
            f"Archivo demasiado grande: {file_size} bytes "
            f"(máximo permitido: {MAX_FILE_SIZE} bytes / 1 MB)."
        )

    async with aiofiles.open(file_path, "r", encoding="utf-8", errors="replace") as f:
        content = await f.read()

    access_log.info("READ_FILE | file='%s' | size=%d bytes", filepath, file_size)
    return [
        TextContent(
            type="text",
            text=f"📄 Contenido de '{filepath}' ({file_size} bytes):\n\n{content}",
        )
    ]


async def _tool_write_file(args: dict) -> list[TextContent]:
    """write_file: escribe un archivo solo en sandbox/output/."""
    filepath = args.get("filepath", "")
    content = args.get("content", "")
    overwrite_confirmed = args.get("overwrite_confirmed", False)

    # La ruta destino debe estar dentro de output/
    if not filepath.startswith("output/") and not filepath.startswith("output\\"):
        raise ValueError(
            f"Solo se permite escribir dentro del directorio 'output/'. "
            f"Ruta recibida: '{filepath}'"
        )

    file_path = _validate_path(filepath, must_exist=False)

    # Si el archivo ya existe y no se confirmó sobreescritura
    if file_path.exists() and not overwrite_confirmed:
        access_log.warning(
            "WRITE_BLOCKED_OVERWRITE | file='%s'", filepath
        )
        return [
            TextContent(
                type="text",
                text=(
                    f"⚠️ El archivo '{filepath}' ya existe. "
                    f"Para sobreescribirlo, llama a write_file nuevamente con "
                    f"'overwrite_confirmed': true."
                ),
            )
        ]

    # Crear directorios intermedios si no existen
    file_path.parent.mkdir(parents=True, exist_ok=True)

    async with aiofiles.open(file_path, "w", encoding="utf-8") as f:
        await f.write(content)

    access_log.info(
        "WRITE_FILE | file='%s' | size=%d bytes | overwrite=%s",
        filepath,
        len(content.encode("utf-8")),
        overwrite_confirmed,
    )
    return [
        TextContent(
            type="text",
            text=f"✅ Archivo '{filepath}' escrito correctamente ({len(content)} caracteres).",
        )
    ]


async def _tool_search_in_files(args: dict) -> list[TextContent]:
    """search_in_files: busca un patrón regex en archivos del sandbox."""
    pattern = args.get("pattern", "")
    directory = args.get("directory", ".")

    # Validar que el patrón regex sea válido
    try:
        compiled = re.compile(pattern, re.IGNORECASE)
    except re.error as exc:
        raise ValueError(f"Patrón regex inválido '{pattern}': {exc}") from exc

    dir_path = _validate_path(directory)

    if not dir_path.is_dir():
        raise ValueError(f"'{directory}' no es un directorio.")

    matches = []
    files_scanned = 0

    # Recorrer recursivamente todos los archivos con extensiones permitidas
    for file_path in sorted(dir_path.rglob("*")):
        if not file_path.is_file():
            continue
        if file_path.suffix.lower() not in ALLOWED_EXTENSIONS:
            continue
        if file_path.stat().st_size > MAX_FILE_SIZE:
            continue

        files_scanned += 1
        try:
            async with aiofiles.open(
                file_path, "r", encoding="utf-8", errors="replace"
            ) as f:
                lines = await f.readlines()

            for line_num, line in enumerate(lines, start=1):
                if compiled.search(line):
                    rel_path = file_path.relative_to(SANDBOX_DIR)
                    matches.append(
                        {
                            "file": str(rel_path),
                            "line": line_num,
                            "content": line.rstrip(),
                        }
                    )
        except Exception as exc:
            access_log.warning(
                "SEARCH_FILE_ERROR | file='%s' | error=%s", file_path, exc
            )

    access_log.info(
        "SEARCH_IN_FILES | pattern='%s' | dir='%s' | scanned=%d | matches=%d",
        pattern,
        directory,
        files_scanned,
        len(matches),
    )

    if not matches:
        return [
            TextContent(
                type="text",
                text=(
                    f"🔍 Sin resultados para el patrón '{pattern}' "
                    f"en '{directory}' ({files_scanned} archivos analizados)."
                ),
            )
        ]

    lines_out = [
        f"🔍 {len(matches)} coincidencias para '{pattern}' "
        f"en {files_scanned} archivos:\n"
    ]
    for m in matches[:50]:  # Límite de 50 resultados para evitar respuestas masivas
        lines_out.append(f"  📄 {m['file']}:{m['line']} → {m['content']}")

    if len(matches) > 50:
        lines_out.append(f"  ... y {len(matches) - 50} coincidencias más.")

    return [TextContent(type="text", text="\n".join(lines_out))]


async def _tool_get_file_metadata(args: dict) -> list[TextContent]:
    """get_file_metadata: retorna metadata de un archivo."""
    filepath = args.get("filepath", "")
    file_path = _validate_path(filepath)

    if not file_path.is_file():
        raise ValueError(f"'{filepath}' no es un archivo regular.")

    stat_info = file_path.stat()
    mime_type, _ = mimetypes.guess_type(str(file_path))
    mime_type = mime_type or "application/octet-stream"

    mod_time = datetime.fromtimestamp(stat_info.st_mtime).isoformat()
    create_time = datetime.fromtimestamp(stat_info.st_ctime).isoformat()

    # Permisos en formato octal legible
    perms = oct(stat.S_IMODE(stat_info.st_mode))

    metadata = {
        "filepath": filepath,
        "size_bytes": stat_info.st_size,
        "size_human": _human_size(stat_info.st_size),
        "mime_type": mime_type,
        "modified_at": mod_time,
        "created_at": create_time,
        "permissions": perms,
        "extension": file_path.suffix.lower(),
        "is_readable": file_path.suffix.lower() in ALLOWED_EXTENSIONS,
    }

    import json
    result = f"📊 Metadata de '{filepath}':\n\n{json.dumps(metadata, indent=2, ensure_ascii=False)}"
    access_log.info("GET_METADATA | file='%s' | size=%d", filepath, stat_info.st_size)
    return [TextContent(type="text", text=result)]


def _human_size(size_bytes: int) -> str:
    """Convierte bytes a formato legible (KB, MB, etc.)."""
    for unit in ("B", "KB", "MB", "GB"):
        if size_bytes < 1024:
            return f"{size_bytes:.1f} {unit}"
        size_bytes //= 1024
    return f"{size_bytes:.1f} TB"


# ─────────────────────────────────────────────
# RESOURCES — Árbol de directorios del sandbox
# ─────────────────────────────────────────────

@app.list_resources()
async def list_resources() -> list[Resource]:
    """Expone el árbol del sandbox como recurso estático."""
    return [
        Resource(
            uri="file://sandbox/tree",
            name="Árbol del Sandbox",
            description=(
                "Vista completa del árbol de directorios y archivos "
                "disponibles en el sandbox del servidor MCP."
            ),
            mimeType="text/plain",
        )
    ]


@app.read_resource()
async def read_resource(uri: str) -> str:
    """Retorna el contenido del recurso solicitado."""
    if uri == "file://sandbox/tree":
        tree_lines = [f"📁 SANDBOX: {SANDBOX_DIR}\n"]
        _build_tree(SANDBOX_DIR, tree_lines, prefix="")
        access_log.info("READ_RESOURCE | uri='%s'", uri)
        return "\n".join(tree_lines)
    raise ValueError(f"Recurso desconocido: '{uri}'")


def _build_tree(path: Path, lines: list, prefix: str) -> None:
    """Construye representación en árbol del directorio."""
    try:
        entries = sorted(path.iterdir())
    except PermissionError:
        return

    for i, entry in enumerate(entries):
        connector = "└── " if i == len(entries) - 1 else "├── "
        icon = "📁" if entry.is_dir() else "📄"
        lines.append(f"{prefix}{connector}{icon} {entry.name}")
        if entry.is_dir():
            extension = "    " if i == len(entries) - 1 else "│   "
            _build_tree(entry, lines, prefix + extension)


# ─────────────────────────────────────────────
# PROMPTS — Plantillas de instrucciones
# ─────────────────────────────────────────────

@app.list_prompts()
async def list_prompts() -> list[Prompt]:
    """Declara las plantillas de prompt disponibles."""
    return [
        Prompt(
            name="analizar_directorio",
            description=(
                "Genera un análisis completo del contenido de un directorio del sandbox, "
                "incluyendo estructura, tipos de archivo y sugerencias de organización."
            ),
            arguments=[
                PromptArgument(
                    name="directorio",
                    description="Directorio a analizar (relativo al sandbox)",
                    required=True,
                )
            ],
        ),
        Prompt(
            name="buscar_y_reportar",
            description=(
                "Busca un patrón en los archivos del sandbox y genera un reporte "
                "estructurado con los resultados encontrados."
            ),
            arguments=[
                PromptArgument(
                    name="patron",
                    description="Patrón de búsqueda (texto o regex)",
                    required=True,
                ),
                PromptArgument(
                    name="directorio",
                    description="Directorio donde buscar",
                    required=False,
                ),
            ],
        ),
    ]


@app.get_prompt()
async def get_prompt(name: str, arguments: dict[str, str] | None) -> GetPromptResult:
    """Retorna el prompt renderizado con los argumentos proporcionados."""
    args = arguments or {}

    if name == "analizar_directorio":
        directorio = args.get("directorio", ".")
        return GetPromptResult(
            description=f"Análisis del directorio '{directorio}'",
            messages=[
                PromptMessage(
                    role="user",
                    content=TextContent(
                        type="text",
                        text=(
                            f"Por favor, analiza el directorio '{directorio}' usando las "
                            f"herramientas disponibles. Primero lista su contenido con "
                            f"list_files, luego examina los archivos relevantes con read_file "
                            f"y get_file_metadata. Finalmente, proporciona un resumen "
                            f"estructurado con: (1) tipos de archivos encontrados, "
                            f"(2) tamaño total aproximado, (3) sugerencias de organización."
                        ),
                    ),
                )
            ],
        )

    elif name == "buscar_y_reportar":
        patron = args.get("patron", "")
        directorio = args.get("directorio", ".")
        return GetPromptResult(
            description=f"Búsqueda de '{patron}' en '{directorio}'",
            messages=[
                PromptMessage(
                    role="user",
                    content=TextContent(
                        type="text",
                        text=(
                            f"Busca el patrón '{patron}' en el directorio '{directorio}' "
                            f"usando la herramienta search_in_files. Luego genera un reporte "
                            f"con: (1) número total de coincidencias, (2) archivos afectados, "
                            f"(3) contexto de cada coincidencia, (4) conclusiones."
                        ),
                    ),
                )
            ],
        )

    raise ValueError(f"Prompt desconocido: '{name}'")


# ─────────────────────────────────────────────
# Punto de entrada principal
# ─────────────────────────────────────────────

async def main():
    """Inicia el servidor MCP en modo stdio."""
    # Verificar que el sandbox existe
    if not SANDBOX_DIR.exists():
        access_log.error("SANDBOX_NOT_FOUND | path='%s'", SANDBOX_DIR)
        raise RuntimeError(
            f"El directorio sandbox '{SANDBOX_DIR}' no existe. "
            f"Créalo antes de iniciar el servidor."
        )

    # Verificar que el directorio output existe
    output_dir = SANDBOX_DIR / "output"
    output_dir.mkdir(exist_ok=True)

    access_log.info(
        "SERVER_START | sandbox='%s' | max_file_size=%d bytes",
        SANDBOX_DIR,
        MAX_FILE_SIZE,
    )

    # Iniciar el servidor con transporte stdio (JSON-RPC 2.0 sobre stdin/stdout)
    async with stdio_server() as (read_stream, write_stream):
        await app.run(
            read_stream,
            write_stream,
            app.create_initialization_options(),
        )


if __name__ == "__main__":
    asyncio.run(main())
```

**Salida esperada al iniciar el servidor (en stderr / log):**
```
2024-XX-XX XX:XX:XX [INFO] SERVER_START | sandbox='/ruta/absoluta/lab09/sandbox' | max_file_size=1048576 bytes
```

**Verificación:**
```bash
# Verificar que el archivo no tiene errores de sintaxis
python -m py_compile filesystem_mcp_server.py && echo "✅ Sintaxis OK"
```

---

### Paso 2 — Implementar el cliente de prueba

**Objetivo:** Crear `test_mcp_client.py` que conecte al servidor via stdio y ejecute una secuencia automatizada de operaciones, incluyendo pruebas de seguridad.

**Instrucciones:**

1. Crea el archivo `test_mcp_client.py`:

```python
# test_mcp_client.py
# Cliente de prueba para el servidor MCP de sistema de archivos
# Lab 09-00-01

import asyncio
import json
import sys
from pathlib import Path

from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client


# Ruta al servidor MCP (relativa al directorio actual)
SERVER_SCRIPT = Path(__file__).parent / "filesystem_mcp_server.py"

# Colores para la salida de terminal
GREEN = "\033[92m"
RED = "\033[91m"
YELLOW = "\033[93m"
BLUE = "\033[94m"
RESET = "\033[0m"
BOLD = "\033[1m"


def print_header(title: str) -> None:
    print(f"\n{BOLD}{BLUE}{'='*60}{RESET}")
    print(f"{BOLD}{BLUE}  {title}{RESET}")
    print(f"{BOLD}{BLUE}{'='*60}{RESET}")


def print_result(label: str, content: str, success: bool = True) -> None:
    icon = f"{GREEN}✅{RESET}" if success else f"{RED}❌{RESET}"
    print(f"\n{icon} {BOLD}{label}{RESET}")
    print(f"   {content[:300]}{'...' if len(content) > 300 else ''}")


async def run_test_suite():
    """Ejecuta la suite completa de pruebas contra el servidor MCP."""

    print_header("Suite de Pruebas — Servidor MCP Filesystem")
    print(f"   Servidor: {SERVER_SCRIPT}")
    print(f"   Python:   {sys.version.split()[0]}")

    # Parámetros para lanzar el servidor como subproceso via stdio
    server_params = StdioServerParameters(
        command=sys.executable,  # Usa el mismo Python del entorno virtual
        args=[str(SERVER_SCRIPT)],
        env=None,  # Hereda el entorno (incluye .env cargado)
    )

    results = {"passed": 0, "failed": 0}

    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:

            # ── Inicialización ──────────────────────────────────────
            print_header("Fase 1: Inicialización del protocolo MCP")
            await session.initialize()
            print_result("Handshake MCP completado", "Sesión inicializada correctamente")
            results["passed"] += 1

            # ── Listar herramientas disponibles ─────────────────────
            print_header("Fase 2: Descubrimiento de capacidades")

            tools_response = await session.list_tools()
            tool_names = [t.name for t in tools_response.tools]
            expected_tools = {
                "list_files", "read_file", "write_file",
                "search_in_files", "get_file_metadata"
            }
            if expected_tools.issubset(set(tool_names)):
                print_result(
                    "Herramientas registradas",
                    f"Encontradas: {', '.join(tool_names)}",
                )
                results["passed"] += 1
            else:
                missing = expected_tools - set(tool_names)
                print_result(
                    f"Herramientas faltantes: {missing}",
                    "Verificar implementación del servidor",
                    success=False,
                )
                results["failed"] += 1

            # Listar recursos
            resources_response = await session.list_resources()
            resource_uris = [r.uri for r in resources_response.resources]
            print_result(
                "Recursos registrados",
                f"URIs: {', '.join(resource_uris) or 'ninguno'}",
            )
            results["passed"] += 1

            # Listar prompts
            prompts_response = await session.list_prompts()
            prompt_names = [p.name for p in prompts_response.prompts]
            print_result(
                "Prompts registrados",
                f"Nombres: {', '.join(prompt_names) or 'ninguno'}",
            )
            results["passed"] += 1

            # ── Pruebas funcionales ─────────────────────────────────
            print_header("Fase 3: Pruebas funcionales de herramientas")

            # Test 1: list_files en raíz del sandbox
            resp = await session.call_tool("list_files", {"directory": "."})
            content = resp.content[0].text if resp.content else ""
            if "docs" in content or "data" in content:
                print_result("list_files('.')", content)
                results["passed"] += 1
            else:
                print_result("list_files('.') — sin resultados esperados", content, False)
                results["failed"] += 1

            # Test 2: list_files en subdirectorio
            resp = await session.call_tool("list_files", {"directory": "docs"})
            content = resp.content[0].text if resp.content else ""
            print_result("list_files('docs')", content)
            results["passed"] += 1

            # Test 3: read_file de un archivo .md
            resp = await session.call_tool(
                "read_file", {"filepath": "docs/readme.md"}
            )
            content = resp.content[0].text if resp.content else ""
            if "Documentación" in content or "Error" not in content:
                print_result("read_file('docs/readme.md')", content)
                results["passed"] += 1
            else:
                print_result("read_file — error inesperado", content, False)
                results["failed"] += 1

            # Test 4: write_file en output/
            resp = await session.call_tool(
                "write_file",
                {
                    "filepath": "output/test_output.txt",
                    "content": "Archivo de prueba generado por test_mcp_client.py\nFecha: 2024",
                    "overwrite_confirmed": False,
                },
            )
            content = resp.content[0].text if resp.content else ""
            print_result("write_file('output/test_output.txt')", content)
            results["passed"] += 1

            # Test 5: write_file con sobreescritura (debe pedir confirmación)
            resp = await session.call_tool(
                "write_file",
                {
                    "filepath": "output/test_output.txt",
                    "content": "Contenido actualizado",
                    "overwrite_confirmed": False,
                },
            )
            content = resp.content[0].text if resp.content else ""
            if "ya existe" in content or "overwrite_confirmed" in content:
                print_result(
                    "write_file — protección de sobreescritura activada ✅",
                    content,
                )
                results["passed"] += 1
            else:
                print_result(
                    "write_file — protección de sobreescritura NO activada",
                    content,
                    False,
                )
                results["failed"] += 1

            # Test 6: search_in_files
            resp = await session.call_tool(
                "search_in_files",
                {"pattern": "DEBUG|config|id", "directory": "."},
            )
            content = resp.content[0].text if resp.content else ""
            print_result("search_in_files(pattern='DEBUG|config|id')", content)
            results["passed"] += 1

            # Test 7: get_file_metadata
            resp = await session.call_tool(
                "get_file_metadata", {"filepath": "data/sample.csv"}
            )
            content = resp.content[0].text if resp.content else ""
            if "size_bytes" in content or "mime_type" in content:
                print_result("get_file_metadata('data/sample.csv')", content)
                results["passed"] += 1
            else:
                print_result("get_file_metadata — respuesta inesperada", content, False)
                results["failed"] += 1

            # ── Pruebas de seguridad ────────────────────────────────
            print_header("Fase 4: Pruebas de seguridad (ataques de path traversal)")

            security_tests = [
                ("../../../etc/passwd", "Traversal hacia /etc/passwd"),
                ("../../.env", "Traversal hacia .env"),
                ("/etc/hosts", "Ruta absoluta fuera del sandbox"),
                ("docs/../../../../tmp/evil.txt", "Traversal profundo"),
            ]

            for malicious_path, description in security_tests:
                resp = await session.call_tool(
                    "read_file", {"filepath": malicious_path}
                )
                content = resp.content[0].text if resp.content else ""
                if "denegado" in content.lower() or "error" in content.lower() or "❌" in content:
                    print_result(
                        f"BLOQUEADO: {description}",
                        f"Respuesta: {content[:100]}",
                    )
                    results["passed"] += 1
                else:
                    print_result(
                        f"⚠️  NO BLOQUEADO: {description} — FALLA DE SEGURIDAD",
                        content[:100],
                        False,
                    )
                    results["failed"] += 1

            # Test: write fuera de output/
            resp = await session.call_tool(
                "write_file",
                {
                    "filepath": "docs/malicious.md",
                    "content": "Escritura no autorizada",
                    "overwrite_confirmed": True,
                },
            )
            content = resp.content[0].text if resp.content else ""
            if "output/" in content or "error" in content.lower() or "❌" in content:
                print_result(
                    "BLOQUEADO: Escritura fuera de output/",
                    f"Respuesta: {content[:100]}",
                )
                results["passed"] += 1
            else:
                print_result(
                    "⚠️  NO BLOQUEADO: Escritura fuera de output/",
                    content[:100],
                    False,
                )
                results["failed"] += 1

            # ── Leer recurso MCP ────────────────────────────────────
            print_header("Fase 5: Lectura de Resource MCP")

            resource_resp = await session.read_resource("file://sandbox/tree")
            tree_content = ""
            if resource_resp.contents:
                first = resource_resp.contents[0]
                tree_content = getattr(first, "text", str(first))
            print_result("Árbol del sandbox (Resource MCP)", tree_content)
            results["passed"] += 1

            # ── Resumen final ───────────────────────────────────────
            print_header("Resumen de Pruebas")
            total = results["passed"] + results["failed"]
            pct = (results["passed"] / total * 100) if total > 0 else 0
            print(f"\n  {GREEN}✅ Pasadas: {results['passed']}{RESET}")
            print(f"  {RED}❌ Fallidas: {results['failed']}{RESET}")
            print(f"  📊 Porcentaje: {pct:.1f}%")

            if results["failed"] == 0:
                print(f"\n  {GREEN}{BOLD}🎉 ¡Todas las pruebas pasaron correctamente!{RESET}")
            else:
                print(
                    f"\n  {YELLOW}{BOLD}⚠️  Revisa las pruebas fallidas antes de continuar.{RESET}"
                )

            return results["failed"] == 0


if __name__ == "__main__":
    success = asyncio.run(run_test_suite())
    sys.exit(0 if success else 1)
```

2. Verifica la sintaxis del cliente:

```bash
python -m py_compile test_mcp_client.py && echo "✅ Sintaxis OK"
```

**Salida esperada:**
```
✅ Sintaxis OK
```

---

### Paso 3 — Ejecutar la suite de pruebas

**Objetivo:** Ejecutar `test_mcp_client.py` y verificar que todas las pruebas funcionales y de seguridad pasan correctamente.

**Instrucciones:**

1. Asegúrate de estar en el directorio `lab09/` con el entorno virtual activado:

```bash
cd lab09
source ../venv_lab09/bin/activate  # macOS/Linux
# o: ..\venv_lab09\Scripts\activate  # Windows
```

2. Ejecuta el cliente de prueba:

```bash
python test_mcp_client.py
```

**Salida esperada (extracto):**

```
============================================================
  Suite de Pruebas — Servidor MCP Filesystem
============================================================
   Servidor: /ruta/lab09/filesystem_mcp_server.py
   Python:   3.11.x

============================================================
  Fase 1: Inicialización del protocolo MCP
============================================================

✅ Handshake MCP completado
   Sesión inicializada correctamente

============================================================
  Fase 2: Descubrimiento de capacidades
============================================================

✅ Herramientas registradas
   Encontradas: list_files, read_file, write_file, search_in_files, get_file_metadata

✅ Recursos registrados
   URIs: file://sandbox/tree

✅ Prompts registrados
   Nombres: analizar_directorio, buscar_y_reportar

...

============================================================
  Fase 4: Pruebas de seguridad (ataques de path traversal)
============================================================

✅ BLOQUEADO: Traversal hacia /etc/passwd
   Respuesta: ❌ Error: Acceso denegado: la ruta '../../../etc/passwd' intenta salir del sandbox...

✅ BLOQUEADO: Traversal hacia .env
   Respuesta: ❌ Error: Acceso denegado...

...

============================================================
  Resumen de Pruebas
============================================================

  ✅ Pasadas: 17
  ❌ Fallidas: 0
  📊 Porcentaje: 100.0%

  🎉 ¡Todas las pruebas pasaron correctamente!
```

3. Verifica que el log de accesos se generó correctamente:

```bash
cat mcp_access.log
```

**Salida esperada (extracto del log):**
```
2024-XX-XX XX:XX:XX [INFO] SERVER_START | sandbox='/ruta/lab09/sandbox' | max_file_size=1048576 bytes
2024-XX-XX XX:XX:XX [INFO] TOOL_CALL | tool='list_files' | args={'directory': '.'}
2024-XX-XX XX:XX:XX [INFO] PATH_VALIDATED | resolved='/ruta/lab09/sandbox'
2024-XX-XX XX:XX:XX [INFO] LIST_FILES | dir='.' | count=3
2024-XX-XX XX:XX:XX [WARNING] PATH_TRAVERSAL_ATTEMPT | raw='../../../etc/passwd' | resolved='/etc/passwd' | sandbox='/ruta/lab09/sandbox'
```

**Verificación:**
```bash
# Contar entradas de seguridad en el log
grep -c "PATH_TRAVERSAL_ATTEMPT\|EXTENSION_DENIED\|WRITE_BLOCKED" mcp_access.log
# Debe retornar >= 4 (una por cada prueba de seguridad)
```

---

### Paso 4 — Integración con un agente Claude o GPT-4o (opcional avanzado)

**Objetivo:** Conectar el servidor MCP con un agente real usando la API de Anthropic o OpenAI para demostrar la integración end-to-end.

**Instrucciones:**

1. Crea el archivo `agent_with_mcp.py`:

```python
# agent_with_mcp.py
# Agente que usa el servidor MCP filesystem para tareas de gestión de archivos
# Lab 09-00-01 — Integración end-to-end

import asyncio
import sys
from pathlib import Path

from dotenv import load_dotenv
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

load_dotenv()

SERVER_SCRIPT = Path(__file__).parent / "filesystem_mcp_server.py"

# ── Intentar importar Anthropic o OpenAI ──────────────────
try:
    import anthropic
    USE_ANTHROPIC = True
except ImportError:
    USE_ANTHROPIC = False

try:
    import openai
    USE_OPENAI = True
except ImportError:
    USE_OPENAI = False

if not USE_ANTHROPIC and not USE_OPENAI:
    print("❌ Instala 'anthropic' o 'openai': pip install anthropic openai")
    sys.exit(1)


async def run_agent_task(task: str):
    """
    Ejecuta una tarea usando un agente LLM que tiene acceso
    a las herramientas del servidor MCP filesystem.
    """
    print(f"\n🤖 Tarea del agente: {task}\n")

    server_params = StdioServerParameters(
        command=sys.executable,
        args=[str(SERVER_SCRIPT)],
    )

    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # Obtener la lista de herramientas disponibles
            tools_response = await session.list_tools()

            # Convertir herramientas MCP al formato del proveedor LLM
            if USE_ANTHROPIC:
                await _run_with_anthropic(session, tools_response.tools, task)
            else:
                await _run_with_openai(session, tools_response.tools, task)


async def _run_with_anthropic(session: ClientSession, mcp_tools, task: str):
    """Ejecuta el agente usando la API de Anthropic Claude."""
    import os
    client = anthropic.Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

    # Convertir herramientas MCP a formato Anthropic
    anthropic_tools = [
        {
            "name": t.name,
            "description": t.description,
            "input_schema": t.inputSchema,
        }
        for t in mcp_tools
    ]

    messages = [{"role": "user", "content": task}]
    print(f"📡 Usando Claude (Anthropic) con {len(anthropic_tools)} herramientas MCP\n")

    # Bucle agentico: el modelo llama herramientas hasta completar la tarea
    max_iterations = 10
    for iteration in range(max_iterations):
        response = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=2048,
            tools=anthropic_tools,
            messages=messages,
        )

        # Procesar la respuesta
        for block in response.content:
            if hasattr(block, "text"):
                print(f"🤖 Claude: {block.text}")

        if response.stop_reason == "end_turn":
            print("\n✅ Agente completó la tarea.")
            break

        if response.stop_reason == "tool_use":
            # Procesar llamadas a herramientas
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"\n🔧 Llamando herramienta MCP: {block.name}")
                    print(f"   Argumentos: {block.input}")

                    # Invocar la herramienta en el servidor MCP
                    result = await session.call_tool(block.name, block.input)
                    result_text = result.content[0].text if result.content else "Sin resultado"
                    print(f"   Resultado: {result_text[:200]}")

                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result_text,
                    })

            # Actualizar el historial de mensajes
            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})
        else:
            print(f"⚠️  Stop reason inesperado: {response.stop_reason}")
            break


async def _run_with_openai(session: ClientSession, mcp_tools, task: str):
    """Ejecuta el agente usando la API de OpenAI GPT-4o."""
    import json
    import os
    client = openai.OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

    # Convertir herramientas MCP a formato OpenAI function calling
    openai_tools = [
        {
            "type": "function",
            "function": {
                "name": t.name,
                "description": t.description,
                "parameters": t.inputSchema,
            },
        }
        for t in mcp_tools
    ]

    messages = [
        {
            "role": "system",
            "content": (
                "Eres un asistente de gestión de archivos. Tienes acceso a un sandbox "
                "de sistema de archivos mediante herramientas MCP. Usa las herramientas "
                "para completar la tarea del usuario de forma segura y eficiente."
            ),
        },
        {"role": "user", "content": task},
    ]
    print(f"📡 Usando GPT-4o (OpenAI) con {len(openai_tools)} herramientas MCP\n")

    max_iterations = 10
    for iteration in range(max_iterations):
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=openai_tools,
            tool_choice="auto",
        )

        choice = response.choices[0]

        if choice.message.content:
            print(f"🤖 GPT-4o: {choice.message.content}")

        if choice.finish_reason == "stop":
            print("\n✅ Agente completó la tarea.")
            break

        if choice.finish_reason == "tool_calls":
            messages.append(choice.message)

            for tool_call in choice.message.tool_calls:
                tool_name = tool_call.function.name
                tool_args = json.loads(tool_call.function.arguments)

                print(f"\n🔧 Llamando herramienta MCP: {tool_name}")
                print(f"   Argumentos: {tool_args}")

                result = await session.call_tool(tool_name, tool_args)
                result_text = result.content[0].text if result.content else "Sin resultado"
                print(f"   Resultado: {result_text[:200]}")

                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": result_text,
                })
        else:
            print(f"⚠️  Finish reason inesperado: {choice.finish_reason}")
            break


if __name__ == "__main__":
    # Tarea de ejemplo para el agente
    tarea = (
        "Explora el directorio sandbox y dime: "
        "1) qué archivos hay disponibles, "
        "2) el contenido del archivo readme.md, "
        "3) busca la palabra 'config' en todos los archivos. "
        "Finalmente, crea un archivo output/resumen.txt con un resumen de lo que encontraste."
    )
    asyncio.run(run_agent_task(tarea))
```

2. Ejecuta el agente con la tarea de ejemplo:

```bash
python agent_with_mcp.py
```

**Salida esperada (extracto):**
```
🤖 Tarea del agente: Explora el directorio sandbox y dime: ...

📡 Usando Claude (Anthropic) con 5 herramientas MCP

🔧 Llamando herramienta MCP: list_files
   Argumentos: {'directory': '.'}
   Resultado: 📁 Contenido de '.' (3 entradas):
📁 data
📁 docs
📁 output

🔧 Llamando herramienta MCP: read_file
   Argumentos: {'filepath': 'docs/readme.md'}
   Resultado: 📄 Contenido de 'docs/readme.md' (XXX bytes):
...

🤖 Claude: He explorado el sandbox y encontré lo siguiente: ...

✅ Agente completó la tarea.
```

---

## 7. Validación y Pruebas

Ejecuta los siguientes comandos de validación para confirmar que el laboratorio está completo:

```bash
# Desde lab09/ con el entorno virtual activado

# 1. Verificar que todos los archivos del lab existen
echo "=== Verificando estructura de archivos ==="
for f in filesystem_mcp_server.py test_mcp_client.py .env .gitignore sandbox/docs/readme.md sandbox/data/sample.csv; do
    [ -f "$f" ] && echo "✅ $f" || echo "❌ FALTA: $f"
done

# 2. Verificar que .env NO está trackeado por git
echo -e "\n=== Verificando seguridad de .env ==="
if git -C .. check-ignore -q lab09/.env 2>/dev/null; then
    echo "✅ .env está en .gitignore"
else
    echo "⚠️  Verifica que .env esté en .gitignore"
fi

# 3. Ejecutar suite de pruebas completa
echo -e "\n=== Ejecutando suite de pruebas MCP ==="
python test_mcp_client.py
TEST_EXIT=$?

# 4. Verificar que el log de seguridad tiene entradas de path traversal
echo -e "\n=== Verificando log de seguridad ==="
TRAVERSAL_COUNT=$(grep -c "PATH_TRAVERSAL_ATTEMPT" mcp_access.log 2>/dev/null || echo "0")
if [ "$TRAVERSAL_COUNT" -ge "4" ]; then
    echo "✅ Log de seguridad: $TRAVERSAL_COUNT intentos de traversal registrados"
else
    echo "⚠️  Log de seguridad: solo $TRAVERSAL_COUNT entradas (esperadas >= 4)"
fi

# 5. Verificar que el archivo output se creó
echo -e "\n=== Verificando operaciones de escritura ==="
[ -f "sandbox/output/test_output.txt" ] && echo "✅ sandbox/output/test_output.txt creado" || echo "❌ Archivo de output no encontrado"

# Resultado final
echo -e "\n=== Resultado final ==="
[ $TEST_EXIT -eq 0 ] && echo "🎉 Lab completado exitosamente" || echo "❌ Hay pruebas fallidas — revisa los errores"
```

**Criterios de aceptación:**

| Criterio | Verificación |
|---|---|
| Servidor MCP inicia sin errores | `python -m py_compile filesystem_mcp_server.py` retorna 0 |
| Las 5 herramientas están registradas | Suite de pruebas Fase 2 pasa |
| Pruebas funcionales pasan | Suite Fase 3: 7/7 pruebas verdes |
| Path traversal es bloqueado | Suite Fase 4: 5/5 ataques bloqueados |
| Log de accesos se genera | `mcp_access.log` existe con entradas `PATH_TRAVERSAL_ATTEMPT` |
| Resource MCP responde | Suite Fase 5 pasa |
| Archivo output se crea correctamente | `sandbox/output/test_output.txt` existe |

---

## 8. Solución de Problemas

### Problema 1: `ImportError: cannot import name 'stdio_server' from 'mcp.server.stdio'`

**Síntomas:**
```
ImportError: cannot import name 'stdio_server' from 'mcp.server.stdio'
```
El servidor no inicia y el cliente de prueba falla inmediatamente.

**Causa:**
El SDK de MCP para Python ha cambiado su estructura de módulos entre versiones. La versión instalada puede ser anterior a 1.0.0 (donde `stdio_server` estaba en una ruta diferente) o posterior con una API pública reorganizada.

**Solución:**

```bash
# Paso 1: Verificar la versión instalada
pip show mcp

# Paso 2: Actualizar a la versión más reciente estable
pip install --upgrade "mcp>=1.0.0"

# Paso 3: Si el problema persiste, verificar la API disponible
python -c "import mcp.server.stdio; print(dir(mcp.server.stdio))"

# Paso 4 (alternativa): Si la importación correcta es diferente,
# ajustar el import en filesystem_mcp_server.py según la versión:
# Para versiones < 1.0.0:
# from mcp.server.stdio import stdio_server  →  puede requerir:
# from mcp.server import stdio_server
# Consultar: python -c "from mcp.server import stdio_server; print('OK')"
```

> 📌 **Nota para el instructor:** Si la versión del SDK cambió la API pública, la alternativa de respaldo es implementar el protocolo MCP manualmente: lanzar el servidor como subproceso, enviar mensajes JSON-RPC 2.0 por stdin y leer respuestas por stdout. Consultar la especificación en `github.com/modelcontextprotocol/specification`.

---

### Problema 2: El cliente de prueba se cuelga indefinidamente sin salida

**Síntomas:**
```bash
python test_mcp_client.py
# Cursor parpadeando, sin salida, el proceso no termina
```
El proceso parece bloqueado después de imprimir el encabezado o sin imprimir nada.

**Causa:**
El servidor MCP inicia pero escribe mensajes de log en `stdout` en lugar de `stderr`, contaminando el canal de comunicación stdio del protocolo JSON-RPC. El cliente MCP espera JSON-RPC válido en stdout pero recibe texto de log, causando un deadlock de lectura.

**Solución:**

```bash
# Paso 1: Verificar que el logging del servidor usa FileHandler + StreamHandler(stderr)
# En filesystem_mcp_server.py, confirmar que StreamHandler no tiene stream=sys.stdout:
grep -n "StreamHandler" filesystem_mcp_server.py
# Debe mostrar: logging.StreamHandler()  — sin argumentos usa stderr por defecto ✅

# Paso 2: Si hay prints() o logging hacia stdout, redirigirlos a stderr:
# Cambiar: print("mensaje")
# Por:     print("mensaje", file=sys.stderr)

# Paso 3: Probar el servidor de forma aislada para ver si produce salida en stdout:
python filesystem_mcp_server.py 2>/dev/null &
SERVER_PID=$!
sleep 1
# Si el servidor está corriendo sin output en stdout, está correcto
kill $SERVER_PID 2>/dev/null

# Paso 4: Agregar timeout al cliente para diagnóstico
# Ejecutar con timeout de 30 segundos:
timeout 30 python test_mcp_client.py || echo "⏱️  Timeout — revisar logs del servidor"

# Paso 5: Revisar mcp_access.log para ver hasta dónde llegó el servidor:
tail -20 mcp_access.log
```

---

## 9. Limpieza del Entorno

```bash
# Desde el directorio lab09/

# 1. Eliminar archivos generados durante las pruebas
rm -f sandbox/output/test_output.txt
rm -f sandbox/output/resumen.txt
rm -f mcp_access.log

# 2. Limpiar caché de Python
find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null
find . -name "*.pyc" -delete 2>/dev/null

# 3. Desactivar el entorno virtual
deactivate

# 4. (Opcional) Eliminar el entorno virtual si ya no se necesita
# cd ..
# rm -rf venv_lab09

echo "✅ Limpieza completada"
```

> 🔒 **Recordatorio de seguridad:** Antes de hacer `git add .`, verifica que `mcp_access.log` y `.env` están en `.gitignore`. Los logs pueden contener rutas del sistema que no deben versionarse.

---

## 10. Resumen

### Conceptos aplicados en este laboratorio

| Concepto | Implementación en el lab |
|---|---|
| **Arquitectura MCP** (Host/Client/Server) | `test_mcp_client.py` actúa como Host+Client; `filesystem_mcp_server.py` es el Server |
| **Transporte stdio (JSON-RPC 2.0)** | `stdio_server()` y `stdio_client()` del SDK oficial |
| **Tools (Herramientas MCP)** | 5 herramientas: `list_files`, `read_file`, `write_file`, `search_in_files`, `get_file_metadata` |
| **Resources (Recursos MCP)** | `file://sandbox/tree` expone el árbol del sandbox como contexto estático |
| **Prompts (Plantillas MCP)** | `analizar_directorio` y `buscar_y_reportar` como plantillas reutilizables |
| **Principio de mínimo privilegio** | Sandbox de directorio, lista blanca de extensiones, escritura solo en `output/` |
| **Defensa contra path traversal** | `_validate_path()` con `Path.relative_to()` y registro en log |
| **Registro de auditoría** | `mcp_access.log` con todos los accesos, intentos denegados y errores |
| **Integración con agente LLM** | `agent_with_mcp.py` con Claude o GPT-4o en bucle agentico |

### Puntos clave

- **MCP estandariza** la comunicación entre modelos y herramientas externas usando JSON-RPC 2.0 sobre stdio o HTTP+SSE, eliminando integraciones ad-hoc.
- **El sandbox de seguridad** se implementa resolviendo rutas con `Path.resolve()` y verificando que el resultado sea relativo al `SANDBOX_DIR` — cualquier intento de `../` es detectado y bloqueado.
- **Las tres primitivas** (Tools, Resources, Prompts) cubren diferentes necesidades: acciones dinámicas, contexto estático y plantillas de interacción respectivamente.
- **El registro de auditoría** es fundamental en sistemas de producción: cada operación, especialmente los intentos de acceso denegado, debe quedar registrada para análisis forense.
- **La reutilización** es el valor central de MCP: el servidor implementado puede ser consumido por Claude Desktop, LangGraph o cualquier host compatible sin modificaciones.

### Recursos adicionales

- [Documentación oficial MCP — modelcontextprotocol.io](https://modelcontextprotocol.io/introduction)
- [SDK Python oficial de MCP — GitHub](https://github.com/modelcontextprotocol/python-sdk)
- [Especificación técnica completa — GitHub](https://github.com/modelcontextprotocol/specification)
- [Servidores MCP de referencia de la comunidad](https://github.com/modelcontextprotocol/servers)
- [Guía de seguridad: Path Traversal — OWASP](https://owasp.org/www-community/attacks/Path_Traversal)

---
