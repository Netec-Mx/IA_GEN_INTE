<div align="center">

# 🧪 Laboratorio 4

## Pipeline de auditoría de seguridad con LLM para commits de Git

![Nivel](https://img.shields.io/badge/Nivel-Intermedio--Alto-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)
![Seguridad](https://img.shields.io/badge/Seguridad-OWASP%20Top%2010-DC2626?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio vas a construir un **pipeline de auditoría de seguridad asistido por LLM**. El sistema extraerá diffs de commits de Git, enviará el código a un modelo de IA con un prompt especializado en seguridad, validará la respuesta con Pydantic y generará un reporte Markdown con vulnerabilidades, severidad, categoría OWASP, recomendaciones y limitaciones.
>
> El código vulnerable que crearás es **intencionalmente inseguro** y solo sirve para análisis educativo. No lo ejecutes como aplicación real ni reutilices esos patrones en proyectos productivos.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Revisión de código con IA</td>
<td width="25%"><strong>⏱️ Duración</strong><br>40 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar, evaluar y crear</td>
<td width="25%"><strong>📦 Entregable</strong><br>Reporte Markdown + pipeline Python</td>
</tr>
</table>

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a construir un pipeline local en **Windows**, usando **Visual Studio Code** y **Git Bash**, para auditar commits de un repositorio Git con ayuda de un LLM.

El flujo será progresivo. Primero prepararás el ambiente local y crearás un repositorio de ejemplo con vulnerabilidades controladas. Después definirás modelos Pydantic para representar commits, hallazgos y reportes. Luego extraerás diffs con GitPython, construirás un prompt especializado en seguridad, probarás el análisis de forma simulada sin consumir API y finalmente ejecutarás una auditoría real con un modelo de OpenAI usando salida estructurada.

Al terminar, tendrás un sistema que puede leer los últimos commits, extraer metadata, enviar el diff al modelo, recibir hallazgos estructurados, clasificar riesgos por severidad y generar un reporte Markdown profesional.

> [!NOTE]
> Este laboratorio usa OpenAI como proveedor principal porque permite integrar de forma directa modelos Pydantic con respuestas estructuradas. Puedes extenderlo después a otros proveedores, pero la práctica base se mantiene enfocada para reducir complejidad en aula.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Preparar un proyecto Python local para auditoría de código con IA generativa.
2. Configurar variables de entorno de forma segura usando `.env`.
3. Crear un repositorio Git de ejemplo con commits vulnerables controlados.
4. Extraer diffs y metadata de commits usando GitPython.
5. Modelar hallazgos de seguridad con Pydantic.
6. Diseñar un prompt especializado para revisión de código con enfoque OWASP.
7. Ejecutar una prueba local simulada sin consumir API.
8. Implementar un motor de auditoría con OpenAI, salida estructurada y reintentos.
9. Generar un reporte Markdown consolidado con severidad, recomendaciones y métricas.
10. Evaluar críticamente los resultados del LLM identificando aciertos, falsos positivos y falsos negativos.
11. Explicar por qué un LLM complementa, pero no reemplaza, herramientas SAST ni revisiones humanas.
12. Proponer mejoras para llevar el pipeline hacia un flujo DevSecOps real.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Tener conocimientos básicos de Python.
2. Saber crear y activar un entorno virtual.
3. Conocer comandos básicos de Git: `git init`, `git add`, `git commit`, `git log` y `git diff`.
4. Comprender qué es un commit y qué es un diff.
5. Tener nociones básicas de JSON.
6. Haber trabajado previamente con Pydantic o haber completado el laboratorio anterior de respuestas estructuradas.
7. Tener una API Key de OpenAI activa.
8. Tener acceso a internet.
9. Tener instalado Git Bash en Windows.
10. Tener instalado Visual Studio Code.

---

### 💻 Hardware

| Recurso | Requisito mínimo | Recomendado |
|---|---:|---:|
| Equipo | Laptop o PC con Windows | Laptop o PC con Windows 11 |
| Procesador | Intel Core i5 / Ryzen 5 | 4 núcleos o más |
| Memoria RAM | 8 GB | 16 GB |
| Almacenamiento libre | 1 GB | 2 GB |
| GPU | No requerida | No requerida |
| Internet | Requerido para consumir API | 20 Mbps o más |

---

### 🧰 Software

| Software | Uso |
|---|---|
| Windows 10 o Windows 11 | Sistema operativo base |
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Git 2.40 o superior | Creación y análisis de commits |
| Python 3.11 o superior | Runtime del laboratorio |
| pip | Instalación de dependencias |
| OpenAI API Key | Consumo del modelo LLM |
| Extensión Markdown Preview para VS Code | Visualización del reporte Markdown |

---

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 40 minutos |
| Complejidad | Intermedia - Alta |
| Nivel de Bloom | Aplicar, analizar, evaluar y crear |
| Modalidad | Individual o equipos de 2 personas |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Proveedor principal | OpenAI |
| Modelo sugerido | Definido por variable `OPENAI_MODEL` |
| Entregable principal | Reporte Markdown de auditoría |
| Entregable secundario | Scripts Python del pipeline |
| Tipo de práctica | Técnica, aplicada y evaluativa |

---

## 🛡️ Consideraciones para estudiantes

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>No subas `.env` ni claves reales.</td>
<td><strong>💸 Costo</strong><br>Las llamadas al modelo pueden consumir saldo.</td>
<td><strong>⚠️ Alcance</strong><br>El código vulnerable es solo educativo.</td>
</tr>
</table>

1. **No compartas tu API Key.** Debe quedarse únicamente en tu equipo local.
2. **No pegues claves dentro del código.** Siempre usa `.env`.
3. **No entregues el archivo `.env`.** Solo entrega scripts y reportes.
4. **No ejecutes el código vulnerable como aplicación real.** Solo se analizará como texto dentro de commits.
5. **No uses repositorios productivos con secretos reales.** Esta práctica usa un repositorio de ejemplo.
6. **Los resultados del LLM pueden variar.** Un modelo puede detectar hallazgos distintos entre ejecuciones.
7. **El LLM puede equivocarse.** Puede generar falsos positivos o falsos negativos.
8. **El reporte generado no reemplaza una auditoría profesional.** Debe validarse con revisión humana y herramientas SAST.
9. **Controla el costo.** Audita primero 1 commit antes de correr los 5 commits.
10. **Mantén baja la temperatura.** En revisión técnica se busca consistencia, no creatividad.
11. **Complementa con herramientas de seguridad.** Bandit, Semgrep y pip-audit pueden detectar patrones que el LLM omite.
12. **Revisa la documentación del proveedor antes de impartir.** Los nombres de modelos, costos y límites pueden cambiar.

---

## 🔗 Fuentes oficiales que debes revisar antes de ejecutar

> [!NOTE]
> Los identificadores de modelos, límites, precios y capacidades cambian con frecuencia. Antes de impartir el laboratorio, valida los modelos disponibles y ajusta el valor de `OPENAI_MODEL` en el archivo `.env`.

| Tema | Qué revisar | Fuente sugerida |
|---|---|---|
| OpenAI | Modelo disponible, salida estructurada, límites y precio | Documentación oficial de OpenAI API |
| OWASP Top 10 | Categorías y criterios de riesgo | OWASP Top 10 2021 |
| GitPython | Uso de `Repo`, commits y diffs | Documentación oficial de GitPython |
| Pydantic | Modelos, enums, validación y serialización | Documentación oficial de Pydantic |
| Tenacity | Reintentos, backoff y errores transitorios | Documentación oficial de Tenacity |
| Bandit / Semgrep | Herramientas SAST complementarias | Documentación oficial de cada herramienta |

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear una carpeta de trabajo, abrirla en Visual Studio Code, preparar un entorno virtual de Python e instalar las dependencias necesarias para el pipeline de auditoría.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear una carpeta local para guardar todos los archivos del laboratorio.

**⚙️ Contenido del paso:**  
Abre **Git Bash** y ejecuta:

```bash
mkdir -p ~/labs-ia-gen/lab-04-security-auditor
cd ~/labs-ia-gen/lab-04-security-auditor
```

**✅ Validación del paso:**  
Ejecuta:

```bash
pwd
```

Debes estar dentro de una ruta similar a:

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-04-security-auditor
```

**📌 Resultado esperado:**  
Tienes una carpeta dedicada para el laboratorio 4.

---

### ✅ Paso 2. Abre la carpeta en Visual Studio Code

**📝 Descripción del paso:**  
Vas a abrir el proyecto desde Git Bash para trabajar con los archivos en VS Code.

**⚙️ Contenido del paso:**

```bash
code .
```

Si `code .` no funciona, abre VS Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-04-security-auditor
```

**✅ Validación del paso:**  
Confirma que VS Code muestre la carpeta `lab-04-security-auditor`.

**📌 Resultado esperado:**  
El proyecto está abierto en Visual Studio Code.

---

### ✅ Paso 3. Crea y activa el entorno virtual

**📝 Descripción del paso:**  
Vas a aislar las dependencias de este laboratorio para evitar conflictos con otros proyectos.

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

Debes ver que Python se ejecuta desde `.venv`.

**📌 Resultado esperado:**  
Tienes un entorno virtual activo para la práctica.

---

### ✅ Paso 4. Crea el archivo `requirements.txt`

**📝 Descripción del paso:**  
Vas a declarar las librerías que usará el pipeline.

**⚙️ Contenido del paso:**  
Crea el archivo:

```text
requirements.txt
```

Agrega:

```txt
openai>=1.90,<2
gitpython>=3.1,<4
pydantic>=2.10,<3
python-dotenv>=1.0,<2
tenacity>=8.5,<10
```

**✅ Validación del paso:**

```bash
cat requirements.txt
```

**📌 Resultado esperado:**  
Tienes declaradas las dependencias principales del laboratorio.

---

### ✅ Paso 5. Instala las dependencias

**📝 Descripción del paso:**  
Vas a instalar el SDK de OpenAI, GitPython, Pydantic, dotenv y tenacity.

**⚙️ Contenido del paso:**

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**✅ Validación del paso:**

```bash
python -c "import openai, git, pydantic, dotenv, tenacity; print('Dependencias instaladas correctamente')"
```

**📌 Resultado esperado:**

```text
Dependencias instaladas correctamente
```

---

### ✅ Paso 6. Crea el archivo `.gitignore`

**📝 Descripción del paso:**  
Vas a proteger archivos sensibles y temporales para evitar subirlos por accidente a un repositorio.

**⚙️ Contenido del paso:**

```bash
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
*.pyo
.pytest_cache/
reports/
*.log
EOF
```

**✅ Validación del paso:**

```bash
cat .gitignore
```

**📌 Resultado esperado:**  
El archivo `.gitignore` contiene `.env`, `.venv/` y `reports/`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20de%20un%20laboratorio%20donde%20prepar%C3%A9%20un%20proyecto%20Python%20en%20Windows%20con%20VS%20Code%2C%20Git%20Bash%2C%20entorno%20virtual%2C%20requirements.txt%20y%20.gitignore%20para%20construir%20un%20pipeline%20de%20auditor%C3%ADa%20de%20seguridad%20con%20LLM.)

---

# 🧩 Tarea 2. Configurar credenciales y parámetros del modelo

## 🎯 Objetivo de la tarea

Crear un archivo `.env` para configurar la API Key, el modelo y parámetros básicos de ejecución sin escribir secretos dentro del código.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `.env`

**📝 Descripción del paso:**  
Vas a guardar la clave de OpenAI y el nombre del modelo en variables de entorno.

**⚙️ Contenido del paso:**

```bash
cat > .env << 'EOF'
OPENAI_API_KEY=pega_aqui_tu_api_key_de_openai
OPENAI_MODEL=gpt-4o-mini
MAX_COMMITS_DEFAULT=3
EOF
```

**🔧 Qué debes cambiar:**  
Reemplaza `pega_aqui_tu_api_key_de_openai` por tu API Key real.

**✅ Validación del paso:**

```bash
cat .env
```

**📌 Resultado esperado:**  
Ves las variables configuradas. No compartas ni entregues este archivo.

---

### ✅ Paso 2. Crea un script para validar el entorno

**📝 Descripción del paso:**  
Vas a confirmar que Python puede leer la API Key y el modelo desde `.env` sin imprimir la clave completa.

**⚙️ Contenido del paso:**  
Crea el archivo `00_validar_entorno.py` con este contenido:

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv("OPENAI_API_KEY")
model = os.getenv("OPENAI_MODEL", "")

if not api_key or api_key.startswith("pega_aqui"):
    print("Falta configurar OPENAI_API_KEY en el archivo .env")
    raise SystemExit(1)

if not model:
    print("Falta configurar OPENAI_MODEL en el archivo .env")
    raise SystemExit(1)

print("Variables de entorno cargadas correctamente.")
print("OPENAI_API_KEY: configurada, no se muestra por seguridad.")
print(f"OPENAI_MODEL: {model}")
```

**✅ Validación del paso:**

```bash
python 00_validar_entorno.py
```

**📌 Resultado esperado:**

```text
Variables de entorno cargadas correctamente.
OPENAI_API_KEY: configurada, no se muestra por seguridad.
OPENAI_MODEL: gpt-4o-mini
```

---

### ✅ Paso 3. Verifica que `.env` esté protegido

**📝 Descripción del paso:**  
Vas a confirmar que `.env` está dentro de `.gitignore`.

**⚙️ Contenido del paso:**

```bash
grep -n "^.env$" .gitignore
```

**✅ Validación del paso:**  
El comando debe devolver una línea con `.env`.

**📌 Resultado esperado:**  
El archivo de credenciales está protegido contra commits accidentales.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20de%20un%20laboratorio%20donde%20configur%C3%A9%20un%20archivo%20.env%20con%20OPENAI_API_KEY%2C%20OPENAI_MODEL%20y%20valid%C3%A9%20que%20Python%20pueda%20leer%20las%20variables%20sin%20exponer%20secretos.)

---

# 🧩 Tarea 3. Crear un repositorio Git vulnerable de ejemplo

## 🎯 Objetivo de la tarea

Crear un repositorio Git local con 5 commits que contengan vulnerabilidades controladas para que el pipeline pueda auditarlas.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea e inicializa el repositorio vulnerable

**📝 Descripción del paso:**  
Vas a crear un repositorio independiente dentro del laboratorio. Este repositorio será usado como objeto de análisis.

**⚙️ Contenido del paso:**

```bash
mkdir vulnerable_repo
cd vulnerable_repo
git init
git config user.email "lab-student@example.com"
git config user.name "Lab Student"
```

**✅ Validación del paso:**

```bash
git status
```

**📌 Resultado esperado:**  
Git indica que estás en un repositorio vacío o recién inicializado.

---

### ✅ Paso 2. Crea el Commit 1 con SQL Injection

**📝 Descripción del paso:**  
Vas a crear código vulnerable por concatenación directa de entradas en consultas SQL.

**⚙️ Contenido del paso:**

```bash
cat > db_utils.py << 'EOF'
# db_utils.py - Utilidades de base de datos
import sqlite3

def get_db_connection():
    conn = sqlite3.connect("app.db")
    return conn

def get_user_by_id(user_id):
    conn = get_db_connection()
    cursor = conn.cursor()
    query = "SELECT * FROM users WHERE id = " + user_id
    cursor.execute(query)
    result = cursor.fetchone()
    conn.close()
    return result

def get_user_by_name(username):
    conn = get_db_connection()
    cursor = conn.cursor()
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor.execute(query)
    results = cursor.fetchall()
    conn.close()
    return results
EOF

git add db_utils.py
git commit -m "feat: add database utility functions for user lookup"
```

**✅ Validación del paso:**

```bash
git log --oneline
```

**📌 Resultado esperado:**  
Existe 1 commit en el historial.

---

### ✅ Paso 3. Crea el Commit 2 con credenciales hardcodeadas

**📝 Descripción del paso:**  
Vas a crear un archivo de configuración con secretos ficticios escritos en código.

**⚙️ Contenido del paso:**

```bash
cat > config.py << 'EOF'
# config.py - Configuración insegura de ejemplo

AWS_ACCESS_KEY = "EXAMPLE_AWS_ACCESS_KEY_DO_NOT_USE"
AWS_SECRET_KEY = "EXAMPLE_AWS_SECRET_KEY_DO_NOT_USE"
DB_PASSWORD = "EXAMPLE_DB_PASSWORD_DO_NOT_USE"
ADMIN_TOKEN = "EXAMPLE_ADMIN_TOKEN_DO_NOT_USE"

def get_db_config():
    return {
        "host": "prod-db.internal",
        "port": 5432,
        "database": "production",
        "user": "admin",
        "password": DB_PASSWORD,
    }
EOF

git add config.py
git commit -m "feat: add application configuration values"
```

**✅ Validación del paso:**

```bash
git log --oneline | wc -l
```

**📌 Resultado esperado:**  
El resultado debe ser `2`.

---

### ✅ Paso 4. Crea el Commit 3 con Path Traversal

**📝 Descripción del paso:**  
Vas a crear funciones que leen archivos usando rutas formadas directamente con entradas del usuario.

**⚙️ Contenido del paso:**

```bash
cat > file_handler.py << 'EOF'
# file_handler.py - Manejo inseguro de archivos
import os

UPLOAD_DIR = "uploads"
REPORTS_DIR = "reports"

def read_user_file(filename):
    file_path = os.path.join(UPLOAD_DIR, filename)
    with open(file_path, "r", encoding="utf-8") as file:
        return file.read()

def get_report(report_name):
    full_path = REPORTS_DIR + "/" + report_name
    with open(full_path, "rb") as file:
        return file.read()
EOF

git add file_handler.py
git commit -m "feat: implement file reading and report download"
```

**✅ Validación del paso:**

```bash
git log --oneline | wc -l
```

**📌 Resultado esperado:**  
El resultado debe ser `3`.

---

### ✅ Paso 5. Crea el Commit 4 con deserialización insegura

**📝 Descripción del paso:**  
Vas a crear funciones que usan `pickle.loads()` sobre datos que podrían provenir del cliente.

**⚙️ Contenido del paso:**

```bash
cat > session_manager.py << 'EOF'
# session_manager.py - Gestión insegura de sesiones
import base64
import pickle

def load_session(session_data: str):
    raw_data = base64.b64decode(session_data)
    session = pickle.loads(raw_data)
    return session

def restore_user_preferences(cookie_value: str) -> dict:
    data = base64.b64decode(cookie_value)
    prefs = pickle.loads(data)
    return prefs
EOF

git add session_manager.py
git commit -m "feat: add cookie based session persistence"
```

**✅ Validación del paso:**

```bash
git log --oneline | wc -l
```

**📌 Resultado esperado:**  
El resultado debe ser `4`.

---

### ✅ Paso 6. Crea el Commit 5 con Command Injection

**📝 Descripción del paso:**  
Vas a crear funciones que envían entrada del usuario a comandos del sistema.

**⚙️ Contenido del paso:**

```bash
cat > api_handlers.py << 'EOF'
# api_handlers.py - Handlers inseguros de API
import os
import subprocess

def process_user_input(data: dict):
    user_id = data["user_id"]
    output_format = data["format"]
    cmd = f"generate_report.sh {user_id} {output_format}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

def resize_image(image_path: str, width: str, height: str):
    os.system(f"convert {image_path} -resize {width}x{height} output.jpg")
EOF

git add api_handlers.py
git commit -m "feat: add report generation and image processing handlers"
```

**✅ Validación del paso:**

```bash
git log --oneline
```

**📌 Resultado esperado:**  
Debes ver 5 commits. Los hashes serán diferentes en cada equipo.

---

### ✅ Paso 7. Regresa al directorio principal del laboratorio

**📝 Descripción del paso:**  
Vas a regresar a la carpeta donde crearás el pipeline de auditoría.

**⚙️ Contenido del paso:**

```bash
cd ..
```

**✅ Validación del paso:**

```bash
pwd
ls
```

**📌 Resultado esperado:**  
Debes ver la carpeta `vulnerable_repo` dentro del laboratorio.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20de%20un%20laboratorio%20donde%20cre%C3%A9%20un%20repositorio%20Git%20local%20con%205%20commits%20vulnerables%3A%20SQL%20Injection%2C%20credenciales%20hardcodeadas%2C%20Path%20Traversal%2C%20deserializaci%C3%B3n%20insegura%20y%20Command%20Injection.)

---

# 🧩 Tarea 4. Definir los modelos Pydantic del pipeline

## 🎯 Objetivo de la tarea

Crear los modelos de datos que representarán commits, vulnerabilidades, resultados del LLM y reportes de auditoría.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `models.py`

**📝 Descripción del paso:**  
Vas a centralizar las estructuras de datos del pipeline.

**⚙️ Contenido del paso:**  
Crea el archivo `models.py` y agrega:

```python
from __future__ import annotations
from datetime import datetime
from enum import Enum
from pydantic import BaseModel, Field

class Severidad(str, Enum):
    CRITICA = "CRITICA"
    ALTA = "ALTA"
    MEDIA = "MEDIA"
    BAJA = "BAJA"
    INFORMATIVA = "INFORMATIVA"

class CategoriaOWASP(str, Enum):
    A01_BROKEN_ACCESS_CONTROL = "A01:2021 - Broken Access Control"
    A02_CRYPTOGRAPHIC_FAILURES = "A02:2021 - Cryptographic Failures"
    A03_INJECTION = "A03:2021 - Injection"
    A04_INSECURE_DESIGN = "A04:2021 - Insecure Design"
    A05_SECURITY_MISCONFIGURATION = "A05:2021 - Security Misconfiguration"
    A08_INTEGRITY_FAILURES = "A08:2021 - Software and Data Integrity Failures"
    DESCONOCIDA = "Desconocida"

class Vulnerabilidad(BaseModel):
    tipo: str = Field(description="Nombre técnico de la vulnerabilidad.")
    categoria_owasp: CategoriaOWASP = Field(description="Categoría OWASP asociada.")
    severidad: Severidad = Field(description="Severidad del hallazgo.")
    archivo: str | None = Field(default=None, description="Archivo afectado.")
    linea_afectada: int | None = Field(default=None, description="Línea aproximada afectada.")
    descripcion: str = Field(description="Explicación técnica del hallazgo.")
    codigo_vulnerable: str | None = Field(default=None, description="Fragmento vulnerable.")
    recomendacion: str = Field(description="Recomendación de corrección.")
    confianza: float = Field(default=0.8, ge=0.0, le=1.0, description="Confianza del hallazgo.")

class CommitDiff(BaseModel):
    commit_hash: str
    hash_corto: str
    autor: str
    email_autor: str = ""
    mensaje: str
    timestamp: datetime
    diff_texto: str
    archivos_modificados: list[str] = Field(default_factory=list)

class AuditLLMOutput(BaseModel):
    vulnerabilidades: list[Vulnerabilidad] = Field(default_factory=list)
    resumen_ejecutivo: str = ""
    puntuacion_riesgo: float = Field(default=0.0, ge=0.0, le=10.0)

class AuditReport(BaseModel):
    commit: CommitDiff
    vulnerabilidades: list[Vulnerabilidad] = Field(default_factory=list)
    resumen_ejecutivo: str = ""
    puntuacion_riesgo: float = Field(default=0.0, ge=0.0, le=10.0)
    llm_proveedor: str = "openai"
    llm_modelo: str = ""
    error_auditoria: str | None = None

    @property
    def total_por_severidad(self) -> dict[str, int]:
        conteo = {severidad.value: 0 for severidad in Severidad}
        for vuln in self.vulnerabilidades:
            conteo[vuln.severidad.value] += 1
        return conteo
```

**✅ Validación del paso:**

```bash
python -m py_compile models.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Valida una vulnerabilidad de prueba

**📝 Descripción del paso:**  
Vas a confirmar que Pydantic acepta una vulnerabilidad válida y rechaza valores fuera de rango.

**⚙️ Contenido del paso:**

```bash
python -c "
from models import Vulnerabilidad, Severidad, CategoriaOWASP
v = Vulnerabilidad(
    tipo='SQL Injection',
    categoria_owasp=CategoriaOWASP.A03_INJECTION,
    severidad=Severidad.CRITICA,
    archivo='db_utils.py',
    linea_afectada=12,
    descripcion='Consulta SQL construida con concatenación de entrada del usuario.',
    codigo_vulnerable='query = \"SELECT * FROM users WHERE id = \" + user_id',
    recomendacion='Usa consultas parametrizadas.',
    confianza=0.95,
)
print(v.model_dump_json(indent=2))
"
```

**✅ Validación del paso:**  
Debes ver un JSON con la vulnerabilidad.

**📌 Resultado esperado:**  
El modelo Pydantic está funcionando correctamente.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20de%20un%20laboratorio%20donde%20defin%C3%AD%20modelos%20Pydantic%20para%20representar%20commits%2C%20vulnerabilidades%2C%20salidas%20del%20LLM%20y%20reportes%20de%20auditor%C3%ADa%20de%20seguridad.)

---

# 🧩 Tarea 5. Extraer diffs de commits con GitPython

## 🎯 Objetivo de la tarea

Crear un módulo que lea los últimos commits del repositorio vulnerable y extraiga su metadata y diff de código.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `git_extractor.py`

**📝 Descripción del paso:**  
Vas a implementar funciones para abrir un repositorio Git y recuperar diffs.

**⚙️ Contenido del paso:**

```python
import logging
from datetime import datetime, timezone
from pathlib import Path
import git
from git import InvalidGitRepositoryError, Repo
from models import CommitDiff

logger = logging.getLogger(__name__)

def get_commit_diffs(repo_path: str, n_commits: int = 3, branch: str = "HEAD") -> list[CommitDiff]:
    if n_commits <= 0:
        raise ValueError("n_commits debe ser mayor que 0")
    repo_path_obj = Path(repo_path).resolve()
    if not repo_path_obj.exists():
        raise FileNotFoundError(f"Ruta no encontrada: {repo_path_obj}")
    try:
        repo = Repo(str(repo_path_obj))
    except InvalidGitRepositoryError as exc:
        raise InvalidGitRepositoryError(f"La ruta no es un repositorio Git válido: {repo_path_obj}") from exc
    commits = list(repo.iter_commits(branch, max_count=n_commits))
    resultados: list[CommitDiff] = []
    for commit in commits:
        diff_texto = _extraer_diff_texto(commit)
        archivos = _extraer_archivos_modificados(commit)
        timestamp = datetime.fromtimestamp(commit.committed_date, tz=timezone.utc)
        resultados.append(CommitDiff(
            commit_hash=commit.hexsha,
            hash_corto=commit.hexsha[:8],
            autor=commit.author.name or "Desconocido",
            email_autor=commit.author.email or "",
            mensaje=commit.message.strip(),
            timestamp=timestamp,
            diff_texto=diff_texto,
            archivos_modificados=archivos,
        ))
    return resultados

def _extraer_diff_texto(commit: git.Commit) -> str:
    if commit.parents:
        diffs = commit.parents[0].diff(commit, create_patch=True)
    else:
        diffs = commit.diff(git.NULL_TREE, create_patch=True)
    partes: list[str] = []
    for diff_item in diffs:
        archivo = diff_item.b_path or diff_item.a_path or "archivo_desconocido"
        patch = diff_item.diff
        if isinstance(patch, bytes):
            patch = patch.decode("utf-8", errors="replace")
        partes.append(f"--- Archivo: {archivo} ---\n{patch}")
    return "\n".join(partes) if partes else "[Sin cambios detectados]"

def _extraer_archivos_modificados(commit: git.Commit) -> list[str]:
    archivos = set()
    if commit.parents:
        for diff_item in commit.parents[0].diff(commit):
            if diff_item.a_path:
                archivos.add(diff_item.a_path)
            if diff_item.b_path:
                archivos.add(diff_item.b_path)
    else:
        for item in commit.tree.traverse():
            if hasattr(item, "path"):
                archivos.add(item.path)
    return sorted(archivos)
```

**✅ Validación del paso:**

```bash
python -m py_compile git_extractor.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Prueba la extracción de commits

**📝 Descripción del paso:**  
Vas a comprobar que el extractor puede leer los últimos commits del repositorio vulnerable.

**⚙️ Contenido del paso:**

```bash
python -c "
from git_extractor import get_commit_diffs
commits = get_commit_diffs('./vulnerable_repo', n_commits=3)
for commit in commits:
    print(f'[{commit.hash_corto}] {commit.mensaje}')
    print('Archivos:', commit.archivos_modificados)
    print('Diff chars:', len(commit.diff_texto))
    print('-' * 60)
"
```

**✅ Validación del paso:**  
Debes ver 3 commits, del más reciente al más antiguo.

**📌 Resultado esperado:**  
GitPython puede extraer metadata y diff del repositorio local.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20de%20un%20laboratorio%20donde%20us%C3%A9%20GitPython%20para%20extraer%20los%20%C3%BAltimos%20commits%20de%20un%20repositorio%2C%20incluyendo%20hash%2C%20autor%2C%20mensaje%2C%20fecha%2C%20archivos%20modificados%20y%20diff%20de%20c%C3%B3digo.)

---

# 🧩 Tarea 6. Construir el prompt de auditoría de seguridad

## 🎯 Objetivo de la tarea

Crear un prompt especializado que indique al LLM cómo analizar código, clasificar vulnerabilidades y producir una salida compatible con los modelos Pydantic.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `prompt_builder.py`

**📝 Descripción del paso:**  
Vas a separar el prompt del resto del código para poder ajustarlo fácilmente.

**⚙️ Contenido del paso:**

```python
SYSTEM_PROMPT_SEGURIDAD = """
Eres un auditor senior de seguridad de aplicaciones especializado en Python.
Tu tarea es revisar diffs de commits de Git e identificar vulnerabilidades reales.
No inventes vulnerabilidades que no estén respaldadas por el diff.
Clasifica los hallazgos usando OWASP Top 10 2021 cuando aplique.

Categorías frecuentes:
- A03:2021 - Injection: SQL Injection, Command Injection.
- A02:2021 - Cryptographic Failures: secretos o credenciales hardcodeadas.
- A01:2021 - Broken Access Control: Path Traversal.
- A08:2021 - Software and Data Integrity Failures: deserialización insegura.
- A04:2021 - Insecure Design: ausencia de validaciones importantes.

Criterios de severidad:
- CRITICA: explotable remotamente con impacto severo.
- ALTA: impacto importante, aunque requiera condiciones específicas.
- MEDIA: riesgo relevante con impacto o explotación limitada.
- BAJA: mala práctica con impacto bajo.
- INFORMATIVA: observación técnica sin vulnerabilidad directa.
"""

def get_system_prompt() -> str:
    return SYSTEM_PROMPT_SEGURIDAD.strip()

def build_security_prompt(diff: str, archivos: list[str]) -> str:
    max_chars = 40_000
    if len(diff) > max_chars:
        diff = diff[:max_chars] + "\n\n[DIFF TRUNCADO POR LONGITUD]"
    archivos_txt = ", ".join(archivos) if archivos else "No especificados"
    
    # Almacenamos los bloques de código en variables para no romper el f-string
    inicio_bloque = "```diff"
    fin_bloque = "```"
    
    return f"""
Analiza el siguiente diff de Git.

Archivos modificados:
{archivos_txt}

Identifica vulnerabilidades de seguridad introducidas o evidenciadas por el cambio.
Para cada vulnerabilidad, incluye tipo, categoria_owasp, severidad, archivo, linea_afectada, descripcion, codigo_vulnerable, recomendacion y confianza.

Diff:
{inicio_bloque}
{diff}
{fin_bloque}
""".strip()
```

**✅ Validación del paso:**

```bash
python -m py_compile prompt_builder.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Prueba la construcción del prompt

**📝 Descripción del paso:**  
Vas a validar que el prompt incluya el diff y los archivos modificados.

**⚙️ Contenido del paso:**

```bash
python -c "
from prompt_builder import build_security_prompt, get_system_prompt
prompt = build_security_prompt('+ query = \"SELECT * FROM users WHERE id = \" + user_id', ['db_utils.py'])
print('System prompt chars:', len(get_system_prompt()))
print('User prompt chars:', len(prompt))
print(prompt[:500])
"
```

**✅ Validación del paso:**  
Debes ver el inicio del prompt y la referencia a `db_utils.py`.

**📌 Resultado esperado:**  
El constructor de prompts está listo para usarse con el motor LLM.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20de%20un%20laboratorio%20donde%20constru%C3%AD%20un%20prompt%20especializado%20para%20auditor%C3%ADa%20de%20seguridad%20de%20commits%20con%20enfoque%20OWASP%20Top%2010%20y%20revisi%C3%B3n%20defensiva%20de%20c%C3%B3digo.)

---

# 🧩 Tarea 7. Probar una auditoría simulada sin consumir API

## 🎯 Objetivo de la tarea

Validar el flujo de datos con una respuesta simulada antes de llamar al modelo real. Esto reduce errores, costo y dependencia de red.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `fake_auditor.py`

**📝 Descripción del paso:**  
Vas a crear una función local que simula la respuesta del LLM para un commit vulnerable.

**⚙️ Contenido del paso:**

```python
from models import AuditLLMOutput, CategoriaOWASP, Severidad, Vulnerabilidad


def fake_audit_diff(diff: str) -> AuditLLMOutput:
    if "subprocess.run" in diff or "os.system" in diff:
        return AuditLLMOutput(
            vulnerabilidades=[
                Vulnerabilidad(
                    tipo="Command Injection",
                    categoria_owasp=CategoriaOWASP.A03_INJECTION,
                    severidad=Severidad.CRITICA,
                    archivo="api_handlers.py",
                    linea_afectada=None,
                    descripcion="El código construye comandos del sistema con entrada del usuario y los ejecuta con shell=True u os.system.",
                    codigo_vulnerable="subprocess.run(cmd, shell=True, capture_output=True, text=True)",
                    recomendacion="Evita shell=True y usa listas de argumentos validados. Valida user_id y output_format antes de ejecutar cualquier comando.",
                    confianza=0.95,
                )
            ],
            resumen_ejecutivo="El commit introduce ejecución de comandos con entrada controlada por el usuario.",
            puntuacion_riesgo=9.5,
        )
    return AuditLLMOutput(
        vulnerabilidades=[],
        resumen_ejecutivo="No se detectaron vulnerabilidades en la simulación.",
        puntuacion_riesgo=0.0,
    )
```

**✅ Validación del paso:**

```bash
python -m py_compile fake_auditor.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Prueba la auditoría simulada con el commit más reciente

**📝 Descripción del paso:**  
Vas a extraer el último commit y auditarlo con la función falsa.

**⚙️ Contenido del paso:**

```bash
python -c "
from fake_auditor import fake_audit_diff
from git_extractor import get_commit_diffs
commit = get_commit_diffs('./vulnerable_repo', n_commits=1)[0]
resultado = fake_audit_diff(commit.diff_texto)
print(resultado.model_dump_json(indent=2))
"
```

**✅ Validación del paso:**  
Debes ver un JSON con una vulnerabilidad de tipo `Command Injection`.

**📌 Resultado esperado:**  
El flujo local de extracción y validación funciona sin consumir API.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%207%20de%20un%20laboratorio%20donde%20cre%C3%A9%20una%20auditor%C3%ADa%20simulada%20sin%20consumir%20API%20para%20validar%20el%20flujo%20de%20extracci%C3%B3n%20de%20diffs%2C%20modelos%20Pydantic%20y%20hallazgos%20de%20seguridad.)

---

# 🧩 Tarea 8. Implementar el motor de auditoría con OpenAI y salida estructurada

## 🎯 Objetivo de la tarea

Crear el motor que llama al LLM, fuerza una salida estructurada validada por Pydantic y maneja errores transitorios con reintentos.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `auditor_engine.py`

**📝 Descripción del paso:**  
Vas a implementar la llamada real al modelo. El resultado se convertirá directamente en un objeto `AuditLLMOutput` validado.

**⚙️ Contenido del paso:**

```python
import logging
import os
from dotenv import load_dotenv
from openai import APIConnectionError, APIStatusError, APITimeoutError, OpenAI, RateLimitError
from tenacity import before_sleep_log, retry, retry_if_exception_type, stop_after_attempt, wait_random_exponential
from models import AuditLLMOutput, AuditReport, CommitDiff
from prompt_builder import build_security_prompt, get_system_prompt

load_dotenv()
logger = logging.getLogger(__name__)

class AuditorConfigurationError(RuntimeError):
    pass

def get_openai_client() -> OpenAI:
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key or api_key.startswith("pega_aqui"):
        raise AuditorConfigurationError("OPENAI_API_KEY no está configurada correctamente en .env")
    return OpenAI(api_key=api_key)

@retry(
    wait=wait_random_exponential(multiplier=1, min=2, max=30),
    stop=stop_after_attempt(4),
    retry=retry_if_exception_type((RateLimitError, APIStatusError, APITimeoutError, APIConnectionError)),
    before_sleep=before_sleep_log(logger, logging.WARNING),
    reraise=True,
)
def call_openai_structured(system_prompt: str, user_prompt: str) -> AuditLLMOutput:
    client = get_openai_client()
    model = os.getenv("OPENAI_MODEL", "gpt-4o-mini")
    response = client.beta.chat.completions.parse(
        model=model,
        temperature=0.1,
        max_tokens=1800,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt},
        ],
        response_format=AuditLLMOutput,
    )
    parsed = response.choices[0].message.parsed
    if parsed is None:
        raise RuntimeError("El modelo no devolvió una salida estructurada parseable.")
    return parsed

def audit_commit(commit_diff: CommitDiff) -> AuditReport:
    logger.info("Auditando commit %s", commit_diff.hash_corto)
    system_prompt = get_system_prompt()
    user_prompt = build_security_prompt(commit_diff.diff_texto, commit_diff.archivos_modificados)
    model = os.getenv("OPENAI_MODEL", "gpt-4o-mini")
    try:
        output = call_openai_structured(system_prompt, user_prompt)
        return AuditReport(
            commit=commit_diff,
            vulnerabilidades=output.vulnerabilidades,
            resumen_ejecutivo=output.resumen_ejecutivo,
            puntuacion_riesgo=output.puntuacion_riesgo,
            llm_proveedor="openai",
            llm_modelo=model,
        )
    except Exception as exc:
        logger.error("Error auditando commit %s: %s", commit_diff.hash_corto, exc)
        return AuditReport(
            commit=commit_diff,
            vulnerabilidades=[],
            resumen_ejecutivo="La auditoría falló para este commit.",
            puntuacion_riesgo=0.0,
            llm_proveedor="openai",
            llm_modelo=model,
            error_auditoria=str(exc),
        )
```

**✅ Validación del paso:**

```bash
python -m py_compile auditor_engine.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Prueba una auditoría real de un solo commit

**📝 Descripción del paso:**  
Vas a probar el motor con el commit más reciente antes de auditar todo el repositorio.

**⚙️ Contenido del paso:**

```bash
python -c "
import logging
from auditor_engine import audit_commit
from git_extractor import get_commit_diffs
logging.basicConfig(level=logging.INFO)
commit = get_commit_diffs('./vulnerable_repo', n_commits=1)[0]
report = audit_commit(commit)
print(report.model_dump_json(indent=2))
"
```

**✅ Validación del paso:**  
Debes obtener un reporte JSON con `commit`, `vulnerabilidades`, `resumen_ejecutivo` y `puntuacion_riesgo`.

**📌 Resultado esperado:**  
El modelo detecta al menos una vulnerabilidad relacionada con Command Injection en el commit más reciente.

> [!WARNING]
> Si no detecta la vulnerabilidad, no significa automáticamente que el código esté bien. Registra el caso como posible falso negativo.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%208%20de%20un%20laboratorio%20donde%20implement%C3%A9%20un%20motor%20de%20auditor%C3%ADa%20con%20OpenAI%2C%20salida%20estructurada%20con%20Pydantic%2C%20temperatura%20baja%20y%20reintentos%20con%20backoff%20exponencial.)

---

# 🧩 Tarea 9. Generar un reporte Markdown profesional

## 🎯 Objetivo de la tarea

Crear un generador de reportes que consolide las auditorías por commit, muestre métricas globales y documente limitaciones.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `report_generator.py`

**📝 Descripción del paso:**  
Vas a transformar los objetos `AuditReport` en un documento Markdown legible.

**⚙️ Contenido del paso:**

````python
from datetime import datetime, timezone
from models import AuditReport, Severidad


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


def md_text(valor) -> str:
    """
    Escapa texto para celdas de tabla Markdown.
    Evita que pipes, saltos de línea o valores None rompan la tabla.
    """
    if valor is None:
        return "N/A"

    texto = str(valor)
    texto = texto.replace("|", r"\|")
    texto = texto.replace("\r", "")
    texto = texto.replace("\n", "<br>")
    return texto


def md_code_inline(valor) -> str:
    """
    Formatea texto como código inline seguro.
    """
    if valor is None or valor == "":
        return "`N/A`"

    texto = str(valor)
    texto = texto.replace("`", r"\`")
    return f"`{texto}`"


def fenced_code(codigo: str, lenguaje: str = "python") -> str:
    """
    Genera un bloque de código Markdown seguro para el reporte.
    Usa ~~~ en lugar de ``` para evitar conflictos al documentar este archivo en README.md.
    """
    if not codigo:
        return ""

    return f"~~~{lenguaje}\n{codigo.rstrip()}\n~~~"


def generate_markdown_report(audits: list[AuditReport]) -> str:
    ahora = datetime.now(tz=timezone.utc).strftime("%Y-%m-%d %H:%M:%S UTC")

    partes: list[str] = []

    total_vulnerabilidades = sum(len(a.vulnerabilidades) for a in audits)
    max_riesgo = max((a.puntuacion_riesgo for a in audits), default=0.0)
    errores = [a for a in audits if a.error_auditoria]

    conteo = {s: 0 for s in Severidad}

    for audit in audits:
        for vuln in audit.vulnerabilidades:
            conteo[vuln.severidad] += 1

    partes.append(
        f"""# 🔐 Reporte de Auditoría de Seguridad con LLM

**Generado:** {ahora}  
**Commits analizados:** {len(audits)}  
**Total de vulnerabilidades:** {total_vulnerabilidades}  
**Riesgo máximo:** {max_riesgo:.1f} / 10.0  
**Errores de auditoría:** {len(errores)}

---

## 📊 Resumen por severidad

| Severidad | Cantidad |
|---|---:|
"""
    )

    for severidad in ORDEN_SEVERIDAD:
        partes.append(
            f"| {ICONOS_SEVERIDAD[severidad]} {md_text(severidad.value)} | {conteo[severidad]} |\n"
        )

    partes.append(
        """

---

## 🗂️ Tabla consolidada de vulnerabilidades

"""
    )

    filas = [
        (audit, vuln)
        for audit in audits
        for vuln in audit.vulnerabilidades
    ]

    if filas:
        partes.append(
            "| Commit | Archivo | Tipo | OWASP | Severidad | Confianza |\n"
            "|---|---|---|---|---|---:|\n"
        )

        for audit, vuln in filas:
            icono = ICONOS_SEVERIDAD.get(vuln.severidad, "⚪")

            partes.append(
                f"| {md_code_inline(audit.commit.hash_corto)} "
                f"| {md_code_inline(vuln.archivo or 'N/A')} "
                f"| {md_text(vuln.tipo)} "
                f"| {md_text(vuln.categoria_owasp.value)} "
                f"| {icono} {md_text(vuln.severidad.value)} "
                f"| {vuln.confianza:.2f} |\n"
            )
    else:
        partes.append("No se encontraron vulnerabilidades.\n")

    partes.append(
        """

---

## 🔍 Análisis detallado por commit

"""
    )

    for audit in audits:
        commit = audit.commit

        if audit.puntuacion_riesgo >= 7:
            riesgo_icono = "🔴"
        elif audit.puntuacion_riesgo >= 4:
            riesgo_icono = "🟠"
        else:
            riesgo_icono = "🟢"

        archivos_modificados = ", ".join(
            md_code_inline(archivo)
            for archivo in commit.archivos_modificados
        )

        partes.append(
            f"""### {riesgo_icono} Commit {md_code_inline(commit.hash_corto)} — {md_text(commit.mensaje)}

| Campo | Valor |
|---|---|
| Hash completo | {md_code_inline(commit.commit_hash)} |
| Autor | {md_text(commit.autor)} ({md_text(commit.email_autor)}) |
| Fecha | {commit.timestamp.strftime('%Y-%m-%d %H:%M:%S UTC')} |
| Archivos modificados | {archivos_modificados or "N/A"} |
| Puntuación de riesgo | {audit.puntuacion_riesgo:.1f} / 10.0 |
| Vulnerabilidades | {len(audit.vulnerabilidades)} |

"""
        )

        if audit.error_auditoria:
            partes.append(
                f"> ⚠️ Error de auditoría: {md_code_inline(audit.error_auditoria)}\n\n"
            )
            continue

        if audit.resumen_ejecutivo:
            partes.append(
                f"**Resumen ejecutivo:** {audit.resumen_ejecutivo}\n\n"
            )

        if not audit.vulnerabilidades:
            partes.append(
                "✅ No se detectaron vulnerabilidades en este commit.\n\n"
            )

        for i, vuln in enumerate(audit.vulnerabilidades, start=1):
            icono = ICONOS_SEVERIDAD.get(vuln.severidad, "⚪")

            partes.append(
                f"""#### {icono} Vulnerabilidad {i}: {md_text(vuln.tipo)}

- **Archivo:** {md_code_inline(vuln.archivo or "No especificado")}
- **Severidad:** {md_text(vuln.severidad.value)}
- **Categoría OWASP:** {md_text(vuln.categoria_owasp.value)}
- **Línea afectada:** {md_text(vuln.linea_afectada or "No especificada")}
- **Confianza:** {vuln.confianza:.2f}

**Descripción:**  
{vuln.descripcion}

"""
            )

            if vuln.codigo_vulnerable:
                partes.append(
                    "**Código vulnerable:**\n\n"
                    f"{fenced_code(vuln.codigo_vulnerable, 'python')}\n\n"
                )

            if vuln.recomendacion:
                partes.append(
                    f"""**Recomendación:**  
{vuln.recomendacion}

"""
                )

    partes.append(
        """---

## ⚠️ Limitaciones de la auditoría con LLM

Este reporte fue generado por IA generativa y debe ser revisado por una persona con criterio técnico.

| Limitación | Riesgo | Mitigación |
|---|---|---|
| Falsos positivos | El modelo puede reportar hallazgos que no son explotables en el contexto real | Revisar manualmente cada hallazgo |
| Falsos negativos | El modelo puede omitir vulnerabilidades reales | Complementar con Bandit, Semgrep y revisión humana |
| Contexto limitado | El análisis por commit no entiende toda la arquitectura | Revisar módulos completos y flujos críticos |
| Variabilidad del modelo | Dos ejecuciones pueden producir diferencias | Usar temperatura baja y salida estructurada |
| Costos y límites | Auditar muchos commits puede consumir saldo | Limitar commits, truncar diffs y auditar por lotes |

---

## 🧰 Herramientas complementarias sugeridas

**Bandit**

~~~bash
pip install bandit
bandit -r vulnerable_repo
~~~

**Semgrep**

~~~bash
semgrep --config=p/python vulnerable_repo
~~~

**Compilación del generador**

~~~bash
python -m py_compile report_generator.py
~~~

---

## ✅ Cierre del reporte

Usa este reporte como punto de partida para priorizar remediaciones, no como veredicto final de seguridad.
"""
    )

    return "".join(partes).strip()
````

**✅ Validación del paso:**

```bash
python -m py_compile report_generator.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Prueba el generador con datos simulados

**📝 Descripción del paso:**  
Vas a crear un reporte mínimo para confirmar que el generador produce Markdown válido.

**⚙️ Contenido del paso:**

```bash
python -c "
from datetime import datetime, timezone
from models import AuditReport, CommitDiff
from report_generator import generate_markdown_report
commit = CommitDiff(commit_hash='abc123', hash_corto='abc123', autor='Lab Student', email_autor='lab@example.com', mensaje='test commit', timestamp=datetime.now(tz=timezone.utc), diff_texto='+ print(\"test\")', archivos_modificados=['test.py'])
report = AuditReport(commit=commit, resumen_ejecutivo='Prueba local', puntuacion_riesgo=0.0)
markdown = generate_markdown_report([report])
print(markdown[:800])
"
```

**✅ Validación del paso:**  
El texto debe iniciar con `# 🔐 Reporte de Auditoría de Seguridad con LLM`.

**📌 Resultado esperado:**  
El generador de reportes funciona correctamente.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 9 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%209%20de%20un%20laboratorio%20donde%20cre%C3%A9%20un%20generador%20de%20reportes%20Markdown%20para%20auditor%C3%ADas%20de%20seguridad%20con%20LLM%2C%20incluyendo%20resumen%20por%20severidad%2C%20tabla%20de%20vulnerabilidades%2C%20detalle%20por%20commit%20y%20limitaciones.)

---

# 🧩 Tarea 10. Ensamblar el pipeline principal

## 🎯 Objetivo de la tarea

Crear el script principal que orquesta extracción de commits, auditoría con LLM y generación de reporte Markdown.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `security_auditor.py`

**📝 Descripción del paso:**  
Vas a construir el punto de entrada del laboratorio.

**⚙️ Contenido del paso:**

```python
import argparse
import logging
import os
import sys
import time
from pathlib import Path
from dotenv import load_dotenv
from auditor_engine import audit_commit
from git_extractor import get_commit_diffs
from report_generator import generate_markdown_report

load_dotenv()
logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(name)s: %(message)s", handlers=[logging.StreamHandler(sys.stdout)])
logger = logging.getLogger("security_auditor")

def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(description="Pipeline de auditoría de seguridad con LLM para commits de Git")
    parser.add_argument("--repo", default="./vulnerable_repo", help="Ruta al repositorio Git que quieres auditar.")
    parser.add_argument("--commits", type=int, default=int(os.getenv("MAX_COMMITS_DEFAULT", "3")), help="Número de commits a auditar desde el más reciente.")
    parser.add_argument("--output", default="", help="Ruta del reporte Markdown de salida.")
    return parser.parse_args()

def main() -> int:
    args = parse_args()
    print("\n" + "═" * 70)
    print("  🔐 Pipeline de Auditoría de Seguridad con LLM")
    print("═" * 70)
    print(f"  Repositorio : {args.repo}")
    print(f"  Commits     : {args.commits}")
    print("═" * 70 + "\n")
    try:
        commit_diffs = get_commit_diffs(args.repo, n_commits=args.commits)
    except Exception as exc:
        logger.error("No se pudieron extraer commits: %s", exc)
        return 1
    if not commit_diffs:
        logger.warning("No se encontraron commits para auditar.")
        return 0
    print(f"✅ Commits extraídos: {len(commit_diffs)}\n")
    audit_reports = []
    for index, commit_diff in enumerate(commit_diffs, start=1):
        print(f"🔍 [{index}/{len(commit_diffs)}] Auditando commit `{commit_diff.hash_corto}`: {commit_diff.mensaje}")
        inicio = time.perf_counter()
        report = audit_commit(commit_diff)
        elapsed = time.perf_counter() - inicio
        audit_reports.append(report)
        if report.error_auditoria:
            print(f"   ⚠️ Error: {report.error_auditoria[:120]}")
        else:
            icono = "🔴" if report.puntuacion_riesgo >= 7 else "🟠" if report.puntuacion_riesgo >= 4 else "🟢"
            print(f"   {icono} Riesgo: {report.puntuacion_riesgo:.1f}/10 | Vulnerabilidades: {len(report.vulnerabilidades)} | Tiempo: {elapsed:.1f}s")
        if index < len(commit_diffs):
            time.sleep(1)
    markdown = generate_markdown_report(audit_reports)
    if args.output:
        output_path = Path(args.output)
    else:
        output_dir = Path("reports")
        output_dir.mkdir(exist_ok=True)
        timestamp = time.strftime("%Y%m%d_%H%M%S")
        output_path = output_dir / f"audit_{timestamp}.md"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(markdown, encoding="utf-8")
    total_vulns = sum(len(report.vulnerabilidades) for report in audit_reports)
    print("\n" + "═" * 70)
    print("  ✅ Auditoría completada")
    print("═" * 70)
    print(f"  Commits auditados      : {len(audit_reports)}")
    print(f"  Vulnerabilidades       : {total_vulns}")
    print(f"  Reporte generado       : {output_path.resolve()}")
    print("═" * 70 + "\n")
    return 0

if __name__ == "__main__":
    raise SystemExit(main())
```

**✅ Validación del paso:**

```bash
python -m py_compile security_auditor.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Ejecuta la auditoría de 1 commit

**📝 Descripción del paso:**  
Vas a probar una ejecución pequeña para controlar costo y validar funcionamiento.

**⚙️ Contenido del paso:**

```bash
python security_auditor.py --repo ./vulnerable_repo --commits 1 --output reports/audit_1_commit.md
```

**✅ Validación del paso:**

```bash
ls reports
```

**📌 Resultado esperado:**  
Existe el archivo `audit_1_commit.md`.

---

### ✅ Paso 3. Abre el reporte en VS Code

**📝 Descripción del paso:**  
Vas a revisar visualmente el reporte generado.

**⚙️ Contenido del paso:**

```bash
code reports/audit_1_commit.md
```

En VS Code, abre la vista previa de Markdown con `Ctrl + Shift + V`.

**✅ Validación del paso:**  
El reporte debe mostrar resumen, tabla de severidad y análisis por commit.

**📌 Resultado esperado:**  
Puedes leer el reporte Markdown de forma clara.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 10 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2010%20de%20un%20laboratorio%20donde%20ensambl%C3%A9%20un%20pipeline%20principal%20que%20extrae%20commits%20de%20Git%2C%20los%20audita%20con%20un%20LLM%20y%20genera%20un%20reporte%20Markdown%20de%20seguridad.)

---

# 🧩 Tarea 11. Ejecutar la auditoría completa y evaluar resultados

## 🎯 Objetivo de la tarea

Auditar los 5 commits vulnerables, revisar el reporte generado y comparar hallazgos esperados contra hallazgos detectados por el LLM.

---

## 🛠️ Pasos

### ✅ Paso 1. Ejecuta la auditoría completa

**📝 Descripción del paso:**  
Vas a analizar los 5 commits del repositorio vulnerable.

**⚙️ Contenido del paso:**

```bash
python security_auditor.py --repo ./vulnerable_repo --commits 5 --output reports/audit_completa.md
```

**✅ Validación del paso:**

```bash
test -f reports/audit_completa.md && echo "Reporte generado correctamente"
```

**📌 Resultado esperado:**

```text
Reporte generado correctamente
```

---

### ✅ Paso 2. Verifica la estructura del reporte

**📝 Descripción del paso:**  
Vas a comprobar que el reporte contiene las secciones esperadas.

**⚙️ Contenido del paso:**

```bash
python -c "
from pathlib import Path
reporte = Path('reports/audit_completa.md').read_text(encoding='utf-8')
checks = ['# 🔐 Reporte de Auditoría de Seguridad con LLM', '## 📊 Resumen por severidad', '## 🗂️ Tabla consolidada de vulnerabilidades', '## 🔍 Análisis detallado por commit', '## ⚠️ Limitaciones de la auditoría con LLM']
for check in checks:
    print(('✅' if check in reporte else '❌'), check)
"
```

**✅ Validación del paso:**  
Todas las líneas deben iniciar con ✅.

**📌 Resultado esperado:**  
La estructura del reporte es correcta.

---

### ✅ Paso 3. Crea una tabla de evaluación manual

**📝 Descripción del paso:**  
Vas a documentar si el LLM detectó las vulnerabilidades esperadas.

**⚙️ Contenido del paso:**  
Crea el archivo `evaluacion_hallazgos.md` y complétalo con base en `reports/audit_completa.md`:

```markdown
# Evaluación de hallazgos esperados vs. detectados

| Commit / Archivo | Vulnerabilidad esperada | ¿Detectada por el LLM? | Severidad reportada | Comentario |
|---|---|---|---|---|
| db_utils.py | SQL Injection | Sí/No/Parcial | — | — |
| config.py | Credenciales hardcodeadas | Sí/No/Parcial | — | — |
| file_handler.py | Path Traversal | Sí/No/Parcial | — | — |
| session_manager.py | Insecure Deserialization | Sí/No/Parcial | — | — |
| api_handlers.py | Command Injection | Sí/No/Parcial | — | — |

## Observaciones

- Posibles falsos positivos:
- Posibles falsos negativos:
- Hallazgos mejor explicados:
- Hallazgos que requieren revisión humana:
- Mejoras sugeridas al prompt:
```

**✅ Validación del paso:**  
La tabla debe tener una fila por cada vulnerabilidad esperada.

**📌 Resultado esperado:**  
Tienes una evaluación crítica, no solo un reporte automático.

---

### ✅ Paso 4. Busca hallazgos clave en el reporte

**📝 Descripción del paso:**  
Vas a usar búsquedas simples para ubicar vulnerabilidades específicas.

**⚙️ Contenido del paso:**

```bash
grep -i "SQL Injection\|Command Injection\|Path Traversal\|pickle\|credential\|credencial" reports/audit_completa.md
```

**✅ Validación del paso:**  
El comando debe mostrar líneas relacionadas con varias vulnerabilidades.

**📌 Resultado esperado:**  
Puedes confirmar de forma rápida si el LLM reportó hallazgos clave.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 11 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2011%20de%20un%20laboratorio%20donde%20ejecut%C3%A9%20una%20auditor%C3%ADa%20completa%20de%205%20commits%20vulnerables%2C%20revis%C3%A9%20el%20reporte%20Markdown%20y%20compar%C3%A9%20hallazgos%20esperados%20contra%20hallazgos%20detectados%20por%20el%20LLM.)

---

# 🧩 Tarea 12. Validar funcionamiento y preparar entrega

## 🎯 Objetivo de la tarea

Confirmar que todos los scripts compilan, que el reporte existe, que las evidencias son seguras y que no se entrega ningún secreto.

---

## 🛠️ Pasos

### ✅ Paso 1. Valida que todos los scripts compilen

**📝 Descripción del paso:**  
Vas a verificar que los archivos Python no tengan errores de sintaxis.

**⚙️ Contenido del paso:**

```bash
python -m py_compile 00_validar_entorno.py
python -m py_compile models.py
python -m py_compile git_extractor.py
python -m py_compile prompt_builder.py
python -m py_compile fake_auditor.py
python -m py_compile auditor_engine.py
python -m py_compile report_generator.py
python -m py_compile security_auditor.py
```

**✅ Validación del paso:**  
Ningún comando debe mostrar errores.

**📌 Resultado esperado:**  
Todos los scripts tienen sintaxis válida.

---

### ✅ Paso 2. Valida las variables de entorno

**📝 Descripción del paso:**  
Vas a comprobar que el ambiente sigue cargando la configuración.

**⚙️ Contenido del paso:**

```bash
python 00_validar_entorno.py
```

**✅ Validación del paso:**  
Debe mostrarse que la API Key está configurada sin imprimirla completa.

**📌 Resultado esperado:**  
El ambiente está listo para ejecutar el pipeline.

---

### ✅ Paso 3. Valida que no entregas `.env`

**📝 Descripción del paso:**  
Vas a confirmar que no incluyes secretos en la entrega.

**⚙️ Contenido del paso:**

```bash
ls -la
```

Entrega estos archivos:

```text
requirements.txt
00_validar_entorno.py
models.py
git_extractor.py
prompt_builder.py
fake_auditor.py
auditor_engine.py
report_generator.py
security_auditor.py
reports/audit_completa.md
evaluacion_hallazgos.md
```

No entregues:

```text
.env
.venv/
```

**✅ Validación del paso:**  
Confirma manualmente que `.env` no se incluye en la entrega.

**📌 Resultado esperado:**  
La entrega es funcional y segura.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 12 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2012%20de%20un%20laboratorio%20donde%20valid%C3%A9%20que%20todos%20los%20scripts%20compilan%2C%20que%20el%20pipeline%20funciona%2C%20que%20el%20reporte%20Markdown%20existe%20y%20que%20no%20entrego%20archivos%20sensibles%20como%20.env.)

---

# 🧪 Reto opcional. Analizar cambios staged como pre-commit

> [!WARNING]
> Este reto es opcional. Un hook `pre-commit` analiza cambios antes de crear el commit, por lo que no debe analizar “el último commit existente”. Debe analizar el contenido staged con `git diff --cached`.

## 🎯 Objetivo del reto

Extender el pipeline para analizar cambios staged antes de crear un commit.

## 🛠️ Idea de implementación

1. Crear una función nueva que ejecute:

```bash
git diff --cached
```

2. Convertir ese diff en un objeto similar a `CommitDiff`.
3. Ejecutar `audit_commit()` o una función equivalente sobre ese diff.
4. Generar un reporte en `reports/pre_commit_audit.md`.
5. Decidir si el hook solo avisa o bloquea el commit.

## ✅ Criterio de éxito

El hook debe auditar cambios staged y no commits existentes.

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Variables de entorno configuradas de forma segura.
4. Repositorio Git vulnerable de ejemplo con 5 commits.
5. Modelos Pydantic para commits, vulnerabilidades y reportes.
6. Extractor de diffs con GitPython.
7. Constructor de prompts de seguridad.
8. Auditor simulado sin consumo de API.
9. Motor real de auditoría con OpenAI y salida estructurada.
10. Generador de reporte Markdown.
11. Script principal `security_auditor.py`.
12. Reporte `reports/audit_completa.md`.
13. Archivo `evaluacion_hallazgos.md` con análisis de precisión.
14. Identificación de falsos positivos y falsos negativos.
15. Conclusión sobre el valor y límites del LLM para revisión de código.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del ambiente local | 10% |
| Configuración segura de credenciales | 10% |
| Creación correcta del repositorio vulnerable | 10% |
| Modelado Pydantic correcto | 10% |
| Extracción de commits y diffs | 10% |
| Prompt de auditoría de seguridad | 10% |
| Motor LLM con salida estructurada y reintentos | 15% |
| Reporte Markdown generado | 10% |
| Evaluación de hallazgos esperados vs. detectados | 10% |
| Reflexión sobre limitaciones del LLM | 5% |
| Total | 100% |

---

# ⚠️ Errores comunes que debes evitar

1. Pegar la API Key directamente en código Python.
2. Entregar el archivo `.env`.
3. Ejecutar la auditoría completa antes de probar 1 commit.
4. Confundir commits existentes con cambios staged.
5. Confiar ciegamente en el LLM sin revisión humana.
6. Considerar que un reporte del LLM equivale a una auditoría profesional.
7. No documentar falsos positivos y falsos negativos.
8. Usar un modelo no disponible en tu cuenta.
9. Olvidar activar el entorno virtual.
10. Auditar repositorios reales con secretos productivos.
11. No revisar costos ni límites antes de ejecutar varias pruebas.
12. Omitir el reporte de evaluación manual.

---

# 🧹 Limpieza del entorno

Cuando termines la práctica, puedes limpiar archivos temporales.

```bash
# Desactivar entorno virtual
deactivate

# Opcional: eliminar reportes generados
rm -rf reports/

# Opcional: eliminar entorno virtual
rm -rf .venv/

# Opcional: eliminar repositorio vulnerable de ejemplo
rm -rf vulnerable_repo/
```

> [!IMPORTANT]
> Si subiste accidentalmente `.env` a un repositorio remoto, rota inmediatamente tu API Key desde la consola del proveedor.

---

# Cierre de la práctica

En este laboratorio construiste un pipeline completo de auditoría de seguridad asistido por IA generativa. Preparaste un ambiente local en Windows, configuraste credenciales de forma segura, creaste un repositorio Git vulnerable, extrajiste diffs de commits, diseñaste modelos Pydantic, construiste un prompt especializado, ejecutaste auditorías con salida estructurada y generaste un reporte Markdown profesional.

El aprendizaje principal es que un LLM puede aportar valor como copiloto de revisión de código, especialmente para explicar riesgos y proponer recomendaciones. Sin embargo, también aprendiste que sus resultados deben validarse, porque puede cometer errores, omitir hallazgos o reportar vulnerabilidades fuera de contexto.

La práctica te deja una base sólida para evolucionar hacia un flujo DevSecOps más completo, combinando LLMs con herramientas SAST, revisión humana, políticas de seguridad y automatización controlada.
