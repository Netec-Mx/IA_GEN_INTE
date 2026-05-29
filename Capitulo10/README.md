# Crear un script que evalúe la fidelidad de las respuestas de un chatbot comparándolas contra un "Golden Dataset"

## Metadatos

| Campo            | Valor                                                                 |
|------------------|-----------------------------------------------------------------------|
| **Duración**     | 40 minutos                                                            |
| **Complejidad**  | Media                                                                 |
| **Nivel Bloom**  | Crear                                                                 |
| **Modalidad**    | Individual / Guiada                                                   |
| **Costo estimado** | ~$0.20–$0.50 USD (llamadas a GPT-4o para G-Eval sobre 20 ejemplos) |

---

## Descripción General

En este laboratorio construirás un framework completo de evaluación automatizada para chatbots llamado `chatbot_evaluator.py`. Partirás de un **Golden Dataset** de 20 pares pregunta-respuesta sobre historia de la computación, generarás respuestas con un modelo GPT, y las evaluarás usando tres familias de métricas: las clásicas de superposición de n-gramas (**ROUGE-1, ROUGE-2, ROUGE-L, BLEU**) y la moderna métrica basada en LLM-as-a-Judge (**G-Eval**). Al finalizar, producirás un reporte en Markdown y HTML que compara todas las métricas, identifica los mejores y peores casos, y analiza la correlación entre los enfoques léxico y semántico.

---

## Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Implementar un pipeline de evaluación que aplique ROUGE y BLEU usando `rouge-score` y `nltk` sobre un dataset estructurado en JSON.
- [ ] Diseñar y ejecutar evaluaciones G-Eval con GPT-4o como juez, interpretando tanto las puntuaciones numéricas como las justificaciones textuales.
- [ ] Analizar las diferencias entre métricas léxicas y semánticas a partir de casos concretos donde divergen.
- [ ] Generar un reporte de evaluación profesional en Markdown/HTML con tablas comparativas, promedios por categoría y análisis de casos extremos.

---

## Prerrequisitos

### Conocimiento previo
- Comprensión de métricas ROUGE, BLEU y G-Eval (cubierto en la lección 10.1).
- Familiaridad con `pandas` para análisis tabular.
- Experiencia con el SDK de OpenAI y gestión de `.env` (labs anteriores del curso).

### Acceso y credenciales
- `OPENAI_API_KEY` activa con acceso a `gpt-4o` (para G-Eval y generación de respuestas).
- Conexión a internet estable.
- Límite de gasto mensual configurado en la consola de OpenAI (recomendado: $5 USD máximo para todo el curso).

---

## Entorno del Laboratorio

### Hardware requerido

| Componente   | Mínimo                              |
|--------------|-------------------------------------|
| RAM          | 8 GB (16 GB recomendado)            |
| Disco        | 500 MB libres                       |
| CPU          | 4 núcleos                           |
| Red          | 10 Mbps estable                     |

### Software y librerías

| Librería / Herramienta | Versión       | Propósito                            |
|------------------------|---------------|--------------------------------------|
| Python                 | 3.11.x        | Entorno de ejecución                 |
| openai                 | 1.35.x        | Generación de respuestas y G-Eval    |
| rouge-score            | 0.1.2         | Cálculo de métricas ROUGE            |
| nltk                   | 3.8.x         | Cálculo de métricas BLEU             |
| pandas                 | 2.2.x         | Análisis tabular de resultados       |
| pydantic               | 2.7.x         | Modelos de datos tipados             |
| python-dotenv          | 1.0.x         | Gestión segura de credenciales       |
| jinja2                 | 3.1.x         | Generación de reporte HTML           |

### Configuración del entorno

**Paso 1 — Crear directorio y entorno virtual:**

```bash
mkdir lab10_evaluacion
cd lab10_evaluacion
python -m venv .venv

# Activar en macOS/Linux:
source .venv/bin/activate

# Activar en Windows (PowerShell):
.venv\Scripts\Activate.ps1
```

**Paso 2 — Crear `requirements.txt` e instalar dependencias:**

```bash
cat > requirements.txt << 'EOF'
openai==1.35.3
rouge-score==0.1.2
nltk==3.8.1
pandas==2.2.2
pydantic==2.7.4
python-dotenv==1.0.1
jinja2==3.1.4
EOF

pip install -r requirements.txt
```

**Paso 3 — Descargar recursos de NLTK:**

```bash
python -c "import nltk; nltk.download('punkt'); nltk.download('punkt_tab')"
```

**Paso 4 — Crear archivo `.env` y `.gitignore`:**

```bash
cat > .env << 'EOF'
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
OPENAI_MODEL_GENERATION=gpt-4o-mini
OPENAI_MODEL_JUDGE=gpt-4o
EOF

cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
*.pyo
reports/
EOF
```

> ⚠️ **Seguridad:** Nunca subas el archivo `.env` a un repositorio. Verifica que `.gitignore` lo incluya antes de hacer cualquier commit.

> 💡 **Control de costos:** Se usa `gpt-4o-mini` para generar las respuestas del chatbot (más económico) y `gpt-4o` solo para el rol de juez en G-Eval. Esto equilibra calidad de evaluación con costo.

---

## Instrucciones Paso a Paso

---

### Paso 1 — Crear el Golden Dataset

**Objetivo:** Diseñar un archivo `golden_dataset.json` con 20 pares pregunta-respuesta de referencia sobre historia de la computación, organizados por categoría y nivel de dificultad.

#### Instrucciones

1. Crea el archivo `golden_dataset.json` en el directorio `lab10_evaluacion/`:

```bash
cat > golden_dataset.json << 'ENDJSON'
[
  {
    "id": 1,
    "pregunta": "¿Quién es considerado el padre de la computación moderna?",
    "respuesta_referencia": "Alan Turing es considerado el padre de la computación moderna. Desarrolló la máquina de Turing en 1936, un modelo matemático abstracto que define los fundamentos teóricos de la computación.",
    "categoria": "factual",
    "nivel_dificultad": "easy"
  },
  {
    "id": 2,
    "pregunta": "¿Qué fue el proyecto ENIAC y cuándo se completó?",
    "respuesta_referencia": "ENIAC (Electronic Numerical Integrator and Computer) fue la primera computadora electrónica de propósito general. Fue completada en 1945 en la Universidad de Pensilvania por John Mauchly y J. Presper Eckert.",
    "categoria": "factual",
    "nivel_dificultad": "easy"
  },
  {
    "id": 3,
    "pregunta": "¿Cuál fue la contribución de Grace Hopper al desarrollo del software?",
    "respuesta_referencia": "Grace Hopper desarrolló el primer compilador de la historia, el A-0, en 1952. También lideró el desarrollo de COBOL, uno de los primeros lenguajes de programación de alto nivel orientado a negocios.",
    "categoria": "factual",
    "nivel_dificultad": "medium"
  },
  {
    "id": 4,
    "pregunta": "¿Qué es la Ley de Moore y cuál es su relevancia actual?",
    "respuesta_referencia": "La Ley de Moore, formulada por Gordon Moore en 1965, establece que el número de transistores en un microprocesador se duplica aproximadamente cada dos años. Actualmente enfrenta limitaciones físicas debido al tamaño atómico de los transistores, aunque sigue siendo una guía de referencia en la industria.",
    "categoria": "inferencia",
    "nivel_dificultad": "medium"
  },
  {
    "id": 5,
    "pregunta": "¿Cuál fue el papel de Ada Lovelace en la historia de la programación?",
    "respuesta_referencia": "Ada Lovelace es reconocida como la primera programadora de la historia. Trabajó con Charles Babbage en la Máquina Analítica y escribió el primer algoritmo diseñado para ser procesado por una máquina, en 1843.",
    "categoria": "factual",
    "nivel_dificultad": "easy"
  },
  {
    "id": 6,
    "pregunta": "¿Qué diferencia fundamental existe entre la arquitectura Von Neumann y la arquitectura Harvard?",
    "respuesta_referencia": "La arquitectura Von Neumann utiliza un único bus compartido para datos e instrucciones, lo que puede causar cuellos de botella. La arquitectura Harvard separa físicamente la memoria y los buses de datos e instrucciones, permitiendo accesos simultáneos y mayor velocidad en aplicaciones específicas como microcontroladores.",
    "categoria": "inferencia",
    "nivel_dificultad": "hard"
  },
  {
    "id": 7,
    "pregunta": "¿Qué fue ARPANET y cómo se relaciona con el Internet actual?",
    "respuesta_referencia": "ARPANET fue la primera red de computadoras de área amplia, desarrollada por el Departamento de Defensa de Estados Unidos en 1969. Es el precursor directo de Internet, ya que estableció los protocolos de comunicación en paquetes que evolucionaron hacia TCP/IP.",
    "categoria": "factual",
    "nivel_dificultad": "easy"
  },
  {
    "id": 8,
    "pregunta": "¿Qué impacto tuvo la creación del microprocesador Intel 4004 en la industria tecnológica?",
    "respuesta_referencia": "El Intel 4004, lanzado en 1971, fue el primer microprocesador comercial integrado en un solo chip. Su creación democratizó la computación al reducir drásticamente el tamaño y costo de los procesadores, sentando las bases para las computadoras personales y la revolución digital.",
    "categoria": "inferencia",
    "nivel_dificultad": "medium"
  },
  {
    "id": 9,
    "pregunta": "¿Cuáles fueron los principales lenguajes de programación de la década de 1950?",
    "respuesta_referencia": "Los principales lenguajes de programación de la década de 1950 fueron FORTRAN (1957), desarrollado por IBM para cálculos científicos, y COBOL (1959), orientado a aplicaciones de negocios. También destacó LISP (1958), diseñado para procesamiento de listas e inteligencia artificial.",
    "categoria": "factual",
    "nivel_dificultad": "medium"
  },
  {
    "id": 10,
    "pregunta": "¿Por qué el desarrollo del sistema operativo UNIX fue significativo para la computación moderna?",
    "respuesta_referencia": "UNIX, desarrollado en Bell Labs por Ken Thompson y Dennis Ritchie en 1969, fue significativo porque introdujo conceptos como la portabilidad entre hardware diferente, el diseño modular y la filosofía de herramientas pequeñas combinables. Influyó directamente en Linux, macOS y otros sistemas modernos.",
    "categoria": "inferencia",
    "nivel_dificultad": "hard"
  },
  {
    "id": 11,
    "pregunta": "¿Qué es el test de Turing y qué pretende medir?",
    "respuesta_referencia": "El test de Turing, propuesto por Alan Turing en 1950, es una prueba para evaluar si una máquina puede exhibir comportamiento inteligente indistinguible del humano. Un evaluador humano interactúa por texto con una máquina y un humano; si no puede distinguir cuál es cuál, la máquina supera el test.",
    "categoria": "factual",
    "nivel_dificultad": "easy"
  },
  {
    "id": 12,
    "pregunta": "¿Cuál fue la importancia del lenguaje C en el desarrollo del software de sistemas?",
    "respuesta_referencia": "El lenguaje C, creado por Dennis Ritchie en 1972, fue fundamental porque combinó la eficiencia del lenguaje ensamblador con la legibilidad de un lenguaje de alto nivel. Permitió reescribir UNIX en C, estableciendo el paradigma de escribir sistemas operativos en lenguajes de alto nivel.",
    "categoria": "inferencia",
    "nivel_dificultad": "medium"
  },
  {
    "id": 13,
    "pregunta": "¿Qué fue la 'burbuja puntocom' y cuáles fueron sus principales causas?",
    "respuesta_referencia": "La burbuja puntocom fue una burbuja especulativa entre 1995 y 2001 en torno a empresas de Internet. Sus principales causas fueron la sobrevaluación de startups sin modelos de negocio sostenibles, la especulación masiva de inversores y la creencia irracional en un crecimiento ilimitado del comercio electrónico.",
    "categoria": "inferencia",
    "nivel_dificultad": "hard"
  },
  {
    "id": 14,
    "pregunta": "¿Qué es el código abierto (open source) y cuál fue su impacto en la industria del software?",
    "respuesta_referencia": "El código abierto es un modelo de desarrollo donde el código fuente es público y puede ser modificado y distribuido libremente. Su impacto fue transformador: proyectos como Linux, Apache y Python demostraron que el software colaborativo podía competir y superar al software propietario en calidad y adopción.",
    "categoria": "resumen",
    "nivel_dificultad": "medium"
  },
  {
    "id": 15,
    "pregunta": "¿Cómo funcionan los transistores y por qué reemplazaron a los tubos de vacío?",
    "respuesta_referencia": "Los transistores son semiconductores que actúan como interruptores o amplificadores de señales eléctricas. Reemplazaron a los tubos de vacío porque son más pequeños, consumen menos energía, generan menos calor, son más confiables y tienen una vida útil mucho mayor.",
    "categoria": "factual",
    "nivel_dificultad": "medium"
  },
  {
    "id": 16,
    "pregunta": "¿Cuál fue la contribución de Tim Berners-Lee a la computación global?",
    "respuesta_referencia": "Tim Berners-Lee inventó la World Wide Web en 1989 mientras trabajaba en el CERN. Desarrolló el protocolo HTTP, el lenguaje HTML y el primer navegador web, creando la infraestructura que transformó Internet de una red académica en un sistema de información global accesible.",
    "categoria": "factual",
    "nivel_dificultad": "easy"
  },
  {
    "id": 17,
    "pregunta": "¿Qué lecciones dejó el fracaso del proyecto Multics para el diseño de sistemas operativos?",
    "respuesta_referencia": "El proyecto Multics, aunque influyente, fue excesivamente ambicioso y complejo, lo que dificultó su implementación práctica. Sus lecciones llevaron a Bell Labs a diseñar UNIX con una filosofía opuesta: simplicidad, modularidad y herramientas con una sola responsabilidad bien definida.",
    "categoria": "inferencia",
    "nivel_dificultad": "hard"
  },
  {
    "id": 18,
    "pregunta": "Resume los hitos más importantes de la evolución de los lenguajes de programación desde 1950 hasta 2000.",
    "respuesta_referencia": "La evolución de los lenguajes de programación desde 1950 hasta 2000 incluyó: la programación en ensamblador (1950s), los primeros lenguajes de alto nivel como FORTRAN y COBOL (1957-1959), la programación estructurada con C y Pascal (1970s), la orientación a objetos con Smalltalk, C++ y Java (1980s-1990s), y el surgimiento de lenguajes de scripting como Python y JavaScript (1990s).",
    "categoria": "resumen",
    "nivel_dificultad": "hard"
  },
  {
    "id": 19,
    "pregunta": "¿Por qué se considera que el desarrollo de la GUI (interfaz gráfica de usuario) fue un punto de inflexión en la adopción masiva de computadoras?",
    "respuesta_referencia": "La GUI eliminó la barrera de la línea de comandos, haciendo las computadoras accesibles para usuarios sin conocimientos técnicos. Xerox PARC desarrolló el concepto en los 1970s, y Apple lo popularizó con el Macintosh en 1984, seguido por Windows, lo que desencadenó la adopción masiva de computadoras personales.",
    "categoria": "inferencia",
    "nivel_dificultad": "medium"
  },
  {
    "id": 20,
    "pregunta": "¿Qué es la computación cuántica y en qué se diferencia fundamentalmente de la computación clásica?",
    "respuesta_referencia": "La computación cuántica utiliza qubits que pueden existir en superposición de estados (0 y 1 simultáneamente) gracias a principios de mecánica cuántica, a diferencia de los bits clásicos que solo pueden ser 0 o 1. Esto permite resolver ciertos problemas exponencialmente más rápido, como factorización de números grandes y simulación molecular.",
    "categoria": "factual",
    "nivel_dificultad": "hard"
  }
]
ENDJSON
```

2. Verifica que el archivo se creó correctamente:

```bash
python -c "
import json
with open('golden_dataset.json') as f:
    data = json.load(f)
print(f'Total de entradas: {len(data)}')
categorias = set(d['categoria'] for d in data)
dificultades = set(d['nivel_dificultad'] for d in data)
print(f'Categorías: {categorias}')
print(f'Dificultades: {dificultades}')
"
```

#### Salida esperada

```
Total de entradas: 20
Categorías: {'factual', 'inferencia', 'resumen'}
Dificultades: {'easy', 'medium', 'hard'}
```

#### Verificación

El dataset debe tener exactamente 20 entradas con las tres categorías y tres niveles de dificultad representados.

---

### Paso 2 — Definir los modelos de datos con Pydantic

**Objetivo:** Crear el archivo principal `chatbot_evaluator.py` con los modelos de datos tipados que estructurarán todos los resultados del pipeline de evaluación.

#### Instrucciones

1. Crea el archivo `chatbot_evaluator.py` con la siguiente estructura base:

```python
# chatbot_evaluator.py
"""
Framework de evaluación de fidelidad para chatbots.
Compara respuestas generadas contra un Golden Dataset usando
métricas léxicas (ROUGE, BLEU) y semánticas (G-Eval con LLM-as-Judge).
"""

from __future__ import annotations

import json
import os
import time
from pathlib import Path
from typing import Optional

import nltk
import pandas as pd
from dotenv import load_dotenv
from nltk.translate.bleu_score import SmoothingFunction, sentence_bleu
from openai import OpenAI
from pydantic import BaseModel, Field
from rouge_score import rouge_scorer

# ─── Configuración inicial ───────────────────────────────────────────────────

load_dotenv()

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
MODEL_GENERATION = os.getenv("OPENAI_MODEL_GENERATION", "gpt-4o-mini")
MODEL_JUDGE = os.getenv("OPENAI_MODEL_JUDGE", "gpt-4o")

client = OpenAI(api_key=OPENAI_API_KEY)

# ─── Modelos de datos ─────────────────────────────────────────────────────────

class GoldenEntry(BaseModel):
    """Una entrada del Golden Dataset."""
    id: int
    pregunta: str
    respuesta_referencia: str
    categoria: str
    nivel_dificultad: str


class RougeScores(BaseModel):
    """Puntuaciones ROUGE para un par de textos."""
    rouge1_precision: float
    rouge1_recall: float
    rouge1_f1: float
    rouge2_precision: float
    rouge2_recall: float
    rouge2_f1: float
    rougeL_precision: float
    rougeL_recall: float
    rougeL_f1: float


class GEvalResult(BaseModel):
    """Resultado de la evaluación G-Eval con LLM como juez."""
    fidelidad: float = Field(ge=0, le=5, description="Precisión factual respecto a la referencia")
    relevancia: float = Field(ge=0, le=5, description="Qué tan bien responde a la pregunta")
    coherencia: float = Field(ge=0, le=5, description="Fluidez lógica y consistencia interna")
    fluencia: float = Field(ge=0, le=5, description="Calidad gramatical y naturalidad del lenguaje")
    justificacion: str = Field(description="Explicación textual del juez LLM")
    score_promedio: float = Field(ge=0, le=5)

    @classmethod
    def from_scores(
        cls,
        fidelidad: float,
        relevancia: float,
        coherencia: float,
        fluencia: float,
        justificacion: str,
    ) -> "GEvalResult":
        promedio = (fidelidad + relevancia + coherencia + fluencia) / 4
        return cls(
            fidelidad=fidelidad,
            relevancia=relevancia,
            coherencia=coherencia,
            fluencia=fluencia,
            justificacion=justificacion,
            score_promedio=round(promedio, 4),
        )


class EvaluationEntry(BaseModel):
    """Resultado completo de evaluación para una entrada del dataset."""
    golden: GoldenEntry
    respuesta_generada: str
    rouge_scores: RougeScores
    bleu_score: float
    geval_result: Optional[GEvalResult] = None


class EvaluationResults(BaseModel):
    """Contenedor de todos los resultados de evaluación."""
    entries: list[EvaluationEntry]
    model_generation: str
    model_judge: str
    timestamp: str
```

2. Guarda el archivo. No lo ejecutes aún; continuarás añadiendo funciones en los pasos siguientes.

#### Verificación

```bash
python -c "from chatbot_evaluator import GoldenEntry, GEvalResult, EvaluationResults; print('Modelos Pydantic importados correctamente')"
```

---

### Paso 3 — Implementar el módulo de generación de respuestas

**Objetivo:** Añadir la función que obtiene respuestas del chatbot para cada pregunta del Golden Dataset.

#### Instrucciones

1. Agrega las siguientes funciones al final de `chatbot_evaluator.py`:

```python
# ─── Módulo de generación ─────────────────────────────────────────────────────

def load_golden_dataset(path: str = "golden_dataset.json") -> list[GoldenEntry]:
    """Carga y valida el Golden Dataset desde un archivo JSON."""
    with open(path, encoding="utf-8") as f:
        raw = json.load(f)
    return [GoldenEntry(**entry) for entry in raw]


def generate_chatbot_responses(
    golden_dataset: list[GoldenEntry],
    model: str = MODEL_GENERATION,
) -> list[str]:
    """
    Genera respuestas del chatbot para cada pregunta del Golden Dataset.

    Usa gpt-4o-mini por defecto para minimizar costos en la fase de generación.
    El modelo juez (G-Eval) se configura por separado.
    """
    responses = []
    total = len(golden_dataset)

    print(f"\n{'='*60}")
    print(f"Generando respuestas con modelo: {model}")
    print(f"Total de preguntas: {total}")
    print(f"{'='*60}")

    for i, entry in enumerate(golden_dataset, 1):
        print(f"  [{i:02d}/{total}] Procesando: {entry.pregunta[:60]}...")

        try:
            completion = client.chat.completions.create(
                model=model,
                messages=[
                    {
                        "role": "system",
                        "content": (
                            "Eres un asistente experto en historia de la computación. "
                            "Responde de forma precisa, clara y concisa en español. "
                            "Limita tu respuesta a 3-4 oraciones como máximo."
                        ),
                    },
                    {"role": "user", "content": entry.pregunta},
                ],
                temperature=0.3,
                max_tokens=300,
            )
            response_text = completion.choices[0].message.content.strip()
            responses.append(response_text)

        except Exception as e:
            print(f"    ⚠️  Error en entrada {entry.id}: {e}")
            responses.append("")

        # Pausa breve para respetar rate limits
        time.sleep(0.5)

    print(f"\n✅ Generación completada: {len(responses)} respuestas obtenidas.\n")
    return responses
```

#### Verificación rápida (opcional, consume tokens)

```bash
python -c "
from chatbot_evaluator import load_golden_dataset, generate_chatbot_responses
dataset = load_golden_dataset()
# Solo probamos con la primera entrada para verificar conectividad
respuestas = generate_chatbot_responses(dataset[:1])
print('Respuesta de prueba:', respuestas[0][:100])
"
```

---

### Paso 4 — Implementar los módulos ROUGE y BLEU

**Objetivo:** Añadir las funciones de cálculo de métricas léxicas ROUGE y BLEU.

#### Instrucciones

1. Añade el siguiente bloque al final de `chatbot_evaluator.py`:

```python
# ─── Módulo ROUGE ─────────────────────────────────────────────────────────────

def calculate_rouge_scores(
    predictions: list[str],
    references: list[str],
) -> list[RougeScores]:
    """
    Calcula métricas ROUGE-1, ROUGE-2 y ROUGE-L para cada par predicción/referencia.

    Usa use_stemmer=True para normalizar variantes morfológicas del español,
    reduciendo penalizaciones injustas por conjugaciones o plurales.
    """
    scorer = rouge_scorer.RougeScorer(
        ["rouge1", "rouge2", "rougeL"],
        use_stemmer=True,
    )

    results = []
    for pred, ref in zip(predictions, references):
        # Manejar respuestas vacías
        if not pred.strip():
            results.append(RougeScores(
                rouge1_precision=0.0, rouge1_recall=0.0, rouge1_f1=0.0,
                rouge2_precision=0.0, rouge2_recall=0.0, rouge2_f1=0.0,
                rougeL_precision=0.0, rougeL_recall=0.0, rougeL_f1=0.0,
            ))
            continue

        scores = scorer.score(ref, pred)
        results.append(RougeScores(
            rouge1_precision=round(scores["rouge1"].precision, 4),
            rouge1_recall=round(scores["rouge1"].recall, 4),
            rouge1_f1=round(scores["rouge1"].fmeasure, 4),
            rouge2_precision=round(scores["rouge2"].precision, 4),
            rouge2_recall=round(scores["rouge2"].recall, 4),
            rouge2_f1=round(scores["rouge2"].fmeasure, 4),
            rougeL_precision=round(scores["rougeL"].precision, 4),
            rougeL_recall=round(scores["rougeL"].recall, 4),
            rougeL_f1=round(scores["rougeL"].fmeasure, 4),
        ))

    return results


# ─── Módulo BLEU ──────────────────────────────────────────────────────────────

def calculate_bleu_scores(
    predictions: list[str],
    references: list[str],
) -> list[float]:
    """
    Calcula BLEU-4 para cada par predicción/referencia.

    Usa SmoothingFunction.method1 para evitar log(0) cuando algún
    n-grama de orden superior no tiene coincidencias (frecuente en
    oraciones cortas o con vocabulario muy distinto).
    """
    smoothing = SmoothingFunction().method1
    bleu_scores = []

    for pred, ref in zip(predictions, references):
        if not pred.strip():
            bleu_scores.append(0.0)
            continue

        # Tokenización simple por espacios (adecuada para esta demo)
        ref_tokens = [ref.lower().split()]
        pred_tokens = pred.lower().split()

        score = sentence_bleu(
            ref_tokens,
            pred_tokens,
            weights=(0.25, 0.25, 0.25, 0.25),  # BLEU-4 uniforme
            smoothing_function=smoothing,
        )
        bleu_scores.append(round(score, 4))

    return bleu_scores
```

#### Verificación

```bash
python -c "
from chatbot_evaluator import calculate_rouge_scores, calculate_bleu_scores

ref = ['Alan Turing es considerado el padre de la computación moderna.']
pred_buena = ['Alan Turing es reconocido como el padre de la computación moderna y desarrolló la máquina de Turing.']
pred_mala = ['La pizza es un plato italiano muy popular en todo el mundo.']

rouge_buena = calculate_rouge_scores(pred_buena, ref)
rouge_mala = calculate_rouge_scores(pred_mala, ref)
bleu_buena = calculate_bleu_scores(pred_buena, ref)
bleu_mala = calculate_bleu_scores(pred_mala, ref)

print(f'ROUGE-1 F1 (buena): {rouge_buena[0].rouge1_f1}')
print(f'ROUGE-1 F1 (mala):  {rouge_mala[0].rouge1_f1}')
print(f'BLEU-4 (buena): {bleu_buena[0]}')
print(f'BLEU-4 (mala):  {bleu_mala[0]}')
"
```

#### Salida esperada

```
ROUGE-1 F1 (buena): ~0.55–0.70  (valor alto: vocabulario compartido)
ROUGE-1 F1 (mala):  ~0.05–0.10  (valor bajo: vocabulario completamente diferente)
BLEU-4 (buena): ~0.15–0.35
BLEU-4 (mala):  ~0.00–0.02
```

---

### Paso 5 — Implementar el módulo G-Eval

**Objetivo:** Añadir la función que usa GPT-4o como juez para evaluar fidelidad, relevancia, coherencia y fluencia.

#### Instrucciones

1. Añade el siguiente bloque al final de `chatbot_evaluator.py`:

```python
# ─── Módulo G-Eval ────────────────────────────────────────────────────────────

GEVAL_SYSTEM_PROMPT = """Eres un evaluador experto de sistemas de IA. Tu tarea es evaluar la calidad de una respuesta generada por un chatbot comparándola con una respuesta de referencia escrita por un experto humano.

Evalúa la respuesta generada en CUATRO dimensiones usando una escala del 0 al 5:

- **Fidelidad** (0-5): ¿Los hechos en la respuesta generada son precisos y consistentes con la referencia? 
  - 5: Todos los hechos son correctos y completos
  - 3: La mayoría son correctos, algún detalle menor falta o es impreciso
  - 0: Los hechos son incorrectos o contradicen la referencia

- **Relevancia** (0-5): ¿La respuesta aborda directamente la pregunta formulada?
  - 5: Responde perfectamente la pregunta sin información irrelevante
  - 3: Responde parcialmente o incluye algo de información innecesaria
  - 0: No responde la pregunta o es completamente irrelevante

- **Coherencia** (0-5): ¿El texto fluye de forma lógica y consistente internamente?
  - 5: Perfectamente estructurado y lógico
  - 3: Mayormente coherente con alguna inconsistencia menor
  - 0: Incoherente o contradictorio internamente

- **Fluencia** (0-5): ¿El lenguaje es natural, gramaticalmente correcto y fácil de leer?
  - 5: Lenguaje completamente natural y sin errores
  - 3: Comprensible con algunos errores menores
  - 0: Agramatical o muy difícil de leer

Responde ÚNICAMENTE con un objeto JSON válido con esta estructura exacta:
{
  "fidelidad": <número 0-5>,
  "relevancia": <número 0-5>,
  "coherencia": <número 0-5>,
  "fluencia": <número 0-5>,
  "justificacion": "<explicación concisa de 2-3 oraciones>"
}"""


def evaluate_with_geval(
    question: str,
    reference: str,
    prediction: str,
    model: str = MODEL_JUDGE,
) -> GEvalResult:
    """
    Evalúa una respuesta generada usando GPT-4o como juez (patrón LLM-as-a-Judge).

    Implementa el patrón G-Eval: el LLM recibe la pregunta, la respuesta
    de referencia y la respuesta generada, y devuelve puntuaciones
    estructuradas con justificación textual.
    """
    user_prompt = f"""**Pregunta:** {question}

**Respuesta de referencia (ground truth):**
{reference}

**Respuesta generada por el chatbot:**
{prediction}

Evalúa la respuesta generada según los criterios indicados."""

    try:
        completion = client.chat.completions.create(
            model=model,
            messages=[
                {"role": "system", "content": GEVAL_SYSTEM_PROMPT},
                {"role": "user", "content": user_prompt},
            ],
            temperature=0.0,  # Determinístico para evaluación reproducible
            max_tokens=400,
            response_format={"type": "json_object"},
        )

        raw_json = completion.choices[0].message.content
        data = json.loads(raw_json)

        return GEvalResult.from_scores(
            fidelidad=float(data.get("fidelidad", 0)),
            relevancia=float(data.get("relevancia", 0)),
            coherencia=float(data.get("coherencia", 0)),
            fluencia=float(data.get("fluencia", 0)),
            justificacion=data.get("justificacion", "Sin justificación disponible."),
        )

    except Exception as e:
        print(f"    ⚠️  Error en G-Eval: {e}")
        return GEvalResult.from_scores(
            fidelidad=0.0,
            relevancia=0.0,
            coherencia=0.0,
            fluencia=0.0,
            justificacion=f"Error durante la evaluación: {str(e)}",
        )


def run_geval_evaluation(
    dataset: list[GoldenEntry],
    predictions: list[str],
    model: str = MODEL_JUDGE,
) -> list[GEvalResult]:
    """Ejecuta G-Eval sobre todo el dataset con manejo de rate limits."""
    results = []
    total = len(dataset)

    print(f"\n{'='*60}")
    print(f"Ejecutando G-Eval con modelo juez: {model}")
    print(f"{'='*60}")

    for i, (entry, pred) in enumerate(zip(dataset, predictions), 1):
        print(f"  [{i:02d}/{total}] Evaluando G-Eval para entrada {entry.id}...")
        result = evaluate_with_geval(
            question=entry.pregunta,
            reference=entry.respuesta_referencia,
            prediction=pred,
            model=model,
        )
        results.append(result)
        print(f"    → Score promedio: {result.score_promedio:.2f}/5.0")

        # Pausa para respetar rate limits de GPT-4o
        time.sleep(1.0)

    print(f"\n✅ G-Eval completado: {len(results)} evaluaciones.\n")
    return results
```

#### Verificación

```bash
python -c "
from chatbot_evaluator import evaluate_with_geval
resultado = evaluate_with_geval(
    question='¿Quién inventó la World Wide Web?',
    reference='Tim Berners-Lee inventó la World Wide Web en 1989 mientras trabajaba en el CERN.',
    prediction='La World Wide Web fue creada por Tim Berners-Lee en 1989 en el CERN, Suiza.'
)
print(f'Fidelidad:   {resultado.fidelidad}/5')
print(f'Relevancia:  {resultado.relevancia}/5')
print(f'Coherencia:  {resultado.coherencia}/5')
print(f'Fluencia:    {resultado.fluencia}/5')
print(f'Promedio:    {resultado.score_promedio}/5')
print(f'Justificación: {resultado.justificacion}')
"
```

#### Salida esperada

```
Fidelidad:   5.0/5
Relevancia:  5.0/5
Coherencia:  5.0/5
Fluencia:    5.0/5
Promedio:    5.0/5
Justificación: La respuesta generada es factualmente correcta y completa...
```

---

### Paso 6 — Implementar el módulo de reporte

**Objetivo:** Añadir las funciones que generan el reporte de evaluación en Markdown y HTML.

#### Instrucciones

1. Añade el siguiente bloque al final de `chatbot_evaluator.py`:

```python
# ─── Módulo de reporte ────────────────────────────────────────────────────────

def build_results_dataframe(results: EvaluationResults) -> pd.DataFrame:
    """Construye un DataFrame pandas con todas las métricas para análisis."""
    rows = []
    for entry in results.entries:
        row = {
            "id": entry.golden.id,
            "pregunta": entry.golden.pregunta[:60] + "...",
            "categoria": entry.golden.categoria,
            "dificultad": entry.golden.nivel_dificultad,
            "rouge1_f1": entry.rouge_scores.rouge1_f1,
            "rouge2_f1": entry.rouge_scores.rouge2_f1,
            "rougeL_f1": entry.rouge_scores.rougeL_f1,
            "bleu4": entry.bleu_score,
        }
        if entry.geval_result:
            row.update({
                "geval_fidelidad": entry.geval_result.fidelidad,
                "geval_relevancia": entry.geval_result.relevancia,
                "geval_coherencia": entry.geval_result.coherencia,
                "geval_fluencia": entry.geval_result.fluencia,
                "geval_promedio": entry.geval_result.score_promedio,
                "geval_justificacion": entry.geval_result.justificacion,
            })
        rows.append(row)

    return pd.DataFrame(rows)


def generate_evaluation_report(results: EvaluationResults) -> str:
    """
    Genera un reporte completo de evaluación en formato Markdown.

    Incluye:
    - Tabla comparativa de todas las métricas
    - Promedios por categoría y dificultad
    - Top 3 mejores y peores respuestas
    - Análisis de correlación entre métricas léxicas y G-Eval
    - Conclusiones sobre la calidad del chatbot
    """
    df = build_results_dataframe(results)
    has_geval = "geval_promedio" in df.columns

    lines = []

    # ── Encabezado ──────────────────────────────────────────────────────────
    lines.append("# Reporte de Evaluación de Fidelidad del Chatbot\n")
    lines.append(f"**Modelo evaluado:** `{results.model_generation}`  ")
    lines.append(f"**Modelo juez (G-Eval):** `{results.model_judge}`  ")
    lines.append(f"**Fecha:** {results.timestamp}  ")
    lines.append(f"**Total de ejemplos:** {len(results.entries)}\n")
    lines.append("---\n")

    # ── Resumen de métricas globales ─────────────────────────────────────────
    lines.append("## 1. Resumen de Métricas Globales\n")
    lines.append("| Métrica | Promedio | Mínimo | Máximo |")
    lines.append("|---------|----------|--------|--------|")

    metric_cols = ["rouge1_f1", "rouge2_f1", "rougeL_f1", "bleu4"]
    metric_labels = ["ROUGE-1 F1", "ROUGE-2 F1", "ROUGE-L F1", "BLEU-4"]
    if has_geval:
        metric_cols += ["geval_fidelidad", "geval_relevancia", "geval_coherencia",
                        "geval_fluencia", "geval_promedio"]
        metric_labels += ["G-Eval Fidelidad", "G-Eval Relevancia", "G-Eval Coherencia",
                          "G-Eval Fluencia", "G-Eval Promedio"]

    for col, label in zip(metric_cols, metric_labels):
        if col in df.columns:
            avg = df[col].mean()
            mn = df[col].min()
            mx = df[col].max()
            lines.append(f"| {label} | {avg:.4f} | {mn:.4f} | {mx:.4f} |")

    lines.append("")

    # ── Promedios por categoría ──────────────────────────────────────────────
    lines.append("## 2. Análisis por Categoría\n")
    agg_cols = ["rouge1_f1", "rouge2_f1", "rougeL_f1", "bleu4"]
    if has_geval:
        agg_cols.append("geval_promedio")

    cat_group = df.groupby("categoria")[agg_cols].mean().round(4)
    lines.append(cat_group.to_markdown())
    lines.append("")

    # ── Promedios por dificultad ─────────────────────────────────────────────
    lines.append("## 3. Análisis por Nivel de Dificultad\n")
    diff_order = ["easy", "medium", "hard"]
    diff_group = df.groupby("dificultad")[agg_cols].mean().round(4)
    diff_group = diff_group.reindex([d for d in diff_order if d in diff_group.index])
    lines.append(diff_group.to_markdown())
    lines.append("")

    # ── Top 3 mejores respuestas ─────────────────────────────────────────────
    lines.append("## 4. Top 3 Mejores Respuestas\n")
    sort_col = "geval_promedio" if has_geval else "rouge1_f1"
    top3 = df.nlargest(3, sort_col)

    for _, row in top3.iterrows():
        entry = next(e for e in results.entries if e.golden.id == row["id"])
        lines.append(f"### Entrada #{entry.golden.id} — Score: {row[sort_col]:.4f}\n")
        lines.append(f"**Pregunta:** {entry.golden.pregunta}\n")
        lines.append(f"**Referencia:** {entry.golden.respuesta_referencia}\n")
        lines.append(f"**Generada:** {entry.respuesta_generada}\n")
        lines.append(
            f"**Métricas:** ROUGE-1={row['rouge1_f1']:.4f} | "
            f"ROUGE-L={row['rougeL_f1']:.4f} | BLEU-4={row['bleu4']:.4f}"
        )
        if has_geval and entry.geval_result:
            lines.append(
                f" | G-Eval={row['geval_promedio']:.4f}"
            )
            lines.append(f"\n**Justificación G-Eval:** {entry.geval_result.justificacion}\n")
        else:
            lines.append("\n")

    # ── Top 3 peores respuestas ──────────────────────────────────────────────
    lines.append("## 5. Top 3 Peores Respuestas\n")
    bottom3 = df.nsmallest(3, sort_col)

    for _, row in bottom3.iterrows():
        entry = next(e for e in results.entries if e.golden.id == row["id"])
        lines.append(f"### Entrada #{entry.golden.id} — Score: {row[sort_col]:.4f}\n")
        lines.append(f"**Pregunta:** {entry.golden.pregunta}\n")
        lines.append(f"**Referencia:** {entry.golden.respuesta_referencia}\n")
        lines.append(f"**Generada:** {entry.respuesta_generada}\n")
        lines.append(
            f"**Métricas:** ROUGE-1={row['rouge1_f1']:.4f} | "
            f"ROUGE-L={row['rougeL_f1']:.4f} | BLEU-4={row['bleu4']:.4f}"
        )
        if has_geval and entry.geval_result:
            lines.append(f" | G-Eval={row['geval_promedio']:.4f}")
            lines.append(f"\n**Justificación G-Eval:** {entry.geval_result.justificacion}\n")
        else:
            lines.append("\n")

    # ── Análisis de correlación ──────────────────────────────────────────────
    if has_geval:
        lines.append("## 6. Correlación entre Métricas Léxicas y G-Eval\n")
        corr_cols = ["rouge1_f1", "rouge2_f1", "rougeL_f1", "bleu4", "geval_promedio"]
        corr_matrix = df[corr_cols].corr().round(4)
        lines.append(corr_matrix.to_markdown())
        lines.append("")
        lines.append(
            "> **Interpretación:** Una correlación alta entre ROUGE/BLEU y G-Eval indica "
            "que las métricas léxicas son buenos proxies de calidad para este dominio. "
            "Una correlación baja sugiere que el chatbot usa vocabulario diferente al de "
            "la referencia pero puede ser semánticamente correcto (o incorrecto).\n"
        )

    # ── Tabla completa ───────────────────────────────────────────────────────
    lines.append("## 7. Tabla Completa de Resultados\n")
    display_cols = ["id", "categoria", "dificultad", "rouge1_f1", "rouge2_f1",
                    "rougeL_f1", "bleu4"]
    if has_geval:
        display_cols.append("geval_promedio")
    lines.append(df[display_cols].to_markdown(index=False))
    lines.append("")

    # ── Conclusiones ────────────────────────────────────────────────────────
    lines.append("## 8. Conclusiones\n")

    avg_rouge1 = df["rouge1_f1"].mean()
    avg_bleu = df["bleu4"].mean()

    lines.append(f"- **ROUGE-1 F1 promedio:** {avg_rouge1:.4f} — "
                 f"{'Aceptable' if avg_rouge1 > 0.4 else 'Bajo: el chatbot usa vocabulario diferente a la referencia'}.")
    lines.append(f"- **BLEU-4 promedio:** {avg_bleu:.4f} — "
                 f"{'Razonable para texto libre' if avg_bleu > 0.1 else 'Muy bajo: esperado en generación libre con sinónimos'}.")

    if has_geval:
        avg_geval = df["geval_promedio"].mean()
        avg_fidelidad = df["geval_fidelidad"].mean()
        lines.append(f"- **G-Eval promedio:** {avg_geval:.4f}/5.0 — "
                     f"{'Buena calidad general' if avg_geval > 3.5 else 'Calidad mejorable'}.")
        lines.append(f"- **Fidelidad promedio (G-Eval):** {avg_fidelidad:.4f}/5.0 — "
                     f"{'Los hechos son mayormente correctos' if avg_fidelidad > 3.5 else 'Hay problemas de precisión factual'}.")

        # Detectar divergencia léxico vs semántico
        corr_rouge_geval = df["rouge1_f1"].corr(df["geval_promedio"])
        lines.append(
            f"- **Correlación ROUGE-1 ↔ G-Eval:** {corr_rouge_geval:.4f} — "
            f"{'Alta: las métricas léxicas son buen proxy' if abs(corr_rouge_geval) > 0.6 else 'Baja: el chatbot puede ser correcto semánticamente aunque use vocabulario diferente'}."
        )

    lines.append(
        "\n**Recomendación:** Para sistemas de producción, combinar métricas léxicas "
        "(rápidas y sin costo) como filtro inicial con G-Eval (más costoso pero preciso) "
        "para los casos límite o en auditorías periódicas de calidad."
    )

    return "\n".join(lines)
```

---

### Paso 7 — Implementar el pipeline principal y exportación

**Objetivo:** Añadir la función `main()` que orquesta todo el pipeline y exporta los resultados.

#### Instrucciones

1. Añade el bloque final a `chatbot_evaluator.py`:

```python
# ─── Exportación HTML ─────────────────────────────────────────────────────────

def export_html_report(markdown_content: str, output_path: str) -> None:
    """
    Convierte el reporte Markdown a HTML usando una plantilla Jinja2 mínima.
    Requiere: pip install jinja2
    """
    try:
        from jinja2 import Template

        html_template = Template("""<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reporte de Evaluación de Chatbot</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
               max-width: 1100px; margin: 40px auto; padding: 0 20px;
               color: #333; line-height: 1.6; }
        h1 { color: #1a1a2e; border-bottom: 3px solid #4a90d9; padding-bottom: 10px; }
        h2 { color: #16213e; border-bottom: 1px solid #ddd; padding-bottom: 5px; margin-top: 40px; }
        h3 { color: #0f3460; }
        table { border-collapse: collapse; width: 100%; margin: 20px 0; font-size: 0.9em; }
        th { background-color: #4a90d9; color: white; padding: 10px 12px; text-align: left; }
        td { padding: 8px 12px; border-bottom: 1px solid #eee; }
        tr:hover { background-color: #f5f9ff; }
        code { background: #f4f4f4; padding: 2px 6px; border-radius: 3px; font-size: 0.9em; }
        pre { background: #f4f4f4; padding: 15px; border-radius: 5px; overflow-x: auto; }
        blockquote { border-left: 4px solid #4a90d9; margin: 0; padding: 10px 20px;
                     background: #f0f7ff; border-radius: 0 5px 5px 0; }
        strong { color: #1a1a2e; }
        hr { border: none; border-top: 1px solid #ddd; margin: 30px 0; }
    </style>
</head>
<body>
{{ content }}
</body>
</html>""")

        # Conversión Markdown → HTML básica (sin dependencias extra)
        import re
        html_content = markdown_content

        # Encabezados
        html_content = re.sub(r'^### (.+)$', r'<h3>\1</h3>', html_content, flags=re.MULTILINE)
        html_content = re.sub(r'^## (.+)$', r'<h2>\1</h2>', html_content, flags=re.MULTILINE)
        html_content = re.sub(r'^# (.+)$', r'<h1>\1</h1>', html_content, flags=re.MULTILINE)

        # Negritas e itálicas
        html_content = re.sub(r'\*\*(.+?)\*\*', r'<strong>\1</strong>', html_content)
        html_content = re.sub(r'\*(.+?)\*', r'<em>\1</em>', html_content)

        # Código inline
        html_content = re.sub(r'`(.+?)`', r'<code>\1</code>', html_content)

        # Blockquotes
        html_content = re.sub(r'^> (.+)$', r'<blockquote>\1</blockquote>', html_content, flags=re.MULTILINE)

        # Separadores
        html_content = html_content.replace('\n---\n', '\n<hr>\n')

        # Saltos de línea para párrafos
        html_content = re.sub(r'\n{2,}', '</p>\n<p>', html_content)
        html_content = f'<p>{html_content}</p>'

        final_html = html_template.render(content=html_content)

        with open(output_path, "w", encoding="utf-8") as f:
            f.write(final_html)

        print(f"✅ Reporte HTML exportado: {output_path}")

    except Exception as e:
        print(f"⚠️  No se pudo generar el HTML: {e}")


# ─── Pipeline principal ───────────────────────────────────────────────────────

def main(skip_geval: bool = False) -> None:
    """
    Orquesta el pipeline completo de evaluación:
    1. Carga el Golden Dataset
    2. Genera respuestas del chatbot
    3. Calcula métricas ROUGE y BLEU
    4. Ejecuta G-Eval (opcional)
    5. Genera y exporta el reporte
    """
    from datetime import datetime

    print("\n" + "="*60)
    print("  FRAMEWORK DE EVALUACIÓN DE FIDELIDAD DE CHATBOT")
    print("="*60)

    # 1. Cargar dataset
    print("\n[1/5] Cargando Golden Dataset...")
    dataset = load_golden_dataset("golden_dataset.json")
    print(f"      ✅ {len(dataset)} entradas cargadas.")

    # 2. Generar respuestas
    print("\n[2/5] Generando respuestas del chatbot...")
    predictions = generate_chatbot_responses(dataset, model=MODEL_GENERATION)

    # 3. Calcular ROUGE
    print("\n[3/5] Calculando métricas ROUGE...")
    references = [e.respuesta_referencia for e in dataset]
    rouge_results = calculate_rouge_scores(predictions, references)
    print(f"      ✅ ROUGE calculado para {len(rouge_results)} pares.")

    # 4. Calcular BLEU
    print("\n[4/5] Calculando métricas BLEU...")
    bleu_results = calculate_bleu_scores(predictions, references)
    print(f"      ✅ BLEU calculado para {len(bleu_results)} pares.")

    # 5. G-Eval (opcional)
    geval_results = None
    if not skip_geval:
        print("\n[5/5] Ejecutando G-Eval (LLM-as-Judge)...")
        print("      ⚠️  Esto consume tokens de GPT-4o (~$0.20-0.50 USD)")
        geval_results = run_geval_evaluation(dataset, predictions, model=MODEL_JUDGE)
    else:
        print("\n[5/5] G-Eval omitido (skip_geval=True).")

    # Ensamblar resultados
    entries = []
    for i, (golden, pred, rouge, bleu) in enumerate(
        zip(dataset, predictions, rouge_results, bleu_results)
    ):
        entry = EvaluationEntry(
            golden=golden,
            respuesta_generada=pred,
            rouge_scores=rouge,
            bleu_score=bleu,
            geval_result=geval_results[i] if geval_results else None,
        )
        entries.append(entry)

    eval_results = EvaluationResults(
        entries=entries,
        model_generation=MODEL_GENERATION,
        model_judge=MODEL_JUDGE,
        timestamp=datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
    )

    # Generar reporte
    print("\n[6/6] Generando reporte de evaluación...")
    Path("reports").mkdir(exist_ok=True)

    report_md = generate_evaluation_report(eval_results)

    md_path = "reports/evaluation_report.md"
    with open(md_path, "w", encoding="utf-8") as f:
        f.write(report_md)
    print(f"      ✅ Reporte Markdown: {md_path}")

    html_path = "reports/evaluation_report.html"
    export_html_report(report_md, html_path)

    # Guardar resultados en JSON para análisis posterior
    json_path = "reports/evaluation_results.json"
    with open(json_path, "w", encoding="utf-8") as f:
        json.dump(eval_results.model_dump(), f, ensure_ascii=False, indent=2)
    print(f"      ✅ Datos JSON: {json_path}")

    # Resumen en consola
    df = build_results_dataframe(eval_results)
    print("\n" + "="*60)
    print("  RESUMEN DE MÉTRICAS GLOBALES")
    print("="*60)
    print(f"  ROUGE-1 F1 promedio: {df['rouge1_f1'].mean():.4f}")
    print(f"  ROUGE-2 F1 promedio: {df['rouge2_f1'].mean():.4f}")
    print(f"  ROUGE-L F1 promedio: {df['rougeL_f1'].mean():.4f}")
    print(f"  BLEU-4 promedio:     {df['bleu4'].mean():.4f}")
    if "geval_promedio" in df.columns:
        print(f"  G-Eval promedio:     {df['geval_promedio'].mean():.4f}/5.0")
    print("="*60)
    print("\n✅ Evaluación completada. Revisa la carpeta 'reports/'.\n")


if __name__ == "__main__":
    import argparse

    parser = argparse.ArgumentParser(
        description="Evalúa la fidelidad de un chatbot contra un Golden Dataset."
    )
    parser.add_argument(
        "--skip-geval",
        action="store_true",
        help="Omite la evaluación G-Eval (sin costo de GPT-4o como juez).",
    )
    args = parser.parse_args()
    main(skip_geval=args.skip_geval)
```

---

## Validación y Pruebas

### Ejecución completa del pipeline

**Opción A — Sin G-Eval (sin costo adicional, solo ROUGE y BLEU):**

```bash
python chatbot_evaluator.py --skip-geval
```

**Opción B — Pipeline completo con G-Eval (costo estimado: ~$0.20–$0.50 USD):**

```bash
python chatbot_evaluator.py
```

### Verificación de los archivos generados

```bash
ls -la reports/
```

Debes ver:

```
reports/
├── evaluation_report.md     (~15-25 KB)
├── evaluation_report.html   (~20-35 KB)
└── evaluation_results.json  (~30-50 KB)
```

### Verificación del contenido del reporte

```bash
# Verificar que el reporte contiene todas las secciones esperadas
python -c "
with open('reports/evaluation_report.md', 'r', encoding='utf-8') as f:
    content = f.read()

secciones = [
    '## 1. Resumen de Métricas Globales',
    '## 2. Análisis por Categoría',
    '## 3. Análisis por Nivel de Dificultad',
    '## 4. Top 3 Mejores Respuestas',
    '## 5. Top 3 Peores Respuestas',
    '## 7. Tabla Completa de Resultados',
    '## 8. Conclusiones'
]

for seccion in secciones:
    status = '✅' if seccion in content else '❌'
    print(f'{status} {seccion}')
"
```

### Verificación de métricas mínimas esperadas

```bash
python -c "
import json
with open('reports/evaluation_results.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

entries = data['entries']
rouge1_scores = [e['rouge_scores']['rouge1_f1'] for e in entries]
bleu_scores = [e['bleu_score'] for e in entries]

avg_rouge1 = sum(rouge1_scores) / len(rouge1_scores)
avg_bleu = sum(bleu_scores) / len(bleu_scores)

print(f'Total de entradas evaluadas: {len(entries)}')
print(f'ROUGE-1 F1 promedio: {avg_rouge1:.4f}')
print(f'BLEU-4 promedio: {avg_bleu:.4f}')

# Validaciones básicas
assert len(entries) == 20, f'Se esperaban 20 entradas, se encontraron {len(entries)}'
assert avg_rouge1 > 0.0, 'ROUGE-1 promedio debe ser mayor a 0'
assert avg_bleu > 0.0, 'BLEU-4 promedio debe ser mayor a 0'
print('\\n✅ Todas las validaciones pasaron correctamente.')
"
```

#### Salida esperada (valores aproximados)

```
Total de entradas evaluadas: 20
ROUGE-1 F1 promedio: 0.3200–0.5500  (varía según el modelo)
BLEU-4 promedio:     0.0500–0.1500  (esperado bajo en generación libre)

✅ Todas las validaciones pasaron correctamente.
```

> 💡 **Nota sobre los valores:** Los valores de ROUGE-1 entre 0.30 y 0.55 son completamente normales para respuestas generadas por LLMs. Un BLEU-4 bajo (0.05–0.15) es esperado porque los modelos modernos usan vocabulario diferente al de la referencia aunque sean semánticamente correctos. Esto es precisamente por lo que G-Eval añade valor.

---

## Resolución de Problemas

### Problema 1: `ModuleNotFoundError: No module named 'rouge_score'`

**Síntoma:** Al ejecutar `chatbot_evaluator.py`, aparece el error `ModuleNotFoundError: No module named 'rouge_score'` o similar para `nltk` o `pandas`.

**Causa:** El entorno virtual no está activado, o las dependencias se instalaron en el Python del sistema en lugar del entorno virtual del lab.

**Solución:**

```bash
# 1. Verificar que el entorno virtual está activo
# (debe aparecer (.venv) al inicio del prompt)
which python  # macOS/Linux: debe apuntar a .venv/bin/python
# Windows: where python  (debe apuntar a .venv\Scripts\python.exe)

# 2. Si no está activo, activarlo:
source .venv/bin/activate        # macOS/Linux
.venv\Scripts\Activate.ps1       # Windows PowerShell

# 3. Reinstalar dependencias dentro del entorno activo:
pip install -r requirements.txt

# 4. Verificar la instalación:
python -c "import rouge_score, nltk, pandas, openai; print('Todas las dependencias OK')"
```

---

### Problema 2: G-Eval devuelve `GEvalResult` con todos los scores en 0.0 y error en la justificación

**Síntoma:** Todas las entradas de G-Eval muestran `score_promedio: 0.0` y la justificación dice `"Error durante la evaluación: ..."`.

**Causa más común:** La `OPENAI_API_KEY` en el archivo `.env` es inválida, ha expirado, no tiene acceso al modelo `gpt-4o`, o el límite de gasto mensual ha sido alcanzado.

**Solución:**

```bash
# 1. Verificar que el archivo .env existe y tiene la key correcta
cat .env  # No compartas esta salida con nadie

# 2. Probar la API key directamente
python -c "
from dotenv import load_dotenv
import os
from openai import OpenAI
load_dotenv()
client = OpenAI(api_key=os.getenv('OPENAI_API_KEY'))
try:
    resp = client.chat.completions.create(
        model='gpt-4o',
        messages=[{'role': 'user', 'content': 'Di hola'}],
        max_tokens=10
    )
    print('✅ API key válida. Respuesta:', resp.choices[0].message.content)
except Exception as e:
    print(f'❌ Error de API: {e}')
"

# 3. Si el modelo gpt-4o no está disponible, cambiar el juez en .env:
# OPENAI_MODEL_JUDGE=gpt-4o-mini
# (menos preciso pero más económico y con mayor disponibilidad)

# 4. Verificar límites en https://platform.openai.com/usage
```

---

## Limpieza

Una vez completado el laboratorio, ejecuta los siguientes pasos para liberar recursos:

```bash
# 1. Desactivar el entorno virtual
deactivate

# 2. (Opcional) Eliminar el entorno virtual si no lo necesitas más
# rm -rf .venv  # macOS/Linux
# Remove-Item -Recurse -Force .venv  # Windows PowerShell

# 3. Verificar que .env NO está en el repositorio antes de hacer commit
git status  # .env no debe aparecer como archivo rastreado
git check-ignore -v .env  # Debe confirmar que está ignorado

# 4. Los reportes generados son seguros para compartir (no contienen credenciales)
ls reports/
```

> ⚠️ **Importante:** Nunca elimines el archivo `.env` sin antes guardar tu API key en un gestor de contraseñas. El archivo `.env` es el único lugar donde está almacenada localmente.

---

## Resumen

En este laboratorio construiste un framework completo de evaluación de fidelidad para chatbots. Los conceptos clave que aplicaste:

| Componente | Tecnología | Lección aplicada |
|------------|------------|-----------------|
| Golden Dataset | JSON + Pydantic v2 | Diseño de datos de evaluación estructurados |
| Generación de respuestas | OpenAI SDK (`gpt-4o-mini`) | Separación de modelo evaluado vs. modelo juez |
| Métricas ROUGE | `rouge-score` con `use_stemmer=True` | Evaluación orientada al recall con normalización morfológica |
| Métricas BLEU | `nltk` con `SmoothingFunction.method1` | Evaluación orientada a la precisión con suavizado para textos cortos |
| G-Eval | GPT-4o + `response_format: json_object` | Patrón LLM-as-a-Judge con evaluación multidimensional |
| Reporte | `pandas` + Markdown + HTML | Análisis tabular, correlaciones y casos extremos |

### Hallazgos esperados

- **BLEU-4 bajo (0.05–0.15) no significa mala calidad:** Los LLMs modernos parafrasean naturalmente, usando sinónimos que penalizan las métricas léxicas pero que G-Eval reconoce como correctos.
- **ROUGE-L es más robusto que ROUGE-2** para capturar estructura sin penalizar reordenamientos menores de palabras.
- **G-Eval añade valor crítico** especialmente en preguntas de tipo `inferencia` y `resumen`, donde la equivalencia semántica es más importante que la coincidencia léxica exacta.
- **La correlación ROUGE ↔ G-Eval** en dominios factuales tiende a ser moderada (~0.4–0.6), confirmando que ambas métricas capturan dimensiones complementarias de la calidad.

### Recursos adicionales

- [Artículo original ROUGE — Lin, 2004](https://aclanthology.org/W04-1013/)
- [Artículo original BLEU — Papineni et al., 2002](https://aclanthology.org/P02-1040/)
- [G-Eval: NLG Evaluation using GPT-4 — Liu et al., 2023](https://arxiv.org/abs/2303.16634)
- [Documentación rouge-score](https://github.com/google-research/google-research/tree/master/rouge)
- [NLTK BLEU Score API](https://www.nltk.org/api/nltk.translate.bleu_score.html)

---
