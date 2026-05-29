---LAB_START---
LAB_ID: 13-00-01
---MARKDOWN---
# Desarrollo de un sistema completo: API + RAG avanzado + Agente funcional + Métricas de evaluación + Documentación técnica

## Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 179 minutos (6 fases de ~25–30 min) |
| **Complejidad** | Alta |
| **Nivel Bloom** | Crear |
| **Costo estimado de API** | ~$2–5 USD (OpenAI GPT-4o-mini + text-embedding-3-small) |
| **Módulo** | 13 — Proyecto Integrador Capstone |

---

## Descripción General

Este lab capstone integra todos los conocimientos del curso en un único sistema GenAI funcional de nivel producción. Construirás un asistente inteligente sobre una base de conocimiento técnica que expone una API REST con FastAPI, procesa documentos mediante un pipeline RAG avanzado con recuperación híbrida y reranking, orquesta respuestas a través de un agente ReAct con memoria persistente, mide la calidad de cada respuesta con métricas RAGAS integradas en LangSmith, y produce documentación técnica profesional completa. Al finalizar dispondrás de un sistema desplegable con Docker Compose que demuestra competencias de nivel intermedio-avanzado en ingeniería de IA Generativa.

---

## Objetivos de Aprendizaje

Al completar este lab serás capaz de:

- [ ] Diseñar e implementar una arquitectura GenAI de cinco capas (API, RAG, Agente, Evaluación, Documentación) con responsabilidades claramente delimitadas entre módulos Python.
- [ ] Construir un pipeline RAG avanzado con chunking semántico, embeddings `text-embedding-3-small`, recuperación híbrida `EnsembleRetriever` (dense + BM25) y reranking por relevancia en ChromaDB.
- [ ] Implementar un agente ReAct con LangChain `AgentExecutor`, herramientas personalizadas (`rag_search`, `calculator`, `document_summarizer`), memoria `ConversationBufferWindowMemory` y manejo de errores con `tenacity`.
- [ ] Ejecutar un benchmark de evaluación con RAGAS (faithfulness, answer_relevancy, context_recall, context_precision) sobre 20 preguntas con ground truth e integrar los resultados como experimento en LangSmith.
- [ ] Producir documentación técnica profesional: diagrama de arquitectura Mermaid, ADR, README completo y reporte de evaluación en Markdown.

---

## Prerrequisitos

### Conocimiento previo

- Haber completado Labs 11 (LangSmith + Observabilidad) y 12 (Docker + Seguridad) de este módulo, o tener experiencia equivalente.
- Python intermedio-avanzado: `async/await`, decoradores, manejo de excepciones tipadas, `typing`.
- Familiaridad con FastAPI: endpoints, middleware, modelos Pydantic v2.
- Comprensión de conceptos RAG: embeddings, vector stores, chunking, retrieval.
- Docker Desktop instalado y funcional (`docker compose version` ≥ 2.x).

### Acceso y credenciales

- **OpenAI API key** con créditos suficientes (~$3–5 USD). Configurar límite de gasto mensual en [platform.openai.com](https://platform.openai.com/account/billing/limits).
- **LangSmith account** activa con `LANGCHAIN_API_KEY` disponible ([smith.langchain.com](https://smith.langchain.com)).
- Acceso a internet estable (≥10 Mbps) para llamadas a APIs y descarga de modelos.

> ⚠️ **Seguridad:** Nunca escribas API keys directamente en el código. Este lab usa exclusivamente archivos `.env` con `python-dotenv`. Verifica que `.env` esté en `.gitignore` antes de hacer cualquier commit.

---

## Entorno del Lab

### Hardware requerido

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 16 GB | 32 GB |
| CPU | 4 núcleos (i5 8va gen) | 8 núcleos |
| Disco libre | 20 GB SSD | 30 GB SSD |
| GPU | No requerida | NVIDIA CUDA 11.8+ (opcional) |

### Software requerido

| Paquete | Versión |
|---|---|
| Python | 3.11.x |
| pip | 23.x o superior |
| Docker Desktop | 4.x (con Compose v2) |
| openai | 1.35.x |
| langchain | 0.2.x |
| langchain-community | 0.2.x |
| langchain-openai | 0.1.x |
| fastapi | 0.111.x |
| uvicorn | 0.30.x |
| pydantic | 2.7.x |
| chromadb | 0.5.x |
| rank-bm25 | 0.2.x |
| ragas | 0.1.x |
| langsmith | 0.1.x |
| tenacity | 8.3.x |
| pypdf | 4.x |
| slowapi | 0.1.x |
| pytest | 8.x |
| httpx | 0.27.x |

### Configuración inicial del entorno

```bash
# 1. Crear directorio del proyecto y entorno virtual aislado
mkdir sistema-genai-capstone && cd sistema-genai-capstone
python3.11 -m venv .venv
source .venv/bin/activate          # macOS/Linux
# .venv\Scripts\activate           # Windows PowerShell

# 2. Crear requirements.txt
cat > requirements.txt << 'EOF'
openai==1.35.3
langchain==0.2.6
langchain-community==0.2.6
langchain-openai==0.1.13
fastapi==0.111.0
uvicorn==0.30.1
pydantic==2.7.4
chromadb==0.5.3
rank-bm25==0.2.2
ragas==0.1.10
langsmith==0.1.77
tenacity==8.3.0
pypdf==4.2.0
slowapi==0.1.9
pytest==8.2.2
httpx==0.27.0
python-dotenv==1.0.1
EOF

pip install -r requirements.txt

# 3. Crear estructura de carpetas del proyecto
mkdir -p api agent rag evaluation docs/adr tests knowledge_base

# 4. Crear .gitignore
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
chroma_db/
*.egg-info/
.pytest_cache/
EOF

# 5. Crear plantilla de variables de entorno
cat > .env.example << 'EOF'
OPENAI_API_KEY=sk-...
LANGCHAIN_API_KEY=ls__...
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=sistema-genai-capstone
API_BEARER_TOKEN=mi-token-secreto-cambiar-en-produccion
CHROMA_PERSIST_DIR=./chroma_db
EOF

cp .env.example .env
echo ">>> Edita .env con tus credenciales reales antes de continuar"
```

---

## Instrucciones Paso a Paso

### Fase 1 — Ingesta de Documentos y Pipeline RAG Avanzado (~30 min)

**Objetivo:** Construir el pipeline de ingesta que procesa documentos Markdown/PDF, genera embeddings con `text-embedding-3-small`, los almacena en ChromaDB con metadatos enriquecidos, y expone un retriever híbrido (dense + BM25) con reranking.

#### Paso 1.1 — Crear la base de conocimiento de prueba

**Instrucciones:**

1. Crea documentos Markdown de ejemplo que servirán como base de conocimiento del asistente (dominio: documentación técnica de sistemas GenAI):

```bash
cat > knowledge_base/rag_concepts.md << 'EOF'
# Conceptos de RAG (Retrieval-Augmented Generation)

## ¿Qué es RAG?
RAG es una técnica que combina la recuperación de información con la generación de texto.
Permite a los modelos de lenguaje acceder a conocimiento externo actualizado sin necesidad
de reentrenamiento. El pipeline básico tiene tres etapas: indexación, recuperación y generación.

## Recuperación Híbrida
La recuperación híbrida combina búsqueda vectorial densa (semántica) con búsqueda dispersa
léxica (BM25). La búsqueda densa captura similitud semántica mientras BM25 captura coincidencias
exactas de términos. La combinación ponderada (ej. 60% densa, 40% BM25) supera a cualquier
método individual en benchmarks estándar como BEIR.

## Chunking Semántico
El chunking divide documentos largos en fragmentos manejables. El RecursiveCharacterTextSplitter
divide por párrafos, luego oraciones, luego palabras. El tamaño óptimo depende del caso de uso:
512 tokens para respuestas precisas, 1024 tokens para síntesis de temas amplios.

## Reranking
El reranking aplica un modelo cross-encoder más preciso (pero más lento) sobre los top-k
resultados del retriever inicial para reordenarlos por relevancia real. Mejora la precisión
en un 15-25% con un overhead de latencia de ~100-300ms.
EOF

cat > knowledge_base/agent_patterns.md << 'EOF'
# Patrones de Agentes con LangChain

## Patrón ReAct
ReAct (Reasoning + Acting) intercala pasos de razonamiento y acción. El agente genera
un pensamiento, decide una acción (herramienta), observa el resultado y repite hasta
tener suficiente información para responder. Es el patrón más robusto para tareas
que requieren múltiples fuentes de información.

## Memoria de Conversación
ConversationBufferWindowMemory mantiene los últimos k turnos de conversación en memoria.
Para k=5, el agente recuerda los últimos 5 intercambios. Es suficiente para la mayoría
de conversaciones sin saturar el contexto del modelo.

## Manejo de Errores en Agentes
Los agentes deben implementar retry logic con backoff exponencial para errores de API
transitorios. La librería tenacity permite configurar @retry con stop_after_attempt y
wait_exponential. Los fallbacks deben retornar mensajes de error claros al usuario.

## Herramientas Personalizadas
Las herramientas son funciones Python decoradas con @tool que el agente puede invocar.
Cada herramienta debe tener un docstring claro que describa cuándo usarla, ya que el
LLM usa ese texto para decidir si invocarla o no.
EOF

cat > knowledge_base/evaluation_metrics.md << 'EOF'
# Métricas de Evaluación para Sistemas RAG

## RAGAS Framework
RAGAS evalúa sistemas RAG con cuatro métricas principales sin necesidad de anotaciones
humanas costosas: faithfulness, answer_relevancy, context_recall y context_precision.

## Faithfulness
Mide si la respuesta generada está respaldada por el contexto recuperado. Una puntuación
de 1.0 indica que cada afirmación de la respuesta puede verificarse en los documentos
recuperados. Valores < 0.7 indican alucinaciones significativas.

## Answer Relevancy
Mide qué tan relevante es la respuesta para la pregunta original. Genera preguntas
hipotéticas desde la respuesta y calcula similitud con la pregunta original.
Valores > 0.8 indican respuestas bien enfocadas.

## Context Precision y Context Recall
Context precision mide si los documentos recuperados son relevantes (sin ruido).
Context recall mide si se recuperaron todos los documentos necesarios para responder.
El balance óptimo depende del caso de uso: sistemas de soporte priorizan recall,
sistemas legales priorizan precision.
EOF
```

2. Verifica que los archivos se crearon correctamente:

```bash
ls -la knowledge_base/
wc -l knowledge_base/*.md
```

**Salida esperada:**
```
knowledge_base/agent_patterns.md
knowledge_base/evaluation_metrics.md
knowledge_base/rag_concepts.md
# Cada archivo debe tener entre 20-35 líneas
```

**Verificación:** `ls knowledge_base/ | wc -l` debe devolver `3`.

---

#### Paso 1.2 — Implementar el pipeline de ingesta y chunking

**Instrucciones:**

1. Crea el módulo de chunking semántico:

```python
# rag/chunker.py
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_core.documents import Document
from pathlib import Path
import logging

logger = logging.getLogger(__name__)

def load_markdown_documents(source_dir: str) -> list[Document]:
    """Carga archivos Markdown desde un directorio como Documents de LangChain."""
    docs = []
    source_path = Path(source_dir)
    for md_file in source_path.glob("**/*.md"):
        text = md_file.read_text(encoding="utf-8")
        doc = Document(
            page_content=text,
            metadata={
                "source": str(md_file),
                "filename": md_file.name,
                "category": md_file.stem,
                "file_type": "markdown",
            }
        )
        docs.append(doc)
        logger.info(f"Cargado: {md_file.name} ({len(text)} chars)")
    return docs

def chunk_documents(
    documents: list[Document],
    chunk_size: int = 512,
    chunk_overlap: int = 64
) -> list[Document]:
    """
    Divide documentos en chunks con RecursiveCharacterTextSplitter.
    Preserva y enriquece los metadatos de cada chunk.
    """
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=chunk_overlap,
        separators=["\n\n", "\n", ". ", " ", ""],
        length_function=len,
    )
    chunks = splitter.split_documents(documents)
    # Enriquecer metadatos con índice de chunk
    for i, chunk in enumerate(chunks):
        chunk.metadata["chunk_index"] = i
        chunk.metadata["chunk_size"] = len(chunk.page_content)
    logger.info(f"Total chunks generados: {len(chunks)}")
    return chunks
```

2. Crea el módulo del pipeline RAG con recuperación híbrida:

```python
# rag/rag_pipeline.py
import os
import logging
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever
from langchain_core.documents import Document
from dotenv import load_dotenv

load_dotenv()
logger = logging.getLogger(__name__)

# Instancia global del retriever (se inicializa una vez)
_retriever_instance = None
_vectorstore_instance = None

def build_retriever(documents: list[Document]) -> EnsembleRetriever:
    """
    Construye un EnsembleRetriever híbrido:
    - 60% búsqueda vectorial densa (text-embedding-3-small)
    - 40% BM25 disperso (léxico)
    """
    global _retriever_instance, _vectorstore_instance

    embeddings = OpenAIEmbeddings(
        model="text-embedding-3-small",
        openai_api_key=os.getenv("OPENAI_API_KEY")
    )

    persist_dir = os.getenv("CHROMA_PERSIST_DIR", "./chroma_db")

    # Retriever denso: ChromaDB
    _vectorstore_instance = Chroma.from_documents(
        documents=documents,
        embedding=embeddings,
        persist_directory=persist_dir,
        collection_name="knowledge_base"
    )
    dense_retriever = _vectorstore_instance.as_retriever(
        search_type="mmr",           # Maximum Marginal Relevance para diversidad
        search_kwargs={"k": 5, "fetch_k": 10}
    )

    # Retriever disperso: BM25
    sparse_retriever = BM25Retriever.from_documents(documents)
    sparse_retriever.k = 5

    # Ensemble: combinación ponderada
    ensemble = EnsembleRetriever(
        retrievers=[dense_retriever, sparse_retriever],
        weights=[0.6, 0.4]
    )
    _retriever_instance = ensemble
    logger.info("Retriever híbrido construido correctamente.")
    return ensemble

def get_retriever() -> EnsembleRetriever:
    """Retorna el retriever ya inicializado o lanza error si no se ha inicializado."""
    if _retriever_instance is None:
        raise RuntimeError(
            "El retriever no ha sido inicializado. "
            "Ejecuta primero el script de ingesta."
        )
    return _retriever_instance

def retrieve_with_metadata_filter(
    query: str,
    category_filter: str | None = None
) -> list[Document]:
    """
    Recupera documentos con filtrado opcional por metadato 'category'.
    Retorna los top-5 documentos más relevantes.
    """
    retriever = get_retriever()
    docs = retriever.invoke(query)

    if category_filter:
        docs = [d for d in docs if d.metadata.get("category") == category_filter]
        logger.info(f"Filtrado por category='{category_filter}': {len(docs)} docs")

    return docs[:5]
```

3. Crea el script de ingesta:

```python
# rag/ingest.py
"""
Script de ingesta: carga documentos, genera chunks y los almacena en ChromaDB.
Uso: python rag/ingest.py --source ./knowledge_base
"""
import argparse
import logging
import sys
from pathlib import Path

# Agregar raíz del proyecto al path
sys.path.insert(0, str(Path(__file__).parent.parent))

from rag.chunker import load_markdown_documents, chunk_documents
from rag.rag_pipeline import build_retriever
from dotenv import load_dotenv

load_dotenv()
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")
logger = logging.getLogger(__name__)

def main(source_dir: str):
    logger.info(f"=== Iniciando ingesta desde: {source_dir} ===")

    # 1. Cargar documentos
    documents = load_markdown_documents(source_dir)
    if not documents:
        logger.error(f"No se encontraron archivos .md en {source_dir}")
        sys.exit(1)
    logger.info(f"Documentos cargados: {len(documents)}")

    # 2. Chunking semántico
    chunks = chunk_documents(documents, chunk_size=512, chunk_overlap=64)
    logger.info(f"Chunks generados: {len(chunks)}")

    # 3. Construir retriever (indexa en ChromaDB)
    retriever = build_retriever(chunks)
    logger.info("=== Ingesta completada exitosamente ===")
    logger.info(f"ChromaDB persistida en: {__import__('os').getenv('CHROMA_PERSIST_DIR', './chroma_db')}")

    return len(chunks)

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Ingestar documentos en ChromaDB")
    parser.add_argument("--source", default="./knowledge_base", help="Directorio fuente")
    args = parser.parse_args()
    main(args.source)
```

4. Ejecuta la ingesta:

```bash
python rag/ingest.py --source ./knowledge_base
```

**Salida esperada:**
```
2024-XX-XX - INFO - === Iniciando ingesta desde: ./knowledge_base ===
2024-XX-XX - INFO - Cargado: agent_patterns.md (XXX chars)
2024-XX-XX - INFO - Cargado: evaluation_metrics.md (XXX chars)
2024-XX-XX - INFO - Cargado: rag_concepts.md (XXX chars)
2024-XX-XX - INFO - Documentos cargados: 3
2024-XX-XX - INFO - Total chunks generados: XX
2024-XX-XX - INFO - Retriever híbrido construido correctamente.
2024-XX-XX - INFO - === Ingesta completada exitosamente ===
```

**Verificación:** `ls chroma_db/` debe mostrar archivos de la base de datos ChromaDB.

---

### Fase 2 — Agente ReAct con Herramientas y Memoria (~30 min)

**Objetivo:** Implementar el agente LangChain con patrón ReAct, herramientas personalizadas que usan el pipeline RAG, memoria de conversación `ConversationBufferWindowMemory` y manejo de errores con `tenacity`.

#### Paso 2.1 — Definir las herramientas del agente

**Instrucciones:**

1. Crea el módulo de herramientas:

```python
# agent/tools.py
import logging
import sys
from pathlib import Path
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
from langchain_core.tools import tool
from openai import APIConnectionError, RateLimitError

sys.path.insert(0, str(Path(__file__).parent.parent))
logger = logging.getLogger(__name__)

@tool
def rag_search(query: str) -> str:
    """
    Busca información en la base de conocimiento técnica del sistema.
    Usa esta herramienta cuando el usuario pregunte sobre RAG, agentes,
    métricas de evaluación, embeddings o conceptos técnicos de IA Generativa.
    Retorna los fragmentos de documentación más relevantes.
    """
    try:
        from rag.rag_pipeline import retrieve_with_metadata_filter
        docs = retrieve_with_metadata_filter(query)
        if not docs:
            return "No se encontró información relevante en la base de conocimiento."
        
        results = []
        for i, doc in enumerate(docs, 1):
            source = doc.metadata.get("filename", "desconocido")
            results.append(f"[Fuente {i}: {source}]\n{doc.page_content}")
        
        return "\n\n---\n\n".join(results)
    except Exception as e:
        logger.error(f"Error en rag_search: {e}")
        return f"Error al buscar en la base de conocimiento: {str(e)}"

@tool
def calculator(expression: str) -> str:
    """
    Evalúa expresiones matemáticas simples.
    Usa esta herramienta para cálculos numéricos como sumas, porcentajes,
    multiplicaciones o divisiones. Ejemplo: '(0.6 * 0.85) + (0.4 * 0.72)'
    """
    try:
        # Validación de seguridad: solo permitir caracteres matemáticos
        allowed = set("0123456789+-*/()., ")
        if not all(c in allowed for c in expression):
            return "Error: La expresión contiene caracteres no permitidos."
        result = eval(expression, {"__builtins__": {}})  # noqa: S307
        return f"Resultado: {result}"
    except ZeroDivisionError:
        return "Error: División por cero."
    except Exception as e:
        return f"Error al evaluar la expresión: {str(e)}"

@tool
def document_summarizer(topic: str) -> str:
    """
    Genera un resumen conciso de todos los documentos disponibles sobre un tema.
    Usa esta herramienta cuando el usuario pida un resumen general o una visión
    general de un concepto amplio como 'RAG', 'agentes' o 'evaluación'.
    """
    try:
        from rag.rag_pipeline import retrieve_with_metadata_filter
        docs = retrieve_with_metadata_filter(topic)
        if not docs:
            return f"No hay documentos sobre '{topic}' en la base de conocimiento."
        
        combined_text = " ".join([d.page_content for d in docs])
        # Retorna los primeros 800 caracteres como resumen aproximado
        summary = combined_text[:800] + "..." if len(combined_text) > 800 else combined_text
        return f"Resumen sobre '{topic}':\n{summary}"
    except Exception as e:
        logger.error(f"Error en document_summarizer: {e}")
        return f"Error al generar resumen: {str(e)}"

# Lista de todas las herramientas disponibles para el agente
AGENT_TOOLS = [rag_search, calculator, document_summarizer]
```

#### Paso 2.2 — Implementar el agente ReAct con memoria

**Instrucciones:**

1. Crea el módulo del agente:

```python
# agent/agent.py
import os
import logging
from tenacity import retry, stop_after_attempt, wait_exponential
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_react_agent
from langchain.memory import ConversationBufferWindowMemory
from langchain_core.prompts import PromptTemplate
from dotenv import load_dotenv

load_dotenv()
logger = logging.getLogger(__name__)

# Prompt ReAct con few-shot examples en español
REACT_PROMPT_TEMPLATE = """Eres un asistente técnico especializado en IA Generativa.
Tienes acceso a las siguientes herramientas:

{tools}

Usa el siguiente formato ESTRICTAMENTE:

Pregunta: la pregunta de entrada que debes responder
Pensamiento: siempre debes pensar qué hacer
Acción: la acción a tomar, debe ser una de [{tool_names}]
Entrada de Acción: la entrada para la acción
Observación: el resultado de la acción
... (este ciclo Pensamiento/Acción/Entrada/Observación puede repetirse N veces)
Pensamiento: Ahora sé la respuesta final
Respuesta Final: la respuesta final a la pregunta original

Ejemplos:
Pregunta: ¿Qué es la recuperación híbrida en RAG?
Pensamiento: Debo buscar información sobre recuperación híbrida en la base de conocimiento.
Acción: rag_search
Entrada de Acción: recuperación híbrida RAG BM25 vectorial
Observación: [resultado de la búsqueda]
Pensamiento: Tengo información suficiente para responder.
Respuesta Final: La recuperación híbrida combina...

Historial de conversación:
{chat_history}

Pregunta: {input}
Pensamiento: {agent_scratchpad}"""

def build_agent() -> AgentExecutor:
    """
    Construye y retorna un AgentExecutor ReAct con:
    - GPT-4o-mini como LLM base
    - Herramientas: rag_search, calculator, document_summarizer
    - Memoria: ConversationBufferWindowMemory (k=5)
    - Manejo de errores: handle_parsing_errors=True
    """
    from agent.tools import AGENT_TOOLS

    llm = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0.1,
        openai_api_key=os.getenv("OPENAI_API_KEY"),
        max_retries=3,
    )

    prompt = PromptTemplate.from_template(REACT_PROMPT_TEMPLATE)

    memory = ConversationBufferWindowMemory(
        k=5,
        memory_key="chat_history",
        return_messages=False
    )

    agent = create_react_agent(
        llm=llm,
        tools=AGENT_TOOLS,
        prompt=prompt
    )

    executor = AgentExecutor(
        agent=agent,
        tools=AGENT_TOOLS,
        memory=memory,
        verbose=True,
        handle_parsing_errors=True,
        max_iterations=6,
        return_intermediate_steps=True,
    )
    logger.info("AgentExecutor ReAct construido correctamente.")
    return executor

# Instancia global del agente
_agent_instance: AgentExecutor | None = None

def get_agent() -> AgentExecutor:
    global _agent_instance
    if _agent_instance is None:
        _agent_instance = build_agent()
    return _agent_instance

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10),
    reraise=True
)
async def run_agent(query: str, session_id: str | None = None) -> dict:
    """
    Ejecuta el agente con la query del usuario.
    Retorna respuesta final, fuentes citadas y pasos intermedios.
    """
    agent = get_agent()
    try:
        result = agent.invoke({"input": query})
        answer = result.get("output", "No se pudo generar una respuesta.")
        
        # Extraer fuentes de los pasos intermedios
        sources = []
        for step in result.get("intermediate_steps", []):
            action, observation = step
            if action.tool == "rag_search" and "[Fuente" in str(observation):
                import re
                found = re.findall(r'\[Fuente \d+: (.+?)\]', str(observation))
                sources.extend(found)
        
        sources = list(set(sources))  # Deduplicar
        logger.info(f"Agente respondió. Fuentes: {sources}")
        return {"answer": answer, "sources": sources}
    
    except Exception as e:
        logger.error(f"Error en run_agent: {e}")
        return {
            "answer": f"Lo siento, ocurrió un error al procesar tu consulta: {str(e)}",
            "sources": []
        }
```

**Verificación:**

```bash
# Prueba rápida del agente desde la línea de comandos
python -c "
import asyncio, sys
sys.path.insert(0, '.')
from dotenv import load_dotenv
load_dotenv()
from agent.agent import run_agent
result = asyncio.run(run_agent('¿Qué es la recuperación híbrida en RAG?'))
print('Respuesta:', result['answer'][:200])
print('Fuentes:', result['sources'])
"
```

**Salida esperada:** El agente debe invocar `rag_search`, recuperar documentos y generar una respuesta coherente sobre recuperación híbrida.

---

### Fase 3 — API REST con FastAPI, Autenticación y Rate Limiting (~25 min)

**Objetivo:** Exponer el agente y el pipeline RAG a través de una API REST con FastAPI, incluyendo autenticación Bearer token, rate limiting con `slowapi`, y los endpoints requeridos.

#### Paso 3.1 — Implementar los schemas Pydantic

**Instrucciones:**

1. Crea los modelos de entrada/salida:

```python
# api/schemas.py
from pydantic import BaseModel, Field
from typing import Optional

class ChatRequest(BaseModel):
    user_id: str = Field(..., min_length=1, max_length=64, description="ID único del usuario")
    query: str = Field(..., min_length=3, max_length=2000, description="Pregunta del usuario")
    session_id: Optional[str] = Field(None, description="ID de sesión para memoria")
    category_filter: Optional[str] = Field(None, description="Filtro por categoría de documentos")

class ChatResponse(BaseModel):
    answer: str
    sources: list[str]
    session_id: Optional[str]

class IngestRequest(BaseModel):
    source_dir: str = Field(default="./knowledge_base", description="Directorio con documentos")

class IngestResponse(BaseModel):
    status: str
    chunks_indexed: int
    message: str

class HealthResponse(BaseModel):
    status: str
    retriever_ready: bool
    agent_ready: bool

class MetricsResponse(BaseModel):
    faithfulness: Optional[float]
    answer_relevancy: Optional[float]
    context_precision: Optional[float]
    context_recall: Optional[float]
    evaluation_timestamp: Optional[str]
    total_questions_evaluated: int
```

#### Paso 3.2 — Implementar la aplicación FastAPI

**Instrucciones:**

1. Crea el archivo principal de la API:

```python
# api/main.py
import os
import logging
from datetime import datetime
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from fastapi.middleware.cors import CORSMiddleware
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from starlette.requests import Request
from dotenv import load_dotenv

load_dotenv()
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Rate limiter
limiter = Limiter(key_func=get_remote_address)

app = FastAPI(
    title="Sistema GenAI Capstone",
    description="API REST para asistente inteligente con RAG avanzado y agente funcional.",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["GET", "POST"],
    allow_headers=["*"],
)

# Autenticación Bearer token
security = HTTPBearer()
EXPECTED_TOKEN = os.getenv("API_BEARER_TOKEN", "token-de-desarrollo")

def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    if credentials.credentials != EXPECTED_TOKEN:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token de autenticación inválido."
        )
    return credentials.credentials

# Almacén en memoria para métricas del último benchmark
_last_metrics: dict = {}

# ── Importar schemas ──────────────────────────────────────────────────────────
import sys
from pathlib import Path
sys.path.insert(0, str(Path(__file__).parent.parent))

from api.schemas import (
    ChatRequest, ChatResponse,
    IngestRequest, IngestResponse,
    HealthResponse, MetricsResponse
)

# ── Endpoints ─────────────────────────────────────────────────────────────────

@app.get("/health", response_model=HealthResponse, tags=["Sistema"])
def health_check():
    """Verifica el estado de los componentes del sistema."""
    from rag.rag_pipeline import _retriever_instance
    from agent.agent import _agent_instance
    return HealthResponse(
        status="ok",
        retriever_ready=_retriever_instance is not None,
        agent_ready=_agent_instance is not None
    )

@app.post("/ingest", response_model=IngestResponse, tags=["Ingesta"],
          dependencies=[Depends(verify_token)])
def ingest_documents(request: IngestRequest):
    """Procesa e indexa documentos en ChromaDB. Requiere autenticación."""
    try:
        from rag.chunker import load_markdown_documents, chunk_documents
        from rag.rag_pipeline import build_retriever
        
        docs = load_markdown_documents(request.source_dir)
        if not docs:
            raise HTTPException(status_code=404, detail="No se encontraron documentos.")
        
        chunks = chunk_documents(docs)
        build_retriever(chunks)
        
        return IngestResponse(
            status="success",
            chunks_indexed=len(chunks),
            message=f"Se indexaron {len(chunks)} chunks de {len(docs)} documentos."
        )
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error en ingesta: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/chat", response_model=ChatResponse, tags=["Chat"],
          dependencies=[Depends(verify_token)])
@limiter.limit("20/minute")
async def chat(request: Request, body: ChatRequest):
    """
    Endpoint principal: procesa la consulta del usuario con el agente ReAct.
    Rate limit: 20 peticiones por minuto por IP.
    """
    import asyncio
    from agent.agent import run_agent
    
    logger.info(f"Chat request - user_id={body.user_id}, query={body.query[:50]}...")
    
    result = await run_agent(body.query, body.session_id)
    
    return ChatResponse(
        answer=result["answer"],
        sources=result["sources"],
        session_id=body.session_id
    )

@app.get("/metrics", response_model=MetricsResponse, tags=["Evaluación"])
def get_metrics():
    """Retorna las métricas del último benchmark de evaluación ejecutado."""
    if not _last_metrics:
        return MetricsResponse(
            faithfulness=None,
            answer_relevancy=None,
            context_precision=None,
            context_recall=None,
            evaluation_timestamp=None,
            total_questions_evaluated=0
        )
    return MetricsResponse(**_last_metrics)

def update_metrics(metrics_data: dict):
    """Actualiza las métricas globales tras ejecutar un benchmark."""
    global _last_metrics
    _last_metrics = {**metrics_data, "evaluation_timestamp": datetime.now().isoformat()}
```

2. Inicia la API y verifica que responde:

```bash
# Terminal 1: iniciar el servidor
uvicorn api.main:app --reload --port 8000 --log-level info

# Terminal 2: probar health check
curl http://localhost:8000/health
```

**Salida esperada:**
```json
{"status":"ok","retriever_ready":false,"agent_ready":false}
```

3. Ejecuta la ingesta vía API y prueba el chat:

```bash
# Ingestar documentos (reemplaza TOKEN con el valor en tu .env)
curl -X POST http://localhost:8000/ingest \
  -H "Authorization: Bearer mi-token-secreto-cambiar-en-produccion" \
  -H "Content-Type: application/json" \
  -d '{"source_dir": "./knowledge_base"}'

# Probar el endpoint /chat
curl -X POST http://localhost:8000/chat \
  -H "Authorization: Bearer mi-token-secreto-cambiar-en-produccion" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "u001", "query": "¿Qué es el reranking y por qué mejora la calidad del RAG?"}'
```

**Salida esperada del chat:**
```json
{
  "answer": "El reranking es una técnica que aplica un modelo cross-encoder...",
  "sources": ["rag_concepts.md"],
  "session_id": null
}
```

**Verificación:** Abre `http://localhost:8000/docs` en el navegador. Deben aparecer todos los endpoints documentados con Swagger UI.

---

### Fase 4 — Evaluación con RAGAS e Integración LangSmith (~30 min)

**Objetivo:** Construir un dataset de evaluación con 20 preguntas y ground truth, ejecutar las métricas RAGAS completas, e integrar los resultados como experimento en LangSmith.

#### Paso 4.1 — Crear el dataset de evaluación

**Instrucciones:**

1. Crea el dataset de benchmark:

```python
# evaluation/benchmark_dataset.py
"""
Dataset de evaluación: 20 preguntas con ground truth sobre la base de conocimiento.
"""

EVALUATION_DATASET = [
    {
        "question": "¿Qué es RAG y cuáles son sus tres etapas principales?",
        "ground_truth": "RAG (Retrieval-Augmented Generation) combina recuperación de información con generación de texto. Sus tres etapas son: indexación, recuperación y generación."
    },
    {
        "question": "¿Qué porcentaje de peso se asigna a la búsqueda semántica en la recuperación híbrida?",
        "ground_truth": "En la recuperación híbrida, la búsqueda semántica densa recibe un peso del 60% y BM25 el 40%."
    },
    {
        "question": "¿Cuál es el tamaño de chunk recomendado para respuestas precisas?",
        "ground_truth": "Para respuestas precisas se recomienda un tamaño de 512 tokens. Para síntesis de temas amplios, 1024 tokens."
    },
    {
        "question": "¿Cuánto mejora el reranking la precisión y cuál es su overhead de latencia?",
        "ground_truth": "El reranking mejora la precisión en un 15-25% con un overhead de latencia de 100-300ms."
    },
    {
        "question": "¿Qué significa el patrón ReAct en agentes de IA?",
        "ground_truth": "ReAct significa Reasoning + Acting. El agente intercala pasos de razonamiento y acción, generando un pensamiento, decidiendo una acción, observando el resultado y repitiendo hasta tener información suficiente."
    },
    {
        "question": "¿Cuántos turnos de conversación recuerda ConversationBufferWindowMemory con k=5?",
        "ground_truth": "Con k=5, ConversationBufferWindowMemory mantiene los últimos 5 intercambios de conversación en memoria."
    },
    {
        "question": "¿Qué librería se usa para retry logic con backoff exponencial en agentes?",
        "ground_truth": "La librería tenacity permite configurar @retry con stop_after_attempt y wait_exponential para implementar retry logic con backoff exponencial."
    },
    {
        "question": "¿Qué cuatro métricas principales evalúa el framework RAGAS?",
        "ground_truth": "RAGAS evalúa: faithfulness, answer_relevancy, context_recall y context_precision."
    },
    {
        "question": "¿Qué mide la métrica faithfulness en RAGAS?",
        "ground_truth": "Faithfulness mide si la respuesta generada está respaldada por el contexto recuperado. Una puntuación de 1.0 indica que cada afirmación puede verificarse en los documentos recuperados."
    },
    {
        "question": "¿Qué valor de faithfulness indica alucinaciones significativas?",
        "ground_truth": "Valores de faithfulness menores a 0.7 indican alucinaciones significativas."
    },
    {
        "question": "¿Cómo calcula RAGAS la métrica answer_relevancy?",
        "ground_truth": "Answer_relevancy genera preguntas hipotéticas desde la respuesta y calcula la similitud con la pregunta original. Valores mayores a 0.8 indican respuestas bien enfocadas."
    },
    {
        "question": "¿Qué diferencia hay entre context_precision y context_recall?",
        "ground_truth": "Context_precision mide si los documentos recuperados son relevantes (sin ruido). Context_recall mide si se recuperaron todos los documentos necesarios para responder."
    },
    {
        "question": "¿Qué tipo de búsqueda captura BM25 en la recuperación híbrida?",
        "ground_truth": "BM25 es un retriever disperso que captura coincidencias exactas de términos (búsqueda léxica), complementando la búsqueda semántica vectorial."
    },
    {
        "question": "¿Qué es un Architecture Decision Record (ADR)?",
        "ground_truth": "Un ADR es un documento breve que justifica cada decisión técnica relevante del proyecto, explicando el contexto, la decisión tomada y sus consecuencias positivas y negativas."
    },
    {
        "question": "¿Qué separadores usa RecursiveCharacterTextSplitter por defecto?",
        "ground_truth": "RecursiveCharacterTextSplitter divide por párrafos (doble salto de línea), luego oraciones (salto de línea), luego palabras (punto y espacio), luego espacios y finalmente caracteres individuales."
    },
    {
        "question": "¿Qué ventaja tiene Chroma sobre Pinecone en desarrollo?",
        "ground_truth": "Chroma permite despliegue local sin dependencias externas, sin costos de API y es fácil de resetear en tests, aunque requiere migración a Pinecone en producción para escalabilidad horizontal."
    },
    {
        "question": "¿Qué es MMR (Maximum Marginal Relevance) en la búsqueda vectorial?",
        "ground_truth": "MMR es un tipo de búsqueda que balancea relevancia y diversidad en los resultados recuperados, evitando retornar documentos redundantes entre sí."
    },
    {
        "question": "¿Cuántas iteraciones máximas debe tener un agente ReAct para evitar bucles infinitos?",
        "ground_truth": "Se recomienda configurar max_iterations=6 en el AgentExecutor para evitar bucles infinitos manteniendo capacidad de razonamiento multi-paso."
    },
    {
        "question": "¿Qué casos de uso priorizan context_recall sobre context_precision?",
        "ground_truth": "Los sistemas de soporte priorizan context_recall (recuperar todos los documentos necesarios), mientras que los sistemas legales priorizan context_precision (sin documentos irrelevantes)."
    },
    {
        "question": "¿Qué hace el parámetro handle_parsing_errors=True en AgentExecutor?",
        "ground_truth": "handle_parsing_errors=True permite al agente continuar ejecutándose cuando el LLM genera una salida que no sigue el formato esperado, en lugar de lanzar una excepción."
    }
]
```

#### Paso 4.2 — Implementar el evaluador RAGAS con LangSmith

**Instrucciones:**

1. Crea el módulo de evaluación:

```python
# evaluation/evaluator.py
import os
import logging
from datetime import datetime
from dotenv import load_dotenv

load_dotenv()
logger = logging.getLogger(__name__)

def run_ragas_evaluation(
    questions: list[str],
    answers: list[str],
    contexts: list[list[str]],
    ground_truths: list[str]
) -> dict:
    """
    Ejecuta evaluación RAGAS sobre un conjunto de preguntas/respuestas.
    Retorna un diccionario con las cuatro métricas principales.
    """
    try:
        from ragas import evaluate
        from ragas.metrics import (
            faithfulness,
            answer_relevancy,
            context_precision,
            context_recall,
        )
        from datasets import Dataset

        # Construir dataset en formato RAGAS
        data = {
            "question": questions,
            "answer": answers,
            "contexts": contexts,
            "ground_truth": ground_truths,
        }
        dataset = Dataset.from_dict(data)

        result = evaluate(
            dataset=dataset,
            metrics=[faithfulness, answer_relevancy, context_precision, context_recall],
        )

        metrics = {
            "faithfulness": round(float(result["faithfulness"]), 4),
            "answer_relevancy": round(float(result["answer_relevancy"]), 4),
            "context_precision": round(float(result["context_precision"]), 4),
            "context_recall": round(float(result["context_recall"]), 4),
        }
        logger.info(f"Métricas RAGAS: {metrics}")
        return metrics

    except ImportError as e:
        logger.error(f"RAGAS no disponible: {e}")
        return {"error": "RAGAS no instalado correctamente."}
    except Exception as e:
        logger.error(f"Error en evaluación RAGAS: {e}")
        return {"error": str(e)}

def log_experiment_to_langsmith(metrics: dict, experiment_name: str = "benchmark-capstone"):
    """
    Registra los resultados del benchmark como experimento en LangSmith.
    """
    try:
        from langsmith import Client
        client = Client()
        
        # Crear dataset en LangSmith si no existe
        dataset_name = "sistema-genai-capstone-eval"
        
        # Registrar el experimento como run de evaluación
        run = client.create_run(
            name=experiment_name,
            run_type="chain",
            inputs={"benchmark": "20 preguntas con ground truth"},
            outputs=metrics,
            project_name=os.getenv("LANGCHAIN_PROJECT", "sistema-genai-capstone"),
        )
        logger.info(f"Experimento registrado en LangSmith: {experiment_name}")
        return str(run.id) if run else None
    except Exception as e:
        logger.warning(f"No se pudo registrar en LangSmith: {e}")
        return None

def generate_evaluation_report(metrics: dict, output_path: str = "docs/evaluation_report.md"):
    """
    Genera un reporte de evaluación en formato Markdown.
    """
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    
    report = f"""# Reporte de Evaluación — Sistema GenAI Capstone

**Fecha:** {timestamp}
**Dataset:** 20 preguntas con ground truth
**Framework:** RAGAS 0.1.x

## Resultados de Métricas

| Métrica | Puntuación | Interpretación |
|---|---|---|
| **Faithfulness** | {metrics.get('faithfulness', 'N/A')} | {'✅ Excelente' if metrics.get('faithfulness', 0) >= 0.8 else '⚠️ Mejorable'} |
| **Answer Relevancy** | {metrics.get('answer_relevancy', 'N/A')} | {'✅ Excelente' if metrics.get('answer_relevancy', 0) >= 0.8 else '⚠️ Mejorable'} |
| **Context Precision** | {metrics.get('context_precision', 'N/A')} | {'✅ Excelente' if metrics.get('context_precision', 0) >= 0.7 else '⚠️ Mejorable'} |
| **Context Recall** | {metrics.get('context_recall', 'N/A')} | {'✅ Excelente' if metrics.get('context_recall', 0) >= 0.7 else '⚠️ Mejorable'} |

## Análisis

### Faithfulness
Mide si las respuestas están respaldadas por el contexto recuperado.
Valor objetivo: ≥ 0.80

### Answer Relevancy
Mide qué tan relevantes son las respuestas para las preguntas formuladas.
Valor objetivo: ≥ 0.80

### Context Precision
Mide la calidad de los documentos recuperados (sin ruido).
Valor objetivo: ≥ 0.70

### Context Recall
Mide si se recuperaron todos los documentos necesarios.
Valor objetivo: ≥ 0.70

## Recomendaciones

{generate_recommendations(metrics)}

---
*Reporte generado automáticamente por evaluation/evaluator.py*
"""
    
    import os
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    with open(output_path, "w", encoding="utf-8") as f:
        f.write(report)
    
    logger.info(f"Reporte guardado en: {output_path}")
    return output_path

def generate_recommendations(metrics: dict) -> str:
    recs = []
    if metrics.get("faithfulness", 1.0) < 0.8:
        recs.append("- **Faithfulness bajo:** Revisar el prompt del agente para instruirle a citar solo información del contexto recuperado.")
    if metrics.get("answer_relevancy", 1.0) < 0.8:
        recs.append("- **Answer Relevancy bajo:** Mejorar el prompt de generación para mantener el foco en la pregunta original.")
    if metrics.get("context_precision", 1.0) < 0.7:
        recs.append("- **Context Precision bajo:** Reducir el valor de k en el retriever o ajustar los pesos del EnsembleRetriever.")
    if metrics.get("context_recall", 1.0) < 0.7:
        recs.append("- **Context Recall bajo:** Aumentar k en el retriever o revisar la estrategia de chunking.")
    if not recs:
        recs.append("- ✅ Todas las métricas están dentro de los rangos objetivo. Sistema listo para producción.")
    return "\n".join(recs)
```

2. Crea el script de benchmark completo:

```python
# evaluation/run_benchmark.py
"""
Script de benchmark: ejecuta evaluación RAGAS sobre las 20 preguntas del dataset.
Uso: python evaluation/run_benchmark.py
"""
import asyncio
import logging
import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).parent.parent))

from dotenv import load_dotenv
load_dotenv()

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

async def run_benchmark():
    from evaluation.benchmark_dataset import EVALUATION_DATASET
    from evaluation.evaluator import (
        run_ragas_evaluation,
        log_experiment_to_langsmith,
        generate_evaluation_report
    )
    from agent.agent import run_agent
    from rag.rag_pipeline import retrieve_with_metadata_filter

    logger.info(f"=== Iniciando benchmark con {len(EVALUATION_DATASET)} preguntas ===")

    questions, answers, contexts, ground_truths = [], [], [], []

    # Para cada pregunta: ejecutar agente + recuperar contextos
    for i, item in enumerate(EVALUATION_DATASET, 1):
        logger.info(f"Procesando pregunta {i}/{len(EVALUATION_DATASET)}: {item['question'][:60]}...")
        
        # Obtener respuesta del agente
        result = await run_agent(item["question"])
        answers.append(result["answer"])
        
        # Obtener contextos recuperados para RAGAS
        docs = retrieve_with_metadata_filter(item["question"])
        context_texts = [d.page_content for d in docs]
        contexts.append(context_texts if context_texts else ["No context retrieved"])
        
        questions.append(item["question"])
        ground_truths.append(item["ground_truth"])

    logger.info("Ejecutando métricas RAGAS...")
    metrics = run_ragas_evaluation(questions, answers, contexts, ground_truths)
    
    logger.info(f"\n{'='*50}")
    logger.info("RESULTADOS DEL BENCHMARK:")
    for k, v in metrics.items():
        logger.info(f"  {k}: {v}")
    logger.info(f"{'='*50}\n")

    # Registrar en LangSmith
    run_id = log_experiment_to_langsmith(metrics)
    if run_id:
        logger.info(f"Experimento registrado en LangSmith (run_id: {run_id})")

    # Generar reporte
    report_path = generate_evaluation_report(metrics)
    logger.info(f"Reporte generado: {report_path}")

    # Actualizar métricas en la API
    try:
        from api.main import update_metrics
        update_metrics({
            **metrics,
            "total_questions_evaluated": len(EVALUATION_DATASET)
        })
    except Exception:
        pass

    return metrics

if __name__ == "__main__":
    asyncio.run(run_benchmark())
```

3. Ejecuta el benchmark (asegúrate de que la ingesta ya se realizó):

```bash
python evaluation/run_benchmark.py
```

**Salida esperada:**
```
=== Iniciando benchmark con 20 preguntas ===
Procesando pregunta 1/20: ¿Qué es RAG y cuáles son sus tres etapas...
...
RESULTADOS DEL BENCHMARK:
  faithfulness: 0.XXXX
  answer_relevancy: 0.XXXX
  context_precision: 0.XXXX
  context_recall: 0.XXXX
Reporte generado: docs/evaluation_report.md
```

**Verificación:** `cat docs/evaluation_report.md` debe mostrar el reporte con tabla de métricas completa.

---

### Fase 5 — Documentación Técnica Profesional (~25 min)

**Objetivo:** Producir los artefactos de documentación requeridos: diagrama de arquitectura Mermaid, ADR-001, README técnico completo y exportación de la especificación OpenAPI.

#### Paso 5.1 — Diagrama de arquitectura con Mermaid

**Instrucciones:**

1. Crea el archivo de arquitectura:

```bash
cat > docs/architecture.md << 'EOF'
# Arquitectura del Sistema GenAI Capstone

## Diagrama de Flujo Principal

```mermaid
flowchart TD
    Cliente([Cliente HTTP]) -->|POST /chat| API[FastAPI\napi/main.py]
    API -->|Verifica Bearer Token| Auth{Auth OK?}
    Auth -->|No| Error401[401 Unauthorized]
    Auth -->|Sí| Agent[AgentExecutor ReAct\nagent/agent.py]
    
    Agent -->|Decide herramienta| Tools{¿Qué herramienta?}
    Tools -->|rag_search| RAG[Pipeline RAG\nrag/rag_pipeline.py]
    Tools -->|calculator| Calc[Calculadora\ntools.py]
    Tools -->|document_summarizer| Sum[Summarizer\ntools.py]
    
    RAG -->|Query| Hybrid{Retriever Híbrido}
    Hybrid -->|60% Dense| Chroma[(ChromaDB\ntext-embedding-3-small)]
    Hybrid -->|40% Sparse| BM25[BM25Retriever]
    Chroma --> Merge[EnsembleRetriever\nTop-5 docs]
    BM25 --> Merge
    
    Merge -->|Contexto enriquecido| LLM[GPT-4o-mini\nGeneración]
    Calc --> LLM
    Sum --> LLM
    
    LLM -->|Respuesta final| Eval[Evaluador RAGAS\nevaluation/evaluator.py]
    Eval -->|Métricas| LangSmith[(LangSmith\nExperimentos)]
    Eval -->|Respuesta + fuentes| API
    API -->|JSON Response| Cliente
```

## Diagrama de Secuencia — Flujo de Chat

```mermaid
sequenceDiagram
    participant U as Usuario
    participant A as FastAPI
    participant Ag as AgentExecutor
    participant R as RAG Pipeline
    participant C as ChromaDB
    participant L as GPT-4o-mini
    participant E as RAGAS Evaluator

    U->>A: POST /chat {query, user_id}
    A->>A: Verificar Bearer Token
    A->>Ag: run_agent(query)
    Ag->>Ag: ReAct: Pensamiento inicial
    Ag->>R: rag_search(query)
    R->>C: Búsqueda vectorial MMR (k=5)
    R->>R: BM25 búsqueda léxica (k=5)
    R->>R: EnsembleRetriever merge (60/40)
    R-->>Ag: Top-5 documentos con metadatos
    Ag->>L: Prompt con contexto + historial
    L-->>Ag: Respuesta generada
    Ag->>Ag: Extraer fuentes citadas
    Ag-->>A: {answer, sources}
    A->>E: evaluate_response (async)
    E-->>A: {faithfulness, relevancy...}
    A-->>U: {answer, sources, session_id}
```

## Dependencias entre Módulos

```mermaid
graph LR
    API[api/] --> Agent[agent/]
    API --> Eval[evaluation/]
    Agent --> RAG[rag/]
    Agent --> Tools[agent/tools.py]
    Tools --> RAG
    RAG --> Chroma[(ChromaDB)]
    Eval --> LangSmith[(LangSmith)]
```
EOF
```

#### Paso 5.2 — Architecture Decision Record ADR-001

**Instrucciones:**

1. Crea el ADR principal:

```bash
cat > docs/adr/ADR-001-vector-store.md << 'EOF'
# ADR-001: Selección de ChromaDB como Base Vectorial

**Estado:** Aceptado  
**Fecha:** 2024-07-01  
**Autores:** Equipo de desarrollo

## Contexto

El sistema requiere una base vectorial para almacenar y recuperar embeddings del
pipeline RAG. Las opciones evaluadas fueron: ChromaDB, FAISS y Pinecone.

## Criterios de Evaluación

| Criterio | Chroma | FAISS | Pinecone |
|---|---|---|---|
| Despliegue local | ✅ Nativo | ✅ Nativo | ❌ Solo cloud |
| Metadatos y filtrado | ✅ Completo | ⚠️ Limitado | ✅ Completo |
| Persistencia en disco | ✅ Automática | ⚠️ Manual | ✅ Gestionada |
| Costo | Gratis | Gratis | $70/mes (prod) |
| Escalabilidad horizontal | ❌ Single-node | ❌ Single-node | ✅ Serverless |

## Decisión

Se selecciona **ChromaDB** para el entorno de desarrollo y demostración del lab
por su despliegue local sin dependencias externas, soporte nativo de metadatos
para filtrado dinámico, y persistencia automática en disco.

## Consecuencias

**Positivas:**
- Sin costos de API durante desarrollo y evaluación.
- Fácil de resetear entre ejecuciones de tests (`rm -rf chroma_db/`).
- Integración directa con LangChain sin configuración adicional.

**Negativas:**
- Migración necesaria a Pinecone o Weaviate en producción para
  escalabilidad horizontal y alta disponibilidad.
- No soporta búsqueda distribuida en múltiples nodos.

## Ruta de Migración a Producción

```python
# Cambio mínimo para migrar a Pinecone:
# Reemplazar en rag_pipeline.py:
# from langchain_community.vectorstores import Chroma
# por:
# from langchain_pinecone import PineconeVectorStore
```
EOF

cat > docs/adr/ADR-002-agent-framework.md << 'EOF'
# ADR-002: Selección de LangChain ReAct sobre LangGraph

**Estado:** Aceptado  
**Fecha:** 2024-07-01

## Contexto

El agente funcional requiere capacidad de razonamiento multi-paso con herramientas.
Las opciones fueron: LangChain AgentExecutor (ReAct), LangGraph y OpenAI Assistants API.

## Decisión

Se selecciona **LangChain AgentExecutor con patrón ReAct** por su madurez, documentación
extensa, integración nativa con herramientas personalizadas y compatibilidad directa
con ConversationBufferWindowMemory para memoria de sesión.

## Consecuencias

**Positivas:** Implementación más rápida, debugging con verbose=True, handle_parsing_errors nativo.  
**Negativas:** Menos flexible que LangGraph para flujos con múltiples agentes paralelos.
EOF
```

#### Paso 5.3 — README técnico completo

**Instrucciones:**

1. Genera el README principal:

```bash
cat > README.md << 'EOF'
# Sistema GenAI Capstone 🤖

Sistema de asistente inteligente de nivel producción que integra API REST, pipeline RAG
avanzado con recuperación híbrida, agente ReAct con memoria, y evaluación RAGAS.

## Arquitectura

Ver [docs/architecture.md](docs/architecture.md) para diagramas detallados.

**Componentes principales:**
- `api/` — FastAPI con autenticación Bearer y rate limiting
- `agent/` — AgentExecutor ReAct con herramientas personalizadas
- `rag/` — Pipeline híbrido (ChromaDB + BM25) con chunking semántico
- `evaluation/` — Benchmark RAGAS integrado con LangSmith
- `docs/adr/` — Architecture Decision Records

## Requisitos

- Python 3.11.x
- Docker Desktop 4.x
- OpenAI API key (~$2-5 USD para el lab completo)
- LangSmith account (gratuita)

## Instalación Rápida

```bash
git clone <repo-url> && cd sistema-genai-capstone
python3.11 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# Editar .env con tus credenciales
```

## Uso

### 1. Ingestar documentos
```bash
python rag/ingest.py --source ./knowledge_base
```

### 2. Iniciar la API
```bash
uvicorn api.main:app --reload --port 8000
```

### 3. Probar el chat
```bash
curl -X POST http://localhost:8000/chat \
  -H "Authorization: Bearer <tu-token>" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "u001", "query": "¿Qué es el reranking?"}'
```

### 4. Ejecutar benchmark de evaluación
```bash
python evaluation/run_benchmark.py
```

### 5. Desplegar con Docker Compose
```bash
docker compose up --build
```

## Variables de Entorno

| Variable | Descripción | Requerida |
|---|---|---|
| `OPENAI_API_KEY` | API key de OpenAI | ✅ |
| `LANGCHAIN_API_KEY` | API key de LangSmith | ✅ |
| `LANGCHAIN_PROJECT` | Nombre del proyecto en LangSmith | ✅ |
| `API_BEARER_TOKEN` | Token de autenticación de la API | ✅ |
| `CHROMA_PERSIST_DIR` | Directorio de persistencia ChromaDB | ⚠️ Opcional |

## Endpoints de la API

| Método | Ruta | Descripción | Auth |
|---|---|---|---|
| GET | `/health` | Estado del sistema | No |
| POST | `/ingest` | Indexar documentos | Bearer |
| POST | `/chat` | Chat con el agente | Bearer |
| GET | `/metrics` | Últimas métricas RAGAS | No |
| GET | `/docs` | Documentación OpenAPI | No |

## Métricas de Evaluación (Benchmark 20 preguntas)

Ver [docs/evaluation_report.md](docs/evaluation_report.md) para el reporte completo.

## Decisiones de Diseño

Ver [docs/adr/](docs/adr/) para los Architecture Decision Records.

## Licencia

MIT
EOF
```

2. Exporta la especificación OpenAPI:

```bash
# Asegúrate de que la API está corriendo, luego:
curl http://localhost:8000/openapi.json > docs/openapi.json
echo "OpenAPI spec exportada: $(wc -l < docs/openapi.json) líneas"
```

**Verificación:** `ls docs/` debe mostrar `architecture.md`, `evaluation_report.md`, `openapi.json`, `adr/ADR-001-vector-store.md`, `adr/ADR-002-agent-framework.md`.

---

### Fase 6 — Contenedorización con Docker Compose y Tests de Integración (~25 min)

**Objetivo:** Crear el `Dockerfile` y `docker-compose.yml` para desplegar el sistema completo, y ejecutar una suite de tests de integración con `pytest` y `httpx`.

#### Paso 6.1 — Dockerfile y Docker Compose

**Instrucciones:**

1. Crea el `Dockerfile`:

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Metadatos
LABEL maintainer="curso-genai"
LABEL version="1.0.0"

# Variables de entorno de seguridad
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Instalar dependencias Python
COPY requirements.txt .
RUN pip install --upgrade pip && pip install -r requirements.txt

# Copiar código fuente (excluye .env y chroma_db via .dockerignore)
COPY . .

# Crear directorio para ChromaDB
RUN mkdir -p /app/chroma_db

# Usuario no-root para seguridad
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

EXPOSE 8000

CMD ["uvicorn", "api.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "1"]
```

2. Crea el `.dockerignore`:

```bash
cat > .dockerignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
chroma_db/
.git/
tests/
*.egg-info/
.pytest_cache/
EOF
```

3. Crea el `docker-compose.yml`:

```yaml
# docker-compose.yml
version: "3.9"

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: sistema-genai-api
    ports:
      - "8000:8000"
    environment:
      # Las variables se leen desde el archivo .env del host
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - LANGCHAIN_API_KEY=${LANGCHAIN_API_KEY}
      - LANGCHAIN_TRACING_V2=${LANGCHAIN_TRACING_V2}
      - LANGCHAIN_PROJECT=${LANGCHAIN_PROJECT}
      - API_BEARER_TOKEN=${API_BEARER_TOKEN}
      - CHROMA_PERSIST_DIR=/app/chroma_db
    volumes:
      # Persistir ChromaDB entre reinicios del contenedor
      - chroma_data:/app/chroma_db
      # Montar knowledge_base para ingesta
      - ./knowledge_base:/app/knowledge_base:ro
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

volumes:
  chroma_data:
    driver: local
```

4. Construye y levanta el sistema:

```bash
docker compose up --build -d
# Esperar ~30 segundos para que el servicio esté listo
sleep 30
docker compose ps
curl http://localhost:8000/health
```

**Salida esperada:**
```
NAME                    STATUS          PORTS
sistema-genai-api       Up (healthy)    0.0.0.0:8000->8000/tcp
{"status":"ok","retriever_ready":false,"agent_ready":false}
```

#### Paso 6.2 — Suite de tests de integración

**Instrucciones:**

1. Crea los tests de integración:

```python
# tests/test_api.py
"""
Suite de tests de integración para la API REST.
Requiere que el servidor esté corriendo en localhost:8000.
"""
import os
import pytest
import httpx
from dotenv import load_dotenv

load_dotenv()

BASE_URL = "http://localhost:8000"
TOKEN = os.getenv("API_BEARER_TOKEN", "mi-token-secreto-cambiar-en-produccion")
AUTH_HEADERS = {"Authorization": f"Bearer {TOKEN}"}

@pytest.fixture(scope="session")
def client():
    return httpx.Client(base_url=BASE_URL, timeout=60.0)

class TestHealthEndpoint:
    def test_health_returns_200(self, client):
        response = client.get("/health")
        assert response.status_code == 200

    def test_health_response_schema(self, client):
        data = client.get("/health").json()
        assert "status" in data
        assert data["status"] == "ok"
        assert "retriever_ready" in data
        assert "agent_ready" in data

class TestAuthMiddleware:
    def test_chat_without_token_returns_403(self, client):
        response = client.post("/chat", json={"user_id": "u1", "query": "test"})
        assert response.status_code in [401, 403]

    def test_chat_with_invalid_token_returns_401(self, client):
        response = client.post(
            "/chat",
            headers={"Authorization": "Bearer token-invalido"},
            json={"user_id": "u1", "query": "test"}
        )
        assert response.status_code == 401

class TestIngestEndpoint:
    def test_ingest_returns_success(self, client):
        response = client.post(
            "/ingest",
            headers=AUTH_HEADERS,
            json={"source_dir": "./knowledge_base"}
        )
        assert response.status_code == 200
        data = response.json()
        assert data["status"] == "success"
        assert data["chunks_indexed"] > 0

class TestChatEndpoint:
    def test_chat_returns_answer(self, client):
        # Asegurarse de que la ingesta se realizó primero
        client.post("/ingest", headers=AUTH_HEADERS, json={"source_dir": "./knowledge_base"})
        
        response = client.post(
            "/chat",
            headers=AUTH_HEADERS,
            json={"user_id": "test-u001", "query": "¿Qué es RAG?"}
        )
        assert response.status_code == 200
        data = response.json()
        assert "answer" in data
        assert len(data["answer"]) > 10
        assert "sources" in data
        assert isinstance(data["sources"], list)

    def test_chat_query_validation(self, client):
        # Query demasiado corta (< 3 chars)
        response = client.post(
            "/chat",
            headers=AUTH_HEADERS,
            json={"user_id": "u1", "query": "ab"}
        )
        assert response.status_code == 422

class TestMetricsEndpoint:
    def test_metrics_endpoint_accessible(self, client):
        response = client.get("/metrics")
        assert response.status_code == 200

    def test_metrics
