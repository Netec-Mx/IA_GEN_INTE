# Script de preprocesamiento de datos para convertir una base de datos de preguntas y respuestas en un dataset validado para fine-tuning

## 1. Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 50 minutos                                   |
| **Complejidad**  | Media                                        |
| **Nivel Bloom**  | Crear                                        |
| **Módulo**       | 7 — Fine-Tuning vs. RAG                      |
| **Costo estimado** | $0 USD (no se ejecuta fine-tuning real)    |

---

## 2. Descripción General

En este laboratorio construirás un pipeline completo de preprocesamiento en Python que transforma un dataset crudo de preguntas y respuestas de soporte técnico (con errores intencionales) en un dataset limpio, validado y listo para ser enviado a la API de fine-tuning de OpenAI. Aplicarás técnicas de limpieza, deduplicación por similitud, conteo de tokens con `tiktoken` y análisis de costo comparativo entre fine-tuning y few-shot prompting. Al finalizar, generarás los archivos `train.jsonl`, `validation.jsonl` y un reporte técnico `preprocessing_report.md`, consolidando así el conocimiento conceptual de la Lección 7.1 en una implementación de nivel producción.

---

## 3. Objetivos de Aprendizaje

- [ ] Implementar un pipeline de preprocesamiento que convierta un CSV de preguntas y respuestas al formato JSONL requerido por la API de fine-tuning de OpenAI.
- [ ] Aplicar técnicas de limpieza, deduplicación por similitud (`difflib`) y balanceo de datos para garantizar la calidad del dataset.
- [ ] Validar el dataset resultante contra los criterios técnicos de OpenAI (longitud máxima de tokens, formato de mensajes, distribución por categoría) y generar un reporte de calidad.
- [ ] Calcular y comparar el costo estimado del fine-tuning frente al costo equivalente de usar few-shot prompting con el modelo base para tomar una decisión informada.

---

## 4. Prerrequisitos

### Conocimiento previo
- Haber completado Lab 01-00-01 (análisis de costos con `tiktoken`).
- Manejo básico de `pandas` para manipulación de DataFrames.
- Comprensión conceptual de Fine-Tuning vs. RAG (Lección 7.1 de este módulo).
- Familiaridad con el formato JSON/JSONL.

### Acceso y credenciales
- **No se requiere API key de OpenAI** para este laboratorio; el fine-tuning real no se ejecuta.
- Acceso a terminal con Python 3.11 y permisos para instalar paquetes vía `pip`.
- Conexión a Internet para instalar dependencias desde PyPI.

---

## 5. Entorno del Laboratorio

### Hardware mínimo recomendado

| Componente       | Mínimo                          |
|------------------|---------------------------------|
| CPU              | 4 núcleos (Intel i5 8va gen / Ryzen 5) |
| RAM              | 8 GB (16 GB recomendado)        |
| Almacenamiento   | 500 MB libres en disco SSD      |
| Red              | Acceso a PyPI (pip install)     |

### Software requerido

| Paquete           | Versión recomendada |
|-------------------|---------------------|
| Python            | 3.11.x              |
| pandas            | 2.2.x               |
| tiktoken          | 0.7.x               |
| jsonlines         | 4.0.x               |
| difflib           | stdlib (incluido)   |
| python-dotenv     | 1.0.x               |

### Configuración inicial del entorno

Ejecuta los siguientes comandos en tu terminal antes de comenzar:

```bash
# 1. Crear y activar entorno virtual aislado para el Lab 07
python -m venv venv_lab07
# En Windows:
venv_lab07\Scripts\activate
# En macOS/Linux:
source venv_lab07/bin/activate

# 2. Instalar dependencias
pip install pandas==2.2.2 tiktoken==0.7.0 jsonlines==4.0.0 python-dotenv==1.0.1

# 3. Verificar instalaciones
python -c "import pandas, tiktoken, jsonlines; print('OK')"
```

### Estructura de archivos del laboratorio

```
lab07/
├── raw_qa_dataset.csv          # Dataset crudo (generado en el Paso 1)
├── dataset_preprocessor.py     # Script principal (construido en los pasos 2-6)
├── train.jsonl                 # Salida: ejemplos de entrenamiento
├── validation.jsonl            # Salida: ejemplos de validación
├── preprocessing_report.md     # Salida: reporte de calidad
├── requirements.txt
└── .gitignore
```

Crea el archivo `.gitignore` y `requirements.txt`:

```bash
# .gitignore
cat > .gitignore << 'EOF'
.env
*.pyc
__pycache__/
venv_lab07/
*.key
EOF

# requirements.txt
cat > requirements.txt << 'EOF'
pandas==2.2.2
tiktoken==0.7.0
jsonlines==4.0.0
python-dotenv==1.0.1
EOF
```

---

## 6. Pasos del Laboratorio

---

### Paso 1: Generar el Dataset Crudo de Prueba

**Objetivo:** Crear el archivo `raw_qa_dataset.csv` con 500 pares de preguntas y respuestas de soporte técnico, incluyendo errores intencionales (duplicados, respuestas vacías, respuestas cortas, caracteres especiales y desbalance de categorías).

#### Instrucciones

1. Crea el archivo `generate_raw_dataset.py` con el siguiente contenido:

```python
# generate_raw_dataset.py
"""
Genera un dataset crudo de soporte técnico con errores intencionales
para usar como entrada del pipeline de preprocesamiento.
"""
import pandas as pd
import random
import csv

random.seed(42)

# ------------------------------------------------------------------
# Datos base por categoría (distribución DESBALANCEADA intencional)
# ------------------------------------------------------------------
CATEGORIES = {
    "instalacion": {
        "count": 200,  # Categoría sobrerepresentada
        "qa_pairs": [
            ("¿Cómo instalo el software en Windows 11?",
             "Para instalar el software en Windows 11, descarga el instalador desde nuestra página oficial, "
             "haz doble clic en el archivo .exe y sigue el asistente de instalación. Asegúrate de tener "
             "permisos de administrador. El proceso tarda aproximadamente 5 minutos."),
            ("¿Puedo instalar en Mac con chip M1?",
             "Sí, el software es compatible con Mac M1 y M2. Descarga la versión ARM64 desde la sección "
             "de descargas. Si tienes problemas, verifica que tu macOS sea Monterey 12 o superior."),
            ("El instalador dice que falta .NET Framework, ¿qué hago?",
             "Debes instalar .NET Framework 4.8 o superior. Ve a la página oficial de Microsoft, descarga "
             "el instalador de .NET y ejecútalo antes de volver a intentar la instalación de nuestro software."),
            ("¿Cuánto espacio en disco necesito?",
             "La instalación completa requiere un mínimo de 2 GB de espacio libre en disco. Recomendamos "
             "tener al menos 5 GB disponibles para archivos temporales y actualizaciones futuras."),
            ("¿Puedo instalar en múltiples computadoras con una sola licencia?",
             "Una licencia estándar permite la instalación en hasta 2 dispositivos simultáneamente. "
             "Si necesitas más instalaciones, considera nuestro plan empresarial que permite instalaciones ilimitadas."),
        ]
    },
    "configuracion": {
        "count": 150,
        "qa_pairs": [
            ("¿Cómo cambio el idioma de la interfaz?",
             "Ve a Configuración > General > Idioma y selecciona tu idioma preferido del menú desplegable. "
             "Los cambios se aplican inmediatamente sin necesidad de reiniciar la aplicación."),
            ("¿Cómo configuro las notificaciones por correo?",
             "En Configuración > Notificaciones > Correo Electrónico, ingresa tu dirección de email y "
             "selecciona los eventos para los que deseas recibir alertas. Haz clic en 'Guardar' para confirmar."),
            ("¿Puedo personalizar los atajos de teclado?",
             "Sí, en Configuración > Accesibilidad > Atajos de Teclado encontrarás un editor visual donde "
             "puedes asignar combinaciones personalizadas a cualquier acción del software."),
            ("¿Cómo configuro el proxy de red?",
             "Ve a Configuración > Red > Proxy. Puedes elegir entre detección automática, sin proxy, o "
             "configuración manual donde ingresas host, puerto, usuario y contraseña de tu proxy corporativo."),
        ]
    },
    "errores": {
        "count": 80,
        "qa_pairs": [
            ("El programa se cierra inesperadamente al abrir un archivo grande",
             "Este error suele deberse a memoria insuficiente. Cierra otras aplicaciones, aumenta la memoria "
             "virtual en las opciones de rendimiento del sistema, o considera actualizar a la versión Pro que "
             "optimiza el manejo de archivos grandes mediante procesamiento por bloques."),
            ("Aparece el error 'Access Denied' al guardar",
             "El error 'Access Denied' indica que no tienes permisos de escritura en la carpeta de destino. "
             "Haz clic derecho en la carpeta, ve a Propiedades > Seguridad y agrega permisos de escritura "
             "para tu usuario, o guarda en una carpeta donde tengas acceso completo."),
            ("La aplicación no responde después de una actualización",
             "Intenta limpiar la caché de la aplicación: ve a %AppData% en Windows o ~/Library/Caches en Mac, "
             "encuentra la carpeta de la aplicación y elimínala. Luego reinicia el software. Si el problema "
             "persiste, usa la opción de reparación en el panel de control."),
        ]
    },
    "licencias": {
        "count": 40,  # Categoría subrepresentada
        "qa_pairs": [
            ("¿Cómo activo mi licencia después de comprar?",
             "Recibirás un correo con tu clave de activación. Abre el software, ve a Ayuda > Activar Licencia "
             "e ingresa la clave. Necesitas conexión a internet para la activación en línea. Si no tienes "
             "internet, usa la activación manual siguiendo las instrucciones del correo."),
            ("Mi licencia expiró, ¿puedo seguir usando el software?",
             "Con la licencia expirada puedes usar el software en modo de solo lectura. Para restaurar la "
             "funcionalidad completa, renueva tu suscripción en nuestra tienda en línea. Los datos y "
             "configuraciones se conservan durante el período de gracia de 30 días."),
        ]
    },
    "rendimiento": {
        "count": 30,  # Categoría muy subrepresentada
        "qa_pairs": [
            ("El software va muy lento en mi computadora",
             "Para mejorar el rendimiento: cierra las pestañas y proyectos que no estés usando, desactiva "
             "las animaciones en Configuración > Rendimiento, y verifica que tu computadora cumpla los "
             "requisitos mínimos. En Windows, asegúrate de que el software use la GPU dedicada en el "
             "panel de control de gráficos."),
            ("¿Cómo puedo reducir el uso de memoria RAM?",
             "En Configuración > Rendimiento > Memoria, reduce el tamaño del caché de trabajo. También "
             "puedes activar el modo de bajo consumo que limita los procesos en segundo plano. Para "
             "proyectos grandes, considera aumentar la RAM de tu sistema a 16 GB o más."),
        ]
    }
}

# ------------------------------------------------------------------
# Errores intencionales a inyectar
# ------------------------------------------------------------------
EMPTY_RESPONSES = [
    ("¿Cómo desinstalo el software completamente?", ""),
    ("¿Hay versión para Linux?", ""),
    ("¿Cómo exporto mis datos?", None),
]

SHORT_RESPONSES = [
    ("¿Qué navegador recomiendas?", "Chrome.", "configuracion"),
    ("¿Puedo usar sin internet?", "Sí.", "configuracion"),
    ("¿Tienen soporte 24/7?", "No.", "licencias"),
]

SPECIAL_CHARS = [
    ("¿Cómo uso la función de búsqueda avanzada\x00?",
     "La búsqueda avanzada permite filtrar por fecha\x00, tipo y etiquetas usando operadores booleanos.",
     "configuracion"),
    ("Error código 0x80070005\r\n¿cómo lo soluciono?",
     "Este código indica permisos insuficientes\r\n. Ejecuta como administrador.",
     "errores"),
]

rows = []

# Generar pares válidos con distribución desbalanceada
for category, data in CATEGORIES.items():
    qa_pool = data["qa_pairs"]
    for i in range(data["count"]):
        qa = qa_pool[i % len(qa_pool)]
        # Agregar variación mínima para simular datos reales
        suffix = f" (caso #{i+1})" if i >= len(qa_pool) else ""
        rows.append({
            "id": len(rows) + 1,
            "category": category,
            "question": qa[0] + suffix,
            "answer": qa[1]
        })

# Inyectar duplicados exactos (10% del total)
duplicates = random.sample(rows[:100], 30)
for d in duplicates:
    rows.append({
        "id": len(rows) + 1,
        "category": d["category"],
        "question": d["question"],
        "answer": d["answer"]
    })

# Inyectar respuestas vacías
for q, a in EMPTY_RESPONSES:
    rows.append({"id": len(rows)+1, "category": "instalacion", "question": q, "answer": a})

# Inyectar respuestas cortas
for q, a, cat in SHORT_RESPONSES:
    rows.append({"id": len(rows)+1, "category": cat, "question": q, "answer": a})

# Inyectar caracteres especiales
for q, a, cat in SPECIAL_CHARS:
    rows.append({"id": len(rows)+1, "category": cat, "question": q, "answer": a})

# Mezclar filas
random.shuffle(rows)
for i, row in enumerate(rows):
    row["id"] = i + 1

df = pd.DataFrame(rows)
df.to_csv("raw_qa_dataset.csv", index=False, encoding="utf-8", quoting=csv.QUOTE_ALL)
print(f"Dataset generado: {len(df)} filas → raw_qa_dataset.csv")
print(df["category"].value_counts())
```

2. Ejecuta el generador:

```bash
python generate_raw_dataset.py
```

#### Salida esperada

```
Dataset generado: 535 filas → raw_qa_dataset.csv
category
instalacion      233
configuracion    157
errores           83
licencias         43
rendimiento       32
dtype: int64
```

#### Verificación

```bash
# Verificar que el archivo fue creado y tiene el número correcto de filas
python -c "import pandas as pd; df = pd.read_csv('raw_qa_dataset.csv'); print(f'Filas: {len(df)}, Columnas: {list(df.columns)}')"
```

---

### Paso 2: Implementar `load_and_inspect()`

**Objetivo:** Crear el script principal `dataset_preprocessor.py` con la función de carga e inspección estadística del dataset crudo.

#### Instrucciones

1. Crea el archivo `dataset_preprocessor.py` con el siguiente contenido inicial (se irá completando en los pasos siguientes):

```python
# dataset_preprocessor.py
"""
Pipeline de preprocesamiento de datos para fine-tuning de OpenAI.
Transforma raw_qa_dataset.csv en train.jsonl y validation.jsonl validados.

Lección 7.1: Fine-Tuning vs. RAG
"""

import csv
import difflib
import json
import os
import re
import unicodedata
from dataclasses import dataclass, field
from datetime import datetime
from typing import List, Optional, Tuple

import jsonlines
import pandas as pd
import tiktoken

# ──────────────────────────────────────────────────────────────────
# CONSTANTES DE CONFIGURACIÓN
# ──────────────────────────────────────────────────────────────────
MAX_TOKENS_PER_EXAMPLE = 4096
MIN_WORDS_IN_ANSWER = 20
DEDUP_SIMILARITY_THRESHOLD = 0.85
MIN_EXAMPLES_PER_CATEGORY = 10
TRAIN_SPLIT_RATIO = 0.90
FINE_TUNING_MODEL = "gpt-4o-mini-2024-07-18"
BASE_MODEL_FOR_FEWSHOT = "gpt-4o"

# Precios por token (USD) — Actualiza según la página de OpenAI
# https://openai.com/pricing
PRICE_FINETUNING_INPUT_PER_1K = 0.003   # $0.003 / 1K tokens de entrenamiento
PRICE_GPT4O_INPUT_PER_1K = 0.005        # $0.005 / 1K tokens de entrada
PRICE_GPT4O_OUTPUT_PER_1K = 0.015       # $0.015 / 1K tokens de salida

SYSTEM_PROMPT = (
    "Eres un agente de soporte técnico especializado en software empresarial. "
    "Responde de manera clara, concisa y profesional. "
    "Si no tienes información suficiente para responder, indícalo explícitamente "
    "y sugiere contactar al equipo de soporte avanzado."
)


# ──────────────────────────────────────────────────────────────────
# DATACLASSES DE RESULTADOS
# ──────────────────────────────────────────────────────────────────
@dataclass
class ValidationReport:
    total_examples: int = 0
    valid_examples: int = 0
    invalid_examples: int = 0
    token_violations: List[dict] = field(default_factory=list)
    format_violations: List[dict] = field(default_factory=list)
    category_distribution: dict = field(default_factory=dict)
    category_violations: List[str] = field(default_factory=list)
    avg_tokens_per_example: float = 0.0
    max_tokens_found: int = 0
    train_count: int = 0
    validation_count: int = 0
    passed: bool = False


@dataclass
class CostAnalysis:
    total_training_tokens: int = 0
    epochs: int = 3
    finetuning_cost_usd: float = 0.0
    fewshot_cost_usd: float = 0.0
    expected_queries_per_month: int = 1000
    break_even_queries: int = 0
    recommendation: str = ""


# ──────────────────────────────────────────────────────────────────
# PASO 2: CARGA E INSPECCIÓN
# ──────────────────────────────────────────────────────────────────
def load_and_inspect(filepath: str) -> pd.DataFrame:
    """
    Carga el CSV y genera estadísticas descriptivas del dataset crudo.

    Args:
        filepath: Ruta al archivo CSV de entrada.

    Returns:
        DataFrame con los datos cargados.
    """
    print("\n" + "="*60)
    print("PASO 1: CARGA E INSPECCIÓN DEL DATASET")
    print("="*60)

    if not os.path.exists(filepath):
        raise FileNotFoundError(f"No se encontró el archivo: {filepath}")

    df = pd.read_csv(filepath, encoding="utf-8", keep_default_na=True)

    print(f"\n📂 Archivo cargado: {filepath}")
    print(f"   Filas totales:    {len(df)}")
    print(f"   Columnas:         {list(df.columns)}")

    # Estadísticas de valores nulos
    null_counts = df.isnull().sum()
    print(f"\n🔍 Valores nulos por columna:")
    for col, count in null_counts.items():
        status = "⚠️ " if count > 0 else "✅"
        print(f"   {status} {col}: {count}")

    # Distribución de categorías
    print(f"\n📊 Distribución por categoría:")
    cat_dist = df["category"].value_counts()
    for cat, count in cat_dist.items():
        bar = "█" * (count // 10)
        print(f"   {cat:<15} {count:>4}  {bar}")

    # Estadísticas de longitud
    df["question_len"] = df["question"].fillna("").apply(lambda x: len(str(x).split()))
    df["answer_len"] = df["answer"].fillna("").apply(lambda x: len(str(x).split()))

    print(f"\n📏 Estadísticas de longitud (palabras):")
    print(f"   Preguntas — min: {df['question_len'].min()}, "
          f"media: {df['question_len'].mean():.1f}, "
          f"max: {df['question_len'].max()}")
    print(f"   Respuestas — min: {df['answer_len'].min()}, "
          f"media: {df['answer_len'].mean():.1f}, "
          f"max: {df['answer_len'].max()}")

    # Detectar respuestas vacías o muy cortas
    empty_answers = df[df["answer"].isnull() | (df["answer"].fillna("").str.strip() == "")]
    short_answers = df[df["answer_len"] < MIN_WORDS_IN_ANSWER]
    print(f"\n⚠️  Problemas detectados (antes de limpiar):")
    print(f"   Respuestas vacías/nulas:     {len(empty_answers)}")
    print(f"   Respuestas < {MIN_WORDS_IN_ANSWER} palabras:      {len(short_answers)}")

    # Limpiar columnas auxiliares antes de retornar
    df.drop(columns=["question_len", "answer_len"], inplace=True)

    return df
```

2. Prueba la función de forma aislada:

```bash
python -c "
from dataset_preprocessor import load_and_inspect
df = load_and_inspect('raw_qa_dataset.csv')
print(f'\nDataFrame shape: {df.shape}')
"
```

#### Salida esperada

```
============================================================
PASO 1: CARGA E INSPECCIÓN DEL DATASET
============================================================

📂 Archivo cargado: raw_qa_dataset.csv
   Filas totales:    535
   Columnas:         ['id', 'category', 'question', 'answer']

🔍 Valores nulos por columna:
   ✅ id: 0
   ✅ category: 0
   ✅ question: 0
   ⚠️  answer: 1

📊 Distribución por categoría:
   instalacion       233  ███████████████████████
   configuracion     157  ███████████████
   errores            83  ████████
   licencias          43  ████
   rendimiento        32  ███

📏 Estadísticas de longitud (palabras):
   Preguntas — min: 3, media: 9.2, max: 18
   Respuestas — min: 0, media: 48.3, max: 89

⚠️  Problemas detectados (antes de limpiar):
   Respuestas vacías/nulas:     4
   Respuestas < 20 palabras:    9
```

#### Verificación

```bash
python -c "
from dataset_preprocessor import load_and_inspect
df = load_and_inspect('raw_qa_dataset.csv')
assert len(df) > 500, 'El dataset debe tener más de 500 filas'
assert 'category' in df.columns, 'Debe existir columna category'
print('✅ load_and_inspect: OK')
"
```

---

### Paso 3: Implementar `clean_dataset()`

**Objetivo:** Agregar al script la función de limpieza que elimina filas problemáticas, deduplica por similitud y normaliza el encoding.

#### Instrucciones

1. Agrega la siguiente función al final de `dataset_preprocessor.py`:

```python
# ──────────────────────────────────────────────────────────────────
# PASO 3: LIMPIEZA DEL DATASET
# ──────────────────────────────────────────────────────────────────
def _normalize_text(text: str) -> str:
    """Normaliza encoding UTF-8 y elimina caracteres de control."""
    # Normalizar a NFC (forma canónica compuesta)
    text = unicodedata.normalize("NFC", text)
    # Eliminar caracteres de control (excepto newline y tab)
    text = re.sub(r"[\x00-\x08\x0b\x0c\x0e-\x1f\x7f]", "", text)
    # Normalizar saltos de línea
    text = text.replace("\r\n", "\n").replace("\r", "\n")
    # Eliminar espacios múltiples
    text = re.sub(r" {2,}", " ", text)
    return text.strip()


def _is_near_duplicate(text_a: str, text_b: str, threshold: float) -> bool:
    """Retorna True si la similitud entre dos textos supera el umbral."""
    ratio = difflib.SequenceMatcher(None, text_a.lower(), text_b.lower()).ratio()
    return ratio >= threshold


def clean_dataset(df: pd.DataFrame) -> pd.DataFrame:
    """
    Limpia el dataset aplicando:
    1. Eliminación de filas con campos vacíos o nulos.
    2. Normalización de encoding UTF-8.
    3. Filtrado de respuestas con menos de MIN_WORDS_IN_ANSWER palabras.
    4. Deduplicación por similitud (difflib, umbral DEDUP_SIMILARITY_THRESHOLD).

    Args:
        df: DataFrame crudo cargado con load_and_inspect().

    Returns:
        DataFrame limpio y deduplicado.
    """
    print("\n" + "="*60)
    print("PASO 2: LIMPIEZA DEL DATASET")
    print("="*60)

    initial_count = len(df)
    df = df.copy()

    # ── 1. Eliminar filas con campos vacíos o nulos ──────────────
    df["question"] = df["question"].fillna("")
    df["answer"] = df["answer"].fillna("")
    before = len(df)
    df = df[
        (df["question"].str.strip() != "") &
        (df["answer"].str.strip() != "")
    ]
    removed_empty = before - len(df)
    print(f"\n🗑️  Filas con campos vacíos eliminadas:  {removed_empty}")

    # ── 2. Normalizar encoding UTF-8 ────────────────────────────
    df["question"] = df["question"].apply(_normalize_text)
    df["answer"] = df["answer"].apply(_normalize_text)
    print(f"✅ Normalización UTF-8 aplicada a {len(df)} filas")

    # ── 3. Filtrar respuestas muy cortas ────────────────────────
    before = len(df)
    df["answer_word_count"] = df["answer"].apply(lambda x: len(x.split()))
    df = df[df["answer_word_count"] >= MIN_WORDS_IN_ANSWER]
    removed_short = before - len(df)
    df.drop(columns=["answer_word_count"], inplace=True)
    print(f"🗑️  Respuestas < {MIN_WORDS_IN_ANSWER} palabras eliminadas: {removed_short}")

    # ── 4. Deduplicación por similitud ──────────────────────────
    print(f"\n🔄 Deduplicando con umbral de similitud {DEDUP_SIMILARITY_THRESHOLD}...")
    df = df.reset_index(drop=True)
    questions = df["question"].tolist()
    to_remove = set()

    for i in range(len(questions)):
        if i in to_remove:
            continue
        for j in range(i + 1, len(questions)):
            if j in to_remove:
                continue
            if _is_near_duplicate(questions[i], questions[j], DEDUP_SIMILARITY_THRESHOLD):
                to_remove.add(j)  # Conservar el primero (índice i)

    before_dedup = len(df)
    df = df.drop(index=list(to_remove)).reset_index(drop=True)
    removed_dupes = before_dedup - len(df)
    print(f"🗑️  Duplicados/casi-duplicados eliminados: {removed_dupes}")

    # ── Resumen ──────────────────────────────────────────────────
    final_count = len(df)
    print(f"\n📊 Resumen de limpieza:")
    print(f"   Filas iniciales:  {initial_count}")
    print(f"   Filas finales:    {final_count}")
    print(f"   Total eliminadas: {initial_count - final_count} "
          f"({(initial_count - final_count)/initial_count*100:.1f}%)")
    print(f"\n📊 Distribución post-limpieza por categoría:")
    for cat, count in df["category"].value_counts().items():
        print(f"   {cat:<15} {count:>4}")

    return df
```

2. Prueba la función:

```bash
python -c "
from dataset_preprocessor import load_and_inspect, clean_dataset
df = load_and_inspect('raw_qa_dataset.csv')
df_clean = clean_dataset(df)
print(f'\nShape limpio: {df_clean.shape}')
assert len(df_clean) < 535, 'Debe haber eliminado filas'
print('✅ clean_dataset: OK')
"
```

#### Salida esperada

```
============================================================
PASO 2: LIMPIEZA DEL DATASET
============================================================

🗑️  Filas con campos vacíos eliminadas:  4
✅ Normalización UTF-8 aplicada a 531 filas
🗑️  Respuestas < 20 palabras eliminadas: 5

🔄 Deduplicando con umbral de similitud 0.85...
🗑️  Duplicados/casi-duplicados eliminados: 47

📊 Resumen de limpieza:
   Filas iniciales:  535
   Filas finales:    479
   Total eliminadas: 56 (10.5%)
```

#### Verificación

```bash
python -c "
from dataset_preprocessor import load_and_inspect, clean_dataset
df = load_and_inspect('raw_qa_dataset.csv')
df_clean = clean_dataset(df)
# No debe haber respuestas vacías
assert df_clean['answer'].str.strip().eq('').sum() == 0, 'No deben quedar respuestas vacías'
# No debe haber respuestas muy cortas
word_counts = df_clean['answer'].apply(lambda x: len(x.split()))
assert word_counts.min() >= 20, 'Todas las respuestas deben tener >= 20 palabras'
print('✅ Verificación de clean_dataset: PASÓ')
"
```

---

### Paso 4: Implementar `convert_to_jsonl()` y `validate_dataset()`

**Objetivo:** Convertir el DataFrame limpio al formato de mensajes de OpenAI y validar cada ejemplo contra los criterios técnicos requeridos.

#### Instrucciones

1. Agrega las siguientes funciones al final de `dataset_preprocessor.py`:

```python
# ──────────────────────────────────────────────────────────────────
# PASO 4A: CONVERSIÓN A FORMATO JSONL
# ──────────────────────────────────────────────────────────────────
def convert_to_jsonl(df: pd.DataFrame, system_prompt: str) -> List[dict]:
    """
    Convierte cada fila del DataFrame al formato de mensajes de OpenAI:
    {"messages": [
        {"role": "system",    "content": "..."},
        {"role": "user",      "content": "..."},
        {"role": "assistant", "content": "..."}
    ]}

    Args:
        df:            DataFrame limpio con columnas question y answer.
        system_prompt: Instrucción de sistema para todos los ejemplos.

    Returns:
        Lista de diccionarios en formato de fine-tuning de OpenAI.
    """
    print("\n" + "="*60)
    print("PASO 3: CONVERSIÓN A FORMATO JSONL")
    print("="*60)

    examples = []
    for _, row in df.iterrows():
        example = {
            "messages": [
                {"role": "system",    "content": system_prompt},
                {"role": "user",      "content": str(row["question"])},
                {"role": "assistant", "content": str(row["answer"])}
            ],
            # Metadato auxiliar (se elimina antes de exportar)
            "_category": str(row.get("category", "unknown"))
        }
        examples.append(example)

    print(f"✅ {len(examples)} ejemplos convertidos al formato de mensajes OpenAI")
    print(f"   Estructura de un ejemplo:")
    sample = examples[0]
    for msg in sample["messages"]:
        preview = msg["content"][:60].replace("\n", " ")
        print(f"   [{msg['role']:>10}] {preview}...")

    return examples


# ──────────────────────────────────────────────────────────────────
# PASO 4B: VALIDACIÓN DEL DATASET
# ──────────────────────────────────────────────────────────────────
def _count_tokens(messages: List[dict], model: str = FINE_TUNING_MODEL) -> int:
    """Cuenta tokens en una lista de mensajes usando tiktoken."""
    try:
        enc = tiktoken.encoding_for_model(model)
    except KeyError:
        enc = tiktoken.get_encoding("cl100k_base")

    # Fórmula de OpenAI para contar tokens en mensajes de chat
    num_tokens = 3  # Overhead por conversación
    for message in messages:
        num_tokens += 4  # Overhead por mensaje
        for key, value in message.items():
            num_tokens += len(enc.encode(str(value)))
    num_tokens += 3  # Overhead de respuesta del asistente
    return num_tokens


def validate_dataset(examples: List[dict]) -> ValidationReport:
    """
    Valida cada ejemplo contra los criterios técnicos de OpenAI:
    - Formato correcto de mensajes (system + user + assistant)
    - Máximo MAX_TOKENS_PER_EXAMPLE tokens por ejemplo
    - Mínimo MIN_EXAMPLES_PER_CATEGORY por categoría
    - Split train/validation 90/10

    Args:
        examples: Lista de ejemplos en formato de mensajes OpenAI.

    Returns:
        ValidationReport con estadísticas detalladas.
    """
    print("\n" + "="*60)
    print("PASO 4: VALIDACIÓN DEL DATASET")
    print("="*60)

    report = ValidationReport()
    report.total_examples = len(examples)
    token_counts = []
    category_counts = {}

    required_roles = {"system", "user", "assistant"}

    for idx, example in enumerate(examples):
        is_valid = True
        messages = example.get("messages", [])
        category = example.get("_category", "unknown")

        # ── Validar formato de mensajes ──────────────────────────
        actual_roles = {m.get("role") for m in messages}
        if not required_roles.issubset(actual_roles):
            report.format_violations.append({
                "index": idx,
                "issue": f"Roles faltantes: {required_roles - actual_roles}"
            })
            is_valid = False

        for msg in messages:
            if not msg.get("content", "").strip():
                report.format_violations.append({
                    "index": idx,
                    "issue": f"Contenido vacío en rol '{msg.get('role')}'"
                })
                is_valid = False

        # ── Validar conteo de tokens ─────────────────────────────
        token_count = _count_tokens(messages)
        token_counts.append(token_count)

        if token_count > MAX_TOKENS_PER_EXAMPLE:
            report.token_violations.append({
                "index": idx,
                "tokens": token_count,
                "limit": MAX_TOKENS_PER_EXAMPLE
            })
            is_valid = False

        # ── Conteo por categoría ─────────────────────────────────
        category_counts[category] = category_counts.get(category, 0) + 1

        if is_valid:
            report.valid_examples += 1
        else:
            report.invalid_examples += 1

    # ── Verificar mínimo por categoría ──────────────────────────
    report.category_distribution = category_counts
    for cat, count in category_counts.items():
        if count < MIN_EXAMPLES_PER_CATEGORY:
            report.category_violations.append(
                f"Categoría '{cat}': {count} ejemplos (mínimo: {MIN_EXAMPLES_PER_CATEGORY})"
            )

    # ── Estadísticas de tokens ───────────────────────────────────
    if token_counts:
        report.avg_tokens_per_example = sum(token_counts) / len(token_counts)
        report.max_tokens_found = max(token_counts)

    # ── Split train/validation ───────────────────────────────────
    report.train_count = int(report.valid_examples * TRAIN_SPLIT_RATIO)
    report.validation_count = report.valid_examples - report.train_count

    # ── Determinar si el dataset pasó la validación ──────────────
    report.passed = (
        report.invalid_examples == 0 and
        len(report.category_violations) == 0 and
        report.valid_examples >= 50
    )

    # ── Imprimir resumen ─────────────────────────────────────────
    status = "✅ PASÓ" if report.passed else "⚠️  REQUIERE ATENCIÓN"
    print(f"\n{status} — Validación del dataset")
    print(f"   Total ejemplos:    {report.total_examples}")
    print(f"   Válidos:           {report.valid_examples}")
    print(f"   Inválidos:         {report.invalid_examples}")
    print(f"   Tokens promedio:   {report.avg_tokens_per_example:.0f}")
    print(f"   Tokens máximo:     {report.max_tokens_found}")
    print(f"   Violaciones token: {len(report.token_violations)}")
    print(f"   Violaciones fmt:   {len(report.format_violations)}")
    print(f"\n   Split entrenamiento: {report.train_count} ejemplos")
    print(f"   Split validación:    {report.validation_count} ejemplos")

    if report.category_violations:
        print(f"\n⚠️  Categorías con pocos ejemplos:")
        for v in report.category_violations:
            print(f"   • {v}")
    else:
        print(f"\n✅ Todas las categorías tienen ≥ {MIN_EXAMPLES_PER_CATEGORY} ejemplos")

    return report
```

2. Prueba las funciones:

```bash
python -c "
from dataset_preprocessor import load_and_inspect, clean_dataset, convert_to_jsonl, validate_dataset, SYSTEM_PROMPT
df = load_and_inspect('raw_qa_dataset.csv')
df_clean = clean_dataset(df)
examples = convert_to_jsonl(df_clean, SYSTEM_PROMPT)
report = validate_dataset(examples)
print(f'\nPasó validación: {report.passed}')
"
```

#### Salida esperada (fragmento)

```
============================================================
PASO 4: VALIDACIÓN DEL DATASET
============================================================

✅ PASÓ — Validación del dataset
   Total ejemplos:    479
   Válidos:           479
   Inválidos:         0
   Tokens promedio:   198
   Tokens máximo:     312
   Violaciones token: 0
   Violaciones fmt:   0

   Split entrenamiento: 431 ejemplos
   Split validación:    48 ejemplos

✅ Todas las categorías tienen ≥ 10 ejemplos
```

#### Verificación

```bash
python -c "
from dataset_preprocessor import *
df = load_and_inspect('raw_qa_dataset.csv')
df_clean = clean_dataset(df)
examples = convert_to_jsonl(df_clean, SYSTEM_PROMPT)
report = validate_dataset(examples)
assert report.total_examples > 0
assert report.train_count > report.validation_count
assert report.train_count + report.validation_count == report.valid_examples
print('✅ Validación de validate_dataset: OK')
"
```

---

### Paso 5: Implementar `calculate_finetuning_cost()` y la exportación

**Objetivo:** Calcular el análisis de costo comparativo entre fine-tuning y few-shot prompting, y exportar los archivos JSONL finales.

#### Instrucciones

1. Agrega las siguientes funciones al final de `dataset_preprocessor.py`:

```python
# ──────────────────────────────────────────────────────────────────
# PASO 5: ANÁLISIS DE COSTO
# ──────────────────────────────────────────────────────────────────
def calculate_finetuning_cost(
    examples: List[dict],
    epochs: int = 3,
    expected_queries_per_month: int = 1000
) -> CostAnalysis:
    """
    Calcula el costo estimado de fine-tuning y lo compara con
    el costo equivalente de few-shot prompting con el modelo base.

    Fórmula fine-tuning:
        costo = total_tokens_entrenamiento * epochs * precio_por_1k_tokens / 1000

    Fórmula few-shot (5 ejemplos en el prompt):
        tokens_por_consulta ≈ tokens_sistema + 5*tokens_ejemplo + tokens_pregunta + tokens_respuesta
        costo_mensual = consultas * tokens_entrada * precio_input + consultas * tokens_salida * precio_output

    Args:
        examples:                   Lista de ejemplos en formato de mensajes.
        epochs:                     Número de épocas de entrenamiento (default: 3).
        expected_queries_per_month: Consultas esperadas al mes para comparación.

    Returns:
        CostAnalysis con todos los campos calculados.
    """
    print("\n" + "="*60)
    print("PASO 5: ANÁLISIS DE COSTO")
    print("="*60)

    analysis = CostAnalysis(epochs=epochs, expected_queries_per_month=expected_queries_per_month)

    # ── Tokens totales de entrenamiento ─────────────────────────
    total_tokens = sum(
        _count_tokens(ex["messages"]) for ex in examples
    )
    analysis.total_training_tokens = total_tokens

    # ── Costo de fine-tuning ─────────────────────────────────────
    # Precio: $0.003 por 1K tokens de entrenamiento (gpt-4o-mini)
    analysis.finetuning_cost_usd = (
        total_tokens * epochs * PRICE_FINETUNING_INPUT_PER_1K / 1000
    )

    # ── Costo de few-shot prompting ──────────────────────────────
    # Estimación: 5 ejemplos en el prompt ≈ 5 * avg_tokens_por_ejemplo
    avg_tokens = total_tokens / len(examples) if examples else 0
    fewshot_context_tokens = 5 * avg_tokens + 150  # 150 tokens para system + pregunta
    avg_response_tokens = avg_tokens * 0.6          # Respuesta ≈ 60% del ejemplo

    monthly_input_cost = (
        expected_queries_per_month * fewshot_context_tokens *
        PRICE_GPT4O_INPUT_PER_1K / 1000
    )
    monthly_output_cost = (
        expected_queries_per_month * avg_response_tokens *
        PRICE_GPT4O_OUTPUT_PER_1K / 1000
    )
    analysis.fewshot_cost_usd = monthly_input_cost + monthly_output_cost

    # ── Punto de equilibrio (break-even) ────────────────────────
    # ¿Cuántas consultas mensuales hacen que fine-tuning sea más barato?
    cost_per_query_fewshot = analysis.fewshot_cost_usd / expected_queries_per_month
    if cost_per_query_fewshot > 0:
        analysis.break_even_queries = int(
            analysis.finetuning_cost_usd / cost_per_query_fewshot
        )

    # ── Recomendación ────────────────────────────────────────────
    if analysis.finetuning_cost_usd < analysis.fewshot_cost_usd * 3:
        analysis.recommendation = (
            "✅ FINE-TUNING RECOMENDADO: El costo de entrenamiento se amortiza "
            f"en aproximadamente {analysis.break_even_queries:,} consultas mensuales. "
            "Adicionalmente, el modelo fine-tuned tendrá menor latencia al no requerir "
            "el contexto de 5 ejemplos en cada llamada."
        )
    else:
        analysis.recommendation = (
            "⚠️  EVALÚA RAG O FEW-SHOT: El costo de fine-tuning es significativamente "
            "mayor que el few-shot prompting para el volumen de consultas esperado. "
            "Considera RAG si el conocimiento cambia frecuentemente, o few-shot "
            "si el volumen mensual es bajo."
        )

    # ── Imprimir análisis ────────────────────────────────────────
    print(f"\n💰 Análisis de Costo (modelo: {FINE_TUNING_MODEL})")
    print(f"\n   FINE-TUNING:")
    print(f"   • Tokens totales de entrenamiento: {total_tokens:,}")
    print(f"   • Épocas:                          {epochs}")
    print(f"   • Costo de entrenamiento:          ${analysis.finetuning_cost_usd:.4f} USD")
    print(f"\n   FEW-SHOT PROMPTING (base: {BASE_MODEL_FOR_FEWSHOT}):")
    print(f"   • Tokens de entrada por consulta:  {fewshot_context_tokens:.0f}")
    print(f"   • Tokens de salida por consulta:   {avg_response_tokens:.0f}")
    print(f"   • Costo mensual ({expected_queries_per_month:,} consultas): "
          f"${analysis.fewshot_cost_usd:.4f} USD")
    print(f"\n   PUNTO DE EQUILIBRIO:")
    print(f"   • Fine-tuning se amortiza en:      {analysis.break_even_queries:,} consultas")
    print(f"\n   RECOMENDACIÓN:")
    print(f"   {analysis.recommendation}")

    return analysis


# ──────────────────────────────────────────────────────────────────
# PASO 6: EXPORTACIÓN DE ARCHIVOS
# ──────────────────────────────────────────────────────────────────
def export_datasets(
    examples: List[dict],
    report: ValidationReport,
    train_path: str = "train.jsonl",
    validation_path: str = "validation.jsonl"
) -> Tuple[int, int]:
    """
    Exporta los ejemplos válidos en archivos JSONL para fine-tuning.
    Elimina el campo auxiliar '_category' antes de exportar.

    Args:
        examples:        Lista completa de ejemplos.
        report:          ValidationReport con conteos de split.
        train_path:      Ruta de salida para entrenamiento.
        validation_path: Ruta de salida para validación.

    Returns:
        Tupla (train_count, validation_count).
    """
    print("\n" + "="*60)
    print("PASO 6: EXPORTACIÓN DE ARCHIVOS JSONL")
    print("="*60)

    # Solo exportar ejemplos válidos y limpiar metadatos auxiliares
    valid_examples = []
    for ex in examples:
        clean_ex = {"messages": ex["messages"]}
        valid_examples.append(clean_ex)

    # Dividir en train y validation
    train_examples = valid_examples[:report.train_count]
    val_examples = valid_examples[report.train_count:]

    # Exportar train.jsonl
    with jsonlines.open(train_path, mode="w") as writer:
        writer.write_all(train_examples)

    # Exportar validation.jsonl
    with jsonlines.open(validation_path, mode="w") as writer:
        writer.write_all(val_examples)

    print(f"\n✅ Archivos exportados:")
    print(f"   📄 {train_path}:      {len(train_examples)} ejemplos")
    print(f"   📄 {validation_path}: {len(val_examples)} ejemplos")

    # Verificar que los archivos son JSONL válidos
    for path in [train_path, validation_path]:
        with jsonlines.open(path) as reader:
            count = sum(1 for _ in reader)
        print(f"   ✅ {path} verificado: {count} líneas JSONL válidas")

    return len(train_examples), len(val_examples)
```

2. Prueba el análisis de costo y la exportación:

```bash
python -c "
from dataset_preprocessor import *
df = load_and_inspect('raw_qa_dataset.csv')
df_clean = clean_dataset(df)
examples = convert_to_jsonl(df_clean, SYSTEM_PROMPT)
report = validate_dataset(examples)
cost = calculate_finetuning_cost(examples, epochs=3, expected_queries_per_month=1000)
train_n, val_n = export_datasets(examples, report)
print(f'Exportados: {train_n} train, {val_n} validation')
"
```

#### Verificación

```bash
# Verificar que los archivos JSONL son válidos
python -c "
import jsonlines
for path in ['train.jsonl', 'validation.jsonl']:
    with jsonlines.open(path) as reader:
        items = list(reader)
    # Verificar estructura de cada ejemplo
    for item in items:
        assert 'messages' in item, f'Falta campo messages en {path}'
        roles = [m['role'] for m in item['messages']]
        assert 'system' in roles and 'user' in roles and 'assistant' in roles
    print(f'✅ {path}: {len(items)} ejemplos válidos')
"
```

---

### Paso 6: Generar el Reporte de Calidad y el Script Principal

**Objetivo:** Crear la función de generación del reporte Markdown y el punto de entrada `main()` que orquesta todo el pipeline.

#### Instrucciones

1. Agrega la función de reporte y el `main()` al final de `dataset_preprocessor.py`:

```python
# ──────────────────────────────────────────────────────────────────
# PASO 7: GENERACIÓN DEL REPORTE MARKDOWN
# ──────────────────────────────────────────────────────────────────
def generate_report(
    raw_count: int,
    clean_count: int,
    report: ValidationReport,
    cost: CostAnalysis,
    output_path: str = "preprocessing_report.md"
) -> None:
    """
    Genera un reporte técnico en formato Markdown con todas las
    métricas del pipeline de preprocesamiento.
    """
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    status_emoji = "✅" if report.passed else "⚠️"

    md = f"""# Reporte de Preprocesamiento de Dataset para Fine-Tuning

**Generado:** {timestamp}
**Estado:** {status_emoji} {"DATASET LISTO PARA FINE-TUNING" if report.passed else "REQUIERE REVISIÓN"}

---

## 1. Resumen Ejecutivo

| Métrica                        | Valor                        |
|-------------------------------|------------------------------|
| Filas en dataset crudo         | {raw_count}                  |
| Filas después de limpieza      | {clean_count}                |
| Reducción por limpieza         | {(raw_count - clean_count) / raw_count * 100:.1f}%  |
| Ejemplos válidos para training | {report.valid_examples}      |
| Split entrenamiento            | {report.train_count} ({TRAIN_SPLIT_RATIO*100:.0f}%) |
| Split validación               | {report.validation_count} ({(1-TRAIN_SPLIT_RATIO)*100:.0f}%) |

---

## 2. Estadísticas de Tokens

| Métrica                        | Valor                        |
|-------------------------------|------------------------------|
| Tokens promedio por ejemplo    | {report.avg_tokens_per_example:.0f}  |
| Tokens máximo encontrado       | {report.max_tokens_found}    |
| Límite máximo (OpenAI)         | {MAX_TOKENS_PER_EXAMPLE}     |
| Violaciones de token           | {len(report.token_violations)} |
| Violaciones de formato         | {len(report.format_violations)} |

---

## 3. Distribución por Categoría

| Categoría       | Ejemplos | Estado                          |
|----------------|----------|---------------------------------|
"""
    for cat, count in sorted(report.category_distribution.items()):
        status = "✅" if count >= MIN_EXAMPLES_PER_CATEGORY else "⚠️ Insuficiente"
        md += f"| {cat:<15} | {count:>8} | {status:<31} |\n"

    md += f"""
---

## 4. Análisis de Costo: Fine-Tuning vs. Few-Shot Prompting

### Fine-Tuning ({FINE_TUNING_MODEL})

| Parámetro                          | Valor                               |
|-----------------------------------|-------------------------------------|
| Tokens totales de entrenamiento    | {cost.total_training_tokens:,}      |
| Épocas                             | {cost.epochs}                       |
| Precio por 1K tokens               | ${PRICE_FINETUNING_INPUT_PER_1K:.4f} USD |
| **Costo total de entrenamiento**   | **${cost.finetuning_cost_usd:.4f} USD** |

### Few-Shot Prompting ({BASE_MODEL_FOR_FEWSHOT})

| Parámetro                          | Valor                               |
|-----------------------------------|-------------------------------------|
| Consultas esperadas/mes            | {cost.expected_queries_per_month:,} |
| Precio input por 1K tokens         | ${PRICE_GPT4O_INPUT_PER_1K:.4f} USD |
| Precio output por 1K tokens        | ${PRICE_GPT4O_OUTPUT_PER_1K:.4f} USD |
| **Costo mensual estimado**         | **${cost.fewshot_cost_usd:.4f} USD** |

### Punto de Equilibrio

El costo del fine-tuning se amortiza en aproximadamente **{cost.break_even_queries:,} consultas**.

### Recomendación

{cost.recommendation}

---

## 5. Archivos Generados

| Archivo                  | Descripción                                |
|-------------------------|--------------------------------------------|
| `train.jsonl`           | {report.train_count} ejemplos de entrenamiento (JSONL) |
| `validation.jsonl`      | {report.validation_count} ejemplos de validación (JSONL) |
| `preprocessing_report.md` | Este reporte                             |

---

## 6. Próximos Pasos

1. **Revisar `validation.jsonl`** manualmente para confirmar calidad de las respuestas.
2. **Subir el dataset** a la API de OpenAI usando `openai files create`.
3. **Iniciar el fine-tuning** con `openai fine_tuning.jobs.create`.
4. **Monitorear métricas** de training loss y validation loss durante el entrenamiento.
5. **Evaluar el modelo fine-tuned** con un conjunto de prueba independiente.

---

*Reporte generado por `dataset_preprocessor.py` — Lab 07-00-01*
"""

    with open(output_path, "w", encoding="utf-8") as f:
        f.write(md)

    print(f"\n✅ Reporte generado: {output_path}")


# ──────────────────────────────────────────────────────────────────
# PUNTO DE ENTRADA PRINCIPAL
# ──────────────────────────────────────────────────────────────────
def main():
    """Orquesta el pipeline completo de preprocesamiento."""
    print("\n" + "🚀 " + "="*56)
    print("   PIPELINE DE PREPROCESAMIENTO PARA FINE-TUNING")
    print("   Lección 7.1: Fine-Tuning vs. RAG")
    print("🚀 " + "="*56)

    INPUT_FILE = "raw_qa_dataset.csv"

    # 1. Cargar e inspeccionar
    df_raw = load_and_inspect(INPUT_FILE)
    raw_count = len(df_raw)

    # 2. Limpiar
    df_clean = clean_dataset(df_raw)
    clean_count = len(df_clean)

    # 3. Convertir a formato JSONL
    examples = convert_to_jsonl(df_clean, SYSTEM_PROMPT)

    # 4. Validar
    report = validate_dataset(examples)

    # 5. Análisis de costo
    cost = calculate_finetuning_cost(
        examples,
        epochs=3,
        expected_queries_per_month=1000
    )

    # 6. Exportar archivos JSONL
    export_datasets(examples, report)

    # 7. Generar reporte
    generate_report(raw_count, clean_count, report, cost)

    print("\n" + "="*60)
    print("✅ PIPELINE COMPLETADO EXITOSAMENTE")
    print("="*60)
    print(f"\nArchivos generados:")
    print(f"  📄 train.jsonl")
    print(f"  📄 validation.jsonl")
    print(f"  📄 preprocessing_report.md")


if __name__ == "__main__":
    main()
```

2. Ejecuta el pipeline completo:

```bash
python dataset_preprocessor.py
```

#### Salida esperada (resumen final)

```
🚀 ========================================================
   PIPELINE DE PREPROCESAMIENTO PARA FINE-TUNING
   Lección 7.1: Fine-Tuning vs. RAG
🚀 ========================================================

[... salidas de pasos anteriores ...]

============================================================
PASO 5: ANÁLISIS DE COSTO
============================================================

💰 Análisis de Costo (modelo: gpt-4o-mini-2024-07-18)

   FINE-TUNING:
   • Tokens totales de entrenamiento: 94,762
   • Épocas:                          3
   • Costo de entrenamiento:          $0.8529 USD

   FEW-SHOT PROMPTING (base: gpt-4o):
   • Tokens de entrada por consulta:  1,140
   • Tokens de salida por consulta:   118
   • Costo mensual (1,000 consultas): $7.4700 USD

   PUNTO DE EQUILIBRIO:
   • Fine-tuning se amortiza en:      114 consultas

   RECOMENDACIÓN:
   ✅ FINE-TUNING RECOMENDADO: El costo de entrenamiento se amortiza
   en aproximadamente 114 consultas mensuales.

============================================================
✅ PIPELINE COMPLETADO EXITOSAMENTE
============================================================

Archivos generados:
  📄 train.jsonl
  📄 validation.jsonl
  📄 preprocessing_report.md
```

#### Verificación

```bash
# Verificar que los 3 archivos de salida existen y tienen contenido
python -c "
import os, jsonlines
for f in ['train.jsonl', 'validation.jsonl', 'preprocessing_report.md']:
    assert os.path.exists(f), f'Falta: {f}'
    size = os.path.getsize(f)
    print(f'✅ {f}: {size:,} bytes')

# Verificar estructura del primer ejemplo de train.jsonl
with jsonlines.open('train.jsonl') as r:
    first = next(iter(r))
    roles = [m['role'] for m in first['messages']]
    assert roles == ['system', 'user', 'assistant'], f'Roles incorrectos: {roles}'
print('✅ Estructura JSONL: correcta')
"
```

---

## 7. Validación y Pruebas

Ejecuta el siguiente script de validación integral para confirmar que todos los componentes del pipeline funcionan correctamente:

```bash
# validation_test.py
cat > validation_test.py << 'TESTEOF'
"""
Suite de validación integral para el Lab 07-00-01.
Ejecutar después de completar todos los pasos.
"""
import os
import json
import jsonlines
import pandas as pd

print("=" * 60)
print("SUITE DE VALIDACIÓN — Lab 07-00-01")
print("=" * 60)

tests_passed = 0
tests_failed = 0

def check(condition, name, detail=""):
    global tests_passed, tests_failed
    if condition:
        print(f"  ✅ PASS: {name}")
        tests_passed += 1
    else:
        print(f"  ❌ FAIL: {name} — {detail}")
        tests_failed += 1

# ── Test 1: Archivos de salida existen ──────────────────────────
print("\n[1] Archivos de salida")
for f in ["train.jsonl", "validation.jsonl", "preprocessing_report.md"]:
    check(os.path.exists(f), f"Existe {f}")

# ── Test 2: Estructura JSONL correcta ───────────────────────────
print("\n[2] Estructura JSONL")
for path in ["train.jsonl", "validation.jsonl"]:
    with jsonlines.open(path) as reader:
        items = list(reader)
    check(len(items) > 0, f"{path} no está vacío", f"items={len(items)}")
    for i, item in enumerate(items[:5]):  # Verificar primeros 5
        msgs = item.get("messages", [])
        roles = [m.get("role") for m in msgs]
        check(
            set(roles) == {"system", "user", "assistant"},
            f"{path}[{i}] tiene roles correctos",
            f"roles={roles}"
        )
        check(
            "_category" not in item,
            f"{path}[{i}] no tiene metadatos auxiliares"
        )

# ── Test 3: Ratio train/validation ──────────────────────────────
print("\n[3] Split train/validation")
with jsonlines.open("train.jsonl") as r: train_n = sum(1 for _ in r)
with jsonlines.open("validation.jsonl") as r: val_n = sum(1 for _ in r)
total = train_n + val_n
ratio = train_n / total if total > 0 else 0
check(0.88 <= ratio <= 0.92, f"Ratio train ~90%", f"ratio={ratio:.2f}")
check(val_n >= 10, "Validación tiene al menos 10 ejemplos", f"val_n={val_n}")

# ── Test 4: Contenido del reporte ───────────────────────────────
print("\n[4] Reporte Markdown")
with open("preprocessing_report.md", encoding="utf-8") as f:
    content = f.read()
for section in ["Resumen Ejecutivo", "Análisis de Costo", "Distribución por Categoría"]:
    check(section in content, f"Reporte contiene sección '{section}'")

# ── Test 5: Tokens dentro del límite ────────────────────────────
print("\n[5] Límite de tokens")
import tiktoken
enc = tiktoken.get_encoding("cl100k_base")
with jsonlines.open("train.jsonl") as reader:
    for i, item in enumerate(reader):
        total_tokens = sum(len(enc.encode(m["content"])) for m in item["messages"])
        check(
            total_tokens <= 4096,
            f"train[{i}] dentro del límite de tokens",
            f"tokens={total_tokens}"
        ) if i < 3 else None  # Verificar primeros 3

# ── Resumen ──────────────────────────────────────────────────────
print(f"\n{'='*60}")
print(f"RESULTADO: {tests_passed} PASS / {tests_failed} FAIL")
if tests_failed == 0:
    print("🎉 Todos los tests pasaron — Lab completado exitosamente")
else:
    print("⚠️  Revisa los tests fallidos antes de continuar")
print("=" * 60)
TESTEOF

python validation_test.py
```

**Resultado esperado:** todos los tests en estado `PASS`.

---

## 8. Resolución de Problemas

### Problema 1: `tiktoken` no puede encontrar el encoding para el modelo

**Síntomas:**
```
KeyError: 'gpt-4o-mini-2024-07-18'
```
El script falla al intentar contar tokens porque la versión instalada de `tiktoken` no reconoce el modelo especificado en `FINE_TUNING_MODEL`.

**Causa:**
La constante `FINE_TUNING_MODEL = "gpt-4o-mini-2024-07-18"` usa un nombre de modelo que versiones antiguas de `tiktoken` (< 0.6.0) no tienen en su registro interno.

**Solución:**
```bash
# 1. Actualizar tiktoken a la versión más reciente compatible
pip install tiktoken --upgrade

# 2. Si el problema persiste, la función _count_tokens() ya tiene
#    un fallback a cl100k_base. Verificar que el bloque try/except
#    esté presente en el código:
python -c "
import tiktoken
# Probar fallback manual
try:
    enc = tiktoken.encoding_for_model('gpt-4o-mini-2024-07-18')
except KeyError:
    enc = tiktoken.get_encoding('cl100k_base')
    print('Usando cl100k_base como fallback')
print(f'Encoding: {enc.name}')
"
```

---

### Problema 2: La deduplicación tarda demasiado tiempo con datasets grandes

**Síntomas:**
El script se queda "colgado" en el paso de deduplicación durante varios minutos cuando el dataset tiene más de 1,000 filas. La deduplicación O(n²) con `difflib.SequenceMatcher` se vuelve muy lenta.

**Causa:**
El algoritmo de deduplicación actual compara cada par de preguntas (complejidad O(n²)). Para 500 filas hay ~125,000 comparaciones; para 2,000 filas serían ~2,000,000 comparaciones, lo que puede tardar varios minutos.

**Solución:**
```python
# Reemplaza el bloque de deduplicación en clean_dataset() con esta
# versión optimizada que usa hashing para pre-filtrar candidatos:

from collections import defaultdict

def _get_shingles(text: str, k: int = 3) -> set:
    """Genera k-shingles de palabras para pre-filtrado rápido."""
    words = text.lower().split()
    return set(tuple(words[i:i+k]) for i in range(len(words)-k+1))

# Dentro de clean_dataset(), reemplaza el bucle de deduplicación:
print(f"\n🔄 Deduplicando (versión optimizada) con umbral {DEDUP_SIMILARITY_THRESHOLD}...")
df = df.reset_index(drop=True)
questions = df["question"].tolist()
to_remove = set()

# Pre-agrupar por shingles similares para reducir comparaciones
shingle_index = defaultdict(list)
for i, q in enumerate(questions):
    for shingle in _get_shingles(q):
        shingle_index[shingle].append(i)

# Solo comparar pares con al menos 1 shingle en común
candidate_pairs = set()
for indices in shingle_index.values():
    for a in range(len(indices)):
        for b in range(a+1, len(indices)):
            if indices[a] not in to_remove and indices[b] not in to_remove:
                candidate_pairs.add((min(indices[a], indices[b]),
                                     max(indices[a], indices[b])))

for i, j in candidate_pairs:
    if i not in to_remove and j not in to_remove:
        if _is_near_duplicate(questions[i], questions[j], DEDUP_SIMILARITY_THRESHOLD):
            to_remove.add(j)
```

---

## 9. Limpieza

Ejecuta los siguientes comandos para limpiar los archivos temporales generados durante el laboratorio:

```bash
# Opción 1: Limpiar solo archivos generados (conservar el script)
rm -f raw_qa_dataset.csv generate_raw_dataset.py validation_test.py
echo "✅ Archivos temporales eliminados"

# Opción 2: Conservar los archivos de salida para revisión
# (train.jsonl, validation.jsonl, preprocessing_report.md se conservan)
ls -lh *.jsonl *.md 2>/dev/null

# Opción 3: Limpieza completa del entorno virtual
deactivate
rm -rf venv_lab07/
echo "✅ Entorno virtual eliminado"

# Verificar que no hay API keys en el código antes de hacer commit
grep -r "sk-" . --include="*.py" && echo "⚠️ ALERTA: Posible API key encontrada" || echo "✅ Sin API keys en el código"
```

> **Nota de seguridad:** Verifica que el archivo `.gitignore` incluya `*.jsonl` si los datasets contienen información sensible de tu organización antes de hacer commit al repositorio.

---

## 10. Resumen

En este laboratorio construiste un pipeline completo de preprocesamiento de datos para fine-tuning que abarcó:

1. **Carga e inspección** (`load_and_inspect`): análisis estadístico del dataset crudo con detección automática de problemas.
2. **Limpieza** (`clean_dataset`): eliminación de nulos, normalización UTF-8, filtrado por longitud mínima y deduplicación por similitud con `difflib.SequenceMatcher` (umbral 0.85).
3. **Conversión** (`convert_to_jsonl`): transformación al formato de mensajes `[system, user, assistant]` requerido por la API de fine-tuning de OpenAI.
4. **Validación** (`validate_dataset`): verificación de límites de tokens con `tiktoken`, formato de mensajes, distribución mínima por categoría y split 90/10.
5. **Análisis de costo** (`calculate_finetuning_cost`): comparación cuantitativa entre fine-tuning y few-shot prompting, con cálculo del punto de equilibrio.
6. **Exportación**: archivos `train.jsonl` y `validation.jsonl` listos para subir a OpenAI, más un reporte técnico `preprocessing_report.md`.

Este pipeline conecta directamente con el marco conceptual de la **Lección 7.1**: el análisis de costo que implementaste en el Paso 5 es exactamente el tipo de decisión informada que distingue entre elegir Fine-Tuning, RAG o few-shot prompting. El dataset que generaste hoy es el insumo directo para el proceso de fine-tuning que se abordará en las próximas lecciones.

### Recursos Adicionales

- [Guía oficial de Fine-Tuning de OpenAI](https://platform.openai.com/docs/guides/fine-tuning)
- [Documentación de tiktoken](https://github.com/openai/tiktoken)
- [Formato de datos para fine-tuning — OpenAI Cookbook](https://cookbook.openai.com/examples/chat_finetuning_data_prep)
- [difflib — Documentación oficial de Python](https://docs.python.org/3/library/difflib.html)
- [Pandas 2.2 — Guía de usuario](https://pandas.pydata.org/docs/user_guide/index.html)

---
*Lab 07-00-01 — Módulo 7: Fine-Tuning vs. RAG — Nivel: Crear (Bloom)*
