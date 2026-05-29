---LAB_START---
LAB_ID: 03-00-01
---MARKDOWN---
# Implementar un cliente Python asíncrono con Pydantic para forzar respuestas estructuradas y manejo de reintentos exponenciales.

## 1. Metadatos

| Campo         | Valor                                                                 |
|---------------|-----------------------------------------------------------------------|
| **Duración**  | 50 minutos                                                            |
| **Complejidad** | Alta                                                                |
| **Nivel Bloom** | Aplicar                                                             |
| **Módulo**    | 3 — Uso de SDKs y Clientes Robustos                                   |
| **Costo estimado** | < $0.10 USD (GPT-4o-mini, batch de 10 incidentes)              |

---

## 2. Descripción General

En este laboratorio construirás un módulo Python de nivel producción llamado `structured_llm_client.py` que combina tres pilares de ingeniería de software aplicada a IA Generativa: **programación asíncrona** con `asyncio`, **validación de esquemas** con Pydantic v2, y **resiliencia ante fallos** con reintentos exponenciales mediante `tenacity`. El caso de uso concreto es la extracción automática de información estructurada a partir de descripciones de incidentes técnicos en texto libre, un patrón ampliamente usado en operaciones de TI, SRE y plataformas de monitoreo. Al finalizar, habrás medido empíricamente la ganancia de rendimiento del procesamiento concurrente frente al secuencial.

---

## 3. Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Implementar un cliente asíncrono con `openai.AsyncOpenAI` que use **Structured Outputs** para forzar respuestas JSON validadas por Pydantic v2.
- [ ] Aplicar modelos `BaseModel` de Pydantic v2 con tipos restrictivos (`Literal`, `Optional`, `List`) para definir y validar esquemas de respuesta de un LLM.
- [ ] Configurar el decorador `@retry` de `tenacity` con backoff exponencial para manejar errores `RateLimitError` y `APIStatusError` de forma transparente.
- [ ] Procesar un batch de incidentes concurrentemente con `asyncio.gather()` y controlar la tasa de solicitudes con `asyncio.Semaphore`.
- [ ] Medir y comparar el tiempo de procesamiento secuencial vs. concurrente sobre el mismo conjunto de datos.

---

## 4. Prerrequisitos

### Conocimientos
- Python asíncrono: `async def`, `await`, `asyncio.run()`, `asyncio.gather()`
- Pydantic v2: `BaseModel`, `Field`, `ValidationError` (cubierto en Lab 02-00-01)
- SDK de OpenAI para Python (cubierto en Lección 3.1)
- Manejo de variables de entorno con `python-dotenv`

### Acceso y Credenciales
- `OPENAI_API_KEY` activa con acceso a **GPT-4o-mini** o superior
- Límite de gasto mensual configurado en [platform.openai.com/account/limits](https://platform.openai.com/account/limits) (recomendado: $5 USD máximo para todo el curso)
- Conexión a internet estable (≥ 10 Mbps)

> ⚠️ **Advertencia de costos**: Este lab procesa 10 incidentes con GPT-4o-mini. El costo estimado es inferior a $0.05 USD. Usa `gpt-4o-mini` (no `gpt-4o`) para minimizar costos durante el desarrollo.

---

## 5. Entorno del Laboratorio

### Hardware Recomendado

| Componente | Mínimo | Recomendado |
|------------|--------|-------------|
| RAM | 8 GB | 16 GB |
| CPU | 4 núcleos | 4+ núcleos |
| Almacenamiento libre | 500 MB | 1 GB |
| Conexión | 10 Mbps | 25 Mbps |

### Software Requerido

| Paquete | Versión | Propósito |
|---------|---------|-----------|
| Python | 3.11.x | Runtime principal |
| `openai` | 1.35.x | SDK AsyncOpenAI + Structured Outputs |
| `pydantic` | 2.7.x | Validación de esquemas |
| `tenacity` | 8.3.x | Reintentos con backoff exponencial |
| `httpx` | 0.27.x | Cliente HTTP asíncrono (dependencia de openai) |
| `python-dotenv` | 1.0.x | Gestión segura de credenciales |

### Comandos de Configuración del Entorno

```bash
# 1. Crear directorio del lab
mkdir lab-03-00-01 && cd lab-03-00-01

# 2. Crear entorno virtual aislado
python3.11 -m venv .venv

# 3. Activar el entorno virtual
# En macOS/Linux:
source .venv/bin/activate
# En Windows (PowerShell):
# .venv\Scripts\Activate.ps1

# 4. Actualizar pip
pip install --upgrade pip

# 5. Instalar dependencias
pip install openai==1.35.3 pydantic==2.7.4 tenacity==8.3.0 httpx==0.27.0 python-dotenv==1.0.1

# 6. Verificar instalaciones
python -c "import openai, pydantic, tenacity, httpx; print('OK')"
```

### Configuración de Credenciales

```bash
# 7. Crear archivo .env (NUNCA subir al repositorio)
cat > .env << 'EOF'
OPENAI_API_KEY=sk-proj-REEMPLAZA_CON_TU_CLAVE_REAL
EOF

# 8. Crear .gitignore
cat > .gitignore << 'EOF'
.env
.venv/
__pycache__/
*.pyc
*.pyo
.pytest_cache/
*.log
EOF
```

> 🔒 **Seguridad**: Verifica que `.env` aparece en `.gitignore` **antes** de hacer cualquier `git add`. El instructor revisará esto al inicio de la sesión.

---

## 6. Desarrollo Paso a Paso

### Paso 1: Definir los Modelos Pydantic v2

**Objetivo**: Crear los esquemas de datos que representarán un reporte de incidente técnico y validarán automáticamente la respuesta del LLM.

#### Instrucciones

1. Crea el archivo `structured_llm_client.py` en el directorio del lab.

2. Agrega las importaciones y el modelo `IncidentReport`:

```python
# structured_llm_client.py
"""
Cliente LLM asíncrono con respuestas estructuradas y reintentos exponenciales.
Caso de uso: extracción de información de incidentes técnicos.
"""

from __future__ import annotations

import asyncio
import logging
import os
import time
from typing import List, Optional

from dotenv import load_dotenv
from openai import AsyncOpenAI, RateLimitError, APIStatusError
from pydantic import BaseModel, Field, ValidationError
from tenacity import (
    retry,
    retry_if_exception_type,
    stop_after_attempt,
    wait_exponential,
    before_sleep_log,
)

# ── Configuración de logging ───────────────────────────────────────────────
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
)
logger = logging.getLogger(__name__)

# ── Carga de variables de entorno ──────────────────────────────────────────
load_dotenv()


# ── Modelos Pydantic v2 ────────────────────────────────────────────────────
class IncidentReport(BaseModel):
    """
    Esquema estructurado para reportes de incidentes técnicos.
    Todos los campos son validados automáticamente por Pydantic v2.
    """

    severity: str = Field(
        description="Nivel de severidad del incidente",
        pattern="^(low|medium|high|critical)$",
    )
    affected_systems: List[str] = Field(
        description="Lista de sistemas o servicios afectados por el incidente",
        min_length=1,
    )
    root_cause: str = Field(
        description="Causa raíz identificada o probable del incidente",
        min_length=10,
    )
    recommended_actions: List[str] = Field(
        description="Lista de acciones recomendadas para resolver el incidente",
        min_length=1,
    )
    estimated_resolution_hours: Optional[float] = Field(
        default=None,
        description="Tiempo estimado de resolución en horas (None si no se puede determinar)",
        ge=0.0,
    )
```

> **Nota sobre `severity`**: Usamos `pattern` con una expresión regular en lugar de `Literal` para mantener compatibilidad total con el esquema JSON que OpenAI Structured Outputs genera internamente. Ambos enfoques son válidos en Pydantic v2.

3. Guarda el archivo. Verifica que los modelos son válidos ejecutando:

```bash
python -c "
from structured_llm_client import IncidentReport
r = IncidentReport(
    severity='high',
    affected_systems=['API Gateway', 'Auth Service'],
    root_cause='Certificado SSL expirado en el balanceador de carga',
    recommended_actions=['Renovar certificado', 'Configurar alerta de expiración'],
    estimated_resolution_hours=2.5
)
print(r.model_dump_json(indent=2))
"
```

#### Salida Esperada

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

#### Verificación
- El modelo se instancia sin errores.
- `model_dump_json()` produce JSON válido con todos los campos.
- Intenta crear una instancia con `severity="CRITICAL"` (mayúsculas): debe lanzar `ValidationError`.

---

### Paso 2: Implementar la Función de Extracción con Structured Outputs y Reintentos

**Objetivo**: Crear la función asíncrona principal que llama a OpenAI con Structured Outputs y aplica el decorador de reintentos exponenciales.

#### Instrucciones

1. Agrega el cliente asíncrono y la función `extract_incident_data` al final de `structured_llm_client.py`:

```python
# ── Cliente AsyncOpenAI (instancia global reutilizable) ────────────────────
# El cliente lee OPENAI_API_KEY automáticamente desde el entorno
_async_client = AsyncOpenAI()

# ── Función principal con reintentos exponenciales ─────────────────────────
@retry(
    wait=wait_exponential(multiplier=1, min=2, max=60),
    stop=stop_after_attempt(5),
    retry=retry_if_exception_type((RateLimitError, APIStatusError)),
    before_sleep=before_sleep_log(logger, logging.WARNING),
    reraise=True,
)
async def extract_incident_data(text: str) -> IncidentReport:
    """
    Extrae información estructurada de una descripción de incidente técnico.

    Args:
        text: Descripción en texto libre del incidente técnico.

    Returns:
        IncidentReport validado con los campos extraídos por el LLM.

    Raises:
        ValidationError: Si el LLM no cumple el esquema Pydantic esperado.
        RateLimitError: Si se superan los reintentos por rate limit.
        APIStatusError: Si el servidor retorna 5xx después de los reintentos.
    """
    logger.debug("Enviando texto al LLM para extracción estructurada...")

    try:
        response = await _async_client.beta.chat.completions.parse(
            model="gpt-4o-mini",
            messages=[
                {
                    "role": "system",
                    "content": (
                        "Eres un experto en análisis de incidentes técnicos de TI. "
                        "Tu tarea es extraer información estructurada de descripciones "
                        "de incidentes. Sé preciso y conciso. "
                        "Para 'severity' usa ÚNICAMENTE: low, medium, high, o critical. "
                        "Para 'estimated_resolution_hours' usa null si no hay suficiente "
                        "información para estimarlo."
                    ),
                },
                {
                    "role": "user",
                    "content": f"Extrae la información estructurada del siguiente incidente:\n\n{text}",
                },
            ],
            response_format=IncidentReport,
            temperature=0.1,   # Baja temperatura para mayor determinismo
            max_tokens=500,
        )

        # Structured Outputs garantiza que el objeto ya está parseado y validado
        parsed_report = response.choices[0].message.parsed

        if parsed_report is None:
            # Caso edge: el modelo retornó refusal en lugar de datos
            raise ValidationError.from_exception_data(
                title="IncidentReport",
                input_data={"error": "El modelo rechazó generar la respuesta (refusal)."},
                input_type="python",
            )

        logger.info(
            "Extracción exitosa | severity=%s | sistemas=%d",
            parsed_report.severity,
            len(parsed_report.affected_systems),
        )
        return parsed_report

    except ValidationError as ve:
        # El LLM no cumplió el esquema: loguear y propagar
        logger.error("ValidationError en la respuesta del LLM: %s", ve.errors())
        raise
```

> **¿Por qué `beta.chat.completions.parse`?** Esta es la forma recomendada por OpenAI para Structured Outputs en el SDK Python 1.35.x. Internamente convierte el modelo Pydantic a JSON Schema, lo envía como `response_format`, y deserializa la respuesta directamente al objeto Pydantic. El resultado está disponible en `response.choices[0].message.parsed`.

> **Parámetros de generación**: `temperature=0.1` minimiza la variabilidad en la extracción de datos estructurados (queremos determinismo, no creatividad). `max_tokens=500` es suficiente para el esquema definido y evita respuestas innecesariamente largas.

#### Verificación del Decorador de Reintentos

La configuración `@retry` implementa la siguiente estrategia:

| Intento | Espera mínima | Espera máxima | Condición |
|---------|--------------|--------------|-----------|
| 1 → 2  | 2 segundos   | —            | `RateLimitError` o `APIStatusError` |
| 2 → 3  | 4 segundos   | —            | ídem |
| 3 → 4  | 8 segundos   | —            | ídem |
| 4 → 5  | 16 segundos  | 60 segundos  | ídem |
| 5+     | —            | —            | Relanza la excepción |

---

### Paso 3: Implementar el Procesamiento Batch Concurrente

**Objetivo**: Crear la función `process_batch_async` que procesa múltiples incidentes en paralelo usando `asyncio.gather()` con control de concurrencia mediante `asyncio.Semaphore`.

#### Instrucciones

1. Agrega la función de procesamiento batch al final de `structured_llm_client.py`:

```python
# ── Procesamiento batch concurrente ───────────────────────────────────────
async def process_batch_async(
    incidents: List[str],
    max_concurrent: int = 3,
) -> List[IncidentReport | Exception]:
    """
    Procesa una lista de descripciones de incidentes de forma concurrente.

    Usa asyncio.Semaphore para limitar la concurrencia y respetar los
    rate limits de la API de OpenAI (típicamente 3-5 RPM en tier gratuito).

    Args:
        incidents: Lista de descripciones de incidentes en texto libre.
        max_concurrent: Número máximo de solicitudes simultáneas (default: 3).

    Returns:
        Lista de IncidentReport (o Exception si un ítem falló individualmente).
        El orden de los resultados corresponde al orden de entrada.
    """
    semaphore = asyncio.Semaphore(max_concurrent)

    async def _process_with_semaphore(
        idx: int, text: str
    ) -> IncidentReport | Exception:
        async with semaphore:
            logger.info("Procesando incidente %d/%d...", idx + 1, len(incidents))
            try:
                return await extract_incident_data(text)
            except Exception as exc:
                logger.error(
                    "Error en incidente %d: %s: %s",
                    idx + 1,
                    type(exc).__name__,
                    exc,
                )
                return exc  # Retorna la excepción para no cancelar el batch completo

    tasks = [
        _process_with_semaphore(i, text) for i, text in enumerate(incidents)
    ]

    results = await asyncio.gather(*tasks)
    return list(results)
```

> **Diseño de `return_exceptions`**: En lugar de usar `asyncio.gather(*tasks, return_exceptions=True)`, aquí manejamos las excepciones dentro de cada tarea individual. Esto nos da control granular sobre el logging y permite que el batch continúe aunque algunos ítems fallen.

> **`asyncio.Semaphore(3)`**: Garantiza que como máximo 3 solicitudes estén en vuelo simultáneamente. Esto es crítico para respetar los rate limits de OpenAI, especialmente en cuentas con límites bajos (Tier 1: ~500 RPM para gpt-4o-mini, pero el semáforo protege contra ráfagas).

---

### Paso 4: Crear el Script de Prueba con Medición de Rendimiento

**Objetivo**: Implementar el script principal que ejecuta el batch de 10 incidentes, mide tiempos y compara procesamiento concurrente vs. secuencial.

#### Instrucciones

1. Crea el archivo `test_batch.py` en el mismo directorio:

```python
# test_batch.py
"""
Script de prueba para structured_llm_client.py.
Procesa 10 incidentes técnicos y mide el tiempo de procesamiento
concurrente vs. secuencial.
"""

import asyncio
import time
import json
from typing import List

from pydantic import ValidationError
from structured_llm_client import (
    IncidentReport,
    extract_incident_data,
    process_batch_async,
    logger,
)

# ── Dataset de 10 incidentes técnicos de ejemplo ──────────────────────────
SAMPLE_INCIDENTS: List[str] = [
    # 1
    (
        "A las 14:32 UTC el servicio de autenticación comenzó a retornar errores 503. "
        "Los logs muestran que la base de datos PostgreSQL en el nodo primario agotó "
        "las conexiones disponibles (pool de 100 conexiones). Todos los usuarios "
        "activos fueron desconectados. El equipo de base de datos está investigando "
        "una posible fuga de conexiones introducida en el deploy de las 13:00 UTC."
    ),
    # 2
    (
        "El pipeline de CI/CD falló en producción a las 09:15. "
        "El contenedor Docker del servicio de pagos no pudo iniciarse por un "
        "error de memoria insuficiente (OOM). El pod fue terminado por Kubernetes. "
        "Revisar los límites de memoria en el deployment manifest. "
        "Impacto: transacciones de pago bloqueadas por 23 minutos."
    ),
    # 3
    (
        "Alerta crítica: el certificado SSL del dominio api.empresa.com expiró "
        "a las 00:00 UTC. Los clientes móviles reciben errores SSL_ERROR_RX_RECORD_TOO_LONG. "
        "El servicio de renovación automática de Let's Encrypt falló la semana pasada "
        "sin generar alertas. Necesitamos renovación manual urgente y revisar la "
        "configuración del cron job de renovación."
    ),
    # 4
    (
        "Degradación de rendimiento en el microservicio de recomendaciones. "
        "La latencia P99 subió de 200ms a 4500ms. El análisis de trazas muestra "
        "que las consultas a Redis están tardando 3000ms en promedio. "
        "Posible causa: la instancia de Redis alcanzó el 95% de memoria y "
        "está ejecutando eviction agresiva de claves."
    ),
    # 5
    (
        "El bucket S3 de backups dejó de recibir archivos desde las 18:00 de ayer. "
        "El script de backup reporta AccessDenied. Verificamos que las credenciales "
        "IAM del servicio expiraron. El rol IAM asociado tenía una política de "
        "expiración de 90 días que no fue renovada. No hay pérdida de datos "
        "pero los últimos backups tienen 14 horas de antigüedad."
    ),
    # 6
    (
        "Caída total del servicio de notificaciones push. "
        "El proveedor externo (Firebase Cloud Messaging) retorna error 401 "
        "para todas las solicitudes. La clave de API de FCM fue rotada "
        "manualmente por el equipo de seguridad sin actualizar la variable "
        "de entorno en el servidor de producción. Ningún usuario está "
        "recibiendo notificaciones desde hace 6 horas."
    ),
    # 7
    (
        "El servicio de búsqueda Elasticsearch muestra estado yellow. "
        "Uno de los tres nodos del clúster se desconectó por un fallo de disco. "
        "Los índices de productos y usuarios tienen shards sin asignar. "
        "Las búsquedas funcionan pero con mayor latencia. "
        "Se requiere reemplazar el nodo fallido y reasignar los shards."
    ),
    # 8
    (
        "Error masivo en el proceso de facturación mensual. "
        "El job batch que genera facturas en PDF falló al procesar 847 de 1200 facturas. "
        "El error es NullPointerException en el módulo de formateo de direcciones "
        "cuando el campo 'estado/provincia' está vacío. "
        "Clientes afectados no recibirán su factura del mes."
    ),
    # 9
    (
        "Incremento anómalo en el tráfico de la API pública: "
        "de 1000 RPM normal a 45000 RPM en los últimos 20 minutos. "
        "El WAF no bloqueó las solicitudes porque provienen de IPs distribuidas. "
        "Parece un ataque de scraping coordinado. El servidor está respondiendo "
        "lento pero operativo. Se necesita implementar rate limiting por user-agent."
    ),
    # 10
    (
        "El proceso de sincronización entre la base de datos principal (MySQL) "
        "y el data warehouse (BigQuery) lleva 8 horas atrasado. "
        "El conector de Kafka Connect muestra lag creciente en el topic de eventos. "
        "Un mensaje malformado en el topic está bloqueando el consumer. "
        "Los dashboards de analytics muestran datos desactualizados."
    ),
]


async def run_concurrent_benchmark() -> tuple[List[IncidentReport | Exception], float]:
    """Ejecuta el procesamiento concurrente y retorna resultados + tiempo."""
    start = time.perf_counter()
    results = await process_batch_async(SAMPLE_INCIDENTS, max_concurrent=3)
    elapsed = time.perf_counter() - start
    return results, elapsed


async def run_sequential_benchmark() -> tuple[List[IncidentReport | Exception], float]:
    """Ejecuta el procesamiento secuencial y retorna resultados + tiempo."""
    start = time.perf_counter()
    results = []
    for i, incident in enumerate(SAMPLE_INCIDENTS):
        logger.info("Procesando secuencialmente incidente %d/%d...", i + 1, len(SAMPLE_INCIDENTS))
        try:
            result = await extract_incident_data(incident)
            results.append(result)
        except Exception as exc:
            logger.error("Error en incidente %d: %s", i + 1, exc)
            results.append(exc)
    elapsed = time.perf_counter() - start
    return results, elapsed


def print_results_summary(
    results: List[IncidentReport | Exception],
    mode: str,
    elapsed: float,
) -> None:
    """Imprime un resumen legible de los resultados del batch."""
    successes = [r for r in results if isinstance(r, IncidentReport)]
    failures = [r for r in results if isinstance(r, Exception)]

    print(f"\n{'=' * 70}")
    print(f"  MODO: {mode.upper()}")
    print(f"{'=' * 70}")
    print(f"  Total procesados : {len(results)}")
    print(f"  Exitosos         : {len(successes)}")
    print(f"  Fallidos         : {len(failures)}")
    print(f"  Tiempo total     : {elapsed:.2f} segundos")
    print(f"  Promedio/incidente: {elapsed / len(results):.2f} segundos")
    print(f"{'=' * 70}")

    # Distribución de severidad
    severity_counts: dict[str, int] = {}
    for report in successes:
        severity_counts[report.severity] = severity_counts.get(report.severity, 0) + 1

    print("\n  Distribución de Severidad:")
    for sev in ["critical", "high", "medium", "low"]:
        count = severity_counts.get(sev, 0)
        bar = "█" * count
        print(f"    {sev:10s} | {bar} ({count})")

    # Detalle de cada reporte exitoso
    print(f"\n  Detalle de Reportes Extraídos:")
    for i, result in enumerate(results):
        print(f"\n  Incidente #{i + 1}:")
        if isinstance(result, IncidentReport):
            print(f"    severity             : {result.severity}")
            print(f"    affected_systems     : {', '.join(result.affected_systems)}")
            print(f"    root_cause           : {result.root_cause[:80]}...")
            print(f"    recommended_actions  : {len(result.recommended_actions)} acciones")
            print(f"    est. resolution (h)  : {result.estimated_resolution_hours}")
        else:
            print(f"    ❌ ERROR: {type(result).__name__}: {result}")

    if failures:
        print(f"\n  ⚠️  Errores encontrados ({len(failures)}):")
        for exc in failures:
            print(f"    - {type(exc).__name__}: {exc}")


async def main() -> None:
    """Punto de entrada principal del script de prueba."""
    print("\n" + "=" * 70)
    print("  LAB 03-00-01: Cliente LLM Asíncrono con Structured Outputs")
    print("  Caso de uso: Extracción de Incidentes Técnicos")
    print("=" * 70)
    print(f"  Total de incidentes a procesar: {len(SAMPLE_INCIDENTS)}")
    print(f"  Modelo: gpt-4o-mini | Concurrencia máxima: 3")
    print("=" * 70)

    # ── Procesamiento CONCURRENTE ──────────────────────────────────────────
    print("\n[1/2] Iniciando procesamiento CONCURRENTE...")
    concurrent_results, concurrent_time = await run_concurrent_benchmark()
    print_results_summary(concurrent_results, "Concurrente", concurrent_time)

    # ── Procesamiento SECUENCIAL ───────────────────────────────────────────
    print("\n[2/2] Iniciando procesamiento SECUENCIAL...")
    sequential_results, sequential_time = await run_sequential_benchmark()
    print_results_summary(sequential_results, "Secuencial", sequential_time)

    # ── Comparación Final ──────────────────────────────────────────────────
    speedup = sequential_time / concurrent_time if concurrent_time > 0 else 0
    print(f"\n{'=' * 70}")
    print(f"  COMPARACIÓN DE RENDIMIENTO")
    print(f"{'=' * 70}")
    print(f"  Tiempo concurrente : {concurrent_time:.2f}s")
    print(f"  Tiempo secuencial  : {sequential_time:.2f}s")
    print(f"  Speedup            : {speedup:.2f}x más rápido")
    print(f"  Ahorro de tiempo   : {sequential_time - concurrent_time:.2f}s")
    print(f"{'=' * 70}\n")

    # ── Exportar resultados a JSON ─────────────────────────────────────────
    output = {
        "benchmark": {
            "concurrent_seconds": round(concurrent_time, 3),
            "sequential_seconds": round(sequential_time, 3),
            "speedup_factor": round(speedup, 2),
        },
        "concurrent_results": [
            r.model_dump() if isinstance(r, IncidentReport) else {"error": str(r)}
            for r in concurrent_results
        ],
    }

    with open("results.json", "w", encoding="utf-8") as f:
        json.dump(output, f, indent=2, ensure_ascii=False)

    print(f"  ✅ Resultados exportados a results.json")


if __name__ == "__main__":
    asyncio.run(main())
```

#### Salida Esperada (parcial)

```
======================================================================
  LAB 03-00-01: Cliente LLM Asíncrono con Structured Outputs
  Caso de uso: Extracción de Incidentes Técnicos
======================================================================
  Total de incidentes a procesar: 10
  Modelo: gpt-4o-mini | Concurrencia máxima: 3
======================================================================

[1/2] Iniciando procesamiento CONCURRENTE...
2024-XX-XX [INFO] structured_llm_client: Procesando incidente 1/10...
2024-XX-XX [INFO] structured_llm_client: Procesando incidente 2/10...
2024-XX-XX [INFO] structured_llm_client: Procesando incidente 3/10...
...

======================================================================
  MODO: CONCURRENTE
======================================================================
  Total procesados : 10
  Exitosos         : 10
  Fallidos         : 0
  Tiempo total     : ~18.5 segundos
  Promedio/incidente: ~1.85 segundos
======================================================================

  Distribución de Severidad:
    critical   | ███ (3)
    high       | ████ (4)
    medium     | ██ (2)
    low        |  (1)
...

  COMPARACIÓN DE RENDIMIENTO
======================================================================
  Tiempo concurrente : 18.5s
  Tiempo secuencial  : 52.3s
  Speedup            : 2.83x más rápido
  Ahorro de tiempo   : 33.8s
======================================================================

  ✅ Resultados exportados a results.json
```

> **Nota**: Los tiempos variarán según la latencia de la API de OpenAI y las condiciones de red. El speedup típico es entre 2x y 4x con `Semaphore(3)`.

#### Verificación

```bash
# Ejecutar el script completo
python test_batch.py

# Verificar que se generó el archivo de resultados
python -c "
import json
with open('results.json') as f:
    data = json.load(f)
print('Speedup:', data['benchmark']['speedup_factor'])
print('Resultados:', len(data['concurrent_results']))
"
```

---

### Paso 5: Verificar el Manejo de Errores de Validación

**Objetivo**: Comprobar que el sistema maneja correctamente los casos donde la respuesta del LLM no cumple el esquema Pydantic.

#### Instrucciones

1. Crea el archivo `test_validation.py` para probar el manejo de errores:

```python
# test_validation.py
"""
Pruebas de manejo de errores de validación Pydantic.
"""

import asyncio
from pydantic import ValidationError
from structured_llm_client import IncidentReport, logger


def test_pydantic_validation_errors():
    """Verifica que Pydantic v2 rechaza datos inválidos correctamente."""

    print("\n=== Pruebas de Validación Pydantic v2 ===\n")

    # Caso 1: severity inválida
    try:
        IncidentReport(
            severity="CRITICAL",  # Debe ser 'critical' en minúsculas
            affected_systems=["API"],
            root_cause="Error en el servidor de base de datos principal",
            recommended_actions=["Reiniciar el servicio"],
        )
        print("❌ FALLO: Debería haber lanzado ValidationError")
    except ValidationError as e:
        print(f"✅ Caso 1 (severity inválida): ValidationError capturado correctamente")
        print(f"   Campos con error: {[err['loc'] for err in e.errors()]}")

    # Caso 2: affected_systems vacío
    try:
        IncidentReport(
            severity="high",
            affected_systems=[],  # min_length=1
            root_cause="Error en el servidor de base de datos principal",
            recommended_actions=["Reiniciar el servicio"],
        )
        print("❌ FALLO: Debería haber lanzado ValidationError")
    except ValidationError as e:
        print(f"✅ Caso 2 (lista vacía): ValidationError capturado correctamente")
        print(f"   Campos con error: {[err['loc'] for err in e.errors()]}")

    # Caso 3: estimated_resolution_hours negativo
    try:
        IncidentReport(
            severity="low",
            affected_systems=["Servicio de logs"],
            root_cause="Disco lleno en servidor de logs secundario",
            recommended_actions=["Limpiar logs antiguos", "Aumentar capacidad de disco"],
            estimated_resolution_hours=-1.0,  # ge=0.0
        )
        print("❌ FALLO: Debería haber lanzado ValidationError")
    except ValidationError as e:
        print(f"✅ Caso 3 (horas negativas): ValidationError capturado correctamente")
        print(f"   Campos con error: {[err['loc'] for err in e.errors()]}")

    # Caso 4: Instancia válida con estimated_resolution_hours=None
    try:
        report = IncidentReport(
            severity="medium",
            affected_systems=["Servicio de reportes"],
            root_cause="Timeout en la generación de reportes PDF por alta carga",
            recommended_actions=["Escalar horizontalmente el servicio de reportes"],
            estimated_resolution_hours=None,  # Optional, debe aceptarse
        )
        print(f"✅ Caso 4 (None válido): Instancia creada correctamente")
        print(f"   estimated_resolution_hours = {report.estimated_resolution_hours}")
    except ValidationError as e:
        print(f"❌ FALLO: No debería haber lanzado ValidationError: {e}")

    print("\n=== Fin de Pruebas de Validación ===\n")


if __name__ == "__main__":
    test_pydantic_validation_errors()
```

2. Ejecuta las pruebas de validación:

```bash
python test_validation.py
```

#### Salida Esperada

```
=== Pruebas de Validación Pydantic v2 ===

✅ Caso 1 (severity inválida): ValidationError capturado correctamente
   Campos con error: [('severity',)]
✅ Caso 2 (lista vacía): ValidationError capturado correctamente
   Campos con error: [('affected_systems',)]
✅ Caso 3 (horas negativas): ValidationError capturado correctamente
   Campos con error: [('estimated_resolution_hours',)]
✅ Caso 4 (None válido): Instancia creada correctamente
   estimated_resolution_hours = None

=== Fin de Pruebas de Validación ===
```

#### Verificación
- Los 4 casos deben mostrar el resultado esperado (✅).
- Ningún caso debe mostrar ❌.

---

## 7. Validación y Pruebas

### Checklist de Validación Completa

Ejecuta los siguientes comandos en orden para verificar que todo el lab funciona correctamente:

```bash
# 1. Verificar que el módulo importa sin errores
python -c "import structured_llm_client; print('✅ Módulo importado correctamente')"

# 2. Verificar que el modelo Pydantic funciona
python -c "
from structured_llm_client import IncidentReport
r = IncidentReport(
    severity='critical',
    affected_systems=['DB', 'API'],
    root_cause='Fallo en el nodo primario de la base de datos por corrupción de índices',
    recommended_actions=['Failover al nodo secundario', 'Reconstruir índices'],
    estimated_resolution_hours=4.0
)
assert r.severity == 'critical'
assert len(r.affected_systems) == 2
print('✅ Modelo Pydantic validado correctamente')
"

# 3. Ejecutar pruebas de validación de errores
python test_validation.py

# 4. Ejecutar el batch completo (requiere OPENAI_API_KEY)
python test_batch.py

# 5. Verificar el archivo de resultados generado
python -c "
import json
with open('results.json') as f:
    data = json.load(f)
assert 'benchmark' in data
assert 'concurrent_results' in data
assert len(data['concurrent_results']) == 10
speedup = data['benchmark']['speedup_factor']
print(f'✅ results.json válido | Speedup: {speedup}x')
assert speedup > 1.0, 'El procesamiento concurrente debe ser más rápido que el secuencial'
print('✅ Speedup concurrente confirmado')
"
```

### Criterios de Éxito

| Criterio | Condición de Éxito |
|----------|--------------------|
| Importación del módulo | Sin errores de importación |
| Validación Pydantic | 4/4 casos de prueba pasan |
| Extracción LLM | ≥ 9/10 incidentes procesados exitosamente |
| Structured Outputs | Todos los reportes exitosos tienen `severity` en `{low,medium,high,critical}` |
| Speedup concurrente | `speedup_factor > 1.0` |
| Archivo de resultados | `results.json` generado con estructura correcta |

---

## 8. Solución de Problemas

### Problema 1: `openai.AuthenticationError` — La API Key no es reconocida

**Síntoma**: Al ejecutar `test_batch.py`, el programa falla inmediatamente con:
```
openai.AuthenticationError: Error code: 401 - {'error': {'message': 'Incorrect API key provided'}}
```

**Causa**: La variable de entorno `OPENAI_API_KEY` no está configurada correctamente, o el archivo `.env` no está en el directorio de trabajo actual, o la clave tiene espacios/caracteres extra al copiarla.

**Solución**:
```bash
# Paso 1: Verificar que el archivo .env existe y tiene el formato correcto
cat .env
# Debe mostrar: OPENAI_API_KEY=sk-proj-...  (sin comillas, sin espacios)

# Paso 2: Verificar que Python lee la variable correctamente
python -c "
from dotenv import load_dotenv
import os
load_dotenv()
key = os.getenv('OPENAI_API_KEY', 'NO_ENCONTRADA')
print(f'Longitud de la clave: {len(key)} caracteres')
print(f'Prefijo: {key[:10]}...')
"
# Una clave válida de OpenAI tiene ~164 caracteres y comienza con 'sk-proj-' o 'sk-'

# Paso 3: Si la clave es correcta pero sigue fallando, verificar que está activa
# en platform.openai.com/api-keys
```

---

### Problema 2: `tenacity.RetryError` — Se agotaron todos los reintentos por Rate Limit

**Síntoma**: Durante el procesamiento del batch, varios incidentes fallan con:
```
tenacity.RetryError: RetryError[<Future at 0x... state=finished raised RateLimitError>]
2024-XX-XX [WARNING] structured_llm_client: Retrying ... after 2.0 seconds (attempt 1)
2024-XX-XX [WARNING] structured_llm_client: Retrying ... after 4.0 seconds (attempt 2)
...
```

**Causa**: La cuenta de OpenAI está en el Tier gratuito o Tier 1, que tiene límites bajos de RPM (requests per minute). Con `Semaphore(3)` y 10 incidentes, se pueden generar ráfagas que superan el límite.

**Solución**:
```python
# Opción 1: Reducir la concurrencia en process_batch_async
# En test_batch.py, cambiar:
results = await process_batch_async(SAMPLE_INCIDENTS, max_concurrent=1)
# Esto procesa de forma efectivamente secuencial pero con el mismo código

# Opción 2: Aumentar el tiempo mínimo de espera en el decorador @retry
# En structured_llm_client.py, cambiar:
@retry(
    wait=wait_exponential(multiplier=2, min=5, max=120),  # min=5s en lugar de 2s
    stop=stop_after_attempt(5),
    retry=retry_if_exception_type((RateLimitError, APIStatusError)),
    before_sleep=before_sleep_log(logger, logging.WARNING),
    reraise=True,
)

# Opción 3: Agregar un sleep entre grupos de solicitudes
# En _process_with_semaphore, agregar después de async with semaphore:
async with semaphore:
    await asyncio.sleep(0.5)  # 500ms de pausa entre solicitudes
    return await extract_incident_data(text)
```

> **Verificación del tier**: En [platform.openai.com/account/rate-limits](https://platform.openai.com/account/rate-limits) puedes ver tus límites actuales. Para `gpt-4o-mini` en Tier 1: 500 RPM, suficiente para este lab con `Semaphore(3)`.

---

## 9. Limpieza del Entorno

```bash
# 1. Desactivar el entorno virtual
deactivate

# 2. (Opcional) Eliminar el entorno virtual para liberar espacio
rm -rf .venv/

# 3. Verificar que no hay API keys en el código antes de cualquier commit
grep -r "sk-" . --include="*.py" --exclude-dir=".venv"
# Este comando NO debe retornar ninguna línea con claves reales

# 4. (Opcional) Limpiar archivos de caché de Python
find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null
find . -name "*.pyc" -delete 2>/dev/null

# 5. Verificar que .gitignore protege los archivos sensibles
git status --short 2>/dev/null || echo "No es un repositorio git"
# El archivo .env NO debe aparecer en la lista de archivos tracked
```

---

## 10. Resumen

### Lo que Construiste

En este laboratorio implementaste un módulo Python de nivel producción (`structured_llm_client.py`) que integra tres capacidades clave:

| Capacidad | Implementación | Beneficio |
|-----------|---------------|-----------|
| **Respuestas estructuradas** | `AsyncOpenAI.beta.chat.completions.parse()` + Pydantic v2 | Garantiza que el LLM retorna datos validados y tipados |
| **Resiliencia ante fallos** | `@retry` con `wait_exponential` de tenacity | El sistema se recupera automáticamente de errores transitorios |
| **Procesamiento concurrente** | `asyncio.gather()` + `asyncio.Semaphore(3)` | 2-4x más rápido que el procesamiento secuencial |

### Conceptos Clave Aprendidos

- **Structured Outputs de OpenAI** convierte automáticamente un modelo Pydantic v2 en JSON Schema y garantiza que la respuesta cumple el esquema, eliminando la necesidad de parsear JSON manualmente.
- **`temperature=0.1`** en tareas de extracción estructurada maximiza el determinismo; valores altos introducen variabilidad no deseada en campos como `severity`.
- El **backoff exponencial** (2s → 4s → 8s → 16s → 60s) es el estándar de la industria para manejar rate limits: evita sobrecargar el servidor y da tiempo para que los límites se recuperen.
- **`asyncio.Semaphore`** es el mecanismo correcto para controlar la concurrencia en código asíncrono Python; no confundir con `threading.Semaphore` que es para código síncrono con hilos.
- El patrón de retornar `Exception` en lugar de lanzarla dentro de `asyncio.gather` permite que el batch completo termine aunque algunos ítems individuales fallen.

### Recursos Adicionales

- [OpenAI Structured Outputs — Documentación oficial](https://platform.openai.com/docs/guides/structured-outputs)
- [Pydantic v2 — Validators y Field constraints](https://docs.pydantic.dev/latest/concepts/fields/)
- [tenacity — Documentación completa de estrategias de retry](https://tenacity.readthedocs.io/en/latest/)
- [asyncio — Primitivas de sincronización (Semaphore, Lock, Event)](https://docs.python.org/3/library/asyncio-sync.html)
- [OpenAI Rate Limits — Tiers y límites por modelo](https://platform.openai.com/docs/guides/rate-limits)

---
LAB_END---
