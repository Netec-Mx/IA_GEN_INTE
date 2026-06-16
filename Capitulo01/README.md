# Desarrollar un script en Python para evaluar y comparar el costo estimado de procesar un dataset entre OpenAI, Gemini y Anthropic

## 1. Metadatos

| Campo | Detalle |
|---|---|
| Duración estimada | 60 a 75 minutos |
| Complejidad | Media |
| Nivel Bloom | Aplicar |
| Lab ID | 01-00-01 |
| Módulo | 1.1 — Modelos Comerciales vs. Open Source |
| Modalidad | Práctica guiada |
| Producto final | Script modular en Python + reporte CSV |

---

## 2. Descripción General

En este lab vas a construir desde cero un proyecto Python modular para comparar el costo estimado, el consumo de tokens y la latencia de distintos modelos comerciales de IA generativa.

Vas a preparar la estructura del proyecto, crear el dataset de prompts, configurar variables de entorno, instalar dependencias, implementar módulos separados para configuración, tokenización, ejecución de APIs, análisis de costos y generación de reportes.

El script enviará una muestra controlada de prompts a modelos de OpenAI, Google Gemini y Anthropic. Después calculará tokens de entrada, tokens de salida, latencia promedio, costo de la muestra y costo estimado para el dataset completo.

Al finalizar, vas a generar un archivo `cost_comparison_report.csv` con la comparación tabular de los modelos evaluados.

---

## 3. Objetivos de Aprendizaje

Al completar esta práctica, tú podrás:

- Implementar un proyecto Python modular para comparar costos entre proveedores comerciales de IA generativa.
- Preparar archivos de configuración, dataset, variables de entorno y módulos Python desde cero.
- Calcular tokens de entrada usando estrategias específicas por proveedor.
- Ejecutar llamadas reales a APIs comerciales controlando el número de tokens de salida.
- Medir latencia promedio por modelo.
- Calcular costos de entrada, salida y costo total estimado.
- Exportar un reporte comparativo en formato CSV.
- Validar que el proyecto funciona sin exponer credenciales en el código fuente.
- Interpretar qué modelo es más económico, cuál responde más rápido y cuál conviene según el caso de uso.

---

## 4. Cambios aplicados sobre la versión original

| Elemento revisado | Ajuste aplicado |
|---|---|
| Preparación de archivos | Se agregó una sección completa para crear directorio, entorno virtual, `.env`, `.gitignore`, `requirements.txt` y dataset. |
| Conteo de llamadas | Se corrigió la explicación: si usas 6 modelos y 5 prompts por modelo, haces 30 llamadas reales, no 15. |
| SDK de Gemini | Se reemplazó `google-generativeai` por `google-genai`. |
| Conteo de tokens de Claude | Se reemplazó la estimación por caracteres por el conteo oficial de tokens de Anthropic. |
| Precios de modelos | Se dejaron centralizados en `config.py` con una advertencia explícita de revisión antes de impartir o ejecutar el lab. |
| Validaciones | Se agregaron validaciones después de cada tarea, no solo al final. |
| Seguridad | Se reforzó el uso de `.env`, `.gitignore` y revisión de credenciales hardcodeadas. |
| Rate limits | Se agregó pausa configurable y reintentos básicos con backoff. |
| Estructura didáctica | Se separó el lab en tareas, pasos y validaciones funcionales. |
| Lenguaje | Se redactó en segunda persona: “vas a crear”, “ejecuta”, “verifica”, “confirma”. |

---

## 5. Prerrequisitos

### 5.1 Conocimientos previos

Antes de iniciar, tú debes tener conocimientos básicos de:

- Python: funciones, módulos, listas, diccionarios, manejo de archivos y excepciones.
- Uso básico de terminal.
- Variables de entorno.
- Conceptos de tokenización.
- Conceptos básicos de consumo por API.
- Diferencia entre tokens de entrada y tokens de salida.
- Diferencia entre costo, latencia y ventana de contexto.

### 5.2 Cuentas y accesos requeridos

| Recurso | Estado requerido |
|---|---|
| Cuenta OpenAI Platform | Activa, con API key disponible |
| Cuenta Google AI Studio o Google Cloud habilitada para Gemini | Activa, con API key disponible |
| Cuenta Anthropic Console | Activa, con API key disponible |
| Acceso a Internet | Requerido |
| Créditos o límites gratuitos disponibles | Recomendado |

### 5.3 Variables de entorno requeridas

Vas a usar estas variables:

```text
OPENAI_API_KEY
GOOGLE_API_KEY
ANTHROPIC_API_KEY
```

### 5.4 Control de costos

Este lab realiza llamadas reales a APIs comerciales.

Si configuras 6 modelos y una muestra de 5 prompts por modelo, vas a ejecutar:

```text
6 modelos × 5 prompts = 30 llamadas reales
```

Para controlar el costo:

- Limita la salida con `MAX_OUTPUT_TOKENS`.
- Usa una muestra pequeña.
- Verifica los precios actuales antes de ejecutar.
- Configura límites de gasto en cada consola.
- No ejecutes el lab repetidamente sin revisar el consumo.

---

## 6. Entorno del Lab

### 6.1 Hardware mínimo

| Componente | Mínimo | Recomendado |
|---|---|---|
| RAM | 8 GB | 16 GB |
| CPU | 4 núcleos | 8 núcleos |
| Almacenamiento libre | 2 GB | 5 GB |
| Internet | 10 Mbps | 25 Mbps |

### 6.2 Software requerido

| Software | Versión recomendada |
|---|---|
| Python | 3.11 o superior |
| pip | Versión actualizada |
| Terminal | PowerShell, Bash, Zsh o Git Bash |
| Editor | Visual Studio Code o equivalente |

### 6.3 Librerías Python

| Librería | Uso |
|---|---|
| openai | Llamadas a modelos OpenAI |
| google-genai | Llamadas a Gemini usando el SDK actual |
| anthropic | Llamadas a Claude |
| tiktoken | Estimación local de tokens para modelos OpenAI |
| pandas | Generación de tabla y CSV |
| python-dotenv | Carga de variables desde `.env` |

---

# 7. Preparación de archivos del proyecto

## Tarea 1 — Crear el directorio del lab

### Objetivo

- Crear una carpeta limpia para guardar todos los archivos de la práctica. Abre Visual Studio Code.
- Da clic en la opcioón **Terminal** y luego **Nueva terminal**
- Navega al **Escritorio**

### Paso 1. Crear el directorio

Ejecuta:

```bash
cd Desktop
mkdir lab-01-00-01
cd lab-01-00-01
```

Qué haces en este paso:

Creas el directorio principal del laboratorio y entras a esa carpeta.

### Validación

Ejecuta:

```bash
pwd
```

En Windows PowerShell puedes ejecutar:

```powershell
Get-Location
```

Resultado esperado:

Debes estar dentro del directorio `lab-01-00-01`.

---

## Tarea 2 — Crear y activar el entorno virtual

### Objetivo

Aislar las dependencias de este lab para no afectar otros proyectos Python.

### Paso 1. Crear el entorno virtual

Ejecuta:

```bash
python -m venv .venv
```

Qué haces en este paso:

Creas un entorno virtual llamado `.venv`.

### Paso 2. Activar el entorno virtual

En Linux o macOS:

```bash
source .venv/bin/activate
```

En Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

En Windows Git Bash:

```bash
source .venv/Scripts/activate
```

Qué haces en este paso:

Activas el entorno virtual para que las dependencias se instalen dentro del proyecto.

### Paso 3. Actualizar pip

Ejecuta:

```bash
python -m pip install --upgrade pip
```

Qué haces en este paso:

Actualizas el gestor de paquetes para reducir problemas de instalación.

### Validación

Ejecuta:

```bash
python --version
pip --version
```

Resultado esperado:

Debes ver la versión de Python y pip. Además, tu terminal debe mostrar el entorno activo, normalmente con `(.venv)` al inicio.

---

## Tarea 3 — Crear requirements.txt

### Objetivo

Definir las dependencias del proyecto de forma reproducible.

### Paso 1. Crear el archivo `requirements.txt`

Ejecuta:

```bash
cat > requirements.txt << 'EOF'
openai
google-genai
anthropic
tiktoken
pandas
python-dotenv
EOF
```

En Windows PowerShell, si `cat << EOF` no funciona, crea el archivo con este contenido:

```powershell
New-Item -Path "requirements.txt" -ItemType File -Value '
openai
google-genai
anthropic
tiktoken
pandas
python-dotenv
'
```

Qué haces en este paso:

Creas el archivo que lista las librerías necesarias para ejecutar el proyecto.

### Paso 2. Instalar dependencias

Ejecuta:

```bash
pip install -r requirements.txt
```

Qué haces en este paso:

Instalas los SDKs y librerías necesarias para la práctica.

### Validación

Ejecuta:

```bash
python -c "import openai, anthropic, pandas, tiktoken; from google import genai; print('Dependencias instaladas correctamente')"
```

Salida esperada:

```text
Dependencias instaladas correctamente
```

---

## Tarea 4 — Crear archivo `.env`

### Objetivo

Guardar las credenciales de forma segura fuera del código fuente.

### Paso 1. Crear `.env`

Ejecuta:

```bash
cat > .env << 'EOF'
OPENAI_API_KEY=coloca_tu_api_key_de_openai
GOOGLE_API_KEY=coloca_tu_api_key_de_google
ANTHROPIC_API_KEY=coloca_tu_api_key_de_anthropic
EOF
```

En Windows PowerShell puedes crear el archivo manualmente o ejecutar:

```powershell
@"
OPENAI_API_KEY=coloca_tu_api_key_de_openai
GOOGLE_API_KEY=coloca_tu_api_key_de_google
ANTHROPIC_API_KEY=coloca_tu_api_key_de_anthropic
"@ | Out-File -Encoding utf8 .env
```

Qué haces en este paso:

Creas el archivo donde vas a colocar tus claves reales de API.

### Paso 2. Sustituir los valores

Abre `.env` y reemplaza los textos de ejemplo por tus claves reales.

Ejemplo:

```text
OPENAI_API_KEY=sk-...
GOOGLE_API_KEY=AIza...
ANTHROPIC_API_KEY=sk-ant-...
```

Qué haces en este paso:

Configuras las credenciales que el script usará para conectarse a cada proveedor.

### Validación

Ejecuta:

```bash
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print(bool(os.getenv('OPENAI_API_KEY')), bool(os.getenv('GOOGLE_API_KEY')), bool(os.getenv('ANTHROPIC_API_KEY')))"
```

Salida esperada:

```text
True True True
```

Si ves `False`, revisa que el archivo `.env` exista y que las variables estén bien escritas.

---

## Tarea 5 — Crear archivo `.gitignore`

### Objetivo

Evitar que subas credenciales, entornos virtuales y archivos generados a Git.

### Paso 1. Crear `.gitignore`

Ejecuta:

```bash
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
cost_comparison_report.csv
EOF
```

En Windows PowerShell:

```powershell
@"
.env
.venv/
__pycache__/
*.pyc
cost_comparison_report.csv
"@ | Out-File -Encoding utf8 .gitignore
```

Qué haces en este paso:

Defines archivos y carpetas que no deben ser versionados.

### Validación

Ejecuta:

```bash
python -c "content=open('.gitignore', encoding='utf-8').read(); assert '.env' in content; assert '.venv/' in content; print('Archivo .gitignore configurado correctamente')"
```

Salida esperada:

```text
Archivo .gitignore configurado correctamente
```

---

# 8. Desarrollo de la práctica

## Tarea 6 — Crear el dataset de prompts

### Objetivo

Crear el archivo JSON que contiene los 20 prompts que vas a analizar.

### Paso 1. Crear `prompts_dataset.json`

Ejecuta para Linux:

```bash
cat > prompts_dataset.json << 'EOF'
{
  "dataset_name": "cost_analysis_dataset_v1",
  "description": "20 prompts de complejidad variada para análisis de costo entre proveedores",
  "prompts": [
    {
      "id": 1,
      "category": "corto",
      "text": "¿Cuál es la capital de Francia?"
    },
    {
      "id": 2,
      "category": "corto",
      "text": "Traduce 'hello world' al español."
    },
    {
      "id": 3,
      "category": "corto",
      "text": "¿Cuánto es 15% de 200?"
    },
    {
      "id": 4,
      "category": "corto",
      "text": "Dame un sinónimo de 'rápido'."
    },
    {
      "id": 5,
      "category": "corto",
      "text": "¿En qué año se fundó OpenAI?"
    },
    {
      "id": 6,
      "category": "mediano",
      "text": "Explica en 3 oraciones qué es el machine learning y cómo se diferencia del deep learning."
    },
    {
      "id": 7,
      "category": "mediano",
      "text": "Escribe un correo profesional de 100 palabras solicitando una reunión para revisar el avance de un proyecto de software."
    },
    {
      "id": 8,
      "category": "mediano",
      "text": "Describe las ventajas y desventajas de usar microservicios versus una arquitectura monolítica en una startup de tecnología."
    },
    {
      "id": 9,
      "category": "mediano",
      "text": "Genera 5 ideas de nombres para una aplicación móvil de gestión de tareas orientada a desarrolladores de software."
    },
    {
      "id": 10,
      "category": "mediano",
      "text": "Explica el concepto de tokenización en modelos de lenguaje grande y por qué es relevante para el costo de uso de las APIs."
    },
    {
      "id": 11,
      "category": "mediano",
      "text": "¿Cuáles son las mejores prácticas para manejar errores en una API REST construida con FastAPI? Menciona al menos 4 prácticas concretas."
    },
    {
      "id": 12,
      "category": "mediano",
      "text": "Compara brevemente modelos comerciales de IA generativa en términos de ventana de contexto, costo, latencia y casos de uso recomendados."
    },
    {
      "id": 13,
      "category": "largo",
      "text": "Actúa como arquitecto de soluciones de IA. Un cliente del sector salud quiere implementar un chatbot para responder preguntas frecuentes de pacientes sobre sus citas médicas. Los datos son altamente sensibles. Describe una arquitectura completa que incluya selección de modelo, estrategia de almacenamiento de datos, medidas de seguridad y privacidad, estimación de costos mensuales asumiendo 10,000 consultas diarias, y consideraciones de cumplimiento normativo."
    },
    {
      "id": 14,
      "category": "largo",
      "text": "Escribe un tutorial paso a paso de 400 palabras explicando cómo implementar Retrieval-Augmented Generation con Python y una base de datos vectorial. Incluye fragmentos de código comentados para cada etapa: carga de documentos, generación de embeddings, almacenamiento, recuperación de contexto relevante y generación de respuesta final."
    },
    {
      "id": 15,
      "category": "largo",
      "text": "Analiza el siguiente escenario de negocio y proporciona un plan de implementación detallado: Una empresa de e-commerce con 500,000 productos quiere implementar un sistema de recomendaciones personalizadas usando IA generativa. Incluye arquitectura técnica, stack tecnológico recomendado, fases de implementación, KPIs y riesgos principales."
    },
    {
      "id": 16,
      "category": "largo",
      "text": "Genera un documento de especificación técnica para una API REST de análisis de sentimientos. Incluye descripción del servicio, endpoints, métodos HTTP, parámetros de request y response en JSON, códigos de error, ejemplos con curl, autenticación con JWT, rate limiting y SLA de disponibilidad."
    },
    {
      "id": 17,
      "category": "largo",
      "text": "Como experto en MLOps, describe el pipeline completo para llevar un modelo de clasificación de texto desde el experimento local hasta producción en Kubernetes. Incluye MLflow, Docker, CI/CD, Kubernetes, monitoreo de drift y rollback."
    },
    {
      "id": 18,
      "category": "largo",
      "text": "Escribe un análisis comparativo de estrategias de fine-tuning para modelos de lenguaje grande: full fine-tuning, LoRA, QLoRA y prompt tuning. Para cada estrategia explica fundamento técnico, requisitos de hardware, dataset recomendado, costo estimado, casos de uso y limitaciones."
    },
    {
      "id": 19,
      "category": "largo",
      "text": "Desarrolla un plan de seguridad para una aplicación de IA generativa expuesta públicamente. Cubre defensa contra prompt injection, gestión segura de API keys, validación de inputs, límites de tokens por usuario, logging, auditoría y respuesta ante incidentes."
    },
    {
      "id": 20,
      "category": "largo",
      "text": "Como consultor de transformación digital, elabora un roadmap de 12 meses para que una empresa de servicios financieros adopte IA generativa de forma responsable. Incluye fases, entregables, roles, presupuesto aproximado y criterios de go/no-go."
    }
  ]
}
EOF
```

Ejecuta para Windows:

```powershell
# Define the content with a here-string
$jsonContent = @'
{
  "dataset_name": "cost_analysis_dataset_v1",
  "description": "20 prompts de complejidad variada para análisis de costo entre proveedores",
  "prompts": [
    {
      "id": 1,
      "category": "corto",
      "text": "¿Cuál es la capital de Francia?"
    },
    {
      "id": 2,
      "category": "corto",
      "text": "Traduce 'hello world' al español."
    },
    {
      "id": 3,
      "category": "corto",
      "text": "¿Cuánto es 15% de 200?"
    },
    {
      "id": 4,
      "category": "corto",
      "text": "Dame un sinónimo de 'rápido'."
    },
    {
      "id": 5,
      "category": "corto",
      "text": "¿En qué año se fundó OpenAI?"
    },
    {
      "id": 6,
      "category": "mediano",
      "text": "Explica en 3 oraciones qué es el machine learning y cómo se diferencia del deep learning."
    },
    {
      "id": 7,
      "category": "mediano",
      "text": "Escribe un correo profesional de 100 palabras solicitando una reunión para revisar el avance de un proyecto de software."
    },
    {
      "id": 8,
      "category": "mediano",
      "text": "Describe las ventajas y desventajas de usar microservicios versus una arquitectura monolítica en una startup de tecnología."
    },
    {
      "id": 9,
      "category": "mediano",
      "text": "Genera 5 ideas de nombres para una aplicación móvil de gestión de tareas orientada a desarrolladores de software."
    },
    {
      "id": 10,
      "category": "mediano",
      "text": "Explica el concepto de tokenización en modelos de lenguaje grande y por qué es relevante para el costo de uso de las APIs."
    },
    {
      "id": 11,
      "category": "mediano",
      "text": "¿Cuáles son las mejores prácticas para manejar errores en una API REST construida con FastAPI? Menciona al menos 4 prácticas concretas."
    },
    {
      "id": 12,
      "category": "mediano",
      "text": "Compara brevemente modelos comerciales de IA generativa en términos de ventana de contexto, costo, latencia y casos de uso recomendados."
    },
    {
      "id": 13,
      "category": "largo",
      "text": "Actúa como arquitecto de soluciones de IA. Un cliente del sector salud quiere implementar un chatbot para responder preguntas frecuentes de pacientes sobre sus citas médicas. Los datos son altamente sensibles. Describe una arquitectura completa que incluya selección de modelo, estrategia de almacenamiento de datos, medidas de seguridad y privacidad, estimación de costos mensuales asumiendo 10,000 consultas diarias, y consideraciones de cumplimiento normativo."
    },
    {
      "id": 14,
      "category": "largo",
      "text": "Escribe un tutorial paso a paso de 400 palabras explicando cómo implementar Retrieval-Augmented Generation con Python y una base de datos vectorial. Incluye fragmentos de código comentados para cada etapa: carga de documentos, generación de embeddings, almacenamiento, recuperación de contexto relevante y generación de respuesta final."
    },
    {
      "id": 15,
      "category": "largo",
      "text": "Analiza el siguiente escenario de negocio y proporciona un plan de implementación detallado: Una empresa de e-commerce con 500,000 productos quiere implementar un sistema de recomendaciones personalizadas usando IA generativa. Incluye arquitectura técnica, stack tecnológico recomendado, fases de implementación, KPIs y riesgos principales."
    },
    {
      "id": 16,
      "category": "largo",
      "text": "Genera un documento de especificación técnica para una API REST de análisis de sentimientos. Incluye descripción del servicio, endpoints, métodos HTTP, parámetros de request y response en JSON, códigos de error, ejemplos con curl, autenticación con JWT, rate limiting y SLA de disponibilidad."
    },
    {
      "id": 17,
      "category": "largo",
      "text": "Como experto en MLOps, describe el pipeline completo para llevar un modelo de clasificación de texto desde el experimento local hasta producción en Kubernetes. Incluye MLflow, Docker, CI/CD, Kubernetes, monitoreo de drift y rollback."
    },
    {
      "id": 18,
      "category": "largo",
      "text": "Escribe un análisis comparativo de estrategias de fine-tuning para modelos de lenguaje grande: full fine-tuning, LoRA, QLoRA y prompt tuning. Para cada estrategia explica fundamento técnico, requisitos de hardware, dataset recomendado, costo estimado, casos de uso y limitaciones."
    },
    {
      "id": 19,
      "category": "largo",
      "text": "Desarrolla un plan de seguridad para una aplicación de IA generativa expuesta públicamente. Cubre defensa contra prompt injection, gestión segura de API keys, validación de inputs, límites de tokens por usuario, logging, auditoría y respuesta ante incidentes."
    },
    {
      "id": 20,
      "category": "largo",
      "text": "Como consultor de transformación digital, elabora un roadmap de 12 meses para que una empresa de servicios financieros adopte IA generativa de forma responsable. Incluye fases, entregables, roles, presupuesto aproximado y criterios de go/no-go."
    }
  ]
}
'@

# Save the content to the JSON file using UTF-8 encoding
Set-Content -Path ".\prompts_dataset.json" -Value $jsonContent -Encoding UTF8

Write-Host "File 'prompts_dataset.json' created successfully." -ForegroundColor Green
```

Qué haces en este paso:

Creas un dataset con prompts cortos, medianos y largos para probar diferencias de consumo de tokens.

### Paso 2. Validar que el JSON sea correcto

Ejecuta para Linux:

```bash
python -c "import json; data=json.load(open('prompts_dataset.json', encoding='utf-8')); print(f'Dataset cargado: {len(data[\"prompts\"])} prompts')"
```

Ejecuta para Windows:

```powershell
"Dataset cargado: $(((Get-Content .\prompts_dataset.json -Raw | ConvertFrom-Json).prompts).Count) prompts"
```

Salida esperada:

```text
Dataset cargado: 20 prompts
```

### Validación funcional

Ejecuta Linux:

```bash
python -c "import json; data=json.load(open('prompts_dataset.json', encoding='utf-8')); print(set(p['category'] for p in data['prompts']))"
```

Ejecuta Windows:

```powershell
$categories = (Get-Content .\prompts_dataset.json -Raw | ConvertFrom-Json).prompts.category | Select-Object -Unique
"{'$($categories -join "', '")'}"
```

Salida esperada:

```text
{'corto', 'mediano', 'largo'}
```

---

## Tarea 7 — Crear el módulo de configuración

### Objetivo

Centralizar la configuración de modelos, precios, muestra, límites de salida y archivo de reporte.

### Importante sobre precios

Los precios de APIs comerciales cambian. Antes de ejecutar este lab en clase, tú debes revisar las páginas oficiales de precios de cada proveedor y actualizar los valores de `input_price_per_1m` y `output_price_per_1m`.

Los valores incluidos en este archivo son de referencia didáctica y deben validarse antes de usarse para una estimación real.

### Paso 1. Crear `config.py`

- Abre una terminal de Bash dentro de Visual Studio Code.
- Ejecuta el siguiente comando para entrar a la carpeta de la practica.

```bash
cd Desktop/lab-01-00-01/
```


Ejecuta el siguiente comando:

```bash
cat > config.py << 'EOF'
# config.py
# Configuración central del lab.
# IMPORTANTE:
# Los precios de modelos cambian con frecuencia.
# Antes de impartir o ejecutar el lab, valida los precios actuales en las páginas oficiales.

MODEL_CONFIG = {
    # OpenAI
    "gpt-4.1": {
        "provider": "OpenAI",
        "display_name": "GPT-4.1",
        "input_price_per_1m": 2.00,
        "output_price_per_1m": 8.00,
        "context_window_tokens": 1_000_000,
        "tokenizer": "openai_tiktoken",
        "tiktoken_encoding": "cl100k_base",
    },
    "gpt-4.1-mini": {
        "provider": "OpenAI",
        "display_name": "GPT-4.1 mini",
        "input_price_per_1m": 0.40,
        "output_price_per_1m": 1.60,
        "context_window_tokens": 1_000_000,
        "tokenizer": "openai_tiktoken",
        "tiktoken_encoding": "cl100k_base",
    },

    # Google Gemini
    "gemini-2.5-pro": {
        "provider": "Google",
        "display_name": "Gemini 2.5 Pro",
        "input_price_per_1m": 1.25,
        "output_price_per_1m": 10.00,
        "context_window_tokens": 1_000_000,
        "tokenizer": "gemini_api",
    },
    "gemini-2.5-flash": {
        "provider": "Google",
        "display_name": "Gemini 2.5 Flash",
        "input_price_per_1m": 0.30,
        "output_price_per_1m": 2.50,
        "context_window_tokens": 1_000_000,
        "tokenizer": "gemini_api",
    },

    # Anthropic Claude
    "claude-sonnet-4-6": {
        "provider": "Anthropic",
        "display_name": "Claude Sonnet 4.6",
        "input_price_per_1m": 3.00,
        "output_price_per_1m": 15.00,
        "context_window_tokens": 200_000,
        "tokenizer": "anthropic_api",
    },
    "claude-haiku-4-5": {
        "provider": "Anthropic",
        "display_name": "Claude Haiku 4.5",
        "input_price_per_1m": 1.00,
        "output_price_per_1m": 5.00,
        "context_window_tokens": 200_000,
        "tokenizer": "anthropic_api",
    },
}

DATASET_FILE = "prompts_dataset.json"
DATASET_SIZE = 20

SAMPLE_SIZE = 5
SAMPLE_INDICES = [0, 5, 9, 13, 18]

MAX_OUTPUT_TOKENS = 512
OUTPUT_CSV = "cost_comparison_report.csv"

# Pausa base entre llamadas para reducir errores por rate limit.
API_DELAY_SECONDS = 2

# Reintentos por llamada cuando ocurre error temporal.
MAX_RETRIES = 2
EOF
```

Qué haces en este paso:

Defines qué modelos vas a comparar, sus precios por millón de tokens y los parámetros principales del lab.

### Validación

Ejecuta:

```bash
python -c "from config import MODEL_CONFIG; print(f'Modelos configurados: {len(MODEL_CONFIG)}'); print(list(MODEL_CONFIG.keys()))"
```

Salida esperada:

```text
Modelos configurados: 6
['gpt-4.1', 'gpt-4.1-mini', 'gemini-2.5-pro', 'gemini-2.5-flash', 'claude-sonnet-4-6', 'claude-haiku-4-5']
```

---

## Tarea 8 — Crear el módulo de tokenización

### Objetivo

Implementar el conteo de tokens de entrada usando el método adecuado para cada proveedor.

### Paso 1. Crear `tokenizer.py`

Ejecuta el siguiente comando dentro de la carpeta de la practica:

```bash
cat > tokenizer.py << 'EOF'
# tokenizer.py
# Módulo de tokenización multi-proveedor.

import os
import tiktoken
from dotenv import load_dotenv
from google import genai
import anthropic

from config import MODEL_CONFIG

load_dotenv()

google_client = genai.Client(api_key=os.getenv("GOOGLE_API_KEY"))
anthropic_client = anthropic.Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))


def count_tokens_openai_estimate(text: str, encoding_name: str = "cl100k_base") -> int:
    """
    Estima tokens para modelos OpenAI usando tiktoken.

    Nota:
    Este conteo estima tokens del texto plano. La API puede sumar tokens
    adicionales por estructura de mensaje.
    """
    encoding = tiktoken.get_encoding(encoding_name)
    return len(encoding.encode(text))


def count_tokens_gemini(text: str, model_name: str) -> int:
    """
    Cuenta tokens usando la API de Gemini.
    """
    result = google_client.models.count_tokens(
        model=model_name,
        contents=text,
    )
    return int(result.total_tokens)


def count_tokens_anthropic(text: str, model_name: str) -> int:
    """
    Cuenta tokens usando el endpoint oficial de Anthropic.
    """
    result = anthropic_client.messages.count_tokens(
        model=model_name,
        messages=[
            {
                "role": "user",
                "content": text,
            }
        ],
    )
    return int(result.input_tokens)


def count_input_tokens(text: str, model_key: str) -> int:
    """
    Selecciona automáticamente el método de conteo según el modelo.
    """
    config = MODEL_CONFIG[model_key]
    tokenizer_type = config["tokenizer"]

    if tokenizer_type == "openai_tiktoken":
        return count_tokens_openai_estimate(
            text=text,
            encoding_name=config.get("tiktoken_encoding", "cl100k_base"),
        )

    if tokenizer_type == "gemini_api":
        return count_tokens_gemini(
            text=text,
            model_name=model_key,
        )

    if tokenizer_type == "anthropic_api":
        return count_tokens_anthropic(
            text=text,
            model_name=model_key,
        )

    raise ValueError(f"Tipo de tokenizador no soportado: {tokenizer_type}")
EOF
```

Qué haces en este paso:

Creas funciones para contar tokens por proveedor. Para OpenAI usas estimación local con `tiktoken`. Para Gemini y Anthropic usas conteo vía API.

### Validación

Ejecuta:

```bash
python -c "from tokenizer import count_tokens_openai_estimate; print(count_tokens_openai_estimate('Hola, esto es una prueba de tokenización.'))"
```

Salida esperada:

Debes ver un número entero mayor que cero.

Ejemplo:

```text
11
```

---

## Tarea 9 — Crear el módulo de ejecución de APIs

### Objetivo

Enviar prompts reales a cada API, medir latencia y capturar tokens de entrada y salida.

### Paso 1. Crear `api_executor.py`

Ejecuta:

```bash
cat > api_executor.py << 'EOF'
# api_executor.py
# Ejecuta llamadas reales a APIs comerciales y mide latencia.

import os
import time
from dataclasses import dataclass

from dotenv import load_dotenv
from openai import OpenAI
from google import genai
from google.genai import types
import anthropic

from config import MAX_OUTPUT_TOKENS, MAX_RETRIES

load_dotenv()

openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
google_client = genai.Client(api_key=os.getenv("GOOGLE_API_KEY"))
anthropic_client = anthropic.Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))


@dataclass
class APICallResult:
    model_key: str
    prompt_id: int
    input_tokens: int
    output_tokens: int
    latency_seconds: float
    success: bool
    error_message: str = ""


def retry_sleep(attempt: int) -> None:
    """
    Aplica una pausa incremental para reducir fallos temporales por rate limit.
    """
    time.sleep(2 ** attempt)


def call_openai(prompt: str, model_key: str, prompt_id: int) -> APICallResult:
    """
    Envía un prompt a OpenAI usando Responses API.
    """
    start = time.perf_counter()

    for attempt in range(MAX_RETRIES + 1):
        try:
            response = openai_client.responses.create(
                model=model_key,
                input=prompt,
                max_output_tokens=MAX_OUTPUT_TOKENS,
            )

            latency = time.perf_counter() - start
            usage = response.usage

            return APICallResult(
                model_key=model_key,
                prompt_id=prompt_id,
                input_tokens=int(getattr(usage, "input_tokens", 0) or 0),
                output_tokens=int(getattr(usage, "output_tokens", 0) or 0),
                latency_seconds=round(latency, 3),
                success=True,
            )

        except Exception as e:
            if attempt < MAX_RETRIES:
                retry_sleep(attempt)
                continue

            latency = time.perf_counter() - start
            return APICallResult(
                model_key=model_key,
                prompt_id=prompt_id,
                input_tokens=0,
                output_tokens=0,
                latency_seconds=round(latency, 3),
                success=False,
                error_message=str(e),
            )


def call_gemini(prompt: str, model_key: str, prompt_id: int) -> APICallResult:
    """
    Envía un prompt a Gemini usando Google GenAI SDK.
    """
    start = time.perf_counter()

    for attempt in range(MAX_RETRIES + 1):
        try:
            response = google_client.models.generate_content(
                model=model_key,
                contents=prompt,
                config=types.GenerateContentConfig(
                    max_output_tokens=MAX_OUTPUT_TOKENS,
                ),
            )

            latency = time.perf_counter() - start
            usage = response.usage_metadata

            return APICallResult(
                model_key=model_key,
                prompt_id=prompt_id,
                input_tokens=int(getattr(usage, "prompt_token_count", 0) or 0),
                output_tokens=int(getattr(usage, "candidates_token_count", 0) or 0),
                latency_seconds=round(latency, 3),
                success=True,
            )

        except Exception as e:
            if attempt < MAX_RETRIES:
                retry_sleep(attempt)
                continue

            latency = time.perf_counter() - start
            return APICallResult(
                model_key=model_key,
                prompt_id=prompt_id,
                input_tokens=0,
                output_tokens=0,
                latency_seconds=round(latency, 3),
                success=False,
                error_message=str(e),
            )


def call_anthropic(prompt: str, model_key: str, prompt_id: int) -> APICallResult:
    """
    Envía un prompt a Anthropic Claude.
    """
    start = time.perf_counter()

    for attempt in range(MAX_RETRIES + 1):
        try:
            response = anthropic_client.messages.create(
                model=model_key,
                max_tokens=MAX_OUTPUT_TOKENS,
                messages=[
                    {
                        "role": "user",
                        "content": prompt,
                    }
                ],
            )

            latency = time.perf_counter() - start

            return APICallResult(
                model_key=model_key,
                prompt_id=prompt_id,
                input_tokens=int(response.usage.input_tokens),
                output_tokens=int(response.usage.output_tokens),
                latency_seconds=round(latency, 3),
                success=True,
            )

        except Exception as e:
            if attempt < MAX_RETRIES:
                retry_sleep(attempt)
                continue

            latency = time.perf_counter() - start
            return APICallResult(
                model_key=model_key,
                prompt_id=prompt_id,
                input_tokens=0,
                output_tokens=0,
                latency_seconds=round(latency, 3),
                success=False,
                error_message=str(e),
            )


PROVIDER_CALL_MAP = {
    "OpenAI": call_openai,
    "Google": call_gemini,
    "Anthropic": call_anthropic,
}
EOF
```

Qué haces en este paso:

Creas las funciones que ejecutan llamadas reales a OpenAI, Gemini y Anthropic. Cada función mide latencia, captura tokens y maneja errores.

### Validación

Ejecuta:

```bash
python -c "from api_executor import PROVIDER_CALL_MAP; print(list(PROVIDER_CALL_MAP.keys()))"
```

Salida esperada:

```text
['OpenAI', 'Google', 'Anthropic']
```

---

## Tarea 10 — Crear el módulo de análisis de costos

### Objetivo

Calcular costos por modelo usando tokens de entrada, tokens de salida y precios configurados.

### Paso 1. Crear `cost_analyzer.py`

Ejecuta:

```bash
cat > cost_analyzer.py << 'EOF'
# cost_analyzer.py
# Calcula costos y extrapola resultados al dataset completo.

from dataclasses import dataclass
from typing import List

from api_executor import APICallResult
from config import MODEL_CONFIG


@dataclass
class ModelCostSummary:
    model_key: str
    display_name: str
    provider: str
    context_window_tokens: int

    sample_calls: int = 0
    successful_calls: int = 0

    sample_input_tokens: int = 0
    sample_output_tokens: int = 0
    sample_cost_usd: float = 0.0

    avg_latency_seconds: float = 0.0
    avg_output_input_ratio: float = 0.0

    estimated_total_input_tokens: int = 0
    estimated_total_output_tokens: int = 0
    estimated_total_cost_usd: float = 0.0


def calculate_cost(input_tokens: int, output_tokens: int, model_key: str) -> float:
    """
    Calcula el costo total de input y output.
    """
    config = MODEL_CONFIG[model_key]

    input_cost = (input_tokens / 1_000_000) * config["input_price_per_1m"]
    output_cost = (output_tokens / 1_000_000) * config["output_price_per_1m"]

    return round(input_cost + output_cost, 8)


def analyze_model_results(
    model_key: str,
    results: List[APICallResult],
    all_input_tokens: List[int],
) -> ModelCostSummary:
    """
    Analiza resultados de muestra y extrapola al dataset completo.
    """
    config = MODEL_CONFIG[model_key]
    successful = [r for r in results if r.success]

    summary = ModelCostSummary(
        model_key=model_key,
        display_name=config["display_name"],
        provider=config["provider"],
        context_window_tokens=config["context_window_tokens"],
        sample_calls=len(results),
        successful_calls=len(successful),
    )

    if not successful:
        return summary

    summary.sample_input_tokens = sum(r.input_tokens for r in successful)
    summary.sample_output_tokens = sum(r.output_tokens for r in successful)
    summary.sample_cost_usd = calculate_cost(
        input_tokens=summary.sample_input_tokens,
        output_tokens=summary.sample_output_tokens,
        model_key=model_key,
    )

    summary.avg_latency_seconds = round(
        sum(r.latency_seconds for r in successful) / len(successful),
        3,
    )

    if summary.sample_input_tokens > 0:
        summary.avg_output_input_ratio = round(
            summary.sample_output_tokens / summary.sample_input_tokens,
            4,
        )

    summary.estimated_total_input_tokens = sum(all_input_tokens)
    summary.estimated_total_output_tokens = int(
        summary.estimated_total_input_tokens * summary.avg_output_input_ratio
    )

    summary.estimated_total_cost_usd = calculate_cost(
        input_tokens=summary.estimated_total_input_tokens,
        output_tokens=summary.estimated_total_output_tokens,
        model_key=model_key,
    )

    return summary
EOF
```

Qué haces en este paso:

Creas la lógica de cálculo para convertir tokens en costo estimado.

### Validación

Ejecuta:

```bash
python -c "from cost_analyzer import calculate_cost; print(calculate_cost(1000, 1000, 'gpt-4.1-mini'))"
```

Salida esperada:

Debes ver un número decimal pequeño.

Ejemplo:

```text
2e-06
```

---

## Tarea 11 — Crear el módulo de reporte

### Objetivo

Generar una tabla comparativa y exportarla a CSV.

### Paso 1. Crear `reporter.py`

Ejecuta:

```bash
cat > reporter.py << 'EOF'
# reporter.py
# Genera reporte en consola y exporta a CSV.

from typing import List
import pandas as pd

from cost_analyzer import ModelCostSummary
from config import OUTPUT_CSV


def generate_report(summaries: List[ModelCostSummary]) -> pd.DataFrame:
    """
    Convierte los resultados en un DataFrame.
    """
    rows = []

    for s in summaries:
        rows.append({
            "Proveedor": s.provider,
            "Modelo": s.display_name,
            "Ventana Contexto (tokens)": s.context_window_tokens,
            "Latencia Prom. (s)": s.avg_latency_seconds,
            "Tokens Input (muestra)": s.sample_input_tokens,
            "Tokens Output (muestra)": s.sample_output_tokens,
            "Costo Muestra (USD)": round(s.sample_cost_usd, 8),
            "Tokens Input (dataset)": s.estimated_total_input_tokens,
            "Tokens Output (dataset)": s.estimated_total_output_tokens,
            "Costo Dataset (USD)": round(s.estimated_total_cost_usd, 8),
            "Llamadas Exitosas": f"{s.successful_calls}/{s.sample_calls}",
        })

    return pd.DataFrame(rows)


def print_console_report(df: pd.DataFrame) -> None:
    """
    Imprime el reporte en consola.
    """
    print("\n" + "═" * 110)
    print("REPORTE COMPARATIVO DE COSTOS — ANÁLISIS MULTI-PROVEEDOR")
    print("═" * 110)

    pd.set_option("display.max_columns", None)
    pd.set_option("display.width", 160)
    pd.set_option("display.max_colwidth", 35)

    print(df.to_string(index=False))

    print("═" * 110)

    if df.empty:
        print("No hay resultados para analizar.")
        return

    df_numeric = df.copy()

    min_cost_idx = df_numeric["Costo Dataset (USD)"].idxmin()
    max_cost_idx = df_numeric["Costo Dataset (USD)"].idxmax()
    min_latency_idx = df_numeric["Latencia Prom. (s)"].idxmin()

    print(f"\nModelo más económico: {df.loc[min_cost_idx, 'Modelo']} (${df.loc[min_cost_idx, 'Costo Dataset (USD)']})")
    print(f"Modelo más costoso:   {df.loc[max_cost_idx, 'Modelo']} (${df.loc[max_cost_idx, 'Costo Dataset (USD)']})")
    print(f"Menor latencia:       {df.loc[min_latency_idx, 'Modelo']} ({df.loc[min_latency_idx, 'Latencia Prom. (s)']} s)")
    print("═" * 110 + "\n")


def export_to_csv(df: pd.DataFrame, filepath: str = OUTPUT_CSV) -> None:
    """
    Exporta el reporte a CSV.
    """
    df.to_csv(filepath, index=False, encoding="utf-8-sig")
    print(f"Reporte exportado correctamente: {filepath}")
EOF
```

Qué haces en este paso:

Creas el módulo que convierte los resultados en tabla, imprime la comparación y genera el archivo CSV.

### Validación

Ejecuta:

```bash
python -c "from reporter import generate_report, print_console_report, export_to_csv; print('Módulo reporter importado correctamente')"
```

Salida esperada:

```text
Módulo reporter importado correctamente
```

---

## Tarea 12 — Crear el script principal

### Objetivo

Orquestar todo el flujo: validación de entorno, carga de dataset, tokenización, llamadas a APIs, análisis y reporte.

### Paso 1. Crear `main.py`

Ejecuta:

```bash
cat > main.py << 'EOF'
# main.py
# Orquestador principal del lab.

import json
import os
import sys
import time
from dotenv import load_dotenv

from config import (
    MODEL_CONFIG,
    SAMPLE_INDICES,
    SAMPLE_SIZE,
    DATASET_FILE,
    OUTPUT_CSV,
    API_DELAY_SECONDS,
)
from tokenizer import count_input_tokens
from api_executor import PROVIDER_CALL_MAP
from cost_analyzer import analyze_model_results
from reporter import generate_report, print_console_report, export_to_csv


def validate_environment() -> bool:
    """
    Verifica que las API keys estén configuradas.
    """
    required_keys = [
        "OPENAI_API_KEY",
        "GOOGLE_API_KEY",
        "ANTHROPIC_API_KEY",
    ]

    missing = [key for key in required_keys if not os.getenv(key)]

    if missing:
        print(f"ERROR: Faltan variables de entorno: {missing}")
        print("Verifica tu archivo .env antes de continuar.")
        return False

    print("Variables de entorno verificadas correctamente.")
    return True


def load_dataset(filepath: str = DATASET_FILE) -> list:
    """
    Carga el dataset de prompts.
    """
    with open(filepath, "r", encoding="utf-8") as file:
        data = json.load(file)

    prompts = data.get("prompts", [])

    if len(prompts) == 0:
        raise ValueError("El dataset no contiene prompts.")

    return prompts


def validate_dataset(prompts: list) -> None:
    """
    Valida estructura mínima del dataset.
    """
    required_fields = {"id", "category", "text"}

    for prompt in prompts:
        missing = required_fields - set(prompt.keys())
        if missing:
            raise ValueError(f"Prompt inválido. Faltan campos: {missing}")

    print(f"Dataset validado correctamente: {len(prompts)} prompts.")


def show_dataset_distribution(prompts: list) -> None:
    """
    Muestra la distribución por categoría.
    """
    distribution = {}

    for prompt in prompts:
        category = prompt["category"]
        distribution[category] = distribution.get(category, 0) + 1

    print(f"Distribución por categoría: {distribution}")


def tokenize_full_dataset(prompts: list) -> dict:
    """
    Cuenta tokens de entrada para todo el dataset por modelo.
    """
    print("\nTokenizando dataset completo...")

    token_counts = {model_key: [] for model_key in MODEL_CONFIG}

    for index, prompt_data in enumerate(prompts, start=1):
        text = prompt_data["text"]

        for model_key in MODEL_CONFIG:
            try:
                count = count_input_tokens(text=text, model_key=model_key)
            except Exception as e:
                print(f"Error tokenizando modelo {model_key}, prompt {prompt_data['id']}: {e}")
                count = 0

            token_counts[model_key].append(count)

        if index % 5 == 0:
            print(f"Procesados {index}/{len(prompts)} prompts.")

    print("Tokenización completada.")
    return token_counts


def run_api_sample(prompts: list) -> dict:
    """
    Ejecuta la muestra de prompts contra cada modelo configurado.
    """
    sample_prompts = [prompts[index] for index in SAMPLE_INDICES]
    results = {model_key: [] for model_key in MODEL_CONFIG}

    total_calls = SAMPLE_SIZE * len(MODEL_CONFIG)

    print("\nEjecutando llamadas reales a APIs...")
    print(f"Prompts seleccionados: {[p['id'] for p in sample_prompts]}")
    print(f"Total de llamadas reales: {total_calls}")

    for model_key, config in MODEL_CONFIG.items():
        provider = config["provider"]
        call_fn = PROVIDER_CALL_MAP[provider]

        print(f"\nProcesando modelo: {config['display_name']} ({provider})")

        for prompt_data in sample_prompts:
            result = call_fn(
                prompt=prompt_data["text"],
                model_key=model_key,
                prompt_id=prompt_data["id"],
            )

            results[model_key].append(result)

            status = "OK" if result.success else "ERROR"

            print(
                f"{status} | Prompt ID {result.prompt_id} | "
                f"Input: {result.input_tokens} | "
                f"Output: {result.output_tokens} | "
                f"Latencia: {result.latency_seconds}s"
            )

            if not result.success:
                print(f"Detalle del error: {result.error_message[:200]}")

            time.sleep(API_DELAY_SECONDS)

    return results


def analyze_all_models(api_results: dict, token_counts: dict) -> list:
    """
    Analiza resultados para todos los modelos.
    """
    print("\nCalculando costos y extrapolaciones...")

    summaries = []

    for model_key in MODEL_CONFIG:
        summary = analyze_model_results(
            model_key=model_key,
            results=api_results[model_key],
            all_input_tokens=token_counts[model_key],
        )

        summaries.append(summary)

        print(
            f"{summary.display_name}: "
            f"${summary.estimated_total_cost_usd:.8f} USD estimados para dataset completo"
        )

    return summaries


def main() -> None:
    """
    Ejecuta el pipeline completo.
    """
    print("\n" + "═" * 70)
    print("LAB 01-00-01 — Comparador de costos entre modelos comerciales")
    print("═" * 70)

    load_dotenv()

    if not validate_environment():
        sys.exit(1)

    print("\nCargando dataset...")
    prompts = load_dataset()
    validate_dataset(prompts)
    show_dataset_distribution(prompts)

    token_counts = tokenize_full_dataset(prompts)

    api_results = run_api_sample(prompts)

    summaries = analyze_all_models(
        api_results=api_results,
        token_counts=token_counts,
    )

    df_report = generate_report(summaries)
    print_console_report(df_report)
    export_to_csv(df_report, OUTPUT_CSV)

    print("\nLab completado correctamente.")


if __name__ == "__main__":
    main()
EOF
```

Qué haces en este paso:

Creas el archivo principal que ejecuta todo el pipeline.

### Validación de sintaxis

Ejecuta:

```bash
python -m py_compile config.py tokenizer.py api_executor.py cost_analyzer.py reporter.py main.py
```

Salida esperada:

No debe aparecer ningún error.

---

# 9. Ejecución completa del lab

## Tarea 13 — Ejecutar el pipeline

### Objetivo

Ejecutar el proyecto completo y generar el reporte comparativo.

### Paso 1. Confirmar que el entorno virtual está activo

Ejecuta:

```bash
python --version
```

Qué haces en este paso:

Confirmas que Python está disponible antes de ejecutar el proyecto.

### Paso 2. Ejecutar `main.py`

Ejecuta:

```bash
python main.py
```

Qué haces en este paso:

Inicias la validación del entorno, carga del dataset, tokenización, llamadas reales, cálculo de costos y generación del reporte.

### Salida esperada aproximada

La salida exacta puede variar por modelo, disponibilidad, latencia, tokens generados, límites de cuenta y cambios en APIs.

Debes ver una salida similar a esta:

```text
══════════════════════════════════════════════════════════════════════
LAB 01-00-01 — Comparador de costos entre modelos comerciales
══════════════════════════════════════════════════════════════════════
Variables de entorno verificadas correctamente.

Cargando dataset...
Dataset validado correctamente: 20 prompts.
Distribución por categoría: {'corto': 5, 'mediano': 7, 'largo': 8}

Tokenizando dataset completo...
Procesados 5/20 prompts.
Procesados 10/20 prompts.
Procesados 15/20 prompts.
Procesados 20/20 prompts.
Tokenización completada.

Ejecutando llamadas reales a APIs...
Prompts seleccionados: [1, 6, 10, 14, 19]
Total de llamadas reales: 30

Procesando modelo: GPT-4.1 (OpenAI)
OK | Prompt ID 1 | Input: 10 | Output: 20 | Latencia: 1.231s
...

Calculando costos y extrapolaciones...
GPT-4.1: $0.00123456 USD estimados para dataset completo
...

REPORTE COMPARATIVO DE COSTOS — ANÁLISIS MULTI-PROVEEDOR
...

Reporte exportado correctamente: cost_comparison_report.csv

Lab completado correctamente.
```

---

# 10. Validación final de funcionalidad

## Tarea 14 — Validar archivos del proyecto

### Objetivo

Confirmar que todos los archivos requeridos existen.

### Paso 1. Ejecutar validación

Ejecuta:

```bash
python -c "import os; required=['.env','.gitignore','requirements.txt','prompts_dataset.json','config.py','tokenizer.py','api_executor.py','cost_analyzer.py','reporter.py','main.py']; [print(('OK' if os.path.exists(f) else 'FALTA'), f) for f in required]"
```

Resultado esperado:

Todos los archivos deben aparecer con `OK`.

---

## Tarea 15 — Validar dataset

### Objetivo

Confirmar que el dataset tiene exactamente 20 prompts.

### Paso 1. Ejecutar validación

```bash
python -c "import json; data=json.load(open('prompts_dataset.json', encoding='utf-8')); assert len(data['prompts']) == 20; print('Dataset válido con 20 prompts')"
```

Resultado esperado:

```text
Dataset válido con 20 prompts
```

---

## Tarea 16 — Validar CSV generado

### Objetivo

Confirmar que el reporte fue creado y tiene una fila por modelo.

### Paso 1. Ejecutar validación

```bash
python -c "import pandas as pd; df=pd.read_csv('cost_comparison_report.csv'); print(df[['Proveedor','Modelo','Costo Dataset (USD)','Latencia Prom. (s)']].to_string(index=False)); assert len(df) == 6; print('CSV válido con 6 modelos')"
```

Resultado esperado:

Debes ver la tabla con 6 modelos y el mensaje:

```text
CSV válido con 6 modelos
```

---

## Tarea 17 — Validar que no hay credenciales en código

### Objetivo

Verificar que no escribiste API keys directamente en archivos `.py`.

### Paso 1. Ejecutar validación

```bash
python -c "import glob, re; pattern=re.compile(r'(sk-|AIza|sk-ant-)'); found=False; [print('ALERTA:', f) for f in glob.glob('*.py') if pattern.search(open(f, encoding='utf-8').read())]; print('Revisión completada')"
```

Resultado esperado:

No debe aparecer ningún archivo después de `ALERTA`.

---

# 11. Interpretación de resultados

Después de ejecutar el lab, revisa estas columnas del CSV:

| Columna | Qué significa |
|---|---|
| Proveedor | Empresa que ofrece el modelo |
| Modelo | Modelo evaluado |
| Ventana Contexto (tokens) | Límite aproximado de contexto del modelo |
| Latencia Prom. (s) | Tiempo promedio de respuesta en la muestra |
| Tokens Input (muestra) | Tokens de entrada usados en los prompts enviados |
| Tokens Output (muestra) | Tokens generados por el modelo en la muestra |
| Costo Muestra (USD) | Costo real aproximado de la muestra |
| Tokens Input (dataset) | Tokens estimados de entrada para los 20 prompts |
| Tokens Output (dataset) | Tokens de salida extrapolados al dataset completo |
| Costo Dataset (USD) | Costo estimado para procesar todo el dataset |
| Llamadas Exitosas | Número de llamadas correctas sobre llamadas intentadas |

## Preguntas guía para análisis

Responde estas preguntas después de revisar el reporte:

1. ¿Qué modelo fue más económico?
2. ¿Qué modelo tuvo menor latencia promedio?
3. ¿Qué proveedor tuvo más llamadas exitosas?
4. ¿Qué modelo tiene mejor relación costo/latencia?
5. ¿Qué modelo elegirías para prompts cortos?
6. ¿Qué modelo elegirías para prompts largos?
7. ¿Qué modelo elegirías si tu prioridad fuera costo?
8. ¿Qué modelo elegirías si tu prioridad fuera calidad esperada?
9. ¿Cómo cambiaría el costo si procesaras 10,000 prompts diarios?
10. ¿Qué riesgos tendría usar solo costo como criterio de selección?

---

# 12. Resolución de problemas

## Problema 1 — Faltan variables de entorno

### Síntoma

Ves un error como:

```text
ERROR: Faltan variables de entorno
```

### Causa

El archivo `.env` no existe, está mal escrito o no contiene las variables requeridas.

### Solución

Verifica que `.env` tenga exactamente estos nombres:

```text
OPENAI_API_KEY
GOOGLE_API_KEY
ANTHROPIC_API_KEY
```

Después ejecuta de nuevo:

```bash
python main.py
```

---

## Problema 2 — Error de autenticación

### Síntoma

Ves errores de autenticación, permisos o API key inválida.

### Causa

La API key es incorrecta, expiró, no tiene permisos o pertenece a una cuenta sin acceso al modelo.

### Solución

1. Verifica la API key en la consola del proveedor.
2. Confirma que tienes crédito o cuota disponible.
3. Revisa que el modelo configurado esté disponible para tu cuenta.
4. Actualiza `.env`.
5. Ejecuta de nuevo el lab.

---

## Problema 3 — Error por modelo no disponible

### Síntoma

Ves un error indicando que el modelo no existe o no está disponible.

### Causa

El proveedor cambió el nombre del modelo, lo retiró o tu cuenta no tiene acceso.

### Solución

Edita `config.py` y reemplaza el modelo por uno disponible en tu cuenta.

Ejemplo:

```python
"modelo-disponible": {
    "provider": "OpenAI",
    "display_name": "Modelo disponible",
    ...
}
```

Después ejecuta:

```bash
python main.py
```

---

## Problema 4 — Rate limit o cuota excedida

### Síntoma

Ves errores como:

```text
429
rate limit
quota exceeded
resource exhausted
```

### Causa

Hiciste demasiadas llamadas en poco tiempo o tu cuenta tiene una cuota baja.

### Solución

Edita `config.py` y aumenta:

```python
API_DELAY_SECONDS = 5
MAX_RETRIES = 3
```

Después ejecuta de nuevo:

```bash
python main.py
```

Si el error continúa, reduce la muestra:

```python
SAMPLE_SIZE = 3
SAMPLE_INDICES = [0, 9, 18]
```

---

## Problema 5 — El CSV no se genera

### Síntoma

No aparece `cost_comparison_report.csv`.

### Causa

El pipeline falló antes de llegar al módulo de reporte.

### Solución

1. Revisa errores en consola.
2. Confirma que `main.py` terminó correctamente.
3. Ejecuta validación de sintaxis:

```bash
python -m py_compile config.py tokenizer.py api_executor.py cost_analyzer.py reporter.py main.py
```

4. Ejecuta de nuevo:

```bash
python main.py
```

---

## Problema 6 — Conteos de tokens diferentes entre estimación y API

### Síntoma

Los tokens estimados de OpenAI con `tiktoken` no coinciden exactamente con los tokens reportados por la API.

### Causa

`tiktoken` cuenta el texto plano, pero la API puede incluir tokens adicionales por estructura de mensaje, metadatos o formato interno.

### Solución

Para este lab, la diferencia es aceptable porque buscas estimación comparativa. Para máxima precisión, usa los tokens reales devueltos por la API en una muestra más amplia.

---

# 13. Limpieza del entorno

## Tarea 18 — Desactivar entorno virtual

### Objetivo

Salir del entorno virtual cuando termines el lab.

### Paso 1. Desactivar entorno

Ejecuta:

```bash
deactivate
```

Qué haces en este paso:

Sales del entorno virtual activo.

---

## Tarea 19 — Opcional: eliminar entorno virtual

### Objetivo

Liberar espacio si ya no necesitas las dependencias instaladas.

En Linux o macOS:

```bash
rm -rf .venv
```

En Windows PowerShell:

```powershell
Remove-Item -Recurse -Force .venv
```

Qué haces en este paso:

Eliminas el entorno virtual del proyecto.

---

## Tarea 20 — Confirmar que `.env` no se subió a Git

### Objetivo

Evitar exposición accidental de credenciales.

### Paso 1. Revisar estado de Git

Si inicializaste un repositorio, ejecuta:

```bash
git status
```

Resultado esperado:

`.env` no debe aparecer como archivo listo para commit.

---

# 14. Criterios de aceptación

La práctica se considera completada cuando tú puedes confirmar que:

- Creaste el directorio `lab-01-00-01`.
- Creaste y activaste el entorno virtual.
- Instalaste todas las dependencias desde `requirements.txt`.
- Creaste `.env` con tus API keys.
- Creaste `.gitignore` para proteger credenciales.
- Creaste `prompts_dataset.json` con 20 prompts.
- Creaste todos los módulos Python requeridos.
- Validaste la sintaxis de los archivos Python.
- Ejecutaste `python main.py` correctamente.
- Se generó `cost_comparison_report.csv`.
- El CSV contiene 6 filas, una por modelo.
- El reporte muestra tokens, latencia, costos y llamadas exitosas.
- No escribiste API keys directamente en el código fuente.

---

# 15. Resumen

En esta práctica desarrollaste un comparador modular de costos para modelos comerciales de IA generativa. Preparaste el entorno desde cero, creaste un dataset de prompts, configuraste credenciales seguras, implementaste conteo de tokens por proveedor, ejecutaste llamadas reales a OpenAI, Gemini y Anthropic, calculaste costos estimados y exportaste un reporte comparativo en CSV.

Este lab te ayuda a convertir la comparación conceptual de modelos comerciales en una herramienta práctica para tomar decisiones con datos. En lugar de elegir un modelo solo por popularidad, ahora puedes comparar costo, latencia, consumo de tokens y comportamiento real sobre un dataset específico.

---

# 16. Estructura final esperada

Al terminar, tu proyecto debe quedar así:

```text
lab-01-00-01/
├── .env
├── .gitignore
├── requirements.txt
├── prompts_dataset.json
├── config.py
├── tokenizer.py
├── api_executor.py
├── cost_analyzer.py
├── reporter.py
├── main.py
└── cost_comparison_report.csv
```
