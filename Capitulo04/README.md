---LAB_START---
LAB_ID: 04-00-01
---MARKDOWN---
# Pipeline de Auditoría de Seguridad con LLM para Commits de Git

## 1. Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 40 minutos |
| **Complejidad** | Media |
| **Nivel Bloom** | Crear |
| **Lab ID** | 04-00-01 |
| **Módulo** | 4 — Revisión de Código con IA Generativa |

---

## 2. Descripción General

En este laboratorio construirás un pipeline completo en Python que extrae automáticamente los diffs de los últimos N commits de un repositorio Git usando **GitPython**, los envía a un LLM (GPT-4o o Claude 3.5 Sonnet) con un prompt especializado en seguridad, y genera un reporte consolidado en Markdown clasificando las vulnerabilidades encontradas por severidad según el estándar OWASP Top 10.

El laboratorio aplica directamente los conceptos de la lección 4.1: usarás los mismos patrones de prompting con temperatura baja para obtener respuestas deterministas, el rol de revisor experto en seguridad, y la capacidad de los modelos modernos para razonar sobre código Python vulnerable. Al finalizar, tendrás un artefacto funcional que puede configurarse como pre-commit hook en cualquier proyecto.

---

## 3. Objetivos de Aprendizaje

- [ ] Configurar un pipeline Python que use GitPython para extraer diffs de commits con su metadata completa (hash, autor, mensaje, timestamp).
- [ ] Diseñar un prompt de sistema con role prompting y ejemplos few-shot que instruya a un LLM para identificar vulnerabilidades OWASP Top 10 y devolver resultados en JSON estructurado.
- [ ] Implementar funciones de auditoría que parseen la respuesta del LLM, clasifiquen hallazgos por severidad y generen un reporte Markdown profesional.
- [ ] Evaluar críticamente los casos donde el LLM falla (falsos positivos, falsos negativos) documentando las limitaciones observadas.

---

## 4. Prerrequisitos

### Conocimiento previo
- Manejo básico de Git (commits, diffs, repositorios locales).
- Haber completado Lab 01-00-01 (configuración de SDKs de OpenAI/Anthropic) y Lab 03-00-01 (prompt engineering).
- Comprensión conceptual de OWASP Top 10 (SQL Injection, Path Traversal, Insecure Deserialization, Hardcoded Credentials, Missing Input Validation).
- Python 3.11 instalado y operativo.

### Acceso requerido
- API Key válida de **OpenAI** (acceso a `gpt-4o`) **o** de **Anthropic** (acceso a `claude-3-5-sonnet-20241022`). Al menos una es obligatoria.
- Git 2.40+ instalado y configurado (`git config --global user.email` y `user.name` definidos).
- Conexión a internet estable (las llamadas a la API consumen aproximadamente $0.05–$0.15 USD para completar el lab completo).

> ⚠️ **Aviso de costos**: Este lab consume tokens de APIs de pago. Estima un máximo de **$0.20 USD** si auditas los 5 commits con GPT-4o. Verifica tu límite de gasto en la consola del proveedor antes de iniciar.

---

## 5. Entorno del Laboratorio

### Hardware mínimo requerido

| Componente | Mínimo | Recomendado |
|---|---|---|
| RAM | 8 GB | 16 GB |
| Almacenamiento libre | 500 MB | 1 GB |
| CPU | 2 núcleos | 4 núcleos |
| Red | 10 Mbps | 20 Mbps |

### Software y dependencias

| Paquete | Versión | Propósito |
|---|---|---|
| Python | 3.11.x | Runtime principal |
| GitPython | 3.1.x | Extracción de diffs de Git |
| openai | 1.35.x | SDK para GPT-4o |
| anthropic | 0.28.x | SDK para Claude 3.5 Sonnet (alternativa) |
| pydantic | 2.7.x | Validación de modelos de datos |
| python-dotenv | 1.0.x | Gestión segura de API keys |
| tenacity | 8.3.x | Reintentos automáticos en llamadas a la API |

### Configuración inicial del entorno

Ejecuta los siguientes comandos en tu terminal para preparar el entorno de trabajo:

```bash
# 1. Crear directorio del laboratorio
mkdir lab-04-security-auditor
cd lab-04-security-auditor

# 2. Crear y activar entorno virtual aislado
python3.11 -m venv .venv

# En Linux/macOS:
source .venv/bin/activate

# En Windows (PowerShell):
.venv\Scripts\Activate.ps1

# 3. Instalar dependencias
pip install --upgrade pip
pip install gitpython==3.1.43 openai==1.35.3 anthropic==0.28.0 \
            pydantic==2.7.4 python-dotenv==1.0.1 tenacity==8.3.0

# 4. Verificar instalaciones
python -c "import git; import openai; import pydantic; print('Dependencias OK')"
```

### Configuración de credenciales

```bash
# Crear archivo .env (NUNCA subir al repositorio)
cat > .env << 'EOF'
# Usar una o ambas según disponibilidad
OPENAI_API_KEY=sk-...tu_clave_aqui...
ANTHROPIC_API_KEY=sk-ant-...tu_clave_aqui...

# Proveedor activo: "openai" o "anthropic"
LLM_PROVIDER=openai
EOF

# Crear .gitignore para proteger credenciales
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
*.pyo
reports/
*.egg-info/
.DS_Store
EOF
```

> 🔒 **Seguridad**: Verifica que `.env` aparece en `.gitignore` antes de hacer cualquier commit. Nunca hardcodees API keys en el código fuente.

---

## 6. Instrucciones Paso a Paso

### Paso 1: Crear el Repositorio Git de Ejemplo con Código Vulnerable

**Objetivo**: Preparar un repositorio Git local con 5 commits que contengan código Python intencionalmente vulnerable, simulando un historial de desarrollo real con deuda de seguridad.

#### Instrucciones

**1.1** Crea el repositorio y la estructura de directorios:

```bash
# Dentro de lab-04-security-auditor/
mkdir vulnerable_repo
cd vulnerable_repo
git init
git config user.email "lab-student@example.com"
git config user.name "Lab Student"
```

**1.2** Crea el **Commit 1** — SQL Injection:

```bash
cat > db_utils.py << 'EOF'
# db_utils.py - Utilidades de base de datos
import sqlite3

def get_db_connection():
    conn = sqlite3.connect("app.db")
    return conn

def get_user_by_id(user_id):
    """Obtiene un usuario por su ID desde la base de datos."""
    conn = get_db_connection()
    cursor = conn.cursor()
    # VULNERABILIDAD: SQL Injection - concatenación directa de entrada del usuario
    query = "SELECT * FROM users WHERE id = " + user_id
    cursor.execute(query)
    result = cursor.fetchone()
    conn.close()
    return result

def get_user_by_name(username):
    """Busca usuarios por nombre."""
    conn = get_db_connection()
    cursor = conn.cursor()
    # VULNERABILIDAD: SQL Injection con formato de string
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor.execute(query)
    results = cursor.fetchall()
    conn.close()
    return results
EOF

git add db_utils.py
git commit -m "feat: add database utility functions for user lookup"
```

**1.3** Crea el **Commit 2** — Hardcoded Credentials:

```bash
cat > config.py << 'EOF'
# config.py - Configuración de la aplicación
import boto3

# VULNERABILIDAD: Credenciales hardcodeadas en código fuente
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
DB_PASSWORD = "SuperSecretPassword123!"
ADMIN_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.hardcoded"

def get_s3_client():
    """Crea cliente S3 con credenciales hardcodeadas."""
    return boto3.client(
        "s3",
        aws_access_key_id=AWS_ACCESS_KEY,
        aws_secret_access_key=AWS_SECRET_KEY,
        region_name="us-east-1"
    )

def get_db_config():
    return {
        "host": "prod-db.internal",
        "port": 5432,
        "database": "production",
        "password": DB_PASSWORD,
        "user": "admin"
    }
EOF

git add config.py
git commit -m "feat: add AWS S3 client configuration and DB settings"
```

**1.4** Crea el **Commit 3** — Path Traversal:

```bash
cat > file_handler.py << 'EOF'
# file_handler.py - Manejo de archivos del usuario
import os

UPLOAD_DIR = "/var/app/uploads"

def read_user_file(filename):
    """Lee un archivo subido por el usuario."""
    # VULNERABILIDAD: Path Traversal - sin sanitización del nombre de archivo
    file_path = os.path.join(UPLOAD_DIR, filename)
    with open(file_path, "r") as f:
        return f.read()

def get_report(report_name):
    """Descarga un reporte por nombre."""
    # VULNERABILIDAD: Path Traversal permite acceder a /etc/passwd con ../../etc/passwd
    base_dir = "/var/app/reports"
    full_path = base_dir + "/" + report_name
    with open(full_path, "rb") as f:
        return f.read()

def save_user_upload(filename, content):
    """Guarda un archivo subido por el usuario."""
    # Sin validación de extensión ni sanitización
    dest = os.path.join(UPLOAD_DIR, filename)
    with open(dest, "w") as f:
        f.write(content)
    return dest
EOF

git add file_handler.py
git commit -m "feat: implement file upload and report download handlers"
```

**1.5** Crea el **Commit 4** — Insecure Deserialization:

```bash
cat > session_manager.py << 'EOF'
# session_manager.py - Gestión de sesiones de usuario
import pickle
import base64

def load_session(session_data: str):
    """Carga una sesión de usuario desde datos codificados en base64."""
    # VULNERABILIDAD: Insecure Deserialization - pickle con datos no confiables
    # Un atacante puede enviar un payload pickle malicioso para ejecutar código arbitrario
    raw_data = base64.b64decode(session_data)
    session = pickle.loads(raw_data)
    return session

def save_session(session_obj) -> str:
    """Serializa la sesión del usuario para almacenamiento en cookie."""
    # VULNERABILIDAD: Usar pickle para serializar datos que van al cliente
    serialized = pickle.dumps(session_obj)
    return base64.b64encode(serialized).decode("utf-8")

def restore_user_preferences(cookie_value: str) -> dict:
    """Restaura preferencias del usuario desde cookie."""
    # VULNERABILIDAD: Deserialización directa de cookie del cliente
    data = base64.b64decode(cookie_value)
    prefs = pickle.loads(data)
    return prefs
EOF

git add session_manager.py
git commit -m "feat: add session management with cookie-based persistence"
```

**1.6** Crea el **Commit 5** — Missing Input Validation:

```bash
cat > api_handlers.py << 'EOF'
# api_handlers.py - Handlers de la API REST
import subprocess
import os

def process_user_input(data: dict):
    """Procesa entrada del usuario para generar reportes."""
    # VULNERABILIDAD: Sin validación de tipos ni rangos
    user_id = data["user_id"]
    page_size = data["page_size"]
    output_format = data["format"]

    # VULNERABILIDAD: Command Injection - entrada del usuario en subprocess
    cmd = f"generate_report.sh {user_id} {output_format}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

def resize_image(image_path: str, width: str, height: str):
    """Redimensiona una imagen usando ImageMagick."""
    # VULNERABILIDAD: Command Injection con parámetros del usuario
    os.system(f"convert {image_path} -resize {width}x{height} output.jpg")

def get_paginated_results(table: str, page: int, size: int):
    """Obtiene resultados paginados de una tabla."""
    import sqlite3
    conn = sqlite3.connect("app.db")
    # VULNERABILIDAD: Sin validación de 'table', permite inyección
    query = f"SELECT * FROM {table} LIMIT {size} OFFSET {page * size}"
    return conn.execute(query).fetchall()
EOF

git add api_handlers.py
git commit -m "feat: add REST API handlers with pagination and image processing"
```

**1.7** Verifica el historial de commits:

```bash
git log --oneline
```

#### Salida esperada

```
a3f9c21 feat: add REST API handlers with pagination and image processing
b7e2d18 feat: add session management with cookie-based persistence
c1a4f09 feat: implement file upload and report download handlers
d8b3e52 feat: add AWS S3 client configuration and DB settings
e5c9a11 feat: add database utility functions for user lookup
```

*(Los hashes serán diferentes en tu máquina; eso es normal.)*

#### Verificación

```bash
# Confirmar que hay exactamente 5 commits
git log --oneline | wc -l
# Debe imprimir: 5

# Confirmar que los archivos vulnerables existen
ls *.py
# Debe listar: db_utils.py config.py file_handler.py session_manager.py api_handlers.py
```

---

### Paso 2: Definir los Modelos de Datos con Pydantic

**Objetivo**: Crear las estructuras de datos tipadas que representarán un diff de commit, una vulnerabilidad encontrada y el reporte de auditoría completo.

#### Instrucciones

**2.1** Regresa al directorio principal del lab y crea el archivo `models.py`:

```bash
cd ..  # Volver a lab-04-security-auditor/
```

```python
# models.py
"""
Modelos de datos Pydantic v2 para el pipeline de auditoría de seguridad.
"""
from datetime import datetime
from enum import Enum
from typing import Optional
from pydantic import BaseModel, Field


class Severidad(str, Enum):
    """Niveles de severidad para vulnerabilidades de seguridad."""
    CRITICA = "CRITICA"
    ALTA = "ALTA"
    MEDIA = "MEDIA"
    BAJA = "BAJA"
    INFORMATIVA = "INFORMATIVA"


class CategoriaOWASP(str, Enum):
    """Categorías OWASP Top 10 2021 relevantes para Python."""
    A01_BROKEN_ACCESS_CONTROL = "A01:2021 - Broken Access Control"
    A02_CRYPTOGRAPHIC_FAILURES = "A02:2021 - Cryptographic Failures"
    A03_INJECTION = "A03:2021 - Injection"
    A04_INSECURE_DESIGN = "A04:2021 - Insecure Design"
    A05_SECURITY_MISCONFIGURATION = "A05:2021 - Security Misconfiguration"
    A06_VULNERABLE_COMPONENTS = "A06:2021 - Vulnerable and Outdated Components"
    A07_AUTH_FAILURES = "A07:2021 - Identification and Authentication Failures"
    A08_INTEGRITY_FAILURES = "A08:2021 - Software and Data Integrity Failures"
    A09_LOGGING_FAILURES = "A09:2021 - Security Logging and Monitoring Failures"
    A10_SSRF = "A10:2021 - Server-Side Request Forgery"
    DESCONOCIDA = "Desconocida"


class Vulnerabilidad(BaseModel):
    """Representa una vulnerabilidad encontrada en el código."""
    tipo: str = Field(..., description="Nombre técnico de la vulnerabilidad (ej: SQL Injection)")
    categoria_owasp: CategoriaOWASP = Field(
        default=CategoriaOWASP.DESCONOCIDA,
        description="Categoría OWASP Top 10 correspondiente"
    )
    severidad: Severidad = Field(..., description="Nivel de severidad")
    linea_afectada: Optional[int] = Field(
        default=None,
        description="Número de línea aproximado en el diff"
    )
    descripcion: str = Field(..., description="Descripción técnica de la vulnerabilidad")
    codigo_vulnerable: Optional[str] = Field(
        default=None,
        description="Fragmento de código vulnerable"
    )
    recomendacion: str = Field(..., description="Cómo remediar la vulnerabilidad")
    confianza: float = Field(
        default=0.8,
        ge=0.0,
        le=1.0,
        description="Nivel de confianza del LLM en este hallazgo (0.0-1.0)"
    )


class CommitDiff(BaseModel):
    """Representa el diff de un commit con su metadata."""
    commit_hash: str = Field(..., description="Hash SHA completo del commit")
    hash_corto: str = Field(..., description="Primeros 8 caracteres del hash")
    autor: str = Field(..., description="Nombre del autor del commit")
    email_autor: str = Field(default="", description="Email del autor")
    mensaje: str = Field(..., description="Mensaje del commit")
    timestamp: datetime = Field(..., description="Fecha y hora del commit")
    diff_texto: str = Field(..., description="Contenido completo del diff")
    archivos_modificados: list[str] = Field(
        default_factory=list,
        description="Lista de archivos modificados en el commit"
    )


class AuditReport(BaseModel):
    """Reporte de auditoría de seguridad para un commit."""
    commit: CommitDiff = Field(..., description="Información del commit auditado")
    vulnerabilidades: list[Vulnerabilidad] = Field(
        default_factory=list,
        description="Lista de vulnerabilidades encontradas"
    )
    resumen_ejecutivo: str = Field(
        default="",
        description="Resumen en lenguaje natural del análisis"
    )
    puntuacion_riesgo: float = Field(
        default=0.0,
        ge=0.0,
        le=10.0,
        description="Puntuación de riesgo global del commit (0-10)"
    )
    llm_proveedor: str = Field(default="openai", description="Proveedor LLM usado")
    llm_modelo: str = Field(default="gpt-4o", description="Modelo LLM usado")
    error_auditoria: Optional[str] = Field(
        default=None,
        description="Mensaje de error si la auditoría falló"
    )

    @property
    def total_por_severidad(self) -> dict[str, int]:
        """Cuenta vulnerabilidades agrupadas por severidad."""
        conteo = {s.value: 0 for s in Severidad}
        for vuln in self.vulnerabilidades:
            conteo[vuln.severidad.value] += 1
        return conteo
```

#### Verificación

```bash
python -c "from models import CommitDiff, AuditReport, Vulnerabilidad, Severidad; print('Modelos Pydantic OK')"
```

---

### Paso 3: Implementar la Extracción de Diffs con GitPython

**Objetivo**: Crear la función `get_commit_diffs` que usa GitPython para extraer el diff de los últimos N commits con toda la metadata necesaria.

#### Instrucciones

**3.1** Crea el archivo `git_extractor.py`:

```python
# git_extractor.py
"""
Módulo para extraer diffs de commits usando GitPython.
"""
import logging
from datetime import datetime, timezone
from pathlib import Path
from typing import Optional

import git
from git import Repo, InvalidGitRepositoryError

from models import CommitDiff

logger = logging.getLogger(__name__)


def get_commit_diffs(
    repo_path: str,
    n_commits: int = 5,
    branch: str = "HEAD"
) -> list[CommitDiff]:
    """
    Extrae los diffs de los últimos N commits de un repositorio Git.

    Args:
        repo_path: Ruta absoluta o relativa al repositorio Git.
        n_commits: Número de commits a analizar (desde el más reciente).
        branch: Rama o referencia desde la cual contar los commits.

    Returns:
        Lista de CommitDiff ordenada del más reciente al más antiguo.

    Raises:
        InvalidGitRepositoryError: Si la ruta no es un repositorio Git válido.
        ValueError: Si n_commits es menor o igual a 0.
    """
    if n_commits <= 0:
        raise ValueError(f"n_commits debe ser mayor a 0, se recibió: {n_commits}")

    repo_path_obj = Path(repo_path).resolve()
    if not repo_path_obj.exists():
        raise FileNotFoundError(f"Ruta no encontrada: {repo_path_obj}")

    try:
        repo = Repo(str(repo_path_obj))
    except InvalidGitRepositoryError:
        raise InvalidGitRepositoryError(
            f"'{repo_path_obj}' no es un repositorio Git válido. "
            "Asegúrate de haber ejecutado 'git init' en ese directorio."
        )

    if repo.bare:
        raise ValueError("No se puede analizar un repositorio bare.")

    commits_lista = list(repo.iter_commits(branch, max_count=n_commits))
    logger.info(f"Encontrados {len(commits_lista)} commits en {repo_path_obj}")

    result: list[CommitDiff] = []

    for commit in commits_lista:
        diff_texto = _extraer_diff_texto(repo, commit)
        archivos = _extraer_archivos_modificados(commit)

        # Convertir timestamp de Git (Unix epoch) a datetime con timezone
        commit_dt = datetime.fromtimestamp(
            commit.committed_date,
            tz=timezone.utc
        )

        commit_diff = CommitDiff(
            commit_hash=commit.hexsha,
            hash_corto=commit.hexsha[:8],
            autor=commit.author.name or "Desconocido",
            email_autor=commit.author.email or "",
            mensaje=commit.message.strip(),
            timestamp=commit_dt,
            diff_texto=diff_texto,
            archivos_modificados=archivos
        )
        result.append(commit_diff)

    logger.info(f"Extracción completada: {len(result)} CommitDiff generados.")
    return result


def _extraer_diff_texto(repo: Repo, commit: git.Commit) -> str:
    """
    Extrae el texto del diff para un commit dado.
    Para el primer commit (sin padre), muestra el contenido completo de los archivos.
    """
    try:
        if commit.parents:
            # Commit normal: diff contra su padre
            parent = commit.parents[0]
            diffs = parent.diff(commit, create_patch=True)
        else:
            # Primer commit del repositorio: diff contra árbol vacío
            diffs = commit.diff(git.NULL_TREE, create_patch=True)

        partes = []
        for diff_item in diffs:
            try:
                patch = diff_item.diff
                if isinstance(patch, bytes):
                    patch = patch.decode("utf-8", errors="replace")
                partes.append(
                    f"--- Archivo: {diff_item.b_path or diff_item.a_path} ---\n{patch}"
                )
            except Exception as e:
                logger.warning(f"No se pudo decodificar diff para archivo: {e}")
                partes.append(f"--- Archivo: {diff_item.b_path} --- [Error al decodificar]\n")

        return "\n".join(partes) if partes else "[Sin cambios de código detectados]"

    except Exception as e:
        logger.error(f"Error extrayendo diff del commit {commit.hexsha[:8]}: {e}")
        return f"[Error al extraer diff: {str(e)}]"


def _extraer_archivos_modificados(commit: git.Commit) -> list[str]:
    """Retorna la lista de archivos modificados en el commit."""
    archivos = set()
    try:
        if commit.parents:
            for diff_item in commit.parents[0].diff(commit):
                if diff_item.b_path:
                    archivos.add(diff_item.b_path)
                if diff_item.a_path:
                    archivos.add(diff_item.a_path)
        else:
            for blob in commit.tree.traverse():
                if hasattr(blob, "path"):
                    archivos.add(blob.path)
    except Exception as e:
        logger.warning(f"Error listando archivos del commit: {e}")

    return sorted(archivos)
```

#### Verificación

```bash
python -c "
from git_extractor import get_commit_diffs
diffs = get_commit_diffs('./vulnerable_repo', n_commits=3)
for d in diffs:
    print(f'[{d.hash_corto}] {d.mensaje[:50]} | Archivos: {d.archivos_modificados}')
"
```

#### Salida esperada

```
[a3f9c21] feat: add REST API handlers with pagination | Archivos: ['api_handlers.py']
[b7e2d18] feat: add session management with cookie-ba | Archivos: ['session_manager.py']
[c1a4f09] feat: implement file upload and report down | Archivos: ['file_handler.py']
```

---

### Paso 4: Construir el Prompt de Seguridad con Few-Shot

**Objetivo**: Diseñar la función `build_security_prompt` que construye un prompt especializado en OWASP con ejemplos few-shot y formato de respuesta JSON estricto.

#### Instrucciones

**4.1** Crea el archivo `prompt_builder.py`:

```python
# prompt_builder.py
"""
Constructor de prompts especializados en auditoría de seguridad de código.
Aplica role prompting y few-shot examples para maximizar la precisión del LLM.
"""

SYSTEM_PROMPT_SEGURIDAD = """Eres un experto en seguridad de aplicaciones con 15 años de experiencia \
en auditorías de código Python. Tienes certificaciones OSCP, CEH y eres coautor de guías OWASP. \
Tu especialidad es identificar vulnerabilidades críticas en código Python antes de que lleguen a producción.

## Tu Misión
Analizar el diff de un commit de Git e identificar TODAS las vulnerabilidades de seguridad presentes, \
clasificándolas según OWASP Top 10 2021.

## Categorías OWASP que debes detectar en Python
- **A03:2021 - Injection**: SQL Injection, Command Injection, LDAP Injection, Template Injection
- **A02:2021 - Cryptographic Failures**: Credenciales hardcodeadas, claves débiles, datos sensibles en texto plano
- **A01:2021 - Broken Access Control**: Path Traversal, acceso a recursos sin autorización
- **A08:2021 - Software and Data Integrity Failures**: Insecure Deserialization (pickle, yaml.load, marshal)
- **A04:2021 - Insecure Design**: Missing input validation, lógica de negocio insegura
- **A05:2021 - Security Misconfiguration**: Debug mode en producción, permisos excesivos

## Formato de Respuesta OBLIGATORIO
Debes responder ÚNICAMENTE con un objeto JSON válido siguiendo este esquema exacto. \
No incluyas texto antes ni después del JSON:

```json
{
  "vulnerabilidades": [
    {
      "tipo": "string (nombre técnico, ej: SQL Injection)",
      "categoria_owasp": "string (ej: A03:2021 - Injection)",
      "severidad": "CRITICA|ALTA|MEDIA|BAJA|INFORMATIVA",
      "linea_afectada": "integer o null",
      "descripcion": "string (explicación técnica detallada de por qué es vulnerable)",
      "codigo_vulnerable": "string (fragmento exacto del código vulnerable)",
      "recomendacion": "string (cómo corregirlo con ejemplo de código seguro)",
      "confianza": "float entre 0.0 y 1.0"
    }
  ],
  "resumen_ejecutivo": "string (2-3 oraciones resumiendo el riesgo general del commit)",
  "puntuacion_riesgo": "float entre 0.0 y 10.0 (0=sin riesgo, 10=riesgo máximo)"
}
```

## Criterios de Severidad
- **CRITICA (9-10)**: Explotable remotamente, impacto total en confidencialidad/integridad/disponibilidad
- **ALTA (7-8)**: Explotable con condiciones específicas, impacto significativo
- **MEDIA (4-6)**: Requiere privilegios o condiciones especiales para explotar
- **BAJA (1-3)**: Difícil de explotar, impacto limitado
- **INFORMATIVA**: Mala práctica sin impacto de seguridad directo

## Reglas Importantes
1. Si NO encuentras vulnerabilidades, devuelve `"vulnerabilidades": []` con `"puntuacion_riesgo": 0.0`
2. NO inventes vulnerabilidades que no estén en el código (evita falsos positivos)
3. Reporta tu nivel de confianza honestamente (usa valores bajos si hay ambigüedad)
4. El campo `codigo_vulnerable` debe contener el fragmento EXACTO del diff
5. Las recomendaciones deben incluir código Python seguro como ejemplo"""


FEW_SHOT_EJEMPLOS = """
## Ejemplos de Análisis (Few-Shot)

### Ejemplo 1: SQL Injection detectada
**Diff de entrada:**
```
+def buscar_producto(nombre):
+    query = "SELECT * FROM productos WHERE nombre = '" + nombre + "'"
+    return db.execute(query)
```

**Respuesta correcta:**
```json
{
  "vulnerabilidades": [
    {
      "tipo": "SQL Injection",
      "categoria_owasp": "A03:2021 - Injection",
      "severidad": "CRITICA",
      "linea_afectada": 2,
      "descripcion": "La función concatena directamente la entrada del usuario en la query SQL sin sanitización. Un atacante puede inyectar SQL arbitrario, por ejemplo: nombre = \"' OR '1'='1\" para obtener todos los registros, o usar UNION para extraer datos de otras tablas.",
      "codigo_vulnerable": "query = \"SELECT * FROM productos WHERE nombre = '\" + nombre + \"'\"",
      "recomendacion": "Usar consultas parametrizadas: `query = 'SELECT * FROM productos WHERE nombre = ?'; db.execute(query, (nombre,))`",
      "confianza": 0.99
    }
  ],
  "resumen_ejecutivo": "El commit introduce una vulnerabilidad crítica de SQL Injection que permite a cualquier usuario no autenticado leer, modificar o eliminar datos de la base de datos.",
  "puntuacion_riesgo": 9.5
}
```

### Ejemplo 2: Código sin vulnerabilidades
**Diff de entrada:**
```
+def calcular_descuento(precio: float, porcentaje: float) -> float:
+    if not (0 <= porcentaje <= 100):
+        raise ValueError("El porcentaje debe estar entre 0 y 100")
+    return precio * (1 - porcentaje / 100)
```

**Respuesta correcta:**
```json
{
  "vulnerabilidades": [],
  "resumen_ejecutivo": "El commit implementa una función de cálculo con validación de entrada adecuada. No se identificaron vulnerabilidades de seguridad.",
  "puntuacion_riesgo": 0.0
}
```
"""


def build_security_prompt(diff: str, incluir_few_shot: bool = True) -> str:
    """
    Construye el prompt de usuario para auditoría de seguridad de un diff.

    Args:
        diff: Texto del diff del commit a analizar.
        incluir_few_shot: Si True, incluye ejemplos few-shot en el prompt.

    Returns:
        String con el prompt completo listo para enviar al LLM.
    """
    # Truncar diffs muy largos para evitar exceder la ventana de contexto
    # GPT-4o tiene 128K tokens; 1 token ≈ 4 caracteres, limitamos a ~100K chars
    MAX_DIFF_CHARS = 80_000
    if len(diff) > MAX_DIFF_CHARS:
        diff_truncado = diff[:MAX_DIFF_CHARS] + "\n\n[... DIFF TRUNCADO POR LONGITUD ...]"
    else:
        diff_truncado = diff

    partes = []

    if incluir_few_shot:
        partes.append(FEW_SHOT_EJEMPLOS)

    partes.append(f"""
## Diff a Analizar

Analiza el siguiente diff de Git e identifica todas las vulnerabilidades de seguridad presentes.
Recuerda: responde ÚNICAMENTE con el objeto JSON, sin texto adicional.

```diff
{diff_truncado}
```
""")

    return "\n".join(partes)


def get_system_prompt() -> str:
    """Retorna el prompt de sistema para el LLM."""
    return SYSTEM_PROMPT_SEGURIDAD
```

#### Verificación

```bash
python -c "
from prompt_builder import build_security_prompt, get_system_prompt
prompt = build_security_prompt('+ query = \"SELECT * FROM users WHERE id = \" + user_id')
print(f'System prompt: {len(get_system_prompt())} caracteres')
print(f'User prompt: {len(prompt)} caracteres')
print('Prompt builder OK')
"
```

---

### Paso 5: Implementar el Motor de Auditoría con Soporte Multi-Proveedor

**Objetivo**: Crear la función `audit_commit` que llama al LLM, parsea la respuesta JSON y construye un `AuditReport` validado con Pydantic. Incluir manejo de errores robusto y reintentos con `tenacity`.

#### Instrucciones

**5.1** Crea el archivo `auditor_engine.py`:

```python
# auditor_engine.py
"""
Motor de auditoría de seguridad: llama al LLM y parsea los resultados.
Soporta OpenAI (GPT-4o) y Anthropic (Claude 3.5 Sonnet).
"""
import json
import logging
import os
import re
from typing import Optional

from dotenv import load_dotenv
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type,
)

from models import AuditReport, CommitDiff, Vulnerabilidad, Severidad, CategoriaOWASP
from prompt_builder import build_security_prompt, get_system_prompt

load_dotenv()
logger = logging.getLogger(__name__)

# ── Configuración del proveedor ──────────────────────────────────────────────
LLM_PROVIDER = os.getenv("LLM_PROVIDER", "openai").lower()


def _get_openai_client():
    """Inicializa el cliente OpenAI de forma lazy."""
    from openai import OpenAI
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        raise EnvironmentError(
            "OPENAI_API_KEY no está configurada. "
            "Añádela al archivo .env o como variable de entorno."
        )
    return OpenAI(api_key=api_key)


def _get_anthropic_client():
    """Inicializa el cliente Anthropic de forma lazy."""
    import anthropic
    api_key = os.getenv("ANTHROPIC_API_KEY")
    if not api_key:
        raise EnvironmentError(
            "ANTHROPIC_API_KEY no está configurada. "
            "Añádela al archivo .env o como variable de entorno."
        )
    return anthropic.Anthropic(api_key=api_key)


# ── Llamadas al LLM con reintentos ───────────────────────────────────────────

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=30),
    retry=retry_if_exception_type(Exception),
    reraise=True,
)
def _llamar_openai(system_prompt: str, user_prompt: str) -> str:
    """Llama a GPT-4o con temperatura baja para respuestas deterministas."""
    client = _get_openai_client()
    response = client.chat.completions.create(
        model="gpt-4o",
        temperature=0.1,   # Baja temperatura: análisis técnico consistente
        max_tokens=2048,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt},
        ],
    )
    return response.choices[0].message.content or ""


@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=30),
    retry=retry_if_exception_type(Exception),
    reraise=True,
)
def _llamar_anthropic(system_prompt: str, user_prompt: str) -> str:
    """Llama a Claude 3.5 Sonnet con temperatura baja."""
    client = _get_anthropic_client()
    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=2048,
        temperature=0.1,
        system=system_prompt,
        messages=[
            {"role": "user", "content": user_prompt}
        ],
    )
    return message.content[0].text if message.content else ""


def _llamar_llm(system_prompt: str, user_prompt: str) -> tuple[str, str, str]:
    """
    Llama al LLM configurado y retorna (respuesta_texto, proveedor, modelo).
    """
    if LLM_PROVIDER == "anthropic":
        return _llamar_anthropic(system_prompt, user_prompt), "anthropic", "claude-3-5-sonnet-20241022"
    else:
        return _llamar_openai(system_prompt, user_prompt), "openai", "gpt-4o"


# ── Parsing de la respuesta JSON ─────────────────────────────────────────────

def _extraer_json_de_respuesta(texto: str) -> dict:
    """
    Extrae y parsea el JSON de la respuesta del LLM.
    Maneja casos donde el LLM incluye texto adicional o bloques de código.
    """
    # Intentar parseo directo
    texto_limpio = texto.strip()
    try:
        return json.loads(texto_limpio)
    except json.JSONDecodeError:
        pass

    # Buscar JSON dentro de bloques de código ```json ... ```
    patron_json_block = r"```(?:json)?\s*(\{.*?\})\s*```"
    match = re.search(patron_json_block, texto_limpio, re.DOTALL)
    if match:
        try:
            return json.loads(match.group(1))
        except json.JSONDecodeError:
            pass

    # Buscar el primer objeto JSON en el texto
    patron_objeto = r"\{.*\}"
    match = re.search(patron_objeto, texto_limpio, re.DOTALL)
    if match:
        try:
            return json.loads(match.group(0))
        except json.JSONDecodeError:
            pass

    raise ValueError(
        f"No se pudo extraer JSON válido de la respuesta del LLM.\n"
        f"Primeros 500 caracteres de la respuesta:\n{texto[:500]}"
    )


def _construir_vulnerabilidad(vuln_dict: dict) -> Optional[Vulnerabilidad]:
    """
    Construye un objeto Vulnerabilidad desde un diccionario del LLM.
    Aplica valores por defecto para campos faltantes o inválidos.
    """
    try:
        # Normalizar severidad
        severidad_str = str(vuln_dict.get("severidad", "MEDIA")).upper()
        try:
            severidad = Severidad(severidad_str)
        except ValueError:
            logger.warning(f"Severidad desconocida '{severidad_str}', usando MEDIA")
            severidad = Severidad.MEDIA

        # Normalizar categoría OWASP
        categoria_str = vuln_dict.get("categoria_owasp", "Desconocida")
        categoria = CategoriaOWASP.DESCONOCIDA
        for cat in CategoriaOWASP:
            if cat.value.lower() in categoria_str.lower() or categoria_str.lower() in cat.value.lower():
                categoria = cat
                break

        return Vulnerabilidad(
            tipo=vuln_dict.get("tipo", "Vulnerabilidad desconocida"),
            categoria_owasp=categoria,
            severidad=severidad,
            linea_afectada=vuln_dict.get("linea_afectada"),
            descripcion=vuln_dict.get("descripcion", "Sin descripción"),
            codigo_vulnerable=vuln_dict.get("codigo_vulnerable"),
            recomendacion=vuln_dict.get("recomendacion", "Revisar manualmente"),
            confianza=float(vuln_dict.get("confianza", 0.7)),
        )
    except Exception as e:
        logger.error(f"Error construyendo Vulnerabilidad desde dict: {e}\nDict: {vuln_dict}")
        return None


# ── Función principal de auditoría ───────────────────────────────────────────

def audit_commit(commit_diff: CommitDiff) -> AuditReport:
    """
    Audita un commit usando el LLM configurado.

    Args:
        commit_diff: Objeto CommitDiff con el diff y metadata del commit.

    Returns:
        AuditReport con las vulnerabilidades encontradas y el reporte.
    """
    logger.info(
        f"Auditando commit [{commit_diff.hash_corto}]: {commit_diff.mensaje[:60]}"
    )

    system_prompt = get_system_prompt()
    user_prompt = build_security_prompt(commit_diff.diff_texto, incluir_few_shot=True)

    try:
        respuesta_texto, proveedor, modelo = _llamar_llm(system_prompt, user_prompt)
        logger.debug(f"Respuesta del LLM ({len(respuesta_texto)} chars) recibida.")

        datos_json = _extraer_json_de_respuesta(respuesta_texto)

        # Construir lista de vulnerabilidades
        vulnerabilidades = []
        for vuln_dict in datos_json.get("vulnerabilidades", []):
            vuln = _construir_vulnerabilidad(vuln_dict)
            if vuln:
                vulnerabilidades.append(vuln)

        puntuacion = float(datos_json.get("puntuacion_riesgo", 0.0))
        puntuacion = max(0.0, min(10.0, puntuacion))  # Clamp entre 0 y 10

        reporte = AuditReport(
            commit=commit_diff,
            vulnerabilidades=vulnerabilidades,
            resumen_ejecutivo=datos_json.get("resumen_ejecutivo", ""),
            puntuacion_riesgo=puntuacion,
            llm_proveedor=proveedor,
            llm_modelo=modelo,
        )

        logger.info(
            f"Commit [{commit_diff.hash_corto}]: "
            f"{len(vulnerabilidades)} vulnerabilidades, "
            f"riesgo={puntuacion:.1f}/10"
        )
        return reporte

    except Exception as e:
        logger.error(f"Error auditando commit [{commit_diff.hash_corto}]: {e}")
        return AuditReport(
            commit=commit_diff,
            vulnerabilidades=[],
            resumen_ejecutivo="",
            puntuacion_riesgo=0.0,
            llm_proveedor=LLM_PROVIDER,
            llm_modelo="desconocido",
            error_auditoria=str(e),
        )
```

#### Verificación

```bash
python -c "
import os
os.environ.setdefault('LLM_PROVIDER', 'openai')
from auditor_engine import _extraer_json_de_respuesta
test_json = '{\"vulnerabilidades\": [], \"resumen_ejecutivo\": \"OK\", \"puntuacion_riesgo\": 0.0}'
result = _extraer_json_de_respuesta(test_json)
assert result['puntuacion_riesgo'] == 0.0
print('Parsing JSON: OK')
"
```

---

### Paso 6: Generar el Reporte Consolidado en Markdown

**Objetivo**: Implementar `generate_markdown_report` que convierte la lista de `AuditReport` en un documento Markdown profesional con tabla de vulnerabilidades, estadísticas por severidad y recomendaciones priorizadas.

#### Instrucciones

**6.1** Crea el archivo `report_generator.py`:

```python
# report_generator.py
"""
Generador de reportes Markdown consolidados de auditoría de seguridad.
"""
from datetime import datetime, timezone
from models import AuditReport, Severidad


# Iconos de severidad para el reporte visual
ICONOS_SEVERIDAD = {
    Severidad.CRITICA: "🔴",
    Severidad.ALTA: "🟠",
    Severidad.MEDIA: "🟡",
    Severidad.BAJA: "🟢",
    Severidad.INFORMATIVA: "🔵",
}

ORDEN_SEVERIDAD = [
    Severidad.CRITICA,
    Severidad.ALTA,
    Severidad.MEDIA,
    Severidad.BAJA,
    Severidad.INFORMATIVA,
]


def generate_markdown_report(audits: list[AuditReport]) -> str:
    """
    Genera un reporte Markdown consolidado de todas las auditorías.

    Args:
        audits: Lista de AuditReport, uno por commit auditado.

    Returns:
        String con el reporte completo en formato Markdown.
    """
    ahora = datetime.now(tz=timezone.utc).strftime("%Y-%m-%d %H:%M:%S UTC")
    secciones = []

    # ── Encabezado ──────────────────────────────────────────────────────────
    secciones.append(f"""# 🔐 Reporte de Auditoría de Seguridad

**Generado el:** {ahora}
**Commits analizados:** {len(audits)}
**Proveedor LLM:** {audits[0].llm_proveedor if audits else "N/A"} / {audits[0].llm_modelo if audits else "N/A"}

---
""")

    # ── Resumen Ejecutivo Global ─────────────────────────────────────────────
    total_vulns = sum(len(a.vulnerabilidades) for a in audits)
    conteo_global = {s: 0 for s in Severidad}
    for audit in audits:
        for vuln in audit.vulnerabilidades:
            conteo_global[vuln.severidad] += 1

    commits_con_errores = [a for a in audits if a.error_auditoria]
    puntuacion_max = max((a.puntuacion_riesgo for a in audits), default=0.0)

    secciones.append(f"""## 📊 Resumen Ejecutivo

| Métrica | Valor |
|---|---|
| Total de vulnerabilidades | **{total_vulns}** |
| Commits con errores de auditoría | {len(commits_con_errores)} |
| Puntuación de riesgo máxima | {puntuacion_max:.1f} / 10.0 |

### Distribución por Severidad

| Severidad | Cantidad |
|---|---|
""")

    for sev in ORDEN_SEVERIDAD:
        icono = ICONOS_SEVERIDAD[sev]
        count = conteo_global[sev]
        secciones.append(f"| {icono} **{sev.value}** | {count} |\n")

    secciones.append("\n---\n")

    # ── Tabla Completa de Vulnerabilidades ───────────────────────────────────
    secciones.append("## 🗂️ Tabla Completa de Vulnerabilidades\n\n")

    todas_vulns = []
    for audit in audits:
        for vuln in audit.vulnerabilidades:
            todas_vulns.append((audit.commit.hash_corto, audit.commit.mensaje[:40], vuln))

    # Ordenar por severidad (más críticas primero)
    orden_map = {s: i for i, s in enumerate(ORDEN_SEVERIDAD)}
    todas_vulns.sort(key=lambda x: (orden_map.get(x[2].severidad, 99), -x[2].confianza))

    if todas_vulns:
        secciones.append(
            "| Commit | Tipo | OWASP | Severidad | Confianza | Archivo/Línea |\n"
            "|---|---|---|---|---|---|\n"
        )
        for hash_corto, msg, vuln in todas_vulns:
            icono = ICONOS_SEVERIDAD[vuln.severidad]
            linea = f"L{vuln.linea_afectada}" if vuln.linea_afectada else "N/A"
            confianza_pct = f"{vuln.confianza * 100:.0f}%"
            secciones.append(
                f"| `{hash_corto}` | {vuln.tipo} | {vuln.categoria_owasp.value} | "
                f"{icono} {vuln.severidad.value} | {confianza_pct} | {linea} |\n"
            )
    else:
        secciones.append("*No se encontraron vulnerabilidades en los commits analizados.*\n")

    secciones.append("\n---\n")

    # ── Detalle por Commit ───────────────────────────────────────────────────
    secciones.append("## 🔍 Análisis Detallado por Commit\n\n")

    for audit in audits:
        commit = audit.commit
        icono_riesgo = "🔴" if audit.puntuacion_riesgo >= 7 else ("🟠" if audit.puntuacion_riesgo >= 4 else "🟢")

        secciones.append(f"""### {icono_riesgo} Commit `{commit.hash_corto}` — {commit.mensaje[:60]}

| Campo | Valor |
|---|---|
| **Hash completo** | `{commit.commit_hash}` |
| **Autor** | {commit.autor} ({commit.email_autor}) |
| **Fecha** | {commit.timestamp.strftime("%Y-%m-%d %H:%M:%S UTC")} |
| **Archivos modificados** | {", ".join(f"`{f}`" for f in commit.archivos_modificados) or "N/A"} |
| **Puntuación de riesgo** | {audit.puntuacion_riesgo:.1f} / 10.0 |
| **Vulnerabilidades encontradas** | {len(audit.vulnerabilidades)} |

""")

        if audit.error_auditoria:
            secciones.append(f"> ⚠️ **Error durante la auditoría:** `{audit.error_auditoria}`\n\n")
            continue

        if audit.resumen_ejecutivo:
            secciones.append(f"**Resumen:** {audit.resumen_ejecutivo}\n\n")

        if not audit.vulnerabilidades:
            secciones.append("✅ No se detectaron vulnerabilidades en este commit.\n\n")
        else:
            for i, vuln in enumerate(audit.vulnerabilidades, 1):
                icono = ICONOS_SEVERIDAD[vuln.severidad]
                secciones.append(f"""#### {icono} Vulnerabilidad {i}: {vuln.tipo}

- **Severidad:** {vuln.severidad.value}
- **Categoría OWASP:** {vuln.categoria_owasp.value}
- **Línea afectada:** {vuln.linea_afectada or "No especificada"}
- **Confianza del LLM:** {vuln.confianza * 100:.0f}%

**Descripción:**
{vuln.descripcion}

""")
                if vuln.codigo_vulnerable:
                    secciones.append(f"""**Código vulnerable:**
```python
{vuln.codigo_vulnerable}
```

""")
                secciones.append(f"""**Recomendación:**
{vuln.recomendacion}

""")

        secciones.append("---\n\n")

    # ── Recomendaciones Priorizadas ──────────────────────────────────────────
    secciones.append("## 🎯 Recomendaciones Priorizadas\n\n")

    vulns_criticas_altas = [
        (hash_corto, vuln)
        for hash_corto, _, vuln in todas_vulns
        if vuln.severidad in (Severidad.CRITICA, Severidad.ALTA)
    ]

    if vulns_criticas_altas:
        secciones.append("Las siguientes vulnerabilidades requieren atención **inmediata**:\n\n")
        for i, (hash_corto, vuln) in enumerate(vulns_criticas_altas, 1):
            icono = ICONOS_SEVERIDAD[vuln.severidad]
            secciones.append(
                f"{i}. {icono} **[`{hash_corto}`] {vuln.tipo}** "
                f"({vuln.categoria_owasp.value}): {vuln.recomendacion[:150]}...\n\n"
            )
    else:
        secciones.append("✅ No se identificaron vulnerabilidades críticas o altas que requieran atención inmediata.\n\n")

    # ── Advertencia sobre Limitaciones de la IA ──────────────────────────────
    secciones.append("""---

## ⚠️ Limitaciones y Consideraciones

> **Este reporte fue generado por un modelo de IA Generativa y NO reemplaza una auditoría de seguridad profesional.**

### Limitaciones conocidas de la auditoría con LLM:

| Limitación | Descripción | Mitigación recomendada |
|---|---|---|
| **Falsos positivos** | El LLM puede reportar vulnerabilidades en código que en realidad es seguro por contexto | Verificar manualmente cada hallazgo antes de actuar |
| **Falsos negativos** | Vulnerabilidades complejas o de lógica de negocio pueden no ser detectadas | Complementar con herramientas SAST (Bandit, Semgrep) |
| **Alucinaciones** | El modelo puede inventar detalles de vulnerabilidades que no existen | Validar el fragmento de código citado existe en el diff real |
| **Contexto limitado** | El análisis es por commit, sin visión del sistema completo | Realizar revisiones de arquitectura periódicas |
| **Versiones de dependencias** | No analiza CVEs en dependencias de terceros | Usar `pip-audit` o Dependabot complementariamente |

### Herramientas complementarias recomendadas:
- **Bandit**: `pip install bandit && bandit -r ./tu_proyecto`
- **Semgrep**: `semgrep --config=p/python-security ./tu_proyecto`
- **pip-audit**: `pip install pip-audit && pip-audit`
""")

    return "".join(secciones)
```

#### Verificación

```bash
python -c "
from report_generator import generate_markdown_report
from models import AuditReport, CommitDiff
from datetime import datetime, timezone

# Crear un reporte de prueba mínimo
commit = CommitDiff(
    commit_hash='abc123def456',
    hash_corto='abc123de',
    autor='Test User',
    mensaje='test commit',
    timestamp=datetime.now(tz=timezone.utc),
    diff_texto='+ print(\"hello\")',
)
audit = AuditReport(commit=commit, llm_proveedor='openai', llm_modelo='gpt-4o')
reporte = generate_markdown_report([audit])
assert '# 🔐 Reporte de Auditoría de Seguridad' in reporte
print('Report generator OK')
"
```

---

### Paso 7: Ensamblar el Script Principal y Ejecutar la Auditoría

**Objetivo**: Crear el script orquestador `security_auditor.py` que integra todos los módulos, ejecuta la auditoría de los 3 commits más recientes y guarda el reporte.

#### Instrucciones

**7.1** Crea el archivo `security_auditor.py`:

```python
#!/usr/bin/env python3
# security_auditor.py
"""
Pipeline principal de auditoría de seguridad con LLM.

Uso:
    python security_auditor.py --repo ./vulnerable_repo --commits 3
    python security_auditor.py --repo ./vulnerable_repo --commits 5 --output mi_reporte.md
"""
import argparse
import logging
import sys
import time
from pathlib import Path

from dotenv import load_dotenv

from git_extractor import get_commit_diffs
from auditor_engine import audit_commit
from report_generator import generate_markdown_report

# Configurar logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[logging.StreamHandler(sys.stdout)],
)
logger = logging.getLogger("security_auditor")

load_dotenv()


def main():
    parser = argparse.ArgumentParser(
        description="Pipeline de auditoría de seguridad con LLM para commits de Git"
    )
    parser.add_argument(
        "--repo",
        type=str,
        default="./vulnerable_repo",
        help="Ruta al repositorio Git a auditar (default: ./vulnerable_repo)",
    )
    parser.add_argument(
        "--commits",
        type=int,
        default=3,
        help="Número de commits a auditar desde el más reciente (default: 3)",
    )
    parser.add_argument(
        "--output",
        type=str,
        default="",
        help="Ruta del archivo de salida Markdown (default: reports/audit_TIMESTAMP.md)",
    )
    parser.add_argument(
        "--verbose",
        action="store_true",
        help="Mostrar logs de debug",
    )
    args = parser.parse_args()

    if args.verbose:
        logging.getLogger().setLevel(logging.DEBUG)

    print("\n" + "═" * 60)
    print("  🔐 Pipeline de Auditoría de Seguridad con LLM")
    print("═" * 60)
    print(f"  Repositorio : {args.repo}")
    print(f"  Commits     : {args.commits}")
    print("═" * 60 + "\n")

    # ── Paso 1: Extraer diffs ────────────────────────────────────────────────
    logger.info("Extrayendo diffs del repositorio Git...")
    try:
        commit_diffs = get_commit_diffs(args.repo, n_commits=args.commits)
    except Exception as e:
        logger.error(f"Error al extraer diffs: {e}")
        sys.exit(1)

    if not commit_diffs:
        logger.warning("No se encontraron commits para auditar.")
        sys.exit(0)

    print(f"✅ {len(commit_diffs)} commits extraídos correctamente.\n")

    # ── Paso 2: Auditar cada commit ──────────────────────────────────────────
    audit_reports = []
    for i, commit_diff in enumerate(commit_diffs, 1):
        print(f"🔍 [{i}/{len(commit_diffs)}] Auditando: [{commit_diff.hash_corto}] {commit_diff.mensaje[:55]}")
        inicio = time.time()

        report = audit_commit(commit_diff)
        elapsed = time.time() - inicio

        vulns_count = len(report.vulnerabilidades)
        riesgo = report.puntuacion_riesgo

        if report.error_auditoria:
            print(f"   ⚠️  Error: {report.error_auditoria[:80]}")
        else:
            icono = "🔴" if riesgo >= 7 else ("🟠" if riesgo >= 4 else "🟢")
            print(f"   {icono} Riesgo: {riesgo:.1f}/10 | Vulnerabilidades: {vulns_count} | {elapsed:.1f}s")

        audit_reports.append(report)

        # Pequeña pausa entre llamadas para evitar rate limiting
        if i < len(commit_diffs):
            time.sleep(1.0)

    # ── Paso 3: Generar reporte Markdown ─────────────────────────────────────
    logger.info("Generando reporte Markdown consolidado...")
    reporte_md = generate_markdown_report(audit_reports)

    # Determinar ruta de salida
    if args.output:
        output_path = Path(args.output)
    else:
        reports_dir = Path("reports")
        reports_dir.mkdir(exist_ok=True)
        timestamp = time.strftime("%Y%m%d_%H%M%S")
        output_path = reports_dir / f"audit_{timestamp}.md"

    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(reporte_md, encoding="utf-8")

    # ── Resumen final ────────────────────────────────────────────────────────
    total_vulns = sum(len(r.vulnerabilidades) for r in audit_reports)
    print("\n" + "═" * 60)
    print("  ✅ Auditoría completada")
    print("═" * 60)
    print(f"  Commits auditados     : {len(audit_reports)}")
    print(f"  Total vulnerabilidades: {total_vulns}")
    print(f"  Reporte guardado en   : {output_path.resolve()}")
    print("═" * 60 + "\n")

    return 0


if __name__ == "__main__":
    sys.exit(main())
```

**7.2** Ejecuta la auditoría con los 3 commits más recientes:

```bash
python security_auditor.py --repo ./vulnerable_repo --commits 3
```

#### Salida esperada

```
════════════════════════════════════════════════════════════
  🔐 Pipeline de Auditoría de Seguridad con LLM
════════════════════════════════════════════════════════════
  Repositorio : ./vulnerable_repo
  Commits     : 3
════════════════════════════════════════════════════════════

✅ 3 commits extraídos correctamente.

🔍 [1/3] Auditando: [a3f9c21] feat: add REST API handlers with paginati
   🔴 Riesgo: 9.2/10 | Vulnerabilidades: 3 | 8.4s
🔍 [2/3] Auditando: [b7e2d18] feat: add session management with cookie-b
   🔴 Riesgo: 9.5/10 | Vulnerabilidades: 2 | 7.1s
🔍 [3/3] Auditando: [c1a4f09] feat: implement file upload and report down
   🔴 Riesgo: 8.8/10 | Vulnerabilidades: 3 | 7.8s

════════════════════════════════════════════════════════════
  ✅ Auditoría completada
════════════════════════════════════════════════════════════
  Commits auditados     : 3
  Total vulnerabilidades: 8
  Reporte guardado en   : /ruta/a/tu/lab/reports/audit_20241201_143022.md
════════════════════════════════════════════════════════════
```

> ⏱️ **Nota**: Los tiempos por commit dependen de la latencia de la API. Espera entre 5–15 segundos por commit.

---

### Paso 8: Configurar como Pre-commit Hook (Opcional)

**Objetivo**: Configurar el pipeline como hook de Git para que se ejecute automáticamente antes de cada commit.

#### Instrucciones

**8.1** Crea el script del hook en el repositorio vulnerable:

```bash
# Crear el hook pre-commit en el repositorio de ejemplo
cat > vulnerable_repo/.git/hooks/pre-commit << 'HOOK'
#!/bin/bash
# pre-commit hook: Auditoría de seguridad automática con LLM
# Analiza solo el último commit (el que se está a punto de crear)

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/../../.." && pwd)"
AUDITOR="$SCRIPT_DIR/security_auditor.py"
REPO_DIR="$(pwd)"

echo "🔐 Ejecutando auditoría de seguridad pre-commit..."

# Activar entorno virtual si existe
if [ -f "$SCRIPT_DIR/.venv/bin/activate" ]; then
    source "$SCRIPT_DIR/.venv/bin/activate"
fi

# Ejecutar auditoría del último commit staged
python "$AUDITOR" --repo "$REPO_DIR" --commits 1 --output /tmp/pre-commit-audit.md

EXIT_CODE=$?

if [ $EXIT_CODE -ne 0 ]; then
    echo "❌ Error en la auditoría. Revisa el log."
    exit 1
fi

echo "📄 Reporte guardado en: /tmp/pre-commit-audit.md"
echo "⚠️  Revisa el reporte antes de hacer push a producción."
# Para bloquear el commit si hay vulnerabilidades críticas, cambiar la siguiente línea a: exit 1
exit 0
HOOK

chmod +x vulnerable_repo/.git/hooks/pre-commit
echo "✅ Pre-commit hook configurado correctamente."
```

---

## 7. Validación y Pruebas

### Prueba 1: Verificar detección de SQL Injection

```bash
# Auditar específicamente el commit con SQL Injection (el más antiguo)
python security_auditor.py --repo ./vulnerable_repo --commits 5 \
  --output reports/full_audit.md

# Verificar que el reporte menciona SQL Injection
grep -i "SQL Injection" reports/full_audit.md && echo "✅ SQL Injection detectado" \
  || echo "❌ SQL Injection NO detectado (posible falso negativo)"
```

### Prueba 2: Verificar detección de Hardcoded Credentials

```bash
grep -i "hardcoded\|credential\|AWS_SECRET\|contraseña" reports/full_audit.md \
  && echo "✅ Credenciales hardcodeadas detectadas" \
  || echo "❌ Credenciales NO detectadas"
```

### Prueba 3: Verificar estructura del reporte

```bash
python -c "
reporte = open('reports/full_audit.md').read()
checks = [
    ('Encabezado principal', '# 🔐 Reporte de Auditoría'),
    ('Tabla de vulnerabilidades', '## 🗂️ Tabla Completa'),
    ('Análisis por commit', '## 🔍 Análisis Detallado'),
    ('Recomendaciones', '## 🎯 Recomendaciones'),
    ('Advertencia limitaciones', '## ⚠️ Limitaciones'),
]
for nombre, patron in checks:
    estado = '✅' if patron in reporte else '❌'
    print(f'{estado} {nombre}')
"
```

### Prueba 4: Auditar los 5 commits y documentar fallos

```bash
# Ejecutar auditoría completa
python security_auditor.py --repo ./vulnerable_repo --commits 5 \
  --output reports/audit_completa.md --verbose

# Contar vulnerabilidades encontradas vs. esperadas (5 commits, ~10 vulnerabilidades)
python -c "
reporte = open('reports/audit_completa.md').read()
import re
vulns = re.findall(r'Vulnerabilidad \d+:', reporte)
print(f'Vulnerabilidades reportadas: {len(vulns)}')
print(f'Esperadas (mínimo): 5 (una por commit)')
print(f'Cobertura estimada: {min(100, len(vulns)/5*100):.0f}%')
"
```

**Tabla de resultados esperados vs. observados** (completa durante el lab):

| Commit | Vulnerabilidad Esperada | Detectada por LLM | Severidad Reportada | Notas |
|---|---|---|---|---|
| `e5c9a11` | SQL Injection | ✅/❌ | — | |
| `d8b3e52` | Hardcoded Credentials | ✅/❌ | — | |
| `c1a4f09` | Path Traversal | ✅/❌ | — | |
| `b7e2d18` | Insecure Deserialization | ✅/❌ | — | |
| `a3f9c21` | Command Injection | ✅/❌ | — | |

---

## 8. Solución de Problemas

### Problema 1: `JSONDecodeError` — El LLM no devuelve JSON válido

**Síntoma:**
```
ValueError: No se pudo extraer JSON válido de la respuesta del LLM.
Primeros 500 caracteres de la respuesta:
Claro, aquí está mi análisis del código...
```

**Causa:**
El LLM ignoró la instrucción de responder únicamente con JSON y añadió texto introductorio. Esto ocurre con mayor frecuencia cuando el diff es muy largo o cuando el modelo recibe un contexto confuso.

**Solución:**
1. Verifica que el `SYSTEM_PROMPT_SEGURIDAD` en `prompt_builder.py` incluye la instrucción `"responde ÚNICAMENTE con el objeto JSON"`.
2. Reduce el tamaño del diff truncando a menos caracteres: en `build_security_prompt`, cambia `MAX_DIFF_CHARS = 80_000` a `MAX_DIFF_CHARS = 40_000`.
3. Añade `response_format={"type": "json_object"}` al llamado de OpenAI (solo disponible en GPT-4o y gpt-3.5-turbo-1106+):
```python
response = client.chat.completions.create(
    model="gpt-4o",
    temperature=0.1,
    max_tokens=2048,
    response_format={"type": "json_object"},  # ← Añadir esta línea
    messages=[...]
)
```
4. Si el problema persiste, activa `--verbose` para ver la respuesta completa del LLM y ajusta el prompt manualmente.

---

### Problema 2: `InvalidGitRepositoryError` al ejecutar el pipeline

**Síntoma:**
```
git.exc.InvalidGitRepositoryError: /ruta/a/vulnerable_repo es un repositorio inválido
```
o
```
FileNotFoundError: Ruta no encontrada: /ruta/a/vulnerable_repo
```

**Causa:**
La ruta al repositorio es incorrecta, el directorio no existe, o el repositorio no fue inicializado correctamente en el Paso 1 (falta el `git init`).

**Solución:**
1. Verifica que el directorio existe y tiene un subdirectorio `.git`:
```bash
ls -la vulnerable_repo/
ls -la vulnerable_repo/.git/
```
2. Confirma la ruta absoluta y úsala explícitamente:
```bash
pwd  # Obtener directorio actual
python security_auditor.py --repo $(pwd)/vulnerable_repo --commits 3
```
3. Si el directorio `.git` no existe, repite el Paso 1 completo:
```bash
rm -rf vulnerable_repo/
# Volver a ejecutar los comandos del Paso 1
```
4. Verifica que hay al menos 1 commit en el repositorio:
```bash
cd vulnerable_repo && git log --oneline && cd ..
```

---

## 9. Limpieza del Entorno

Una vez completado el laboratorio, ejecuta los siguientes comandos para limpiar los recursos generados:

```bash
# 1. Desactivar el entorno virtual
deactivate

# 2. (Opcional) Eliminar el repositorio vulnerable de ejemplo
# PRECAUCIÓN: Solo si no necesitas los archivos para revisión posterior
rm -rf vulnerable_repo/

# 3. (Opcional) Eliminar los reportes generados
rm -rf reports/

# 4. (Opcional) Eliminar el entorno virtual para liberar espacio (~500 MB)
rm -rf .venv/

# 5. Verificar que el archivo .env NO está en ningún repositorio Git
# Si accidentalmente hiciste git init en lab-04-security-auditor/:
git status  # Verificar que .env aparece como "untracked" o en .gitignore
```

> 🔒 **Importante**: Si compartiste la carpeta del lab o subiste código a GitHub, verifica en `git log --all -- .env` que el archivo `.env` nunca fue commiteado. Si lo fue, rota inmediatamente las API keys afectadas en las consolas de OpenAI/Anthropic.

---

## 10. Resumen

En este laboratorio construiste un **pipeline completo de auditoría de seguridad con LLM** que integra los siguientes componentes:

| Componente | Archivo | Tecnología |
|---|---|---|
| Extracción de diffs | `git_extractor.py` | GitPython 3.1.x |
| Modelos de datos | `models.py` |
