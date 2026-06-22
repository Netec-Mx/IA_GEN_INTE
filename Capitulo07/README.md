
<div align="center">

# 🧪 Laboratorio 7

## Preparación profesional de dataset para fine-tuning: limpieza, validación, seguridad y decisión técnica

![Nivel](https://img.shields.io/badge/Nivel-Intermedio-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)
![Costo](https://img.shields.io/badge/Costo-0%20USD-16A34A?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio **no ejecutarás un fine-tuning real**. Vas a preparar, limpiar, validar y documentar un dataset de preguntas y respuestas para entender cuándo conviene usar **fine-tuning**, **RAG** o **few-shot prompting**. No uses datos reales de clientes, credenciales, tickets internos ni información sensible.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Preparación de datos para fine-tuning</td>
<td width="25%"><strong>⏱️ Duración</strong><br>50 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar, evaluar y crear</td>
<td width="25%"><strong>📦 Entregable</strong><br>JSONL + reportes</td>
</tr>
</table>

---

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a construir un pipeline completo de preprocesamiento de datos para fine-tuning. Partirás de un dataset crudo de preguntas y respuestas de soporte técnico con errores intencionales, lo limpiarás, validarás su estructura, detectarás posibles datos sensibles, lo convertirás al formato JSONL compatible con fine-tuning tipo chat y generarás reportes técnicos de calidad.

A diferencia de una conversión simple de CSV a JSONL, aquí trabajarás como en un flujo profesional: inspeccionarás calidad, corregirás problemas, validarás tokens y estructura, revisarás seguridad, generarás evidencias y tomarás una decisión informada entre **fine-tuning**, **RAG** y **few-shot prompting**.

La práctica original proponía transformar un dataset crudo de preguntas y respuestas en `train.jsonl`, `validation.jsonl` y `preprocessing_report.md`, aplicando limpieza, deduplicación, conteo de tokens y análisis de costo. Esta versión conserva esa base y agrega validación estricta de orden de roles, detección de datos sensibles, matriz de decisión y revisión manual.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Preparar un proyecto Python local en Windows usando VSCode y Git Bash.
2. Generar un dataset crudo sintético con errores representativos de un caso real.
3. Inspeccionar nulos, duplicados, respuestas cortas y desbalance de categorías.
4. Normalizar texto y eliminar caracteres inválidos antes de entrenar un modelo.
5. Deduplicar preguntas exactas y casi duplicadas con similitud textual.
6. Detectar posibles datos sensibles antes de exportar ejemplos para entrenamiento.
7. Convertir preguntas y respuestas al formato JSONL de mensajes tipo chat.
8. Validar estructura, orden de roles, contenido, tokens y distribución de categorías.
9. Dividir el dataset en entrenamiento y validación.
10. Generar reportes profesionales de calidad y decisión técnica.
11. Elegir entre fine-tuning, RAG y few-shot prompting con criterios técnicos.
12. Entregar evidencias sin incluir datos sensibles ni archivos innecesarios.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Tener conocimientos básicos de Python.
2. Saber ejecutar comandos desde Git Bash.
3. Conocer el formato JSON y JSONL.
4. Entender el concepto general de fine-tuning.
5. Entender la diferencia conceptual entre fine-tuning, RAG y few-shot prompting.
6. Haber trabajado previamente con archivos CSV.
7. Tener Visual Studio Code instalado.
8. Tener Python 3.11 o superior instalado.
9. Tener acceso a internet para instalar dependencias desde PyPI.

> [!NOTE]
> No necesitas API key de OpenAI para este laboratorio porque no se ejecutará fine-tuning real ni llamadas a modelos.

---

### 💻 Hardware

| Recurso | Requisito mínimo | Recomendado |
|---|---:|---:|
| Equipo | Laptop o PC con Windows | Laptop o PC con Windows 11 |
| CPU | 2 núcleos | 4 núcleos o más |
| RAM | 8 GB | 16 GB |
| Almacenamiento libre | 500 MB | 1 GB |
| GPU | No requerida | No requerida |
| Internet | Requerido para instalar paquetes | 10 Mbps o superior |

---

### 🧰 Software

| Software | Uso |
|---|---|
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Python 3.11 o superior | Runtime del laboratorio |
| pip | Instalación de dependencias |
| pandas | Manipulación de datos tabulares |
| tiktoken | Estimación de tokens |
| jsonlines | Lectura y escritura JSONL |
| difflib | Deduplicación básica por similitud |

---

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 50 minutos |
| Complejidad | Intermedia |
| Nivel de Bloom | Aplicar, analizar, evaluar y crear |
| Modalidad | Individual o equipos de 2 personas |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Costo estimado | $0 USD |
| Dataset | Sintético, generado localmente |
| Entregable principal | `train.jsonl`, `validation.jsonl`, `preprocessing_report.md` |
| Entregable secundario | `decision_matrix.md`, `manual_review_checklist.md` |
| Tipo de práctica | Preparación de datos, validación y análisis técnico |

---

## 🛡️ Consideraciones para estudiantes

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>No uses datos reales ni sensibles.</td>
<td><strong>🧪 Alcance</strong><br>No se ejecuta fine-tuning real.</td>
<td><strong>📊 Calidad</strong><br>Un dataset limpio no garantiza un buen modelo.</td>
</tr>
</table>

1. No uses tickets reales de soporte.
2. No incluyas credenciales, tokens, correos reales, teléfonos o datos de clientes.
3. No subas archivos JSONL con datos sensibles a un repositorio.
4. El conteo de tokens es una estimación técnica y puede variar por modelo.
5. Los precios y modelos cambian; los valores usados aquí son de referencia didáctica.
6. Fine-tuning no reemplaza RAG. Sirve para especializar comportamiento, tono, formato o patrones de respuesta.
7. La deduplicación con `difflib` es didáctica; para datasets grandes considera embeddings, MinHash o LSH.
8. La validación automática no reemplaza revisión humana.
9. No entrenes con respuestas cortas, contradictorias, incompletas o mal etiquetadas.
10. Documenta cualquier cambio de precios, modelo, split o umbral de similitud.
11. Si en tu terminal aparecen mensajes como `Failed to send telemetry event ClientStartEvent` o `ClientCreateCollectionEvent`, trátalos como advertencias de telemetría de ChromaDB provenientes de librerías usadas en laboratorios anteriores o en el mismo entorno. En este laboratorio 7 no se usa ChromaDB, por lo que esos mensajes no indican que el pipeline de fine-tuning haya fallado.

---

## 🔗 Fuentes oficiales que debes revisar antes de llevar esto a producción

> [!NOTE]
> Esta práctica no ejecuta fine-tuning real. Si después decides entrenar un modelo, revisa la documentación vigente del proveedor antes de usar los archivos generados.

| Tema | Qué revisar |
|---|---|
| Fine-tuning | Modelos soportados, formato requerido, límites, costos y estado actual del servicio |
| Formato JSONL | Estructura exacta esperada para ejemplos tipo chat |
| Tokenización | Modelo de tokenización recomendado para el modelo elegido |
| Privacidad | Políticas de uso, retención, entrenamiento y manejo de datos |
| Costos | Precio de entrenamiento, inferencia, validación y almacenamiento |

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear la carpeta del laboratorio, abrirla en VSCode, configurar un entorno virtual Python y preparar los archivos base del proyecto.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear desde Git Bash la carpeta raíz del laboratorio `lab-07-finetuning-dataset`. En esa carpeta guardarás todos los archivos de trabajo de la práctica: scripts Python, dataset CSV, archivos JSONL y reportes Markdown. No debes crear estos archivos en otra ruta para evitar errores de lectura o escritura más adelante.

**⚙️ Contenido del paso:**

```bash
mkdir -p ~/labs-ia-gen/lab-07-finetuning-dataset
cd ~/labs-ia-gen/lab-07-finetuning-dataset
```

**✅ Validación del paso:**

```bash
pwd
```

**📌 Resultado esperado:**  
Debes estar en una ruta similar a:

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-07-finetuning-dataset
```

---

### ✅ Paso 2. Abre la carpeta en Visual Studio Code

**📝 Descripción del paso:**  
Vas a abrir en Visual Studio Code la carpeta `lab-07-finetuning-dataset` que acabas de crear. A partir de este punto, todos los archivos nuevos se deben crear dentro de esa carpeta y no en el escritorio, descargas u otra ubicación.

**⚙️ Contenido del paso:**

```bash
code .
```

Si el comando no funciona, abre VSCode manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-07-finetuning-dataset
```

**✅ Validación del paso:**  
Confirma que VSCode muestra la carpeta `lab-07-finetuning-dataset`.

**📌 Resultado esperado:**  
El proyecto está abierto en VSCode.

---

### ✅ Paso 3. Crea y activa el entorno virtual

**📝 Descripción del paso:**  
Vas a crear dentro del proyecto un entorno virtual llamado `.venv` y después lo vas a activar desde Git Bash. Esto hace que las librerías instaladas en la práctica queden aisladas de otros proyectos de Python.

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
La ruta de Python debe apuntar a `.venv`.

---

### ✅ Paso 4. Crea `requirements.txt`

**📝 Descripción del paso:**  
Vas a crear en la raíz del proyecto el archivo `requirements.txt`. Este archivo debe contener la lista de librerías que instalarás para manipular datos, contar tokens y escribir archivos JSONL.

**⚙️ Contenido del paso:**

```bash
cat > requirements.txt << 'EOF'
pandas>=2.2,<3
tiktoken>=0.7,<1
jsonlines>=4.0,<5
python-dotenv>=1.0,<2
EOF
```

**✅ Validación del paso:**

```bash
cat requirements.txt
```

**📌 Resultado esperado:**  
El archivo contiene las dependencias necesarias.

---

### ✅ Paso 5. Instala dependencias

**📝 Descripción del paso:**  
Vas a ejecutar la instalación desde Git Bash con el entorno virtual activo. El comando leerá `requirements.txt` e instalará las librerías necesarias para manipular el CSV, generar JSONL y estimar tokens.

**⚙️ Contenido del paso:**

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**✅ Validación del paso:**

```bash
python -c "import pandas, tiktoken, jsonlines; print('Dependencias instaladas correctamente')"
```

**📌 Resultado esperado:**

```text
Dependencias instaladas correctamente
```

---

### ✅ Paso 6. Crea `.gitignore`

**📝 Descripción del paso:**  
Vas a crear en la raíz del proyecto el archivo `.gitignore`. Este archivo indica qué elementos no deben subirse a un repositorio, como el entorno virtual, archivos temporales o posibles archivos sensibles.

**⚙️ Contenido del paso:**

```bash
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
*.pyo
*.key
*.log
*.tmp
# Si usas datos reales, considera excluir también:
# *.jsonl
# raw_qa_dataset.csv
EOF
```

**✅ Validación del paso:**

```bash
cat .gitignore
```

**📌 Resultado esperado:**  
`.venv/` y archivos temporales están excluidos.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20de%20un%20laboratorio%20de%20IA%20generativa.%20Prepar%C3%A9%20un%20proyecto%20local%20en%20Windows%20con%20VSCode%2C%20Git%20Bash%2C%20un%20entorno%20virtual%20Python%2C%20requirements.txt%20y%20.gitignore%20para%20trabajar%20un%20dataset%20de%20fine-tuning.)

---

# 🧩 Tarea 2. Generar un dataset crudo con errores intencionales

## 🎯 Objetivo de la tarea

Crear un archivo CSV sintético de preguntas y respuestas de soporte técnico con problemas realistas: duplicados, nulos, respuestas cortas, caracteres inválidos, posibles datos sensibles y categorías desbalanceadas.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `generate_raw_dataset.py`

**📝 Descripción del paso:**  
Vas a crear en Visual Studio Code el archivo `generate_raw_dataset.py` dentro de la carpeta del laboratorio. Este script generará localmente un dataset sintético y controlado, por lo que no necesitas descargar datos externos ni usar información real.

**⚙️ Contenido del paso:**

Crea el archivo:

```text
generate_raw_dataset.py
```

Agrega el siguiente código:

```python
"""
Genera un dataset crudo sintético de soporte técnico con errores intencionales.
"""

import csv
import random
import pandas as pd

random.seed(42)

CATEGORIES = {
    "instalacion": {
        "count": 120,
        "qa_pairs": [
            (
                "¿Cómo instalo el software en Windows 11?",
                "Para instalar el software en Windows 11, descarga el instalador desde la página oficial, ejecuta el archivo .exe como administrador y sigue el asistente de instalación. Antes de iniciar, verifica que tengas permisos suficientes y espacio disponible en disco. Si el instalador falla, reinicia el equipo y vuelve a ejecutar el proceso."
            ),
            (
                "El instalador indica que falta .NET Framework, ¿qué hago?",
                "Debes instalar .NET Framework 4.8 o superior desde el sitio oficial de Microsoft. Después de instalarlo, reinicia Windows y ejecuta nuevamente el instalador del software. Si el error persiste, revisa que Windows Update esté habilitado y que tu usuario tenga permisos de administrador."
            ),
            (
                "¿Puedo instalar el software en varias computadoras?",
                "Depende del tipo de licencia. Una licencia estándar permite la instalación en un número limitado de dispositivos. Para ambientes empresariales, se recomienda revisar el contrato de licenciamiento o contactar al área comercial para confirmar el número de activaciones permitidas."
            ),
        ],
    },
    "configuracion": {
        "count": 90,
        "qa_pairs": [
            (
                "¿Cómo cambio el idioma de la interfaz?",
                "Abre el menú Configuración, entra a la sección General y selecciona el idioma deseado. Guarda los cambios y reinicia la aplicación si el sistema lo solicita. Si el idioma no aparece en la lista, verifica que tu versión tenga instalado el paquete regional correspondiente."
            ),
            (
                "¿Cómo configuro el proxy corporativo?",
                "Ve a Configuración > Red > Proxy y selecciona configuración manual. Ingresa host, puerto y credenciales si tu organización las requiere. Después guarda los cambios y prueba la conexión. Si la autenticación falla, solicita al equipo de redes los datos correctos del proxy."
            ),
            (
                "¿Puedo personalizar los atajos de teclado?",
                "Sí. En Configuración > Accesibilidad > Atajos de teclado puedes modificar combinaciones de teclas para acciones frecuentes. Evita usar combinaciones reservadas por el sistema operativo y guarda una copia de tu configuración antes de hacer cambios masivos."
            ),
        ],
    },
    "errores": {
        "count": 70,
        "qa_pairs": [
            (
                "La aplicación muestra Access Denied al guardar archivos",
                "El error Access Denied suele indicar falta de permisos de escritura en la carpeta destino. Guarda el archivo en una ubicación donde tengas permisos, ejecuta la aplicación como administrador o solicita al equipo de TI que ajuste los permisos del directorio compartido."
            ),
            (
                "El programa se cierra al abrir archivos grandes",
                "Este comportamiento puede deberse a memoria insuficiente o archivos dañados. Cierra otras aplicaciones, verifica que el archivo no esté corrupto y aumenta la memoria disponible. Si trabajas con archivos grandes frecuentemente, usa la opción de procesamiento por lotes o la versión optimizada para alto volumen."
            ),
        ],
    },
    "licencias": {
        "count": 35,
        "qa_pairs": [
            (
                "¿Cómo activo mi licencia después de comprar?",
                "Después de la compra recibirás un correo con la clave de activación. Abre el software, entra a Ayuda > Activar licencia e ingresa la clave. Necesitas conexión a internet para validar la licencia. Si no tienes conexión, usa el flujo de activación manual."
            ),
            (
                "Mi licencia expiró, ¿puedo seguir usando el software?",
                "Cuando la licencia expira, el software puede quedar en modo limitado o de solo lectura. Para recuperar todas las funciones, renueva la suscripción desde el portal de clientes. Los datos existentes normalmente se conservan, pero algunas funciones avanzadas pueden quedar bloqueadas."
            ),
        ],
    },
    "rendimiento": {
        "count": 25,
        "qa_pairs": [
            (
                "El software va muy lento, ¿cómo puedo mejorar el rendimiento?",
                "Cierra proyectos que no estés usando, reduce el tamaño de caché, desactiva animaciones y verifica que tu equipo cumpla los requisitos mínimos. Si trabajas con archivos grandes, considera aumentar memoria RAM, usar disco SSD y dividir tareas pesadas en procesos más pequeños."
            ),
            (
                "¿Cómo reduzco el uso de memoria RAM?",
                "Puedes reducir el uso de memoria ajustando el tamaño de caché, cerrando módulos no utilizados y activando el modo de bajo consumo. También conviene revisar extensiones instaladas, procesos en segundo plano y configuraciones que carguen datos innecesarios al iniciar."
            ),
        ],
    },
}

EMPTY_OR_NULL = [
    ("¿Cómo desinstalo completamente el software?", "", "instalacion"),
    ("¿Existe versión para Linux?", None, "instalacion"),
    ("¿Cómo exporto mis datos?", "   ", "configuracion"),
]

SHORT_ANSWERS = [
    ("¿Qué navegador recomiendan?", "Chrome.", "configuracion"),
    ("¿Tienen soporte 24/7?", "No.", "licencias"),
    ("¿Funciona sin internet?", "Sí.", "instalacion"),
]

SPECIAL_CHARS = [
    (
        "¿Cómo uso la búsqueda avanzada\\x00?",
        "La búsqueda avanzada permite filtrar por fecha, tipo y etiqueta. También puedes combinar filtros usando operadores booleanos y guardar búsquedas frecuentes para reutilizarlas después.",
        "configuracion",
    ),
    (
        "Error código 0x80070005\\r\\n¿cómo lo soluciono?",
        "El código 0x80070005 normalmente indica permisos insuficientes. Ejecuta la aplicación como administrador, revisa los permisos de la carpeta y confirma que tu antivirus no esté bloqueando el proceso.",
        "errores",
    ),
]

SENSITIVE_EXAMPLES = [
    (
        "No puedo iniciar sesión con mi correo usuario.demo@example.com",
        "No compartas contraseñas ni tokens por correo. Para recuperar el acceso, usa el portal de autoservicio, valida tu correo corporativo y contacta soporte si el segundo factor no funciona.",
        "errores",
    ),
    (
        "Mi token sk-proj-EXAMPLE123 dejó de funcionar",
        "Por seguridad, nunca compartas tokens de API en tickets. Revoca el token expuesto, genera uno nuevo desde la consola del proveedor y actualiza la aplicación usando variables de entorno o un gestor de secretos.",
        "configuracion",
    ),
]

rows = []
for category, data in CATEGORIES.items():
    qa_pool = data["qa_pairs"]
    for i in range(data["count"]):
        question, answer = qa_pool[i % len(qa_pool)]
        variation = "" if i < len(qa_pool) else f" Caso {i + 1}."
        rows.append({
            "id": len(rows) + 1,
            "category": category,
            "question": question + variation,
            "answer": answer,
        })

for row in random.sample(rows[:80], 20):
    rows.append({
        "id": len(rows) + 1,
        "category": row["category"],
        "question": row["question"],
        "answer": row["answer"],
    })

near_duplicates = [
    (
        "¿Cómo puedo instalar el software en Windows 11?",
        "Para instalar el software en Windows 11, descarga el instalador desde la página oficial, ejecuta el archivo .exe como administrador y sigue el asistente de instalación. Antes de iniciar, verifica permisos y espacio en disco.",
        "instalacion",
    ),
    (
        "La aplicación marca Access Denied cuando intento guardar",
        "El error Access Denied suele indicar falta de permisos de escritura. Guarda en una ubicación permitida, ejecuta la aplicación como administrador o solicita al equipo de TI ajustar permisos.",
        "errores",
    ),
]

for question, answer, category in EMPTY_OR_NULL + SHORT_ANSWERS + SPECIAL_CHARS + SENSITIVE_EXAMPLES + near_duplicates:
    rows.append({
        "id": len(rows) + 1,
        "category": category,
        "question": question,
        "answer": answer,
    })

random.shuffle(rows)
for index, row in enumerate(rows, start=1):
    row["id"] = index

df = pd.DataFrame(rows)
df.to_csv("raw_qa_dataset.csv", index=False, encoding="utf-8", quoting=csv.QUOTE_ALL)

print(f"Dataset generado: {len(df)} filas -> raw_qa_dataset.csv")
print(df["category"].value_counts())
```

**✅ Validación del paso:**

```bash
python -m py_compile generate_raw_dataset.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Ejecuta el generador del dataset

**📝 Descripción del paso:**  
Vas a ejecutar desde Git Bash el script `generate_raw_dataset.py`. Al ejecutarlo, se creará en la misma carpeta del proyecto el archivo `raw_qa_dataset.csv`, que será la entrada principal del pipeline de preprocesamiento.

**⚙️ Contenido del paso:**

```bash
python generate_raw_dataset.py
```

**✅ Validación del paso:**

```bash
python -c "import pandas as pd; df = pd.read_csv('raw_qa_dataset.csv'); print(df.shape); print(df.columns.tolist())"
```

**📌 Resultado esperado:**  
Debes ver más de 340 filas y columnas:

```text
['id', 'category', 'question', 'answer']
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20de%20un%20laboratorio%20de%20fine-tuning.%20Gener%C3%A9%20un%20dataset%20crudo%20sint%C3%A9tico%20de%20preguntas%20y%20respuestas%20con%20duplicados%2C%20nulos%2C%20respuestas%20cortas%2C%20caracteres%20inv%C3%A1lidos%2C%20datos%20sensibles%20simulados%20y%20categor%C3%ADas%20desbalanceadas.)

---

# 🧩 Tarea 3. Crear el pipeline base e inspeccionar el dataset

## 🎯 Objetivo de la tarea

Crear el script principal `dataset_preprocessor.py` e implementar una función de inspección inicial para entender la calidad del dataset antes de limpiarlo.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `dataset_preprocessor.py`

**📝 Descripción del paso:**  
Vas a crear en Visual Studio Code el archivo principal `dataset_preprocessor.py`. En este archivo agregarás el código del pipeline por partes durante las siguientes tareas, por lo que debes mantenerlo abierto para editarlo paso a paso.

**⚙️ Contenido del paso:**

Crea el archivo:

```text
dataset_preprocessor.py
```

Agrega este contenido inicial:

```python
"""
Pipeline profesional de preprocesamiento de datos para fine-tuning.
"""

from __future__ import annotations

import difflib
import re
import unicodedata
from dataclasses import dataclass, field
from datetime import datetime
from pathlib import Path
from typing import Any

import jsonlines
import pandas as pd
import tiktoken

INPUT_FILE = "raw_qa_dataset.csv"
TRAIN_FILE = "train.jsonl"
VALIDATION_FILE = "validation.jsonl"
REPORT_FILE = "preprocessing_report.md"
DECISION_MATRIX_FILE = "decision_matrix.md"
MANUAL_REVIEW_FILE = "manual_review_checklist.md"

MAX_TOKENS_PER_EXAMPLE = 4096
MIN_WORDS_IN_ANSWER = 20
DEDUP_SIMILARITY_THRESHOLD = 0.99
MIN_EXAMPLES_PER_CATEGORY = 10
TRAIN_SPLIT_RATIO = 0.90

FINE_TUNING_MODEL = "modelo-fine-tuning-a-validar"
BASE_MODEL_FOR_FEWSHOT = "modelo-base-a-validar"
PRICE_FINETUNING_INPUT_PER_1K = 0.003
PRICE_FEWSHOT_INPUT_PER_1K = 0.005
PRICE_FEWSHOT_OUTPUT_PER_1K = 0.015
EXPECTED_QUERIES_PER_MONTH = 1000
EPOCHS = 3

SYSTEM_PROMPT = (
    "Eres un agente de soporte técnico especializado en software empresarial. "
    "Responde de manera clara, concisa, profesional y orientada a la solución. "
    "Si no tienes información suficiente, indícalo y sugiere contactar al equipo de soporte avanzado."
)

SENSITIVE_PATTERNS = {
    "email": re.compile(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+"),
    "openai_like_key": re.compile(r"sk-[a-zA-Z0-9_-]{8,}"),
    "aws_access_key": re.compile(r"AKIA[0-9A-Z]{12,}"),
    "password_word": re.compile(r"(?i)(password|contraseña|passwd|secret|token|api[_-]?key)"),
    "phone_like": re.compile(r"\+?\d[\d\s().-]{8,}\d"),
}

@dataclass
class CleaningStats:
    raw_count: int = 0
    removed_empty: int = 0
    removed_short: int = 0
    removed_duplicates: int = 0
    final_count: int = 0

@dataclass
class SensitiveFinding:
    row_index: int
    category: str
    field: str
    pattern_name: str
    preview: str

@dataclass
class ValidationReport:
    total_examples: int = 0
    valid_examples: int = 0
    invalid_examples: int = 0
    token_violations: list[dict[str, Any]] = field(default_factory=list)
    format_violations: list[dict[str, Any]] = field(default_factory=list)
    category_distribution: dict[str, int] = field(default_factory=dict)
    category_violations: list[str] = field(default_factory=list)
    avg_tokens_per_example: float = 0.0
    max_tokens_found: int = 0
    train_count: int = 0
    validation_count: int = 0
    passed: bool = False

@dataclass
class CostAnalysis:
    total_training_tokens: int = 0
    epochs: int = EPOCHS
    finetuning_cost_usd: float = 0.0
    fewshot_monthly_cost_usd: float = 0.0
    expected_queries_per_month: int = EXPECTED_QUERIES_PER_MONTH
    break_even_queries: int = 0
    recommendation: str = ""
```

**✅ Validación del paso:**

```bash
python -m py_compile dataset_preprocessor.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Implementa `load_and_inspect()`

**📝 Descripción del paso:**  
Vas a editar el archivo `dataset_preprocessor.py` y agregar la función `load_and_inspect()` al final del archivo. Esta función leerá `raw_qa_dataset.csv` y mostrará estadísticas iniciales para identificar nulos, categorías, longitudes y problemas del dataset crudo.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def load_and_inspect(filepath: str) -> pd.DataFrame:
    print("\n" + "=" * 70)
    print("PASO 1: CARGA E INSPECCIÓN DEL DATASET")
    print("=" * 70)

    if not Path(filepath).exists():
        raise FileNotFoundError(f"No se encontró el archivo: {filepath}")

    df = pd.read_csv(filepath, encoding="utf-8", keep_default_na=True)
    required_columns = {"id", "category", "question", "answer"}
    missing = required_columns - set(df.columns)
    if missing:
        raise ValueError(f"Faltan columnas requeridas: {missing}")

    print(f"\n📂 Archivo cargado: {filepath}")
    print(f"   Filas totales : {len(df)}")
    print(f"   Columnas      : {list(df.columns)}")

    print("\n🔍 Valores nulos por columna:")
    for column, count in df.isnull().sum().items():
        icon = "⚠️" if count else "✅"
        print(f"   {icon} {column}: {count}")

    print("\n📊 Distribución por categoría:")
    for category, count in df["category"].value_counts().items():
        bar = "█" * max(1, count // 10)
        print(f"   {category:<15} {count:>4}  {bar}")

    question_words = df["question"].fillna("").astype(str).apply(lambda text: len(text.split()))
    answer_words = df["answer"].fillna("").astype(str).apply(lambda text: len(text.split()))

    print("\n📏 Longitud en palabras:")
    print(f"   Preguntas  min={question_words.min()} media={question_words.mean():.1f} max={question_words.max()}")
    print(f"   Respuestas min={answer_words.min()} media={answer_words.mean():.1f} max={answer_words.max()}")

    empty_answers = df[df["answer"].isnull() | (df["answer"].fillna("").astype(str).str.strip() == "")]
    short_answers = df[answer_words < MIN_WORDS_IN_ANSWER]

    print("\n⚠️ Problemas iniciales detectados:")
    print(f"   Respuestas vacías o nulas  : {len(empty_answers)}")
    print(f"   Respuestas cortas          : {len(short_answers)}")
    print(f"   Categorías únicas          : {df['category'].nunique()}")
    return df
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import load_and_inspect; df = load_and_inspect('raw_qa_dataset.csv'); print('Shape:', df.shape)"
```

**📌 Resultado esperado:**  
Debes ver estadísticas iniciales del dataset y el número de filas cargadas.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20de%20un%20laboratorio%20de%20fine-tuning.%20Cre%C3%A9%20dataset_preprocessor.py%2C%20defin%C3%AD%20constantes%2C%20dataclasses%20de%20reporte%20y%20una%20funci%C3%B3n%20load_and_inspect%20para%20inspeccionar%20nulos%2C%20categor%C3%ADas%2C%20longitudes%20y%20problemas%20iniciales%20del%20dataset.)

---

# 🧩 Tarea 4. Limpiar, normalizar y deduplicar el dataset

## 🎯 Objetivo de la tarea

Eliminar registros problemáticos, normalizar texto y remover duplicados exactos o muy similares para mejorar la calidad del dataset antes de convertirlo a JSONL.

---

## 🛠️ Pasos

### ✅ Paso 1. Implementa funciones auxiliares de limpieza

**📝 Descripción del paso:**  
Vas a volver a abrir o mantener abierto `dataset_preprocessor.py` y agregar al final del archivo las funciones auxiliares `normalize_text()` e `is_near_duplicate()`. Estas funciones serán usadas después por la limpieza principal del dataset.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def normalize_text(text: Any) -> str:
    if pd.isna(text):
        return ""
    normalized = unicodedata.normalize("NFC", str(text))
    normalized = re.sub(r"[\x00-\x08\x0b\x0c\x0e-\x1f\x7f]", "", normalized)
    normalized = normalized.replace("\r\n", "\n").replace("\r", "\n")
    normalized = re.sub(r"[ \t]{2,}", " ", normalized)
    normalized = re.sub(r"\n{3,}", "\n\n", normalized)
    return normalized.strip()

def is_near_duplicate(text_a: str, text_b: str, threshold: float = DEDUP_SIMILARITY_THRESHOLD) -> bool:
    ratio = difflib.SequenceMatcher(None, text_a.lower(), text_b.lower()).ratio()
    return ratio >= threshold
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import normalize_text, is_near_duplicate; print(normalize_text('Hola\\x00 mundo')); print(is_near_duplicate('instalar software', 'instalar el software'))"
```

**📌 Resultado esperado:**  
Se imprime texto limpio y una comparación booleana.

---

### ✅ Paso 2. Implementa `clean_dataset()`

**📝 Descripción del paso:**  
Vas a editar nuevamente `dataset_preprocessor.py` y agregar al final la función `clean_dataset()`. Esta función usará las funciones auxiliares anteriores para limpiar preguntas, respuestas, categorías y eliminar registros problemáticos.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def clean_dataset(df: pd.DataFrame) -> tuple[pd.DataFrame, CleaningStats]:
    print("\n" + "=" * 70)
    print("PASO 2: LIMPIEZA, NORMALIZACIÓN Y DEDUPLICACIÓN")
    print("=" * 70)

    stats = CleaningStats(raw_count=len(df))
    clean_df = df.copy()
    clean_df["question"] = clean_df["question"].apply(normalize_text)
    clean_df["answer"] = clean_df["answer"].apply(normalize_text)
    clean_df["category"] = clean_df["category"].apply(normalize_text).str.lower()

    before = len(clean_df)
    clean_df = clean_df[(clean_df["question"] != "") & (clean_df["answer"] != "")]
    stats.removed_empty = before - len(clean_df)
    print(f"\n🗑️ Filas con pregunta/respuesta vacía eliminadas: {stats.removed_empty}")

    before = len(clean_df)
    answer_word_count = clean_df["answer"].apply(lambda text: len(text.split()))
    clean_df = clean_df[answer_word_count >= MIN_WORDS_IN_ANSWER]
    stats.removed_short = before - len(clean_df)
    print(f"🗑️ Respuestas < {MIN_WORDS_IN_ANSWER} palabras eliminadas: {stats.removed_short}")

    print(f"\n🔄 Deduplicando preguntas con umbral {DEDUP_SIMILARITY_THRESHOLD}...")
    clean_df = clean_df.reset_index(drop=True)
    questions = clean_df["question"].tolist()
    to_remove: set[int] = set()
    for i in range(len(questions)):
        if i in to_remove:
            continue
        for j in range(i + 1, len(questions)):
            if j in to_remove:
                continue
            if is_near_duplicate(questions[i], questions[j]):
                to_remove.add(j)

    before = len(clean_df)
    clean_df = clean_df.drop(index=list(to_remove)).reset_index(drop=True)
    stats.removed_duplicates = before - len(clean_df)
    stats.final_count = len(clean_df)

    print(f"🗑️ Duplicados o casi duplicados eliminados: {stats.removed_duplicates}")
    print("\n📊 Resumen de limpieza:")
    print(f"   Filas iniciales : {stats.raw_count}")
    print(f"   Filas finales   : {stats.final_count}")
    print(f"   Eliminadas      : {stats.raw_count - stats.final_count}")
    return clean_df, stats
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import load_and_inspect, clean_dataset; df = load_and_inspect('raw_qa_dataset.csv'); clean, stats = clean_dataset(df); print(clean.shape); print(stats)"
```

**📌 Resultado esperado:**  
El número de filas debe reducirse y no deben quedar respuestas vacías o demasiado cortas.

---

### ✅ Paso 3. Valida la limpieza

**📝 Descripción del paso:**  
Vas a ejecutar desde Git Bash una validación directa sobre el código que ya agregaste en `dataset_preprocessor.py`. Esta prueba carga el CSV, ejecuta la limpieza y confirma que no queden preguntas vacías, respuestas vacías ni respuestas demasiado cortas.

**⚙️ Contenido del paso:**

```bash
python -c "
from dataset_preprocessor import load_and_inspect, clean_dataset, MIN_WORDS_IN_ANSWER

df = load_and_inspect('raw_qa_dataset.csv')
clean, stats = clean_dataset(df)
assert clean['question'].str.strip().eq('').sum() == 0
assert clean['answer'].str.strip().eq('').sum() == 0
assert clean['answer'].apply(lambda x: len(x.split())).min() >= MIN_WORDS_IN_ANSWER
assert len(clean) < len(df)
print('✅ Limpieza validada correctamente')
"
```

**📌 Resultado esperado:**

```text
✅ Limpieza validada correctamente
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20de%20un%20laboratorio%20de%20fine-tuning.%20Limpi%C3%A9%20un%20dataset%20eliminando%20nulos%2C%20respuestas%20cortas%2C%20caracteres%20de%20control%20y%20preguntas%20duplicadas%20o%20casi%20duplicadas%20usando%20difflib.SequenceMatcher.)

---

# 🧩 Tarea 5. Detectar posibles datos sensibles

## 🎯 Objetivo de la tarea

Identificar posibles datos sensibles o secretos dentro del dataset antes de exportar archivos para fine-tuning.

---

## 🛠️ Pasos

### ✅ Paso 1. Implementa `detect_sensitive_data()`

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final las funciones `mask_preview()` y `detect_sensitive_data()`. Estas funciones revisarán el dataset limpio para localizar posibles correos, tokens, contraseñas, claves o teléfonos antes de exportar datos.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def mask_preview(text: str, max_len: int = 90) -> str:
    preview = text.replace("\n", " ")[:max_len]
    return preview + ("..." if len(text) > max_len else "")

def detect_sensitive_data(df: pd.DataFrame) -> list[SensitiveFinding]:
    print("\n" + "=" * 70)
    print("PASO 3: DETECCIÓN DE POSIBLES DATOS SENSIBLES")
    print("=" * 70)
    findings: list[SensitiveFinding] = []
    for row_index, row in df.iterrows():
        for field_name in ["question", "answer"]:
            text = str(row.get(field_name, ""))
            for pattern_name, pattern in SENSITIVE_PATTERNS.items():
                if pattern.search(text):
                    findings.append(SensitiveFinding(
                        row_index=int(row_index),
                        category=str(row.get("category", "unknown")),
                        field=field_name,
                        pattern_name=pattern_name,
                        preview=mask_preview(text),
                    ))
    if findings:
        print(f"\n⚠️ Posibles datos sensibles detectados: {len(findings)}")
        for finding in findings[:10]:
            print(f"   fila={finding.row_index} campo={finding.field} tipo={finding.pattern_name} preview={finding.preview}")
    else:
        print("\n✅ No se detectaron patrones sensibles conocidos")
    return findings
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import load_and_inspect, clean_dataset, detect_sensitive_data; df = load_and_inspect('raw_qa_dataset.csv'); clean, _ = clean_dataset(df); findings = detect_sensitive_data(clean); print('Hallazgos:', len(findings))"
```

**📌 Resultado esperado:**  
Debes ver algunos hallazgos simulados, como correos, tokens o palabras relacionadas con contraseña.

---

### ✅ Paso 2. Interpreta el resultado

**📝 Descripción del paso:**  
Vas a revisar en la terminal los hallazgos que devuelve `detect_sensitive_data()`. En este paso no editas código; interpretas el resultado y confirmas que cualquier patrón detectado debe revisarse manualmente antes de usar un dataset en entrenamiento real.

**⚙️ Contenido del paso:**

| Resultado | Interpretación |
|---|---|
| 0 hallazgos | No se encontraron patrones conocidos, pero aún se requiere revisión manual |
| 1 a 10 hallazgos | Revisa manualmente cada caso |
| Más de 10 hallazgos | Detén el proceso y revisa la fuente de datos |

**✅ Validación del paso:**  
Confirma que los hallazgos aparecen en el reporte final.

**📌 Resultado esperado:**  
Comprendes que el dataset no debe exportarse a entrenamiento real sin revisión humana.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20de%20un%20laboratorio%20de%20fine-tuning.%20Agregu%C3%A9%20detecci%C3%B3n%20de%20datos%20sensibles%20con%20regex%20para%20identificar%20emails%2C%20tokens%2C%20contrase%C3%B1as%2C%20claves%20y%20tel%C3%A9fonos%20antes%20de%20exportar%20un%20dataset%20a%20JSONL.)

---

# 🧩 Tarea 6. Convertir el dataset al formato JSONL de mensajes

## 🎯 Objetivo de la tarea

Transformar cada fila limpia del dataset en un ejemplo tipo chat con roles `system`, `user` y `assistant`.

---

## 🛠️ Pasos

### ✅ Paso 1. Implementa `convert_to_jsonl_examples()`

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `convert_to_jsonl_examples()`. Esta función convertirá cada fila limpia en una estructura de mensajes con roles `system`, `user` y `assistant`.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def convert_to_jsonl_examples(df: pd.DataFrame, system_prompt: str = SYSTEM_PROMPT) -> list[dict[str, Any]]:
    print("\n" + "=" * 70)
    print("PASO 4: CONVERSIÓN A FORMATO JSONL")
    print("=" * 70)
    examples: list[dict[str, Any]] = []
    for _, row in df.iterrows():
        examples.append({
            "messages": [
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": str(row["question"])},
                {"role": "assistant", "content": str(row["answer"])},
            ],
            "_category": str(row.get("category", "unknown")),
        })
    print(f"\n✅ Ejemplos convertidos: {len(examples)}")
    for message in examples[0]["messages"]:
        print(f"   {message['role']:<9}: {message['content'][:80]}...")
    return examples
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import *; df = load_and_inspect('raw_qa_dataset.csv'); clean, _ = clean_dataset(df); examples = convert_to_jsonl_examples(clean); print(examples[0].keys())"
```

**📌 Resultado esperado:**  
Cada ejemplo tiene `messages` y `_category` como metadato auxiliar interno.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20de%20un%20laboratorio%20de%20fine-tuning.%20Convert%C3%AD%20un%20dataset%20limpio%20de%20preguntas%20y%20respuestas%20al%20formato%20JSONL%20de%20mensajes%20con%20roles%20system%2C%20user%20y%20assistant.)

---

# 🧩 Tarea 7. Validar estructura, tokens y distribución

## 🎯 Objetivo de la tarea

Validar que los ejemplos tengan formato correcto, orden exacto de roles, contenido no vacío, límite de tokens aceptable y una distribución mínima por categoría.

---

## 🛠️ Pasos

### ✅ Paso 1. Implementa el contador de tokens

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `count_tokens_for_messages()`. Esta función usará `tiktoken` para estimar cuántos tokens contiene cada ejemplo antes de exportarlo.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def count_tokens_for_messages(messages: list[dict[str, str]], model: str = "gpt-4o-mini") -> int:
    try:
        encoding = tiktoken.encoding_for_model(model)
    except KeyError:
        encoding = tiktoken.get_encoding("cl100k_base")
    tokens = 3
    for message in messages:
        tokens += 4
        tokens += len(encoding.encode(message.get("role", "")))
        tokens += len(encoding.encode(message.get("content", "")))
    tokens += 3
    return tokens
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import count_tokens_for_messages; print(count_tokens_for_messages([{'role':'user','content':'Hola'}]))"
```

**📌 Resultado esperado:**  
Debes ver un número entero de tokens.

---

### ✅ Paso 2. Implementa `validate_dataset()`

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `validate_dataset()`. Esta función revisará que cada ejemplo tenga los roles en el orden correcto, contenido no vacío, tokens dentro del límite y categorías con suficientes ejemplos.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def validate_dataset(examples: list[dict[str, Any]]) -> ValidationReport:
    print("\n" + "=" * 70)
    print("PASO 5: VALIDACIÓN DEL DATASET")
    print("=" * 70)
    report = ValidationReport(total_examples=len(examples))
    token_counts: list[int] = []
    expected_roles = ["system", "user", "assistant"]

    for index, example in enumerate(examples):
        is_valid = True
        messages = example.get("messages", [])
        category = str(example.get("_category", "unknown"))
        roles = [message.get("role") for message in messages]
        if roles != expected_roles:
            report.format_violations.append({"index": index, "issue": f"Orden de roles inválido: {roles}"})
            is_valid = False
        for message in messages:
            role = message.get("role")
            content = str(message.get("content", "")).strip()
            if role not in expected_roles:
                report.format_violations.append({"index": index, "issue": f"Rol no permitido: {role}"})
                is_valid = False
            if not content:
                report.format_violations.append({"index": index, "issue": f"Contenido vacío en rol: {role}"})
                is_valid = False
        token_count = count_tokens_for_messages(messages)
        token_counts.append(token_count)
        if token_count > MAX_TOKENS_PER_EXAMPLE:
            report.token_violations.append({"index": index, "tokens": token_count, "limit": MAX_TOKENS_PER_EXAMPLE})
            is_valid = False
        report.category_distribution[category] = report.category_distribution.get(category, 0) + 1
        if is_valid:
            report.valid_examples += 1
        else:
            report.invalid_examples += 1

    for category, count in report.category_distribution.items():
        if count < MIN_EXAMPLES_PER_CATEGORY:
            report.category_violations.append(
                f"Categoría '{category}' tiene {count} ejemplos; mínimo recomendado: {MIN_EXAMPLES_PER_CATEGORY}"
            )

    if token_counts:
        report.avg_tokens_per_example = sum(token_counts) / len(token_counts)
        report.max_tokens_found = max(token_counts)

    report.train_count = int(report.valid_examples * TRAIN_SPLIT_RATIO)
    report.validation_count = report.valid_examples - report.train_count
    report.passed = (
        report.invalid_examples == 0
        and len(report.token_violations) == 0
        and len(report.format_violations) == 0
        and len(report.category_violations) == 0
        and report.valid_examples >= 50
    )

    status = "✅ PASÓ" if report.passed else "⚠️ REQUIERE REVISIÓN"
    print(f"\n{status}")
    print(f"   Ejemplos totales     : {report.total_examples}")
    print(f"   Ejemplos válidos     : {report.valid_examples}")
    print(f"   Ejemplos inválidos   : {report.invalid_examples}")
    print(f"   Tokens promedio      : {report.avg_tokens_per_example:.1f}")
    print(f"   Tokens máximo        : {report.max_tokens_found}")
    print(f"   Train                : {report.train_count}")
    print(f"   Validation           : {report.validation_count}")
    return report
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import *; df = load_and_inspect('raw_qa_dataset.csv'); clean, _ = clean_dataset(df); examples = convert_to_jsonl_examples(clean); report = validate_dataset(examples); print(report.passed)"
```

**📌 Resultado esperado:**  
La validación debe pasar si el dataset quedó limpio y con categorías suficientes.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%207%20de%20un%20laboratorio%20de%20fine-tuning.%20Valid%C3%A9%20un%20dataset%20JSONL%20revisando%20orden%20exacto%20de%20roles%2C%20contenido%20no%20vac%C3%ADo%2C%20conteo%20de%20tokens%2C%20distribuci%C3%B3n%20por%20categor%C3%ADa%20y%20split%20train-validation.)

---

# 🧩 Tarea 8. Calcular costos y construir matriz de decisión

## 🎯 Objetivo de la tarea

Comparar fine-tuning, RAG y few-shot prompting usando criterios técnicos, no solo costo.

---

## 🛠️ Pasos

### ✅ Paso 1. Implementa `calculate_cost_analysis()`

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `calculate_cost_analysis()`. Esta función usará los ejemplos ya convertidos para estimar costos didácticos de fine-tuning y compararlos con un escenario de few-shot prompting.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def calculate_cost_analysis(examples: list[dict[str, Any]]) -> CostAnalysis:
    print("\n" + "=" * 70)
    print("PASO 6: ANÁLISIS DE COSTO")
    print("=" * 70)
    analysis = CostAnalysis()
    total_training_tokens = sum(count_tokens_for_messages(example["messages"]) for example in examples)
    analysis.total_training_tokens = total_training_tokens
    analysis.finetuning_cost_usd = total_training_tokens * analysis.epochs * PRICE_FINETUNING_INPUT_PER_1K / 1000
    avg_tokens = total_training_tokens / len(examples) if examples else 0
    fewshot_input_tokens = 5 * avg_tokens + 200
    fewshot_output_tokens = avg_tokens * 0.6
    monthly_input_cost = analysis.expected_queries_per_month * fewshot_input_tokens * PRICE_FEWSHOT_INPUT_PER_1K / 1000
    monthly_output_cost = analysis.expected_queries_per_month * fewshot_output_tokens * PRICE_FEWSHOT_OUTPUT_PER_1K / 1000
    analysis.fewshot_monthly_cost_usd = monthly_input_cost + monthly_output_cost
    cost_per_query = analysis.fewshot_monthly_cost_usd / analysis.expected_queries_per_month
    analysis.break_even_queries = int(analysis.finetuning_cost_usd / cost_per_query) if cost_per_query else 0
    analysis.recommendation = (
        "Fine-tuning puede ser conveniente si necesitas respuestas con formato, tono o comportamiento consistente "
        "y tienes suficientes ejemplos de alta calidad. Si el conocimiento cambia con frecuencia o necesitas citar fuentes, "
        "RAG suele ser mejor opción. Si el volumen es bajo, few-shot puede ser suficiente."
    )
    print(f"\n💰 Costo estimado de fine-tuning: ${analysis.finetuning_cost_usd:.4f} USD")
    print(f"💬 Costo mensual few-shot estimado: ${analysis.fewshot_monthly_cost_usd:.4f} USD")
    print(f"⚖️ Punto de equilibrio aproximado: {analysis.break_even_queries:,} consultas")
    print("\n⚠️ Estos precios son valores de referencia. Actualízalos antes de una decisión real.")
    return analysis
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import *; df = load_and_inspect('raw_qa_dataset.csv'); clean, _ = clean_dataset(df); examples = convert_to_jsonl_examples(clean); cost = calculate_cost_analysis(examples); print(cost)"
```

**📌 Resultado esperado:**  
Debes ver costos estimados y una recomendación conceptual.

---

### ✅ Paso 2. Implementa la matriz de decisión

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `generate_decision_matrix()`. Esta función generará el archivo `decision_matrix.md` en la raíz del proyecto para comparar Fine-Tuning, RAG y Few-shot Prompting.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def generate_decision_matrix(output_path: str = DECISION_MATRIX_FILE) -> None:
    content = """# Matriz de decisión: Fine-Tuning vs. RAG vs. Few-shot

| Criterio | Fine-Tuning | RAG | Few-shot Prompting |
|---|---|---|---|
| Necesitas formato o tono consistente | ✅ Muy fuerte | ⚠️ Depende del prompt | ✅ Bueno |
| El conocimiento cambia frecuentemente | ❌ Débil | ✅ Muy fuerte | ⚠️ Manual |
| Necesitas citar fuentes | ❌ Débil | ✅ Muy fuerte | ❌ Débil |
| Tienes muchos ejemplos de alta calidad | ✅ Requerido | ⚠️ No obligatorio | ✅ Útil |
| Quieres reducir prompts largos | ✅ Bueno | ⚠️ Depende del contexto | ❌ Débil |
| Volumen mensual bajo | ⚠️ Puede no convenir | ✅ Puede convenir | ✅ Muy conveniente |
| Necesitas trazabilidad documental | ❌ No ideal | ✅ Ideal | ❌ No ideal |
| Cambia el estilo, no el conocimiento | ✅ Ideal | ⚠️ No es el foco | ✅ Bueno |

## Recomendación práctica

- Usa **Fine-Tuning** cuando quieres especializar comportamiento, tono, formato, clasificación o patrones de respuesta y tienes ejemplos de alta calidad.
- Usa **RAG** cuando necesitas responder con conocimiento actualizado, documentos internos, trazabilidad o citas.
- Usa **Few-shot prompting** cuando el volumen es bajo, estás prototipando o necesitas flexibilidad sin entrenar un modelo.
"""
    Path(output_path).write_text(content, encoding="utf-8")
    print(f"✅ Matriz de decisión generada: {output_path}")
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import generate_decision_matrix; generate_decision_matrix()"
ls -la decision_matrix.md
```

**📌 Resultado esperado:**  
Se genera `decision_matrix.md`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%208%20de%20un%20laboratorio%20de%20fine-tuning.%20Calcul%C3%A9%20costos%20estimados%20y%20constru%C3%AD%20una%20matriz%20de%20decisi%C3%B3n%20para%20comparar%20Fine-Tuning%2C%20RAG%20y%20Few-shot%20Prompting%20usando%20criterios%20t%C3%A9cnicos.)

---

# 🧩 Tarea 9. Exportar archivos JSONL y generar reportes

## 🎯 Objetivo de la tarea

Exportar `train.jsonl`, `validation.jsonl`, `preprocessing_report.md` y `manual_review_checklist.md` como evidencias finales del pipeline.

---

## 🛠️ Pasos

### ✅ Paso 1. Implementa `export_datasets()`

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `export_datasets()`. Esta función dividirá los ejemplos en `train.jsonl` y `validation.jsonl`, y eliminará el metadato auxiliar `_category` antes de guardar los archivos finales.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def export_datasets(examples: list[dict[str, Any]], report: ValidationReport) -> tuple[int, int]:
    print("\n" + "=" * 70)
    print("PASO 7: EXPORTACIÓN DE JSONL")
    print("=" * 70)
    valid_examples = [{"messages": example["messages"]} for example in examples]
    train_examples = valid_examples[:report.train_count]
    validation_examples = valid_examples[report.train_count:report.train_count + report.validation_count]
    with jsonlines.open(TRAIN_FILE, mode="w") as writer:
        writer.write_all(train_examples)
    with jsonlines.open(VALIDATION_FILE, mode="w") as writer:
        writer.write_all(validation_examples)
    print(f"\n✅ {TRAIN_FILE}: {len(train_examples)} ejemplos")
    print(f"✅ {VALIDATION_FILE}: {len(validation_examples)} ejemplos")
    return len(train_examples), len(validation_examples)
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import *; df = load_and_inspect('raw_qa_dataset.csv'); clean, _ = clean_dataset(df); examples = convert_to_jsonl_examples(clean); report = validate_dataset(examples); export_datasets(examples, report)"
```

**📌 Resultado esperado:**  
Se generan `train.jsonl` y `validation.jsonl`.

---

### ✅ Paso 2. Implementa el reporte técnico

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `generate_preprocessing_report()`. Esta función generará el archivo `preprocessing_report.md` con métricas de limpieza, validación, posibles datos sensibles y análisis de costo.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def generate_preprocessing_report(
    cleaning_stats: CleaningStats,
    sensitive_findings: list[SensitiveFinding],
    validation_report: ValidationReport,
    cost_analysis: CostAnalysis,
    output_path: str = REPORT_FILE,
) -> None:
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    status = "✅ DATASET LISTO PARA REVISIÓN FINAL" if validation_report.passed else "⚠️ DATASET REQUIERE REVISIÓN"
    category_rows = "\n".join(
        f"| {category} | {count} | {'✅ OK' if count >= MIN_EXAMPLES_PER_CATEGORY else '⚠️ Bajo'} |"
        for category, count in sorted(validation_report.category_distribution.items())
    )
    sensitive_rows = "\n".join(
        f"| {f.row_index} | {f.category} | {f.field} | {f.pattern_name} | {f.preview} |"
        for f in sensitive_findings[:25]
    ) or "| N/A | N/A | N/A | N/A | No se detectaron hallazgos |"

    markdown = f"""# Reporte de Preprocesamiento para Fine-Tuning

**Generado:** {timestamp}  
**Estado:** {status}

---

## 1. Resumen ejecutivo

| Métrica | Valor |
|---|---:|
| Filas crudas | {cleaning_stats.raw_count} |
| Filas finales | {cleaning_stats.final_count} |
| Filas vacías eliminadas | {cleaning_stats.removed_empty} |
| Respuestas cortas eliminadas | {cleaning_stats.removed_short} |
| Duplicados eliminados | {cleaning_stats.removed_duplicates} |
| Ejemplos válidos | {validation_report.valid_examples} |
| Ejemplos inválidos | {validation_report.invalid_examples} |

---

## 2. Validación de estructura y tokens

| Métrica | Valor |
|---|---:|
| Tokens promedio por ejemplo | {validation_report.avg_tokens_per_example:.1f} |
| Tokens máximos encontrados | {validation_report.max_tokens_found} |
| Límite configurado | {MAX_TOKENS_PER_EXAMPLE} |
| Violaciones de formato | {len(validation_report.format_violations)} |
| Violaciones de tokens | {len(validation_report.token_violations)} |
| Ejemplos de entrenamiento | {validation_report.train_count} |
| Ejemplos de validación | {validation_report.validation_count} |

---

## 3. Distribución por categoría

| Categoría | Ejemplos | Estado |
|---|---:|---|
{category_rows}

---

## 4. Posibles datos sensibles detectados

| Fila | Categoría | Campo | Patrón | Vista previa |
|---:|---|---|---|---|
{sensitive_rows}

> Revisa manualmente estos hallazgos antes de usar un dataset real para entrenamiento.

---

## 5. Análisis de costo didáctico

| Concepto | Valor |
|---|---:|
| Tokens de entrenamiento | {cost_analysis.total_training_tokens:,} |
| Épocas | {cost_analysis.epochs} |
| Costo estimado fine-tuning | ${cost_analysis.finetuning_cost_usd:.4f} USD |
| Costo mensual few-shot estimado | ${cost_analysis.fewshot_monthly_cost_usd:.4f} USD |
| Punto de equilibrio aproximado | {cost_analysis.break_even_queries:,} consultas |

**Recomendación:**  
{cost_analysis.recommendation}
"""
    Path(output_path).write_text(markdown, encoding="utf-8")
    print(f"✅ Reporte generado: {output_path}")
```

**✅ Validación del paso:**

```bash
python -m py_compile dataset_preprocessor.py
```

**📌 Resultado esperado:**  
El script compila sin errores.

---

### ✅ Paso 3. Genera checklist de revisión manual

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `generate_manual_review_checklist()`. Esta función creará el archivo `manual_review_checklist.md`, que servirá como evidencia de revisión humana del dataset.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def generate_manual_review_checklist(output_path: str = MANUAL_REVIEW_FILE) -> None:
    content = """# Checklist de revisión manual del dataset

Revisa al menos 10 ejemplos de `train.jsonl` y 5 ejemplos de `validation.jsonl` antes de usar el dataset para entrenamiento real.

| Ejemplo | Pregunta clara | Respuesta útil | Categoría correcta | Sin datos sensibles | Formato correcto | Aprobado |
|---:|---|---|---|---|---|---|
| 1 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 2 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 3 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 4 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 5 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 6 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 7 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 8 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 9 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| 10 | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |

## Criterios de revisión

- La pregunta debe ser comprensible sin contexto externo.
- La respuesta debe resolver la pregunta de forma completa y segura.
- La respuesta no debe inventar información.
- No debe haber credenciales, tokens, correos reales, teléfonos ni datos personales.
- El ejemplo debe tener exactamente los roles `system`, `user` y `assistant` en ese orden.
- Si el conocimiento cambia con frecuencia, considera RAG en lugar de fine-tuning.
"""
    Path(output_path).write_text(content, encoding="utf-8")
    print(f"✅ Checklist manual generado: {output_path}")
```

**✅ Validación del paso:**

```bash
python -c "from dataset_preprocessor import generate_manual_review_checklist; generate_manual_review_checklist()"
```

**📌 Resultado esperado:**  
Se genera `manual_review_checklist.md`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 9 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%209%20de%20un%20laboratorio%20de%20fine-tuning.%20Export%C3%A9%20train.jsonl%2C%20validation.jsonl%2C%20gener%C3%A9%20un%20reporte%20Markdown%20de%20calidad%20y%20un%20checklist%20de%20revisi%C3%B3n%20manual%20para%20validar%20el%20dataset.)

---

# 🧩 Tarea 10. Orquestar el pipeline completo y validar evidencias

## 🎯 Objetivo de la tarea

Crear la función `main()` para ejecutar todo el pipeline de inicio a fin y validar los archivos generados.

---

## 🛠️ Pasos

### ✅ Paso 1. Implementa `main()`

**📝 Descripción del paso:**  
Vas a editar `dataset_preprocessor.py` y agregar al final la función `main()` junto con el bloque `if __name__ == "__main__":`. Este bloque permitirá ejecutar todo el pipeline completo con un solo comando.

**⚙️ Contenido del paso:**

Agrega este bloque al final de `dataset_preprocessor.py`:

```python
def main() -> None:
    print("\n" + "🚀" + "=" * 69)
    print(" PIPELINE DE PREPROCESAMIENTO DE DATASET PARA FINE-TUNING")
    print("🚀" + "=" * 69)
    df_raw = load_and_inspect(INPUT_FILE)
    df_clean, cleaning_stats = clean_dataset(df_raw)
    sensitive_findings = detect_sensitive_data(df_clean)
    examples = convert_to_jsonl_examples(df_clean)
    validation_report = validate_dataset(examples)
    cost_analysis = calculate_cost_analysis(examples)
    export_datasets(examples, validation_report)
    generate_decision_matrix()
    generate_manual_review_checklist()
    generate_preprocessing_report(
        cleaning_stats=cleaning_stats,
        sensitive_findings=sensitive_findings,
        validation_report=validation_report,
        cost_analysis=cost_analysis,
    )
    print("\n" + "=" * 70)
    print("✅ PIPELINE COMPLETADO")
    print("=" * 70)
    print("\nArchivos generados:")
    for filename in [TRAIN_FILE, VALIDATION_FILE, REPORT_FILE, DECISION_MATRIX_FILE, MANUAL_REVIEW_FILE]:
        print(f"   📄 {filename}")
    if sensitive_findings:
        print("\n⚠️ Revisa los posibles datos sensibles antes de usar el dataset en un entrenamiento real.")

if __name__ == "__main__":
    main()
```

**✅ Validación del paso:**

```bash
python -m py_compile dataset_preprocessor.py
```

**📌 Resultado esperado:**  
El script principal compila sin errores.

---

### ✅ Paso 2. Ejecuta el pipeline completo

**📝 Descripción del paso:**  
Vas a ejecutar desde Git Bash el archivo `dataset_preprocessor.py`. Este comando tomará `raw_qa_dataset.csv` como entrada y generará los archivos finales del laboratorio en la misma carpeta del proyecto.

**⚙️ Contenido del paso:**

```bash
python dataset_preprocessor.py
```

**✅ Validación del paso:**

```bash
ls -la
```

Debes encontrar:

```text
train.jsonl
validation.jsonl
preprocessing_report.md
decision_matrix.md
manual_review_checklist.md
```

**📌 Resultado esperado:**  
El pipeline se ejecuta completo y genera las evidencias.

---

### ✅ Paso 3. Valida la estructura de los JSONL

**📝 Descripción del paso:**  
Vas a ejecutar desde Git Bash una validación sobre `train.jsonl` y `validation.jsonl`. Esta prueba confirma que los archivos existen, no están vacíos, tienen la clave `messages`, conservan el orden de roles y no incluyen el metadato auxiliar `_category`.

**⚙️ Contenido del paso:**

```bash
python -c "
import jsonlines
for path in ['train.jsonl', 'validation.jsonl']:
    with jsonlines.open(path) as reader:
        items = list(reader)
    assert len(items) > 0, f'{path} está vacío'
    for item in items[:10]:
        assert 'messages' in item
        roles = [m['role'] for m in item['messages']]
        assert roles == ['system', 'user', 'assistant'], roles
        assert '_category' not in item
    print(f'✅ {path}: {len(items)} ejemplos válidos')
"
```

**📌 Resultado esperado:**

```text
✅ train.jsonl: ... ejemplos válidos
✅ validation.jsonl: ... ejemplos válidos
```

---

### ✅ Paso 4. Crea una suite de validación final

**📝 Descripción del paso:**  
Vas a crear en la raíz del proyecto el archivo `validation_test.py`. Este script realizará una validación final de evidencias: existencia de archivos, contenido no vacío, roles correctos en JSONL y secciones esperadas del reporte Markdown.

**⚙️ Contenido del paso:**

Crea `validation_test.py`:

```bash
cat > validation_test.py << 'EOF'
"""Suite de validación final del Laboratorio 7."""
from pathlib import Path
import jsonlines

REQUIRED_FILES = [
    "train.jsonl",
    "validation.jsonl",
    "preprocessing_report.md",
    "decision_matrix.md",
    "manual_review_checklist.md",
]

passed = 0
failed = 0

def check(condition: bool, name: str) -> None:
    global passed, failed
    if condition:
        print(f"✅ PASS: {name}")
        passed += 1
    else:
        print(f"❌ FAIL: {name}")
        failed += 1

print("=" * 70)
print("VALIDACIÓN FINAL — LABORATORIO 7")
print("=" * 70)

for file in REQUIRED_FILES:
    check(Path(file).exists(), f"Existe {file}")
    check(Path(file).stat().st_size > 0, f"{file} no está vacío")

for path in ["train.jsonl", "validation.jsonl"]:
    with jsonlines.open(path) as reader:
        items = list(reader)
    check(len(items) > 0, f"{path} tiene ejemplos")
    for index, item in enumerate(items[:5]):
        roles = [message.get("role") for message in item.get("messages", [])]
        check(roles == ["system", "user", "assistant"], f"{path}[{index}] roles correctos")
        check("_category" not in item, f"{path}[{index}] no contiene metadatos auxiliares")

report = Path("preprocessing_report.md").read_text(encoding="utf-8")
for section in ["Resumen ejecutivo", "Validación de estructura", "Posibles datos sensibles", "Análisis de costo"]:
    check(section in report, f"Reporte contiene sección: {section}")

print("=" * 70)
print(f"Resultado: {passed} PASS / {failed} FAIL")
print("=" * 70)
if failed:
    raise SystemExit(1)
EOF
```

Ejecuta:

```bash
python validation_test.py
```

**📌 Resultado esperado:**  
Todos los checks deben aparecer como `PASS`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 10 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2010%20de%20un%20laboratorio%20de%20fine-tuning.%20Orquest%C3%A9%20un%20pipeline%20completo%20que%20carga%2C%20limpia%2C%20detecta%20datos%20sensibles%2C%20convierte%2C%20valida%2C%20calcula%20costos%2C%20exporta%20JSONL%20y%20genera%20reportes%20profesionales.)

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Dataset crudo `raw_qa_dataset.csv`.
4. Script generador `generate_raw_dataset.py`.
5. Pipeline principal `dataset_preprocessor.py`.
6. Dataset limpio convertido a formato JSONL.
7. Archivo `train.jsonl`.
8. Archivo `validation.jsonl`.
9. Reporte `preprocessing_report.md`.
10. Matriz `decision_matrix.md`.
11. Checklist `manual_review_checklist.md`.
12. Suite de validación `validation_test.py`.
13. Evidencia de limpieza, deduplicación y validación.
14. Detección de posibles datos sensibles.
15. Decisión técnica entre Fine-Tuning, RAG y Few-shot.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del entorno local | 10% |
| Generación del dataset crudo | 10% |
| Inspección y limpieza del dataset | 15% |
| Deduplicación y normalización | 10% |
| Detección de datos sensibles | 10% |
| Conversión correcta a JSONL | 15% |
| Validación de roles, tokens y categorías | 15% |
| Reporte técnico y matriz de decisión | 10% |
| Entrega segura de evidencias | 5% |
| Total | 100% |

---

# ⚠️ Errores comunes que debes evitar

1. Usar datos reales de clientes en el dataset.
2. Subir archivos JSONL con información sensible.
3. Validar solo que existan roles, pero no su orden.
4. No revisar manualmente muestras del dataset.
5. Confundir fine-tuning con actualización de conocimiento documental.
6. Tomar decisiones solo por costo y no por caso de uso.
7. Entrenar con respuestas cortas, incompletas o contradictorias.
8. Ignorar categorías con pocos ejemplos.
9. Usar precios desactualizados para justificar una decisión real.
10. Considerar la deduplicación con `difflib` como suficiente para datasets grandes.

---


# 🧯 Nota sobre mensajes de telemetría de ChromaDB

Aunque este laboratorio no usa ChromaDB, es posible que algunos estudiantes vean mensajes heredados de otro laboratorio o del mismo entorno Python, por ejemplo:

```text
Failed to send telemetry event ClientStartEvent: capture() takes 1 positional argument but 3 were given
Failed to send telemetry event ClientCreateCollectionEvent: capture() takes 1 positional argument but 3 were given
```

Estos mensajes corresponden a eventos de telemetría de ChromaDB y no significan, por sí solos, que el código del laboratorio 7 esté mal. Si el script genera `train.jsonl`, `validation.jsonl`, `preprocessing_report.md`, `decision_matrix.md` y `manual_review_checklist.md`, el pipeline funcionó.

Si quieres evitar confusión en el aula, puedes indicar al participante que valide primero si el laboratorio actual realmente usa ChromaDB. En esta práctica, las dependencias declaradas son `pandas`, `tiktoken`, `jsonlines` y `python-dotenv`, por lo que cualquier mensaje de ChromaDB normalmente viene de otro entorno, notebook, terminal previa o práctica anterior.

Si el mensaje aparece en una práctica que sí usa ChromaDB, se puede documentar como advertencia no crítica o desactivar telemetría en el código de ChromaDB usando configuración explícita, por ejemplo con `anonymized_telemetry=False` en los settings del cliente cuando aplique.


# 🧹 Limpieza del entorno

Ejecuta estos comandos solo si quieres limpiar archivos temporales:

```bash
rm -f validation_test.py
```

Para limpiar por completo el entorno virtual:

```bash
deactivate
rm -rf .venv/
```

Si decides subir el laboratorio a un repositorio, revisa antes:

```bash
grep -r "sk-" . --include="*.py" --include="*.txt" --include="*.jsonl" 2>/dev/null \
  && echo "⚠️ Posibles secretos encontrados" \
  || echo "✅ No se encontraron claves con patrón sk-"
```

---

# Cierre de la práctica

En este laboratorio construiste un pipeline profesional para preparar datos antes de un posible fine-tuning. Generaste un dataset sintético con errores, inspeccionaste su calidad, limpiaste registros problemáticos, deduplicaste preguntas similares, detectaste posibles datos sensibles, convertiste ejemplos al formato JSONL, validaste estructura y tokens, generaste reportes y construiste una matriz de decisión entre Fine-Tuning, RAG y Few-shot Prompting.

El aprendizaje más importante es que fine-tuning no inicia entrenando un modelo. Inicia con datos confiables, seguros, representativos y bien validados. Si el dataset no tiene calidad, el modelo tampoco la tendrá.
