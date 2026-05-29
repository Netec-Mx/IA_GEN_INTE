# Construir un Pipeline de Ingesta con Semantic Chunking para un Asistente Documental RAG

## 1. Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 58 minutos |
| **Complejidad** | Alta (Hard) |
| **Nivel Bloom** | Crear |
| **Módulo** | Capítulo 5 — RAG: Retrieval-Augmented Generation |
| **Costo API estimado** | ~$0.10–$0.25 USD (embeddings + GPT-4o-mini) |

---

## 2. Descripción General

En este laboratorio construirás un **pipeline completo de ingesta y consulta RAG** que procesa PDFs técnicos, aplica dos estrategias de chunking comparables y almacena los embeddings en ChromaDB con persistencia en disco. El sistema implementa Semantic Chunking con `SemanticChunker` de `langchain-experimental` como estrategia principal, comparándola contra el baseline de `RecursiveCharacterTextSplitter`. Al finalizar, dispondrás de un asistente funcional que responde preguntas sobre tu base documental y un script de evaluación comparativa que evidencia cuantitativamente las diferencias en calidad de recuperación entre ambas estrategias.

---

## 3. Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Construir un pipeline de ingesta de PDFs que aplique Semantic Chunking para producir fragmentos semánticamente coherentes usando `SemanticChunker` de `langchain-experimental`.
- [ ] Generar y almacenar embeddings con `text-embedding-3-small` en ChromaDB con persistencia en disco y metadata enriquecida (fuente, página, fecha, hash MD5).
- [ ] Implementar la capa de recuperación RAG con búsqueda por similitud coseno y MMR (Maximal Marginal Relevance) para maximizar diversidad de contexto.
- [ ] Evaluar cualitativamente la diferencia en calidad de respuestas entre Semantic Chunking y Fixed-size Chunking usando un conjunto de 10 preguntas de prueba.

---

## 4. Prerrequisitos

### Conocimiento previo
- Haber completado Labs 01-00-01 y 03-00-01 del curso.
- Comprensión de embeddings y similitud coseno (cubierto en IA GEN ESS).
- Familiaridad con LangChain 0.2.x: cadenas, loaders y retrievers.
- Conceptos de RAG cubiertos en la Lección 5.1 de este módulo.

### Acceso y credenciales
- `OPENAI_API_KEY` válida con acceso a `text-embedding-3-small` y `gpt-4o-mini`.
- Límite de gasto mensual configurado en la consola de OpenAI (≤ $5 USD recomendado para el curso completo).
- Mínimo 500 MB de espacio libre en disco para ChromaDB y PDFs de prueba.

---

## 5. Entorno del Laboratorio

### Hardware requerido

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 8 GB | 16 GB |
| CPU | 4 núcleos | 8 núcleos |
| Disco libre | 500 MB | 1 GB |
| Internet | 10 Mbps | 25 Mbps |

### Software requerido

| Componente | Versión |
|---|---|
| Python | 3.11.x |
| LangChain | 0.2.x |
| langchain-experimental | 0.0.62.x |
| langchain-community | 0.2.x |
| langchain-openai | 0.1.x |
| chromadb | 0.5.x |
| pypdf | 4.x |
| openai SDK | 1.35.x |
| python-dotenv | 1.0.x |

### Configuración del entorno

Ejecuta los siguientes comandos en tu terminal para crear el entorno aislado del laboratorio:

```bash
# 1. Crear directorio del laboratorio
mkdir lab05_rag_pipeline && cd lab05_rag_pipeline

# 2. Crear entorno virtual aislado
python3.11 -m venv .venv

# 3. Activar el entorno virtual
# En macOS/Linux:
source .venv/bin/activate
# En Windows (PowerShell):
# .venv\Scripts\Activate.ps1

# 4. Actualizar pip
pip install --upgrade pip

# 5. Instalar dependencias del laboratorio
pip install \
  "langchain==0.2.16" \
  "langchain-community==0.2.16" \
  "langchain-openai==0.1.25" \
  "langchain-experimental==0.0.62" \
  "chromadb==0.5.5" \
  "pypdf==4.3.1" \
  "openai==1.35.0" \
  "python-dotenv==1.0.1" \
  "tiktoken==0.7.0"

# 6. Verificar instalación
python -c "import langchain; import chromadb; import langchain_experimental; print('OK — todas las dependencias instaladas')"
```

### Archivo `.env` y `.gitignore`

```bash
# Crear archivo .env con tu API key
cat > .env << 'EOF'
OPENAI_API_KEY=sk-proj-TU_API_KEY_AQUI
EOF

# Crear .gitignore para proteger credenciales
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
chroma_db/
chroma_db_fixed/
pdfs/
*.pdf
EOF
```

> ⚠️ **Seguridad**: Nunca hardcodees tu API key en el código. Verifica que `.env` aparezca en `.gitignore` antes de cualquier commit.

### Estructura de directorios del proyecto

```
lab05_rag_pipeline/
├── .env
├── .gitignore
├── requirements.txt
├── pdfs/                          # PDFs técnicos de prueba
│   ├── doc1.pdf
│   ├── doc2.pdf
│   └── doc3.pdf
├── chroma_db/                     # Persistencia ChromaDB (semantic chunking)
├── chroma_db_fixed/               # Persistencia ChromaDB (fixed chunking)
├── rag_ingestion_pipeline.py      # Pipeline principal
└── evaluation_comparison.py       # Script de evaluación comparativa
```

```bash
# Crear directorios necesarios
mkdir -p pdfs chroma_db chroma_db_fixed
```

---

## 6. Instrucciones Paso a Paso

---

### Paso 1: Obtener los PDFs técnicos de prueba

**Objetivo:** Preparar la base documental con 3 PDFs técnicos reales que el pipeline procesará.

#### Instrucciones

Para este laboratorio necesitas 3 PDFs técnicos de aproximadamente 10–30 páginas cada uno. Puedes usar cualquiera de las siguientes opciones:

**Opción A — Descargar PDFs de documentación técnica pública (recomendado):**

```bash
# Descargar el artículo original de RAG (Lewis et al., 2020) — ~10 páginas
curl -L "https://arxiv.org/pdf/2005.11401" -o pdfs/rag_paper.pdf

# Descargar la guía técnica de embeddings de OpenAI (si está disponible como PDF)
# Alternativamente, usa cualquier PDF técnico de tu elección
# Ejemplo: documentación de FastAPI, artículos de arXiv, manuales técnicos

# Verificar que los PDFs se descargaron correctamente
ls -lh pdfs/
```

**Opción B — Usar PDFs propios del curso:**
Si el instructor proporcionó PDFs específicos para el laboratorio, cópialos al directorio `pdfs/`:

```bash
cp /ruta/a/tus/pdfs/*.pdf pdfs/
```

**Opción C — Crear PDFs de prueba sintéticos (fallback):**

```python
# crear_pdfs_prueba.py — Ejecutar solo si no tienes PDFs disponibles
# Requiere: pip install reportlab
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

def crear_pdf_tecnico(nombre_archivo, titulo, contenido_paginas):
    c = canvas.Canvas(nombre_archivo, pagesize=letter)
    width, height = letter
    for i, contenido in enumerate(contenido_paginas):
        c.setFont("Helvetica-Bold", 16)
        c.drawString(72, height - 72, f"{titulo} — Página {i+1}")
        c.setFont("Helvetica", 11)
        y = height - 120
        for linea in contenido.split('\n'):
            if y < 72:
                c.showPage()
                y = height - 72
            c.drawString(72, y, linea[:90])
            y -= 16
        c.showPage()
    c.save()

# Contenido técnico de ejemplo
contenido_api = [
    "REST API Design Principles\nHTTP methods define the type of operation: GET retrieves data,\n"
    "POST creates new resources, PUT updates existing resources,\nDELETE removes resources.\n\n"
    "Status codes communicate the result: 200 OK, 201 Created,\n400 Bad Request, 401 Unauthorized,\n"
    "404 Not Found, 500 Internal Server Error.\n\nAuthentication methods include API Keys,\n"
    "OAuth 2.0, JWT tokens, and Basic Auth.\nRate limiting protects APIs from abuse.\n"
    "Pagination handles large result sets using cursor-based or offset pagination.\n"
    "Versioning strategies: URL path versioning (/v1/), header versioning,\nquery parameter versioning."
] * 5  # Repetir para simular ~5 páginas

crear_pdf_tecnico("pdfs/api_design.pdf", "API Design Guide", contenido_api)
crear_pdf_tecnico("pdfs/rag_concepts.pdf", "RAG Systems", contenido_api)
crear_pdf_tecnico("pdfs/embeddings_guide.pdf", "Embeddings Guide", contenido_api)
print("PDFs de prueba creados en pdfs/")
```

#### Resultado esperado

```
pdfs/
├── rag_paper.pdf       (o tus PDFs técnicos)
├── doc2.pdf
└── doc3.pdf
```

#### Verificación

```bash
python -c "
from pypdf import PdfReader
import os
for f in os.listdir('pdfs'):
    if f.endswith('.pdf'):
        r = PdfReader(f'pdfs/{f}')
        print(f'{f}: {len(r.pages)} páginas')
"
```

---

### Paso 2: Implementar el módulo de carga de PDFs

**Objetivo:** Crear la función `load_pdfs()` que extrae texto de los PDFs con metadata enriquecida y detección de duplicados por hash MD5.

#### Instrucciones

Crea el archivo principal del pipeline:

```bash
touch rag_ingestion_pipeline.py
```

Abre `rag_ingestion_pipeline.py` y agrega el siguiente código:

```python
# rag_ingestion_pipeline.py
"""
Pipeline completo de ingesta RAG con Semantic Chunking.
Lab 05-00-01 — Curso de IA Generativa
"""

import os
import hashlib
import logging
from datetime import datetime
from typing import List, Dict, Optional
from pathlib import Path
from dotenv import load_dotenv

# LangChain imports
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain.schema import Document
from langchain.prompts import PromptTemplate
from langchain.chains import RetrievalQA

# Configuración de logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)
logger = logging.getLogger(__name__)

# Cargar variables de entorno desde .env
load_dotenv()

# ─────────────────────────────────────────────
# CONSTANTES DE CONFIGURACIÓN
# ─────────────────────────────────────────────
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
if not OPENAI_API_KEY:
    raise EnvironmentError(
        "OPENAI_API_KEY no encontrada. Verifica tu archivo .env"
    )

EMBEDDING_MODEL = "text-embedding-3-small"
LLM_MODEL = "gpt-4o-mini"
CHROMA_DIR_SEMANTIC = "./chroma_db"
CHROMA_DIR_FIXED = "./chroma_db_fixed"
COLLECTION_SEMANTIC = "rag_semantic_chunks"
COLLECTION_FIXED = "rag_fixed_chunks"
PDF_DIRECTORY = "./pdfs"

# ─────────────────────────────────────────────
# MÓDULO 1: CARGA DE PDFs
# ─────────────────────────────────────────────

def compute_md5(text: str) -> str:
    """Calcula el hash MD5 del contenido de texto para detectar duplicados."""
    return hashlib.md5(text.encode("utf-8")).hexdigest()


def load_pdfs(directory: str) -> List[Document]:
    """
    Carga todos los PDFs de un directorio usando PyPDFLoader.
    
    Extrae metadata enriquecida por página:
    - source: nombre del archivo
    - page: número de página (base 0)
    - processing_date: timestamp ISO de procesamiento
    - content_hash: MD5 del contenido para deduplicación
    
    Args:
        directory: Ruta al directorio que contiene los PDFs.
    
    Returns:
        Lista de Document con contenido y metadata. Excluye duplicados por MD5.
    """
    pdf_dir = Path(directory)
    if not pdf_dir.exists():
        raise FileNotFoundError(f"Directorio no encontrado: {directory}")
    
    pdf_files = list(pdf_dir.glob("*.pdf"))
    if not pdf_files:
        raise ValueError(f"No se encontraron archivos PDF en: {directory}")
    
    logger.info(f"Encontrados {len(pdf_files)} archivos PDF en '{directory}'")
    
    all_documents: List[Document] = []
    seen_hashes: set = set()
    processing_date = datetime.now().isoformat()
    
    for pdf_path in sorted(pdf_files):
        logger.info(f"  Cargando: {pdf_path.name}")
        try:
            loader = PyPDFLoader(str(pdf_path))
            pages = loader.load()
            
            pages_added = 0
            for doc in pages:
                # Enriquecer metadata
                content_hash = compute_md5(doc.page_content)
                
                # Saltar páginas duplicadas
                if content_hash in seen_hashes:
                    logger.debug(
                        f"    Página duplicada omitida: {pdf_path.name} "
                        f"p.{doc.metadata.get('page', '?')}"
                    )
                    continue
                
                seen_hashes.add(content_hash)
                
                # Saltar páginas con contenido insuficiente (< 50 caracteres)
                if len(doc.page_content.strip()) < 50:
                    continue
                
                doc.metadata.update({
                    "source": pdf_path.name,
                    "page": doc.metadata.get("page", 0),
                    "processing_date": processing_date,
                    "content_hash": content_hash,
                    "file_path": str(pdf_path.resolve())
                })
                
                all_documents.append(doc)
                pages_added += 1
            
            logger.info(
                f"    ✓ {pages_added} páginas cargadas "
                f"({len(pages) - pages_added} omitidas)"
            )
        
        except Exception as e:
            logger.error(f"    ✗ Error cargando {pdf_path.name}: {e}")
            continue
    
    logger.info(
        f"Carga completada: {len(all_documents)} páginas totales "
        f"de {len(pdf_files)} archivos"
    )
    return all_documents
```

#### Resultado esperado

Al importar el módulo no deben aparecer errores. La función `load_pdfs` está lista para ser usada.

#### Verificación

```bash
python -c "
from rag_ingestion_pipeline import load_pdfs
docs = load_pdfs('./pdfs')
print(f'Documentos cargados: {len(docs)}')
print(f'Primer doc — fuente: {docs[0].metadata[\"source\"]}')
print(f'Primer doc — página: {docs[0].metadata[\"page\"]}')
print(f'Primer doc — hash: {docs[0].metadata[\"content_hash\"]}')
print(f'Primeros 200 chars: {docs[0].page_content[:200]}')
"
```

---

### Paso 3: Implementar los módulos de Chunking (Fixed y Semantic)

**Objetivo:** Crear las dos funciones de chunking comparables: `RecursiveCharacterTextSplitter` como baseline y `SemanticChunker` como estrategia principal.

#### Instrucciones

Agrega el siguiente código al final de `rag_ingestion_pipeline.py`:

```python
# ─────────────────────────────────────────────
# MÓDULO 2: CHUNKING — DOS ESTRATEGIAS
# ─────────────────────────────────────────────

def chunk_fixed_size(
    documents: List[Document],
    chunk_size: int = 500,
    chunk_overlap: int = 50
) -> List[Document]:
    """
    Estrategia BASELINE: Divide documentos en chunks de tamaño fijo.
    
    Usa RecursiveCharacterTextSplitter que intenta respetar separadores
    naturales (párrafos, oraciones) antes de cortar por longitud.
    
    Args:
        documents: Lista de documentos cargados.
        chunk_size: Número máximo de caracteres por chunk.
        chunk_overlap: Caracteres de superposición entre chunks consecutivos.
    
    Returns:
        Lista de Document con chunks de tamaño fijo y metadata preservada.
    """
    logger.info(
        f"[Fixed Chunking] Dividiendo {len(documents)} páginas "
        f"(chunk_size={chunk_size}, overlap={chunk_overlap})"
    )
    
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=chunk_overlap,
        length_function=len,
        separators=["\n\n", "\n", ". ", " ", ""]
    )
    
    chunks = splitter.split_documents(documents)
    
    # Agregar metadata de estrategia y índice de chunk
    for i, chunk in enumerate(chunks):
        chunk.metadata["chunk_strategy"] = "fixed_size"
        chunk.metadata["chunk_index"] = i
        chunk.metadata["chunk_size_chars"] = len(chunk.page_content)
    
    logger.info(
        f"[Fixed Chunking] ✓ {len(chunks)} chunks generados "
        f"(promedio: {sum(len(c.page_content) for c in chunks)//len(chunks) if chunks else 0} chars)"
    )
    return chunks


def chunk_semantic(
    documents: List[Document],
    breakpoint_threshold_type: str = "percentile",
    breakpoint_threshold_amount: float = 95.0
) -> List[Document]:
    """
    Estrategia PRINCIPAL: Divide documentos usando SemanticChunker.
    
    SemanticChunker calcula embeddings de oraciones consecutivas y detecta
    puntos de quiebre semántico donde el significado cambia significativamente.
    Produce chunks que representan unidades conceptuales coherentes.
    
    Args:
        documents: Lista de documentos cargados.
        breakpoint_threshold_type: Método para detectar quiebres semánticos.
            - "percentile": usa el percentil de las diferencias de similitud
            - "standard_deviation": usa desviaciones estándar
            - "interquartile": usa el rango intercuartílico
        breakpoint_threshold_amount: Umbral numérico para el método elegido.
            Para "percentile", valores más altos = menos chunks (más grandes).
    
    Returns:
        Lista de Document con chunks semánticamente coherentes.
    """
    logger.info(
        f"[Semantic Chunking] Dividiendo {len(documents)} páginas "
        f"(threshold_type={breakpoint_threshold_type}, "
        f"threshold={breakpoint_threshold_amount})"
    )
    logger.info(
        "[Semantic Chunking] ⚠️  Esto llamará a la API de embeddings. "
        "Puede tomar 1-3 minutos..."
    )
    
    embeddings = OpenAIEmbeddings(
        model=EMBEDDING_MODEL,
        openai_api_key=OPENAI_API_KEY
    )
    
    semantic_splitter = SemanticChunker(
        embeddings=embeddings,
        breakpoint_threshold_type=breakpoint_threshold_type,
        breakpoint_threshold_amount=breakpoint_threshold_amount
    )
    
    chunks = semantic_splitter.split_documents(documents)
    
    # Agregar metadata de estrategia y estadísticas
    for i, chunk in enumerate(chunks):
        chunk.metadata["chunk_strategy"] = "semantic"
        chunk.metadata["chunk_index"] = i
        chunk.metadata["chunk_size_chars"] = len(chunk.page_content)
    
    if chunks:
        sizes = [len(c.page_content) for c in chunks]
        logger.info(
            f"[Semantic Chunking] ✓ {len(chunks)} chunks generados\n"
            f"  Tamaño mín: {min(sizes)} chars | "
            f"Tamaño máx: {max(sizes)} chars | "
            f"Promedio: {sum(sizes)//len(sizes)} chars"
        )
    
    return chunks
```

#### Resultado esperado

Las dos funciones están disponibles. `chunk_fixed_size` es rápida (sin llamadas API). `chunk_semantic` realiza llamadas a la API de embeddings de OpenAI durante el proceso de chunking.

#### Verificación

```bash
python -c "
from rag_ingestion_pipeline import load_pdfs, chunk_fixed_size
docs = load_pdfs('./pdfs')
chunks = chunk_fixed_size(docs)
print(f'Chunks fixed: {len(chunks)}')
print(f'Ejemplo chunk: {chunks[0].page_content[:150]}')
print(f'Metadata: {chunks[0].metadata}')
"
```

---

### Paso 4: Implementar el módulo de embeddings y almacenamiento en ChromaDB

**Objetivo:** Crear la función `generate_embeddings()` que vectoriza los chunks y los persiste en ChromaDB con colecciones separadas por estrategia.

#### Instrucciones

Agrega el siguiente código al final de `rag_ingestion_pipeline.py`:

```python
# ─────────────────────────────────────────────
# MÓDULO 3: EMBEDDINGS Y ALMACENAMIENTO
# ─────────────────────────────────────────────

def generate_embeddings(
    chunks: List[Document],
    persist_directory: str,
    collection_name: str,
    force_rebuild: bool = False
) -> Chroma:
    """
    Genera embeddings para cada chunk y los almacena en ChromaDB.
    
    Usa text-embedding-3-small de OpenAI (1536 dimensiones).
    Implementa persistencia en disco para evitar reindexar en cada ejecución.
    
    Args:
        chunks: Lista de Document con chunks a indexar.
        persist_directory: Directorio donde ChromaDB persiste los datos.
        collection_name: Nombre de la colección en ChromaDB.
        force_rebuild: Si True, elimina la colección existente y reconstruye.
    
    Returns:
        Instancia de Chroma lista para búsquedas.
    """
    embeddings = OpenAIEmbeddings(
        model=EMBEDDING_MODEL,
        openai_api_key=OPENAI_API_KEY
    )
    
    persist_path = Path(persist_directory)
    
    # Verificar si ya existe una colección persistida
    if persist_path.exists() and any(persist_path.iterdir()) and not force_rebuild:
        logger.info(
            f"[ChromaDB] Cargando colección existente '{collection_name}' "
            f"desde '{persist_directory}'"
        )
        vectorstore = Chroma(
            collection_name=collection_name,
            embedding_function=embeddings,
            persist_directory=persist_directory
        )
        count = vectorstore._collection.count()
        logger.info(f"[ChromaDB] ✓ Colección cargada con {count} vectores")
        return vectorstore
    
    # Construir nueva colección
    logger.info(
        f"[ChromaDB] Creando colección '{collection_name}' "
        f"con {len(chunks)} chunks..."
    )
    logger.info(
        "[ChromaDB] ⚠️  Generando embeddings vía API. "
        "Esto puede tomar 2-5 minutos..."
    )
    
    # ChromaDB procesa en lotes automáticamente
    # Dividir en lotes de 100 para evitar timeouts en colecciones grandes
    BATCH_SIZE = 100
    
    if len(chunks) <= BATCH_SIZE:
        vectorstore = Chroma.from_documents(
            documents=chunks,
            embedding=embeddings,
            collection_name=collection_name,
            persist_directory=persist_directory
        )
    else:
        # Procesar en lotes para colecciones grandes
        vectorstore = Chroma.from_documents(
            documents=chunks[:BATCH_SIZE],
            embedding=embeddings,
            collection_name=collection_name,
            persist_directory=persist_directory
        )
        for i in range(BATCH_SIZE, len(chunks), BATCH_SIZE):
            batch = chunks[i:i + BATCH_SIZE]
            vectorstore.add_documents(batch)
            logger.info(
                f"[ChromaDB] Procesado lote {i//BATCH_SIZE + 1}/"
                f"{(len(chunks) + BATCH_SIZE - 1)//BATCH_SIZE}"
            )
    
    count = vectorstore._collection.count()
    logger.info(
        f"[ChromaDB] ✓ Colección '{collection_name}' creada "
        f"con {count} vectores en '{persist_directory}'"
    )
    
    return vectorstore
```

#### Resultado esperado

La función crea o carga una colección ChromaDB. En la primera ejecución, genera los embeddings vía API. En ejecuciones posteriores, carga desde disco sin costo adicional.

#### Verificación

```bash
python -c "
from rag_ingestion_pipeline import load_pdfs, chunk_fixed_size, generate_embeddings
docs = load_pdfs('./pdfs')
chunks = chunk_fixed_size(docs[:2])  # Solo 2 páginas para prueba rápida
vs = generate_embeddings(chunks, './chroma_db_test', 'test_collection')
print(f'Vectores almacenados: {vs._collection.count()}')
# Limpiar colección de prueba
import shutil; shutil.rmtree('./chroma_db_test', ignore_errors=True)
print('Prueba completada y limpiada')
"
```

---

### Paso 5: Implementar el módulo de recuperación con MMR

**Objetivo:** Crear la función `retrieve_context()` que realiza búsqueda semántica con MMR para obtener contexto diverso y relevante.

#### Instrucciones

Agrega el siguiente código al final de `rag_ingestion_pipeline.py`:

```python
# ─────────────────────────────────────────────
# MÓDULO 4: RECUPERACIÓN CON MMR
# ─────────────────────────────────────────────

def retrieve_context(
    query: str,
    vectorstore: Chroma,
    k: int = 5,
    use_mmr: bool = True,
    fetch_k: int = 20,
    lambda_mult: float = 0.5
) -> List[Document]:
    """
    Recupera los chunks más relevantes para una consulta usando búsqueda semántica.
    
    Implementa dos modos de búsqueda:
    - Similarity Search: devuelve los k chunks con mayor similitud coseno.
    - MMR (Maximal Marginal Relevance): balancea relevancia y diversidad,
      evitando recuperar múltiples chunks casi idénticos del mismo documento.
    
    MMR selecciona iterativamente documentos que maximizan:
        MMR = λ * sim(query, doc) - (1-λ) * max_sim(doc, selected_docs)
    
    Args:
        query: Pregunta del usuario en lenguaje natural.
        vectorstore: Instancia de ChromaDB con los documentos indexados.
        k: Número de chunks a devolver.
        use_mmr: Si True, usa MMR; si False, usa similarity search puro.
        fetch_k: Candidatos iniciales para MMR (debe ser > k).
        lambda_mult: Balance relevancia/diversidad en MMR (0=máx diversidad, 1=máx relevancia).
    
    Returns:
        Lista de k Document ordenados por relevancia/diversidad.
    """
    if use_mmr:
        retriever = vectorstore.as_retriever(
            search_type="mmr",
            search_kwargs={
                "k": k,
                "fetch_k": fetch_k,
                "lambda_mult": lambda_mult
            }
        )
        logger.debug(
            f"[Retrieval] MMR — query='{query[:50]}...', "
            f"k={k}, fetch_k={fetch_k}, λ={lambda_mult}"
        )
    else:
        retriever = vectorstore.as_retriever(
            search_type="similarity",
            search_kwargs={"k": k}
        )
        logger.debug(
            f"[Retrieval] Similarity — query='{query[:50]}...', k={k}"
        )
    
    docs = retriever.invoke(query)
    
    logger.info(
        f"[Retrieval] ✓ {len(docs)} chunks recuperados para: "
        f"'{query[:60]}...'"
    )
    
    return docs
```

#### Resultado esperado

La función devuelve una lista de `Document` con los chunks más relevantes, incluyendo su metadata original (fuente, página, hash).

---

### Paso 6: Implementar el módulo de generación RAG

**Objetivo:** Crear la función `answer_question()` que construye el prompt RAG con el contexto recuperado y genera la respuesta usando GPT-4o-mini.

#### Instrucciones

Agrega el siguiente código al final de `rag_ingestion_pipeline.py`:

```python
# ─────────────────────────────────────────────
# MÓDULO 5: GENERACIÓN RAG
# ─────────────────────────────────────────────

# Prompt template para el sistema RAG
RAG_PROMPT_TEMPLATE = """Eres un asistente técnico especializado que responde preguntas 
basándose EXCLUSIVAMENTE en el contexto documental proporcionado.

REGLAS:
1. Responde ÚNICAMENTE con información presente en el contexto.
2. Si la información no está en el contexto, di explícitamente: 
   "No encontré información sobre esto en los documentos disponibles."
3. Cita la fuente (nombre del archivo y página) cuando sea posible.
4. Sé preciso y conciso. Evita repetir información.

CONTEXTO DOCUMENTAL:
{context}

PREGUNTA: {question}

RESPUESTA:"""

RAG_PROMPT = PromptTemplate(
    template=RAG_PROMPT_TEMPLATE,
    input_variables=["context", "question"]
)


def answer_question(
    query: str,
    vectorstore: Chroma,
    k: int = 5,
    use_mmr: bool = True,
    return_sources: bool = True
) -> Dict:
    """
    Responde una pregunta usando el pipeline RAG completo.
    
    Flujo:
    1. Recupera contexto relevante de ChromaDB (con MMR opcional).
    2. Construye prompt enriquecido con el contexto.
    3. Llama a GPT-4o-mini para generar la respuesta.
    4. Devuelve respuesta + fuentes utilizadas.
    
    Args:
        query: Pregunta del usuario.
        vectorstore: Base vectorial ChromaDB con documentos indexados.
        k: Número de chunks de contexto a recuperar.
        use_mmr: Si True, usa MMR para diversidad de contexto.
        return_sources: Si True, incluye los documentos fuente en el resultado.
    
    Returns:
        Dict con 'answer' (str), 'sources' (List[Document]), 
        'chunks_used' (int), 'query' (str).
    """
    llm = ChatOpenAI(
        model=LLM_MODEL,
        temperature=0,
        openai_api_key=OPENAI_API_KEY
    )
    
    # Configurar retriever según estrategia
    if use_mmr:
        retriever = vectorstore.as_retriever(
            search_type="mmr",
            search_kwargs={"k": k, "fetch_k": k * 4, "lambda_mult": 0.5}
        )
    else:
        retriever = vectorstore.as_retriever(
            search_type="similarity",
            search_kwargs={"k": k}
        )
    
    # Crear cadena RAG con RetrievalQA
    rag_chain = RetrievalQA.from_chain_type(
        llm=llm,
        chain_type="stuff",
        retriever=retriever,
        return_source_documents=return_sources,
        chain_type_kwargs={"prompt": RAG_PROMPT}
    )
    
    logger.info(f"[RAG] Procesando consulta: '{query[:80]}'")
    
    result = rag_chain.invoke({"query": query})
    
    answer = result.get("result", "Sin respuesta")
    sources = result.get("source_documents", [])
    
    # Log de fuentes utilizadas
    if sources:
        source_info = [
            f"{doc.metadata.get('source', 'N/A')} p.{doc.metadata.get('page', '?')}"
            for doc in sources
        ]
        logger.info(f"[RAG] Fuentes: {', '.join(set(source_info))}")
    
    return {
        "query": query,
        "answer": answer,
        "sources": sources,
        "chunks_used": len(sources)
    }


# ─────────────────────────────────────────────
# FUNCIÓN PRINCIPAL DE INGESTA
# ─────────────────────────────────────────────

def run_ingestion_pipeline(
    pdf_directory: str = PDF_DIRECTORY,
    force_rebuild: bool = False
) -> Dict[str, Chroma]:
    """
    Ejecuta el pipeline completo de ingesta para ambas estrategias de chunking.
    
    Returns:
        Dict con claves 'semantic' y 'fixed' mapeadas a sus vectorstores.
    """
    logger.info("=" * 60)
    logger.info("INICIANDO PIPELINE DE INGESTA RAG")
    logger.info("=" * 60)
    
    # Paso 1: Cargar PDFs
    documents = load_pdfs(pdf_directory)
    
    # Paso 2a: Chunking Fixed-size (baseline)
    logger.info("\n--- Estrategia 1: Fixed-size Chunking ---")
    fixed_chunks = chunk_fixed_size(documents)
    
    # Paso 2b: Chunking Semántico (principal)
    logger.info("\n--- Estrategia 2: Semantic Chunking ---")
    semantic_chunks = chunk_semantic(documents)
    
    # Paso 3: Generar embeddings y almacenar en ChromaDB
    logger.info("\n--- Almacenando en ChromaDB ---")
    
    vectorstore_fixed = generate_embeddings(
        chunks=fixed_chunks,
        persist_directory=CHROMA_DIR_FIXED,
        collection_name=COLLECTION_FIXED,
        force_rebuild=force_rebuild
    )
    
    vectorstore_semantic = generate_embeddings(
        chunks=semantic_chunks,
        persist_directory=CHROMA_DIR_SEMANTIC,
        collection_name=COLLECTION_SEMANTIC,
        force_rebuild=force_rebuild
    )
    
    logger.info("\n" + "=" * 60)
    logger.info("INGESTA COMPLETADA")
    logger.info(f"  Fixed chunks:    {len(fixed_chunks)}")
    logger.info(f"  Semantic chunks: {len(semantic_chunks)}")
    logger.info("=" * 60)
    
    return {
        "semantic": vectorstore_semantic,
        "fixed": vectorstore_fixed
    }


if __name__ == "__main__":
    # Ejecutar pipeline de ingesta completo
    vectorstores = run_ingestion_pipeline()
    
    # Prueba rápida de consulta
    pregunta_prueba = "¿Qué es RAG y cómo funciona?"
    
    print("\n" + "=" * 60)
    print("PRUEBA DE CONSULTA")
    print("=" * 60)
    print(f"Pregunta: {pregunta_prueba}\n")
    
    resultado = answer_question(
        query=pregunta_prueba,
        vectorstore=vectorstores["semantic"],
        k=4
    )
    
    print(f"Respuesta (Semantic):\n{resultado['answer']}")
    print(f"\nChunks utilizados: {resultado['chunks_used']}")
```

#### Resultado esperado

```
============================================================
INICIANDO PIPELINE DE INGESTA RAG
============================================================
[INFO] Encontrados 3 archivos PDF en './pdfs'
...
[INFO] INGESTA COMPLETADA
[INFO]   Fixed chunks:    XXX
[INFO]   Semantic chunks: YYY
============================================================
```

#### Verificación

```bash
# Ejecutar el pipeline completo
python rag_ingestion_pipeline.py
```

---

### Paso 7: Implementar el script de evaluación comparativa

**Objetivo:** Crear `evaluation_comparison.py` que procesa 10 preguntas con ambas estrategias y registra los resultados para comparación manual.

#### Instrucciones

Crea el archivo de evaluación:

```python
# evaluation_comparison.py
"""
Script de evaluación comparativa: Semantic Chunking vs. Fixed-size Chunking.
Procesa 10 preguntas con ambas estrategias y genera un reporte de comparación.
Lab 05-00-01
"""

import json
import logging
from datetime import datetime
from pathlib import Path
from dotenv import load_dotenv

from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

from rag_ingestion_pipeline import (
    answer_question,
    run_ingestion_pipeline,
    CHROMA_DIR_SEMANTIC,
    CHROMA_DIR_FIXED,
    COLLECTION_SEMANTIC,
    COLLECTION_FIXED,
    EMBEDDING_MODEL,
    OPENAI_API_KEY
)

load_dotenv()
logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

# ─────────────────────────────────────────────
# PREGUNTAS DE EVALUACIÓN
# Adapta estas preguntas al contenido de TUS PDFs
# ─────────────────────────────────────────────
EVALUATION_QUESTIONS = [
    "¿Qué es RAG y cuál es su propósito principal?",
    "¿Cómo funciona la búsqueda por similitud semántica en sistemas de recuperación?",
    "¿Cuáles son las diferencias entre los métodos de autenticación en APIs REST?",
    "¿Qué ventajas tienen los embeddings densos sobre los métodos de recuperación sparse?",
    "¿Cómo se implementa la paginación en APIs REST para manejar grandes volúmenes de datos?",
    "¿Qué modelos de embeddings se recomiendan para búsqueda semántica en español?",
    "¿Cuáles son las mejores prácticas para el versionado de APIs?",
    "¿Cómo se mide la calidad de un sistema RAG en producción?",
    "¿Qué es el fine-tuning y cuándo es preferible a RAG?",
    "¿Cuáles son los principales desafíos de escalar un sistema RAG a millones de documentos?"
]

# ─────────────────────────────────────────────
# FUNCIONES DE EVALUACIÓN
# ─────────────────────────────────────────────

def load_existing_vectorstores() -> dict:
    """Carga las colecciones ChromaDB ya creadas por el pipeline de ingesta."""
    embeddings = OpenAIEmbeddings(
        model=EMBEDDING_MODEL,
        openai_api_key=OPENAI_API_KEY
    )
    
    vectorstores = {}
    
    for strategy, chroma_dir, collection in [
        ("semantic", CHROMA_DIR_SEMANTIC, COLLECTION_SEMANTIC),
        ("fixed", CHROMA_DIR_FIXED, COLLECTION_FIXED)
    ]:
        path = Path(chroma_dir)
        if path.exists() and any(path.iterdir()):
            vectorstores[strategy] = Chroma(
                collection_name=collection,
                embedding_function=embeddings,
                persist_directory=chroma_dir
            )
            count = vectorstores[strategy]._collection.count()
            logger.info(f"Colección '{strategy}' cargada: {count} vectores")
        else:
            logger.warning(
                f"Colección '{strategy}' no encontrada en '{chroma_dir}'. "
                "Ejecuta rag_ingestion_pipeline.py primero."
            )
    
    return vectorstores


def evaluate_strategy(
    questions: list,
    vectorstore: Chroma,
    strategy_name: str,
    k: int = 5
) -> list:
    """
    Evalúa una estrategia de chunking con el conjunto de preguntas.
    
    Returns:
        Lista de dicts con pregunta, respuesta, fuentes y métricas básicas.
    """
    results = []
    
    for i, question in enumerate(questions, 1):
        logger.info(
            f"[{strategy_name}] Pregunta {i}/{len(questions)}: "
            f"{question[:60]}..."
        )
        
        try:
            result = answer_question(
                query=question,
                vectorstore=vectorstore,
                k=k,
                use_mmr=True
            )
            
            # Extraer información de fuentes
            sources_info = []
            for doc in result["sources"]:
                sources_info.append({
                    "source": doc.metadata.get("source", "N/A"),
                    "page": doc.metadata.get("page", "N/A"),
                    "chunk_size": doc.metadata.get("chunk_size_chars", 0),
                    "preview": doc.page_content[:100] + "..."
                })
            
            results.append({
                "question_id": i,
                "question": question,
                "answer": result["answer"],
                "chunks_used": result["chunks_used"],
                "sources": sources_info,
                "answer_length": len(result["answer"]),
                "has_no_info_response": "No encontré información" in result["answer"]
            })
            
        except Exception as e:
            logger.error(f"  Error en pregunta {i}: {e}")
            results.append({
                "question_id": i,
                "question": question,
                "answer": f"ERROR: {str(e)}",
                "chunks_used": 0,
                "sources": [],
                "answer_length": 0,
                "has_no_info_response": False
            })
    
    return results


def generate_comparison_report(
    semantic_results: list,
    fixed_results: list,
    output_file: str = "evaluation_report.json"
) -> dict:
    """
    Genera un reporte comparativo entre ambas estrategias.
    
    Métricas calculadas:
    - Longitud promedio de respuesta (proxy de completitud)
    - Tasa de respuestas "sin información" (indica cobertura)
    - Número promedio de chunks utilizados
    - Diversidad de fuentes por respuesta
    """
    
    def compute_metrics(results: list) -> dict:
        total = len(results)
        if total == 0:
            return {}
        
        avg_answer_len = sum(r["answer_length"] for r in results) / total
        no_info_rate = sum(1 for r in results if r["has_no_info_response"]) / total
        avg_chunks = sum(r["chunks_used"] for r in results) / total
        
        # Diversidad de fuentes: cuántos archivos distintos aparecen por respuesta
        source_diversities = []
        for r in results:
            unique_sources = len(set(s["source"] for s in r["sources"]))
            source_diversities.append(unique_sources)
        avg_source_diversity = sum(source_diversities) / total if source_diversities else 0
        
        return {
            "total_questions": total,
            "avg_answer_length_chars": round(avg_answer_len, 1),
            "no_info_response_rate": round(no_info_rate, 3),
            "avg_chunks_used": round(avg_chunks, 2),
            "avg_source_diversity": round(avg_source_diversity, 2)
        }
    
    report = {
        "evaluation_timestamp": datetime.now().isoformat(),
        "strategies_compared": ["semantic_chunking", "fixed_size_chunking"],
        "metrics": {
            "semantic": compute_metrics(semantic_results),
            "fixed": compute_metrics(fixed_results)
        },
        "per_question_comparison": []
    }
    
    # Comparación pregunta por pregunta
    for s_result, f_result in zip(semantic_results, fixed_results):
        report["per_question_comparison"].append({
            "question_id": s_result["question_id"],
            "question": s_result["question"],
            "semantic": {
                "answer": s_result["answer"],
                "chunks_used": s_result["chunks_used"],
                "answer_length": s_result["answer_length"],
                "sources_count": len(s_result["sources"])
            },
            "fixed": {
                "answer": f_result["answer"],
                "chunks_used": f_result["chunks_used"],
                "answer_length": f_result["answer_length"],
                "sources_count": len(f_result["sources"])
            }
        })
    
    # Guardar reporte en JSON
    with open(output_file, "w", encoding="utf-8") as f:
        json.dump(report, f, ensure_ascii=False, indent=2)
    
    logger.info(f"Reporte guardado en: {output_file}")
    return report


def print_summary(report: dict) -> None:
    """Imprime un resumen legible del reporte comparativo."""
    print("\n" + "=" * 70)
    print("REPORTE COMPARATIVO: SEMANTIC CHUNKING vs. FIXED-SIZE CHUNKING")
    print("=" * 70)
    
    metrics = report["metrics"]
    
    print(f"\n{'Métrica':<40} {'Semantic':>12} {'Fixed-size':>12}")
    print("-" * 66)
    
    metric_labels = {
        "total_questions": "Total preguntas evaluadas",
        "avg_answer_length_chars": "Long. promedio respuesta (chars)",
        "no_info_response_rate": "Tasa 'sin información' (0=mejor)",
        "avg_chunks_used": "Chunks promedio por respuesta",
        "avg_source_diversity": "Diversidad promedio de fuentes"
    }
    
    for key, label in metric_labels.items():
        sem_val = metrics["semantic"].get(key, "N/A")
        fix_val = metrics["fixed"].get(key, "N/A")
        print(f"{label:<40} {str(sem_val):>12} {str(fix_val):>12}")
    
    print("\n" + "=" * 70)
    print("MUESTRA DE RESPUESTAS COMPARATIVAS (primeras 3 preguntas)")
    print("=" * 70)
    
    for item in report["per_question_comparison"][:3]:
        print(f"\n[Q{item['question_id']}] {item['question']}")
        print(f"\n  SEMANTIC ({item['semantic']['chunks_used']} chunks):")
        print(f"  {item['semantic']['answer'][:300]}...")
        print(f"\n  FIXED ({item['fixed']['chunks_used']} chunks):")
        print(f"  {item['fixed']['answer'][:300]}...")
        print("\n" + "-" * 70)


# ─────────────────────────────────────────────
# EJECUCIÓN PRINCIPAL
# ─────────────────────────────────────────────

if __name__ == "__main__":
    print("Iniciando evaluación comparativa...")
    print("Verificando vectorstores existentes...")
    
    vectorstores = load_existing_vectorstores()
    
    # Si no existen vectorstores, ejecutar ingesta primero
    if len(vectorstores) < 2:
        print("\nVectorstores no encontrados. Ejecutando pipeline de ingesta...")
        vectorstores = run_ingestion_pipeline()
    
    print(f"\nEvaluando {len(EVALUATION_QUESTIONS)} preguntas con cada estrategia...")
    print("Estrategia 1: Semantic Chunking")
    semantic_results = evaluate_strategy(
        EVALUATION_QUESTIONS,
        vectorstores["semantic"],
        "Semantic"
    )
    
    print("\nEstrategia 2: Fixed-size Chunking")
    fixed_results = evaluate_strategy(
        EVALUATION_QUESTIONS,
        vectorstores["fixed"],
        "Fixed"
    )
    
    # Generar y mostrar reporte
    report = generate_comparison_report(
        semantic_results,
        fixed_results,
        output_file="evaluation_report.json"
    )
    
    print_summary(report)
    print("\n✓ Evaluación completada. Reporte guardado en 'evaluation_report.json'")
```

#### Resultado esperado

```
======================================================================
REPORTE COMPARATIVO: SEMANTIC CHUNKING vs. FIXED-SIZE CHUNKING
======================================================================

Métrica                                    Semantic   Fixed-size
------------------------------------------------------------------
Total preguntas evaluadas                        10           10
Long. promedio respuesta (chars)              XXX.X        YYY.Y
Tasa 'sin información' (0=mejor)              0.XXX        0.YYY
Chunks promedio por respuesta                  X.XX         Y.YY
Diversidad promedio de fuentes                 X.XX         Y.YY
```

#### Verificación

```bash
python evaluation_comparison.py
ls -la evaluation_report.json
```

---

## 7. Validación y Pruebas

Ejecuta la siguiente secuencia completa para validar que todos los módulos funcionan correctamente:

```bash
# Test 1: Verificar que el pipeline de ingesta completo funciona
echo "=== Test 1: Pipeline de ingesta ==="
python rag_ingestion_pipeline.py

# Test 2: Verificar persistencia de ChromaDB
echo "=== Test 2: Persistencia ChromaDB ==="
python -c "
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
import os

embeddings = OpenAIEmbeddings(
    model='text-embedding-3-small',
    openai_api_key=os.getenv('OPENAI_API_KEY')
)

for strategy, chroma_dir, collection in [
    ('semantic', './chroma_db', 'rag_semantic_chunks'),
    ('fixed', './chroma_db_fixed', 'rag_fixed_chunks')
]:
    vs = Chroma(
        collection_name=collection,
        embedding_function=embeddings,
        persist_directory=chroma_dir
    )
    count = vs._collection.count()
    print(f'{strategy}: {count} vectores almacenados')
    assert count > 0, f'ERROR: Colección {strategy} vacía'

print('✓ Ambas colecciones tienen datos')
"

# Test 3: Verificar recuperación MMR
echo "=== Test 3: Recuperación MMR ==="
python -c "
import os
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from rag_ingestion_pipeline import retrieve_context

embeddings = OpenAIEmbeddings(
    model='text-embedding-3-small',
    openai_api_key=os.getenv('OPENAI_API_KEY')
)
vs = Chroma(
    collection_name='rag_semantic_chunks',
    embedding_function=embeddings,
    persist_directory='./chroma_db'
)

docs = retrieve_context('¿Qué es RAG?', vs, k=3, use_mmr=True)
print(f'Chunks recuperados: {len(docs)}')
for i, doc in enumerate(docs):
    print(f'  Chunk {i+1}: {doc.metadata.get(\"source\", \"N/A\")} p.{doc.metadata.get(\"page\", \"?\")} — {len(doc.page_content)} chars')
assert len(docs) == 3, 'ERROR: No se recuperaron 3 chunks'
print('✓ MMR funciona correctamente')
"

# Test 4: Verificar generación de respuesta
echo "=== Test 4: Generación RAG ==="
python -c "
import os
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from rag_ingestion_pipeline import answer_question

embeddings = OpenAIEmbeddings(
    model='text-embedding-3-small',
    openai_api_key=os.getenv('OPENAI_API_KEY')
)
vs = Chroma(
    collection_name='rag_semantic_chunks',
    embedding_function=embeddings,
    persist_directory='./chroma_db'
)

result = answer_question('¿Cómo funciona la recuperación semántica?', vs, k=3)
print(f'Respuesta generada: {len(result[\"answer\"])} chars')
print(f'Chunks utilizados: {result[\"chunks_used\"]}')
assert len(result['answer']) > 50, 'ERROR: Respuesta demasiado corta'
assert result['chunks_used'] > 0, 'ERROR: No se usaron chunks'
print('✓ Generación RAG funciona correctamente')
"

# Test 5: Evaluación comparativa completa
echo "=== Test 5: Evaluación comparativa ==="
python evaluation_comparison.py

echo ""
echo "✓ TODOS LOS TESTS COMPLETADOS"
```

---

## 8. Solución de Problemas

### Problema 1: `ValueError: Could not connect to tenant default_tenant`

**Síntomas:**
```
chromadb.errors.InvalidCollectionException: Collection rag_semantic_chunks does not exist.
```
o
```
ValueError: Could not connect to tenant default_tenant. Are you sure it exists?
```

**Causa:**
ChromaDB 0.5.x cambió su API interna de gestión de colecciones. El directorio de persistencia existe pero fue creado con una versión incompatible de ChromaDB, o la colección fue creada con un nombre diferente al que se intenta cargar.

**Solución:**
```bash
# Paso 1: Verificar la versión instalada de ChromaDB
python -c "import chromadb; print(chromadb.__version__)"

# Paso 2: Si la versión no es 0.5.x, reinstalar
pip install "chromadb==0.5.5" --force-reinstall

# Paso 3: Eliminar las colecciones corruptas y reconstruir
rm -rf chroma_db/ chroma_db_fixed/

# Paso 4: Re-ejecutar el pipeline con force_rebuild=True
python -c "
from rag_ingestion_pipeline import run_ingestion_pipeline
run_ingestion_pipeline(force_rebuild=True)
"
```

---

### Problema 2: `SemanticChunker` produce un número muy bajo de chunks (1–3 para todo el corpus)

**Síntomas:**
El log muestra algo como:
```
[Semantic Chunking] ✓ 2 chunks generados
Tamaño máx: 45000 chars | Promedio: 22500 chars
```
Los chunks semánticos son enormes y la recuperación devuelve contexto irrelevante.

**Causa:**
El parámetro `breakpoint_threshold_amount` es demasiado bajo (por ejemplo, 50 para el tipo `percentile`), lo que significa que el chunker detecta muy pocos puntos de quiebre semántico y agrupa casi todo el texto en un solo chunk. Esto ocurre frecuentemente con PDFs técnicos que tienen terminología muy consistente a lo largo del documento.

**Solución:**
```python
# Ajustar el umbral en chunk_semantic() para producir más chunks
# Aumentar breakpoint_threshold_amount para ser más agresivo en los cortes

# Opción 1: Subir el percentil (más puntos de quiebre)
semantic_chunks = chunk_semantic(
    documents,
    breakpoint_threshold_type="percentile",
    breakpoint_threshold_amount=85.0  # Reducido desde 95.0
)

# Opción 2: Cambiar a standard_deviation para mayor sensibilidad
semantic_chunks = chunk_semantic(
    documents,
    breakpoint_threshold_type="standard_deviation",
    breakpoint_threshold_amount=1.5
)

# Verificar el resultado
print(f"Chunks generados: {len(semantic_chunks)}")
print(f"Tamaño promedio: {sum(len(c.page_content) for c in semantic_chunks)//len(semantic_chunks)} chars")
# Objetivo: entre 50 y 500 chunks para un corpus de 3 PDFs de ~20 páginas
```

Si el problema persiste, considera aplicar `chunk_fixed_size` como pre-procesamiento antes de `chunk_semantic` para evitar que páginas muy largas sean enviadas como una sola unidad al SemanticChunker.

---

## 9. Limpieza

Ejecuta los siguientes comandos al finalizar el laboratorio para liberar recursos:

```bash
# Opción A: Conservar las colecciones ChromaDB (recomendado si continuarás con Lab 06)
# Solo desactivar el entorno virtual
deactivate

# Opción B: Limpieza completa (elimina datos y entorno virtual)
# ADVERTENCIA: Esto eliminará las colecciones ChromaDB creadas
deactivate
rm -rf chroma_db/ chroma_db_fixed/
rm -rf .venv/

# Verificar espacio recuperado
du -sh lab05_rag_pipeline/ 2>/dev/null || echo "Directorio limpiado"

# Opción C: Conservar solo el código, eliminar datos grandes
rm -rf chroma_db/ chroma_db_fixed/
rm -rf pdfs/
echo "Datos eliminados. Código fuente conservado."
```

> 💡 **Nota:** Si planeas continuar con el Lab 06 (que extiende este pipeline), conserva el directorio `chroma_db/` y los PDFs. El pipeline detecta automáticamente colecciones existentes y no regenera embeddings innecesariamente.

---

## 10. Resumen

### Lo que construiste

En este laboratorio implementaste un **pipeline RAG de nivel producción** con los siguientes componentes:

| Componente | Implementación | Archivo |
|---|---|---|
| Carga de PDFs | `PyPDFLoader` + metadata + deduplicación MD5 | `rag_ingestion_pipeline.py` |
| Fixed Chunking | `RecursiveCharacterTextSplitter` (baseline) | `rag_ingestion_pipeline.py` |
| Semantic Chunking | `SemanticChunker` con embeddings OpenAI | `rag_ingestion_pipeline.py` |
| Almacenamiento vectorial | ChromaDB 0.5.x con persistencia en disco | `rag_ingestion_pipeline.py` |
| Recuperación MMR | `Chroma.as_retriever(search_type="mmr")` | `rag_ingestion_pipeline.py` |
| Generación RAG | `RetrievalQA` + `GPT-4o-mini` + prompt template | `rag_ingestion_pipeline.py` |
| Evaluación comparativa | 10 preguntas × 2 estrategias + métricas | `evaluation_comparison.py` |

### Conceptos clave reforzados

- **Semantic Chunking** produce fragmentos basados en quiebres de significado semántico, no en límites de caracteres. Esto preserva la coherencia conceptual de cada chunk, mejorando la precisión de la recuperación.
- **MMR (Maximal Marginal Relevance)** balancea relevancia y diversidad en la recuperación, evitando que el contexto enviado al LLM esté dominado por fragmentos casi idénticos del mismo párrafo.
- **La deduplicación por MD5** previene indexar el mismo contenido múltiples veces, lo que degradaría la calidad de las búsquedas por similitud.
- **La persistencia en ChromaDB** permite separar la fase de ingesta (costosa en tiempo y tokens) de la fase de consulta (rápida), que es el patrón correcto para sistemas en producción.

### Recursos adicionales

- [Documentación oficial de SemanticChunker en LangChain](https://python.langchain.com/docs/how_to/semantic-chunker/)
- [ChromaDB — Guía de colecciones y persistencia](https://docs.trychroma.com/guides)
- [Artículo: "Chunking Strategies for LLM Applications" — Pinecone](https://www.pinecone.io/learn/chunking-strategies/)
- [MMR: "The Use of MMR, Diversity-Based Reranking" — Carbonell & Goldstein (1998)](https://dl.acm.org/doi/10.1145/290941.291025)
- [OpenAI Embeddings API — text-embedding-3-small](https://platform.openai.com/docs/guides/embeddings/embedding-models)

---
