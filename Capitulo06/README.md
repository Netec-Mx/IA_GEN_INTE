---LAB_START---
LAB_ID: 06-00-01
---MARKDOWN---
# Desarrollar una aplicación que integre ChromaDB para persistir conversaciones y recuperar contexto mediante búsquedas semánticas

## Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 50 minutos                                   |
| **Complejidad**  | Alta                                         |
| **Nivel Bloom**  | Crear                                        |
| **Módulo**       | 6 — Almacenamiento y Memoria en Sistemas GenAI |
| **Costo estimado** | < $0.10 USD (OpenAI Embeddings + gpt-4o-mini) |

---

## Descripción General

En este laboratorio construirás un chatbot CLI interactivo llamado `semantic_memory_chat.py` que combina dos estrategias de memoria: una **ventana deslizante** de los últimos 5 mensajes (contexto inmediato) y una **memoria semántica** basada en ChromaDB que recupera fragmentos relevantes de conversaciones pasadas mediante búsqueda por similitud. Esta arquitectura refleja el patrón de separación de responsabilidades que viste en la lección 6.1: SQLite actúa como capa de logging estructurado (auditoría, métricas, trazabilidad) mientras ChromaDB provee la capa de recuperación semántica. El resultado es un sistema capaz de "recordar" información mencionada hace decenas de turnos, superando las limitaciones de una ventana de contexto fija.

---

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Implementar la clase `ConversationStore` que persiste mensajes simultáneamente en SQLite (logs estructurados) y ChromaDB (embeddings vectoriales).
- [ ] Construir la función `retrieve_relevant_context()` que recupera los fragmentos semánticamente más similares al mensaje actual, excluyendo la ventana deslizante reciente.
- [ ] Combinar memoria semántica y ventana deslizante en `build_prompt_with_memory()` para generar prompts enriquecidos con contexto relevante.
- [ ] Demostrar cuantitativamente la ventaja de la memoria semántica sobre la ventana deslizante en conversaciones largas (30+ turnos).
- [ ] Aplicar el patrón de logging estructurado de la lección 6.1 (SQLite como capa de trazabilidad) en un pipeline GenAI completo.

---

## Prerrequisitos

### Conocimiento Previo

- Haber completado el Lab 05-00-01 (ChromaDB y embeddings básicos).
- Comprensión del esquema de mensajes de la API de OpenAI (`system`, `user`, `assistant`).
- Familiaridad con SQLite usando el módulo `sqlite3` de la librería estándar de Python.
- Conocimiento de los conceptos de logging estructurado presentados en la lección 6.1.

### Acceso y Credenciales

- `OPENAI_API_KEY` válida configurada en el entorno (con acceso a `text-embedding-3-small` y `gpt-4o-mini`).
- Presupuesto estimado: < $0.10 USD para completar el lab completo.
- Conexión a Internet estable para llamadas a la API de OpenAI.

---

## Entorno del Laboratorio

### Hardware Requerido

| Componente     | Mínimo                          | Recomendado                     |
|----------------|---------------------------------|---------------------------------|
| RAM            | 16 GB DDR4                      | 32 GB DDR4                      |
| Almacenamiento | 500 MB libres en SSD            | 2 GB libres en SSD              |
| CPU            | 4 núcleos físicos               | 8 núcleos                       |
| Internet       | 10 Mbps estable                 | 25 Mbps                         |

### Software Requerido

| Paquete              | Versión         | Propósito                              |
|----------------------|-----------------|----------------------------------------|
| Python               | 3.11.x          | Entorno de ejecución                   |
| `openai`             | 1.35.x          | SDK para embeddings y chat completions |
| `chromadb`           | 0.5.x           | Base de datos vectorial local          |
| `python-dotenv`      | 1.0.x           | Carga segura de variables de entorno   |
| `sqlite3`            | stdlib          | Logging estructurado (incluido en Python) |
| `uuid`               | stdlib          | Generación de identificadores únicos   |
| `rich`               | 13.x            | Salida formateada en terminal (opcional pero recomendado) |

### Configuración del Entorno

Ejecuta los siguientes comandos en tu terminal para preparar el entorno aislado:

```bash
# 1. Crear directorio del laboratorio
mkdir lab06_semantic_memory && cd lab06_semantic_memory

# 2. Crear y activar entorno virtual aislado
python3.11 -m venv .venv

# Linux/macOS
source .venv/bin/activate

# Windows (PowerShell)
# .venv\Scripts\Activate.ps1

# 3. Actualizar pip
pip install --upgrade pip

# 4. Instalar dependencias
pip install openai==1.35.3 chromadb==0.5.3 python-dotenv==1.0.1 rich==13.7.1

# 5. Verificar instalación
python -c "import chromadb; import openai; print('OK — ChromaDB:', chromadb.__version__, '| OpenAI SDK:', openai.__version__)"
```

**Salida esperada:**
```
OK — ChromaDB: 0.5.3 | OpenAI SDK: 1.35.3
```

```bash
# 6. Crear archivo .env con tu API key
cat > .env << 'EOF'
OPENAI_API_KEY=sk-tu_clave_aqui
EOF

# 7. Crear .gitignore para proteger credenciales
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
chroma_db/
conversation_logs.db
EOF
```

> ⚠️ **Seguridad:** Nunca escribas tu API key directamente en el código fuente. Verifica que `.env` aparezca en `.gitignore` antes de continuar.

---

## Pasos del Laboratorio

### Paso 1: Diseñar el Esquema SQLite y la Clase `ConversationStore`

**Objetivo:** Crear la estructura base de datos dual (SQLite + ChromaDB) que persiste cada mensaje en ambos backends simultáneamente, replicando el patrón de logging estructurado de la lección 6.1.

#### Instrucciones

1. Crea el archivo principal del laboratorio:

```bash
touch semantic_memory_chat.py
```

2. Abre `semantic_memory_chat.py` en tu editor y escribe el siguiente bloque de importaciones y la clase `ConversationStore`:

```python
"""
semantic_memory_chat.py
Lab 06-00-01: Chatbot con memoria semántica usando ChromaDB + SQLite

Arquitectura de almacenamiento dual:
  - SQLite  → logs estructurados (auditoría, métricas, trazabilidad)
  - ChromaDB → embeddings vectoriales (recuperación semántica)
"""

import os
import uuid
import sqlite3
import time
from datetime import datetime, timezone
from typing import List, Optional

import chromadb
from chromadb.utils import embedding_functions
from openai import OpenAI
from dotenv import load_dotenv

# ── Carga de configuración ─────────────────────────────────────────────────
load_dotenv()
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
if not OPENAI_API_KEY:
    raise EnvironmentError("OPENAI_API_KEY no encontrada. Verifica tu archivo .env")

# ── Clientes globales ──────────────────────────────────────────────────────
openai_client = OpenAI(api_key=OPENAI_API_KEY)

EMBEDDING_MODEL  = "text-embedding-3-small"
CHAT_MODEL       = "gpt-4o-mini"
SQLITE_DB_PATH   = "conversation_logs.db"
CHROMA_DB_PATH   = "./chroma_db"
WINDOW_SIZE      = 5   # Últimos N mensajes en la ventana deslizante
SEMANTIC_K       = 3   # Fragmentos semánticos a recuperar


# ══════════════════════════════════════════════════════════════════════════
# CLASE PRINCIPAL: ConversationStore
# ══════════════════════════════════════════════════════════════════════════

class ConversationStore:
    """
    Gestiona el almacenamiento dual de conversaciones:
      1. SQLite  — logs estructurados con schema fijo (lección 6.1)
      2. ChromaDB — embeddings para recuperación semántica (lab 5)
    """

    def __init__(self):
        # ── Backend 1: SQLite ──────────────────────────────────────────
        self.sqlite_conn = sqlite3.connect(SQLITE_DB_PATH)
        self.sqlite_conn.row_factory = sqlite3.Row  # Acceso por nombre de columna
        self._init_sqlite_schema()

        # ── Backend 2: ChromaDB ────────────────────────────────────────
        self.chroma_client = chromadb.PersistentClient(path=CHROMA_DB_PATH)

        # Función de embedding usando OpenAI (igual que lab 5)
        self.embedding_fn = embedding_functions.OpenAIEmbeddingFunction(
            api_key=OPENAI_API_KEY,
            model_name=EMBEDDING_MODEL
        )

        # Colección única para todos los mensajes de todas las sesiones
        self.collection = self.chroma_client.get_or_create_collection(
            name="conversation_memory",
            embedding_function=self.embedding_fn,
            metadata={"hnsw:space": "cosine"}  # Distancia coseno para similitud semántica
        )

        print(f"[ConversationStore] SQLite: {SQLITE_DB_PATH}")
        print(f"[ConversationStore] ChromaDB: {CHROMA_DB_PATH}")
        print(f"[ConversationStore] Documentos en memoria: {self.collection.count()}")

    def _init_sqlite_schema(self):
        """
        Crea las tablas si no existen.
        Schema inspirado en la lección 6.1 (sessions + conversation_turns).
        """
        cursor = self.sqlite_conn.cursor()

        # Tabla de sesiones (equivalente al CREATE TABLE sessions de la lección)
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS sessions (
                session_id   TEXT PRIMARY KEY,
                created_at   TEXT NOT NULL,
                model_name   TEXT NOT NULL,
                description  TEXT
            )
        """)

        # Tabla de mensajes con métricas (equivalente a conversation_turns)
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS messages (
                message_id   TEXT PRIMARY KEY,
                session_id   TEXT NOT NULL,
                timestamp    TEXT NOT NULL,
                role         TEXT NOT NULL CHECK(role IN ('user','assistant','system')),
                content      TEXT NOT NULL,
                token_count  INTEGER DEFAULT 0,
                turn_index   INTEGER NOT NULL DEFAULT 0,
                FOREIGN KEY (session_id) REFERENCES sessions(session_id)
            )
        """)

        # Índices para consultas frecuentes (patrón de la lección 6.1)
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_messages_session
            ON messages(session_id, turn_index)
        """)

        self.sqlite_conn.commit()
        print("[ConversationStore] Esquema SQLite inicializado correctamente.")
```

#### Salida Esperada al Importar el Módulo

```
[ConversationStore] SQLite: conversation_logs.db
[ConversationStore] ChromaDB: ./chroma_db
[ConversationStore] Documentos en memoria: 0
[ConversationStore] Esquema SQLite inicializado correctamente.
```

#### Verificación

```bash
# Verifica que los archivos de base de datos se crean al instanciar la clase
python -c "
from semantic_memory_chat import ConversationStore
store = ConversationStore()
print('Tablas SQLite:', [row[0] for row in store.sqlite_conn.execute(\"SELECT name FROM sqlite_master WHERE type='table'\").fetchall()])
"
```

**Salida esperada:**
```
Tablas SQLite: ['sessions', 'messages']
```

---

### Paso 2: Implementar `add_message()` — Escritura Dual Simultánea

**Objetivo:** Implementar el método que persiste cada mensaje en ambos backends (SQLite y ChromaDB) de forma atómica, garantizando consistencia entre la capa de logging y la capa semántica.

#### Instrucciones

1. Agrega los siguientes métodos a la clase `ConversationStore` (dentro de la misma clase, después de `_init_sqlite_schema`):

```python
    def create_session(self, description: str = "") -> str:
        """
        Registra una nueva sesión de conversación en SQLite.
        Retorna el session_id generado (UUID4).
        """
        session_id = str(uuid.uuid4())
        now = datetime.now(timezone.utc).isoformat()

        self.sqlite_conn.execute(
            "INSERT INTO sessions (session_id, created_at, model_name, description) VALUES (?, ?, ?, ?)",
            (session_id, now, CHAT_MODEL, description)
        )
        self.sqlite_conn.commit()
        return session_id

    def add_message(self, session_id: str, role: str, content: str,
                    token_count: int = 0) -> str:
        """
        Persiste un mensaje en AMBOS backends simultáneamente:
          1. SQLite  → log estructurado con metadata completa
          2. ChromaDB → embedding vectorial para búsqueda semántica

        Args:
            session_id:  Identificador de la sesión activa
            role:        'user' | 'assistant' | 'system'
            content:     Texto del mensaje
            token_count: Tokens consumidos (0 si no aplica)

        Returns:
            message_id: UUID del mensaje recién creado
        """
        message_id = str(uuid.uuid4())
        now = datetime.now(timezone.utc).isoformat()

        # ── 1. Calcular turn_index (posición en la sesión) ─────────────
        row = self.sqlite_conn.execute(
            "SELECT COUNT(*) FROM messages WHERE session_id = ?",
            (session_id,)
        ).fetchone()
        turn_index = row[0]

        # ── 2. Persistir en SQLite (capa de logging estructurado) ──────
        self.sqlite_conn.execute(
            """
            INSERT INTO messages
                (message_id, session_id, timestamp, role, content, token_count, turn_index)
            VALUES (?, ?, ?, ?, ?, ?, ?)
            """,
            (message_id, session_id, now, role, content, token_count, turn_index)
        )
        self.sqlite_conn.commit()

        # ── 3. Persistir en ChromaDB (capa semántica) ──────────────────
        # El documento incluye metadatos ricos para filtrado posterior
        self.collection.add(
            ids=[message_id],
            documents=[content],
            metadatas=[{
                "session_id":  session_id,
                "role":        role,
                "timestamp":   now,
                "turn_index":  turn_index,
                "token_count": token_count
            }]
        )

        return message_id

    def get_window_messages(self, session_id: str, n: int = WINDOW_SIZE) -> List[dict]:
        """
        Recupera los últimos N mensajes de la sesión desde SQLite.
        Estos forman la 'ventana deslizante' de contexto inmediato.

        Returns:
            Lista de dicts con keys: role, content
        """
        rows = self.sqlite_conn.execute(
            """
            SELECT role, content
            FROM messages
            WHERE session_id = ?
            ORDER BY turn_index DESC
            LIMIT ?
            """,
            (session_id, n)
        ).fetchall()

        # Invertir para orden cronológico (más antiguo primero)
        return [{"role": row["role"], "content": row["content"]}
                for row in reversed(rows)]
```

#### Verificación

```bash
python -c "
from semantic_memory_chat import ConversationStore
store = ConversationStore()
sid = store.create_session('Test de escritura dual')
mid = store.add_message(sid, 'user', 'Hola, ¿cómo funciona ChromaDB?', token_count=12)
print('message_id:', mid)
print('Documentos en ChromaDB:', store.collection.count())
print('Mensajes en SQLite:', store.sqlite_conn.execute('SELECT COUNT(*) FROM messages').fetchone()[0])
"
```

**Salida esperada:**
```
message_id: <uuid>
Documentos en ChromaDB: 1
Mensajes en SQLite: 1
```

---

### Paso 3: Implementar `retrieve_relevant_context()` — Búsqueda Semántica

**Objetivo:** Construir la función de recuperación que busca en ChromaDB los mensajes históricos más similares semánticamente al query actual, **excluyendo** los últimos `WINDOW_SIZE` mensajes para evitar duplicación con la ventana deslizante.

#### Instrucciones

1. Agrega el siguiente método a la clase `ConversationStore`:

```python
    def retrieve_relevant_context(self, query: str, session_id: str,
                                   k: int = SEMANTIC_K) -> List[dict]:
        """
        Busca en ChromaDB los mensajes históricos más similares al query.

        IMPORTANTE: Excluye los últimos WINDOW_SIZE mensajes para evitar
        duplicar el contexto con la ventana deslizante.

        Args:
            query:      Texto del mensaje actual del usuario
            session_id: Sesión actual (filtra solo mensajes de esta sesión)
            k:          Número máximo de fragmentos a recuperar

        Returns:
            Lista de dicts con keys: content, role, turn_index, distance
        """
        total_messages = self.sqlite_conn.execute(
            "SELECT COUNT(*) FROM messages WHERE session_id = ?",
            (session_id,)
        ).fetchone()[0]

        # Si no hay suficiente historial, no hay contexto semántico útil
        if total_messages <= WINDOW_SIZE:
            return []

        # Calcular el turn_index máximo a excluir (ventana deslizante)
        exclude_from_index = total_messages - WINDOW_SIZE

        # ── Consulta semántica en ChromaDB con filtro de metadata ──────
        # Filtra por session_id Y por turn_index menor al umbral de exclusión
        results = self.collection.query(
            query_texts=[query],
            n_results=min(k, exclude_from_index),  # No pedir más de lo disponible
            where={
                "$and": [
                    {"session_id": {"$eq": session_id}},
                    {"turn_index": {"$lt": exclude_from_index}}
                ]
            },
            include=["documents", "metadatas", "distances"]
        )

        # ── Formatear resultados ───────────────────────────────────────
        context_fragments = []
        if results["documents"] and results["documents"][0]:
            for doc, meta, dist in zip(
                results["documents"][0],
                results["metadatas"][0],
                results["distances"][0]
            ):
                context_fragments.append({
                    "content":    doc,
                    "role":       meta.get("role", "unknown"),
                    "turn_index": meta.get("turn_index", 0),
                    "distance":   round(dist, 4)  # Distancia coseno (menor = más similar)
                })

        # Ordenar por turno (cronológico) para coherencia narrativa
        context_fragments.sort(key=lambda x: x["turn_index"])
        return context_fragments
```

#### Verificación

```bash
python -c "
from semantic_memory_chat import ConversationStore, WINDOW_SIZE
store = ConversationStore()
sid = store.create_session('Test búsqueda semántica')

# Agregar más mensajes que WINDOW_SIZE para que la búsqueda sea posible
mensajes = [
    ('user', 'Mi nombre es Carlos y trabajo en DevOps'),
    ('assistant', 'Hola Carlos, es un placer. ¿En qué puedo ayudarte con DevOps?'),
    ('user', 'Tengo problemas con mis pipelines de CI/CD en GitLab'),
    ('assistant', 'Los pipelines de GitLab CI/CD pueden fallar por varias razones...'),
    ('user', 'También uso Kubernetes para orquestar mis contenedores'),
    ('assistant', 'Kubernetes es excelente para orquestación. ¿Usas Helm para gestionar charts?'),
    ('user', '¿Qué herramientas recomiendas para monitoreo?'),
    ('assistant', 'Para monitoreo en Kubernetes recomiendo Prometheus y Grafana.'),
]
for role, content in mensajes:
    store.add_message(sid, role, content)

# Buscar contexto relevante para una pregunta sobre el nombre del usuario
resultados = store.retrieve_relevant_context(
    query='¿Cuál es el nombre del usuario?',
    session_id=sid,
    k=3
)
print(f'Fragmentos recuperados: {len(resultados)}')
for r in resultados:
    print(f'  [turno {r[\"turn_index\"]}] ({r[\"role\"]}) dist={r[\"distance\"]}: {r[\"content\"][:60]}...')
"
```

**Salida esperada (los valores de distancia variarán):**
```
Fragmentos recuperados: 3
  [turno 0] (user) dist=0.1823: Mi nombre es Carlos y trabajo en DevOps...
  [turno 1] (assistant) dist=0.2341: Hola Carlos, es un placer. ¿En qué puedo ayudarte con DevOps?...
  [turno 2] (user) dist=0.3102: Tengo problemas con mis pipelines de CI/CD en GitLab...
```

---

### Paso 4: Implementar `build_prompt_with_memory()` — Construcción del Prompt Enriquecido

**Objetivo:** Combinar el system prompt, los fragmentos de memoria semántica y la ventana deslizante en una lista de mensajes lista para enviar a la API de OpenAI.

#### Instrucciones

1. Agrega los siguientes métodos a la clase `ConversationStore` y la función de llamada al LLM:

```python
    def build_prompt_with_memory(self, current_message: str,
                                  session_id: str) -> List[dict]:
        """
        Construye la lista de mensajes para la API de OpenAI combinando:
          1. System prompt con instrucciones base
          2. Contexto semántico recuperado (memoria a largo plazo)
          3. Ventana deslizante de los últimos WINDOW_SIZE mensajes
          4. Mensaje actual del usuario

        La estructura resultante sigue el formato de la API de OpenAI:
          [{"role": "system", "content": "..."},
           {"role": "user",   "content": "..."},
           ...]

        Returns:
            Lista de dicts con keys: role, content
        """
        messages = []

        # ── 1. System prompt base ──────────────────────────────────────
        system_content = (
            "Eres un asistente técnico experto. Tienes acceso a fragmentos "
            "de conversaciones previas que pueden ser relevantes para la "
            "pregunta actual. Úsalos para dar respuestas más coherentes y "
            "personalizadas. Si el contexto previo no es relevante, ignóralo."
        )

        # ── 2. Recuperar contexto semántico ────────────────────────────
        semantic_fragments = self.retrieve_relevant_context(
            query=current_message,
            session_id=session_id,
            k=SEMANTIC_K
        )

        # Inyectar fragmentos semánticos en el system prompt
        if semantic_fragments:
            context_text = "\n\n--- Relevant past context ---\n"
            for frag in semantic_fragments:
                context_text += f"[Turno {frag['turn_index']} - {frag['role']}]: {frag['content']}\n"
            context_text += "--- End of past context ---"
            system_content += context_text

        messages.append({"role": "system", "content": system_content})

        # ── 3. Ventana deslizante (últimos WINDOW_SIZE mensajes) ───────
        window = self.get_window_messages(session_id, n=WINDOW_SIZE)
        messages.extend(window)

        # ── 4. Mensaje actual del usuario ──────────────────────────────
        # Solo agregar si no está ya en la ventana (evitar duplicado)
        if not window or window[-1]["content"] != current_message:
            messages.append({"role": "user", "content": current_message})

        return messages

    def get_session_stats(self, session_id: str) -> dict:
        """
        Retorna estadísticas de la sesión desde SQLite.
        Útil para el comando /stats en el CLI.
        """
        row = self.sqlite_conn.execute(
            """
            SELECT
                COUNT(*)                                    AS total_messages,
                SUM(CASE WHEN role='user' THEN 1 ELSE 0 END) AS user_messages,
                SUM(token_count)                            AS total_tokens,
                MIN(timestamp)                              AS first_message,
                MAX(timestamp)                              AS last_message
            FROM messages
            WHERE session_id = ?
            """,
            (session_id,)
        ).fetchone()

        return {
            "total_messages": row["total_messages"],
            "user_messages":  row["user_messages"],
            "total_tokens":   row["total_tokens"] or 0,
            "first_message":  row["first_message"],
            "last_message":   row["last_message"],
            "chroma_docs":    self.collection.count()
        }

    def close(self):
        """Cierra la conexión SQLite limpiamente."""
        self.sqlite_conn.close()


# ══════════════════════════════════════════════════════════════════════════
# FUNCIÓN DE LLAMADA AL LLM
# ══════════════════════════════════════════════════════════════════════════

def chat_with_memory(store: ConversationStore, session_id: str,
                     user_message: str) -> tuple[str, int, List[dict]]:
    """
    Envía un mensaje al LLM usando el prompt enriquecido con memoria semántica.

    Returns:
        Tuple de (respuesta_texto, tokens_usados, fragmentos_semanticos)
    """
    # Construir prompt con memoria
    messages = store.build_prompt_with_memory(user_message, session_id)

    # Recuperar fragmentos para el comando /memory
    semantic_frags = store.retrieve_relevant_context(user_message, session_id)

    # Persistir mensaje del usuario ANTES de llamar al LLM
    store.add_message(session_id, "user", user_message)

    # Llamada a la API de OpenAI
    inicio = time.time()
    response = openai_client.chat.completions.create(
        model=CHAT_MODEL,
        messages=messages,
        temperature=0.7,
        max_tokens=500
    )
    latencia_ms = int((time.time() - inicio) * 1000)

    assistant_content = response.choices[0].message.content
    total_tokens = response.usage.total_tokens

    # Persistir respuesta del asistente con conteo de tokens
    store.add_message(session_id, "assistant", assistant_content,
                      token_count=total_tokens)

    print(f"  [debug] Latencia: {latencia_ms}ms | Tokens: {total_tokens} | "
          f"Fragmentos semánticos: {len(semantic_frags)}")

    return assistant_content, total_tokens, semantic_frags
```

#### Verificación

```bash
python -c "
from semantic_memory_chat import ConversationStore, chat_with_memory
store = ConversationStore()
sid = store.create_session('Test build_prompt')
# Agregar contexto previo
store.add_message(sid, 'user', 'Me llamo Ana y soy desarrolladora Python')
store.add_message(sid, 'assistant', 'Hola Ana, encantado de conocerte.')
store.add_message(sid, 'user', 'Trabajo con FastAPI y PostgreSQL')
store.add_message(sid, 'assistant', 'FastAPI es excelente para APIs REST.')
store.add_message(sid, 'user', 'También me interesa ChromaDB para RAG')
store.add_message(sid, 'assistant', 'ChromaDB es ideal para proyectos RAG.')

messages = store.build_prompt_with_memory('¿Qué tecnologías mencioné antes?', sid)
print('Número de mensajes en el prompt:', len(messages))
print('Roles:', [m['role'] for m in messages])
print('System prompt contiene contexto semántico:', 'Relevant past context' in messages[0]['content'])
store.close()
"
```

**Salida esperada:**
```
Número de mensajes en el prompt: 8
Roles: ['system', 'user', 'assistant', 'user', 'assistant', 'user', 'assistant', 'user']
System prompt contiene contexto semántico: True
```

---

### Paso 5: Construir el Loop Principal del CLI

**Objetivo:** Implementar el loop interactivo de chat con soporte para comandos especiales (`/memory`, `/stats`, `/new`, `/quit`).

#### Instrucciones

1. Agrega el loop principal al final del archivo `semantic_memory_chat.py`:

```python
# ══════════════════════════════════════════════════════════════════════════
# LOOP PRINCIPAL DEL CLI
# ══════════════════════════════════════════════════════════════════════════

def print_separator(char: str = "─", width: int = 60):
    print(char * width)

def print_semantic_fragments(fragments: List[dict]):
    """Muestra los fragmentos de memoria semántica recuperados."""
    if not fragments:
        print("  (No se recuperaron fragmentos semánticos — historial insuficiente)")
        return
    print(f"\n  🧠 Memoria semántica recuperada ({len(fragments)} fragmentos):")
    for i, frag in enumerate(fragments, 1):
        preview = frag['content'][:80] + "..." if len(frag['content']) > 80 else frag['content']
        print(f"  [{i}] Turno {frag['turn_index']} ({frag['role']}) "
              f"dist={frag['distance']}: {preview}")

def main():
    """Loop principal del chatbot con memoria semántica."""
    print("\n" + "═" * 60)
    print("  🤖 Chatbot con Memoria Semántica — Lab 06-00-01")
    print("  Comandos: /memory | /stats | /new | /quit")
    print("═" * 60)

    store = ConversationStore()
    session_id = store.create_session("Sesión interactiva")
    last_fragments: List[dict] = []

    print(f"\n✅ Nueva sesión iniciada: {session_id[:8]}...")
    print(f"   Modelo: {CHAT_MODEL} | Embeddings: {EMBEDDING_MODEL}")
    print(f"   Ventana deslizante: {WINDOW_SIZE} mensajes | Recuperación semántica: k={SEMANTIC_K}")
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

            # ── Comandos especiales ────────────────────────────────────
            if user_input.lower() == "/quit":
                print("👋 ¡Hasta luego!")
                break

            elif user_input.lower() == "/new":
                session_id = store.create_session("Nueva sesión interactiva")
                last_fragments = []
                print(f"✅ Nueva sesión iniciada: {session_id[:8]}...")
                continue

            elif user_input.lower() == "/memory":
                print_separator("·")
                print_semantic_fragments(last_fragments)
                print_separator("·")
                continue

            elif user_input.lower() == "/stats":
                stats = store.get_session_stats(session_id)
                print_separator("·")
                print(f"  📊 Estadísticas de la sesión {session_id[:8]}...")
                print(f"     Mensajes totales : {stats['total_messages']}")
                print(f"     Turnos del usuario: {stats['user_messages']}")
                print(f"     Tokens consumidos : {stats['total_tokens']}")
                print(f"     Docs en ChromaDB  : {stats['chroma_docs']}")
                print_separator("·")
                continue

            # ── Procesamiento normal del mensaje ───────────────────────
            print("🤔 Pensando...", end="", flush=True)

            try:
                response, tokens, last_fragments = chat_with_memory(
                    store, session_id, user_input
                )
                print(f"\r🤖 Asistente: {response}")

            except Exception as e:
                print(f"\r❌ Error al procesar el mensaje: {e}")
                print("   Intenta de nuevo o escribe /quit para salir.")

    finally:
        store.close()
        print("\n💾 Sesión guardada. Datos persistidos en SQLite y ChromaDB.")

if __name__ == "__main__":
    main()
```

#### Salida Esperada al Iniciar

```
════════════════════════════════════════════════════════════
  🤖 Chatbot con Memoria Semántica — Lab 06-00-01
  Comandos: /memory | /stats | /new | /quit
════════════════════════════════════════════════════════════
[ConversationStore] SQLite: conversation_logs.db
[ConversationStore] ChromaDB: ./chroma_db
[ConversationStore] Documentos en memoria: 0
[ConversationStore] Esquema SQLite inicializado correctamente.

✅ Nueva sesión iniciada: a3f8b2c1...
   Modelo: gpt-4o-mini | Embeddings: text-embedding-3-small
   Ventana deslizante: 5 mensajes | Recuperación semántica: k=3
────────────────────────────────────────────────────────────

👤 Tú: 
```

---

### Paso 6: Script de Demostración con Conversación Larga

**Objetivo:** Crear el script `demo_semantic_memory.py` que simula una conversación de 30+ turnos sobre un tema técnico y luego demuestra la superioridad de la memoria semántica al hacer preguntas que requieren información de los primeros turnos.

#### Instrucciones

1. Crea el archivo de demostración:

```bash
touch demo_semantic_memory.py
```

2. Escribe el siguiente contenido:

```python
"""
demo_semantic_memory.py
Lab 06-00-01: Demostración de memoria semántica vs. ventana deslizante

Simula una conversación técnica larga (30+ turnos) y luego prueba
si el chatbot puede recordar información de los primeros turnos.
"""

import time
from semantic_memory_chat import ConversationStore, chat_with_memory, WINDOW_SIZE

# ── Conversación de demostración (30 turnos) ───────────────────────────────
# Tema: Arquitectura de un sistema de recomendaciones con ML
DEMO_CONVERSATION = [
    # Turnos 1-5: Presentación del proyecto y contexto personal
    "Hola, me llamo Sofía y soy ML Engineer en una startup de e-commerce.",
    "Estamos construyendo un sistema de recomendaciones de productos para nuestra tienda.",
    "El stack técnico que usamos es Python 3.11, FastAPI, PostgreSQL y Redis.",
    "Tenemos aproximadamente 500,000 productos en nuestro catálogo.",
    "Nuestros usuarios hacen en promedio 3 compras al mes.",

    # Turnos 6-10: Detalles del modelo de recomendaciones
    "Estamos evaluando usar embeddings de productos para las recomendaciones.",
    "¿Qué dimensión de embedding recomiendas para 500k productos?",
    "También queremos incorporar el historial de compras del usuario.",
    "El presupuesto de infraestructura es de $2,000 USD mensuales.",
    "Nuestro SLA requiere respuestas en menos de 100ms.",

    # Turnos 11-15: Discusión de bases de datos vectoriales
    "¿Cuál base de datos vectorial recomiendas para este volumen?",
    "Hemos evaluado Pinecone, Weaviate y ChromaDB.",
    "El equipo tiene más experiencia con soluciones self-hosted.",
    "¿Qué tan difícil es escalar ChromaDB a producción?",
    "¿Qdrant sería una mejor opción que ChromaDB para producción?",

    # Turnos 16-20: Pipeline de entrenamiento
    "¿Cómo estructurarías el pipeline de generación de embeddings?",
    "Tenemos acceso a GPUs NVIDIA A100 en AWS.",
    "¿Cada cuánto deberíamos re-entrenar los embeddings?",
    "¿Cómo manejarías los cold start para nuevos productos?",
    "El equipo de datos tiene experiencia con PyTorch y Hugging Face.",

    # Turnos 21-25: Evaluación y métricas
    "¿Qué métricas usarías para evaluar el sistema de recomendaciones?",
    "Nuestro click-through rate actual es del 2.3%.",
    "¿Cómo implementarías A/B testing para las recomendaciones?",
    "¿Qué es el NDCG y cómo se calcula?",
    "¿Cómo balancearías exploración vs. explotación en las recomendaciones?",

    # Turnos 26-30: Preguntas de cierre técnico
    "¿Cómo manejarías la privacidad de datos de los usuarios?",
    "¿Qué estrategia usarías para el caché de recomendaciones en Redis?",
    "¿Cómo monitorearías el drift del modelo en producción?",
    "¿Qué herramientas de MLOps recomiendas para este proyecto?",
    "Dame un resumen de los puntos más importantes que discutimos.",
]

# ── Preguntas de prueba que requieren memoria de los primeros turnos ───────
MEMORY_TEST_QUESTIONS = [
    {
        "question": "¿Cuál es el nombre de la persona con quien estoy hablando y en qué empresa trabaja?",
        "requires_turn": 0,  # Turno donde se mencionó esta info
        "expected_info": "Sofía, ML Engineer, e-commerce"
    },
    {
        "question": "¿Cuántos productos tiene el catálogo y cuál es el presupuesto mensual de infraestructura?",
        "requires_turn": 3,
        "expected_info": "500,000 productos, $2,000 USD"
    },
    {
        "question": "¿Qué bases de datos vectoriales se evaluaron para el proyecto?",
        "requires_turn": 11,
        "expected_info": "Pinecone, Weaviate, ChromaDB"
    },
    {
        "question": "¿Qué tipo de GPUs tienen disponibles y en qué proveedor cloud?",
        "requires_turn": 16,
        "expected_info": "NVIDIA A100, AWS"
    },
    {
        "question": "¿Cuál era el click-through rate inicial del sistema?",
        "requires_turn": 21,
        "expected_info": "2.3%"
    },
]


def run_demo():
    """Ejecuta la demostración completa de memoria semántica."""
    print("\n" + "═" * 70)
    print("  🧪 DEMOSTRACIÓN: Memoria Semántica vs. Ventana Deslizante")
    print("  Lab 06-00-01 — Conversación de 30+ turnos")
    print("═" * 70)

    store = ConversationStore()
    session_id = store.create_session("Demo memoria semántica — 30 turnos")

    print(f"\n📋 Sesión de demostración: {session_id[:8]}...")
    print(f"   Ventana deslizante: últimos {WINDOW_SIZE} mensajes")
    print(f"   Total de turnos a simular: {len(DEMO_CONVERSATION)}")
    print("\n⏳ Simulando conversación larga...\n")

    # ── Fase 1: Simular la conversación larga ──────────────────────────────
    for i, user_msg in enumerate(DEMO_CONVERSATION, 1):
        print(f"  Turno {i:2d}/{len(DEMO_CONVERSATION)}: {user_msg[:55]}...", end="")

        try:
            response, tokens, _ = chat_with_memory(store, session_id, user_msg)
            print(f" ✓ ({tokens} tokens)")
            # Pequeña pausa para no saturar la API
            time.sleep(0.5)
        except Exception as e:
            print(f" ✗ Error: {e}")
            time.sleep(2)  # Esperar más en caso de rate limit

    stats = store.get_session_stats(session_id)
    print(f"\n✅ Conversación completada:")
    print(f"   Mensajes en SQLite : {stats['total_messages']}")
    print(f"   Docs en ChromaDB   : {stats['chroma_docs']}")
    print(f"   Tokens consumidos  : {stats['total_tokens']}")

    # ── Fase 2: Pruebas de memoria ─────────────────────────────────────────
    print("\n" + "═" * 70)
    print("  🔍 PRUEBAS DE RECUPERACIÓN DE MEMORIA")
    print("  Estas preguntas requieren información de los primeros turnos")
    print("  (fuera de la ventana deslizante de 5 mensajes)")
    print("═" * 70)

    resultados = []
    for test in MEMORY_TEST_QUESTIONS:
        print(f"\n❓ Pregunta: {test['question']}")
        print(f"   (Información en turno ~{test['requires_turn']+1}, "
              f"fuera de la ventana actual)")

        response, tokens, fragments = chat_with_memory(
            store, session_id, test["question"]
        )

        print(f"🤖 Respuesta: {response[:200]}...")
        print(f"\n   🧠 Fragmentos semánticos recuperados: {len(fragments)}")
        for frag in fragments:
            print(f"      [Turno {frag['turn_index']}] dist={frag['distance']}: "
                  f"{frag['content'][:60]}...")

        # Verificar si la respuesta contiene la información esperada
        contains_expected = any(
            keyword.lower() in response.lower()
            for keyword in test["expected_info"].split(", ")
        )
        resultados.append(contains_expected)
        print(f"   {'✅ CORRECTO' if contains_expected else '⚠️  PARCIAL'}: "
              f"Información esperada: {test['expected_info']}")

        time.sleep(0.5)

    # ── Resumen final ──────────────────────────────────────────────────────
    print("\n" + "═" * 70)
    print("  📊 RESUMEN DE LA DEMOSTRACIÓN")
    print("═" * 70)
    correctas = sum(resultados)
    print(f"\n  Preguntas respondidas correctamente: {correctas}/{len(resultados)}")
    print(f"  Tasa de recuperación: {correctas/len(resultados)*100:.0f}%")
    print(f"\n  💡 Conclusión:")
    print(f"     La memoria semántica permite recuperar información de los")
    print(f"     turnos 1-{len(DEMO_CONVERSATION)-WINDOW_SIZE}, que están FUERA")
    print(f"     de la ventana deslizante de {WINDOW_SIZE} mensajes.")
    print(f"     Sin ChromaDB, estas preguntas no podrían responderse correctamente.")

    store.close()


if __name__ == "__main__":
    run_demo()
```

#### Instrucciones de Ejecución

```bash
# Ejecutar el script de demostración
python demo_semantic_memory.py
```

> ⏱️ **Nota:** La demostración completa tarda aproximadamente 20-25 minutos debido a los 30 turnos de conversación. Puedes reducir `DEMO_CONVERSATION` a los primeros 15 elementos para una prueba más rápida.

#### Salida Esperada (fragmento)

```
══════════════════════════════════════════════════════════════════════
  🧪 DEMOSTRACIÓN: Memoria Semántica vs. Ventana Deslizante
  Lab 06-00-01 — Conversación de 30+ turnos
══════════════════════════════════════════════════════════════════════

📋 Sesión de demostración: b7d2a4f1...
   Ventana deslizante: últimos 5 mensajes
   Total de turnos a simular: 30

⏳ Simulando conversación larga...

  Turno  1/30: Hola, me llamo Sofía y soy ML Engineer en una startup... ✓ (87 tokens)
  Turno  2/30: Estamos construyendo un sistema de recomendaciones de... ✓ (134 tokens)
  ...
  Turno 30/30: Dame un resumen de los puntos más importantes que disc... ✓ (312 tokens)

✅ Conversación completada:
   Mensajes en SQLite : 60
   Docs en ChromaDB   : 60
   Tokens consumidos  : 8432

══════════════════════════════════════════════════════════════════════
  🔍 PRUEBAS DE RECUPERACIÓN DE MEMORIA
══════════════════════════════════════════════════════════════════════

❓ Pregunta: ¿Cuál es el nombre de la persona con quien estoy hablando...
   🧠 Fragmentos semánticos recuperados: 3
      [Turno 0] dist=0.1234: Hola, me llamo Sofía y soy ML Engineer...
   ✅ CORRECTO: Información esperada: Sofía, ML Engineer, e-commerce
```

---

## Validación y Pruebas

### Prueba de Integración Completa

Ejecuta la siguiente secuencia de verificación para confirmar que todos los componentes funcionan correctamente:

```bash
python -c "
import os
import sqlite3
import chromadb
from semantic_memory_chat import ConversationStore, chat_with_memory, WINDOW_SIZE

print('=== VALIDACIÓN COMPLETA Lab 06-00-01 ===\n')

store = ConversationStore()
sid = store.create_session('Sesión de validación')

# TEST 1: Escritura dual
print('TEST 1: Escritura dual (SQLite + ChromaDB)')
for i, (role, content) in enumerate([
    ('user', 'Soy ingeniero de software y me especializo en Python'),
    ('assistant', 'Excelente, Python es un lenguaje muy versátil.'),
    ('user', 'Trabajo principalmente con APIs REST y microservicios'),
    ('assistant', 'Los microservicios son una arquitectura muy popular.'),
    ('user', 'Mi stack favorito es FastAPI + PostgreSQL + Redis'),
    ('assistant', 'FastAPI es excelente para APIs de alto rendimiento.'),
    ('user', 'También tengo experiencia con Docker y Kubernetes'),
    ('assistant', 'Docker y Kubernetes son esenciales en producción.'),
]):
    store.add_message(sid, role, content)

sqlite_count = store.sqlite_conn.execute(
    'SELECT COUNT(*) FROM messages WHERE session_id = ?', (sid,)
).fetchone()[0]
chroma_count = store.collection.count()
assert sqlite_count == 8, f'SQLite: esperado 8, obtenido {sqlite_count}'
assert chroma_count >= 8, f'ChromaDB: esperado >= 8, obtenido {chroma_count}'
print(f'  ✅ SQLite: {sqlite_count} mensajes | ChromaDB: {chroma_count} docs\n')

# TEST 2: Ventana deslizante
print('TEST 2: Ventana deslizante')
window = store.get_window_messages(sid, n=WINDOW_SIZE)
assert len(window) == WINDOW_SIZE, f'Ventana: esperado {WINDOW_SIZE}, obtenido {len(window)}'
assert window[-1]['content'] == 'Docker y Kubernetes son esenciales en producción.'
print(f'  ✅ Ventana de {len(window)} mensajes, último: \"{window[-1][\"content\"][:40]}...\"\n')

# TEST 3: Recuperación semántica (excluye ventana)
print('TEST 3: Recuperación semántica')
frags = store.retrieve_relevant_context('¿Qué lenguaje de programación usa?', sid, k=3)
assert len(frags) > 0, 'Debería recuperar al menos 1 fragmento'
# Verificar que ningún fragmento es de los últimos WINDOW_SIZE mensajes
total = store.sqlite_conn.execute(
    'SELECT COUNT(*) FROM messages WHERE session_id = ?', (sid,)
).fetchone()[0]
exclude_from = total - WINDOW_SIZE
for frag in frags:
    assert frag['turn_index'] < exclude_from, \
        f'Fragmento del turno {frag[\"turn_index\"]} está dentro de la ventana!'
print(f'  ✅ {len(frags)} fragmentos recuperados, todos fuera de la ventana deslizante\n')

# TEST 4: Construcción del prompt
print('TEST 4: Construcción del prompt enriquecido')
messages = store.build_prompt_with_memory('¿Qué tecnologías mencioné?', sid)
assert messages[0]['role'] == 'system', 'El primer mensaje debe ser system'
has_context = 'Relevant past context' in messages[0]['content']
print(f'  ✅ Prompt con {len(messages)} mensajes | Contexto semántico inyectado: {has_context}\n')

# TEST 5: Estadísticas SQLite
print('TEST 5: Estadísticas de sesión')
stats = store.get_session_stats(sid)
assert stats['total_messages'] == 8
assert stats['user_messages'] == 4
print(f'  ✅ {stats[\"total_messages\"]} mensajes | {stats[\"user_messages\"]} del usuario\n')

store.close()
print('=== TODOS LOS TESTS PASARON ✅ ===')
"
```

**Salida esperada:**
```
=== VALIDACIÓN COMPLETA Lab 06-00-01 ===

TEST 1: Escritura dual (SQLite + ChromaDB)
  ✅ SQLite: 8 mensajes | ChromaDB: 8 docs

TEST 2: Ventana deslizante
  ✅ Ventana de 5 mensajes, último: "Docker y Kubernetes son esenciales en..."

TEST 3: Recuperación semántica
  ✅ 3 fragmentos recuperados, todos fuera de la ventana deslizante

TEST 4: Construcción del prompt enriquecido
  ✅ Prompt con 8 mensajes | Contexto semántico inyectado: True

TEST 5: Estadísticas de sesión
  ✅ 8 mensajes | 4 del usuario

=== TODOS LOS TESTS PASARON ✅ ===
```

### Prueba de Persistencia Entre Sesiones

```bash
# Verificar que los datos persisten al reiniciar la aplicación
python -c "
from semantic_memory_chat import ConversationStore
store = ConversationStore()
stats_global = store.sqlite_conn.execute(
    'SELECT COUNT(*) AS total FROM messages'
).fetchone()[0]
print(f'Mensajes persistidos en SQLite: {stats_global}')
print(f'Documentos en ChromaDB: {store.collection.count()}')
store.close()
"
```

---

## Solución de Problemas

### Problema 1: `chromadb.errors.InvalidDimensionException` al consultar la colección

**Síntoma:**
```
chromadb.errors.InvalidDimensionException: Embedding dimension 1536 does not match collection dimensionality 768
```

**Causa:** La colección `conversation_memory` en ChromaDB fue creada previamente con un modelo de embedding diferente (por ejemplo, `text-embedding-ada-002` con 1536 dimensiones vs. `text-embedding-3-small` con 1536 dimensiones pero con configuración distinta), o el directorio `./chroma_db` contiene datos de una ejecución anterior del Lab 05 con otro modelo.

**Solución:**

```bash
# 1. Eliminar la base de datos vectorial existente
rm -rf ./chroma_db

# 2. Eliminar también la base SQLite si hay inconsistencia
rm -f conversation_logs.db

# 3. Reiniciar la aplicación — se crearán nuevas bases de datos limpias
python -c "from semantic_memory_chat import ConversationStore; s = ConversationStore(); s.close()"
```

Si el problema persiste, verifica que el modelo de embedding en `EMBEDDING_MODEL` sea consistente:

```bash
python -c "
from semantic_memory_chat import EMBEDDING_MODEL
print('Modelo de embedding configurado:', EMBEDDING_MODEL)
# Debe ser: text-embedding-3-small
"
```

---

### Problema 2: `openai.RateLimitError` durante la demostración de 30 turnos

**Síntoma:**
```
openai.RateLimitError: Error code: 429 - {'error': {'message': 'Rate limit reached for gpt-4o-mini', 'type': 'requests', 'code': 'rate_limit_exceeded'}}
```

**Causa:** El script `demo_semantic_memory.py` realiza 30+ llamadas consecutivas a la API de OpenAI en poco tiempo. Las cuentas con nivel de uso bajo (Tier 1) tienen un límite de 500 RPM (requests per minute) para `gpt-4o-mini`, pero el límite de tokens por minuto (TPM) puede alcanzarse antes con conversaciones largas.

**Solución:**

Modifica el `time.sleep()` en `demo_semantic_memory.py` para aumentar el intervalo entre llamadas:

```python
# En demo_semantic_memory.py, busca las líneas con time.sleep y modifica:

# Opción 1: Aumentar la pausa entre turnos (más lento pero seguro)
time.sleep(2.0)  # Cambiar de 0.5 a 2.0 segundos

# Opción 2: Agregar manejo de reintentos con backoff exponencial
import time

def chat_with_retry(store, session_id, message, max_retries=3):
    """Wrapper con reintentos para manejar rate limits."""
    for attempt in range(max_retries):
        try:
            return chat_with_memory(store, session_id, message)
        except Exception as e:
            if "rate_limit" in str(e).lower() and attempt < max_retries - 1:
                wait_time = (2 ** attempt) * 5  # 5s, 10s, 20s
                print(f"\n  ⏳ Rate limit. Esperando {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise
```

También puedes reducir la demostración a los primeros 15 turnos editando `DEMO_CONVERSATION`:

```python
# En demo_semantic_memory.py, reducir la conversación para pruebas rápidas
DEMO_CONVERSATION = DEMO_CONVERSATION[:15]  # Solo los primeros 15 turnos
```

---

## Limpieza del Entorno

Ejecuta los siguientes comandos para limpiar los archivos generados durante el laboratorio:

```bash
# 1. Eliminar bases de datos locales generadas
rm -rf ./chroma_db
rm -f conversation_logs.db

# 2. Desactivar el entorno virtual
deactivate

# 3. (Opcional) Eliminar el entorno virtual completo
# rm -rf .venv

# 4. Verificar que no queden archivos con API keys
grep -r "sk-" . --include="*.py" --include="*.txt" 2>/dev/null && \
    echo "⚠️  ADVERTENCIA: Posibles API keys encontradas en archivos" || \
    echo "✅ No se encontraron API keys hardcodeadas"
```

> 💡 **Nota:** Los archivos `.env`, `chroma_db/` y `conversation_logs.db` ya están en el `.gitignore` creado al inicio. Verifica antes de hacer cualquier `git commit`.

---

## Resumen

En este laboratorio construiste un chatbot CLI completo que implementa **memoria semántica híbrida** combinando dos estrategias complementarias:

| Estrategia | Backend | Propósito | Alcance |
|---|---|---|---|
| **Ventana deslizante** | SQLite | Contexto inmediato | Últimos 5 mensajes |
| **Memoria semántica** | ChromaDB | Contexto relevante | Todo el historial |

### Conceptos Clave Aplicados

- **Separación de responsabilidades**: SQLite actúa como capa de logging estructurado (auditoría, métricas, trazabilidad) tal como se diseñó en la lección 6.1, mientras ChromaDB provee recuperación semántica. Cada backend hace lo que mejor sabe hacer.

- **Patrón de escritura dual**: `add_message()` persiste cada mensaje en ambos backends simultáneamente, garantizando consistencia entre la capa de logs y la capa vectorial.

- **Exclusión de la ventana activa**: `retrieve_relevant_context()` filtra los últimos `WINDOW_SIZE` mensajes de la búsqueda semántica para evitar duplicar contexto en el prompt, optimizando el uso de tokens.

- **Inyección de contexto etiquetado**: El sistema prompt incluye los fragmentos recuperados bajo la etiqueta `Relevant past context:`, permitiendo al LLM distinguir entre contexto histórico y conversación activa.

- **Logging estructurado con SQLite**: El schema `sessions` + `messages` refleja directamente el diseño de la lección 6.1 (`sessions` + `conversation_turns`), demostrando cómo los patrones relacionales se aplican incluso en bases de datos embebidas.

### Arquitectura Final

```
Usuario
   │
   ▼
[CLI Loop]
   │
   ├─── add_message() ──────┬──► SQLite (logs estructurados)
   │                        └──► ChromaDB (embeddings)
   │
   ├─── build_prompt_with_memory()
   │         │
   │         ├─► retrieve_relevant_context() ──► ChromaDB (búsqueda semántica)
   │         └─► get_window_messages()        ──► SQLite (últimos N mensajes)
   │
   └─── OpenAI API (gpt-4o-mini)
```

### Recursos Adicionales

- [ChromaDB Documentation — Filtering](https://docs.trychroma.com/usage-guide#filtering-by-metadata)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)
- [SQLite Python Tutorial — sqlite3 module](https://docs.python.org/3.11/library/sqlite3.html)
- [Retrieval-Augmented Generation (RAG) — LangChain Docs](https://python.langchain.com/docs/use_cases/question_answering/)

---
LAB_END---
