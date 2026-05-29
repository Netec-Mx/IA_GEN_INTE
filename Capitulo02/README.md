# Crear el boilerplate de una arquitectura en FastAPI que implemente un "Router de Modelos" para optimizar la carga de trabajo según la complejidad del prompt.

## 1. Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 45 minutos |
| **Complejidad** | Media |
| **Nivel Bloom** | Crear |
| **Módulo** | 2 — Componentes de una Solución GenAI |
| **Lección de referencia** | 2.1 — Componentes de una Solución GenAI |
| **Costo API estimado** | $0.00 USD (se usan mocks; no se consumen APIs de pago) |

---

## 2. Descripción General

En la Lección 2.1 aprendiste que una solución GenAI productiva está compuesta por capas con responsabilidades bien delimitadas: orquestación, motor de prompts, modelo de lenguaje y observabilidad. En esta práctica materializarás esos conceptos construyendo el **boilerplate** de un servicio FastAPI que actúa como **Router de Modelos**: una capa de orquestación que clasifica la complejidad de cada prompt entrante y despacha la solicitud al LLM más adecuado según criterios de costo y capacidad.

El servicio será completamente funcional y testeable con `curl`. La integración real con APIs de pago (OpenAI, Anthropic, Google) se simula mediante mocks, lo que permite validar toda la arquitectura sin incurrir en costos.

---

## 3. Objetivos de Aprendizaje

Al finalizar esta práctica serás capaz de:

- [ ] Diseñar e implementar la estructura de archivos de un servicio FastAPI siguiendo el principio de separación de responsabilidades de la arquitectura GenAI.
- [ ] Aplicar el patrón de diseño **Strategy** para seleccionar dinámicamente el modelo LLM más adecuado según la complejidad clasificada del prompt.
- [ ] Modelar contratos de API robustos utilizando **Pydantic v2** para requests, responses y configuración interna del router.
- [ ] Documentar riesgos arquitectónicos y puntos de extensión mediante comentarios estructurados `TODO` / `RISK` en el código.
- [ ] Verificar el comportamiento del router mediante pruebas con `curl` y el endpoint de estado.

---

## 4. Prerrequisitos

### Conocimiento previo
- Python intermedio: clases, herencia, decoradores y manejo de excepciones.
- Conceptos básicos de APIs REST: métodos HTTP, códigos de estado y JSON.
- Haber revisado la Lección 2.1 (componentes de una solución GenAI).
- Conocimiento básico de FastAPI o Flask (al nivel de crear un endpoint `GET`/`POST`).

### Acceso y cuentas
- **No se requieren API keys** para esta práctica (todo es mock).
- Acceso a terminal con permisos para crear directorios e instalar paquetes Python.
- Conexión a internet para descargar dependencias de PyPI.

---

## 5. Entorno del Laboratorio

### Hardware mínimo requerido

| Recurso | Mínimo | Recomendado |
|---|---|---|
| CPU | 4 núcleos | 8 núcleos |
| RAM | 8 GB | 16 GB |
| Disco libre | 500 MB | 2 GB |
| Red | No requerida durante ejecución | — |

### Software requerido

| Paquete | Versión | Uso |
|---|---|---|
| Python | 3.11.x | Lenguaje base |
| FastAPI | 0.111.x | Framework web |
| uvicorn | 0.30.x | Servidor ASGI |
| Pydantic | 2.7.x | Validación de datos |
| python-dotenv | 1.0.x | Gestión de variables de entorno |
| httpx | 0.27.x | Cliente HTTP (para pruebas internas) |

### Configuración inicial del entorno

Ejecuta los siguientes comandos en tu terminal para preparar el entorno aislado:

```bash
# 1. Crear el directorio del proyecto
mkdir lab-02-model-router
cd lab-02-model-router

# 2. Crear y activar el entorno virtual
python3.11 -m venv .venv

# En macOS/Linux:
source .venv/bin/activate

# En Windows (PowerShell):
# .venv\Scripts\Activate.ps1

# 3. Verificar la versión de Python activa
python --version
# Esperado: Python 3.11.x

# 4. Actualizar pip
pip install --upgrade pip

# 5. Instalar dependencias
pip install "fastapi==0.111.0" "uvicorn[standard]==0.30.1" \
            "pydantic==2.7.4" "python-dotenv==1.0.1" \
            "httpx==0.27.0"

# 6. Verificar instalaciones clave
python -c "import fastapi, pydantic; print(f'FastAPI {fastapi.__version__} | Pydantic {pydantic.__version__}')"
# Esperado: FastAPI 0.111.0 | Pydantic 2.7.4
```

> ⚠️ **Seguridad de credenciales**: aunque esta práctica no usa API keys reales, crearemos el archivo `.env` y `.gitignore` desde el inicio para establecer el hábito correcto. **Nunca** agregues credenciales directamente en el código.

---

## 6. Desarrollo Paso a Paso

### Paso 1 — Crear la estructura de archivos del proyecto

**Objetivo:** Establecer la arquitectura de directorios que refleja la separación de responsabilidades de la Lección 2.1.

#### Instrucciones

1. Desde el directorio `lab-02-model-router`, ejecuta los siguientes comandos para crear la estructura completa:

```bash
# Crear la estructura de directorios
mkdir -p config routers services models

# Crear todos los archivos necesarios (vacíos por ahora)
touch main.py
touch config/__init__.py config/settings.py
touch routers/__init__.py routers/llm_router.py
touch services/__init__.py services/complexity_classifier.py services/model_dispatcher.py
touch models/__init__.py models/schemas.py
touch .env .gitignore requirements.txt
```

2. Crea el archivo `.gitignore` con el siguiente contenido:

```bash
cat > .gitignore << 'EOF'
# Entorno virtual
.venv/
venv/
env/

# Variables de entorno y secretos — NUNCA commitear API keys
.env
.env.*
!.env.example

# Python cache
__pycache__/
*.py[cod]
*.pyo
*.pyd
.Python

# Pytest
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
EOF
```

3. Crea el archivo `.env` con valores de ejemplo (sin keys reales):

```bash
cat > .env << 'EOF'
# Configuración del Router de Modelos
# En producción, estas variables contendrían las API keys reales
APP_NAME="Model Router Service"
APP_VERSION="0.1.0"
ENVIRONMENT="development"

# API Keys (vacías en esta práctica — se usan mocks)
OPENAI_API_KEY=""
ANTHROPIC_API_KEY=""
GOOGLE_API_KEY=""

# Configuración del router
DEFAULT_SIMPLE_MODEL="gpt-4o-mini"
DEFAULT_MEDIUM_MODEL="gpt-4o"
DEFAULT_COMPLEX_MODEL="claude-3-5-sonnet-20241022"

# Umbrales de clasificación (en tokens aproximados)
SIMPLE_MAX_TOKENS=50
MEDIUM_MAX_TOKENS=200
EOF
```

4. Genera el archivo `requirements.txt`:

```bash
cat > requirements.txt << 'EOF'
fastapi==0.111.0
uvicorn[standard]==0.30.1
pydantic==2.7.4
python-dotenv==1.0.1
httpx==0.27.0
EOF
```

#### Salida esperada

```
lab-02-model-router/
├── .env
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

#### Verificación

```bash
# Verificar la estructura con tree (o find si tree no está disponible)
find . -not -path './.venv/*' -not -path './__pycache__/*' | sort
```

---

### Paso 2 — Implementar la configuración centralizada (`config/settings.py`)

**Objetivo:** Crear una capa de configuración tipada con Pydantic v2 que centralice todos los parámetros del servicio, siguiendo el principio de configuración externalizada.

#### Instrucciones

1. Abre `config/settings.py` y escribe el siguiente código:

```python
# config/settings.py
"""
Configuración centralizada del Model Router Service.

Usa Pydantic BaseSettings para cargar y validar variables de entorno
automáticamente desde el archivo .env. Esto garantiza que el servicio
falle rápido (fail-fast) si falta alguna configuración crítica.

Patrón aplicado: Configuration Object (centraliza toda la config en un lugar)
"""

from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field
from functools import lru_cache


class RouterSettings(BaseSettings):
    """
    Configuración del servicio con validación automática via Pydantic v2.

    Todas las variables se cargan desde el archivo .env o variables
    de entorno del sistema operativo.
    """

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
        extra="ignore"  # Ignora variables de entorno no declaradas aquí
    )

    # --- Metadatos del servicio ---
    app_name: str = Field(default="Model Router Service", description="Nombre del servicio")
    app_version: str = Field(default="0.1.0", description="Versión semántica del servicio")
    environment: str = Field(default="development", description="Entorno de ejecución")

    # --- API Keys (vacías en modo mock) ---
    # RISK: En producción, rotar estas keys periódicamente y usar un secrets manager
    # (AWS Secrets Manager, GCP Secret Manager, HashiCorp Vault) en lugar de .env
    openai_api_key: str = Field(default="", description="API Key de OpenAI")
    anthropic_api_key: str = Field(default="", description="API Key de Anthropic")
    google_api_key: str = Field(default="", description="API Key de Google AI")

    # --- Modelos por nivel de complejidad ---
    # TODO: Hacer esta configuración dinámica vía endpoint admin para cambiar
    # modelos en caliente sin reiniciar el servicio
    default_simple_model: str = Field(
        default="gpt-4o-mini",
        description="Modelo para prompts simples (< 50 tokens)"
    )
    default_medium_model: str = Field(
        default="gpt-4o",
        description="Modelo para prompts de complejidad media (50-200 tokens)"
    )
    default_complex_model: str = Field(
        default="claude-3-5-sonnet-20241022",
        description="Modelo para prompts complejos (> 200 tokens)"
    )

    # --- Umbrales del clasificador ---
    simple_max_tokens: int = Field(
        default=50,
        ge=1,
        description="Límite superior de tokens para clasificar como SIMPLE"
    )
    medium_max_tokens: int = Field(
        default=200,
        ge=1,
        description="Límite superior de tokens para clasificar como MEDIUM"
    )

    @property
    def is_mock_mode(self) -> bool:
        """
        Retorna True si ninguna API key está configurada.
        El servicio operará con respuestas simuladas en este caso.
        """
        return not any([self.openai_api_key, self.anthropic_api_key, self.google_api_key])


@lru_cache(maxsize=1)
def get_settings() -> RouterSettings:
    """
    Retorna la instancia singleton de configuración.

    El decorador @lru_cache garantiza que el archivo .env se lea
    una sola vez durante el ciclo de vida del proceso.

    TODO: Invalidar el cache en tests unitarios usando
    get_settings.cache_clear() para inyectar configuración de prueba.
    """
    return RouterSettings()
```

2. Instala `pydantic-settings` (necesario para `BaseSettings` en Pydantic v2):

```bash
pip install "pydantic-settings==2.3.4"
echo "pydantic-settings==2.3.4" >> requirements.txt
```

#### Verificación

```bash
python -c "
from config.settings import get_settings
s = get_settings()
print(f'App: {s.app_name} v{s.app_version}')
print(f'Entorno: {s.environment}')
print(f'Modo mock: {s.is_mock_mode}')
print(f'Modelo simple: {s.default_simple_model}')
print(f'Modelo complejo: {s.default_complex_model}')
"
```

**Salida esperada:**
```
App: Model Router Service v0.1.0
Entorno: development
Modo mock: True
Modelo simple: gpt-4o-mini
Modelo complejo: claude-3-5-sonnet-20241022
```

---

### Paso 3 — Definir los esquemas Pydantic (`models/schemas.py`)

**Objetivo:** Modelar los contratos de la API con Pydantic v2. Estos esquemas son el "lenguaje común" entre la capa de presentación y la capa de orquestación.

#### Instrucciones

1. Abre `models/schemas.py` y escribe:

```python
# models/schemas.py
"""
Contratos de la API definidos con Pydantic v2.

Estos modelos actúan como la capa de validación entre la solicitud HTTP
y la lógica de negocio. Siguiendo los principios de la Lección 2.1,
cada modelo tiene una responsabilidad única y bien delimitada.

Principio aplicado: Fail-fast validation — si el dato de entrada no cumple
el contrato, Pydantic lanza un error antes de que llegue a la lógica de negocio.
"""

from pydantic import BaseModel, Field, field_validator
from typing import Optional, Literal
from enum import Enum
from datetime import datetime


# --- Enumeraciones de dominio ---

class ComplexityLevel(str, Enum):
    """
    Niveles de complejidad que el clasificador puede asignar a un prompt.
    Usar un Enum garantiza que solo valores válidos circulen por el sistema.
    """
    SIMPLE = "SIMPLE"
    MEDIUM = "MEDIUM"
    COMPLEX = "COMPLEX"


class ModelProvider(str, Enum):
    """Proveedores de LLM soportados por el router."""
    OPENAI = "openai"
    ANTHROPIC = "anthropic"
    GOOGLE = "google"
    MOCK = "mock"  # Proveedor especial para entornos sin API keys


# --- Modelos de Request ---

class ChatMessage(BaseModel):
    """Representa un turno individual en el historial de conversación."""
    role: Literal["user", "assistant", "system"]
    content: str = Field(min_length=1, max_length=32_000)


class ChatCompleteRequest(BaseModel):
    """
    Contrato de entrada para el endpoint POST /v1/chat/complete.

    El campo 'messages' sigue la convención de la API de OpenAI para
    maximizar la compatibilidad con distintos proveedores.
    """
    messages: list[ChatMessage] = Field(
        min_length=1,
        description="Lista de mensajes del historial de conversación"
    )
    max_tokens: Optional[int] = Field(
        default=512,
        ge=1,
        le=4096,
        description="Número máximo de tokens en la respuesta"
    )
    temperature: Optional[float] = Field(
        default=0.7,
        ge=0.0,
        le=2.0,
        description="Temperatura de muestreo del modelo"
    )
    # TODO: Añadir campo 'user_id' para implementar rate limiting por usuario
    # y auditoría de uso en la capa de observabilidad

    @field_validator("messages")
    @classmethod
    def validate_last_message_is_user(cls, messages: list[ChatMessage]) -> list[ChatMessage]:
        """
        Valida que el último mensaje sea del usuario.
        Esto previene errores silenciosos donde se envía un historial
        mal formado al LLM.
        """
        if messages and messages[-1].role != "user":
            raise ValueError(
                "El último mensaje del historial debe ser del rol 'user'. "
                f"Se recibió: '{messages[-1].role}'"
            )
        return messages


# --- Modelos de clasificación interna ---

class ClassificationResult(BaseModel):
    """
    Resultado interno del clasificador de complejidad.
    Este modelo circula entre la capa de orquestación y el despachador.
    No se expone directamente en la API pública.
    """
    level: ComplexityLevel
    estimated_tokens: int = Field(ge=0)
    reasoning: str = Field(description="Explicación legible de por qué se asignó este nivel")
    detected_keywords: list[str] = Field(
        default_factory=list,
        description="Palabras clave de complejidad detectadas en el prompt"
    )
    selected_model: str = Field(description="Nombre del modelo seleccionado para este nivel")
    provider: ModelProvider


# --- Modelos de Response ---

class RouterDecision(BaseModel):
    """Metadatos de la decisión de routing incluidos en la respuesta."""
    complexity_level: ComplexityLevel
    model_used: str
    provider: ModelProvider
    estimated_tokens: int
    routing_reasoning: str


class ChatCompleteResponse(BaseModel):
    """
    Contrato de salida para el endpoint POST /v1/chat/complete.

    Incluye tanto la respuesta del modelo como los metadatos de routing,
    lo que permite al cliente entender qué modelo respondió y por qué.
    """
    request_id: str = Field(description="UUID único de la solicitud para trazabilidad")
    content: str = Field(description="Texto generado por el modelo")
    router_decision: RouterDecision
    tokens_used: Optional[int] = Field(
        default=None,
        description="Tokens consumidos (None en modo mock)"
    )
    latency_ms: Optional[float] = Field(
        default=None,
        description="Latencia de la llamada al LLM en milisegundos"
    )
    timestamp: datetime = Field(default_factory=datetime.utcnow)
    is_mock: bool = Field(
        default=False,
        description="True si la respuesta fue generada por el mock (sin LLM real)"
    )


class RouterStatusResponse(BaseModel):
    """Respuesta del endpoint GET /v1/router/status."""
    service_name: str
    version: str
    environment: str
    mode: Literal["live", "mock"]
    configured_models: dict[str, str] = Field(
        description="Mapa de nivel de complejidad → modelo configurado"
    )
    thresholds: dict[str, int] = Field(
        description="Umbrales de tokens para cada nivel de complejidad"
    )
    total_requests_served: int = Field(default=0)


class HealthResponse(BaseModel):
    """Respuesta del endpoint GET /v1/health."""
    status: Literal["healthy", "degraded", "unhealthy"]
    version: str
    environment: str
    uptime_seconds: float
    checks: dict[str, bool] = Field(
        description="Estado de cada dependencia verificada"
    )
```

#### Verificación

```bash
python -c "
from models.schemas import ChatCompleteRequest, ChatMessage, ComplexityLevel
# Test: request válido
req = ChatCompleteRequest(messages=[ChatMessage(role='user', content='Hola, ¿cómo estás?')])
print(f'Request válido: {req.messages[0].content}')

# Test: enum de complejidad
for level in ComplexityLevel:
    print(f'Nivel: {level.value}')
"
```

**Salida esperada:**
```
Request válido: Hola, ¿cómo estás?
Nivel: SIMPLE
Nivel: MEDIUM
Nivel: COMPLEX
```

---

### Paso 4 — Implementar el clasificador de complejidad (`services/complexity_classifier.py`)

**Objetivo:** Construir la lógica de clasificación que analiza el prompt entrante y determina su nivel de complejidad. Este componente corresponde al **motor de prompts** de la Lección 2.1 en su rol de análisis previo.

#### Instrucciones

1. Abre `services/complexity_classifier.py` y escribe:

```python
# services/complexity_classifier.py
"""
Clasificador de complejidad de prompts.

Responsabilidad única: recibir el texto del último mensaje del usuario
y retornar un ClassificationResult con el nivel asignado.

Implementa heurísticas basadas en:
  1. Longitud estimada en tokens (aproximación: 1 token ≈ 4 caracteres en inglés,
     1 token ≈ 3 caracteres en español)
  2. Presencia de palabras clave asociadas a tareas complejas o simples

RISK: Las heurísticas basadas en longitud y palabras clave son una aproximación.
Para producción, considerar un clasificador ML ligero (e.g., un modelo de
embeddings + clasificador logístico) o usar el propio LLM para auto-clasificarse
en un pre-paso de bajo costo. Ver TODO al final del archivo.
"""

import re
import logging
from models.schemas import ClassificationResult, ComplexityLevel, ModelProvider
from config.settings import get_settings

logger = logging.getLogger(__name__)


# --- Catálogo de palabras clave por nivel de complejidad ---

COMPLEX_KEYWORDS: list[str] = [
    # Razonamiento y análisis profundo
    "analiza", "analyze", "razona", "reason", "deduce",
    "compara", "compare", "contrasta", "contrast",
    "argumenta", "argue", "evalúa", "evaluate",
    # Generación de contenido extenso
    "escribe un ensayo", "write an essay", "redacta un informe",
    "genera un plan", "diseña una arquitectura", "create a plan",
    # Código complejo
    "implementa", "implement", "refactoriza", "refactor",
    "optimiza el algoritmo", "optimize the algorithm",
    # Multi-paso
    "paso a paso", "step by step", "primero... luego... finalmente",
]

SIMPLE_KEYWORDS: list[str] = [
    # Preguntas directas de QA
    "qué es", "what is", "quién es", "who is",
    "cuándo fue", "when was", "dónde está", "where is",
    "define", "significa", "means",
    # Saludos y conversación trivial
    "hola", "hello", "hi", "gracias", "thanks", "bye", "adiós",
    # Preguntas de sí/no
    "es verdad", "is it true", "puedes confirmar", "can you confirm",
]


class ComplexityClassifier:
    """
    Clasificador de complejidad de prompts.

    Implementa el patrón Strategy de forma implícita: cada nivel de
    complejidad mapea a una estrategia de modelo diferente.
    La selección del modelo ocurre en ModelDispatcher, pero la
    clasificación aquí es la entrada a ese proceso de decisión.
    """

    def __init__(self) -> None:
        self._settings = get_settings()
        logger.info(
            "ComplexityClassifier inicializado | "
            f"Umbral SIMPLE: {self._settings.simple_max_tokens} tokens | "
            f"Umbral MEDIUM: {self._settings.medium_max_tokens} tokens"
        )

    def estimate_tokens(self, text: str) -> int:
        """
        Estima el número de tokens en el texto.

        Usa la heurística: 1 token ≈ 4 caracteres (válida para inglés y
        aproximada para español). Para mayor precisión en producción,
        usar tiktoken (OpenAI) o el tokenizador nativo del proveedor.

        TODO: Integrar tiktoken para conteo exacto de tokens de OpenAI:
            import tiktoken
            enc = tiktoken.encoding_for_model("gpt-4o")
            return len(enc.encode(text))
        """
        # Limpieza básica: eliminar espacios múltiples
        cleaned = re.sub(r'\s+', ' ', text.strip())
        estimated = max(1, len(cleaned) // 4)
        logger.debug(f"Tokens estimados para texto de {len(cleaned)} chars: {estimated}")
        return estimated

    def detect_keywords(self, text: str) -> tuple[list[str], list[str]]:
        """
        Detecta palabras clave de complejidad en el texto.

        Returns:
            Tupla (complex_found, simple_found) con las keywords detectadas.
        """
        text_lower = text.lower()
        complex_found = [kw for kw in COMPLEX_KEYWORDS if kw in text_lower]
        simple_found = [kw for kw in SIMPLE_KEYWORDS if kw in text_lower]
        return complex_found, simple_found

    def classify(self, user_message: str) -> ClassificationResult:
        """
        Clasifica la complejidad del prompt y retorna el resultado con
        el modelo recomendado.

        Lógica de decisión (en orden de prioridad):
          1. Si se detectan keywords de COMPLEJIDAD → COMPLEX (override)
          2. Si se detectan keywords de SIMPLICIDAD y tokens < umbral → SIMPLE
          3. Si tokens > umbral MEDIUM → COMPLEX
          4. Si tokens > umbral SIMPLE → MEDIUM
          5. Default → SIMPLE

        Args:
            user_message: El texto del último mensaje del usuario.

        Returns:
            ClassificationResult con el nivel, modelo y razonamiento.
        """
        estimated_tokens = self.estimate_tokens(user_message)
        complex_keywords, simple_keywords = self.detect_keywords(user_message)

        # --- Lógica de clasificación por prioridad ---
        level: ComplexityLevel
        reasoning: str

        if complex_keywords:
            # Las keywords de complejidad tienen máxima prioridad
            level = ComplexityLevel.COMPLEX
            reasoning = (
                f"Se detectaron {len(complex_keywords)} palabra(s) clave de alta complejidad: "
                f"{', '.join(complex_keywords[:3])}. "
                "Estas indican tareas de razonamiento, análisis o generación extensa."
            )
        elif simple_keywords and estimated_tokens <= self._settings.simple_max_tokens:
            # Keywords simples + texto corto → definitivamente SIMPLE
            level = ComplexityLevel.SIMPLE
            reasoning = (
                f"Prompt corto ({estimated_tokens} tokens estimados) con "
                f"palabras clave de consulta directa: {', '.join(simple_keywords[:3])}. "
                "Adecuado para un modelo económico."
            )
        elif estimated_tokens > self._settings.medium_max_tokens:
            # Texto muy largo sin keywords simples → COMPLEX por volumen
            level = ComplexityLevel.COMPLEX
            reasoning = (
                f"Prompt extenso ({estimated_tokens} tokens estimados, "
                f"umbral MEDIUM: {self._settings.medium_max_tokens}). "
                "La longitud sugiere una tarea compleja o con mucho contexto."
            )
        elif estimated_tokens > self._settings.simple_max_tokens:
            # Rango medio
            level = ComplexityLevel.MEDIUM
            reasoning = (
                f"Prompt de longitud media ({estimated_tokens} tokens estimados, "
                f"rango: {self._settings.simple_max_tokens}–{self._settings.medium_max_tokens}). "
                "Asignado a modelo de capacidad intermedia."
            )
        else:
            # Default: texto corto sin keywords especiales
            level = ComplexityLevel.SIMPLE
            reasoning = (
                f"Prompt corto ({estimated_tokens} tokens estimados, "
                f"umbral SIMPLE: {self._settings.simple_max_tokens}). "
                "Asignado al modelo más económico."
            )

        # Seleccionar modelo y proveedor según el nivel
        selected_model, provider = self._resolve_model(level)

        result = ClassificationResult(
            level=level,
            estimated_tokens=estimated_tokens,
            reasoning=reasoning,
            detected_keywords=complex_keywords + simple_keywords,
            selected_model=selected_model,
            provider=provider
        )

        logger.info(
            f"Clasificación completada | Nivel: {level.value} | "
            f"Tokens: {estimated_tokens} | Modelo: {selected_model} | "
            f"Keywords detectadas: {len(complex_keywords + simple_keywords)}"
        )

        return result

    def _resolve_model(self, level: ComplexityLevel) -> tuple[str, ModelProvider]:
        """
        Mapea un nivel de complejidad al modelo y proveedor correspondiente.

        RISK: Si el modelo configurado en .env no existe o el proveedor
        cambia su nomenclatura, este método retornará un nombre inválido
        que fallará en tiempo de ejecución, no en configuración.
        TODO: Añadir validación de nombres de modelos contra una lista
        de modelos soportados por cada proveedor al iniciar el servicio.
        """
        settings = self._settings

        if settings.is_mock_mode:
            # En modo mock, todos los niveles usan el proveedor MOCK
            model_map = {
                ComplexityLevel.SIMPLE: (settings.default_simple_model, ModelProvider.MOCK),
                ComplexityLevel.MEDIUM: (settings.default_medium_model, ModelProvider.MOCK),
                ComplexityLevel.COMPLEX: (settings.default_complex_model, ModelProvider.MOCK),
            }
        else:
            # En modo live, inferir el proveedor por el nombre del modelo
            model_map = {
                ComplexityLevel.SIMPLE: (
                    settings.default_simple_model,
                    self._infer_provider(settings.default_simple_model)
                ),
                ComplexityLevel.MEDIUM: (
                    settings.default_medium_model,
                    self._infer_provider(settings.default_medium_model)
                ),
                ComplexityLevel.COMPLEX: (
                    settings.default_complex_model,
                    self._infer_provider(settings.default_complex_model)
                ),
            }

        return model_map[level]

    @staticmethod
    def _infer_provider(model_name: str) -> ModelProvider:
        """Infiere el proveedor a partir del nombre del modelo."""
        name_lower = model_name.lower()
        if "gpt" in name_lower or "o1" in name_lower:
            return ModelProvider.OPENAI
        elif "claude" in name_lower:
            return ModelProvider.ANTHROPIC
        elif "gemini" in name_lower:
            return ModelProvider.GOOGLE
        else:
            logger.warning(f"Proveedor no reconocido para el modelo '{model_name}'. Usando MOCK.")
            return ModelProvider.MOCK
```

#### Verificación

```bash
python -c "
from services.complexity_classifier import ComplexityClassifier
from models.schemas import ComplexityLevel

clf = ComplexityClassifier()

# Test 1: Prompt simple
r1 = clf.classify('Hola, ¿qué es Python?')
assert r1.level == ComplexityLevel.SIMPLE, f'Esperado SIMPLE, obtenido {r1.level}'
print(f'[OK] SIMPLE: {r1.reasoning[:60]}...')

# Test 2: Prompt complejo por keyword
r2 = clf.classify('Analiza las ventajas y desventajas de los transformers')
assert r2.level == ComplexityLevel.COMPLEX, f'Esperado COMPLEX, obtenido {r2.level}'
print(f'[OK] COMPLEX: {r2.reasoning[:60]}...')

# Test 3: Prompt medio por longitud
medium_text = 'Necesito un resumen de los conceptos básicos de aprendizaje automático incluyendo supervisado no supervisado y por refuerzo para un estudiante universitario'
r3 = clf.classify(medium_text)
print(f'[INFO] Prompt medio: nivel={r3.level.value}, tokens={r3.estimated_tokens}')
"
```

---

### Paso 5 — Implementar el despachador de modelos (`services/model_dispatcher.py`)

**Objetivo:** Construir el componente que aplica el **patrón Strategy** para ejecutar la llamada al LLM (o mock) apropiado según el resultado de la clasificación.

#### Instrucciones

1. Abre `services/model_dispatcher.py` y escribe:

```python
# services/model_dispatcher.py
"""
Despachador de modelos — implementa el Patrón Strategy.

Responsabilidad: recibir un ClassificationResult y los mensajes del usuario,
y retornar la respuesta del modelo (real o mock) junto con metadatos de uso.

Patrón Strategy aplicado:
  - Interfaz común: ModelStrategy (clase base abstracta)
  - Estrategias concretas: MockStrategy, OpenAIStrategy, AnthropicStrategy, GoogleStrategy
  - Contexto: ModelDispatcher selecciona y ejecuta la estrategia correcta

Esto permite añadir nuevos proveedores sin modificar el código del dispatcher:
solo se agrega una nueva clase Strategy y se registra en el mapa de estrategias.

RISK: Las estrategias reales (OpenAI, Anthropic, Google) requieren que las
API keys estén configuradas en .env. Si las keys son inválidas, el error
ocurrirá en tiempo de ejecución. Implementar circuit breaker en producción.
"""

import time
import uuid
import logging
from abc import ABC, abstractmethod
from unittest.mock import MagicMock
from typing import Any

from models.schemas import (
    ClassificationResult,
    ChatMessage,
    ModelProvider,
    ComplexityLevel
)
from config.settings import get_settings

logger = logging.getLogger(__name__)


# ============================================================
# INTERFAZ BASE (Patrón Strategy)
# ============================================================

class ModelStrategy(ABC):
    """
    Interfaz abstracta que todas las estrategias de modelo deben implementar.

    Cada estrategia encapsula la lógica de llamada a un proveedor específico,
    incluyendo el formato de mensajes, manejo de errores y extracción de tokens.
    """

    @abstractmethod
    def execute(
        self,
        messages: list[ChatMessage],
        model_name: str,
        max_tokens: int,
        temperature: float
    ) -> dict[str, Any]:
        """
        Ejecuta la llamada al modelo y retorna un diccionario estandarizado.

        Returns:
            {
                "content": str,          # Texto generado
                "tokens_used": int | None,
                "latency_ms": float
            }
        """
        ...


# ============================================================
# ESTRATEGIA MOCK (para entornos sin API keys)
# ============================================================

class MockStrategy(ModelStrategy):
    """
    Estrategia mock que simula respuestas de LLM sin consumir APIs de pago.

    Útil para:
    - Tests unitarios y de integración
    - Demos y presentaciones offline
    - Desarrollo cuando no se tienen API keys disponibles

    TODO: Mejorar el mock para retornar respuestas más realistas según
    el nivel de complejidad, usando plantillas por tipo de tarea.
    """

    MOCK_RESPONSES: dict[str, str] = {
        ComplexityLevel.SIMPLE: (
            "[MOCK - Modelo simple] Esta es una respuesta simulada para un prompt simple. "
            "En producción, esta respuesta sería generada por {model_name}."
        ),
        ComplexityLevel.MEDIUM: (
            "[MOCK - Modelo medio] Esta es una respuesta simulada para un prompt de "
            "complejidad media. El modelo {model_name} procesaría este tipo de solicitudes "
            "con mayor capacidad de razonamiento y síntesis."
        ),
        ComplexityLevel.COMPLEX: (
            "[MOCK - Modelo complejo] Esta es una respuesta simulada para un prompt complejo. "
            "El modelo {model_name} aplicaría razonamiento profundo, análisis multi-paso "
            "y generaría una respuesta extensa y estructurada."
        ),
    }

    def __init__(self, complexity_level: ComplexityLevel) -> None:
        self._complexity_level = complexity_level

    def execute(
        self,
        messages: list[ChatMessage],
        model_name: str,
        max_tokens: int,
        temperature: float
    ) -> dict[str, Any]:
        # Simular latencia realista según complejidad
        simulated_latency = {
            ComplexityLevel.SIMPLE: 120.0,
            ComplexityLevel.MEDIUM: 450.0,
            ComplexityLevel.COMPLEX: 1200.0,
        }.get(self._complexity_level, 200.0)

        template = self.MOCK_RESPONSES.get(
            self._complexity_level,
            "[MOCK] Respuesta genérica simulada para {model_name}."
        )
        content = template.format(model_name=model_name)

        logger.info(
            f"[MOCK] Estrategia ejecutada | Modelo: {model_name} | "
            f"Latencia simulada: {simulated_latency}ms"
        )

        return {
            "content": content,
            "tokens_used": None,  # Mock no consume tokens reales
            "latency_ms": simulated_latency,
        }


# ============================================================
# ESTRATEGIA OPENAI (stub para integración futura)
# ============================================================

class OpenAIStrategy(ModelStrategy):
    """
    Estrategia para modelos de OpenAI (GPT-4o, GPT-4o-mini, etc.).

    TODO: Implementar la integración real con el SDK de OpenAI:
        from openai import OpenAI
        client = OpenAI(api_key=settings.openai_api_key)
        response = client.chat.completions.create(...)

    RISK: La API de OpenAI puede retornar errores 429 (rate limit) o 503
    (servicio no disponible). Implementar reintentos con backoff exponencial
    usando la librería 'tenacity' antes de pasar a producción.
    """

    def execute(
        self,
        messages: list[ChatMessage],
        model_name: str,
        max_tokens: int,
        temperature: float
    ) -> dict[str, Any]:
        settings = get_settings()
        if not settings.openai_api_key:
            raise ValueError(
                "OPENAI_API_KEY no configurada. "
                "Añádela al archivo .env o usa el modo mock."
            )

        # TODO: Reemplazar este stub con la llamada real al SDK de OpenAI
        # import openai
        # client = openai.OpenAI(api_key=settings.openai_api_key)
        # start = time.time()
        # response = client.chat.completions.create(
        #     model=model_name,
        #     messages=[{"role": m.role, "content": m.content} for m in messages],
        #     max_tokens=max_tokens,
        #     temperature=temperature
        # )
        # latency_ms = (time.time() - start) * 1000
        # return {
        #     "content": response.choices[0].message.content,
        #     "tokens_used": response.usage.total_tokens,
        #     "latency_ms": latency_ms,
        # }

        raise NotImplementedError(
            f"La estrategia OpenAI para '{model_name}' aún no está implementada. "
            "Configura OPENAI_API_KEY y descomenta el código del stub."
        )


# ============================================================
# ESTRATEGIA ANTHROPIC (stub para integración futura)
# ============================================================

class AnthropicStrategy(ModelStrategy):
    """
    Estrategia para modelos de Anthropic (Claude 3.5 Sonnet, etc.).

    TODO: Implementar con el SDK de Anthropic:
        from anthropic import Anthropic
        client = Anthropic(api_key=settings.anthropic_api_key)

    RISK: Claude tiene un formato de mensajes diferente a OpenAI.
    El mensaje 'system' debe pasarse como parámetro separado, no dentro
    de la lista de mensajes. Adaptar el formato antes de la llamada.
    """

    def execute(
        self,
        messages: list[ChatMessage],
        model_name: str,
        max_tokens: int,
        temperature: float
    ) -> dict[str, Any]:
        settings = get_settings()
        if not settings.anthropic_api_key:
            raise ValueError("ANTHROPIC_API_KEY no configurada.")

        # TODO: Implementar llamada real al SDK de Anthropic
        raise NotImplementedError(
            f"La estrategia Anthropic para '{model_name}' aún no está implementada."
        )


# ============================================================
# CONTEXTO DEL PATRÓN STRATEGY: ModelDispatcher
# ============================================================

class ModelDispatcher:
    """
    Contexto del Patrón Strategy que selecciona y ejecuta la estrategia
    de modelo correcta según el ClassificationResult.

    Flujo:
      1. Recibe ClassificationResult del ComplexityClassifier
      2. Selecciona la estrategia (mock, openai, anthropic, google)
      3. Ejecuta la estrategia con los mensajes del usuario
      4. Retorna el resultado estandarizado

    Este componente corresponde a la 'Capa de Orquestación' de la Lección 2.1:
    coordina el flujo y decide qué herramienta invocar.
    """

    def __init__(self) -> None:
        self._settings = get_settings()
        self._request_count = 0  # Contador para observabilidad
        logger.info(
            f"ModelDispatcher inicializado | "
            f"Modo: {'mock' if self._settings.is_mock_mode else 'live'}"
        )

    def dispatch(
        self,
        classification: ClassificationResult,
        messages: list[ChatMessage],
        max_tokens: int = 512,
        temperature: float = 0.7
    ) -> dict[str, Any]:
        """
        Despacha la solicitud al modelo apropiado y retorna la respuesta.

        Args:
            classification: Resultado del ComplexityClassifier
            messages: Historial de mensajes del usuario
            max_tokens: Límite de tokens para la respuesta
            temperature: Temperatura de muestreo

        Returns:
            Diccionario con 'content', 'tokens_used', 'latency_ms' e 'is_mock'
        """
        self._request_count += 1
        strategy = self._select_strategy(classification)

        logger.info(
            f"Despachando solicitud #{self._request_count} | "
            f"Modelo: {classification.selected_model} | "
            f"Proveedor: {classification.provider.value} | "
            f"Nivel: {classification.level.value}"
        )

        start_time = time.time()
        try:
            result = strategy.execute(
                messages=messages,
                model_name=classification.selected_model,
                max_tokens=max_tokens,
                temperature=temperature
            )
            # Si la estrategia no midió la latencia, la medimos aquí
            if result.get("latency_ms") is None:
                result["latency_ms"] = (time.time() - start_time) * 1000

            result["is_mock"] = isinstance(strategy, MockStrategy)
            return result

        except NotImplementedError as e:
            logger.error(f"Estrategia no implementada: {e}")
            raise
        except Exception as e:
            logger.error(
                f"Error en dispatch | Modelo: {classification.selected_model} | "
                f"Error: {type(e).__name__}: {e}"
            )
            raise

    def _select_strategy(self, classification: ClassificationResult) -> ModelStrategy:
        """
        Selecciona la estrategia concreta basándose en el proveedor
        y si el servicio está en modo mock.

        RISK: Si se añaden nuevos proveedores sin actualizar este método,
        el sistema caerá silenciosamente al modo mock en lugar de fallar
        explícitamente. Considerar usar un registro (registry) de estrategias.
        """
        if self._settings.is_mock_mode or classification.provider == ModelProvider.MOCK:
            return MockStrategy(complexity_level=classification.level)

        strategy_map: dict[ModelProvider, ModelStrategy] = {
            ModelProvider.OPENAI: OpenAIStrategy(),
            ModelProvider.ANTHROPIC: AnthropicStrategy(),
            # TODO: Añadir GoogleStrategy cuando esté implementada
        }

        strategy = strategy_map.get(classification.provider)
        if strategy is None:
            logger.warning(
                f"Proveedor '{classification.provider.value}' sin estrategia implementada. "
                "Usando MockStrategy como fallback."
            )
            return MockStrategy(complexity_level=classification.level)

        return strategy

    @property
    def request_count(self) -> int:
        """Número total de solicitudes despachadas (para observabilidad)."""
        return self._request_count
```

---

### Paso 6 — Implementar los endpoints del router (`routers/llm_router.py`)

**Objetivo:** Conectar los servicios de clasificación y despacho con los endpoints HTTP de FastAPI.

#### Instrucciones

1. Abre `routers/llm_router.py` y escribe:

```python
# routers/llm_router.py
"""
Endpoints del Model Router Service.

Define los tres endpoints públicos del servicio:
  - POST /v1/chat/complete  → Enruta y procesa la solicitud
  - GET  /v1/router/status  → Estado actual del router
  - GET  /v1/health         → Health check para load balancers y orquestadores

La separación entre 'routers/' y 'services/' refleja la arquitectura
de la Lección 2.1: los routers son la capa de presentación/entrada,
los services son la capa de orquestación y lógica de negocio.
"""

import uuid
import logging
import time
from datetime import datetime, timezone

from fastapi import APIRouter, HTTPException, Request, status

from models.schemas import (
    ChatCompleteRequest,
    ChatCompleteResponse,
    RouterDecision,
    RouterStatusResponse,
    HealthResponse,
)
from services.complexity_classifier import ComplexityClassifier
from services.model_dispatcher import ModelDispatcher
from config.settings import get_settings

logger = logging.getLogger(__name__)

# Instancias de los servicios (singleton por módulo)
# TODO: En producción, usar inyección de dependencias de FastAPI (Depends)
# para facilitar el testing y el reemplazo de implementaciones.
_classifier = ComplexityClassifier()
_dispatcher = ModelDispatcher()
_settings = get_settings()
_service_start_time = time.time()

router = APIRouter(prefix="/v1", tags=["Model Router"])


@router.post(
    "/chat/complete",
    response_model=ChatCompleteResponse,
    status_code=status.HTTP_200_OK,
    summary="Enruta una solicitud de chat al modelo LLM apropiado",
    description=(
        "Clasifica la complejidad del último mensaje del usuario y despacha "
        "la solicitud al modelo más adecuado según la configuración del router. "
        "Retorna la respuesta del modelo junto con los metadatos de la decisión de routing."
    )
)
async def chat_complete(request_body: ChatCompleteRequest) -> ChatCompleteResponse:
    """
    Endpoint principal del router.

    Flujo interno:
      1. Extraer el último mensaje del usuario para clasificación
      2. Clasificar la complejidad del prompt
      3. Despachar al modelo seleccionado
      4. Construir y retornar la respuesta estandarizada

    RISK: Este endpoint no implementa autenticación en el boilerplate.
    En producción, añadir middleware de API Key o JWT antes del despliegue.
    TODO: Añadir rate limiting por IP o por usuario autenticado.
    """
    request_id = str(uuid.uuid4())
    logger.info(f"Nueva solicitud recibida | request_id: {request_id}")

    # Extraer el último mensaje del usuario para clasificar
    user_messages = [m for m in request_body.messages if m.role == "user"]
    if not user_messages:
        raise HTTPException(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            detail="No se encontró ningún mensaje con role='user' en el historial."
        )

    last_user_message = user_messages[-1].content

    try:
        # Paso 1: Clasificar la complejidad
        classification = _classifier.classify(last_user_message)
        logger.info(
            f"[{request_id}] Clasificación: {classification.level.value} | "
            f"Modelo seleccionado: {classification.selected_model}"
        )

        # Paso 2: Despachar al modelo
        dispatch_result = _dispatcher.dispatch(
            classification=classification,
            messages=request_body.messages,
            max_tokens=request_body.max_tokens or 512,
            temperature=request_body.temperature or 0.7
        )

        # Paso 3: Construir la respuesta
        router_decision = RouterDecision(
            complexity_level=classification.level,
            model_used=classification.selected_model,
            provider=classification.provider,
            estimated_tokens=classification.estimated_tokens,
            routing_reasoning=classification.reasoning
        )

        response = ChatCompleteResponse(
            request_id=request_id,
            content=dispatch_result["content"],
            router_decision=router_decision,
            tokens_used=dispatch_result.get("tokens_used"),
            latency_ms=dispatch_result.get("latency_ms"),
            is_mock=dispatch_result.get("is_mock", True)
        )

        logger.info(
            f"[{request_id}] Solicitud completada | "
            f"Latencia: {response.latency_ms:.1f}ms | "
            f"Mock: {response.is_mock}"
        )

        return response

    except Exception as e:
        logger.error(f"[{request_id}] Error procesando solicitud: {type(e).__name__}: {e}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Error interno del router: {str(e)}"
        )


@router.get(
    "/router/status",
    response_model=RouterStatusResponse,
    status_code=status.HTTP_200_OK,
    summary="Estado actual del router y configuración de modelos"
)
async def router_status() -> RouterStatusResponse:
    """
    Retorna el estado operacional del router incluyendo:
    - Modelos configurados por nivel de complejidad
    - Umbrales de clasificación activos
    - Modo de operación (live vs mock)
    - Contador de solicitudes procesadas
    """
    return RouterStatusResponse(
        service_name=_settings.app_name,
        version=_settings.app_version,
        environment=_settings.environment,
        mode="mock" if _settings.is_mock_mode else "live",
        configured_models={
            "SIMPLE": _settings.default_simple_model,
            "MEDIUM": _settings.default_medium_model,
            "COMPLEX": _settings.default_complex_model,
        },
        thresholds={
            "simple_max_tokens": _settings.simple_max_tokens,
            "medium_max_tokens": _settings.medium_max_tokens,
        },
        total_requests_served=_dispatcher.request_count
    )


@router.get(
    "/health",
    response_model=HealthResponse,
    status_code=status.HTTP_200_OK,
    summary="Health check del servicio"
)
async def health_check() -> HealthResponse:
    """
    Endpoint de health check para load balancers, orquestadores (Kubernetes)
    y herramientas de monitoreo.

    Verifica:
    - Disponibilidad del clasificador
    - Disponibilidad del dispatcher
    - Modo de configuración (mock vs live con keys)

    TODO: En producción, añadir verificación de conectividad con las APIs
    externas (OpenAI, Anthropic) usando llamadas de prueba de bajo costo.
    """
    uptime = time.time() - _service_start_time

    checks = {
        "classifier": True,   # Si llegamos aquí, el clasificador está operativo
        "dispatcher": True,   # Si llegamos aquí, el dispatcher está operativo
        "api_keys_configured": not _settings.is_mock_mode,
    }

    # El servicio está 'degraded' si opera en modo mock (sin API keys reales)
    overall_status = "healthy" if not _settings.is_mock_mode else "degraded"

    return HealthResponse(
        status=overall_status,
        version=_settings.app_version,
        environment=_settings.environment,
        uptime_seconds=uptime,
        checks=checks
    )
```

---

### Paso 7 — Implementar el punto de entrada y el middleware de logging (`main.py`)

**Objetivo:** Ensamblar todos los componentes en la aplicación FastAPI y configurar el middleware de logging que registra las decisiones de routing.

#### Instrucciones

1. Abre `main.py` y escribe:

```python
# main.py
"""
Punto de entrada del Model Router Service.

Responsabilidades de este módulo:
  1. Crear e instanciar la aplicación FastAPI
  2. Configurar el middleware de logging estructurado
  3. Registrar los routers de endpoints
  4. Exponer el endpoint raíz de información del servicio

Arquitectura reflejada (Lección 2.1):
  - Este archivo es la 'Capa de Presentación': el punto de entrada al sistema
  - Los routers/llm_router.py orquestan el flujo
  - Los services/ implementan la lógica de negocio
  - Los models/schemas.py definen los contratos de datos
"""

import logging
import time
import uuid
from contextlib import asynccontextmanager
from typing import AsyncGenerator

from fastapi import FastAPI, Request, Response
from fastapi.middleware.cors import CORSMiddleware

from config.settings import get_settings
from routers.llm_router import router as llm_router

# ============================================================
# CONFIGURACIÓN DE LOGGING
# ============================================================

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
    datefmt="%Y-%m-%dT%H:%M:%S"
)
logger = logging.getLogger("model_router")


# ============================================================
# LIFECYCLE DEL SERVICIO
# ============================================================

@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator:
    """
    Gestiona el ciclo de vida de la aplicación.
    El código antes del 'yield' se ejecuta al iniciar.
    El código después del 'yield' se ejecuta al apagar.
    """
    settings = get_settings()
    logger.info("=" * 60)
    logger.info(f"Iniciando {settings.app_name} v{settings.app_version}")
    logger.info(f"Entorno: {settings.environment}")
    logger.info(f"Modo: {'MOCK (sin API keys)' if settings.is_mock_mode else 'LIVE'}")
    logger.info(f"Modelo SIMPLE : {settings.default_simple_model}")
    logger.info(f"Modelo MEDIUM : {settings.default_medium_model}")
    logger.info(f"Modelo COMPLEX: {settings.default_complex_model}")
    logger.info("=" * 60)

    yield  # El servidor está activo

    logger.info(f"Apagando {settings.app_name}...")


# ============================================================
# INSTANCIA DE LA APLICACIÓN
# ============================================================

settings = get_settings()

app = FastAPI(
    title=settings.app_name,
    version=settings.app_version,
    description=(
        "Router inteligente de modelos LLM que clasifica la complejidad "
        "del prompt entrante y despacha la solicitud al modelo más adecuado "
        "según criterios de costo y capacidad. "
        "Implementa el Patrón Strategy para la selección dinámica de modelos."
    ),
    lifespan=lifespan,
    docs_url="/docs",
    redoc_url="/redoc",
)


# ============================================================
# MIDDLEWARE
# ============================================================

# CORS: configuración permisiva para desarrollo
# TODO: Restringir 'allow_origins' a dominios específicos en producción
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.middleware("http")
async def logging_middleware(request: Request, call_next) -> Response:
    """
    Middleware de logging que registra cada solicitud HTTP con:
    - ID único de solicitud (para trazabilidad)
    - Método y ruta
    - Código de estado de la respuesta
    - Latencia total del endpoint

    Este middleware implementa observabilidad básica, correspondiente
    a la 'Capa de Almacenamiento y Observabilidad' de la Lección 2.1.

    TODO: En producción, reemplazar por logging estructurado en JSON
    compatible con sistemas de log aggregation (ELK, Cloud Logging, Datadog).
    """
    request_id = str(uuid.uuid4())[:8]  # ID corto para los logs
    start_time = time.time()

    # Adjuntar el request_id al estado de la request para uso downstream
    request.state.request_id = request_id

    logger.info(f"[{request_id}] → {request.method} {request.url.path}")

    response = await call_next(request)

    latency_ms = (time.time() - start_time) * 1000
    logger.info(
        f"[{request_id}] ← {response.status_code} | "
        f"Latencia: {latency_ms:.1f}ms | "
        f"Ruta: {request.url.path}"
    )

    # Añadir headers de trazabilidad a la respuesta
    response.headers["X-Request-ID"] = request_id
    response.headers["X-Response-Time-Ms"] = f"{latency_ms:.1f}"

    return response


# ============================================================
# REGISTRO DE ROUTERS
# ============================================================

app.include_router(llm_router)


# ============================================================
# ENDPOINT RAÍZ
# ============================================================

@app.get("/", tags=["Info"])
async def root() -> dict:
    """Información básica del servicio. Redirige a /docs para la documentación completa."""
    return {
        "service": settings.app_name,
        "version": settings.app_version,
        "environment": settings.environment,
        "docs": "/docs",
        "health": "/v1/health",
        "status": "/v1/router/status",
    }


# ============================================================
# PUNTO DE ENTRADA PARA EJECUCIÓN DIRECTA
# ============================================================

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=True,  # Hot-reload para desarrollo
        log_level="info"
    )
```

---

## 7. Validación y Pruebas

### Iniciar el servidor

```bash
# Desde el directorio lab-02-model-router con el venv activo
uvicorn main:app --reload --port 8000
```

**Salida esperada en la consola:**
```
2024-01-15T10:30:00 | INFO     | model_router | ============================================================
2024-01-15T10:30:00 | INFO     | model_router | Iniciando Model Router Service v0.1.0
2024-01-15T10:30:00 | INFO     | model_router | Entorno: development
2024-01-15T10:30:00 | INFO     | model_router | Modo: MOCK (sin API keys)
2024-01-15T10:30:00 | INFO     | model_router | Modelo SIMPLE : gpt-4o-mini
2024-01-15T10:30:00 | INFO     | model_router | Modelo MEDIUM : gpt-4o
2024-01-15T10:30:00 | INFO     | model_router | Modelo COMPLEX: claude-3-5-sonnet-20241022
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000
```

### Prueba 1: Health check

```bash
curl -s http://localhost:8000/v1/health | python3 -m json.tool
```

**Salida esperada:**
```json
{
    "status": "degraded",
    "version": "0.1.0",
    "environment": "development",
    "uptime_seconds": 5.2,
    "checks": {
        "classifier": true,
        "dispatcher": true,
        "api_keys_configured": false
    }
}
```

> 💡 `"status": "degraded"` es correcto: indica que el servicio opera en modo mock sin API keys reales. En producción con keys configuradas, retornaría `"healthy"`.

### Prueba 2: Estado del router

```bash
curl -s http://localhost:8000/v1/router/status | python3 -m json.tool
```

**Salida esperada:**
```json
{
    "service_name": "Model Router Service",
    "version": "0.1.0",
    "environment": "development",
    "mode": "mock",
    "configured_models": {
        "SIMPLE": "gpt-4o-mini",
        "MEDIUM": "gpt-4o",
        "COMPLEX": "claude-3-5-sonnet-20241022"
    },
    "thresholds": {
        "simple_max_tokens": 50,
        "medium_max_tokens": 200
    },
    "total_requests_served": 0
}
```

### Prueba 3: Prompt SIMPLE

```bash
curl -s -X POST http://localhost:8000/v1/chat/complete \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "user", "content": "Hola, ¿qué es Python?"}
    ]
  }' | python3 -m json.tool
```

**Verificar en la respuesta:**
- `router_decision.complexity_level` = `"SIMPLE"`
- `router_decision.model_used` = `"gpt-4o-mini"`
- `is_mock` = `true`

### Prueba 4: Prompt COMPLEX (por keyword)

```bash
curl -s -X POST http://localhost:8000/v1/chat/complete \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "user", "content": "Analiza las ventajas y desventajas de usar microservicios versus arquitectura monolítica para una startup con equipo pequeño."}
    ],
    "temperature": 0.5
  }' | python3 -m json.tool
```

**Verificar en la respuesta:**
- `router_decision.complexity_level` = `"COMPLEX"`
- `router_decision.model_used` = `"claude-3-5-sonnet-20241022"`
- `router_decision.routing_reasoning` contiene mención de la keyword `"analiza"`

### Prueba 5: Verificar el contador de solicitudes

```bash
# Después de las pruebas anteriores
curl -s http://localhost:8000/v1/router/status | python3 -c "
import sys, json
data = json.load(sys.stdin)
count = data['total_requests_served']
print(f'Total solicitudes procesadas: {count}')
assert count >= 2, f'Esperado >= 2, obtenido {count}'
print('[OK] Contador de observabilidad funcionando correctamente')
"
```

### Prueba 6: Validación de error (mensaje inválido)

```bash
# Intentar enviar un request sin mensajes de usuario
curl -s -X POST http://localhost:8000/v1/chat/complete \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "assistant", "content": "Hola"}
    ]
  }' | python3 -m json.tool
```

**Salida esperada:** Error 422 de Pydantic por el validador `validate_last_message_is_user`.

### Acceder a la documentación interactiva

Abre en tu navegador: **http://localhost:8000/docs**

Deberías ver la documentación Swagger UI con los tres endpoints documentados automáticamente.

---

## 8. Resolución de Problemas

### Problema 1: `ModuleNotFoundError: No module named 'pydantic_settings'`

**Síntomas:**
```
ModuleNotFoundError: No module named 'pydantic_settings'
```
El servidor no inicia y el error aparece al importar `config/settings.py`.

**Causa:**
En Pydantic v2, `BaseSettings` se movió a un paquete separado llamado `pydantic-settings`. Si solo se instaló `pydantic`, el módulo `pydantic_settings` no existe.

**Solución:**
```bash
# Verificar si pydantic-settings está instalado
pip show pydantic-settings

# Si no está instalado:
pip install "pydantic-settings==2.3.4"
echo "pydantic-settings==2.3.4" >> requirements.txt

# Verificar la instalación
python -c "from pydantic_settings import BaseSettings; print('OK')"
```

---

### Problema 2: El clasificador siempre retorna `COMPLEX` aunque el prompt sea corto

**Síntomas:**
Prompts cortos como `"¿Qué es Python?"` retornan `complexity_level: "COMPLEX"` en lugar de `"SIMPLE"`. El log muestra keywords detectadas inesperadas.

**Causa:**
Las listas `COMPLEX_KEYWORDS` y `SIMPLE_KEYWORDS` en `complexity_classifier.py` usan búsqueda de subcadena (`if kw in text_lower`). Palabras cortas como `"analiza"` pueden coincidir parcialmente con otras palabras del texto (por ejemplo, `"analizar"` contiene `"analiz"` pero no `"analiza"`). Sin embargo, si el texto contiene palabras compuestas que incluyen alguna keyword, se producirá un falso positivo.

**Diagnóstico:**
```bash
python -c "
from services.complexity_classifier import ComplexityClassifier
clf = ComplexityClassifier()
text = 'Tu pregunta'  # Reemplaza con el texto problemático
complex_kw, simple_kw = clf.detect_keywords(text)
print(f'Keywords complejas detectadas: {complex_kw}')
print(f'Keywords simples detectadas: {simple_kw}')
print(f'Tokens estimados: {clf.estimate_tokens(text)}')
"
```

**Solución:**
Usar búsqueda de palabras completas con expresiones regulares en lugar de búsqueda de subcadena:

```python
# En services/complexity_classifier.py, modificar detect_keywords:
import re

def detect_keywords(self, text: str) -> tuple[list[str], list[str]]:
    text_lower = text.lower()
    # Usar \b para coincidir solo con palabras completas
    complex_found = [
        kw for kw in COMPLEX_KEYWORDS
        if re.search(r'\b' + re.escape(kw) + r'\b', text_lower)
    ]
    simple_found = [
        kw for kw in SIMPLE_KEYWORDS
        if re.search(r'\b' + re.escape(kw) + r'\b', text_lower)
    ]
    return complex_found, simple_found
```

---

## 9. Limpieza del Entorno

Al finalizar la práctica, ejecuta los siguientes pasos para dejar el entorno limpio:

```bash
# 1. Detener el servidor uvicorn
# Presiona Ctrl+C en la terminal donde está corriendo

# 2. Desactivar el entorno virtual
deactivate

# 3. (Opcional) Eliminar el directorio del proyecto si no continuarás con él
# ⚠️ Solo ejecutar si ya no necesitas el código
# cd ..
# rm -rf lab-02-model-router

# 4. Si vas a continuar en otra sesión, simplemente reactiva el venv:
# cd lab-02-model-router
# source .venv/bin/activate  (macOS/Linux)
# .venv\Scripts\Activate.ps1  (Windows)
```

> ✅ **Buena práctica**: Antes de hacer commit a Git, verifica siempre que el archivo `.env` aparezca en `.gitignore` y que no esté en el staging area:
> ```bash
> git status  # .env NO debe aparecer como archivo a commitear
> ```

---

## 10. Resumen

### Lo que construiste

En esta práctica implementaste el **boilerplate completo de un Router de Modelos** con FastAPI, aplicando directamente los conceptos de la Lección 2.1:

| Componente de la Lección 2.1 | Archivo implementado | Responsabilidad |
|---|---|---|
| Capa de presentación | `routers/llm_router.py` | Endpoints HTTP, validación de entrada |
| Capa de orquestación | `services/model_dispatcher.py` | Selección y ejecución de estrategia |
| Motor de prompts (análisis) | `services/complexity_classifier.py` | Clasificación de complejidad |
| Modelo de lenguaje | `MockStrategy` + stubs | Generación de respuesta (simulada) |
| Almacenamiento/Observabilidad | Middleware de logging en `main.py` | Registro de decisiones de routing |
| Configuración | `config/settings.py` + `.env` | Configuración externalizada y tipada |

### Patrones de diseño aplicados

- **Strategy**: `ModelStrategy` → `MockStrategy`, `OpenAIStrategy`, `AnthropicStrategy` permiten añadir nuevos proveedores sin modificar `ModelDispatcher`.
- **Singleton con `@lru_cache`**: La configuración se lee una sola vez del archivo `.env`.
- **Fail-fast validation**: Pydantic v2 valida los contratos de API antes de que la solicitud llegue a la lógica de negocio.
- **Separación de responsabilidades**: Cada archivo tiene una única razón para cambiar.

### Puntos de extensión documentados (TODOs)

El código incluye **12 comentarios TODO/RISK** que mapean el camino hacia una implementación de producción:
- Integración real con SDKs de OpenAI, Anthropic y Google
- Reintentos con backoff exponencial usando `tenacity`
- Rate limiting por usuario
- Autenticación con JWT o API Key
- Clasificador ML en lugar de heurísticas
- Logging estructurado en JSON

### Recursos adicionales

- [FastAPI — Documentación oficial](https://fastapi.tiangolo.com/)
- [Pydantic v2 — Guía de migración desde v1](https://docs.pydantic.dev/latest/migration/)
- [Patrón Strategy — Refactoring Guru](https://refactoring.guru/es/design-patterns/strategy)
- [pydantic-settings — Gestión de configuración](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)
- [Lección 2.1 — Componentes de una Solución GenAI](../lessons/02-01-componentes-genai.md)

---
*Lab 02-00-01 — Model Router Boilerplate | Módulo 2: Componentes de una Solución GenAI*
