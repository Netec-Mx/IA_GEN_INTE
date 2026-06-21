<div align="center">

# 🧪 Laboratorio 5

## Pipeline de ingesta RAG con Semantic Chunking, ChromaDB y evaluación comparativa

![Nivel](https://img.shields.io/badge/Nivel-Intermedio--Alto-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)
![VectorDB](https://img.shields.io/badge/Vector%20DB-ChromaDB-DC2626?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio vas a construir un pipeline RAG completo: cargarás documentos PDF, crearás chunks con dos estrategias, generarás embeddings, persistirás vectores en ChromaDB, recuperarás contexto con MMR y compararás la calidad de respuestas entre **Fixed-size Chunking** y **Semantic Chunking**. No uses documentos sensibles reales ni compartas tu API key.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Ingesta y recuperación RAG</td>
<td width="25%"><strong>⏱️ Duración</strong><br>58 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar, evaluar y crear</td>
<td width="25%"><strong>📦 Entregable</strong><br>Pipeline + ChromaDB + reporte comparativo</td>
</tr>
</table>

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a construir un pipeline completo de **Retrieval-Augmented Generation (RAG)** para consultar documentos técnicos en PDF. El objetivo no es solo “preguntarle a un PDF”, sino entender cómo se construye una base documental consultable desde cero.

Trabajarás en **Windows**, con **Visual Studio Code** y **Git Bash**. Primero generarás documentos PDF sintéticos para que la práctica sea reproducible. Después cargarás los documentos, enriquecerás metadata, aplicarás dos estrategias de chunking, generarás embeddings, guardarás vectores en ChromaDB y crearás un asistente documental que responde únicamente con información recuperada del corpus.

El laboratorio compara dos estrategias:

1. **Fixed-size Chunking**, como baseline simple por tamaño de caracteres.
2. **Semantic Chunking**, como estrategia basada en cambios de significado usando embeddings.

Al finalizar, tendrás evidencia técnica para explicar cómo el chunking impacta la recuperación, las fuentes usadas, la completitud de la respuesta y la calidad percibida.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Preparar un entorno local en Windows para construir un pipeline RAG.
2. Generar documentos PDF técnicos de prueba sin depender de descargas externas.
3. Cargar PDFs y enriquecer metadata por página.
4. Detectar contenido duplicado mediante hash MD5.
5. Crear chunks de tamaño fijo con `RecursiveCharacterTextSplitter`.
6. Crear chunks semánticos con `SemanticChunker`.
7. Generar embeddings con `text-embedding-3-small`.
8. Persistir vectores en ChromaDB usando colecciones separadas.
9. Implementar recuperación de contexto con similitud y MMR.
10. Generar respuestas RAG usando contexto documental.
11. Comparar Fixed-size Chunking contra Semantic Chunking.
12. Evaluar calidad con una matriz manual.
13. Identificar costos, riesgos y límites de un pipeline RAG.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Tener conocimientos básicos de Python.
2. Conocer el concepto de embeddings.
3. Comprender qué es similitud semántica.
4. Conocer el flujo general de RAG: ingesta, chunking, embeddings, vector store, retrieval y generación.
5. Haber usado Visual Studio Code.
6. Haber usado Git Bash en Windows.
7. Tener una API key válida de OpenAI.
8. Tener conexión a internet.
9. No usar documentos sensibles reales durante la práctica.

---

### 💻 Hardware

| Recurso | Requisito mínimo | Recomendado |
|---|---:|---:|
| Equipo | Laptop o PC con Windows | Laptop o PC con Windows 11 |
| Procesador | Intel Core i5 / Ryzen 5 | Intel Core i7 / Ryzen 7 |
| Memoria RAM | 8 GB | 16 GB o más |
| Almacenamiento libre | 2 GB | 5 GB |
| GPU | No requerida | No requerida |
| Internet | 10 Mbps | 25 Mbps o más |

---

### 🧰 Software

| Software | Uso |
|---|---|
| Windows 10/11 | Sistema operativo del laboratorio |
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Python 3.12 recomendado. No uses Python 3.14 para esta práctica. | Runtime principal |
| pip | Instalación de dependencias |
| OpenAI API key | Embeddings y generación de respuestas |
| ChromaDB | Base vectorial local persistente |
| LangChain | Orquestación de loaders, splitters, retrievers y cadenas RAG |
| Excel, Google Sheets o LibreOffice Calc | Evaluación manual de resultados |

---

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 58 minutos |
| Complejidad | Intermedia - Alta |
| Nivel de Bloom | Aplicar, analizar, evaluar y crear |
| Ubicación recomendada | Después de la lección de fundamentos de RAG |
| Modalidad | Individual o equipos de 2 personas |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Proveedor usado | OpenAI |
| Modelo de embeddings | `text-embedding-3-small` |
| Modelo de generación | Configurable mediante `.env` |
| Vector store | ChromaDB local |
| Entregable principal | Pipeline RAG funcional |
| Entregable secundario | Reporte comparativo de chunking |

---

## 🛡️ Consideraciones para estudiantes

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>No subas `.env` ni documentos sensibles.</td>
<td><strong>💸 Costo</strong><br>Semantic Chunking y embeddings consumen API.</td>
<td><strong>📄 Documentos</strong><br>Usa PDFs sintéticos o públicos.</td>
</tr>
</table>

1. No compartas tu API key.
2. No escribas claves dentro del código.
3. No uses documentos confidenciales.
4. Ejecuta primero validaciones sin API.
5. Recuerda que Semantic Chunking consume embeddings.
6. Si cambias documentos o parámetros, reconstruye ChromaDB.
7. Una respuesta más larga no siempre es mejor.
8. Revisa si las respuestas están realmente soportadas por fuentes.
9. Entrega evidencias sin secretos.

---

## 🔗 Fuentes oficiales que debes revisar antes de ejecutar

| Tema | Qué revisar | Fuente sugerida |
|---|---|---|
| OpenAI embeddings | Modelo disponible, precio y uso | OpenAI Embeddings / Models |
| LangChain Chroma | Paquete e imports actuales | LangChain Chroma integration |
| Retrieval chain | Uso de `create_retrieval_chain` | LangChain retrieval chains |
| SemanticChunker | Parámetros de corte semántico | LangChain SemanticChunker |
| ChromaDB | Persistencia local y colecciones | ChromaDB docs |

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear una carpeta de trabajo, abrirla en VS Code, preparar un entorno virtual e instalar las dependencias necesarias.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear una carpeta dedicada para código, PDFs, colecciones vectoriales y reportes.

**⚙️ Contenido del paso:**

```bash
mkdir -p ~/labs-ia-gen/lab-05-rag-semantic-chunking
cd ~/labs-ia-gen/lab-05-rag-semantic-chunking
```

**✅ Validación del paso:**

```bash
pwd
```

**📌 Resultado esperado:**

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-05-rag-semantic-chunking
```

---

### ✅ Paso 2. Abre la carpeta en VS Code

**📝 Descripción del paso:**  
Vas a abrir el proyecto desde Git Bash.

**⚙️ Contenido del paso:**

```bash
code .
```

Si no funciona, abre VS Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-05-rag-semantic-chunking
```

**✅ Validación del paso:**  
Confirma que VS Code muestre la carpeta del laboratorio.

**📌 Resultado esperado:**  
El proyecto está abierto en Visual Studio Code.

---

### ✅ Paso 3. Crea y activa el entorno virtual

**📝 Descripción del paso:**  
Vas a aislar dependencias para no afectar otros proyectos.

**⚙️ Contenido del paso:**

- Si te pide confirmación escribe **Yes**
  
```bash
winget install Python.Python.3.12
```
```bash
py -3.12 -m venv .venv
source .venv/Scripts/activate
python --version
```

**✅ Validación del paso:**

```bash
python --version
which python
```

**📌 Resultado esperado:**  
La ruta de Python apunta a `.venv`.

---

### ✅ Paso 4. Crea `requirements.txt`

**📝 Descripción del paso:**  
Vas a declarar las dependencias del pipeline.

**⚙️ Contenido del paso:**

```bash
cat > requirements.txt << 'REQ'
langchain>=0.3,<0.4
langchain-community>=0.3,<0.4
langchain-openai>=0.3,<0.4
langchain-experimental>=0.3,<0.4
langchain-chroma>=0.2,<0.3
langchain-text-splitters>=0.3,<0.4
chromadb>=0.5,<0.7
pypdf>=5,<7
openai>=1.90,<2
python-dotenv>=1.0,<2
tiktoken>=0.7,<1
reportlab>=4,<5
pandas>=2,<3
tabulate>=0.9,<1
REQ
```

**✅ Validación del paso:**

```bash
cat requirements.txt
```

**📌 Resultado esperado:**  
El archivo contiene todas las dependencias.

---

### ✅ Paso 5. Instala dependencias

**📝 Descripción del paso:**  
Vas a instalar LangChain, ChromaDB, OpenAI, lectura de PDFs y utilidades.

**⚙️ Contenido del paso:**

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
pip check
```

**✅ Validación del paso:**

```bash
python -c "import langchain; import chromadb; import dotenv; import reportlab; print('Dependencias instaladas correctamente')"
```

**📌 Resultado esperado:**

```text
Dependencias instaladas correctamente
```

---

### ✅ Paso 6. Crea carpetas del proyecto

**📝 Descripción del paso:**  
Vas a crear carpetas para documentos, bases vectoriales y reportes.

**⚙️ Contenido del paso:**

```bash
mkdir -p pdfs chroma_db_semantic chroma_db_fixed reports
```

**✅ Validación del paso:**

```bash
ls -la
```

**📌 Resultado esperado:**

```text
pdfs/
chroma_db_semantic/
chroma_db_fixed/
reports/
```

---

### ✅ Paso 7. Crea `.env` y `.gitignore`

**📝 Descripción del paso:**  
Vas a configurar tu API key y proteger archivos sensibles.

**⚙️ Contenido del paso:**

```bash
cat > .env << 'ENV'
OPENAI_API_KEY=REEMPLAZA_CON_TU_API_KEY
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_CHAT_MODEL=gpt-4o-mini
ENV

cat > .gitignore << 'GITIGNORE'
.env
.venv/
__pycache__/
*.pyc
*.pyo
.pytest_cache/
chroma_db_semantic/
chroma_db_fixed/
reports/
pdfs/*.pdf
*.log
GITIGNORE
```

Abre `.env` en VS Code y reemplaza `REEMPLAZA_CON_TU_API_KEY` por tu API key real.

**✅ Validación del paso:**

```bash
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('OPENAI_API_KEY configurada:', bool(os.getenv('OPENAI_API_KEY')))"
```

**📌 Resultado esperado:**

```text
OPENAI_API_KEY configurada: True
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20de%20un%20laboratorio%20RAG.%20Prepar%C3%A9%20un%20proyecto%20en%20Windows%20con%20VSCode%2C%20Git%20Bash%2C%20entorno%20virtual%2C%20requirements.txt%2C%20.env%2C%20.gitignore%20y%20carpetas%20para%20PDFs%2C%20ChromaDB%20y%20reportes.)

---

# 🧩 Tarea 2. Generar documentos PDF técnicos de prueba

## 🎯 Objetivo de la tarea

Crear PDFs técnicos locales para que el laboratorio sea reproducible y no dependa de descargas externas.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el script generador de PDFs

**📝 Descripción del paso:**  
Vas a generar tres PDFs sintéticos con contenido técnico suficiente para probar RAG.

**⚙️ Contenido del paso:**  
Crea `00_crear_pdfs_prueba.py`:

```python
from pathlib import Path
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
from reportlab.lib.units import inch

PDF_DIR = Path("pdfs")
PDF_DIR.mkdir(exist_ok=True)

DOCUMENTOS = {
    "rag_fundamentos.pdf": {
        "titulo": "Fundamentos de Retrieval-Augmented Generation",
        "paginas": [
            """Retrieval-Augmented Generation, conocido como RAG, combina recuperación de información con generación de texto. El objetivo principal es permitir que un modelo de lenguaje responda con base en documentos externos, en lugar de depender únicamente de su conocimiento interno.

Un sistema RAG típico tiene dos fases. La primera fase es la ingesta documental, donde los archivos se cargan, se dividen en fragmentos, se convierten en embeddings y se almacenan en una base vectorial. La segunda fase es la consulta, donde una pregunta del usuario se transforma en embedding, se buscan fragmentos relevantes y se envían al modelo como contexto.

RAG es útil cuando la información cambia con frecuencia, cuando se requiere trazabilidad documental o cuando no es viable reentrenar un modelo completo.""",
            """La calidad de un sistema RAG depende de varios factores. El primero es la calidad de los documentos de entrada. El segundo es la estrategia de chunking. El tercero es la calidad del modelo de embeddings. El cuarto es la recuperación de contexto. El quinto es la instrucción usada para generar la respuesta.

Un error común consiste en crear chunks demasiado pequeños, lo que fragmenta ideas completas. Otro error consiste en crear chunks demasiado grandes, lo que introduce ruido y reduce precisión. El balance depende del tipo de documento y del caso de uso.

Los sistemas RAG deben responder únicamente con información soportada por el contexto recuperado. Si el contexto no contiene la respuesta, el asistente debe indicarlo explícitamente.""",
            """La evaluación de un sistema RAG puede realizarse con métricas automáticas y revisión humana. Algunas métricas útiles son relevancia del contexto, fidelidad de la respuesta, cobertura, precisión de citas y tasa de respuestas sin información.

La evaluación humana sigue siendo importante porque una respuesta larga no necesariamente es correcta. Un buen sistema RAG debe responder de forma precisa, citar fuentes y evitar inventar información que no aparece en los documentos.

En producción se recomienda monitorear consultas fallidas, documentos recuperados, latencia, costo por consulta y cambios en la calidad de respuesta.""",
        ],
    },
    "api_design.pdf": {
        "titulo": "Diseño de APIs REST Empresariales",
        "paginas": [
            """Una API REST permite exponer recursos mediante operaciones HTTP. Los métodos más comunes son GET para consultar información, POST para crear recursos, PUT o PATCH para actualizar recursos y DELETE para eliminar recursos.

Los códigos de estado ayudan a comunicar el resultado de una operación. Un código 200 indica éxito, 201 indica creación, 400 indica solicitud incorrecta, 401 indica falta de autenticación, 403 indica falta de autorización, 404 indica recurso no encontrado y 500 indica error interno del servidor.

Una API empresarial debe ser consistente, versionada, segura, observable y documentada.""",
            """La autenticación y autorización son elementos centrales en el diseño de APIs. Las estrategias comunes incluyen API keys, OAuth 2.0, JWT y autenticación basada en sesiones.

Las API keys son simples, pero deben tratarse como secretos. OAuth 2.0 es adecuado para delegación de permisos. JWT permite transportar claims, pero debe validarse firma, expiración, emisor y audiencia.

La autorización debe aplicarse por recurso y operación. No basta con autenticar al usuario; también se debe validar si tiene permisos para acceder a cada recurso.""",
            """La paginación evita devolver grandes volúmenes de datos en una sola respuesta. Existen estrategias de paginación por offset, por cursor y por token. La paginación por cursor suele ser más estable para datos que cambian frecuentemente.

El rate limiting protege a la API contra abuso, errores de clientes y picos de tráfico. Puede aplicarse por usuario, por IP, por API key o por tenant.

La observabilidad debe incluir métricas de latencia, tasa de error, volumen de solicitudes, trazas distribuidas y logs estructurados.""",
        ],
    },
    "embeddings_vectorstores.pdf": {
        "titulo": "Embeddings y Bases Vectoriales",
        "paginas": [
            """Los embeddings son representaciones numéricas de texto que capturan significado semántico. Dos textos con significado similar tienden a tener vectores cercanos en el espacio embedding.

Los embeddings se usan en búsqueda semántica, clustering, clasificación, recomendación, detección de anomalías y recuperación de información. En RAG, los embeddings permiten buscar fragmentos relevantes aunque la pregunta no use exactamente las mismas palabras del documento.

La similitud coseno es una medida común para comparar embeddings. Mientras más cercana sea la dirección de dos vectores, mayor será su similitud.""",
            """Una base vectorial almacena embeddings junto con metadata. La metadata permite filtrar resultados por fuente, página, fecha, categoría, idioma o permisos de acceso.

ChromaDB es una base vectorial local que permite crear colecciones persistentes. En laboratorios y prototipos es útil porque no requiere infraestructura externa. En producción se debe evaluar escalabilidad, seguridad, backups, aislamiento de tenants y monitoreo.

La persistencia en disco evita recalcular embeddings cada vez que se reinicia la aplicación.""",
            """Maximal Marginal Relevance, o MMR, es una técnica de recuperación que busca balancear relevancia y diversidad. En lugar de devolver únicamente los fragmentos más similares a la pregunta, MMR intenta evitar resultados redundantes.

El parámetro lambda controla el balance. Valores cercanos a 1 priorizan relevancia. Valores cercanos a 0 priorizan diversidad. En sistemas RAG, MMR puede mejorar el contexto cuando varios chunks similares provienen de la misma sección del documento.

MMR no garantiza por sí solo respuestas correctas; debe combinarse con buen chunking, metadata útil y prompts que obliguen a responder con base en el contexto.""",
        ],
    },
}

def escribir_lineas(c: canvas.Canvas, texto: str, x: float, y: float, max_chars: int = 92) -> float:
    c.setFont("Helvetica", 10)
    for parrafo in texto.strip().split("\n"):
        parrafo = parrafo.strip()
        if not parrafo:
            y -= 12
            continue
        palabras = parrafo.split()
        linea = ""
        for palabra in palabras:
            candidata = f"{linea} {palabra}".strip()
            if len(candidata) > max_chars:
                c.drawString(x, y, linea)
                y -= 14
                linea = palabra
            else:
                linea = candidata
        if linea:
            c.drawString(x, y, linea)
            y -= 14
        y -= 8
    return y

def crear_pdf(nombre_archivo: str, titulo: str, paginas: list[str]) -> None:
    ruta = PDF_DIR / nombre_archivo
    c = canvas.Canvas(str(ruta), pagesize=letter)
    _, height = letter
    for index, contenido in enumerate(paginas, start=1):
        c.setFont("Helvetica-Bold", 16)
        c.drawString(0.8 * inch, height - 0.8 * inch, titulo)
        c.setFont("Helvetica-Bold", 11)
        c.drawString(0.8 * inch, height - 1.1 * inch, f"Página {index}")
        escribir_lineas(c, contenido, 0.8 * inch, height - 1.5 * inch)
        c.showPage()
    c.save()
    print(f"PDF creado: {ruta}")

if __name__ == "__main__":
    for nombre, config in DOCUMENTOS.items():
        crear_pdf(nombre, config["titulo"], config["paginas"])
    print("PDFs de prueba generados correctamente en la carpeta pdfs/.")
```

**✅ Validación del paso:**

```bash
python -m py_compile 00_crear_pdfs_prueba.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Ejecuta el generador

**📝 Descripción del paso:**  
Vas a crear la base documental.

**⚙️ Contenido del paso:**

```bash
python 00_crear_pdfs_prueba.py
ls -lh pdfs/
```

**✅ Validación del paso:**  
Deben existir los tres PDFs.

**📌 Resultado esperado:**

```text
rag_fundamentos.pdf
api_design.pdf
embeddings_vectorstores.pdf
```

---

### ✅ Paso 3. Verifica lectura de PDFs

**📝 Descripción del paso:**  
Vas a confirmar que los PDFs tienen páginas legibles.

**⚙️ Contenido del paso:**

```bash
python -c "
from pathlib import Path
from pypdf import PdfReader
for pdf in Path('pdfs').glob('*.pdf'):
    reader = PdfReader(str(pdf))
    print(f'{pdf.name}: {len(reader.pages)} páginas')
"
```

**✅ Validación del paso:**  
Cada PDF debe mostrar 3 páginas.

**📌 Resultado esperado:**  
Los documentos están listos para la ingesta.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20de%20un%20laboratorio%20RAG.%20Gener%C3%A9%20PDFs%20t%C3%A9cnicos%20sint%C3%A9ticos%20con%20ReportLab%20para%20tener%20una%20base%20documental%20local%20y%20reproducible.)

---

# 🧩 Tarea 3. Crear el módulo principal del pipeline RAG

## 🎯 Objetivo de la tarea

Crear el archivo principal con configuración, carga de variables de entorno, constantes y función de carga de PDFs con metadata enriquecida.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `rag_pipeline.py`

**📝 Descripción del paso:**  
Vas a crear el archivo central del pipeline.

**⚙️ Contenido del paso:**

```python
from __future__ import annotations

import hashlib
import json
import logging
import os
import shutil
from datetime import datetime
from pathlib import Path
from typing import Any

from dotenv import load_dotenv
from langchain_chroma import Chroma
from langchain_community.document_loaders import PyPDFLoader
from langchain_core.documents import Document
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_experimental.text_splitter import SemanticChunker
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)
load_dotenv()

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
OPENAI_EMBEDDING_MODEL = os.getenv("OPENAI_EMBEDDING_MODEL", "text-embedding-3-small")
OPENAI_CHAT_MODEL = os.getenv("OPENAI_CHAT_MODEL", "gpt-4o-mini")

PDF_DIRECTORY = "pdfs"
CHROMA_DIR_SEMANTIC = "chroma_db_semantic"
CHROMA_DIR_FIXED = "chroma_db_fixed"
COLLECTION_SEMANTIC = "rag_semantic_chunks"
COLLECTION_FIXED = "rag_fixed_chunks"


def require_openai_key() -> str:
    if not OPENAI_API_KEY or OPENAI_API_KEY == "REEMPLAZA_CON_TU_API_KEY":
        raise RuntimeError("OPENAI_API_KEY no está configurada. Edita .env antes de ejecutar esta tarea.")
    return OPENAI_API_KEY


def compute_md5(text: str) -> str:
    return hashlib.md5(text.encode("utf-8")).hexdigest()


def get_embeddings() -> OpenAIEmbeddings:
    return OpenAIEmbeddings(model=OPENAI_EMBEDDING_MODEL, api_key=require_openai_key())


def get_chat_model() -> ChatOpenAI:
    return ChatOpenAI(model=OPENAI_CHAT_MODEL, temperature=0, api_key=require_openai_key())
```

**✅ Validación del paso:**

```bash
python -m py_compile rag_pipeline.py
```

**📌 Resultado esperado:**  
El archivo compila correctamente.

---

### ✅ Paso 2. Implementa `load_pdfs()`

**📝 Descripción del paso:**  
Vas a extraer páginas y agregar metadata para trazabilidad.

**⚙️ Contenido del paso:**  
Agrega al final de `rag_pipeline.py`:

```python
def load_pdfs(directory: str = PDF_DIRECTORY) -> list[Document]:
    pdf_dir = Path(directory)
    if not pdf_dir.exists():
        raise FileNotFoundError(f"No existe el directorio: {directory}")

    pdf_files = sorted(pdf_dir.glob("*.pdf"))
    if not pdf_files:
        raise ValueError(f"No se encontraron PDFs en: {directory}")

    logger.info("Encontrados %d PDFs en %s", len(pdf_files), directory)
    documents: list[Document] = []
    seen_hashes: set[str] = set()
    processing_date = datetime.now().isoformat(timespec="seconds")

    for pdf_path in pdf_files:
        logger.info("Cargando PDF: %s", pdf_path.name)
        loader = PyPDFLoader(str(pdf_path))
        pages = loader.load()

        for doc in pages:
            clean_content = doc.page_content.strip()
            if len(clean_content) < 50:
                continue

            content_hash = compute_md5(clean_content)
            if content_hash in seen_hashes:
                logger.info("Página duplicada omitida: %s", pdf_path.name)
                continue
            seen_hashes.add(content_hash)

            doc.metadata.update({
                "source": pdf_path.name,
                "page": int(doc.metadata.get("page", 0)) + 1,
                "processing_date": processing_date,
                "content_hash": content_hash,
                "file_path": str(pdf_path.resolve()),
            })
            documents.append(doc)

    logger.info("Carga completada: %d páginas útiles", len(documents))
    return documents
```

**✅ Validación del paso:**

```bash
python -c "
from rag_pipeline import load_pdfs

docs = load_pdfs()
print('Documentos cargados:', len(docs))
print('Fuente:', docs[0].metadata['source'])
print('Página:', docs[0].metadata['page'])
print('Hash:', docs[0].metadata['content_hash'])
print('Preview:', docs[0].page_content[:120])
"
```

**📌 Resultado esperado:**  
Debes ver documentos cargados con metadata.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20de%20un%20laboratorio%20RAG.%20Cre%C3%A9%20el%20m%C3%B3dulo%20principal%20del%20pipeline%2C%20cargu%C3%A9%20variables%20de%20entorno%2C%20configur%C3%A9%20modelos%20de%20OpenAI%20y%20cargu%C3%A9%20PDFs%20con%20metadata.)

---

# 🧩 Tarea 4. Implementar chunking fijo como baseline

## 🎯 Objetivo de la tarea

Crear una estrategia base de chunking por tamaño de caracteres sin consumir API.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega `chunk_fixed_size()`

**📝 Descripción del paso:**  
Vas a crear chunks de tamaño fijo con overlap.

**⚙️ Contenido del paso:**
Agrega al final de `rag_pipeline.py`:

```python
def chunk_fixed_size(documents: list[Document], chunk_size: int = 500, chunk_overlap: int = 80) -> list[Document]:
    logger.info("Fixed-size Chunking: documentos=%d", len(documents))
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=chunk_overlap,
        length_function=len,
        separators=["\n\n", "\n", ". ", " ", ""],
    )
    chunks = splitter.split_documents(documents)
    for index, chunk in enumerate(chunks):
        chunk.metadata["chunk_strategy"] = "fixed_size"
        chunk.metadata["chunk_index"] = index
        chunk.metadata["chunk_size_chars"] = len(chunk.page_content)
    logger.info("Fixed-size Chunking generó %d chunks", len(chunks))
    return chunks
```

**✅ Validación del paso:**

```bash
python -m py_compile rag_pipeline.py
```

**📌 Resultado esperado:**  
El archivo compila correctamente.

---

### ✅ Paso 2. Prueba chunking fijo

**📝 Descripción del paso:**  
Vas a dividir los documentos sin llamar a la API.

**⚙️ Contenido del paso:**

```bash
python -c "
from rag_pipeline import load_pdfs, chunk_fixed_size

docs = load_pdfs()
chunks = chunk_fixed_size(docs)
print('Páginas:', len(docs))
print('Chunks:', len(chunks))
print('Estrategia:', chunks[0].metadata['chunk_strategy'])
print('Preview:', chunks[0].page_content[:200])
"
```

**✅ Validación del paso:**  
El número de chunks debe ser mayor que el número de páginas.

**📌 Resultado esperado:**  
Tienes chunks `fixed_size` listos como baseline.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20de%20un%20laboratorio%20RAG.%20Implement%C3%A9%20Fixed-size%20Chunking%20con%20RecursiveCharacterTextSplitter%20como%20baseline%20sin%20consumir%20API.)

---

# 🧩 Tarea 5. Implementar Semantic Chunking

## 🎯 Objetivo de la tarea

Crear una estrategia de chunking basada en cambios semánticos usando embeddings.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega `chunk_semantic()`

**📝 Descripción del paso:**  
Vas a usar `SemanticChunker` para detectar cortes conceptuales.

**⚙️ Contenido del paso:**
Agrega al final de `rag_pipeline.py`:

```python
def chunk_semantic(
    documents: list[Document],
    breakpoint_threshold_type: str = "percentile",
    breakpoint_threshold_amount: float = 85.0,
) -> list[Document]:
    logger.info("Semantic Chunking: documentos=%d", len(documents))
    logger.info("Esta tarea consume API de embeddings.")

    splitter = SemanticChunker(
        embeddings=get_embeddings(),
        breakpoint_threshold_type=breakpoint_threshold_type,
        breakpoint_threshold_amount=breakpoint_threshold_amount,
    )
    chunks = splitter.split_documents(documents)

    for index, chunk in enumerate(chunks):
        chunk.metadata["chunk_strategy"] = "semantic"
        chunk.metadata["chunk_index"] = index
        chunk.metadata["chunk_size_chars"] = len(chunk.page_content)

    sizes = [len(chunk.page_content) for chunk in chunks]
    if sizes:
        logger.info("Semantic chunks=%d | min=%d | max=%d | promedio=%d", len(chunks), min(sizes), max(sizes), sum(sizes)//len(sizes))
    return chunks
```

**✅ Validación del paso:**

```bash
python -m py_compile rag_pipeline.py
```

**📌 Resultado esperado:**  
El archivo compila correctamente.

---

### ✅ Paso 2. Ejecuta Semantic Chunking

**📝 Descripción del paso:**  
Vas a generar chunks semánticos. Esta operación consume embeddings.

**⚙️ Contenido del paso:**

```bash
python -c "
from rag_pipeline import load_pdfs, chunk_semantic

docs = load_pdfs()
chunks = chunk_semantic(docs)
print('Chunks semánticos:', len(chunks))
print('Estrategia:', chunks[0].metadata['chunk_strategy'])
print('Preview:', chunks[0].page_content[:250])
"
```

**✅ Validación del paso:**  
Debe generarse al menos un chunk semántico por documento o tema.

**📌 Resultado esperado:**  
Tienes chunks `semantic` listos para indexar.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20de%20un%20laboratorio%20RAG.%20Implement%C3%A9%20Semantic%20Chunking%20con%20SemanticChunker%20y%20embeddings%20para%20detectar%20cortes%20conceptuales.)

---

# 🧩 Tarea 6. Persistir embeddings en ChromaDB

## 🎯 Objetivo de la tarea

Crear dos colecciones vectoriales persistentes: una para chunks fijos y otra para chunks semánticos.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega utilidades de persistencia

**📝 Descripción del paso:**  
Vas a limpiar carpetas, contar vectores y crear colecciones.

**⚙️ Contenido del paso:**
Agrega al final de `rag_pipeline.py`:

```python
def reset_directory(path: str) -> None:
    target = Path(path)
    if target.exists():
        shutil.rmtree(target)
    target.mkdir(parents=True, exist_ok=True)


def safe_vector_count(vectorstore: Chroma) -> int:
    try:
        return vectorstore._collection.count()
    except Exception:
        return -1


def build_vectorstore(
    chunks: list[Document],
    persist_directory: str,
    collection_name: str,
    force_rebuild: bool = False,
) -> Chroma:
    persist_path = Path(persist_directory)

    if force_rebuild:
        reset_directory(persist_directory)

    if persist_path.exists() and any(persist_path.iterdir()) and not force_rebuild:
        vectorstore = Chroma(
            collection_name=collection_name,
            embedding_function=get_embeddings(),
            persist_directory=persist_directory,
        )
        logger.info("Colección cargada: %s | vectores=%d", collection_name, safe_vector_count(vectorstore))
        return vectorstore

    vectorstore = Chroma.from_documents(
        documents=chunks,
        embedding=get_embeddings(),
        collection_name=collection_name,
        persist_directory=persist_directory,
    )
    logger.info("Colección creada: %s | vectores=%d", collection_name, safe_vector_count(vectorstore))
    return vectorstore
```

**✅ Validación del paso:**

```bash
python -m py_compile rag_pipeline.py
```

**📌 Resultado esperado:**  
El archivo compila correctamente.

---

### ✅ Paso 2. Agrega `run_ingestion_pipeline()`

**📝 Descripción del paso:**  
Vas a unir carga, chunking e indexación.

**⚙️ Contenido del paso:**
Agrega al final de `rag_pipeline.py`:

```python
def run_ingestion_pipeline(force_rebuild: bool = False) -> dict[str, Chroma]:
    logger.info("=" * 70)
    logger.info("INICIANDO PIPELINE DE INGESTA RAG")
    logger.info("=" * 70)

    documents = load_pdfs()
    fixed_chunks = chunk_fixed_size(documents)
    semantic_chunks = chunk_semantic(documents)

    fixed_store = build_vectorstore(
        chunks=fixed_chunks,
        persist_directory=CHROMA_DIR_FIXED,
        collection_name=COLLECTION_FIXED,
        force_rebuild=force_rebuild,
    )
    semantic_store = build_vectorstore(
        chunks=semantic_chunks,
        persist_directory=CHROMA_DIR_SEMANTIC,
        collection_name=COLLECTION_SEMANTIC,
        force_rebuild=force_rebuild,
    )

    logger.info("INGESTA COMPLETADA | fixed=%d | semantic=%d", safe_vector_count(fixed_store), safe_vector_count(semantic_store))
    return {"fixed": fixed_store, "semantic": semantic_store}
```

**✅ Validación del paso:**

```bash
python -c "from rag_pipeline import run_ingestion_pipeline; print('Pipeline disponible')"
```

**📌 Resultado esperado:**

```text
Pipeline disponible
```

---

### ✅ Paso 3. Ejecuta la ingesta completa

**📝 Descripción del paso:**  
Vas a crear las dos bases vectoriales.

**⚙️ Contenido del paso:**

```bash
python -c "from rag_pipeline import run_ingestion_pipeline; run_ingestion_pipeline(force_rebuild=True)"
```

**Nota:** Si aparecen errores **`Failed to send telemetry event `** puedes ignorarlos. La documentación oficial indica que Chroma ya no recopila telemetría de producto desde la versión 1.5.4, pero tu entorno usa una versión anterior de Chroma por el rango de dependencias del laboratorio.

**✅ Validación del paso:**

```bash
ls -la chroma_db_fixed
ls -la chroma_db_semantic
```

**📌 Resultado esperado:**  
Ambas carpetas contienen archivos de ChromaDB.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20de%20un%20laboratorio%20RAG.%20Gener%C3%A9%20embeddings%20con%20OpenAI%20y%20persist%C3%AD%20dos%20colecciones%20en%20ChromaDB%20para%20comparar%20Fixed-size%20Chunking%20y%20Semantic%20Chunking.)

---

# 🧩 Tarea 7. Implementar recuperación de contexto con MMR

## 🎯 Objetivo de la tarea

Crear una función para recuperar contexto relevante y diverso desde ChromaDB.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega `retrieve_context()`

**📝 Descripción del paso:**  
Vas a recuperar chunks por similitud o MMR.

**⚙️ Contenido del paso:**
Agrega al final de `rag_pipeline.py`:

```python
def retrieve_context(
    query: str,
    vectorstore: Chroma,
    k: int = 4,
    use_mmr: bool = True,
    fetch_k: int = 12,
    lambda_mult: float = 0.5,
) -> list[Document]:
    if use_mmr:
        docs = vectorstore.max_marginal_relevance_search(
            query=query,
            k=k,
            fetch_k=fetch_k,
            lambda_mult=lambda_mult,
        )
    else:
        docs = vectorstore.similarity_search(query=query, k=k)
    logger.info("Recuperados %d chunks para: %s", len(docs), query[:80])
    return docs
```

**✅ Validación del paso:**

```bash
python -m py_compile rag_pipeline.py
```

**📌 Resultado esperado:**  
No debe aparecer ningún error.

---

### ✅ Paso 2. Prueba recuperación

**📝 Descripción del paso:**  
Vas a recuperar chunks relevantes para una pregunta.

**⚙️ Contenido del paso:**

```bash
python -c "
from rag_pipeline import run_ingestion_pipeline, retrieve_context
stores = run_ingestion_pipeline(force_rebuild=False)
docs = retrieve_context('¿Qué es RAG y para qué sirve?', stores['semantic'], k=3)
for i, doc in enumerate(docs, start=1):
    print(f'--- Chunk {i} ---')
    print(doc.metadata.get('source'), 'p.', doc.metadata.get('page'))
    print(doc.page_content[:250])
"
```

**✅ Validación del paso:**  
Debes recuperar 3 chunks relacionados con RAG.

**📌 Resultado esperado:**  
Los chunks muestran fuente, página y contenido relevante.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%207%20de%20un%20laboratorio%20RAG.%20Implement%C3%A9%20recuperaci%C3%B3n%20de%20contexto%20con%20similitud%20sem%C3%A1ntica%20y%20MMR.)

---

# 🧩 Tarea 8. Implementar generación de respuestas RAG

## 🎯 Objetivo de la tarea

Crear una función que recupere contexto y genere una respuesta usando únicamente información documental.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega el prompt y `answer_question()`

**📝 Descripción del paso:**  
Vas a crear una cadena RAG usando `create_retrieval_chain`.

**⚙️ Contenido del paso:**
Agrega al final de `rag_pipeline.py`:

```python
RAG_SYSTEM_PROMPT = """
Eres un asistente técnico especializado en responder preguntas usando documentos internos.

Reglas:
1. Responde únicamente con información presente en el contexto.
2. Si el contexto no contiene la respuesta, di: "No encontré información sobre esto en los documentos disponibles."
3. Cita fuente y página cuando sea posible.
4. Sé claro, breve y técnico.

Contexto:
{context}
"""

RAG_PROMPT = ChatPromptTemplate.from_messages([
    ("system", RAG_SYSTEM_PROMPT),
    ("human", "Pregunta: {input}"),
])


def answer_question(query: str, vectorstore: Chroma, k: int = 4, use_mmr: bool = True) -> dict[str, Any]:
    if use_mmr:
        retriever = vectorstore.as_retriever(
            search_type="mmr",
            search_kwargs={"k": k, "fetch_k": k * 3, "lambda_mult": 0.5},
        )
    else:
        retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": k})

    question_answer_chain = create_stuff_documents_chain(llm=get_chat_model(), prompt=RAG_PROMPT)
    rag_chain = create_retrieval_chain(retriever=retriever, combine_docs_chain=question_answer_chain)
    result = rag_chain.invoke({"input": query})

    return {
        "query": query,
        "answer": result.get("answer", "Sin respuesta"),
        "sources": result.get("context", []),
        "chunks_used": len(result.get("context", [])),
    }
```

**✅ Validación del paso:**

```bash
python -m py_compile rag_pipeline.py
```

**📌 Resultado esperado:**  
No debe aparecer ningún error.

---

### ✅ Paso 2. Crea `01_consultar_rag.py`

**📝 Descripción del paso:**  
Vas a probar el asistente documental con preguntas básicas.

**⚙️ Contenido del paso:**

```python
from rag_pipeline import answer_question, run_ingestion_pipeline

PREGUNTAS = [
    "¿Qué es RAG y cuál es su propósito principal?",
    "¿Qué son los embeddings y para qué se usan?",
    "¿Qué es MMR y cómo ayuda en recuperación?",
]


def imprimir_resultado(resultado: dict) -> None:
    print("\n" + "=" * 80)
    print("Pregunta:", resultado["query"])
    print("-" * 80)
    print("Respuesta:")
    print(resultado["answer"])
    print("\nFuentes utilizadas:")
    for doc in resultado["sources"]:
        print(f"- {doc.metadata.get('source', 'N/A')} p.{doc.metadata.get('page', '?')} [{doc.metadata.get('chunk_strategy', 'N/A')}]")


if __name__ == "__main__":
    stores = run_ingestion_pipeline(force_rebuild=False)
    for pregunta in PREGUNTAS:
        resultado = answer_question(pregunta, stores["semantic"], k=4, use_mmr=True)
        imprimir_resultado(resultado)
```

**✅ Validación del paso:**

```bash
python 01_consultar_rag.py
```

**📌 Resultado esperado:**  
Debes ver respuestas con fuentes documentales.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%208%20de%20un%20laboratorio%20RAG.%20Implement%C3%A9%20generaci%C3%B3n%20de%20respuestas%20con%20create_retrieval_chain%20y%20un%20prompt%20que%20obliga%20a%20responder%20solo%20con%20contexto.)

---

# 🧩 Tarea 9. Crear evaluación comparativa entre estrategias

## 🎯 Objetivo de la tarea

Comparar respuestas generadas con Fixed-size Chunking y Semantic Chunking usando las mismas preguntas.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `02_evaluar_chunking.py`

**📝 Descripción del paso:**  
Vas a ejecutar las mismas preguntas contra ambas colecciones y guardar resultados.

**⚙️ Contenido del paso:**

```python
import json
from datetime import datetime
from pathlib import Path

import pandas as pd

from rag_pipeline import answer_question, run_ingestion_pipeline

QUESTIONS = [
    "¿Qué es RAG y cuál es su propósito principal?",
    "¿Cuáles son las fases principales de un sistema RAG?",
    "¿Por qué la estrategia de chunking afecta la calidad de recuperación?",
    "¿Qué son los embeddings y para qué se usan?",
    "¿Cómo ayuda la similitud coseno en búsqueda semántica?",
    "¿Qué es ChromaDB y por qué es útil en prototipos RAG?",
    "¿Qué es MMR y qué problema resuelve?",
    "¿Qué buenas prácticas se recomiendan para APIs REST?",
    "¿Qué métodos de autenticación se mencionan para APIs?",
    "¿Cómo se evalúa la calidad de un sistema RAG?",
]


def sources_to_text(sources: list) -> str:
    items = [f"{doc.metadata.get('source', 'N/A')} p.{doc.metadata.get('page', '?')}" for doc in sources]
    return "; ".join(sorted(set(items)))


def evaluate() -> list[dict]:
    stores = run_ingestion_pipeline(force_rebuild=False)
    rows = []
    for index, question in enumerate(QUESTIONS, start=1):
        print(f"Evaluando pregunta {index}/{len(QUESTIONS)}: {question}")
        for strategy in ["semantic", "fixed"]:
            result = answer_question(question, stores[strategy], k=4, use_mmr=True)
            rows.append({
                "question_id": index,
                "question": question,
                "strategy": strategy,
                "answer": result["answer"],
                "answer_length": len(result["answer"]),
                "chunks_used": result["chunks_used"],
                "sources": sources_to_text(result["sources"]),
                "no_info_response": "No encontré información" in result["answer"],
            })
    return rows


def summarize(rows: list[dict]) -> pd.DataFrame:
    df = pd.DataFrame(rows)
    return df.groupby("strategy").agg(
        total_questions=("question_id", "count"),
        avg_answer_length=("answer_length", "mean"),
        avg_chunks_used=("chunks_used", "mean"),
        no_info_count=("no_info_response", "sum"),
    ).reset_index()


if __name__ == "__main__":
    Path("reports").mkdir(exist_ok=True)
    rows = evaluate()
    summary = summarize(rows)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    json_path = Path("reports") / f"rag_chunking_evaluation_{timestamp}.json"
    csv_path = Path("reports") / f"rag_chunking_evaluation_{timestamp}.csv"
    summary_path = Path("reports") / f"rag_chunking_summary_{timestamp}.csv"

    with open(json_path, "w", encoding="utf-8") as file:
        json.dump(rows, file, ensure_ascii=False, indent=2)

    pd.DataFrame(rows).to_csv(csv_path, index=False, encoding="utf-8-sig")
    summary.to_csv(summary_path, index=False, encoding="utf-8-sig")

    print("\nResumen comparativo:")
    print(summary.to_markdown(index=False))
    print("\nArchivos generados:")
    print(json_path)
    print(csv_path)
    print(summary_path)
```

**✅ Validación del paso:**

```bash
python -m py_compile 02_evaluar_chunking.py
```

**📌 Resultado esperado:**  
El archivo compila correctamente.

---

### ✅ Paso 2. Ejecuta evaluación

**📝 Descripción del paso:**  
Vas a generar respuestas comparables.

**⚙️ Contenido del paso:**

```bash
python 02_evaluar_chunking.py
ls -lh reports/
```

**✅ Validación del paso:**  
Debes ver archivos JSON y CSV.

**📌 Resultado esperado:**  
La evaluación comparativa fue generada.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 9 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%209%20de%20un%20laboratorio%20RAG.%20Compar%C3%A9%20Fixed-size%20Chunking%20contra%20Semantic%20Chunking%20con%20las%20mismas%20preguntas%20y%20gener%C3%A9%20JSON%20y%20CSV%20de%20resultados.)

---

# 🧩 Tarea 10. Evaluar calidad manual y documentar hallazgos

## 🎯 Objetivo de la tarea

Convertir la comparación técnica en una evaluación profesional que considere relevancia, fidelidad, fuentes y utilidad.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la matriz manual

**📝 Descripción del paso:**  
Vas a crear una tabla para evaluar calidad humana.

**⚙️ Contenido del paso:**

```bash
cat > reports/matriz_evaluacion_manual.md << 'MD'
# Matriz de evaluación manual RAG

| Pregunta | Estrategia ganadora | Relevancia del contexto (1-5) | Fidelidad al contexto (1-5) | Citas/fuentes correctas (1-5) | Claridad de respuesta (1-5) | Observaciones |
|---|---|---:|---:|---:|---:|---|
| ¿Qué es RAG y cuál es su propósito principal? |  |  |  |  |  |  |
| ¿Cuáles son las fases principales de un sistema RAG? |  |  |  |  |  |  |
| ¿Por qué la estrategia de chunking afecta la calidad de recuperación? |  |  |  |  |  |  |
| ¿Qué son los embeddings y para qué se usan? |  |  |  |  |  |  |
| ¿Cómo ayuda la similitud coseno en búsqueda semántica? |  |  |  |  |  |  |
| ¿Qué es ChromaDB y por qué es útil en prototipos RAG? |  |  |  |  |  |  |
| ¿Qué es MMR y qué problema resuelve? |  |  |  |  |  |  |
| ¿Qué buenas prácticas se recomiendan para APIs REST? |  |  |  |  |  |  |
| ¿Qué métodos de autenticación se mencionan para APIs? |  |  |  |  |  |  |
| ¿Cómo se evalúa la calidad de un sistema RAG? |  |  |  |  |  |  |

## Conclusiones

1. La estrategia que funcionó mejor en la mayoría de preguntas fue: ____.
2. Semantic Chunking aportó más valor cuando: ____.
3. Fixed-size Chunking fue suficiente cuando: ____.
4. El principal problema observado fue: ____.
5. Para producción recomendaría: ____.
MD
```

**✅ Validación del paso:**

```bash
cat reports/matriz_evaluacion_manual.md
```

**📌 Resultado esperado:**  
La matriz está lista para completar.

---

### ✅ Paso 2. Evalúa al menos 5 preguntas

**📝 Descripción del paso:**  
Vas a abrir el CSV/JSON y comparar semantic vs fixed.

**⚙️ Contenido del paso:**  
Para cada pregunta revisa:

1. ¿La respuesta responde la pregunta?
2. ¿Está soportada por los documentos?
3. ¿Las fuentes corresponden al tema?
4. ¿La respuesta inventa información?
5. ¿La estrategia recuperó contexto suficiente?

**✅ Validación del paso:**  
Completa al menos 5 filas de la matriz.

**📌 Resultado esperado:**  
Tienes una evaluación cualitativa, no solo automática.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 10 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2010%20de%20un%20laboratorio%20RAG.%20Cre%C3%A9%20una%20matriz%20manual%20para%20evaluar%20calidad%20de%20respuestas%20y%20redact%C3%A9%20conclusiones%20t%C3%A9cnicas.)

---

# 🧩 Tarea 11. Validar funcionamiento y entregar evidencias

## 🎯 Objetivo de la tarea

Confirmar que los scripts funcionan, que las colecciones existen y que la entrega no expone secretos.

---

## 🛠️ Pasos

### ✅ Paso 1. Valida compilación

**📝 Descripción del paso:**  
Vas a revisar errores de sintaxis.

**⚙️ Contenido del paso:**

```bash
python -m py_compile 00_crear_pdfs_prueba.py
python -m py_compile rag_pipeline.py
python -m py_compile 01_consultar_rag.py
python -m py_compile 02_evaluar_chunking.py
```

**✅ Validación del paso:**  
Ningún comando debe mostrar errores.

**📌 Resultado esperado:**  
Todos los scripts son válidos.

---

### ✅ Paso 2. Valida PDFs, colecciones y reportes

**📝 Descripción del paso:**  
Vas a confirmar que existen artefactos del laboratorio.

**⚙️ Contenido del paso:**

```bash
ls -lh pdfs/
ls -lh chroma_db_fixed/
ls -lh chroma_db_semantic/
ls -lh reports/
```

**✅ Validación del paso:**  
Las carpetas deben contener archivos.

**📌 Resultado esperado:**  
Los documentos, bases vectoriales y reportes están disponibles.

---

### ✅ Paso 3. Prepara entrega segura

**📝 Descripción del paso:**  
Vas a identificar archivos entregables.

**⚙️ Contenido del paso:**

Entrega:

```text
00_crear_pdfs_prueba.py
rag_pipeline.py
01_consultar_rag.py
02_evaluar_chunking.py
requirements.txt
reports/*.json
reports/*.csv
reports/matriz_evaluacion_manual.md
```

No entregues:

```text
.env
.venv/
```

**✅ Validación del paso:**  
Confirma que `.env` no está incluido.

**📌 Resultado esperado:**  
Tienes una entrega completa y segura.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 11 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2011%20de%20un%20laboratorio%20RAG.%20Valid%C3%A9%20scripts%2C%20PDFs%2C%20colecciones%20ChromaDB%2C%20reportes%20y%20prepar%C3%A9%20una%20entrega%20sin%20secretos.)

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Variables de entorno configuradas.
4. PDFs técnicos sintéticos generados localmente.
5. Carga de PDFs con metadata enriquecida.
6. Chunking fijo funcionando sin API.
7. Semantic Chunking funcionando con embeddings.
8. Dos colecciones ChromaDB persistentes.
9. Recuperación de contexto con similitud y MMR.
10. Asistente RAG funcional.
11. Script de evaluación comparativa.
12. Reportes JSON y CSV.
13. Matriz manual de evaluación.
14. Conclusiones técnicas sobre qué estrategia funcionó mejor.
15. Evidencias listas para entregar sin exponer `.env`.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del ambiente local | 10% |
| Generación/carga correcta de documentos PDF | 10% |
| Metadata y deduplicación por hash | 10% |
| Implementación de Fixed-size Chunking | 10% |
| Implementación de Semantic Chunking | 15% |
| Persistencia en ChromaDB | 15% |
| Recuperación con MMR | 10% |
| Generación de respuestas RAG con fuentes | 10% |
| Evaluación comparativa y matriz manual | 10% |
| Total | 100% |

---

# ⚠️ Errores comunes que debes evitar

1. No activar el entorno virtual antes de instalar dependencias.
2. Guardar la API key directamente en el código.
3. Subir `.env` a un repositorio.
4. Ejecutar Semantic Chunking repetidamente sin necesidad.
5. Confundir chunking con embeddings.
6. Pensar que más chunks siempre significa mejor recuperación.
7. Pensar que respuestas más largas siempre son mejores.
8. Evaluar RAG solo con métricas automáticas.
9. No revisar las fuentes recuperadas.
10. Olvidar reconstruir ChromaDB cuando cambian documentos o parámetros.
11. Usar documentos sensibles en pruebas.
12. Ignorar costos de embeddings y generación.

---

# 🧯 Solución de problemas

## ❌ Error: `OPENAI_API_KEY no está configurada`

### Causa probable

El archivo `.env` no existe, está mal escrito o todavía contiene `REEMPLAZA_CON_TU_API_KEY`.

### Solución

```env
OPENAI_API_KEY=sk-...
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_CHAT_MODEL=gpt-4o-mini
```

Valida:

```bash
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print(bool(os.getenv('OPENAI_API_KEY')))"
```

---

## ❌ Error: `No se encontraron PDFs`

### Causa probable

No ejecutaste el generador de PDFs o estás en otra carpeta.

### Solución

```bash
pwd
python 00_crear_pdfs_prueba.py
ls -lh pdfs/
```

---

## ❌ Semantic Chunking genera muy pocos chunks

### Causa probable

El percentil es demasiado alto y solo corta cuando detecta cambios semánticos muy fuertes.

### Solución

En `chunk_semantic()`, baja el valor:

```python
breakpoint_threshold_amount: float = 80.0
```

Luego reconstruye:

```bash
python -c "from rag_pipeline import run_ingestion_pipeline; run_ingestion_pipeline(force_rebuild=True)"
```

---

## ❌ ChromaDB carga datos viejos

### Causa probable

Las carpetas persistentes ya existen y el pipeline no reconstruye.

### Solución

```bash
rm -rf chroma_db_fixed chroma_db_semantic
python -c "from rag_pipeline import run_ingestion_pipeline; run_ingestion_pipeline(force_rebuild=True)"
```

---

# 🧹 Limpieza del entorno

## Opción A. Conservar datos para el siguiente laboratorio

```bash
deactivate
```

## Opción B. Eliminar bases vectoriales y reportes

```bash
rm -rf chroma_db_fixed chroma_db_semantic reports
```

## Opción C. Limpieza completa

```bash
deactivate
rm -rf .venv chroma_db_fixed chroma_db_semantic reports pdfs
```

> [!WARNING]
> Si planeas continuar con otro laboratorio RAG, conserva `pdfs/`, `chroma_db_fixed/` y `chroma_db_semantic/` para no recalcular embeddings.

---

# 🧠 Cierre de la práctica

En este laboratorio construiste un pipeline RAG completo y profesional. Preparaste un entorno local en Windows, generaste PDFs técnicos de prueba, cargaste documentos con metadata enriquecida, comparaste Fixed-size Chunking contra Semantic Chunking, generaste embeddings, persististe vectores en ChromaDB, recuperaste contexto con MMR y generaste respuestas basadas exclusivamente en documentos.

Lo más importante no fue solo ejecutar RAG, sino entender que la calidad de un asistente documental depende de decisiones previas a la generación: limpieza de documentos, chunking, metadata, embeddings, estrategia de recuperación y evaluación humana.

Esta práctica te prepara para diseñar pipelines RAG más sólidos, medibles y defendibles en escenarios empresariales.
