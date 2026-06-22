<div align="center">

# 🧪 Laboratorio 3

## Cliente LLM asíncrono con respuestas estructuradas, Pydantic y reintentos

![Nivel](https://img.shields.io/badge/Nivel-Intermedio-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio vas a construir un cliente Python robusto para consumir un modelo de lenguaje desde código, forzar una salida estructurada con Pydantic, procesar incidentes técnicos de forma asíncrona y manejar errores transitorios con reintentos exponenciales. No uses datos sensibles reales ni compartas tu API key.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Cliente LLM robusto</td>
<td width="25%"><strong>⏱️ Duración</strong><br>50 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar y analizar</td>
<td width="25%"><strong>📦 Entregable</strong><br>Scripts + JSON + evidencia</td>
</tr>
</table>

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a construir un cliente Python de nivel intermedio para interactuar con un modelo de lenguaje desde una aplicación real. El caso de uso consiste en recibir descripciones de incidentes técnicos escritas en texto libre y convertirlas en reportes estructurados con campos como severidad, sistemas afectados, causa raíz, acciones recomendadas y tiempo estimado de resolución.

A diferencia de una llamada simple a un modelo, en este laboratorio vas a implementar una base más robusta para aplicaciones de IA generativa. Primero validarás datos localmente con Pydantic, después probarás concurrencia sin consumir API, luego harás una llamada real al modelo con Structured Outputs y finalmente compararás procesamiento secuencial contra procesamiento concurrente.

Durante el laboratorio vas a trabajar con:

1. Windows como sistema operativo.
2. Visual Studio Code como editor.
3. Git Bash como terminal.
4. Python y entorno virtual.
5. Variables de entorno mediante `.env`.
6. Pydantic v2 para definir contratos de salida.
7. OpenAI Python SDK con cliente asíncrono.
8. Reintentos con backoff exponencial usando `tenacity`.
9. Concurrencia controlada con `asyncio.Semaphore`.
10. Exportación de resultados a `results.json`.

> [!NOTE]
> El objetivo no es solo llamar a un LLM. El objetivo es que aprendas a construir una integración más segura, validable, mantenible y preparada para errores comunes de operación.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Preparar un proyecto Python en Windows usando VS Code y Git Bash.
2. Crear y activar un entorno virtual local.
3. Configurar variables de entorno de forma segura mediante `.env`.
4. Definir un contrato de salida usando Pydantic v2.
5. Validar datos correctos e incorrectos antes de consumir una API.
6. Implementar una prueba local simulada sin costo ni dependencia de red.
7. Implementar un cliente asíncrono con `AsyncOpenAI`.
8. Forzar respuestas estructuradas usando un modelo Pydantic como esquema.
9. Agregar reintentos exponenciales para errores transitorios de API.
10. Procesar incidentes en lote usando `asyncio.gather()`.
11. Controlar concurrencia con `asyncio.Semaphore`.
12. Comparar rendimiento entre ejecución secuencial y concurrente.
13. Exportar resultados estructurados a un archivo JSON.
14. Diagnosticar errores comunes de configuración, autenticación, validación y límites de API.
15. Documentar evidencias sin exponer secretos.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Tener conocimientos básicos de Python.
2. Saber ejecutar comandos en una terminal.
3. Conocer la diferencia entre archivo, carpeta y ruta de proyecto.
4. Comprender el concepto básico de JSON.
5. Conocer el concepto general de API key.
6. Tener instalado Visual Studio Code.
7. Tener instalado Git Bash.
8. Tener instalado Python 3.11 o superior.
9. Tener una API key activa de OpenAI.
10. Tener acceso a internet.
11. Haber revisado previamente el uso básico del SDK de OpenAI en Python.
12. Haber revisado previamente conceptos básicos de Pydantic o validación de datos.

---

### 💻 Hardware

| Recurso | Requisito mínimo |
|---|---|
| Equipo | Laptop o PC con Windows |
| Sistema operativo | Windows 10 o Windows 11 |
| Procesador | Intel Core i5, AMD Ryzen 5 o equivalente |
| Memoria RAM | 8 GB mínimo |
| Almacenamiento libre | 1 GB |
| GPU | No requerida |
| Internet | Requerido para consumir la API |

---

### 🧰 Software

| Software | Uso |
|---|---|
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Python 3.11 o superior | Ejecución de scripts |
| pip | Instalación de dependencias |
| OpenAI Python SDK | Cliente para consumir la API |
| Pydantic v2 | Validación y definición de esquemas |
| Tenacity | Reintentos con backoff exponencial |
| python-dotenv | Lectura de variables desde `.env` |
| Cuenta/API key de OpenAI | Ejecución de llamadas reales al modelo |

---

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 50 minutos |
| Complejidad | Intermedia - Alta |
| Nivel de Bloom | Aplicar y analizar |
| Modalidad | Individual o equipos de 2 personas |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Proveedor usado | OpenAI |
| Modelo sugerido | `gpt-4o-mini` o el modelo definido por el instructor |
| Entregable principal | Cliente LLM asíncrono funcional |
| Entregable secundario | `results.json` con benchmark y resultados estructurados |
| Tipo de práctica | Técnica, aplicada y progresiva |

---

## 🛡️ Consideraciones para estudiantes

Antes de comenzar, toma en cuenta lo siguiente:

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>No compartas claves ni subas `.env`.</td>
<td><strong>💸 Costo</strong><br>Cada ejecución real puede consumir saldo o cuota.</td>
<td><strong>🧪 Validación</strong><br>Ejecuta primero las pruebas locales antes de usar la API.</td>
</tr>
</table>

1. **No compartas tu API key.** Tu clave es una credencial sensible.
2. **No pegues tu API key dentro del código.** Usa siempre el archivo `.env`.
3. **No entregues el archivo `.env`.** Este archivo debe quedarse únicamente en tu equipo.
4. **Ejecuta primero la prueba simulada.** Esto te permite validar el flujo sin consumir API.
5. **Usa datos simulados.** No envíes incidentes reales, datos personales, contraseñas, tokens ni información confidencial.
6. **Controla el costo.** Aunque este laboratorio usa un modelo pequeño, cada ejecución real puede consumir cuota.
7. **No ejecutes el benchmark repetidamente sin necesidad.** Cada corrida hace varias llamadas al modelo.
8. **Reduce la concurrencia si tienes errores de límite.** Puedes cambiar `max_concurrent=3` a `max_concurrent=1`.
9. **Interpreta el benchmark con criterio.** La latencia depende de red, servicio, cuota y carga externa.
10. **No asumas que el speedup siempre será mayor que 1.** Si hay reintentos o latencia variable, puede cambiar.
11. **Valida siempre el JSON final.** El archivo `results.json` es parte de la evidencia de la práctica.
12. **Conserva los scripts como evidencia.** Entrega código, resultados y capturas si el instructor las solicita.
13. **No entregues secretos.** Puedes entregar `.py`, `requirements.txt` y `results.json`, pero no `.env`.
14. **Documenta cualquier cambio.** Si cambias modelo, concurrencia, prompt o número de incidentes, regístralo.
15. **Revisa errores con calma.** La mayoría de fallos se deben a entorno virtual inactivo, API key incorrecta o dependencias faltantes.

---

## 🔗 Fuentes oficiales que debes revisar antes de ejecutar

> [!NOTE]
> Los SDKs, modelos disponibles, límites y costos pueden cambiar. Antes de impartir o ejecutar esta práctica en grupo, confirma la disponibilidad del modelo y la versión del SDK.

| Tema | Qué revisar | Fuente sugerida |
|---|---|---|
| OpenAI Python SDK | Cliente `AsyncOpenAI`, uso del SDK y métodos disponibles | Documentación oficial de OpenAI Python SDK |
| Structured Outputs | Uso de esquemas para respuestas estructuradas | Documentación oficial de Structured Outputs |
| Rate limits | Límites por modelo y recomendaciones de reintentos | Documentación oficial de OpenAI Rate Limits |
| Pydantic v2 | `BaseModel`, `Field`, `Literal`, validaciones y serialización | Documentación oficial de Pydantic |
| Tenacity | Decoradores `@retry`, backoff exponencial y manejo de excepciones | Documentación oficial de Tenacity |
| asyncio | `async`, `await`, `gather` y `Semaphore` | Documentación oficial de Python |

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear una carpeta de trabajo en Windows, abrirla en Visual Studio Code y preparar un entorno Python aislado para desarrollar el cliente LLM.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear una carpeta local donde guardarás todos los archivos de la práctica. Esto evita mezclar dependencias, scripts y resultados con otros proyectos.

**⚙️ Contenido del paso:**  
Abre **Git Bash** y ejecuta:

```bash
mkdir -p ~/labs-ia-gen/lab-03-cliente-llm-asincrono
cd ~/labs-ia-gen/lab-03-cliente-llm-asincrono
```

**✅ Validación del paso:**  
Ejecuta:

```bash
pwd
```

Debes estar dentro de una ruta similar a:

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-03-cliente-llm-asincrono
```

**📌 Resultado esperado:**  
Tienes una carpeta dedicada para el laboratorio 3.

---

### ✅ Paso 2. Abre la carpeta en Visual Studio Code

**📝 Descripción del paso:**  
Vas a abrir el proyecto en VS Code desde Git Bash para editar los archivos del laboratorio.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
code .
```

Si el comando `code .` no funciona, abre Visual Studio Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-03-cliente-llm-asincrono
```

**✅ Validación del paso:**  
Confirma que VS Code muestre la carpeta:

```text
lab-03-cliente-llm-asincrono
```

**📌 Resultado esperado:**  
El proyecto está abierto en Visual Studio Code.

---

### ✅ Paso 3. Crea el entorno virtual de Python

**📝 Descripción del paso:**  
Vas a crear un entorno virtual para instalar las dependencias de este laboratorio sin afectar otros proyectos de Python.

**⚙️ Contenido del paso:**  
En Git Bash, dentro de la carpeta del proyecto, ejecuta:

```bash
python -m venv .venv
```

Activa el entorno virtual:

```bash
source .venv/Scripts/activate
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python --version
which python
```

Debes ver que Python se ejecuta desde la carpeta `.venv`.

**📌 Resultado esperado:**  
Tienes un entorno virtual activo para instalar dependencias de forma aislada.

---

### ✅ Paso 4. Crea el archivo de dependencias

**📝 Descripción del paso:**  
Vas a definir las librerías necesarias para implementar el cliente LLM, validar datos, cargar variables de entorno y manejar reintentos.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
requirements.txt
```

Agrega el siguiente contenido:

```txt
openai>=1.90,<2
pydantic>=2.10,<3
tenacity>=8.5,<10
python-dotenv>=1.0,<2
```

**🔧 Qué puedes ajustar:**  
Si el instructor define versiones exactas para la sesión, reemplaza los rangos por las versiones indicadas. Para aula, los rangos permiten mantener compatibilidad sin quedar amarrado a una versión demasiado antigua.

**✅ Validación del paso:**  
Ejecuta:

```bash
cat requirements.txt
```

**📌 Resultado esperado:**  
El archivo existe y contiene las dependencias del laboratorio.

---

### ✅ Paso 5. Instala las dependencias

**📝 Descripción del paso:**  
Vas a instalar las librerías necesarias dentro del entorno virtual.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -c "import openai, pydantic, tenacity, dotenv; print('Dependencias instaladas correctamente')"
```

**📌 Resultado esperado:**  
Debes ver:

```text
Dependencias instaladas correctamente
```

---

### ✅ Paso 6. Crea el archivo `.gitignore`

**📝 Descripción del paso:**  
Vas a evitar que archivos sensibles, temporales o generados se suban por accidente a un repositorio.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
cat > .gitignore << 'END_GITIGNORE'
.env
.venv/
__pycache__/
*.pyc
*.pyo
.pytest_cache/
*.log
results.json
END_GITIGNORE
```

**✅ Validación del paso:**  
Ejecuta:

```bash
cat .gitignore
```

**📌 Resultado esperado:**  
Debes ver `.env`, `.venv/` y `results.json` dentro del archivo.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20de%20un%20laboratorio%20de%20IA%20generativa.%20Prepar%C3%A9%20una%20carpeta%20en%20Windows%2C%20abr%C3%AD%20el%20proyecto%20en%20Visual%20Studio%20Code%2C%20cre%C3%A9%20un%20entorno%20virtual%20de%20Python%20desde%20Git%20Bash%20e%20instal%C3%A9%20dependencias%20para%20construir%20un%20cliente%20LLM%20as%C3%ADncrono%20con%20Pydantic%20y%20reintentos.)

---

# 🧩 Tarea 2. Configurar la API key de forma segura

## 🎯 Objetivo de la tarea

Crear un archivo `.env` para guardar la API key de OpenAI sin escribirla directamente dentro del código.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `.env`

**📝 Descripción del paso:**  
Vas a crear un archivo local para almacenar la API key y el modelo que utilizará el laboratorio.

**⚙️ Contenido del paso:**  
En la raíz del proyecto, ejecuta:

```bash
cat > .env << 'END_ENV'
OPENAI_API_KEY=pega_aqui_tu_api_key_de_openai
OPENAI_MODEL=gpt-4o-mini
END_ENV
```

**🔧 Qué debes cambiar:**  
Abre `.env` en VS Code y reemplaza:

```text
pega_aqui_tu_api_key_de_openai
```

por tu API key real.

**✅ Validación del paso:**  
Confirma que el archivo `.env` existe y que `OPENAI_API_KEY` tiene un valor real.

**📌 Resultado esperado:**  
Tienes la API key configurada localmente sin escribirla en código Python.

---

### ✅ Paso 2. Crea un script para validar variables de entorno

**📝 Descripción del paso:**  
Vas a comprobar que Python puede leer la API key desde el archivo `.env`.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
00_validar_entorno.py
```

Agrega el siguiente código:

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv("OPENAI_API_KEY")
model = os.getenv("OPENAI_MODEL")

if not api_key or api_key.startswith("pega_aqui"):
    print("Falta OPENAI_API_KEY o todavía tiene el valor de ejemplo.")
    raise SystemExit(1)

if not model:
    print("Falta OPENAI_MODEL en el archivo .env.")
    raise SystemExit(1)

print("Variables de entorno cargadas correctamente.")
print("Modelo configurado:", model)
print("No se muestra la API key por seguridad.")
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python 00_validar_entorno.py
```

**📌 Resultado esperado:**  
Debes ver:

```text
Variables de entorno cargadas correctamente.
Modelo configurado: gpt-4o-mini
No se muestra la API key por seguridad.
```

---

### ✅ Paso 3. Confirma que `.env` está protegido

**📝 Descripción del paso:**  
Vas a verificar que `.env` aparece dentro de `.gitignore`.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
grep ".env" .gitignore
```

**✅ Validación del paso:**  
El comando debe imprimir:

```text
.env
```

**📌 Resultado esperado:**  
El archivo de secretos queda protegido contra carga accidental a Git.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20de%20un%20laboratorio%20de%20IA%20generativa.%20Configur%C3%A9%20un%20archivo%20.env%20con%20OPENAI_API_KEY%20y%20OPENAI_MODEL%2C%20valid%C3%A9%20que%20Python%20pueda%20leer%20las%20variables%20de%20entorno%20y%20confirm%C3%A9%20que%20.env%20est%C3%A1%20protegido%20por%20.gitignore.)

---

# 🧩 Tarea 3. Definir el contrato de salida con Pydantic

## 🎯 Objetivo de la tarea

Crear un modelo Pydantic que defina exactamente qué estructura debe tener el reporte generado por el LLM.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo principal del cliente

**📝 Descripción del paso:**  
Vas a crear el archivo donde vivirán el modelo Pydantic, las funciones simuladas, el cliente real y el procesamiento batch.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
structured_llm_client.py
```

Agrega el siguiente código inicial:

```python
"""
Cliente LLM asíncrono con respuestas estructuradas, validación Pydantic
 y reintentos exponenciales.

Caso de uso:
Extracción de información estructurada desde incidentes técnicos.
"""

from __future__ import annotations

import asyncio
import logging
import os
from typing import Literal

from dotenv import load_dotenv
from openai import APIStatusError, AsyncOpenAI, RateLimitError
from pydantic import BaseModel, Field
from tenacity import (
    before_sleep_log,
    retry,
    retry_if_exception_type,
    stop_after_attempt,
    wait_random_exponential,
)

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
)

logger = logging.getLogger(__name__)

load_dotenv()
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile structured_llm_client.py
```

**📌 Resultado esperado:**  
El comando no debe mostrar errores.

---

### ✅ Paso 2. Agrega el modelo `IncidentReport`

**📝 Descripción del paso:**  
Vas a definir el contrato de salida. Este modelo indica qué campos debe devolver el modelo y qué reglas debe cumplir cada campo.

**⚙️ Contenido del paso:**  
Agrega este código al final de `structured_llm_client.py`:

```python
class IncidentReport(BaseModel):
    """
    Representa un reporte estructurado de un incidente técnico.
    Pydantic valida tipos, campos obligatorios y restricciones.
    """

    severity: Literal["low", "medium", "high", "critical"] = Field(
        description="Nivel de severidad del incidente."
    )

    affected_systems: list[str] = Field(
        description="Lista de sistemas, servicios o componentes afectados.",
        min_length=1,
    )

    root_cause: str = Field(
        description="Causa raíz identificada o probable del incidente.",
        min_length=10,
    )

    recommended_actions: list[str] = Field(
        description="Acciones recomendadas para resolver o mitigar el incidente.",
        min_length=1,
    )

    estimated_resolution_hours: float | None = Field(
        default=None,
        description="Tiempo estimado de resolución en horas. Usa None si no se puede estimar.",
        ge=0.0,
    )
```

**🔧 Qué puedes ajustar:**  
Puedes agregar más campos si el caso de uso lo requiere, por ejemplo `business_impact`, `owner_team` o `confidence_score`. Si agregas campos, también debes actualizar las validaciones y las pruebas.

**✅ Validación del paso:**  
Ejecuta:

```bash
python -c "from structured_llm_client import IncidentReport; print(IncidentReport.model_json_schema())"
```

**📌 Resultado esperado:**  
Debes ver un esquema JSON con los campos:

```text
severity
affected_systems
root_cause
recommended_actions
estimated_resolution_hours
```

---

### ✅ Paso 3. Crea una instancia válida del modelo

**📝 Descripción del paso:**  
Vas a comprobar que Pydantic acepta datos correctos y puede serializarlos como JSON.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python -c "
from structured_llm_client import IncidentReport

report = IncidentReport(
    severity='high',
    affected_systems=['API Gateway', 'Auth Service'],
    root_cause='Certificado SSL expirado en el balanceador de carga',
    recommended_actions=['Renovar certificado', 'Configurar alerta de expiración'],
    estimated_resolution_hours=2.5
)

print(report.model_dump_json(indent=2))
"
```

**✅ Validación del paso:**  
Revisa que el JSON se imprime correctamente.

**📌 Resultado esperado:**  
Debes ver una salida similar a:

```json
{
  "severity": "high",
  "affected_systems": [
    "API Gateway",
    "Auth Service"
  ],
  "root_cause": "Certificado SSL expirado en el balanceador de carga",
  "recommended_actions": [
    "Renovar certificado",
    "Configurar alerta de expiración"
  ],
  "estimated_resolution_hours": 2.5
}
```

---

### ✅ Paso 4. Prueba datos inválidos

**📝 Descripción del paso:**  
Vas a comprobar que Pydantic rechaza datos que no cumplen el contrato.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python -c "
from pydantic import ValidationError
from structured_llm_client import IncidentReport

try:
    IncidentReport(
        severity='CRITICAL',
        affected_systems=[],
        root_cause='Error',
        recommended_actions=[],
        estimated_resolution_hours=-1
    )
except ValidationError as e:
    print('ValidationError capturado correctamente')
    print(e)
"
```

**✅ Validación del paso:**  
Debes ver errores de validación.

**📌 Resultado esperado:**  
Debes ver el mensaje:

```text
ValidationError capturado correctamente
```

Además, deben aparecer errores asociados a campos como:

```text
severity
affected_systems
root_cause
recommended_actions
estimated_resolution_hours
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20de%20un%20laboratorio%20de%20IA%20generativa.%20Defin%C3%AD%20un%20modelo%20Pydantic%20llamado%20IncidentReport%20con%20severity%2C%20affected_systems%2C%20root_cause%2C%20recommended_actions%20y%20estimated_resolution_hours.%20Expl%C3%ADcame%20por%20qu%C3%A9%20esto%20sirve%20como%20contrato%20de%20salida%20para%20un%20LLM.)

---

# 🧩 Tarea 4. Crear pruebas locales sin consumir API

## 🎯 Objetivo de la tarea

Validar el modelo y el flujo asíncrono sin hacer llamadas reales a OpenAI. Esto permite entender la concurrencia sin costo ni dependencia de red.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega una función simulada de extracción

**📝 Descripción del paso:**  
Vas a crear una función que imita el comportamiento de una llamada LLM. La función espera un segundo y devuelve un `IncidentReport` válido.

**⚙️ Contenido del paso:**  
Agrega esto al final de `structured_llm_client.py`:

```python
async def fake_extract_incident_data(text: str) -> IncidentReport:
    """
    Simula una extracción de incidente sin consumir API.
    Esta función sirve para probar async, await, gather y Semaphore.
    """

    await asyncio.sleep(1)

    return IncidentReport(
        severity="medium",
        affected_systems=["Demo System"],
        root_cause="Causa simulada para validar el flujo asíncrono local",
        recommended_actions=[
            "Validar estructura de salida",
            "Probar procesamiento concurrente sin consumir API",
        ],
        estimated_resolution_hours=1.0,
    )
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile structured_llm_client.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Agrega una función batch simulada

**📝 Descripción del paso:**  
Vas a procesar varios incidentes al mismo tiempo usando `asyncio.gather()` y vas a limitar la concurrencia con `asyncio.Semaphore`.

**⚙️ Contenido del paso:**  
Agrega esto al final de `structured_llm_client.py`:

```python
async def process_fake_batch_async(
    incidents: list[str],
    max_concurrent: int = 3,
) -> list[IncidentReport | Exception]:
    """
    Procesa incidentes simulados de forma concurrente.
    No consume API y sirve como prueba didáctica.
    """

    semaphore = asyncio.Semaphore(max_concurrent)

    async def _process_one(index: int, text: str) -> IncidentReport | Exception:
        async with semaphore:
            logger.info("Procesando incidente simulado %d/%d", index + 1, len(incidents))

            try:
                return await fake_extract_incident_data(text)
            except Exception as exc:
                logger.error("Error en incidente simulado %d: %s", index + 1, exc)
                return exc

    tasks = [
        _process_one(index, text)
        for index, text in enumerate(incidents)
    ]

    return list(await asyncio.gather(*tasks))
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile structured_llm_client.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 3. Crea el script `test_fake_batch.py`

**📝 Descripción del paso:**  
Vas a ejecutar el batch simulado y medir el tiempo total. Si procesas 5 incidentes con concurrencia 3 y cada incidente tarda 1 segundo simulado, el tiempo esperado será cercano a 2 segundos.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
test_fake_batch.py
```

Agrega el siguiente código:

```python
"""
Prueba local de concurrencia sin consumir API.
"""

import asyncio
import time

from structured_llm_client import IncidentReport, process_fake_batch_async


SAMPLE_INCIDENTS = [
    "Incidente 1: error de autenticación.",
    "Incidente 2: latencia alta en pagos.",
    "Incidente 3: falla de certificado SSL.",
    "Incidente 4: caída de servicio de notificaciones.",
    "Incidente 5: problema en pipeline de datos.",
]


async def main() -> None:
    start = time.perf_counter()

    results = await process_fake_batch_async(
        SAMPLE_INCIDENTS,
        max_concurrent=3,
    )

    elapsed = time.perf_counter() - start

    successes = [r for r in results if isinstance(r, IncidentReport)]
    failures = [r for r in results if isinstance(r, Exception)]

    print("\n=== Resultado de prueba simulada ===")
    print(f"Total procesados : {len(results)}")
    print(f"Exitosos         : {len(successes)}")
    print(f"Fallidos         : {len(failures)}")
    print(f"Tiempo total     : {elapsed:.2f} segundos")

    for index, result in enumerate(results, start=1):
        print(f"\nIncidente #{index}")
        if isinstance(result, IncidentReport):
            print(result.model_dump_json(indent=2))
        else:
            print(f"ERROR: {result}")


if __name__ == "__main__":
    asyncio.run(main())
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python test_fake_batch.py
```

**📌 Resultado esperado:**  
Debes ver 5 resultados exitosos y un tiempo total aproximado cercano a 2 segundos.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20de%20un%20laboratorio%20de%20IA%20generativa.%20Cre%C3%A9%20una%20funci%C3%B3n%20fake_extract_incident_data%20y%20un%20batch%20simulado%20con%20asyncio.gather%20y%20asyncio.Semaphore.%20Expl%C3%ADcame%20por%20qu%C3%A9%20es%20buena%20idea%20probar%20sin%20consumir%20API%20primero.)

---

# 🧩 Tarea 5. Implementar el cliente real con OpenAI y Structured Outputs

## 🎯 Objetivo de la tarea

Crear una función asíncrona que envíe incidentes técnicos al modelo y reciba una respuesta estructurada validada por Pydantic.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega una excepción personalizada

**📝 Descripción del paso:**  
Vas a crear una excepción clara para el caso en el que el modelo no devuelva una respuesta parseable.

**⚙️ Contenido del paso:**  
Agrega esto en `structured_llm_client.py`, después del modelo `IncidentReport`:

```python
class LLMRefusalError(RuntimeError):
    """
    Se lanza cuando el modelo no devuelve una respuesta estructurada parseable.
    """
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile structured_llm_client.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Agrega una función para crear el cliente OpenAI

**📝 Descripción del paso:**  
Vas a validar que la API key exista antes de crear el cliente. Esto produce errores más claros si falta configuración.

**⚙️ Contenido del paso:**  
Agrega esto al final de `structured_llm_client.py`:

```python
def get_openai_client() -> AsyncOpenAI:
    """
    Crea un cliente AsyncOpenAI validando primero que exista OPENAI_API_KEY.
    """

    api_key = os.getenv("OPENAI_API_KEY")

    if not api_key or api_key.startswith("pega_aqui"):
        raise RuntimeError(
            "OPENAI_API_KEY no está configurada. "
            "Edita el archivo .env y coloca tu API key real."
        )

    return AsyncOpenAI(api_key=api_key)
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -c "from structured_llm_client import get_openai_client; print('Función disponible')"
```

**📌 Resultado esperado:**  
Debes ver:

```text
Función disponible
```

---

### ✅ Paso 3. Implementa la función `extract_incident_data`

**📝 Descripción del paso:**  
Vas a crear la función principal que llama al modelo. Esta función utiliza el modelo Pydantic como formato esperado de respuesta y agrega reintentos con backoff exponencial aleatorio.

**⚙️ Contenido del paso:**  
Agrega esto al final de `structured_llm_client.py`:

```python
@retry(
    wait=wait_random_exponential(multiplier=1, min=2, max=60),
    stop=stop_after_attempt(5),
    retry=retry_if_exception_type((RateLimitError, APIStatusError)),
    before_sleep=before_sleep_log(logger, logging.WARNING),
    reraise=True,
)
async def extract_incident_data(text: str) -> IncidentReport:
    """
    Extrae información estructurada desde una descripción de incidente técnico.

    La función usa:
    - Cliente asíncrono de OpenAI.
    - Structured Outputs.
    - Modelo Pydantic como contrato de salida.
    - Reintentos exponenciales para errores transitorios.
    """

    client = get_openai_client()
    model = os.getenv("OPENAI_MODEL", "gpt-4o-mini")

    response = await client.beta.chat.completions.parse(
        model=model,
        messages=[
            {
                "role": "system",
                "content": (
                    "Eres un especialista en análisis de incidentes técnicos de TI. "
                    "Extrae información estructurada de forma precisa. "
                    "Usa únicamente estos valores para severity: "
                    "low, medium, high, critical. "
                    "Si no puedes estimar estimated_resolution_hours, usa null."
                ),
            },
            {
                "role": "user",
                "content": (
                    "Extrae un reporte estructurado del siguiente incidente:\n\n"
                    f"{text}"
                ),
            },
        ],
        response_format=IncidentReport,
        temperature=0.1,
        max_tokens=500,
    )

    parsed_report = response.choices[0].message.parsed

    if parsed_report is None:
        raise LLMRefusalError(
            "El modelo no devolvió una respuesta estructurada parseable."
        )

    logger.info(
        "Extracción exitosa | severity=%s | sistemas=%d",
        parsed_report.severity,
        len(parsed_report.affected_systems),
    )

    return parsed_report
```

**🔧 Qué puedes ajustar:**  
Puedes cambiar `temperature=0.1` si quieres más variación, pero para extracción estructurada se recomienda mantener una temperatura baja. También puedes cambiar `max_tokens=500` si agregas más campos al esquema.

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile structured_llm_client.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 4. Crea una prueba de una sola llamada real

**📝 Descripción del paso:**  
Vas a probar una llamada real antes de ejecutar procesamiento batch. Esto ayuda a detectar errores de API key, modelo o conexión.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
test_single_real_call.py
```

Agrega el siguiente código:

```python
"""
Prueba de una sola llamada real al modelo.
"""

import asyncio

from structured_llm_client import extract_incident_data


async def main() -> None:
    incident = (
        "El servicio de autenticación comenzó a responder 503. "
        "Los logs muestran que la base de datos agotó el pool de conexiones. "
        "Los usuarios no pueden iniciar sesión."
    )

    report = await extract_incident_data(incident)

    print("\n=== Reporte estructurado ===")
    print(report.model_dump_json(indent=2))


if __name__ == "__main__":
    asyncio.run(main())
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python test_single_real_call.py
```

**📌 Resultado esperado:**  
Debes ver un JSON estructurado similar a:

```json
{
  "severity": "high",
  "affected_systems": [
    "servicio de autenticación",
    "base de datos"
  ],
  "root_cause": "La base de datos agotó el pool de conexiones disponible",
  "recommended_actions": [
    "Revisar el uso del pool de conexiones",
    "Identificar posibles fugas de conexiones",
    "Aumentar temporalmente el límite si es necesario"
  ],
  "estimated_resolution_hours": 2.0
}
```

> [!WARNING]
> Los valores pueden variar ligeramente porque el modelo interpreta el incidente, pero la estructura debe respetarse.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20de%20un%20laboratorio%20de%20IA%20generativa.%20Implement%C3%A9%20un%20cliente%20AsyncOpenAI%20con%20Structured%20Outputs%2C%20Pydantic%20y%20una%20funci%C3%B3n%20extract_incident_data.%20Expl%C3%ADcame%20por%20qu%C3%A9%20esto%20es%20m%C3%A1s%20robusto%20que%20pedirle%20al%20modelo%20que%20devuelva%20JSON%20manualmente.)

---

# 🧩 Tarea 6. Implementar procesamiento concurrente real

## 🎯 Objetivo de la tarea

Procesar múltiples incidentes técnicos usando concurrencia controlada para mejorar el rendimiento sin saturar la API.

---

## 🛠️ Pasos

### ✅ Paso 1. Agrega la función `process_batch_async`

**📝 Descripción del paso:**  
Vas a crear una función que procese múltiples incidentes al mismo tiempo, pero limitando la cantidad de solicitudes simultáneas.

**⚙️ Contenido del paso:**  
Agrega esto al final de `structured_llm_client.py`:

```python
async def process_batch_async(
    incidents: list[str],
    max_concurrent: int = 3,
) -> list[IncidentReport | Exception]:
    """
    Procesa incidentes de forma concurrente usando Semaphore.

    El semáforo evita enviar demasiadas solicitudes al mismo tiempo.
    Esto ayuda a reducir errores por rate limits.
    """

    semaphore = asyncio.Semaphore(max_concurrent)

    async def _process_one(index: int, text: str) -> IncidentReport | Exception:
        async with semaphore:
            logger.info("Procesando incidente real %d/%d", index + 1, len(incidents))

            try:
                return await extract_incident_data(text)
            except Exception as exc:
                logger.error(
                    "Error en incidente real %d: %s: %s",
                    index + 1,
                    type(exc).__name__,
                    exc,
                )
                return exc

    tasks = [
        _process_one(index, text)
        for index, text in enumerate(incidents)
    ]

    return list(await asyncio.gather(*tasks))
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile structured_llm_client.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Crea el archivo `test_batch.py`

**📝 Descripción del paso:**  
Vas a crear el script principal de benchmark. Este script usará 10 incidentes simulados y comparará procesamiento concurrente contra procesamiento secuencial.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
test_batch.py
```

Agrega el siguiente bloque inicial:

```python
"""
Procesamiento real de incidentes técnicos con OpenAI.
Compara ejecución concurrente contra ejecución secuencial.
"""

import asyncio
import json
import time

from structured_llm_client import (
    IncidentReport,
    extract_incident_data,
    logger,
    process_batch_async,
)


SAMPLE_INCIDENTS = [
    (
        "El servicio de autenticación comenzó a retornar errores 503. "
        "Los logs muestran que la base de datos PostgreSQL agotó las conexiones disponibles. "
        "Los usuarios activos fueron desconectados."
    ),
    (
        "El contenedor del servicio de pagos no pudo iniciar por error de memoria insuficiente. "
        "Kubernetes terminó el pod y las transacciones quedaron bloqueadas durante 23 minutos."
    ),
    (
        "El certificado SSL del dominio api.empresa.com expiró. "
        "Los clientes móviles reciben errores SSL y no pueden conectarse al backend."
    ),
    (
        "La latencia P99 del servicio de recomendaciones subió de 200ms a 4500ms. "
        "Las trazas muestran consultas lentas a Redis."
    ),
    (
        "El bucket S3 de backups dejó de recibir archivos. "
        "El script reporta AccessDenied porque las credenciales IAM expiraron."
    ),
    (
        "El proveedor externo de notificaciones push retorna error 401. "
        "La clave de API fue rotada, pero no se actualizó en producción."
    ),
    (
        "El clúster Elasticsearch está en estado yellow. "
        "Un nodo falló por daño de disco y hay shards sin asignar."
    ),
    (
        "El proceso mensual de facturación falló al generar 847 de 1200 facturas. "
        "El error ocurre cuando el campo estado o provincia está vacío."
    ),
    (
        "La API pública pasó de 1000 RPM a 45000 RPM en 20 minutos. "
        "Parece un ataque de scraping distribuido."
    ),
    (
        "La sincronización entre MySQL y BigQuery lleva 8 horas atrasada. "
        "Un mensaje malformado está bloqueando el consumer de Kafka Connect."
    ),
]
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile test_batch.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 3. Agrega las funciones de benchmark

**📝 Descripción del paso:**  
Vas a medir el tiempo de procesamiento concurrente y secuencial usando el mismo dataset.

**⚙️ Contenido del paso:**  
Agrega esto a `test_batch.py` después de `SAMPLE_INCIDENTS`:

```python
async def run_concurrent_benchmark() -> tuple[list[IncidentReport | Exception], float]:
    start = time.perf_counter()

    results = await process_batch_async(
        SAMPLE_INCIDENTS,
        max_concurrent=3,
    )

    elapsed = time.perf_counter() - start

    return results, elapsed


async def run_sequential_benchmark() -> tuple[list[IncidentReport | Exception], float]:
    start = time.perf_counter()

    results: list[IncidentReport | Exception] = []

    for index, incident in enumerate(SAMPLE_INCIDENTS):
        logger.info(
            "Procesando secuencialmente incidente %d/%d",
            index + 1,
            len(SAMPLE_INCIDENTS),
        )

        try:
            result = await extract_incident_data(incident)
            results.append(result)
        except Exception as exc:
            logger.error("Error en incidente secuencial %d: %s", index + 1, exc)
            results.append(exc)

    elapsed = time.perf_counter() - start

    return results, elapsed
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile test_batch.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 4. Agrega impresión de resultados

**📝 Descripción del paso:**  
Vas a crear una función que muestre resultados exitosos, fallidos, tiempo total, promedio por incidente y distribución de severidad.

**⚙️ Contenido del paso:**  
Agrega esto a `test_batch.py`:

```python
def print_results_summary(
    results: list[IncidentReport | Exception],
    mode: str,
    elapsed: float,
) -> None:
    successes = [r for r in results if isinstance(r, IncidentReport)]
    failures = [r for r in results if isinstance(r, Exception)]

    print(f"\n{'=' * 70}")
    print(f"  MODO: {mode.upper()}")
    print(f"{'=' * 70}")
    print(f"  Total procesados  : {len(results)}")
    print(f"  Exitosos          : {len(successes)}")
    print(f"  Fallidos          : {len(failures)}")
    print(f"  Tiempo total      : {elapsed:.2f} segundos")
    print(f"  Promedio/incidente: {elapsed / len(results):.2f} segundos")
    print(f"{'=' * 70}")

    severity_counts: dict[str, int] = {}

    for report in successes:
        severity_counts[report.severity] = severity_counts.get(report.severity, 0) + 1

    print("\n  Distribución de severidad:")

    for severity in ["critical", "high", "medium", "low"]:
        count = severity_counts.get(severity, 0)
        bar = "█" * count
        print(f"    {severity:10s} | {bar} ({count})")

    print("\n  Detalle de reportes:")

    for index, result in enumerate(results, start=1):
        print(f"\n  Incidente #{index}")

        if isinstance(result, IncidentReport):
            print(f"    severity            : {result.severity}")
            print(f"    affected_systems    : {', '.join(result.affected_systems)}")
            print(f"    root_cause          : {result.root_cause[:90]}...")
            print(f"    recommended_actions : {len(result.recommended_actions)} acciones")
            print(f"    estimated_hours     : {result.estimated_resolution_hours}")
        else:
            print(f"    ERROR: {type(result).__name__}: {result}")
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile test_batch.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 5. Agrega la función principal y exportación JSON

**📝 Descripción del paso:**  
Vas a unir todo el flujo: ejecución concurrente, ejecución secuencial, comparación de rendimiento y exportación de resultados.

**⚙️ Contenido del paso:**  
Agrega esto al final de `test_batch.py`:

```python
async def main() -> None:
    print("\n" + "=" * 70)
    print("  LABORATORIO 3: Cliente LLM asíncrono con Pydantic")
    print("  Caso de uso: extracción de incidentes técnicos")
    print("=" * 70)
    print(f"  Total de incidentes: {len(SAMPLE_INCIDENTS)}")
    print("  Concurrencia máxima: 3")
    print("=" * 70)

    print("\n[1/2] Procesamiento CONCURRENTE...")
    concurrent_results, concurrent_time = await run_concurrent_benchmark()
    print_results_summary(concurrent_results, "Concurrente", concurrent_time)

    print("\n[2/2] Procesamiento SECUENCIAL...")
    sequential_results, sequential_time = await run_sequential_benchmark()
    print_results_summary(sequential_results, "Secuencial", sequential_time)

    speedup = sequential_time / concurrent_time if concurrent_time > 0 else 0
    saved_time = sequential_time - concurrent_time

    print(f"\n{'=' * 70}")
    print("  COMPARACIÓN DE RENDIMIENTO")
    print(f"{'=' * 70}")
    print(f"  Tiempo concurrente : {concurrent_time:.2f}s")
    print(f"  Tiempo secuencial  : {sequential_time:.2f}s")
    print(f"  Speedup            : {speedup:.2f}x")
    print(f"  Ahorro de tiempo   : {saved_time:.2f}s")

    if speedup > 1.0:
        print("  Resultado          : La ejecución concurrente fue más rápida.")
    else:
        print("  Resultado          : No hubo mejora. Revisa latencia, retries o rate limits.")

    print(f"{'=' * 70}")

    output = {
        "benchmark": {
            "concurrent_seconds": round(concurrent_time, 3),
            "sequential_seconds": round(sequential_time, 3),
            "speedup_factor": round(speedup, 2),
            "saved_seconds": round(saved_time, 3),
        },
        "concurrent_results": [
            result.model_dump()
            if isinstance(result, IncidentReport)
            else {
                "error_type": type(result).__name__,
                "error_message": str(result),
            }
            for result in concurrent_results
        ],
    }

    with open("results.json", "w", encoding="utf-8") as file:
        json.dump(output, file, indent=2, ensure_ascii=False)

    print("\nResultados exportados a results.json")


if __name__ == "__main__":
    asyncio.run(main())
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python test_batch.py
```

**📌 Resultado esperado:**  
Debes ver salida similar a:

```text
MODO: CONCURRENTE
Total procesados  : 10

MODO: SECUENCIAL
Total procesados  : 10

COMPARACIÓN DE RENDIMIENTO
Speedup            : 2.10x
Resultados exportados a results.json
```

> [!NOTE]
> El speedup puede variar. Si aparecen reintentos, latencia alta o límites de API, la mejora puede ser menor.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20de%20un%20laboratorio%20de%20IA%20generativa.%20Proces%C3%A9%20incidentes%20con%20asyncio.gather%2C%20asyncio.Semaphore%2C%20un%20cliente%20AsyncOpenAI%20y%20compar%C3%A9%20procesamiento%20concurrente%20contra%20secuencial.%20Expl%C3%ADcalo%20con%20un%20enfoque%20de%20aplicaciones%20LLM%20robustas.)

---

# 🧩 Tarea 7. Validar errores de Pydantic

## 🎯 Objetivo de la tarea

Comprobar que el modelo rechaza datos inválidos y acepta datos válidos antes de confiar en respuestas de un LLM.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `test_validation.py`

**📝 Descripción del paso:**  
Vas a crear pruebas manuales para comprobar que Pydantic detecta errores de severidad, listas vacías y horas negativas.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
test_validation.py
```

Agrega el siguiente código:

```python
"""
Pruebas manuales de validación Pydantic.
"""

from pydantic import ValidationError

from structured_llm_client import IncidentReport


def test_invalid_severity() -> None:
    try:
        IncidentReport(
            severity="CRITICAL",
            affected_systems=["API"],
            root_cause="Fallo en el servicio de autenticación principal",
            recommended_actions=["Reiniciar servicio"],
            estimated_resolution_hours=1.0,
        )
        print("ERROR: severity inválida fue aceptada")
    except ValidationError as exc:
        print("OK: severity inválida fue rechazada")
        print([error["loc"] for error in exc.errors()])


def test_empty_systems() -> None:
    try:
        IncidentReport(
            severity="high",
            affected_systems=[],
            root_cause="Fallo en el servicio de autenticación principal",
            recommended_actions=["Reiniciar servicio"],
            estimated_resolution_hours=1.0,
        )
        print("ERROR: affected_systems vacío fue aceptado")
    except ValidationError as exc:
        print("OK: affected_systems vacío fue rechazado")
        print([error["loc"] for error in exc.errors()])


def test_negative_hours() -> None:
    try:
        IncidentReport(
            severity="medium",
            affected_systems=["Reportes"],
            root_cause="Timeout al generar reportes por alta carga",
            recommended_actions=["Escalar servicio"],
            estimated_resolution_hours=-2,
        )
        print("ERROR: horas negativas fueron aceptadas")
    except ValidationError as exc:
        print("OK: horas negativas fueron rechazadas")
        print([error["loc"] for error in exc.errors()])


def test_valid_report() -> None:
    report = IncidentReport(
        severity="low",
        affected_systems=["Logs"],
        root_cause="Disco de logs cerca del límite de capacidad",
        recommended_actions=["Eliminar logs antiguos", "Configurar rotación"],
        estimated_resolution_hours=None,
    )

    print("OK: reporte válido aceptado")
    print(report.model_dump_json(indent=2))


if __name__ == "__main__":
    print("\n=== Pruebas de validación Pydantic ===\n")
    test_invalid_severity()
    test_empty_systems()
    test_negative_hours()
    test_valid_report()
    print("\n=== Fin de pruebas ===")
```

**✅ Validación del paso:**  
Ejecuta:

```bash
python test_validation.py
```

**📌 Resultado esperado:**  
Debes ver mensajes similares a:

```text
OK: severity inválida fue rechazada
OK: affected_systems vacío fue rechazado
OK: horas negativas fueron rechazadas
OK: reporte válido aceptado
```

---

### ✅ Paso 2. Interpreta los errores de validación

**📝 Descripción del paso:**  
Vas a revisar qué campos fallaron y por qué. Esta interpretación es importante porque en aplicaciones reales debes saber si el problema viene del dato, del esquema o del modelo.

**⚙️ Contenido del paso:**  
Observa la salida de cada prueba y relaciona cada error con su restricción:

| Campo | Restricción |
|---|---|
| `severity` | Solo acepta `low`, `medium`, `high`, `critical` |
| `affected_systems` | Debe tener al menos un elemento |
| `root_cause` | Debe tener longitud mínima de 10 caracteres |
| `recommended_actions` | Debe tener al menos una acción |
| `estimated_resolution_hours` | Debe ser `None` o un número mayor o igual a 0 |

**✅ Validación del paso:**  
Confirma que cada prueba inválida fue rechazada por el campo correcto.

**📌 Resultado esperado:**  
Comprendes cómo Pydantic protege tu aplicación contra datos incompletos o inválidos.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%207%20de%20un%20laboratorio%20de%20IA%20generativa.%20Cre%C3%A9%20pruebas%20manuales%20para%20validar%20que%20Pydantic%20rechaza%20severity%20inv%C3%A1lida%2C%20listas%20vac%C3%ADas%20y%20horas%20negativas%2C%20y%20acepta%20un%20reporte%20correcto.)

---

# 🧩 Tarea 8. Validar el resultado final y entregar evidencias

## 🎯 Objetivo de la tarea

Confirmar que los scripts funcionan, que el archivo `results.json` contiene datos consistentes y que la práctica está lista para entrega.

---

## 🛠️ Pasos

### ✅ Paso 1. Valida que todos los scripts compilen

**📝 Descripción del paso:**  
Vas a verificar que los archivos Python no tengan errores de sintaxis.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python -m py_compile 00_validar_entorno.py
python -m py_compile structured_llm_client.py
python -m py_compile test_fake_batch.py
python -m py_compile test_single_real_call.py
python -m py_compile test_batch.py
python -m py_compile test_validation.py
```

**✅ Validación del paso:**  
Ningún comando debe mostrar errores.

**📌 Resultado esperado:**  
Todos los scripts tienen sintaxis válida.

---

### ✅ Paso 2. Valida las variables de entorno

**📝 Descripción del paso:**  
Vas a comprobar que la API key y el modelo siguen cargando correctamente.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python 00_validar_entorno.py
```

**✅ Validación del paso:**  
Debe mostrarse:

```text
Variables de entorno cargadas correctamente.
No se muestra la API key por seguridad.
```

**📌 Resultado esperado:**  
Las variables están disponibles para los scripts.

---

### ✅ Paso 3. Ejecuta la prueba simulada

**📝 Descripción del paso:**  
Vas a confirmar que el flujo asíncrono funciona sin consumir API.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python test_fake_batch.py
```

**✅ Validación del paso:**  
Deben procesarse 5 incidentes simulados.

**📌 Resultado esperado:**  
La prueba simulada termina correctamente.

---

### ✅ Paso 4. Ejecuta la prueba real individual

**📝 Descripción del paso:**  
Vas a confirmar que el cliente real puede comunicarse con OpenAI.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python test_single_real_call.py
```

**✅ Validación del paso:**  
Debe imprimirse un reporte JSON estructurado.

**📌 Resultado esperado:**  
La llamada real individual funciona correctamente.

---

### ✅ Paso 5. Ejecuta el benchmark completo

**📝 Descripción del paso:**  
Vas a procesar los 10 incidentes en modo concurrente y secuencial, y generar `results.json`.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python test_batch.py
```

**✅ Validación del paso:**  
Confirma que se genere el archivo:

```text
results.json
```

**📌 Resultado esperado:**  
Tienes un archivo JSON con benchmark y resultados estructurados.

---

### ✅ Paso 6. Valida el archivo `results.json`

**📝 Descripción del paso:**  
Vas a revisar que el archivo tenga las claves esperadas y que contenga 10 resultados concurrentes.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python -c "
import json

with open('results.json', encoding='utf-8') as file:
    data = json.load(file)

assert 'benchmark' in data
assert 'concurrent_results' in data
assert len(data['concurrent_results']) == 10

print('Validación final correcta')
print(data['benchmark'])
"
```

**✅ Validación del paso:**  
El script debe terminar sin errores.

**📌 Resultado esperado:**  
Debes ver:

```text
Validación final correcta
```

---

### ✅ Paso 7. Revisa la estructura final del proyecto

**📝 Descripción del paso:**  
Vas a confirmar que la carpeta contiene todos los archivos esperados.

**⚙️ Contenido del paso:**  
Ejecuta:

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
structured_llm_client.py
test_fake_batch.py
test_single_real_call.py
test_batch.py
test_validation.py
results.json
```

**📌 Resultado esperado:**  
La estructura final del proyecto está completa.

---

### ✅ Paso 8. Guarda evidencias sin secretos

**📝 Descripción del paso:**  
Vas a identificar qué archivos puedes entregar y cuáles no debes compartir.

**⚙️ Contenido del paso:**  
Puedes entregar:

```text
requirements.txt
00_validar_entorno.py
structured_llm_client.py
test_fake_batch.py
test_single_real_call.py
test_batch.py
test_validation.py
results.json
```

No entregues:

```text
.env
.venv/
__pycache__/
```

**✅ Validación del paso:**  
Confirma que `.env` no se incluya en tu entrega.

**📌 Resultado esperado:**  
Tienes una entrega completa y segura.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%208%20de%20un%20laboratorio%20de%20IA%20generativa.%20Valid%C3%A9%20que%20los%20scripts%20compilan%2C%20que%20las%20variables%20de%20entorno%20cargan%2C%20que%20la%20prueba%20simulada%20funciona%2C%20que%20la%20llamada%20real%20responde%2C%20que%20se%20genera%20results.json%20y%20que%20la%20entrega%20no%20incluye%20secretos.)

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Variables de entorno configuradas en `.env`.
4. Archivo `.gitignore` protegiendo secretos y archivos generados.
5. Script de validación de entorno.
6. Modelo Pydantic `IncidentReport`.
7. Pruebas locales sin consumo de API.
8. Cliente asíncrono real con `AsyncOpenAI`.
9. Función con Structured Outputs y validación Pydantic.
10. Reintentos exponenciales con `tenacity`.
11. Procesamiento batch concurrente con `asyncio.gather()`.
12. Control de concurrencia con `asyncio.Semaphore`.
13. Benchmark concurrente vs. secuencial.
14. Archivo `results.json` generado correctamente.
15. Evidencia de validaciones y resultados.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del ambiente local | 10% |
| Configuración segura de API key | 10% |
| Definición correcta del modelo Pydantic | 15% |
| Prueba local simulada funcional | 10% |
| Cliente real con OpenAI funcional | 15% |
| Manejo de reintentos y errores | 10% |
| Procesamiento concurrente funcional | 10% |
| Benchmark y exportación JSON | 10% |
| Validaciones finales y entrega segura | 10% |
| Total | 100% |

---

# ⚠️ Errores comunes que debes evitar

1. Escribir la API key directamente en el código.
2. Subir el archivo `.env` a un repositorio.
3. Ejecutar los scripts sin activar el entorno virtual.
4. Instalar dependencias fuera de `.venv`.
5. Usar un modelo que no existe o no está disponible en tu cuenta.
6. Ejecutar el benchmark repetidamente sin necesidad.
7. Usar incidentes reales con información sensible.
8. Confundir `asyncio.Semaphore` con `threading.Semaphore`.
9. Pensar que concurrencia siempre significa menor tiempo en cualquier condición.
10. Ignorar errores de rate limit o cuota.
11. No revisar el archivo `results.json`.
12. Entregar `.env` por accidente.
13. No validar Pydantic antes de consumir API.
14. Cambiar el esquema sin actualizar pruebas.
15. No documentar cambios de modelo, concurrencia o dataset.

---

# 🧯 Solución de problemas

## ❌ Error: `OPENAI_API_KEY no está configurada`

### Causa probable

El archivo `.env` no existe, está mal escrito o todavía contiene el valor de ejemplo.

### Solución

Abre `.env` y valida:

```env
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini
```

Después ejecuta:

```bash
python 00_validar_entorno.py
```

---

## ❌ Error: `ModuleNotFoundError`

### Causa probable

El entorno virtual no está activo o faltan dependencias.

### Solución

Ejecuta:

```bash
source .venv/Scripts/activate
pip install -r requirements.txt
```

---

## ❌ Error: `RateLimitError`

### Causa probable

La cuenta alcanzó un límite de solicitudes o cuota temporal.

### Solución

Reduce la concurrencia en `test_batch.py`:

```python
results = await process_batch_async(
    SAMPLE_INCIDENTS,
    max_concurrent=1,
)
```

También puedes esperar unos minutos antes de volver a ejecutar.

---

## ❌ El speedup no es mayor a 1

### Causa probable

La latencia de red, los reintentos, los límites de API o la carga del servicio afectaron el resultado.

### Solución

Revisa:

1. Si hubo errores en el batch.
2. Si se activaron reintentos.
3. Si tu conexión está estable.
4. Si `max_concurrent` es adecuado.
5. Si el modelo respondió con tiempos muy variables.

No significa automáticamente que el código esté mal.

---

## ❌ `results.json` no se genera

### Causa probable

`test_batch.py` falló antes de llegar a la sección de exportación.

### Solución

Ejecuta primero:

```bash
python test_single_real_call.py
```

Si la llamada individual funciona, vuelve a ejecutar:

```bash
python test_batch.py
```

---

# Cierre de la práctica

En este laboratorio construiste un cliente LLM más robusto que una llamada básica a un modelo. Preparaste un entorno local en Windows, configuraste credenciales de forma segura, definiste un contrato de salida con Pydantic, validaste errores localmente, probaste concurrencia sin costo, implementaste una llamada real con Structured Outputs, agregaste reintentos exponenciales y comparaste ejecución concurrente contra secuencial.

El resultado más importante es que ahora tienes una base técnica para integrar IA generativa en aplicaciones reales con mejores prácticas de ingeniería: validación, control de errores, control de concurrencia, medición de rendimiento y protección de secretos.
