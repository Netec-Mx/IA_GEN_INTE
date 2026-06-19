<div align="center">
            
# 🧪 Laboratorio 2. Router de Modelos con FastAPI

![Nivel](https://img.shields.io/badge/Nivel-Intermedio-blue)
![Duración](https://img.shields.io/badge/Duración-45_min-green)
![Stack](https://img.shields.io/badge/Stack-FastAPI_+_Python-purple)
![Costo](https://img.shields.io/badge/Costo_API-0_USD-lightgrey)

</div>

---

## 1. Información general

### 📌 Descripción general

En esta práctica vas a construir una API con **FastAPI** que implementa un **Router de Modelos**. Este router recibe una solicitud de chat, analiza la complejidad del prompt y decide qué tipo de modelo debería atenderlo.

La práctica se ejecuta en **Windows**, usando **Visual Studio Code** y **Git Bash**. No consumirás APIs reales de OpenAI, Gemini ni Claude; trabajarás con **mocks** para validar la arquitectura sin costo. Esto permite concentrarte en los componentes base de una solución GenAI: API, contratos, configuración, clasificación, despacho, observabilidad básica y pruebas.

El servicio implementará tres rutas principales:

| Endpoint | Método | Propósito |
|---|---|---|
| `/v1/health` | GET | Validar estado general del servicio |
| `/v1/router/status` | GET | Consultar configuración activa del router |
| `/v1/chat/complete` | POST | Enviar un prompt y recibir una respuesta simulada |

> [!NOTE]
> Esta práctica no busca conectar todavía modelos reales. El objetivo es construir una base arquitectónica sólida para que después puedas reemplazar los mocks por SDKs reales.

---

### 🧠 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Crear una API base con FastAPI siguiendo una estructura profesional de carpetas.
2. Diseñar contratos de entrada y salida usando Pydantic.
3. Implementar una capa de configuración centralizada con `pydantic-settings`.
4. Clasificar prompts según complejidad usando reglas heurísticas.
5. Implementar un despachador de modelos usando el patrón **Strategy**.
6. Simular proveedores de modelos: OpenAI, Gemini y Claude.
7. Validar endpoints con `curl` desde Git Bash.
8. Documentar riesgos técnicos y puntos de extensión en el código.
9. Probar el comportamiento de una API GenAI sin consumir APIs de pago.
10. Preparar una base lista para evolucionar hacia integración real con proveedores.

---

### ✅ Prerrequisitos

- Conocimientos básicos de Python.
- Conocimientos básicos de APIs REST.
- Familiaridad con JSON.
- Uso básico de Visual Studio Code.
- Uso básico de Git Bash.
- Haber revisado el módulo sobre componentes de una solución GenAI.
- Haber completado o revisado la práctica #1 sobre selección técnica de modelos.

---

### 🖥️ Hardware

| Recurso | Requisito mínimo |
|---|---|
| Sistema operativo | Windows 10 o Windows 11 |
| Procesador | Intel Core i5, AMD Ryzen 5 o equivalente |
| Memoria RAM | 8 GB mínimo |
| Almacenamiento | 1 GB libre |
| GPU | No requerida |
| Internet | Requerido para instalar dependencias |

---

### 🧰 Software

| Software | Uso |
|---|---|
| Visual Studio Code | Edición del proyecto |
| Git Bash | Ejecución de comandos |
| Python 3.11 o superior | Lenguaje base del laboratorio |
| pip | Instalación de dependencias |
| FastAPI | Framework de API |
| Uvicorn | Servidor ASGI |
| Pydantic v2 | Validación de contratos |
| pydantic-settings | Carga de configuración desde `.env` |
| python-dotenv | Soporte de variables de entorno |
| curl | Pruebas HTTP desde terminal |

> [!TIP]
> FastAPI recomienda crear un entorno virtual antes de instalar dependencias. Su instalación estándar puede hacerse con `pip install "fastapi[standard]"`, y Uvicorn puede instalarse también con el extra `uvicorn[standard]`.

---

### 📊 Datos de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 45 minutos |
| Complejidad | Intermedia |
| Nivel de Bloom | Crear, aplicar, analizar y validar |
| Modalidad | Individual o equipos de 2 personas |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Framework | FastAPI |
| Proveedores simulados | OpenAI, Gemini y Claude |
| Costo API estimado | 0 USD |
| Entregable principal | Proyecto FastAPI funcional |
| Entregable secundario | Evidencias de pruebas con `curl` |

---

## 2. Consideraciones para estudiantes

1. **No uses API keys reales en esta práctica.** Todo funcionará con mocks.
2. **No subas el archivo `.env` a Git.** Aunque no tendrá claves reales, se mantiene la buena práctica.
3. **Ejecuta todos los comandos desde Git Bash**, no desde PowerShell, para mantener consistencia.
4. **Mantén activo el entorno virtual** antes de ejecutar Python o instalar dependencias.
5. **Copia los archivos completos**, no fragmentos incompletos.
6. **Valida cada tarea antes de avanzar.** Si una tarea falla, las siguientes pueden fallar también.
7. **No cambies nombres de carpetas o archivos** salvo que también actualices los imports.
8. **El estado `degraded` en `/v1/health` es esperado**, porque el laboratorio usa mocks y no APIs reales.
9. **Los modelos configurados son nombres de referencia.** No se consumen APIs reales, pero representan cómo se mapearía una arquitectura productiva.
10. **La práctica prioriza arquitectura, contratos y flujo**, no calidad real de respuesta del modelo.

---

## 3. Estructura final esperada

```text
lab-02-model-router/
├── .env
├── .env.example
├── .gitignore
├── main.py
├── requirements.txt
├── config/
│   ├── __init__.py
│   └── settings.py
├── models/
│   ├── __init__.py
│   └── schemas.py
├── routers/
│   ├── __init__.py
│   └── llm_router.py
└── services/
    ├── __init__.py
    ├── complexity_classifier.py
    └── model_dispatcher.py
```

---

# Tarea 1. Preparar el proyecto local en Windows

## Objetivo de la tarea

Crear la carpeta del proyecto, abrirla en Visual Studio Code y preparar un entorno virtual de Python usando Git Bash.

## Paso 1. Crear la carpeta del laboratorio

### Descripción
Vas a crear una carpeta local donde guardarás todo el código del Router de Modelos.
- Abre GitBash y ejecuta el siguiente comando.
### Contenido

```bash
mkdir -p ~/labs-ia-gen/lab-02-model-router
cd ~/labs-ia-gen/lab-02-model-router
```

### Validación

```bash
pwd
```

Debes ver una ruta similar a:

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-02-model-router
```

### Resultado esperado
Tienes una carpeta dedicada para la práctica #2.

---

## Paso 2. Abrir el proyecto en Visual Studio Code

### Descripción
Vas a abrir la carpeta del laboratorio directamente desde la terminal.

### Contenido

```bash
code .
```

Si el comando no funciona, abre VS Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-02-model-router
```

### Validación
Confirma que VS Code muestre la carpeta `lab-02-model-router`.

### Resultado esperado
El proyecto está abierto en Visual Studio Code.

---

## Paso 3. Crear el entorno virtual

### Descripción
Vas a crear un entorno aislado para instalar dependencias sin afectar otros proyectos.

### Contenido

```bash
python -m venv .venv
source .venv/Scripts/activate
```

### Validación

```bash
python --version
which python
```

### Resultado esperado
El entorno virtual está activo y la ruta de Python apunta a `.venv`.

---

## Paso 4. Crear el archivo de dependencias

### Descripción
Vas a definir las librerías necesarias para construir la API.

### Contenido
Crea el archivo `requirements.txt` con:

```txt
fastapi[standard]
uvicorn[standard]
pydantic
pydantic-settings
python-dotenv
httpx
```

### Qué puedes ajustar
Para ambientes empresariales o cursos con control estricto de versiones, puedes congelar versiones después de instalar:

```bash
pip freeze > requirements.lock.txt
```

### Validación
Confirma que `requirements.txt` exista en la raíz del proyecto.

### Resultado esperado
Tienes declaradas las dependencias principales del laboratorio.

---

## Paso 5. Instalar dependencias

### Descripción
Vas a instalar FastAPI, Uvicorn, Pydantic y librerías de soporte.

### Contenido

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Validación

```bash
python -c "import fastapi, pydantic; print('FastAPI:', fastapi.__version__); print('Pydantic:', pydantic.__version__)"
```

### Resultado esperado
La terminal muestra las versiones instaladas de FastAPI y Pydantic sin errores.

## Prompt de apoyo

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20de%20un%20laboratorio%20de%20FastAPI%20para%20un%20Router%20de%20Modelos.%20Cre%C3%A9%20una%20carpeta%20en%20Windows%2C%20abr%C3%AD%20el%20proyecto%20en%20Visual%20Studio%20Code%2C%20cre%C3%A9%20un%20entorno%20virtual%20con%20Git%20Bash%20e%20instal%C3%A9%20dependencias%20como%20FastAPI%2C%20Uvicorn%2C%20Pydantic%20y%20pydantic-settings.)

---

# Tarea 2. Crear la estructura del proyecto

## Objetivo de la tarea

Crear los directorios y archivos base del servicio para separar responsabilidades entre configuración, modelos, rutas y servicios.

## Paso 1. Crear carpetas y archivos

### Descripción
Vas a construir la estructura inicial del proyecto FastAPI.

### Contenido

```bash
mkdir -p config models routers services

touch main.py
touch config/__init__.py config/settings.py
touch models/__init__.py models/schemas.py
touch routers/__init__.py routers/llm_router.py
touch services/__init__.py services/complexity_classifier.py services/model_dispatcher.py
touch .env .env.example .gitignore
```

### Validación

```bash
find . -maxdepth 3 -type f | sort
```

### Resultado esperado
La terminal muestra los archivos principales del proyecto.

---

## Paso 2. Crear `.gitignore`

### Descripción
Vas a evitar que archivos sensibles, temporales o del entorno virtual se suban a Git.

### Contenido
Crea el archivo `.gitignore` con este contenido:

```gitignore
# Entorno virtual
.venv/
venv/
env/

# Variables de entorno
.env
.env.*
!.env.example

# Python
__pycache__/
*.py[cod]
*.pyo
*.pyd
.Python

# Pruebas y cobertura
.pytest_cache/
htmlcov/
.coverage

# IDEs
.vscode/
.idea/
*.swp

# Logs
*.log
logs/
```

### Validación

```bash
cat .gitignore
```

Confirma que `.env` aparece en el archivo.

### Resultado esperado
El proyecto protege archivos sensibles y temporales.

---

## Paso 3. Crear `.env.example`

### Descripción
Vas a crear una plantilla de configuración segura que sí puede compartirse.

### Contenido
Crea el archivo `.env.example` con este contenido:

```env
APP_NAME="Model Router Service"
APP_VERSION="0.1.0"
ENVIRONMENT="development"

# Esta práctica usa mocks. No agregues API keys reales.
OPENAI_API_KEY=""
GEMINI_API_KEY=""
ANTHROPIC_API_KEY=""

DEFAULT_SIMPLE_MODEL="openai-fast-mock"
DEFAULT_MEDIUM_MODEL="gemini-balanced-mock"
DEFAULT_COMPLEX_MODEL="claude-reasoning-mock"

SIMPLE_MAX_TOKENS=50
MEDIUM_MAX_TOKENS=200
```

### Validación

```bash
cat .env.example
```

### Resultado esperado
Tienes una plantilla de configuración compartible.

---

## Paso 4. Crear `.env` local

### Descripción
Vas a crear el archivo real de configuración que usará la aplicación local.

### Contenido

```bash
cp .env.example .env
```

### Validación

```bash
ls -la .env .env.example
```

### Resultado esperado
Existen `.env` y `.env.example`. Solo `.env.example` debe compartirse.

## Prompt de apoyo

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20de%20un%20laboratorio%20FastAPI.%20Cre%C3%A9%20la%20estructura%20del%20proyecto%20con%20carpetas%20config%2C%20models%2C%20routers%20y%20services%2C%20agregu%C3%A9%20.gitignore%2C%20.env.example%20y%20.env%20para%20configurar%20un%20Router%20de%20Modelos%20con%20mocks.)

---

# Tarea 3. Implementar configuración centralizada

## Objetivo de la tarea

Crear una capa de configuración tipada que cargue variables desde `.env` usando `pydantic-settings`.

## Paso 1. Crear `config/settings.py`

### Descripción
Vas a centralizar la configuración del servicio en una clase validada.

### Contenido
Abre `config/settings.py` y agrega:

```python
from functools import lru_cache
from pydantic import Field, model_validator
from pydantic_settings import BaseSettings, SettingsConfigDict


class RouterSettings(BaseSettings):
    """Configuración centralizada del Router de Modelos."""

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
        extra="ignore",
    )

    app_name: str = Field(default="Model Router Service")
    app_version: str = Field(default="0.1.0")
    environment: str = Field(default="development")

    openai_api_key: str = Field(default="")
    gemini_api_key: str = Field(default="")
    anthropic_api_key: str = Field(default="")

    default_simple_model: str = Field(default="openai-fast-mock")
    default_medium_model: str = Field(default="gemini-balanced-mock")
    default_complex_model: str = Field(default="claude-reasoning-mock")

    simple_max_tokens: int = Field(default=50, ge=1)
    medium_max_tokens: int = Field(default=200, ge=1)

    @model_validator(mode="after")
    def validate_thresholds(self) -> "RouterSettings":
        if self.medium_max_tokens <= self.simple_max_tokens:
            raise ValueError("MEDIUM_MAX_TOKENS debe ser mayor que SIMPLE_MAX_TOKENS")
        return self

    @property
    def is_mock_mode(self) -> bool:
        """Retorna True cuando no hay API keys reales configuradas."""
        return not any([
            self.openai_api_key,
            self.gemini_api_key,
            self.anthropic_api_key,
        ])


@lru_cache(maxsize=1)
def get_settings() -> RouterSettings:
    """Devuelve una única instancia cacheada de configuración."""
    return RouterSettings()
```

### Qué puedes ajustar
Puedes modificar estos valores en `.env`:

```env
DEFAULT_SIMPLE_MODEL="openai-fast-mock"
DEFAULT_MEDIUM_MODEL="gemini-balanced-mock"
DEFAULT_COMPLEX_MODEL="claude-reasoning-mock"
SIMPLE_MAX_TOKENS=50
MEDIUM_MAX_TOKENS=200
```

### Validación

```bash
python -c "from config.settings import get_settings; s=get_settings(); print(s.app_name, s.environment, s.is_mock_mode)"
```

### Resultado esperado

```text
Model Router Service development True
```

## Prompt de apoyo

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20de%20un%20laboratorio%20FastAPI.%20Implement%C3%A9%20una%20configuraci%C3%B3n%20centralizada%20con%20pydantic-settings%2C%20cargu%C3%A9%20variables%20desde%20.env%2C%20valid%C3%A9%20umbrales%20de%20clasificaci%C3%B3n%20y%20detect%C3%A9%20si%20el%20servicio%20opera%20en%20modo%20mock.)

---

# Tarea 4. Definir contratos de API con Pydantic

## Objetivo de la tarea

Crear los modelos de datos que validarán las solicitudes, respuestas y decisiones internas del Router de Modelos.

## Paso 1. Crear `models/schemas.py`

### Descripción
Vas a definir los contratos de entrada y salida del servicio.

### Contenido
Abre `models/schemas.py` y agrega:

```python
from datetime import datetime, timezone
from enum import Enum
from typing import Literal
from pydantic import BaseModel, Field, field_validator


class ComplexityLevel(str, Enum):
    SIMPLE = "SIMPLE"
    MEDIUM = "MEDIUM"
    COMPLEX = "COMPLEX"


class ModelProvider(str, Enum):
    OPENAI = "openai"
    GEMINI = "gemini"
    CLAUDE = "claude"
    MOCK = "mock"


class ChatMessage(BaseModel):
    role: Literal["system", "user", "assistant"]
    content: str = Field(min_length=1, max_length=32_000)


class ChatCompleteRequest(BaseModel):
    messages: list[ChatMessage] = Field(min_length=1)
    max_tokens: int = Field(default=512, ge=1, le=4096)
    temperature: float = Field(default=0.2, ge=0.0, le=2.0)

    @field_validator("messages")
    @classmethod
    def validate_last_message_is_user(cls, messages: list[ChatMessage]) -> list[ChatMessage]:
        if messages[-1].role != "user":
            raise ValueError("El último mensaje debe tener role='user'.")
        return messages


class ClassificationResult(BaseModel):
    level: ComplexityLevel
    estimated_tokens: int = Field(ge=0)
    reasoning: str
    detected_keywords: list[str] = Field(default_factory=list)
    selected_model: str
    provider: ModelProvider


class RouterDecision(BaseModel):
    complexity_level: ComplexityLevel
    model_used: str
    provider: ModelProvider
    estimated_tokens: int
    routing_reasoning: str


class ChatCompleteResponse(BaseModel):
    request_id: str
    content: str
    router_decision: RouterDecision
    tokens_used: int | None = None
    latency_ms: float | None = None
    timestamp: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
    is_mock: bool = True


class RouterStatusResponse(BaseModel):
    service_name: str
    version: str
    environment: str
    mode: Literal["mock", "live"]
    configured_models: dict[str, str]
    thresholds: dict[str, int]
    total_requests_served: int


class HealthResponse(BaseModel):
    status: Literal["healthy", "degraded", "unhealthy"]
    version: str
    environment: str
    uptime_seconds: float
    checks: dict[str, bool]
```

### Qué puedes ajustar
Puedes cambiar el límite máximo de caracteres del prompt modificando:

```python
content: str = Field(min_length=1, max_length=32_000)
```

### Validación

```bash
python -c "from models.schemas import ChatCompleteRequest, ChatMessage; r=ChatCompleteRequest(messages=[ChatMessage(role='user', content='Hola')]); print(r.messages[0].content)"
```

### Resultado esperado

```text
Hola
```

---

## Paso 2. Probar validación de error

### Descripción
Vas a comprobar que Pydantic bloquee un historial mal formado.

### Contenido

```bash
python - << 'PY'
from models.schemas import ChatCompleteRequest, ChatMessage

try:
    ChatCompleteRequest(messages=[ChatMessage(role="assistant", content="Hola")])
except Exception as e:
    print("Validación funcionando")
PY
```

### Validación
Debe mostrarse el mensaje `Validación funcionando`.

### Resultado esperado
El contrato impide solicitudes donde el último mensaje no sea del usuario.

## Prompt de apoyo

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20de%20un%20laboratorio%20FastAPI.%20Defin%C3%AD%20contratos%20con%20Pydantic%20para%20mensajes%2C%20solicitudes%2C%20respuestas%2C%20decisiones%20del%20router%2C%20estado%20del%20servicio%20y%20health%20check.)

---

# Tarea 5. Implementar el clasificador de complejidad

## Objetivo de la tarea

Crear un componente que estime tokens, detecte palabras clave y clasifique el prompt como simple, medio o complejo.

## Paso 1. Crear `services/complexity_classifier.py`

### Descripción
Vas a implementar la lógica que decide la complejidad inicial de una solicitud.

### Contenido
Abre `services/complexity_classifier.py` y agrega:

```python
# services/complexity_classifier.py

"""
Clasificador de complejidad para el Router de Modelos.

Este componente analiza el último prompt del usuario y determina si debe
enrutarse a un modelo simple, medio o complejo.

Criterios usados:
1. Palabras clave de complejidad.
2. Palabras clave de complejidad media.
3. Cantidad estimada de tokens.
4. Configuración de umbrales desde .env.
"""

import re
import unicodedata

from config.settings import get_settings
from models.schemas import ClassificationResult, ComplexityLevel, ModelProvider


COMPLEX_KEYWORDS = [
    "analiza",
    "analizar",
    "compara",
    "comparar",
    "contrasta",
    "evaluar",
    "evalua",
    "evalúa",
    "diseña",
    "disena",
    "diseñar",
    "arquitectura",
    "microservicios",
    "monolitica",
    "monolítica",
    "estrategia",
    "optimiza",
    "optimizar",
    "razona",
    "razonamiento",
    "plan tecnico",
    "plan técnico",
    "paso a paso",
]

MEDIUM_KEYWORDS = [
    "resumen detallado",
    "explica",
    "explicar",
    "presentacion tecnica",
    "presentación técnica",
    "componentes",
    "solucion genai",
    "solución genai",
    "documenta",
    "describe",
]


class ComplexityClassifier:
    def __init__(self):
        self.settings = get_settings()

    def _normalize_text(self, text: str) -> str:
        """
        Normaliza texto para evitar problemas con mayúsculas y acentos.

        Ejemplo:
        'Analiza una solución técnica'
        se convierte en:
        'analiza una solucion tecnica'
        """
        text = text.lower().strip()
        text = unicodedata.normalize("NFD", text)
        text = "".join(char for char in text if unicodedata.category(char) != "Mn")
        text = re.sub(r"\s+", " ", text)
        return text

    def estimate_tokens(self, text: str) -> int:
        """
        Estimación aproximada de tokens.
        Regla simple: 1 token ≈ 4 caracteres.
        """
        clean_text = text.strip()

        if not clean_text:
            return 0

        return max(1, round(len(clean_text) / 4))

    def _contains_keyword(self, normalized_text: str, keywords: list[str]) -> list[str]:
        """
        Busca palabras clave normalizadas dentro del texto.
        """
        found = []

        for keyword in keywords:
            normalized_keyword = self._normalize_text(keyword)

            if normalized_keyword in normalized_text:
                found.append(keyword)

        return found

    def _resolve_provider(self, selected_model: str) -> ModelProvider:
        """
        Define el proveedor asociado al modelo seleccionado.

        En esta práctica se trabaja en modo mock cuando no hay API keys.
        Si existen API keys, se intenta inferir el proveedor por el nombre
        del modelo.
        """
        if self.settings.is_mock_mode:
            return ModelProvider.MOCK

        model_name = selected_model.lower()

        if "openai" in model_name or "gpt" in model_name:
            return ModelProvider.OPENAI

        if "gemini" in model_name or "google" in model_name:
            return ModelProvider.GOOGLE

        if "claude" in model_name or "anthropic" in model_name:
            return ModelProvider.ANTHROPIC

        return ModelProvider.MOCK

    def classify(self, prompt: str) -> ClassificationResult:
        normalized_prompt = self._normalize_text(prompt)
        estimated_tokens = self.estimate_tokens(prompt)

        complex_matches = self._contains_keyword(normalized_prompt, COMPLEX_KEYWORDS)
        medium_matches = self._contains_keyword(normalized_prompt, MEDIUM_KEYWORDS)

        if complex_matches:
            level = ComplexityLevel.COMPLEX
            selected_model = self.settings.default_complex_model
            reasoning = (
                "El prompt se clasificó como COMPLEX porque contiene señales "
                f"de análisis avanzado: {', '.join(complex_matches)}."
            )

        elif medium_matches:
            level = ComplexityLevel.MEDIUM
            selected_model = self.settings.default_medium_model
            reasoning = (
                "El prompt se clasificó como MEDIUM porque contiene señales "
                f"de explicación o síntesis técnica: {', '.join(medium_matches)}."
            )

        elif estimated_tokens <= self.settings.simple_max_tokens:
            level = ComplexityLevel.SIMPLE
            selected_model = self.settings.default_simple_model
            reasoning = (
                "El prompt se clasificó como SIMPLE porque es corto y no contiene "
                "señales de análisis avanzado."
            )

        elif estimated_tokens <= self.settings.medium_max_tokens:
            level = ComplexityLevel.MEDIUM
            selected_model = self.settings.default_medium_model
            reasoning = (
                "El prompt se clasificó como MEDIUM por su longitud estimada."
            )

        else:
            level = ComplexityLevel.COMPLEX
            selected_model = self.settings.default_complex_model
            reasoning = (
                "El prompt se clasificó como COMPLEX por superar el umbral de tokens."
            )

        provider = self._resolve_provider(selected_model)

        return ClassificationResult(
            level=level,
            estimated_tokens=estimated_tokens,
            reasoning=reasoning,
            detected_keywords=complex_matches + medium_matches,
            selected_model=selected_model,
            provider=provider,
        )
```

### Qué puedes ajustar
Puedes modificar las listas `COMPLEX_KEYWORDS` y `SIMPLE_KEYWORDS`. También puedes cambiar los umbrales desde `.env`.

### Validación

```bash
python - << 'PY'
from services.complexity_classifier import ComplexityClassifier

classifier = ComplexityClassifier()

for text in [
    "Hola, ¿qué es Python?",
    "Necesito un resumen detallado de los componentes de una solución GenAI para una presentación técnica.",
    "Analiza y compara una arquitectura monolítica contra microservicios para una plataforma de IA generativa."
]:
    result = classifier.classify(text)
    print(text)
    print(result.level.value, result.estimated_tokens, result.selected_model, result.provider.value)
    print(result.reasoning)
    print("---")
PY
```

### Resultado esperado
Debes observar tres clasificaciones. La tercera debe ser `COMPLEX`.

## Prompt de apoyo

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20de%20un%20laboratorio%20GenAI.%20Implement%C3%A9%20un%20clasificador%20de%20complejidad%20que%20estima%20tokens%2C%20detecta%20palabras%20clave%20y%20clasifica%20prompts%20como%20SIMPLE%2C%20MEDIUM%20o%20COMPLEX.)

---

# Tarea 6. Implementar el despachador de modelos

## Objetivo de la tarea

Crear un componente que reciba la clasificación del prompt y genere una respuesta simulada mediante una estrategia de modelo.

## Paso 1. Crear `services/model_dispatcher.py`

### Descripción
Vas a implementar el patrón **Strategy** para separar la lógica de respuesta por proveedor.

### Contenido
Abre `services/model_dispatcher.py` y agrega:

```python
import logging
import time
from abc import ABC, abstractmethod
from typing import Any
from models.schemas import ChatMessage, ClassificationResult, ComplexityLevel, ModelProvider

logger = logging.getLogger(__name__)


class ModelStrategy(ABC):
    """Interfaz común para estrategias de modelos."""

    @abstractmethod
    def execute(
        self,
        messages: list[ChatMessage],
        model_name: str,
        max_tokens: int,
        temperature: float,
        complexity_level: ComplexityLevel,
    ) -> dict[str, Any]:
        pass


class MockStrategy(ModelStrategy):
    """Estrategia mock para simular respuestas sin consumir APIs reales."""

    def execute(
        self,
        messages: list[ChatMessage],
        model_name: str,
        max_tokens: int,
        temperature: float,
        complexity_level: ComplexityLevel,
    ) -> dict[str, Any]:
        latency_by_level = {
            ComplexityLevel.SIMPLE: 120.0,
            ComplexityLevel.MEDIUM: 450.0,
            ComplexityLevel.COMPLEX: 1200.0,
        }

        provider_hint = self._provider_hint(model_name)
        last_message = messages[-1].content

        content = (
            f"[MOCK - {provider_hint}] El prompt fue clasificado como {complexity_level.value}. "
            f"El modelo simulado '{model_name}' respondería considerando un máximo de "
            f"{max_tokens} tokens y temperatura {temperature}. "
            "Esta respuesta valida el flujo arquitectónico sin consumir APIs reales. "
            f"Fragmento recibido: {last_message[:120]}"
        )

        return {
            "content": content,
            "tokens_used": None,
            "latency_ms": latency_by_level.get(complexity_level, 250.0),
        }

    @staticmethod
    def _provider_hint(model_name: str) -> str:
        lower_name = model_name.lower()
        if "openai" in lower_name:
            return "OpenAI"
        if "gemini" in lower_name:
            return "Gemini"
        if "claude" in lower_name:
            return "Claude"
        return "Generic"


class OpenAIStrategy(ModelStrategy):
    """Stub para integración futura con OpenAI."""

    def execute(self, *args: Any, **kwargs: Any) -> dict[str, Any]:
        raise NotImplementedError("La integración real con OpenAI no forma parte de esta práctica.")


class GeminiStrategy(ModelStrategy):
    """Stub para integración futura con Gemini."""

    def execute(self, *args: Any, **kwargs: Any) -> dict[str, Any]:
        raise NotImplementedError("La integración real con Gemini no forma parte de esta práctica.")


class ClaudeStrategy(ModelStrategy):
    """Stub para integración futura con Claude."""

    def execute(self, *args: Any, **kwargs: Any) -> dict[str, Any]:
        raise NotImplementedError("La integración real con Claude no forma parte de esta práctica.")


class ModelDispatcher:
    """Selecciona y ejecuta la estrategia adecuada según la clasificación."""

    def __init__(self) -> None:
        self._request_count = 0
        self._strategies = {
            ModelProvider.MOCK: MockStrategy(),
            ModelProvider.OPENAI: OpenAIStrategy(),
            ModelProvider.GEMINI: GeminiStrategy(),
            ModelProvider.CLAUDE: ClaudeStrategy(),
        }

    def dispatch(
        self,
        classification: ClassificationResult,
        messages: list[ChatMessage],
        max_tokens: int,
        temperature: float,
    ) -> dict[str, Any]:
        self._request_count += 1
        strategy = self._strategies.get(classification.provider, self._strategies[ModelProvider.MOCK])
        start = time.perf_counter()
        result = strategy.execute(
            messages=messages,
            model_name=classification.selected_model,
            max_tokens=max_tokens,
            temperature=temperature,
            complexity_level=classification.level,
        )
        if result.get("latency_ms") is None:
            result["latency_ms"] = round((time.perf_counter() - start) * 1000, 2)
        result["is_mock"] = isinstance(strategy, MockStrategy)
        return result

    @property
    def request_count(self) -> int:
        return self._request_count
```

### Qué puedes ajustar
Puedes modificar el texto del mock en `content` o la latencia simulada en `latency_by_level`.

### Validación

```bash
python - << 'PY'
from services.complexity_classifier import ComplexityClassifier
from services.model_dispatcher import ModelDispatcher
from models.schemas import ChatMessage

messages = [ChatMessage(role="user", content="Analiza una arquitectura GenAI.")]
classification = ComplexityClassifier().classify(messages[-1].content)
result = ModelDispatcher().dispatch(classification, messages, max_tokens=512, temperature=0.2)
print(result["content"])
print(result["latency_ms"])
PY
```

### Resultado esperado
La terminal muestra una respuesta simulada y una latencia en milisegundos.

## Prompt de apoyo

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20de%20un%20laboratorio%20GenAI.%20Implement%C3%A9%20un%20despachador%20de%20modelos%20con%20el%20patr%C3%B3n%20Strategy%2C%20cre%C3%A9%20una%20estrategia%20mock%20y%20dej%C3%A9%20stubs%20para%20OpenAI%2C%20Gemini%20y%20Claude.)

---

# Tarea 7. Implementar los endpoints FastAPI

## Objetivo de la tarea

Crear las rutas HTTP que permitirán validar el estado del servicio, consultar la configuración del router y enviar prompts al router.

## Paso 1. Crear `routers/llm_router.py`

### Descripción
Vas a conectar el clasificador y el despachador con endpoints HTTP.

### Contenido
Abre `routers/llm_router.py` y agrega:

```python
import logging
import time
import uuid
from fastapi import APIRouter, HTTPException, status
from config.settings import get_settings
from models.schemas import (
    ChatCompleteRequest,
    ChatCompleteResponse,
    HealthResponse,
    RouterDecision,
    RouterStatusResponse,
)
from services.complexity_classifier import ComplexityClassifier
from services.model_dispatcher import ModelDispatcher

logger = logging.getLogger(__name__)
router = APIRouter(prefix="/v1", tags=["Model Router"])
settings = get_settings()
classifier = ComplexityClassifier()
dispatcher = ModelDispatcher()
service_start_time = time.time()


@router.get("/health", response_model=HealthResponse)
async def health_check() -> HealthResponse:
    return HealthResponse(
        status="degraded" if settings.is_mock_mode else "healthy",
        version=settings.app_version,
        environment=settings.environment,
        uptime_seconds=round(time.time() - service_start_time, 2),
        checks={
            "classifier": True,
            "dispatcher": True,
            "api_keys_configured": not settings.is_mock_mode,
        },
    )


@router.get("/router/status", response_model=RouterStatusResponse)
async def router_status() -> RouterStatusResponse:
    return RouterStatusResponse(
        service_name=settings.app_name,
        version=settings.app_version,
        environment=settings.environment,
        mode="mock" if settings.is_mock_mode else "live",
        configured_models={
            "SIMPLE": settings.default_simple_model,
            "MEDIUM": settings.default_medium_model,
            "COMPLEX": settings.default_complex_model,
        },
        thresholds={
            "simple_max_tokens": settings.simple_max_tokens,
            "medium_max_tokens": settings.medium_max_tokens,
        },
        total_requests_served=dispatcher.request_count,
    )


@router.post("/chat/complete", response_model=ChatCompleteResponse)
async def chat_complete(request_body: ChatCompleteRequest) -> ChatCompleteResponse:
    request_id = str(uuid.uuid4())
    try:
        user_message = request_body.messages[-1].content
        classification = classifier.classify(user_message)
        dispatch_result = dispatcher.dispatch(
            classification=classification,
            messages=request_body.messages,
            max_tokens=request_body.max_tokens,
            temperature=request_body.temperature,
        )
        decision = RouterDecision(
            complexity_level=classification.level,
            model_used=classification.selected_model,
            provider=classification.provider,
            estimated_tokens=classification.estimated_tokens,
            routing_reasoning=classification.reasoning,
        )
        return ChatCompleteResponse(
            request_id=request_id,
            content=dispatch_result["content"],
            router_decision=decision,
            tokens_used=dispatch_result.get("tokens_used"),
            latency_ms=dispatch_result.get("latency_ms"),
            is_mock=dispatch_result.get("is_mock", True),
        )
    except Exception as error:
        logger.exception("Error procesando solicitud")
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=str(error)) from error
```

### Qué puedes ajustar
Puedes cambiar el prefijo de rutas en `APIRouter(prefix="/v1")`. Si lo cambias, actualiza también los comandos `curl`.

### Validación

```bash
python -m py_compile routers/llm_router.py
```

### Resultado esperado
El archivo compila sin errores.

## Prompt de apoyo

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%207%20de%20un%20laboratorio%20FastAPI.%20Implement%C3%A9%20endpoints%20para%20health%20check%2C%20estado%20del%20router%20y%20chat%20complete%2C%20conectando%20el%20clasificador%20de%20complejidad%20y%20el%20despachador%20de%20modelos.)

---

# Tarea 8. Implementar el punto de entrada de la API

## Objetivo de la tarea

Crear `main.py` para inicializar FastAPI, registrar rutas y habilitar logging básico.

## Paso 1. Crear `main.py`

### Descripción
Vas a ensamblar la aplicación principal.

### Contenido
Abre `main.py` y agrega:

```python
import logging
import time
import uuid
from contextlib import asynccontextmanager
from collections.abc import AsyncGenerator
from fastapi import FastAPI, Request, Response
from fastapi.middleware.cors import CORSMiddleware
from config.settings import get_settings
from routers.llm_router import router as llm_router

logging.basicConfig(level=logging.INFO, format="%(asctime)s | %(levelname)s | %(name)s | %(message)s")
logger = logging.getLogger("model_router")
settings = get_settings()


@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None, None]:
    logger.info("Iniciando %s v%s", settings.app_name, settings.app_version)
    logger.info("Entorno: %s", settings.environment)
    logger.info("Modo: %s", "mock" if settings.is_mock_mode else "live")
    yield
    logger.info("Apagando %s", settings.app_name)


app = FastAPI(
    title=settings.app_name,
    version=settings.app_version,
    description="Router de Modelos con FastAPI para clasificar prompts y simular enrutamiento a OpenAI, Gemini o Claude.",
    lifespan=lifespan,
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.middleware("http")
async def logging_middleware(request: Request, call_next) -> Response:
    request_id = str(uuid.uuid4())[:8]
    start = time.perf_counter()
    logger.info("[%s] -> %s %s", request_id, request.method, request.url.path)
    response = await call_next(request)
    latency_ms = round((time.perf_counter() - start) * 1000, 2)
    logger.info("[%s] <- %s | %sms", request_id, response.status_code, latency_ms)
    response.headers["X-Request-ID"] = request_id
    response.headers["X-Response-Time-Ms"] = str(latency_ms)
    return response


app.include_router(llm_router)


@app.get("/", tags=["Info"])
async def root() -> dict:
    return {
        "service": settings.app_name,
        "version": settings.app_version,
        "environment": settings.environment,
        "docs": "/docs",
        "health": "/v1/health",
        "router_status": "/v1/router/status",
    }
```

### Qué puedes ajustar
Puedes restringir CORS cambiando `allow_origins=["*"]` por dominios específicos.

### Validación

```bash
python -m py_compile main.py
```

### Resultado esperado
`main.py` compila sin errores.

## Prompt de apoyo

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%208%20de%20un%20laboratorio%20FastAPI.%20Cre%C3%A9%20el%20punto%20de%20entrada%20main.py%2C%20configur%C3%A9%20FastAPI%2C%20registr%C3%A9%20el%20router%2C%20agregu%C3%A9%20CORS%20y%20un%20middleware%20de%20logging%20para%20trazabilidad.)

---

# Tarea 9. Ejecutar el servidor y validar endpoints

## Objetivo de la tarea

Levantar la API localmente y validar su funcionamiento con `curl` desde Git Bash.

## Paso 1. Ejecutar el servidor

### Descripción
Vas a iniciar FastAPI usando Uvicorn.

### Contenido

```bash
uvicorn main:app --reload --port 8000
```

### Validación
La terminal debe mostrar que Uvicorn está corriendo en `http://127.0.0.1:8000`.

### Resultado esperado
El servidor está activo.

> [!IMPORTANT]
> Deja esta terminal abierta. Para ejecutar los comandos `curl`, abre una segunda terminal Git Bash en la misma carpeta y activa el entorno virtual.

---

## Paso 2. Validar endpoint raíz

### Descripción
Vas a comprobar que la aplicación responde.

### Contenido

```bash
curl -s http://localhost:8000/ | python -m json.tool
```

### Validación
La respuesta debe incluir `service`, `docs` y `health`.

### Resultado esperado
La API responde correctamente.

---

## Paso 3. Validar health check

### Descripción
Vas a consultar el estado general del servicio.

### Contenido

```bash
curl -s http://localhost:8000/v1/health | python -m json.tool
```

### Validación
Debes ver un JSON con `status: degraded`, `classifier: true`, `dispatcher: true` y `api_keys_configured: false`.

### Resultado esperado
El servicio está disponible en modo mock.

---

## Paso 4. Validar estado del router

### Descripción
Vas a revisar qué modelos simulados y umbrales están configurados.

### Contenido

```bash
curl -s http://localhost:8000/v1/router/status | python -m json.tool
```

### Validación
La respuesta debe incluir `mode: mock` y los modelos configurados para `SIMPLE`, `MEDIUM` y `COMPLEX`.

### Resultado esperado
El router muestra su configuración activa.

## Prompt de apoyo

[Explicar la Tarea 9 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%209%20de%20un%20laboratorio%20FastAPI.%20Ejecut%C3%A9%20el%20servidor%20con%20Uvicorn%20y%20valid%C3%A9%20los%20endpoints%20ra%C3%ADz%2C%20health%20check%20y%20estado%20del%20router%20usando%20curl%20desde%20Git%20Bash.)

---

# Tarea 10. Probar clasificación y enrutamiento de prompts

## Objetivo de la tarea

Enviar prompts simples, medios y complejos para validar que el router selecciona modelos simulados distintos.

## Paso 1. Probar prompt simple

### Descripción
Vas a enviar una pregunta corta que debería clasificarse como `SIMPLE`.

### Contenido

```bash
mkdir -p payloads
```
```bash
cat > payloads/simple.json << 'EOF'
{
  "messages": [
    {
      "role": "user",
      "content": "Hola, ¿qué es Python?"
    }
  ],
  "max_tokens": 256,
  "temperature": 0.2
}
EOF
```
```bash
curl -s -X POST "http://localhost:8000/v1/chat/complete" \
  -H "Content-Type: application/json" \
  --data-binary @payloads/simple.json | python -m json.tool
```

### Validación
Verifica:

```text
router_decision.complexity_level = SIMPLE
router_decision.model_used = openai-fast-mock
is_mock = true
```

### Resultado esperado
El router envía el prompt al modelo simulado simple.

---

## Paso 2. Probar prompt medio

### Descripción
Vas a enviar un prompt más largo, sin palabras clave complejas explícitas.

### Contenido

```bash
cat > payloads/medium.json << 'EOF'
{
  "messages": [
    {
      "role": "user",
      "content": "Necesito un resumen detallado de los componentes principales de una solución de inteligencia artificial generativa, incluyendo la capa de aplicación, la capa de orquestación, el modelo de lenguaje y la observabilidad básica."
    }
  ],
  "max_tokens": 512,
  "temperature": 0.2
}
EOF
```
```bash
python -m json.tool payloads/medium.json
```
```bash
curl -s -X POST "http://localhost:8000/v1/chat/complete" \
  -H "Content-Type: application/json" \
  --data-binary @payloads/medium.json | python -m json.tool
```

### Validación
Verifica que `complexity_level` sea `MEDIUM` o `COMPLEX`, según los umbrales configurados.

### Resultado esperado
El router selecciona un modelo simulado de mayor capacidad que el modelo simple.

---

## Paso 3. Probar prompt complejo

### Descripción
Vas a enviar un prompt con palabras clave de análisis y comparación.

### Contenido

```bash
cat > payloads/complex.json << 'EOF'
{
  "messages": [
    {
      "role": "user",
      "content": "Analiza y compara una arquitectura monolítica contra una arquitectura de microservicios para una plataforma de IA generativa empresarial. Incluye riesgos, ventajas, desventajas y una recomendación final."
    }
  ],
  "max_tokens": 700,
  "temperature": 0.2
}
EOF
```
```bash
python -m json.tool payloads/complex.json
```
```bash
curl -s -X POST "http://localhost:8000/v1/chat/complete" \
  -H "Content-Type: application/json" \
  --data-binary @payloads/complex.json | python -m json.tool
```

### Validación
Verifica:

```text
router_decision.complexity_level = COMPLEX
router_decision.model_used = claude-reasoning-mock
is_mock = true
```

### Resultado esperado
El router selecciona el modelo simulado complejo.

---

## Paso 4. Validar contador de solicitudes

### Descripción
Vas a confirmar que el router registra cuántas solicitudes ha procesado.

### Contenido

```bash
curl -s http://localhost:8000/v1/router/status | python -c "import sys,json; data=json.load(sys.stdin); print(data['total_requests_served'])"
```

### Validación
El número debe ser mayor o igual a `3` si ejecutaste las pruebas anteriores.

### Resultado esperado
El servicio muestra observabilidad básica de solicitudes procesadas.

## Prompt de apoyo

[Explicar la Tarea 10 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2010%20de%20un%20laboratorio%20GenAI.%20Prob%C3%A9%20un%20Router%20de%20Modelos%20con%20prompts%20simples%2C%20medios%20y%20complejos%2C%20validando%20que%20la%20API%20clasifica%20el%20prompt%20y%20selecciona%20un%20modelo%20simulado%20diferente%20seg%C3%BAn%20la%20complejidad.)

---

# Tarea 11. Validar errores y documentación automática

## Objetivo de la tarea

Comprobar que la API valida solicitudes incorrectas y expone documentación interactiva.

## Paso 1. Probar error por último mensaje inválido

### Descripción
Vas a validar que el contrato no acepte una conversación cuyo último mensaje sea del asistente.

### Contenido

```bash
curl -s -X POST http://localhost:8000/v1/chat/complete   -H "Content-Type: application/json"   -d '{
    "messages": [
      {"role": "assistant", "content": "Hola"}
    ]
  }' | python -m json.tool
```

### Validación
La respuesta debe ser un error de validación `422`.

### Resultado esperado
La API bloquea solicitudes mal formadas antes de entrar a la lógica de negocio.

---

## Paso 2. Probar error por temperatura inválida

### Descripción
Vas a comprobar que Pydantic valida rangos numéricos.

### Contenido

```bash
curl -s -X POST http://localhost:8000/v1/chat/complete   -H "Content-Type: application/json"   -d '{
    "messages": [
      {"role": "user", "content": "Hola"}
    ],
    "temperature": 5
  }' | python -m json.tool
```

### Validación
La respuesta debe indicar error porque `temperature` debe estar entre `0.0` y `2.0`.

### Resultado esperado
La API protege parámetros inválidos.

---

## Paso 3. Abrir documentación Swagger

### Descripción
Vas a revisar la documentación automática generada por FastAPI.

### Contenido
Abre en el navegador:

```text
http://localhost:8000/docs
```

### Validación
Debes ver `GET /v1/health`, `GET /v1/router/status` y `POST /v1/chat/complete`.

### Resultado esperado
La API tiene documentación interactiva disponible.

## Prompt de apoyo

[Explicar la Tarea 11 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2011%20de%20un%20laboratorio%20FastAPI.%20Valid%C3%A9%20errores%20de%20contrato%20con%20Pydantic%2C%20prob%C3%A9%20solicitudes%20inv%C3%A1lidas%20y%20revis%C3%A9%20la%20documentaci%C3%B3n%20autom%C3%A1tica%20Swagger%20generada%20por%20FastAPI.)

---

# Tarea 12. Preparar entrega y evidencias

## Objetivo de la tarea

Validar que todo el proyecto funcione y preparar los archivos que se deben entregar.

## Paso 1. Compilar archivos Python

### Descripción
Vas a revisar que no existan errores de sintaxis.

### Contenido

```bash
python -m py_compile main.py
python -m py_compile config/settings.py
python -m py_compile models/schemas.py
python -m py_compile services/complexity_classifier.py
python -m py_compile services/model_dispatcher.py
python -m py_compile routers/llm_router.py
```

### Validación
Ningún comando debe mostrar errores.

### Resultado esperado
Todos los archivos Python tienen sintaxis válida.

---

## Paso 2. Validar estructura final

### Descripción
Vas a confirmar que el proyecto contiene todos los archivos necesarios.

### Contenido

```bash
find . -maxdepth 3 -type f -not -path './.venv/*' | sort
```

### Validación
Deben aparecer `main.py`, `requirements.txt`, `config/settings.py`, `models/schemas.py`, `routers/llm_router.py`, `services/complexity_classifier.py` y `services/model_dispatcher.py`.

### Resultado esperado
La estructura del proyecto está completa.

---

## Paso 3. Guardar evidencias de ejecución

### Descripción
Vas a documentar que la API funcionó correctamente.

### Contenido
Guarda capturas o copia los resultados de:

1. `GET /v1/health`
2. `GET /v1/router/status`
3. `POST /v1/chat/complete` con prompt simple
4. `POST /v1/chat/complete` con prompt complejo
5. Vista de `http://localhost:8000/docs`

### Validación
Cada evidencia debe mostrar respuesta exitosa o validación esperada.

### Resultado esperado
Tienes evidencia suficiente para entregar la práctica.

---

## Paso 4. Verificar que `.env` no se entregue

### Descripción
Vas a confirmar que no compartes archivos sensibles.

### Contenido

```bash
git status --short
```

### Validación
El archivo `.env` no debe aparecer como archivo listo para commit.

### Resultado esperado
La entrega no contiene configuración sensible.

## Prompt de apoyo

[Explicar la Tarea 12 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2012%20de%20un%20laboratorio%20FastAPI.%20Valid%C3%A9%20la%20sintaxis%20de%20los%20archivos%20Python%2C%20revis%C3%A9%20la%20estructura%20final%2C%20guard%C3%A9%20evidencias%20de%20ejecuci%C3%B3n%20y%20confirm%C3%A9%20que%20.env%20no%20se%20entregue.)

---

## 4. Resultado final esperado

Al finalizar la práctica, debes contar con:

- Proyecto FastAPI funcional en Windows.
- Entorno virtual configurado.
- Configuración centralizada con `.env` y `pydantic-settings`.
- Contratos de API definidos con Pydantic.
- Clasificador de complejidad funcional.
- Despachador de modelos con patrón Strategy.
- Mocks para simular OpenAI, Gemini y Claude.
- Endpoints funcionales para health, status y chat.
- Pruebas realizadas con `curl`.
- Documentación Swagger disponible en `/docs`.
- Evidencias de ejecución.

---

## 5. Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del ambiente local | 10% |
| Estructura profesional del proyecto | 10% |
| Configuración centralizada y segura | 10% |
| Contratos Pydantic correctos | 15% |
| Clasificador de complejidad funcional | 15% |
| Despachador con patrón Strategy | 15% |
| Endpoints FastAPI funcionales | 15% |
| Validaciones, evidencias y documentación | 10% |
| **Total** | **100%** |

---

## 6. Errores comunes que debes evitar

| Error | Cómo evitarlo |
|---|---|
| Ejecutar comandos sin activar `.venv` | Verifica que aparezca `(.venv)` en la terminal |
| Usar `python3` en Windows y que falle | Usa `python` desde Git Bash |
| Subir `.env` a Git | Confirma que `.env` esté en `.gitignore` |
| Cambiar nombres de carpetas | Si cambias nombres, actualiza imports |
| No instalar `pydantic-settings` | Verifica que esté en `requirements.txt` |
| Usar PowerShell para comandos con `cat << EOF` | Ejecuta los comandos desde Git Bash |
| Esperar `healthy` en `/v1/health` | En modo mock, el estado correcto es `degraded` |
| No dejar corriendo Uvicorn | Mantén abierta la terminal del servidor |
| Copiar JSON con comillas incorrectas | Usa exactamente los comandos `curl` proporcionados |
| Confundir mock con modelo real | Recuerda que no se consumen APIs reales |

---

## 7. Limpieza del entorno

Cuando termines, detén el servidor con `Ctrl + C` y desactiva el entorno virtual:

```bash
deactivate
```

Si deseas volver a trabajar después:

```bash
cd ~/labs-ia-gen/lab-02-model-router
source .venv/Scripts/activate
uvicorn main:app --reload --port 8000
```

---

## 8. Fuentes oficiales de referencia

- FastAPI — Documentación oficial: https://fastapi.tiangolo.com/
- FastAPI — Instalación: https://fastapi.tiangolo.com/tutorial/
- Uvicorn — Documentación oficial: https://www.uvicorn.org/
- Pydantic — Documentación oficial: https://docs.pydantic.dev/
- pydantic-settings — Documentación oficial: https://docs.pydantic.dev/latest/concepts/pydantic_settings/
- Patrón Strategy — Refactoring Guru: https://refactoring.guru/design-patterns/strategy

---

## 9. Resumen

En esta práctica construiste una API FastAPI que representa la base arquitectónica de un **Router de Modelos** para soluciones GenAI. Implementaste configuración centralizada, contratos de datos, clasificación de prompts, despacho por estrategia, respuestas simuladas, endpoints de operación y pruebas con `curl`. Aunque no consumiste modelos reales, dejaste preparada una estructura profesional que puede evolucionar hacia integración real con OpenAI, Gemini o Claude en prácticas posteriores.
