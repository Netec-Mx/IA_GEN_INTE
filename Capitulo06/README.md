<div align="center">

# 🧪 Laboratorio 6

## Memoria semántica conversacional con SQLite, ChromaDB y OpenAI

![Nivel](https://img.shields.io/badge/Nivel-Intermedio%20Alto-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)
![VectorDB](https://img.shields.io/badge/VectorDB-ChromaDB-DB2777?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio vas a construir un chatbot CLI con **memoria híbrida**: una ventana deslizante para el contexto reciente y una memoria semántica persistente con **ChromaDB** para recuperar información mencionada muchos turnos atrás. No uses datos personales reales, credenciales, información confidencial ni conversaciones sensibles.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Memoria semántica conversacional</td>
<td width="25%"><strong>⏱️ Duración</strong><br>50 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar, evaluar y crear</td>
<td width="25%"><strong>📦 Entregable</strong><br>Chatbot CLI + SQLite + ChromaDB</td>
</tr>
</table>

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a construir una aplicación de consola llamada `semantic_memory_chat.py`. La aplicación implementa una arquitectura de memoria híbrida para asistentes de IA generativa.

La memoria híbrida combina dos mecanismos:

1. **Ventana deslizante:** conserva los últimos mensajes de la conversación para mantener continuidad inmediata.
2. **Memoria semántica:** guarda mensajes anteriores como embeddings en ChromaDB y recupera fragmentos relevantes cuando el usuario hace una nueva pregunta.

También usarás **SQLite** como capa de logging estructurado. SQLite guardará sesiones, mensajes, timestamps, roles, conteo aproximado de tokens y posición del turno. ChromaDB guardará los textos vectorizados para búsqueda semántica.

La práctica original propone persistir mensajes simultáneamente en SQLite y ChromaDB, recuperar contexto por similitud semántica y demostrar la ventaja de esta memoria sobre una ventana fija de conversación. Esta versión ajustada mantiene esa arquitectura, pero la organiza de forma progresiva, profesional y adecuada para Windows, Visual Studio Code y Git Bash.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Preparar un entorno local en Windows para una aplicación GenAI con memoria persistente.
2. Configurar variables de entorno de forma segura con `.env` y `.gitignore`.
3. Diseñar una base SQLite para registrar sesiones y mensajes.
4. Crear una colección ChromaDB persistente para memoria semántica.
5. Implementar escritura dual: SQLite para trazabilidad y ChromaDB para búsqueda vectorial.
6. Recuperar los últimos mensajes con una ventana deslizante.
7. Recuperar mensajes antiguos relevantes usando similitud semántica.
8. Excluir la ventana reciente de la búsqueda semántica para evitar duplicar contexto.
9. Construir prompts enriquecidos con system prompt, memoria semántica y ventana reciente.
10. Crear un chatbot CLI con comandos `/memory`, `/stats`, `/new` y `/quit`.
11. Ejecutar una demo comparativa para comprobar cuándo la memoria semántica aporta valor.
12. Evaluar limitaciones, costos, riesgos y posibles mejoras del patrón de memoria conversacional.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Haber completado o comprendido el laboratorio anterior sobre RAG y ChromaDB.
2. Tener conocimientos básicos de Python.
3. Conocer el formato de mensajes `system`, `user` y `assistant`.
4. Comprender qué son embeddings y búsqueda semántica.
5. Saber usar comandos básicos de Git Bash.
6. Tener instalado Visual Studio Code.
7. Tener instalado Python 3.11 o superior.
8. Tener una API key válida de OpenAI.
9. Tener conexión a internet.
10. Comprender que la práctica genera consumo bajo de API.

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
| Internet | 10 Mbps | 25 Mbps o más |

---

### 🧰 Software

| Software / Paquete | Uso |
|---|---|
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Python 3.11 o superior | Runtime de la aplicación |
| pip | Instalación de dependencias |
| `openai` | Cliente para embeddings y generación de texto |
| `chromadb` | Base vectorial local persistente |
| `python-dotenv` | Carga segura de variables de entorno |
| `tenacity` | Reintentos con backoff exponencial |
| `rich` | Salida visual opcional en terminal |
| `sqlite3` | Base de datos relacional local incluida con Python |

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
| Base relacional | SQLite |
| Base vectorial | ChromaDB |
| Modelo de embeddings | `text-embedding-3-small` |
| Modelo conversacional | Configurable mediante `.env` |
| Costo estimado | Bajo, aproximadamente $0.10–$0.30 USD según ejecuciones |
| Entregable principal | Chatbot CLI con memoria semántica persistente |
| Entregable secundario | Script de demo y tabla de evaluación |

---

## 🛡️ Consideraciones para estudiantes

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>No compartas claves ni subas `.env`.</td>
<td><strong>💸 Costo</strong><br>Cada mensaje puede consumir tokens y embeddings.</td>
<td><strong>🧠 Memoria</strong><br>No guardes información sensible real.</td>
</tr>
</table>

1. No escribas tu API key dentro del código. Usa siempre `.env`.
2. No entregues el archivo `.env`.
3. No uses datos personales reales. El sistema persiste conversaciones en SQLite y ChromaDB.
4. La memoria semántica no es perfecta: puede recuperar fragmentos parcialmente relacionados.
5. La ventana deslizante conserva continuidad inmediata; la memoria semántica recupera contexto histórico.
6. Las respuestas del asistente pueden guardarse en memoria para continuidad, pero también pueden introducir ruido.
7. Ejecuta primero las pruebas cortas antes de correr la demo.
8. ChromaDB local es ideal para aprendizaje y prototipos; para producción evalúa respaldo, seguridad y operación.
9. SQLite es suficiente para trazabilidad local; en entornos empresariales podrías usar PostgreSQL u otro motor.
10. No confundas memoria con verdad: siempre valida la respuesta.

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear la carpeta del laboratorio, abrirla en Visual Studio Code, crear el entorno virtual e instalar las dependencias necesarias.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear una carpeta nueva para este laboratorio. Esta carpeta será la raíz del proyecto y ahí guardarás todos los archivos que construirás durante la práctica: scripts Python, dependencias, configuración, base SQLite y datos persistentes de ChromaDB. Ejecuta estos comandos desde Git Bash; no necesitas crear archivos manualmente en este paso.

**⚙️ Contenido del paso:**

```bash
mkdir -p ~/labs-ia-gen/lab-06-semantic-memory
cd ~/labs-ia-gen/lab-06-semantic-memory
```

**✅ Validación del paso:**

```bash
pwd
```

**📌 Resultado esperado:**

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-06-semantic-memory
```

---

### ✅ Paso 2. Abre el proyecto en Visual Studio Code

**📝 Descripción del paso:**  
Vas a abrir en Visual Studio Code la carpeta `lab-06-semantic-memory` que acabas de crear. A partir de este punto, todos los archivos nuevos del laboratorio deben crearse dentro de esta carpeta. Puedes abrirla con `code .` desde Git Bash o manualmente desde el menú de VS Code.

**⚙️ Contenido del paso:**

```bash
code .
```

Si `code .` no funciona, abre VS Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-06-semantic-memory
```

**✅ Validación del paso:**  
Confirma que VS Code muestre la carpeta `lab-06-semantic-memory`.

**📌 Resultado esperado:**  
El proyecto está abierto en Visual Studio Code.

---

### ✅ Paso 3. Crea el entorno virtual

**📝 Descripción del paso:**  
Vas a crear un entorno virtual llamado `.venv` dentro de la carpeta del laboratorio y después lo vas a activar. Este entorno permite instalar las librerías de la práctica sin afectar otros proyectos de Python que tengas en tu equipo. Ejecuta los comandos desde Git Bash en la raíz del proyecto.

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
Vas a crear el archivo `requirements.txt` en la raíz del proyecto. Este archivo contendrá la lista de paquetes Python que necesita el laboratorio para conectarse con OpenAI, usar ChromaDB, cargar variables de entorno, aplicar reintentos y mejorar la salida en terminal. No edites otro archivo en este paso.

**⚙️ Contenido del paso:**

```bash
cat > requirements.txt << 'REQ'
openai>=1.90,<2
chromadb>=0.5,<1
python-dotenv>=1.0,<2
tenacity>=8.5,<10
rich>=13,<15
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
Vas a instalar dentro del entorno virtual activo todas las dependencias declaradas en `requirements.txt`. Antes de ejecutar este paso, confirma que Git Bash muestre `(.venv)` al inicio de la línea; eso indica que las librerías se instalarán en el entorno del laboratorio y no en Python global.

**⚙️ Contenido del paso:**

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**✅ Validación del paso:**

```bash
python -c "import openai, chromadb, dotenv, tenacity; print('Dependencias instaladas correctamente')"
```

**📌 Resultado esperado:**

```text
Dependencias instaladas correctamente
```

---

### ✅ Paso 6. Crea `.env` y `.gitignore`

**📝 Descripción del paso:**  
Vas a crear dos archivos en la raíz del proyecto: `.env` y `.gitignore`. En `.env` guardarás la configuración local del laboratorio, incluyendo tu API key y los modelos que usará la aplicación. En `.gitignore` indicarás qué archivos no deben compartirse ni subirse a un repositorio, como credenciales, entorno virtual, base SQLite y carpeta ChromaDB.

**⚙️ Contenido del paso:**

```bash
cat > .env << 'ENV'
OPENAI_API_KEY=pega_aqui_tu_api_key
OPENAI_CHAT_MODEL=gpt-4o-mini
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
STORE_ASSISTANT_MESSAGES=true
ENV

cat > .gitignore << 'GITIGNORE'
.env
.venv/
__pycache__/
*.pyc
*.pyo
.pytest_cache/
chroma_db/
conversation_logs.db
memory_evaluation.md
GITIGNORE
```

**🔧 Qué debes cambiar:**  
Reemplaza `pega_aqui_tu_api_key` por tu API key real.

**✅ Validación del paso:**

```bash
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('API key configurada:', bool(os.getenv('OPENAI_API_KEY')))"
```

**📌 Resultado esperado:**

```text
API key configurada: True
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20de%20un%20laboratorio%20de%20IA%20generativa.%20Prepar%C3%A9%20un%20proyecto%20local%20en%20Windows%20con%20VS%20Code%2C%20Git%20Bash%2C%20entorno%20virtual%2C%20requirements.txt%2C%20.env%20y%20.gitignore%20para%20construir%20un%20chatbot%20con%20memoria%20sem%C3%A1ntica.)

---

# 🧩 Tarea 2. Validar entorno, credenciales y configuración

## 🎯 Objetivo de la tarea

Crear un script de validación para comprobar que el entorno está listo antes de construir el chatbot.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `00_validar_entorno.py`

**📝 Descripción del paso:**  
Vas a crear el archivo `00_validar_entorno.py` en la raíz del proyecto. Este script revisará que el archivo `.env` tenga las variables requeridas, que las dependencias se puedan importar, que SQLite funcione correctamente y que exista la carpeta local `chroma_db/`. Su propósito es detectar problemas de ambiente antes de escribir el chatbot principal.

**⚙️ Contenido del paso:**

```bash
cat > 00_validar_entorno.py << 'PY'
import os
import sqlite3
from pathlib import Path

from dotenv import load_dotenv

load_dotenv()

required_keys = [
    "OPENAI_API_KEY",
    "OPENAI_CHAT_MODEL",
    "OPENAI_EMBEDDING_MODEL",
]

missing = []
for key in required_keys:
    value = os.getenv(key)
    if not value or value.startswith("pega_aqui"):
        missing.append(key)

if missing:
    print("Faltan variables de entorno o tienen valores de ejemplo:")
    for key in missing:
        print(f"- {key}")
    raise SystemExit(1)

try:
    import openai
    import chromadb
    import tenacity
except Exception as exc:
    print("Error importando dependencias:", exc)
    raise SystemExit(1)

conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE test (id INTEGER PRIMARY KEY, value TEXT)")
conn.execute("INSERT INTO test (value) VALUES ('ok')")
row = conn.execute("SELECT value FROM test").fetchone()
conn.close()
assert row[0] == "ok"

Path("chroma_db").mkdir(exist_ok=True)

print("Entorno validado correctamente.")
print("No se muestran claves por seguridad.")
print("Directorio ChromaDB listo: chroma_db/")
PY
```

**✅ Validación del paso:**

```bash
python 00_validar_entorno.py
```

**📌 Resultado esperado:**

```text
Entorno validado correctamente.
No se muestran claves por seguridad.
Directorio ChromaDB listo: chroma_db/
```

---

### ✅ Paso 2. Revisa la estructura inicial

**📝 Descripción del paso:**  
Vas a listar el contenido de la carpeta del laboratorio para confirmar que ya existen los archivos y directorios iniciales. Este paso no modifica archivos; solo verifica que la preparación previa quedó en la ubicación correcta.

**⚙️ Contenido del paso:**

```bash
ls -la
```

**✅ Validación del paso:**  
Debes encontrar:

```text
.env
.gitignore
.venv/
requirements.txt
00_validar_entorno.py
chroma_db/
```

**📌 Resultado esperado:**  
El proyecto tiene una base segura y validada.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20de%20un%20laboratorio%20donde%20valid%C3%A9%20variables%20de%20entorno%2C%20dependencias%2C%20SQLite%20y%20la%20carpeta%20de%20ChromaDB%20antes%20de%20construir%20un%20chatbot%20con%20memoria%20sem%C3%A1ntica.)

---

# 🧩 Tarea 3. Crear el chatbot con memoria híbrida

## 🎯 Objetivo de la tarea

Crear el archivo principal `semantic_memory_chat.py` con SQLite, ChromaDB, escritura dual, ventana deslizante, memoria semántica, prompt enriquecido y CLI interactivo.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `semantic_memory_chat.py`

**📝 Descripción del paso:**  
Vas a crear el archivo principal `semantic_memory_chat.py` en la raíz del proyecto. Este archivo contiene toda la aplicación del chatbot: configuración, conexión a SQLite, conexión a ChromaDB, almacenamiento de mensajes, recuperación semántica, construcción del prompt, llamada al modelo de OpenAI y CLI interactivo. Cópialo completo como un solo bloque para evitar errores de indentación o métodos incompletos.

**⚙️ Contenido del paso:**

```bash
cat > semantic_memory_chat.py << 'PY'
"""
semantic_memory_chat.py
Laboratorio 6: Chatbot con memoria semántica usando SQLite + ChromaDB.
"""

from __future__ import annotations

import os
import sqlite3
import time
import uuid
from datetime import datetime, timezone
from typing import Any

import chromadb
from chromadb.utils import embedding_functions
from dotenv import load_dotenv
from openai import OpenAI, APIConnectionError, APIStatusError, APITimeoutError, RateLimitError
from tenacity import retry, retry_if_exception_type, stop_after_attempt, wait_random_exponential

load_dotenv()

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
if not OPENAI_API_KEY or OPENAI_API_KEY.startswith("pega_aqui"):
    raise RuntimeError("OPENAI_API_KEY no está configurada correctamente en .env")

CHAT_MODEL = os.getenv("OPENAI_CHAT_MODEL", "gpt-4o-mini")
EMBEDDING_MODEL = os.getenv("OPENAI_EMBEDDING_MODEL", "text-embedding-3-small")
STORE_ASSISTANT_MESSAGES = os.getenv("STORE_ASSISTANT_MESSAGES", "true").lower() == "true"

SQLITE_DB_PATH = "conversation_logs.db"
CHROMA_DB_PATH = "./chroma_db"
COLLECTION_NAME = "conversation_memory"
WINDOW_SIZE = 5
SEMANTIC_K = 3

openai_client = OpenAI(api_key=OPENAI_API_KEY)


def estimate_tokens(text: str) -> int:
    """
    Estimación simple: 1 token ≈ 4 caracteres.
    No reemplaza contadores oficiales, pero sirve para trazabilidad local.
    """
    if not text:
        return 0
    return max(1, round(len(text) / 4))


class ConversationStore:
    """
    Administra memoria conversacional con dos capas:
    - SQLite: trazabilidad, sesiones, mensajes y métricas.
    - ChromaDB: búsqueda semántica sobre mensajes históricos.
    """

    def __init__(self) -> None:
        self.sqlite_conn = sqlite3.connect(SQLITE_DB_PATH)
        self.sqlite_conn.row_factory = sqlite3.Row
        self._init_sqlite_schema()

        self.chroma_client = chromadb.PersistentClient(path=CHROMA_DB_PATH)
        self.embedding_fn = embedding_functions.OpenAIEmbeddingFunction(
            api_key=OPENAI_API_KEY,
            model_name=EMBEDDING_MODEL,
        )
        self.collection = self.chroma_client.get_or_create_collection(
            name=COLLECTION_NAME,
            embedding_function=self.embedding_fn,
            metadata={"hnsw:space": "cosine"},
        )

        print(f"[ConversationStore] SQLite: {SQLITE_DB_PATH}")
        print(f"[ConversationStore] ChromaDB: {CHROMA_DB_PATH}")
        print(f"[ConversationStore] Colección: {COLLECTION_NAME}")
        print(f"[ConversationStore] Documentos en memoria: {self.collection.count()}")

    def _init_sqlite_schema(self) -> None:
        cursor = self.sqlite_conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS sessions (
                session_id TEXT PRIMARY KEY,
                created_at TEXT NOT NULL,
                model_name TEXT NOT NULL,
                embedding_model TEXT NOT NULL,
                description TEXT
            )
        """)
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS messages (
                message_id TEXT PRIMARY KEY,
                session_id TEXT NOT NULL,
                timestamp TEXT NOT NULL,
                role TEXT NOT NULL CHECK(role IN ('user', 'assistant', 'system')),
                content TEXT NOT NULL,
                token_count INTEGER DEFAULT 0,
                turn_index INTEGER NOT NULL DEFAULT 0,
                stored_in_chroma INTEGER DEFAULT 0,
                FOREIGN KEY (session_id) REFERENCES sessions(session_id)
            )
        """)
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_messages_session_turn
            ON messages(session_id, turn_index)
        """)
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_messages_role
            ON messages(role)
        """)
        self.sqlite_conn.commit()

    def create_session(self, description: str = "") -> str:
        session_id = str(uuid.uuid4())
        now = datetime.now(timezone.utc).isoformat()
        self.sqlite_conn.execute(
            """
            INSERT INTO sessions
                (session_id, created_at, model_name, embedding_model, description)
            VALUES (?, ?, ?, ?, ?)
            """,
            (session_id, now, CHAT_MODEL, EMBEDDING_MODEL, description),
        )
        self.sqlite_conn.commit()
        return session_id

    def add_message(
        self,
        session_id: str,
        role: str,
        content: str,
        token_count: int | None = None,
        store_in_chroma: bool | None = None,
    ) -> str:
        if role not in {"user", "assistant", "system"}:
            raise ValueError("role debe ser 'user', 'assistant' o 'system'")

        message_id = str(uuid.uuid4())
        now = datetime.now(timezone.utc).isoformat()
        token_count = token_count if token_count is not None else estimate_tokens(content)

        if store_in_chroma is None:
            store_in_chroma = role == "user" or (role == "assistant" and STORE_ASSISTANT_MESSAGES)

        row = self.sqlite_conn.execute(
            "SELECT COUNT(*) FROM messages WHERE session_id = ?",
            (session_id,),
        ).fetchone()
        turn_index = int(row[0])

        self.sqlite_conn.execute(
            """
            INSERT INTO messages
                (message_id, session_id, timestamp, role, content, token_count, turn_index, stored_in_chroma)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
            """,
            (message_id, session_id, now, role, content, token_count, turn_index, 1 if store_in_chroma else 0),
        )
        self.sqlite_conn.commit()

        if store_in_chroma:
            self.collection.add(
                ids=[message_id],
                documents=[content],
                metadatas=[{
                    "session_id": session_id,
                    "role": role,
                    "timestamp": now,
                    "turn_index": turn_index,
                    "token_count": token_count,
                }],
            )
        return message_id

    def get_window_messages(self, session_id: str, n: int = WINDOW_SIZE) -> list[dict[str, str]]:
        rows = self.sqlite_conn.execute(
            """
            SELECT role, content
            FROM messages
            WHERE session_id = ?
            ORDER BY turn_index DESC
            LIMIT ?
            """,
            (session_id, n),
        ).fetchall()
        return [{"role": row["role"], "content": row["content"]} for row in reversed(rows)]

    def retrieve_relevant_context(self, query: str, session_id: str, k: int = SEMANTIC_K) -> list[dict[str, Any]]:
        total_messages = self.sqlite_conn.execute(
            "SELECT COUNT(*) FROM messages WHERE session_id = ?",
            (session_id,),
        ).fetchone()[0]

        if total_messages <= WINDOW_SIZE:
            return []

        exclude_from_index = total_messages - WINDOW_SIZE
        n_results = min(k, exclude_from_index)
        if n_results <= 0:
            return []

        results = self.collection.query(
            query_texts=[query],
            n_results=n_results,
            where={"$and": [
                {"session_id": {"$eq": session_id}},
                {"turn_index": {"$lt": exclude_from_index}},
            ]},
            include=["documents", "metadatas", "distances"],
        )

        fragments: list[dict[str, Any]] = []
        documents = results.get("documents", [[]])[0]
        metadatas = results.get("metadatas", [[]])[0]
        distances = results.get("distances", [[]])[0]

        for doc, meta, distance in zip(documents, metadatas, distances):
            fragments.append({
                "content": doc,
                "role": meta.get("role", "unknown"),
                "turn_index": meta.get("turn_index", 0),
                "distance": round(float(distance), 4),
            })

        fragments.sort(key=lambda item: item["turn_index"])
        return fragments

    def build_prompt_with_memory(self, current_message: str, session_id: str) -> tuple[list[dict[str, str]], list[dict[str, Any]]]:
        semantic_fragments = self.retrieve_relevant_context(
            query=current_message,
            session_id=session_id,
            k=SEMANTIC_K,
        )

        system_content = (
            "Eres un asistente técnico experto. Puedes usar contexto de conversaciones anteriores cuando sea relevante. "
            "Distingue entre la conversación reciente y el contexto histórico recuperado. "
            "Si el contexto histórico no ayuda, ignóralo. No inventes información que no aparezca en la conversación."
        )

        if semantic_fragments:
            context_lines = ["\n\n--- Contexto histórico recuperado ---"]
            for fragment in semantic_fragments:
                context_lines.append(f"[Turno {fragment['turn_index']} - {fragment['role']}]: {fragment['content']}")
            context_lines.append("--- Fin del contexto histórico ---")
            system_content += "\n" + "\n".join(context_lines)

        messages: list[dict[str, str]] = [{"role": "system", "content": system_content}]
        messages.extend(self.get_window_messages(session_id, n=WINDOW_SIZE))
        messages.append({"role": "user", "content": current_message})
        return messages, semantic_fragments

    def get_session_stats(self, session_id: str) -> dict[str, Any]:
        row = self.sqlite_conn.execute(
            """
            SELECT
                COUNT(*) AS total_messages,
                SUM(CASE WHEN role = 'user' THEN 1 ELSE 0 END) AS user_messages,
                SUM(CASE WHEN role = 'assistant' THEN 1 ELSE 0 END) AS assistant_messages,
                SUM(token_count) AS total_tokens,
                MIN(timestamp) AS first_message,
                MAX(timestamp) AS last_message
            FROM messages
            WHERE session_id = ?
            """,
            (session_id,),
        ).fetchone()
        return {
            "total_messages": row["total_messages"] or 0,
            "user_messages": row["user_messages"] or 0,
            "assistant_messages": row["assistant_messages"] or 0,
            "total_tokens": row["total_tokens"] or 0,
            "first_message": row["first_message"],
            "last_message": row["last_message"],
            "chroma_docs_global": self.collection.count(),
        }

    def close(self) -> None:
        self.sqlite_conn.close()


@retry(
    wait=wait_random_exponential(multiplier=1, min=2, max=30),
    stop=stop_after_attempt(4),
    retry=retry_if_exception_type((RateLimitError, APIStatusError, APITimeoutError, APIConnectionError)),
    reraise=True,
)
def call_chat_model(messages: list[dict[str, str]]) -> tuple[str, int]:
    response = openai_client.chat.completions.create(
        model=CHAT_MODEL,
        messages=messages,
        temperature=0.3,
        max_tokens=500,
    )
    content = response.choices[0].message.content or ""
    total_tokens = response.usage.total_tokens if response.usage else estimate_tokens(content)
    return content, total_tokens


def chat_with_memory(store: ConversationStore, session_id: str, user_message: str) -> tuple[str, int, list[dict[str, Any]]]:
    messages, semantic_fragments = store.build_prompt_with_memory(user_message, session_id)
    store.add_message(session_id, "user", user_message)

    start = time.perf_counter()
    assistant_content, total_tokens = call_chat_model(messages)
    elapsed_ms = int((time.perf_counter() - start) * 1000)

    store.add_message(session_id=session_id, role="assistant", content=assistant_content, token_count=total_tokens)
    print(f"  [debug] Latencia: {elapsed_ms}ms | Tokens: {total_tokens} | Fragmentos semánticos: {len(semantic_fragments)}")
    return assistant_content, total_tokens, semantic_fragments


def print_separator(char: str = "─", width: int = 70) -> None:
    print(char * width)


def print_semantic_fragments(fragments: list[dict[str, Any]]) -> None:
    if not fragments:
        print("  No se recuperaron fragmentos semánticos.")
        return

    print(f"\n  🧠 Memoria semántica recuperada ({len(fragments)} fragmentos):")
    for index, fragment in enumerate(fragments, start=1):
        preview = fragment["content"][:90]
        if len(fragment["content"]) > 90:
            preview += "..."
        print(f"  [{index}] Turno {fragment['turn_index']} ({fragment['role']}) dist={fragment['distance']}: {preview}")


def main() -> int:
    print("\n" + "═" * 70)
    print("  🤖 Chatbot con Memoria Semántica — Laboratorio 6")
    print("  Comandos: /memory | /stats | /new | /quit")
    print("═" * 70)

    store = ConversationStore()
    session_id = store.create_session("Sesión interactiva")
    last_fragments: list[dict[str, Any]] = []

    print(f"\n✅ Nueva sesión iniciada: {session_id[:8]}...")
    print(f"   Modelo chat: {CHAT_MODEL}")
    print(f"   Modelo embeddings: {EMBEDDING_MODEL}")
    print(f"   Ventana deslizante: {WINDOW_SIZE} mensajes")
    print(f"   Recuperación semántica: k={SEMANTIC_K}")
    print_separator()

    try:
        while True:
            try:
                user_input = input("\n👤 Tú: ").strip()
            except (EOFError, KeyboardInterrupt):
                print("\n\n👋 Cerrando sesión...")
                break

            if not user_input:
                continue

            command = user_input.lower()
            if command == "/quit":
                print("👋 ¡Hasta luego!")
                break
            if command == "/new":
                session_id = store.create_session("Nueva sesión interactiva")
                last_fragments = []
                print(f"✅ Nueva sesión iniciada: {session_id[:8]}...")
                continue
            if command == "/memory":
                print_separator("·")
                print_semantic_fragments(last_fragments)
                print_separator("·")
                continue
            if command == "/stats":
                stats = store.get_session_stats(session_id)
                print_separator("·")
                print(f"  📊 Estadísticas de la sesión {session_id[:8]}...")
                print(f"     Mensajes totales     : {stats['total_messages']}")
                print(f"     Mensajes del usuario : {stats['user_messages']}")
                print(f"     Mensajes asistente   : {stats['assistant_messages']}")
                print(f"     Tokens registrados   : {stats['total_tokens']}")
                print(f"     Docs Chroma global   : {stats['chroma_docs_global']}")
                print_separator("·")
                continue

            print("🤔 Pensando...", end="", flush=True)
            try:
                response, _tokens, last_fragments = chat_with_memory(store, session_id, user_input)
                print(f"\r🤖 Asistente: {response}")
            except Exception as exc:
                print(f"\r❌ Error al procesar el mensaje: {exc}")
                print("   Revisa tu API key, conexión, cuota o intenta de nuevo.")
    finally:
        store.close()
        print("\n💾 Sesión guardada en SQLite y ChromaDB.")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile semantic_memory_chat.py
```

**📌 Resultado esperado:**  
No debe aparecer ningún error.

---

### ✅ Paso 2. Prueba la escritura dual

**📝 Descripción del paso:**  
Vas a ejecutar una prueba rápida desde Git Bash para importar la clase `ConversationStore` del archivo `semantic_memory_chat.py`, crear una sesión temporal y guardar un mensaje de usuario. La prueba debe confirmar que el mensaje queda registrado en SQLite y también indexado como documento en ChromaDB.

**⚙️ Contenido del paso:**

```bash
python -c "
from semantic_memory_chat import ConversationStore
s = ConversationStore()
sid = s.create_session('validación escritura dual')
mid = s.add_message(sid, 'user', 'Mi nombre es Laura y trabajo con Python.')
print('message_id:', mid)
print('SQLite mensajes:', s.sqlite_conn.execute('SELECT COUNT(*) FROM messages').fetchone()[0])
print('Chroma docs:', s.collection.count())
s.close()
"
```

**✅ Validación del paso:**  
Debe mostrarse al menos 1 mensaje en SQLite y al menos 1 documento en ChromaDB.

**📌 Resultado esperado:**  
La escritura dual funciona.

---

### ✅ Paso 3. Prueba la ventana deslizante y memoria semántica

**📝 Descripción del paso:**  
Vas a ejecutar una prueba controlada que crea una sesión, inserta varios mensajes simulados y luego consulta la memoria semántica. El objetivo es comprobar que la ventana deslizante solo conserva los mensajes recientes, mientras ChromaDB puede recuperar mensajes más antiguos cuando la pregunta se relaciona con ellos.

**⚙️ Contenido del paso:**

```bash
python - << 'PY'
from semantic_memory_chat import ConversationStore, WINDOW_SIZE

s = ConversationStore()
sid = s.create_session('memoria semántica')

mensajes = [
    ('user', 'Mi nombre es Carlos y trabajo como DevOps Engineer.'),
    ('assistant', 'Hola Carlos, puedo ayudarte con temas DevOps.'),
    ('user', 'Uso Kubernetes, Docker y GitHub Actions.'),
    ('assistant', 'Ese stack es común para CI/CD moderno.'),
    ('user', 'También administro infraestructura en AWS.'),
    ('assistant', 'AWS ofrece servicios útiles para infraestructura.'),
    ('user', 'Me interesa monitoreo con Prometheus y Grafana.'),
    ('assistant', 'Prometheus y Grafana son herramientas muy usadas.'),
]

for role, content in mensajes:
    s.add_message(sid, role, content)

window = s.get_window_messages(sid)
frags = s.retrieve_relevant_context('¿Cómo se llama el usuario?', sid, k=3)

print('Ventana:', len(window), 'esperado:', WINDOW_SIZE)
print('Fragmentos:', len(frags))

for f in frags:
    print(f"turno={f['turn_index']} role={f['role']} dist={f['distance']} text={f['content'][:60]}")

s.close()
PY
```

**✅ Validación del paso:**  
Deben recuperarse fragmentos antiguos, idealmente incluyendo el mensaje donde se menciona el nombre Carlos.

**📌 Resultado esperado:**  
La memoria semántica recupera contexto fuera de la ventana reciente.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20de%20un%20laboratorio%20donde%20constru%C3%AD%20un%20chatbot%20con%20SQLite%2C%20ChromaDB%2C%20escritura%20dual%2C%20ventana%20deslizante%2C%20memoria%20sem%C3%A1ntica%2C%20prompt%20enriquecido%20y%20CLI%20interactivo.)

---

# 🧩 Tarea 4. Probar el CLI interactivo

## 🎯 Objetivo de la tarea

Usar el chatbot desde la terminal y comprobar los comandos `/memory`, `/stats`, `/new` y `/quit`.

---

## 🛠️ Pasos

### ✅ Paso 1. Ejecuta el chatbot

**📝 Descripción del paso:**  
Vas a ejecutar el archivo `semantic_memory_chat.py` desde Git Bash para abrir el chatbot en modo consola. Antes de correrlo, asegúrate de estar en la raíz del proyecto y de tener el entorno virtual `.venv` activado.

**⚙️ Contenido del paso:**

```bash
python semantic_memory_chat.py
```

**✅ Validación del paso:**  
Debe mostrarse la pantalla inicial del chatbot.

**📌 Resultado esperado:**

```text
🤖 Chatbot con Memoria Semántica — Laboratorio 6
Comandos: /memory | /stats | /new | /quit
```

---

### ✅ Paso 2. Ejecuta una conversación corta

**📝 Descripción del paso:**  
Vas a escribir manualmente varios mensajes dentro del CLI del chatbot. Cada mensaje que ingreses será procesado por el modelo y guardado en SQLite; además, los mensajes configurados para memoria se indexarán en ChromaDB. Escribe los mensajes uno por uno y espera la respuesta del asistente antes de continuar con el siguiente.

**⚙️ Contenido del paso:**  
Escribe estos mensajes, uno por uno:

```text
Hola, me llamo Valeria y trabajo con análisis de datos.
```
```text
Uso Python, SQL y Power BI.
```
```text
También estoy aprendiendo RAG con ChromaDB.
```
```text
Estoy creando un asistente para consultar reportes internos.
```
```text
Mi equipo trabaja con Azure y bases PostgreSQL.
```
```text
¿Qué tecnologías mencioné?
```

**✅ Validación del paso:**  
El asistente debe responder considerando la conversación reciente.

**📌 Resultado esperado:**  
El chat funciona y se guardan mensajes en SQLite y ChromaDB.

---

### ✅ Paso 3. Usa `/memory`

**📝 Descripción del paso:**  
Vas a escribir el comando `/memory` dentro del CLI del chatbot, no en Git Bash como comando externo. Este comando muestra los fragmentos históricos que ChromaDB recuperó para la última pregunta procesada, junto con su turno, rol y distancia de similitud.

**⚙️ Contenido del paso:**

```text
/memory
```

**✅ Validación del paso:**  
Debe mostrar fragmentos si ya existe suficiente historial fuera de la ventana reciente.

**📌 Resultado esperado:**  
Puedes ver qué recuerdos semánticos se inyectaron en el prompt.

---

### ✅ Paso 4. Usa `/stats`

**📝 Descripción del paso:**  
Vas a escribir el comando `/stats` dentro del CLI del chatbot para consultar el estado de la sesión actual. El comando muestra conteos de mensajes, tokens registrados y documentos almacenados en ChromaDB; sirve para validar trazabilidad y persistencia.

**⚙️ Contenido del paso:**

```text
/stats
```

**✅ Validación del paso:**  
Debe mostrar mensajes totales, mensajes del usuario, mensajes del asistente, tokens registrados y documentos en ChromaDB.

**📌 Resultado esperado:**  
Puedes auditar el estado de la conversación.

---

### ✅ Paso 5. Cierra el chat

**📝 Descripción del paso:**  
Vas a escribir el comando `/quit` dentro del CLI del chatbot para cerrar la sesión de forma controlada. Esto permite finalizar el ciclo interactivo, cerrar la conexión con SQLite y dejar persistidos los datos generados durante la conversación.

**⚙️ Contenido del paso:**

```text
/quit
```

**✅ Validación del paso:**  
Debe cerrarse la conexión SQLite.

**📌 Resultado esperado:**  
La sesión queda persistida.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20de%20un%20laboratorio%20donde%20prob%C3%A9%20un%20chatbot%20CLI%20con%20comandos%20memory%2C%20stats%2C%20new%20y%20quit%20para%20inspeccionar%20memoria%20sem%C3%A1ntica%20y%20trazabilidad.)

---

# 🧩 Tarea 5. Crear demo comparativa de memoria semántica

## 🎯 Objetivo de la tarea

Simular una conversación larga y comprobar si la memoria semántica recupera información fuera de la ventana reciente.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `demo_semantic_memory.py`

**📝 Descripción del paso:**  
Vas a crear el archivo `demo_semantic_memory.py` en la raíz del proyecto. Este script automatiza una conversación larga, guarda los turnos usando el chatbot ya construido y después hace preguntas que requieren recuperar información mencionada al inicio. No reemplaza `semantic_memory_chat.py`; lo importa y lo usa como módulo.

**⚙️ Contenido del paso:**

```bash
cat > demo_semantic_memory.py << 'PY'
"""
Demo de memoria semántica vs. ventana deslizante.
"""

import time

from semantic_memory_chat import ConversationStore, chat_with_memory, WINDOW_SIZE

DEMO_CONVERSATION = [
    "Hola, me llamo Sofía y soy ML Engineer en una startup de e-commerce.",
    "Estamos construyendo un sistema de recomendaciones de productos.",
    "Nuestro stack técnico es Python 3.11, FastAPI, PostgreSQL y Redis.",
    "Tenemos aproximadamente 500000 productos en el catálogo.",
    "El presupuesto de infraestructura es de 2000 dólares mensuales.",
    "Nuestro SLA requiere respuestas en menos de 100 milisegundos.",
    "Estamos evaluando Pinecone, Weaviate, ChromaDB y Qdrant.",
    "El equipo prefiere soluciones self-hosted cuando sea posible.",
    "Tenemos GPUs NVIDIA A100 disponibles en AWS.",
    "El click-through rate actual del sistema es de 2.3 por ciento.",
    "Queremos usar Redis para cachear recomendaciones frecuentes.",
    "También necesitamos monitorear drift del modelo en producción.",
]

MEMORY_TESTS = [
    {"question": "¿Cómo se llama la persona y cuál es su rol?", "expected": ["Sofía", "ML Engineer"]},
    {"question": "¿Cuál era el stack técnico mencionado?", "expected": ["Python", "FastAPI", "PostgreSQL", "Redis"]},
    {"question": "¿Cuántos productos tiene el catálogo y cuál es el presupuesto?", "expected": ["500000", "2000"]},
    {"question": "¿Qué bases vectoriales estaban evaluando?", "expected": ["Pinecone", "Weaviate", "ChromaDB", "Qdrant"]},
]


def response_contains_expected(response: str, expected_terms: list[str]) -> bool:
    response_lower = response.lower()
    return any(term.lower() in response_lower for term in expected_terms)


def run_demo() -> None:
    print("\n" + "═" * 72)
    print("  🧪 Demo: memoria semántica vs. ventana deslizante")
    print("═" * 72)

    store = ConversationStore()
    session_id = store.create_session("Demo laboratorio 6")

    print(f"\nSesión: {session_id[:8]}...")
    print(f"Ventana deslizante: últimos {WINDOW_SIZE} mensajes")
    print(f"Turnos simulados: {len(DEMO_CONVERSATION)}")

    print("\n⏳ Simulando conversación...\n")
    for index, user_message in enumerate(DEMO_CONVERSATION, start=1):
        print(f"Turno {index:02d}: {user_message[:70]}...", end=" ")
        try:
            _response, tokens, _frags = chat_with_memory(store, session_id, user_message)
            print(f"✓ {tokens} tokens")
            time.sleep(0.5)
        except Exception as exc:
            print(f"✗ Error: {exc}")
            time.sleep(2.0)

    print("\n" + "═" * 72)
    print("  🔍 Preguntas que requieren memoria antigua")
    print("═" * 72)

    rows = []
    for test in MEMORY_TESTS:
        print(f"\nPregunta: {test['question']}")
        response, _tokens, fragments = chat_with_memory(store, session_id, test["question"])
        ok = response_contains_expected(response, test["expected"])

        print(f"Respuesta: {response[:250]}...")
        print(f"Fragmentos recuperados: {len(fragments)}")
        for frag in fragments:
            print(f"  - turno={frag['turn_index']} dist={frag['distance']} text={frag['content'][:70]}...")
        print("Resultado:", "✅ correcto/parcial" if ok else "⚠️ revisar")

        rows.append({
            "question": test["question"],
            "expected": ", ".join(test["expected"]),
            "fragments": len(fragments),
            "ok": ok,
        })

    correct = sum(1 for row in rows if row["ok"])
    rate = correct / len(rows) * 100

    print("\n" + "═" * 72)
    print("  📊 Resumen")
    print("═" * 72)
    print(f"Preguntas correctas o parcialmente correctas: {correct}/{len(rows)}")
    print(f"Tasa aproximada de recuperación: {rate:.0f}%")

    with open("memory_evaluation.md", "w", encoding="utf-8") as file:
        file.write("# Evaluación de memoria semántica\n\n")
        file.write("| Pregunta | Esperado | Fragmentos recuperados | Resultado |\n")
        file.write("|---|---|---:|---|\n")
        for row in rows:
            result = "✅" if row["ok"] else "⚠️"
            file.write(f"| {row['question']} | {row['expected']} | {row['fragments']} | {result} |\n")

    print("\nArchivo generado: memory_evaluation.md")
    store.close()


if __name__ == "__main__":
    run_demo()
PY
```

**✅ Validación del paso:**

```bash
python -m py_compile demo_semantic_memory.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Ejecuta la demo

**📝 Descripción del paso:**  
Vas a ejecutar el archivo `demo_semantic_memory.py` desde Git Bash. El script creará una sesión nueva, enviará varios mensajes simulados al chatbot y generará una evaluación en `memory_evaluation.md`. Esta ejecución puede consumir llamadas a la API, por lo que conviene correrla solo cuando las pruebas anteriores ya funcionen.

**⚙️ Contenido del paso:**

```bash
python demo_semantic_memory.py
```

**✅ Validación del paso:**  
Debe generarse el archivo:

```text
memory_evaluation.md
```

**📌 Resultado esperado:**  
Tienes evidencia de cuándo la memoria semántica ayudó a recuperar información antigua.

---

### ✅ Paso 3. Revisa la evaluación

**📝 Descripción del paso:**  
Vas a abrir o imprimir el archivo `memory_evaluation.md` generado por la demo. Este archivo resume las preguntas realizadas, los datos esperados, la cantidad de fragmentos recuperados y si la respuesta fue correcta o parcial. Puedes revisarlo con `cat` en Git Bash o abrirlo desde VS Code.

**⚙️ Contenido del paso:**

```bash
cat memory_evaluation.md
```

**✅ Validación del paso:**  
La tabla debe mostrar preguntas, datos esperados, fragmentos recuperados y resultado.

**📌 Resultado esperado:**  
Puedes discutir con criterio cuándo la memoria semántica funcionó y cuándo fue parcial.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20de%20un%20laboratorio%20donde%20simul%C3%A9%20una%20conversaci%C3%B3n%20larga%20y%20prob%C3%A9%20si%20la%20memoria%20sem%C3%A1ntica%20con%20ChromaDB%20pod%C3%ADa%20recuperar%20informaci%C3%B3n%20mencionada%20fuera%20de%20la%20ventana%20deslizante.)

---

# 🧩 Tarea 6. Validar funcionamiento integral y entregar evidencias

## 🎯 Objetivo de la tarea

Confirmar que todos los componentes funcionan y reunir los archivos que demuestran tu trabajo.

---

## 🛠️ Pasos

### ✅ Paso 1. Valida sintaxis de scripts

**📝 Descripción del paso:**  
Vas a compilar los tres scripts Python del laboratorio sin ejecutarlos completamente. Este paso revisa sintaxis e indentación en `00_validar_entorno.py`, `semantic_memory_chat.py` y `demo_semantic_memory.py`. Si alguno tiene errores, corrige primero el archivo indicado antes de continuar.

**⚙️ Contenido del paso:**

```bash
python -m py_compile 00_validar_entorno.py
python -m py_compile semantic_memory_chat.py
python -m py_compile demo_semantic_memory.py
```

**✅ Validación del paso:**  
Ningún comando debe mostrar errores.

**📌 Resultado esperado:**  
Los scripts son válidos.

---

### ✅ Paso 2. Ejecuta validación integrada rápida

**📝 Descripción del paso:**  
Vas a ejecutar una validación integral desde Git Bash usando el código de `semantic_memory_chat.py`. Esta prueba crea una sesión temporal, inserta mensajes, valida que la ventana deslizante tenga el tamaño esperado, consulta memoria semántica y revisa estadísticas. No necesitas abrir ni editar archivos en este paso.

**⚙️ Contenido del paso:**

```bash
python -c "
from semantic_memory_chat import ConversationStore, WINDOW_SIZE

store = ConversationStore()
sid = store.create_session('validación integral')

items = [
    ('user', 'Soy ingeniero de software y uso Python.'),
    ('assistant', 'Perfecto, Python es muy versátil.'),
    ('user', 'Trabajo con APIs REST y microservicios.'),
    ('assistant', 'Las APIs REST son comunes en microservicios.'),
    ('user', 'Mi stack favorito es FastAPI PostgreSQL Redis.'),
    ('assistant', 'Ese stack es sólido para backend.'),
    ('user', 'También uso Docker y Kubernetes.'),
    ('assistant', 'Docker y Kubernetes ayudan en despliegues.'),
]

for role, content in items:
    store.add_message(sid, role, content)

window = store.get_window_messages(sid)
frags = store.retrieve_relevant_context('¿Qué lenguaje usa el usuario?', sid)
stats = store.get_session_stats(sid)

assert len(window) == WINDOW_SIZE
assert stats['total_messages'] == 8
assert len(frags) > 0

print('✅ Validación integral correcta')
print('Ventana:', len(window))
print('Fragmentos:', len(frags))
print('Stats:', stats)
store.close()
"
```

**✅ Validación del paso:**  
Debe aparecer:

```text
✅ Validación integral correcta
```

**📌 Resultado esperado:**  
SQLite, ChromaDB, ventana y recuperación semántica funcionan juntos.

---

### ✅ Paso 3. Revisa archivos generados

**📝 Descripción del paso:**  
Vas a listar los archivos de la carpeta del laboratorio para confirmar que el código, las bases locales y la evaluación existen. Este paso sirve como revisión final de evidencias antes de preparar la entrega.

**⚙️ Contenido del paso:**

```bash
ls -la
```

**✅ Validación del paso:**  
Debes tener al menos:

```text
00_validar_entorno.py
semantic_memory_chat.py
demo_semantic_memory.py
requirements.txt
conversation_logs.db
chroma_db/
memory_evaluation.md
```

**📌 Resultado esperado:**  
Tienes evidencias de código, base relacional, base vectorial y evaluación.

---

### ✅ Paso 4. Entrega evidencias seguras

**📝 Descripción del paso:**  
Vas a separar los archivos seguros de entrega de los archivos locales o sensibles. Debes entregar únicamente código, dependencias y evaluación; no debes entregar `.env`, `.venv/`, `conversation_logs.db` ni `chroma_db/`, porque pueden contener credenciales, paquetes locales o conversaciones persistidas.

**⚙️ Contenido del paso:**  
Puedes entregar:

```text
00_validar_entorno.py
semantic_memory_chat.py
demo_semantic_memory.py
requirements.txt
memory_evaluation.md
```

No entregues:

```text
.env
.venv/
conversation_logs.db
chroma_db/
```

**✅ Validación del paso:**  
Verifica que `.env` no esté incluido en entregables.

**📌 Resultado esperado:**  
Tu entrega no contiene credenciales ni datos conversacionales persistidos.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20de%20un%20laboratorio%20donde%20valid%C3%A9%20el%20funcionamiento%20integral%20de%20un%20chatbot%20con%20memoria%20sem%C3%A1ntica%2C%20SQLite%2C%20ChromaDB%2C%20ventana%20deslizante%20y%20evaluaci%C3%B3n%20de%20recuperaci%C3%B3n.)

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Variables de entorno configuradas de forma segura.
4. Script de validación de entorno.
5. Clase `ConversationStore` implementada.
6. Base SQLite con tablas `sessions` y `messages`.
7. Colección ChromaDB persistente para memoria semántica.
8. Escritura dual de mensajes.
9. Ventana deslizante funcional.
10. Recuperación semántica filtrada por sesión.
11. Exclusión de la ventana reciente en la búsqueda semántica.
12. Prompt enriquecido con contexto histórico y conversación reciente.
13. Chatbot CLI interactivo con comandos especiales.
14. Demo comparativa de memoria.
15. Archivo `memory_evaluation.md` con observaciones.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del ambiente local | 10% |
| Configuración segura de credenciales | 10% |
| Diseño SQLite correcto | 10% |
| Inicialización y uso de ChromaDB | 10% |
| Implementación de escritura dual | 15% |
| Recuperación con ventana deslizante | 10% |
| Recuperación semántica filtrada | 15% |
| CLI funcional con comandos | 10% |
| Demo y evaluación de memoria | 10% |
| Total | 100% |

---

# ⚠️ Errores comunes que debes evitar

1. Escribir la API key directamente en el código.
2. Subir `.env` a un repositorio.
3. Ejecutar la demo larga muchas veces sin revisar costos.
4. Guardar información sensible en la conversación.
5. No excluir la ventana reciente de la recuperación semántica.
6. Recuperar memoria de otras sesiones por no filtrar `session_id`.
7. Asumir que memoria semántica siempre recupera el fragmento correcto.
8. Guardar respuestas incorrectas del asistente y tratarlas como verdad.
9. Eliminar `chroma_db/` y esperar que la memoria vectorial siga existiendo.
10. Entregar `conversation_logs.db` si contiene conversaciones privadas.

---

# 🧯 Solución de problemas

## Problema 1. `OPENAI_API_KEY no está configurada`

**Causa probable:**  
El archivo `.env` no existe, tiene un valor de ejemplo o está en otra carpeta.

**Solución:**

```bash
cat .env
python 00_validar_entorno.py
```

Verifica que exista:

```text
OPENAI_API_KEY=sk-...
```

---

## Problema 2. Error de ChromaDB por colección inconsistente

**Causa probable:**  
La carpeta `chroma_db/` fue creada con otro modelo, otra versión o datos corruptos.

**Solución:**

```bash
rm -rf chroma_db/
rm -f conversation_logs.db
python -c "from semantic_memory_chat import ConversationStore; s=ConversationStore(); s.close()"
```

---

## Problema 3. `RateLimitError` o lentitud durante la demo

**Causa probable:**  
Demasiadas llamadas seguidas al modelo o límite bajo de cuenta.

**Solución:**  
Aumenta las pausas en `demo_semantic_memory.py`:

```python
time.sleep(2.0)
```

O reduce la conversación:

```python
DEMO_CONVERSATION = DEMO_CONVERSATION[:8]
```

---

## Problema 4. `/memory` no muestra fragmentos

**Causa probable:**  
Aún no hay suficientes mensajes para que exista historial fuera de la ventana deslizante.

**Solución:**  
Agrega más de `WINDOW_SIZE` mensajes y vuelve a preguntar algo relacionado con los primeros turnos.

---

## Problema 5. La respuesta no usa memoria antigua

**Causa probable:**  
El fragmento recuperado no fue relevante, el valor de `k` es bajo o la pregunta no se parece semánticamente al mensaje original.

**Solución:**

```python
SEMANTIC_K = 5
```

También puedes reformular la pregunta para que se parezca más al dato original.

---

# 🧹 Limpieza del entorno

Ejecuta estos comandos si deseas limpiar los datos generados:

```bash
rm -rf chroma_db/
rm -f conversation_logs.db
rm -f memory_evaluation.md
```

Para desactivar el entorno virtual:

```bash
deactivate
```

Para eliminar el entorno virtual completo:

```bash
rm -rf .venv/
```

Antes de compartir el proyecto, valida que no haya claves en código:

```bash
grep -r "sk-" . --include="*.py" --include="*.txt" 2>/dev/null || echo "No se encontraron claves en archivos de código"
```

---

# 📚 Resumen conceptual

En este laboratorio construiste un chatbot con memoria persistente. La arquitectura final separa responsabilidades:

| Capa | Tecnología | Función |
|---|---|---|
| Conversación inmediata | Ventana deslizante | Mantener continuidad de los últimos mensajes |
| Logging estructurado | SQLite | Guardar sesiones, mensajes, roles, tiempos y métricas |
| Memoria semántica | ChromaDB | Recuperar mensajes históricos relevantes |
| Generación | OpenAI | Responder usando prompt enriquecido |

La clave del diseño está en no enviar toda la conversación al modelo. En su lugar, recuperas una ventana reciente y algunos fragmentos históricos relevantes. Este patrón reduce tokens, mejora trazabilidad y permite construir asistentes que recuerdan información importante sin depender únicamente de una ventana de contexto fija.
