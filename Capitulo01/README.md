<div align="center">

# 🧪 Laboratorio 1

## Selección técnica de modelo y estimación de costo, latencia y contexto

**IA Generativa · Nivel Intermedio · Capítulo 1**

![Nivel](https://img.shields.io/badge/Nivel-Intermedio-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio vas a probar **OpenAI**, **Gemini** y **Claude** desde código Python, medir latencia, estimar tokens, calcular costos y construir una matriz técnica de decisión. No uses datos sensibles reales ni compartas tus API keys.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Selección técnica de modelos</td>
<td width="25%"><strong>⏱️ Duración</strong><br>45 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar y evaluar</td>
<td width="25%"><strong>📦 Entregable</strong><br>Scripts + CSV + matriz</td>
</tr>
</table>

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a comparar de forma técnica tres proveedores de modelos de IA generativa: **OpenAI**, **Google Gemini** y **Anthropic Claude**.

A diferencia de una comparación solamente teórica, en este laboratorio vas a preparar un ambiente local en **Windows**, usando **Visual Studio Code** y **Git Bash**, para ejecutar pruebas reales mediante código Python. Vas a enviar los mismos prompts a cada proveedor, medir latencia, estimar tokens de entrada y salida, calcular costo aproximado mensual y registrar observaciones de calidad, privacidad y operación.

El objetivo es que no elijas un modelo solo por popularidad. Vas a construir evidencia técnica mínima para justificar qué modelo conviene para tres casos de uso:

1. Resumen de contenido.
2. Soporte técnico asistido.
3. Análisis documental.

Al finalizar, tendrás una matriz comparativa y scripts funcionales para probar OpenAI, Gemini y Claude desde una terminal local.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Preparar un entorno local en Windows para consumir APIs de IA generativa.
2. Configurar claves de API mediante variables de entorno.
3. Ejecutar pruebas básicas contra OpenAI, Gemini y Claude usando Python.
4. Medir latencia de respuesta por proveedor y caso de uso.
5. Estimar tokens de entrada y salida por solicitud.
6. Calcular costo mensual aproximado por modelo y caso de uso.
7. Comparar modelos con criterios técnicos: costo, latencia, contexto, privacidad y operación.
8. Recomendar un modelo por caso de uso con argumentos técnicos.
9. Identificar qué partes del código deben modificarse para cambiar modelo, prompt, volumen o proveedor.
10. Documentar resultados en una matriz profesional de decisión.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Tener conocimientos básicos de IA generativa.
2. Comprender qué son tokens de entrada y tokens de salida.
3. Conocer el concepto de ventana de contexto.
4. Tener nociones básicas de uso de terminal.
5. Tener conocimientos básicos de Python.
6. Tener cuentas o acceso a API keys de:
   - OpenAI.
   - Google Gemini API.
   - Anthropic Claude API.
7. Tener acceso a internet.
8. Conocer cómo abrir una carpeta de proyecto en Visual Studio Code.

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
| Internet | Requerido para consumir las APIs |

---

### 🧰 Software

| Software | Uso |
|---|---|
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Python 3.11 o superior | Ejecución de scripts |
| pip | Instalación de dependencias |
| Microsoft Excel, Google Sheets o LibreOffice Calc | Matriz de comparación |
| Cuenta/API key de OpenAI | Pruebas con modelos OpenAI |
| Cuenta/API key de Google Gemini | Pruebas con modelos Gemini |
| Cuenta/API key de Anthropic | Pruebas con modelos Claude |

---

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 45 minutos |
| Complejidad | Intermedia |
| Nivel de Bloom | Aplicar, analizar, evaluar y justificar |
| Capítulo | Capítulo 1 |
| Modalidad | Individual o equipos de 2 personas |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Proveedores usados | OpenAI, Gemini y Claude |
| Entregable principal | Matriz técnica comparativa |
| Entregable secundario | Scripts de prueba funcionales |
| Tipo de práctica | Técnica, comparativa y aplicada |

---

## 🛡️ Consideraciones

Antes de comenzar, toma en cuenta lo siguiente:

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>No compartas claves ni subas `.env`.</td>
<td><strong>💸 Costo</strong><br>Cada ejecución puede consumir saldo o cuota.</td>
<td><strong>📏 Comparación justa</strong><br>Usa los mismos prompts en los tres proveedores.</td>
</tr>
</table>


1. **No compartas tus API keys.** Las claves son personales o institucionales y deben tratarse como credenciales sensibles.
2. **No pegues tus API keys dentro del código.** Siempre usa el archivo `.env`.
3. **No entregues el archivo `.env`.** Este archivo debe quedarse únicamente en tu equipo local.
4. **Los modelos y precios pueden cambiar.** Antes de ejecutar la práctica, revisa en la documentación oficial que los modelos configurados sigan disponibles y que los precios sean correctos.
5. **La estimación de tokens es aproximada.** En esta práctica usarás una regla simple para comparar escenarios. La facturación real puede variar según el proveedor.
6. **Las pruebas consumen cuota o saldo.** Aunque los prompts son pequeños, cada ejecución puede generar costo si tu cuenta no tiene capa gratuita disponible.
7. **Ejecuta primero una prueba individual.** Antes de correr la comparación completa, valida un proveedor a la vez para identificar errores de configuración.
8. **Usa los mismos prompts para comparar.** No modifiques los prompts entre proveedores si quieres una comparación justa.
9. **No envíes información sensible real.** Los textos de esta práctica son simulados. Evita usar datos personales, contraseñas, contratos reales o información confidencial.
10. **Interpreta los resultados con criterio.** Una sola ejecución no representa el rendimiento definitivo de un modelo. Latencia, calidad y costo pueden variar.
11. **Documenta tus supuestos.** Si cambias modelos, precios, tokens máximos, prompts o volúmenes mensuales, registra el ajuste en tu matriz.
12. **Valida la calidad manualmente.** El costo y la latencia no son suficientes; también debes revisar si la respuesta es útil, clara, correcta y adecuada para el caso de uso.
13. **Cuida el límite de salida.** Mantener `max_output_tokens` o `max_tokens` en valores moderados ayuda a controlar costos.
14. **Repite solo si es necesario.** Si haces varias ejecuciones para promediar latencia, recuerda que cada corrida puede generar consumo.
15. **Entrega evidencias sin secretos.** Puedes entregar scripts, resultados y matriz, pero nunca tus claves de API.

---

## 🔗 Fuentes oficiales que debes revisar antes de ejecutar

> [!NOTE]
> Los modelos, precios, límites y ventanas de contexto cambian con frecuencia. Antes de ejecutar los scripts, confirma que los identificadores de modelo y precios usados en `precios_modelos.py` coincidan con la documentación oficial.

| Proveedor | Qué revisar | Fuente sugerida |
|---|---|---|
| OpenAI | Modelo disponible, precio de entrada, precio de salida y uso de Responses API | OpenAI API Pricing y documentación de Responses API |
| Google Gemini | Modelo disponible, precio por 1M tokens, límites y modalidad de API | Gemini API Pricing y Gemini API Models |
| Anthropic Claude | Modelo disponible, precio por MTok, contexto y opciones de residencia | Claude API Pricing y Claude Models |

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear una carpeta de trabajo en Windows, abrirla en Visual Studio Code y preparar un entorno Python para ejecutar pruebas contra OpenAI, Gemini y Claude.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear una carpeta local donde guardarás todos los archivos del laboratorio.

**⚙️ Contenido del paso:**  
Abre **Git Bash** y ejecuta:

```bash
mkdir -p ~/labs-ia-gen/lab-01-modelos
cd ~/labs-ia-gen/lab-01-modelos
```

**✅ Validación del paso:**  
Ejecuta:

```bash
pwd
```

Debes estar dentro de una ruta similar a:

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-01-modelos
```

**📌 Resultado esperado:**  
Tienes una carpeta dedicada para la práctica.

---

### ✅ Paso 2. Abre la carpeta en Visual Studio Code

**📝 Descripción del paso:**  
Vas a abrir el proyecto en VS Code desde Git Bash.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
code .
```

Si el comando `code .` no funciona, abre Visual Studio Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-01-modelos
```

**✅ Validación del paso:**  
Confirma que VS Code muestre la carpeta `lab-01-modelos`.

**📌 Resultado esperado:**  
El proyecto está abierto en Visual Studio Code.

---

### ✅ Paso 3. Crea el entorno virtual de Python

**📝 Descripción del paso:**  
Vas a aislar las dependencias de este laboratorio para no afectar otros proyectos.

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
Tienes un entorno virtual activo para la práctica.

---

### ✅ Paso 4. Crea el archivo de dependencias

**📝 Descripción del paso:**  
Vas a definir las librerías necesarias para consumir las APIs.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
requirements.txt
```

Agrega el siguiente contenido:

```txt
openai
google-genai
anthropic
python-dotenv
pandas
tabulate
```

**🔧 Qué puedes ajustar:**  
Puedes agregar otras librerías si necesitas extender la práctica. Por ejemplo, podrías agregar `tiktoken` para conteo más especializado de tokens en modelos OpenAI. En esta práctica se usará una estimación simple para mantener el laboratorio portable entre proveedores.

**✅ Validación del paso:**  
Confirma que el archivo `requirements.txt` exista en la raíz del proyecto.

**📌 Resultado esperado:**  
Tienes declaradas las dependencias del laboratorio.

---

### ✅ Paso 5. Instala las dependencias

**📝 Descripción del paso:**  
Vas a instalar los SDKs necesarios para trabajar con OpenAI, Gemini y Claude.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
pip install -r requirements.txt
```

**✅ Validación del paso:**  
Ejecuta:

```bash
pip list
```

Verifica que aparezcan paquetes relacionados con:

```text
openai
google-genai
anthropic
python-dotenv
pandas
```

**📌 Resultado esperado:**  
El ambiente tiene instaladas las librerías necesarias.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20de%20un%20laboratorio%20de%20IA%20generativa.%20Prepar%C3%A9%20una%20carpeta%20en%20Windows%2C%20abr%C3%AD%20el%20proyecto%20en%20Visual%20Studio%20Code%2C%20cre%C3%A9%20un%20entorno%20virtual%20de%20Python%20desde%20Git%20Bash%20e%20instal%C3%A9%20dependencias%20para%20OpenAI%2C%20Gemini%20y%20Claude.)

---

# 🧩 Tarea 2. Configurar las claves de API de forma segura

## 🎯 Objetivo de la tarea

Crear un archivo `.env` para guardar claves de API sin escribirlas directamente dentro del código.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo `.env`

**📝 Descripción del paso:**  
Vas a crear un archivo local para almacenar las claves de acceso a los proveedores.

**⚙️ Contenido del paso:**  
En la raíz del proyecto, crea un archivo llamado:

```text
.env
```

Agrega esta estructura:

```env
OPENAI_API_KEY=pega_aqui_tu_api_key_de_openai
GEMINI_API_KEY=pega_aqui_tu_api_key_de_gemini
ANTHROPIC_API_KEY=pega_aqui_tu_api_key_de_anthropic
```

**🔧 Qué debes cambiar:**  
Reemplaza:

```text
pega_aqui_tu_api_key_de_openai
pega_aqui_tu_api_key_de_gemini
pega_aqui_tu_api_key_de_anthropic
```

por tus claves reales.

**✅ Validación del paso:**  
Confirma que el archivo `.env` existe y que cada variable tiene un valor.

**📌 Resultado esperado:**  
Tienes las claves configuradas localmente.

---

### ✅ Paso 2. Crea el archivo `.gitignore`

**📝 Descripción del paso:**  
Vas a evitar que las claves de API se suban por error a un repositorio.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
.gitignore
```

Agrega:

```gitignore
.env
.venv/
__pycache__/
*.pyc
resultados_modelos.csv
```

**✅ Validación del paso:**  
Confirma que `.env` está incluido dentro de `.gitignore`.

**📌 Resultado esperado:**  
El archivo de claves queda protegido contra carga accidental a Git.

---

### ✅ Paso 3. Crea un script para validar variables de entorno

**📝 Descripción del paso:**  
Vas a comprobar que Python puede leer las claves desde `.env`.

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

required_keys = [
    "OPENAI_API_KEY",
    "GEMINI_API_KEY",
    "ANTHROPIC_API_KEY",
]

missing_keys = []

for key in required_keys:
    value = os.getenv(key)
    if not value or value.startswith("pega_aqui"):
        missing_keys.append(key)

if missing_keys:
    print("Faltan variables de entorno o tienen valores de ejemplo:")
    for key in missing_keys:
        print(f"- {key}")
    raise SystemExit(1)

print("Variables de entorno cargadas correctamente.")
print("No se muestran las claves por seguridad.")
```

**🔧 Qué puedes ajustar:**  
No imprimas las claves reales en pantalla. Si necesitas validar parcialmente una clave, imprime solo los primeros o últimos 4 caracteres, pero evita hacerlo en ambientes compartidos.

**✅ Validación del paso:**  
Ejecuta:

```bash
python 00_validar_entorno.py
```

**📌 Resultado esperado:**  
Debes ver:

```text
Variables de entorno cargadas correctamente.
No se muestran las claves por seguridad.
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20de%20un%20laboratorio%20de%20IA%20generativa.%20Configur%C3%A9%20un%20archivo%20.env%20con%20claves%20de%20OpenAI%2C%20Gemini%20y%20Claude%2C%20agregu%C3%A9%20.gitignore%20para%20proteger%20las%20claves%20y%20valid%C3%A9%20que%20Python%20pueda%20leer%20las%20variables%20de%20entorno.)

---

# 🧩 Tarea 3. Definir los casos de uso y prompts de prueba

## 🎯 Objetivo de la tarea

Crear tres prompts comparables para probar los modelos bajo los mismos escenarios: resumen, soporte técnico y análisis documental.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo de casos de prueba

**📝 Descripción del paso:**  
Vas a centralizar los prompts para reutilizarlos con los tres proveedores.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
casos_prueba.py
```

Agrega el siguiente código:

```python
CASOS_USO = [
    {
        "id": "CU-01",
        "nombre": "Resumen de contenido",
        "solicitudes_mensuales": 1000,
        "prompt": """
Eres un asistente ejecutivo. Resume el siguiente contenido en:
1. Resumen breve.
2. Ideas principales.
3. Riesgos o puntos de atención.
4. Próximos pasos sugeridos.

Contenido:
La organización está evaluando incorporar IA generativa en procesos internos.
Actualmente existen áreas interesadas en automatizar la creación de reportes,
responder preguntas frecuentes de soporte y analizar documentos largos de
cumplimiento. Sin embargo, la dirección solicita una evaluación previa de costo,
privacidad, latencia y dependencia del proveedor antes de aprobar cualquier
implementación productiva. El equipo técnico debe comparar proveedores, estimar
tokens, identificar riesgos y recomendar una arquitectura inicial.
"""
    },
    {
        "id": "CU-02",
        "nombre": "Soporte técnico asistido",
        "solicitudes_mensuales": 5000,
        "prompt": """
Eres un asistente de soporte técnico. Responde de forma clara, breve y accionable.

Pregunta del usuario:
No puedo conectarme a la VPN corporativa desde Windows. El error indica que las
credenciales no son válidas, pero ya confirmé que mi contraseña funciona en el
correo corporativo.

Base de conocimiento:
- Verificar conexión a internet.
- Confirmar que la cuenta no esté bloqueada.
- Validar que la hora del equipo esté sincronizada.
- Cerrar sesión y volver a iniciar.
- Reiniciar el cliente VPN.
- Si el problema continúa, levantar ticket con captura del error.
"""
    },
    {
        "id": "CU-03",
        "nombre": "Análisis documental",
        "solicitudes_mensuales": 800,
        "prompt": """
Eres un analista de cumplimiento. Revisa el siguiente fragmento de política
interna y genera:
1. Hallazgos principales.
2. Riesgos de cumplimiento.
3. Preguntas que deberían hacerse al área responsable.
4. Recomendaciones de mejora.

Documento:
La empresa podrá utilizar herramientas de inteligencia artificial generativa
para apoyar actividades operativas, administrativas y analíticas. Los usuarios
deberán evitar compartir información altamente confidencial, datos personales
sensibles, secretos comerciales o credenciales. Cada área será responsable de
validar las respuestas generadas antes de utilizarlas en documentos oficiales.
La organización podrá contratar servicios externos de IA cuando exista una
evaluación previa de seguridad, privacidad, costo y cumplimiento. Actualmente no
existe un procedimiento formal para registrar prompts, auditar respuestas,
monitorear consumo o clasificar información antes de enviarla a modelos externos.
"""
    }
]
```

**🔧 Qué puedes ajustar:**  
Puedes cambiar el contenido de cada prompt para alinearlo con tu industria. Mantén los tres tipos de caso de uso para poder comparar resultados.

**✅ Validación del paso:**  
Confirma que el archivo no tenga errores de sintaxis.

Ejecuta:

```bash
python -m py_compile casos_prueba.py
```

**📌 Resultado esperado:**  
El comando no debe mostrar errores.

---

### ✅ Paso 2. Revisa los volúmenes mensuales

**📝 Descripción del paso:**  
Vas a confirmar el volumen de uso que se utilizará en el cálculo de costos.

**⚙️ Contenido del paso:**  
Los valores iniciales son:

| Caso de uso | Solicitudes mensuales |
|---|---:|
| Resumen de contenido | 1,000 |
| Soporte técnico asistido | 5,000 |
| Análisis documental | 800 |

**🔧 Qué puedes ajustar:**  
En el archivo `casos_prueba.py`, puedes modificar:

```python
"solicitudes_mensuales": 1000
```

por el volumen real que quieras analizar.

**✅ Validación del paso:**  
Cada caso debe tener un valor numérico mayor que cero.

**📌 Resultado esperado:**  
Los casos de uso están listos para pruebas y estimaciones.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20de%20un%20laboratorio%20de%20IA%20generativa.%20Defin%C3%AD%20tres%20casos%20de%20uso%20con%20prompts%20comparables%3A%20resumen%2C%20soporte%20t%C3%A9cnico%20y%20an%C3%A1lisis%20documental.%20Tambi%C3%A9n%20defin%C3%AD%20solicitudes%20mensuales%20para%20calcular%20costos.)

---

# 🧩 Tarea 4. Crear funciones comunes para medir tokens, latencia y costo

## 🎯 Objetivo de la tarea

Crear funciones reutilizables para estimar tokens, medir tiempo de respuesta y calcular costos.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo de utilidades

**📝 Descripción del paso:**  
Vas a crear un archivo con funciones compartidas por todos los scripts.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
utils.py
```

Agrega el siguiente código:

```python
import time


def estimar_tokens(texto: str) -> int:
    """
    Estimación simple de tokens.
    Regla aproximada: 1 token equivale a 4 caracteres en inglés/español técnico.
    Esta estimación no reemplaza los contadores oficiales de cada proveedor.
    """
    if not texto:
        return 0

    return max(1, round(len(texto) / 4))


def iniciar_cronometro() -> float:
    return time.perf_counter()


def detener_cronometro(inicio: float) -> float:
    fin = time.perf_counter()
    return round(fin - inicio, 3)


def calcular_costo_usd(
    tokens_entrada: int,
    tokens_salida: int,
    precio_entrada_1m: float,
    precio_salida_1m: float
) -> float:
    costo_entrada = (tokens_entrada / 1_000_000) * precio_entrada_1m
    costo_salida = (tokens_salida / 1_000_000) * precio_salida_1m
    return round(costo_entrada + costo_salida, 6)


def imprimir_resultado(resultado: dict) -> None:
    print("\n" + "=" * 80)
    print(f"Proveedor: {resultado['proveedor']}")
    print(f"Modelo: {resultado['modelo']}")
    print(f"Caso de uso: {resultado['caso_uso']}")
    print(f"Latencia: {resultado['latencia_segundos']} segundos")
    print(f"Tokens entrada estimados: {resultado['tokens_entrada']}")
    print(f"Tokens salida estimados: {resultado['tokens_salida']}")
    print(f"Costo estimado por solicitud: ${resultado['costo_solicitud_usd']} USD")
    print(f"Costo mensual estimado: ${resultado['costo_mensual_usd']} USD")
    print("-" * 80)
    print("Respuesta:")
    print(resultado["respuesta"])
    print("=" * 80)
```

**🔧 Qué puedes ajustar:**  
La función `estimar_tokens()` usa una regla aproximada. Si necesitas mayor precisión, puedes sustituirla por contadores oficiales o librerías específicas por proveedor.

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile utils.py
```

**📌 Resultado esperado:**  
El archivo compila sin errores.

---

### ✅ Paso 2. Define los precios de referencia

**📝 Descripción del paso:**  
Vas a crear un archivo donde registrarás los precios usados para el cálculo.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
precios_modelos.py
```

Agrega el siguiente código:

```python
PRECIOS_MODELOS = {
    "openai": {
        "proveedor": "OpenAI",
        "modelo": "gpt-5.4-mini",
        "precio_entrada_1m": 0.75,
        "precio_salida_1m": 4.50,
        "ventana_contexto": "Consultar documentación oficial",
        "fuente": "OpenAI API Pricing"
    },
    "gemini": {
        "proveedor": "Google Gemini",
        "modelo": "gemini-2.5-flash",
        "precio_entrada_1m": 0.30,
        "precio_salida_1m": 2.50,
        "ventana_contexto": "Consultar documentación oficial",
        "fuente": "Google AI Gemini API Pricing"
    },
    "claude": {
        "proveedor": "Anthropic Claude",
        "modelo": "claude-sonnet-4-6",
        "precio_entrada_1m": 3.00,
        "precio_salida_1m": 15.00,
        "ventana_contexto": "Consultar documentación oficial",
        "fuente": "Anthropic Pricing"
    }
}
```

**🔧 Qué debes cambiar:**  
Antes de ejecutar el laboratorio, revisa los precios actuales en la documentación oficial de cada proveedor y ajusta estos campos:

```python
"modelo": "gpt-5.4-mini"
"precio_entrada_1m": 0.75
"precio_salida_1m": 4.50
```

Haz lo mismo para Gemini y Claude.

**✅ Validación del paso:**  
Ejecuta:

```bash
python -m py_compile precios_modelos.py
```

**📌 Resultado esperado:**  
El archivo de precios compila correctamente.

> [!WARNING]
> Los valores incluidos en `precios_modelos.py` son valores de referencia para el laboratorio. Antes de impartir o entregar la práctica, actualiza modelo y precios con la documentación oficial del día.

**Nota:** 
- [Página de precios OpenAI](https://openai.com/api/pricing/)
- [Página de precios Gemini](https://ai.google.dev/gemini-api/docs/pricing)
- [Página de precios Claude](https://platform.claude.com/docs/es/about-claude/pricing)
---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20de%20un%20laboratorio%20de%20IA%20generativa.%20Cre%C3%A9%20funciones%20para%20estimar%20tokens%2C%20medir%20latencia%2C%20calcular%20costo%20por%20solicitud%20y%20costo%20mensual.%20Tambi%C3%A9n%20cre%C3%A9%20un%20archivo%20de%20precios%20para%20OpenAI%2C%20Gemini%20y%20Claude.)

---

# 🧩 Tarea 5. Probar OpenAI desde Python

## 🎯 Objetivo de la tarea

Ejecutar una prueba real contra OpenAI, medir latencia, estimar tokens y calcular costo aproximado.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el script de prueba para OpenAI

**📝 Descripción del paso:**  
Vas a crear un script que envía los tres casos de uso a OpenAI.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
01_probar_openai.py
```

Agrega el siguiente código:

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

from casos_prueba import CASOS_USO
from precios_modelos import PRECIOS_MODELOS
from utils import (
    estimar_tokens,
    iniciar_cronometro,
    detener_cronometro,
    calcular_costo_usd,
    imprimir_resultado,
)

load_dotenv()

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

CONFIG = PRECIOS_MODELOS["openai"]

# Puedes cambiar el modelo aquí.
# Revisa que el modelo exista y esté habilitado en tu cuenta.
MODELO = CONFIG["modelo"]


def probar_openai(caso: dict) -> dict:
    prompt = caso["prompt"]

    tokens_entrada = estimar_tokens(prompt)

    inicio = iniciar_cronometro()

    response = client.responses.create(
        model=MODELO,
        input=prompt,
        max_output_tokens=500,
        temperature=0.2,
    )

    latencia = detener_cronometro(inicio)

    respuesta = response.output_text

    tokens_salida = estimar_tokens(respuesta)

    costo_solicitud = calcular_costo_usd(
        tokens_entrada=tokens_entrada,
        tokens_salida=tokens_salida,
        precio_entrada_1m=CONFIG["precio_entrada_1m"],
        precio_salida_1m=CONFIG["precio_salida_1m"],
    )

    costo_mensual = round(costo_solicitud * caso["solicitudes_mensuales"], 4)

    return {
        "proveedor": CONFIG["proveedor"],
        "modelo": MODELO,
        "caso_uso": caso["nombre"],
        "latencia_segundos": latencia,
        "tokens_entrada": tokens_entrada,
        "tokens_salida": tokens_salida,
        "costo_solicitud_usd": costo_solicitud,
        "costo_mensual_usd": costo_mensual,
        "respuesta": respuesta,
    }


if __name__ == "__main__":
    for caso in CASOS_USO:
        resultado = probar_openai(caso)
        imprimir_resultado(resultado)
```

**🔧 Qué puedes ajustar:**  
Puedes cambiar:

```python
MODELO = CONFIG["modelo"]
```

si deseas probar otro modelo de OpenAI.

También puedes ajustar:

```python
max_output_tokens=500
temperature=0.2
```

- `max_output_tokens` controla la longitud máxima de respuesta.
- `temperature` controla variabilidad; valores bajos generan respuestas más consistentes.

**✅ Validación del paso:**  
Ejecuta:

```bash
python 01_probar_openai.py
```

**📌 Resultado esperado:**  
Debes ver tres respuestas, una por cada caso de uso, con latencia, tokens estimados y costo aproximado.

---

### ✅ Paso 2. Identifica errores comunes en OpenAI

**📝 Descripción del paso:**  
Vas a reconocer problemas frecuentes al ejecutar la prueba.

**⚙️ Contenido del paso:**

| Error posible | Causa probable | Corrección |
|---|---|---|
| API key inválida | La clave está mal copiada | Revisa `OPENAI_API_KEY` en `.env` |
| Modelo no encontrado | El modelo no existe o no está habilitado | Cambia el campo `modelo` en `precios_modelos.py` |
| Error de cuota | No tienes saldo o límite disponible | Revisa facturación/límites de la cuenta |
| Módulo no encontrado | Faltan dependencias | Ejecuta `pip install -r requirements.txt` |

**✅ Validación del paso:**  
Si ocurre un error, corrígelo y vuelve a ejecutar el script.

**📌 Resultado esperado:**  
OpenAI responde correctamente para los tres casos de uso.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20de%20un%20laboratorio%20de%20IA%20generativa.%20Ejecut%C3%A9%20una%20prueba%20real%20con%20OpenAI%20desde%20Python%2C%20usando%20la%20Responses%20API%2C%20med%C3%AD%20latencia%2C%20estim%C3%A9%20tokens%20de%20entrada%20y%20salida%2C%20y%20calcul%C3%A9%20costo%20aproximado%20por%20solicitud%20y%20por%20mes.)

---

# 🧩 Tarea 6. Probar Gemini desde Python

## 🎯 Objetivo de la tarea

Ejecutar una prueba real contra Google Gemini, medir latencia, estimar tokens y calcular costo aproximado.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el script de prueba para Gemini

**📝 Descripción del paso:**  
Vas a crear un script que envía los mismos tres casos de uso a Gemini.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
02_probar_gemini.py
```

Agrega el siguiente código:

```python
import os
from dotenv import load_dotenv
from google import genai
from google.genai import types

from casos_prueba import CASOS_USO
from precios_modelos import PRECIOS_MODELOS
from utils import (
    estimar_tokens,
    iniciar_cronometro,
    detener_cronometro,
    calcular_costo_usd,
    imprimir_resultado,
)

load_dotenv()

client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))

CONFIG = PRECIOS_MODELOS["gemini"]

# Puedes cambiar el modelo aquí.
# Revisa que el modelo exista y esté habilitado para tu API key.
MODELO = CONFIG["modelo"]


def probar_gemini(caso: dict) -> dict:
    prompt = caso["prompt"]

    tokens_entrada = estimar_tokens(prompt)

    inicio = iniciar_cronometro()

    response = client.models.generate_content(
        model=MODELO,
        contents=prompt,
        config=types.GenerateContentConfig(
            temperature=0.2,
            max_output_tokens=500,
        ),
    )

    latencia = detener_cronometro(inicio)

    respuesta = response.text or ""

    tokens_salida = estimar_tokens(respuesta)

    costo_solicitud = calcular_costo_usd(
        tokens_entrada=tokens_entrada,
        tokens_salida=tokens_salida,
        precio_entrada_1m=CONFIG["precio_entrada_1m"],
        precio_salida_1m=CONFIG["precio_salida_1m"],
    )

    costo_mensual = round(costo_solicitud * caso["solicitudes_mensuales"], 4)

    return {
        "proveedor": CONFIG["proveedor"],
        "modelo": MODELO,
        "caso_uso": caso["nombre"],
        "latencia_segundos": latencia,
        "tokens_entrada": tokens_entrada,
        "tokens_salida": tokens_salida,
        "costo_solicitud_usd": costo_solicitud,
        "costo_mensual_usd": costo_mensual,
        "respuesta": respuesta,
    }


if __name__ == "__main__":
    for caso in CASOS_USO:
        resultado = probar_gemini(caso)
        imprimir_resultado(resultado)
```

**🔧 Qué puedes ajustar:**  
Puedes cambiar:

```python
MODELO = CONFIG["modelo"]
```

para probar otro modelo Gemini.

También puedes ajustar:

```python
temperature=0.2
max_output_tokens=500
```

Si el modelo devuelve respuestas muy largas, reduce `max_output_tokens`. Si necesitas respuestas más creativas, aumenta ligeramente `temperature`.

**✅ Validación del paso:**  
Ejecuta:

```bash
python 02_probar_gemini.py
```

**📌 Resultado esperado:**  
Debes ver tres respuestas generadas por Gemini con latencia, tokens estimados y costo aproximado.

---

### ✅ Paso 2. Identifica errores comunes en Gemini

**📝 Descripción del paso:**  
Vas a reconocer problemas frecuentes al ejecutar la prueba.

**⚙️ Contenido del paso:**

| Error posible | Causa probable | Corrección |
|---|---|---|
| API key inválida | La clave de Gemini está mal configurada | Revisa `GEMINI_API_KEY` en `.env` |
| Modelo no disponible | El nombre del modelo cambió o no está habilitado | Ajusta `modelo` en `precios_modelos.py` |
| Error de permisos | Tu proyecto/API key no tiene acceso | Revisa la configuración en Google AI Studio o Google Cloud |
| Respuesta vacía | El contenido fue bloqueado o no generado | Prueba con un prompt más simple |

**✅ Validación del paso:**  
Si existe error, corrige la clave, el modelo o el prompt y vuelve a ejecutar.

**📌 Resultado esperado:**  
Gemini responde correctamente para los tres casos de uso.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20de%20un%20laboratorio%20de%20IA%20generativa.%20Ejecut%C3%A9%20una%20prueba%20real%20con%20Google%20Gemini%20desde%20Python%2C%20usando%20generate_content%2C%20med%C3%AD%20latencia%2C%20estim%C3%A9%20tokens%20y%20calcul%C3%A9%20costo%20aproximado%20por%20solicitud%20y%20por%20mes.)

---

# 🧩 Tarea 7. Probar Claude desde Python

## 🎯 Objetivo de la tarea

Ejecutar una prueba real contra Anthropic Claude, medir latencia, estimar tokens y calcular costo aproximado.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el script de prueba para Claude

**📝 Descripción del paso:**  
Vas a crear un script que envía los mismos tres casos de uso a Claude.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
03_probar_claude.py
```

Agrega el siguiente código:

```python
import os
from dotenv import load_dotenv
from anthropic import Anthropic

from casos_prueba import CASOS_USO
from precios_modelos import PRECIOS_MODELOS
from utils import (
    estimar_tokens,
    iniciar_cronometro,
    detener_cronometro,
    calcular_costo_usd,
    imprimir_resultado,
)

load_dotenv()

client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

CONFIG = PRECIOS_MODELOS["claude"]

# Puedes cambiar el modelo aquí.
# Revisa que el modelo exista y esté habilitado en tu cuenta.
MODELO = CONFIG["modelo"]


def probar_claude(caso: dict) -> dict:
    prompt = caso["prompt"]

    tokens_entrada = estimar_tokens(prompt)

    inicio = iniciar_cronometro()

    message = client.messages.create(
        model=MODELO,
        max_tokens=500,
        temperature=0.2,
        messages=[
            {
                "role": "user",
                "content": prompt
            }
        ],
    )

    latencia = detener_cronometro(inicio)

    respuesta = ""
    if message.content:
        respuesta = message.content[0].text

    tokens_salida = estimar_tokens(respuesta)

    costo_solicitud = calcular_costo_usd(
        tokens_entrada=tokens_entrada,
        tokens_salida=tokens_salida,
        precio_entrada_1m=CONFIG["precio_entrada_1m"],
        precio_salida_1m=CONFIG["precio_salida_1m"],
    )

    costo_mensual = round(costo_solicitud * caso["solicitudes_mensuales"], 4)

    return {
        "proveedor": CONFIG["proveedor"],
        "modelo": MODELO,
        "caso_uso": caso["nombre"],
        "latencia_segundos": latencia,
        "tokens_entrada": tokens_entrada,
        "tokens_salida": tokens_salida,
        "costo_solicitud_usd": costo_solicitud,
        "costo_mensual_usd": costo_mensual,
        "respuesta": respuesta,
    }


if __name__ == "__main__":
    for caso in CASOS_USO:
        resultado = probar_claude(caso)
        imprimir_resultado(resultado)
```

**🔧 Qué puedes ajustar:**  
Puedes cambiar:

```python
MODELO = CONFIG["modelo"]
```

para probar otro modelo Claude.

También puedes ajustar:

```python
max_tokens=500
temperature=0.2
```

- `max_tokens` controla el máximo de tokens generados.
- `temperature` controla variación en la respuesta.

**✅ Validación del paso:**  
Ejecuta:

```bash
python 03_probar_claude.py
```

**📌 Resultado esperado:**  
Debes ver tres respuestas generadas por Claude con latencia, tokens estimados y costo aproximado.

---

### ✅ Paso 2. Identifica errores comunes en Claude

**📝 Descripción del paso:**  
Vas a reconocer problemas frecuentes al ejecutar la prueba.

**⚙️ Contenido del paso:**

| Error posible | Causa probable | Corrección |
|---|---|---|
| API key inválida | La clave de Anthropic está mal configurada | Revisa `ANTHROPIC_API_KEY` en `.env` |
| Modelo no disponible | El nombre del modelo no existe o no está habilitado | Ajusta `modelo` en `precios_modelos.py` |
| Error de cuota | No tienes crédito o acceso suficiente | Revisa la consola de Anthropic |
| Respuesta no esperada | Prompt ambiguo o muy largo | Ajusta el prompt o reduce contenido |

**✅ Validación del paso:**  
Corrige cualquier error y vuelve a ejecutar el script.

**📌 Resultado esperado:**  
Claude responde correctamente para los tres casos de uso.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%207%20de%20un%20laboratorio%20de%20IA%20generativa.%20Ejecut%C3%A9%20una%20prueba%20real%20con%20Claude%20desde%20Python%2C%20usando%20el%20SDK%20oficial%20de%20Anthropic%2C%20med%C3%AD%20latencia%2C%20estim%C3%A9%20tokens%20y%20calcul%C3%A9%20costo%20aproximado%20por%20solicitud%20y%20por%20mes.)

---

# 🧩 Tarea 8. Ejecutar una prueba comparativa automatizada

## 🎯 Objetivo de la tarea

Crear un script único que ejecute OpenAI, Gemini y Claude sobre los mismos casos de uso y genere un archivo CSV con los resultados.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el script comparativo

**📝 Descripción del paso:**  
Vas a automatizar la comparación para no copiar resultados manualmente.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
04_comparar_modelos.py
```

Agrega el siguiente código:

```python
import os
import pandas as pd
from dotenv import load_dotenv

from openai import OpenAI
from google import genai
from google.genai import types
from anthropic import Anthropic

from casos_prueba import CASOS_USO
from precios_modelos import PRECIOS_MODELOS
from utils import (
    estimar_tokens,
    iniciar_cronometro,
    detener_cronometro,
    calcular_costo_usd,
)

load_dotenv()

openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
gemini_client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
claude_client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))


def ejecutar_openai(caso: dict) -> dict:
    config = PRECIOS_MODELOS["openai"]
    modelo = config["modelo"]
    prompt = caso["prompt"]
    tokens_entrada = estimar_tokens(prompt)

    inicio = iniciar_cronometro()
    response = openai_client.responses.create(
        model=modelo,
        input=prompt,
        max_output_tokens=500,
        temperature=0.2,
    )
    latencia = detener_cronometro(inicio)

    respuesta = response.output_text
    tokens_salida = estimar_tokens(respuesta)

    costo_solicitud = calcular_costo_usd(
        tokens_entrada,
        tokens_salida,
        config["precio_entrada_1m"],
        config["precio_salida_1m"],
    )

    return construir_resultado(
        caso, config, modelo, latencia,
        tokens_entrada, tokens_salida,
        costo_solicitud, respuesta
    )


def ejecutar_gemini(caso: dict) -> dict:
    config = PRECIOS_MODELOS["gemini"]
    modelo = config["modelo"]
    prompt = caso["prompt"]
    tokens_entrada = estimar_tokens(prompt)

    inicio = iniciar_cronometro()
    response = gemini_client.models.generate_content(
        model=modelo,
        contents=prompt,
        config=types.GenerateContentConfig(
            temperature=0.2,
            max_output_tokens=500,
        ),
    )
    latencia = detener_cronometro(inicio)

    respuesta = response.text or ""
    tokens_salida = estimar_tokens(respuesta)

    costo_solicitud = calcular_costo_usd(
        tokens_entrada,
        tokens_salida,
        config["precio_entrada_1m"],
        config["precio_salida_1m"],
    )

    return construir_resultado(
        caso, config, modelo, latencia,
        tokens_entrada, tokens_salida,
        costo_solicitud, respuesta
    )


def ejecutar_claude(caso: dict) -> dict:
    config = PRECIOS_MODELOS["claude"]
    modelo = config["modelo"]
    prompt = caso["prompt"]
    tokens_entrada = estimar_tokens(prompt)

    inicio = iniciar_cronometro()
    message = claude_client.messages.create(
        model=modelo,
        max_tokens=500,
        temperature=0.2,
        messages=[
            {
                "role": "user",
                "content": prompt
            }
        ],
    )
    latencia = detener_cronometro(inicio)

    respuesta = ""
    if message.content:
        respuesta = message.content[0].text

    tokens_salida = estimar_tokens(respuesta)

    costo_solicitud = calcular_costo_usd(
        tokens_entrada,
        tokens_salida,
        config["precio_entrada_1m"],
        config["precio_salida_1m"],
    )

    return construir_resultado(
        caso, config, modelo, latencia,
        tokens_entrada, tokens_salida,
        costo_solicitud, respuesta
    )


def construir_resultado(
    caso: dict,
    config: dict,
    modelo: str,
    latencia: float,
    tokens_entrada: int,
    tokens_salida: int,
    costo_solicitud: float,
    respuesta: str,
) -> dict:
    return {
        "caso_id": caso["id"],
        "caso_uso": caso["nombre"],
        "proveedor": config["proveedor"],
        "modelo": modelo,
        "solicitudes_mensuales": caso["solicitudes_mensuales"],
        "tokens_entrada_estimados": tokens_entrada,
        "tokens_salida_estimados": tokens_salida,
        "latencia_segundos": latencia,
        "precio_entrada_1m": config["precio_entrada_1m"],
        "precio_salida_1m": config["precio_salida_1m"],
        "costo_solicitud_usd": costo_solicitud,
        "costo_mensual_usd": round(costo_solicitud * caso["solicitudes_mensuales"], 4),
        "respuesta_muestra": respuesta.replace("\n", " ")[:500],
    }


def main():
    resultados = []

    for caso in CASOS_USO:
        print(f"Ejecutando caso: {caso['nombre']}")

        resultados.append(ejecutar_openai(caso))
        resultados.append(ejecutar_gemini(caso))
        resultados.append(ejecutar_claude(caso))

    df = pd.DataFrame(resultados)
    df.to_csv("resultados_modelos.csv", index=False, encoding="utf-8-sig")

    print("\nComparación final:")
    print(df[[
        "caso_uso",
        "proveedor",
        "modelo",
        "latencia_segundos",
        "tokens_entrada_estimados",
        "tokens_salida_estimados",
        "costo_mensual_usd",
    ]].to_markdown(index=False))

    print("\nArchivo generado: resultados_modelos.csv")


if __name__ == "__main__":
    main()
```

**🔧 Qué puedes ajustar:**  
Puedes modificar:

```python
max_output_tokens=500
temperature=0.2
```

en cada proveedor para hacer pruebas más cortas, más largas o más creativas.

También puedes ajustar:

```python
"respuesta_muestra": respuesta.replace("\n", " ")[:500]
```

si quieres guardar más o menos texto de muestra en el CSV.

**✅ Validación del paso:**  
Ejecuta:

```bash
python 04_comparar_modelos.py
```

**📌 Resultado esperado:**  
Debes ver una tabla comparativa en terminal y un archivo nuevo llamado:

```text
resultados_modelos.csv
```

---

### ✅ Paso 2. Abre el archivo CSV

**📝 Descripción del paso:**  
Vas a revisar los resultados generados por el script.

**⚙️ Contenido del paso:**  
Desde VS Code, abre:

```text
resultados_modelos.csv
```

También puedes abrirlo con Excel.

**✅ Validación del paso:**  
El archivo debe contener al menos 9 filas:

```text
3 casos de uso x 3 proveedores = 9 resultados
```

**📌 Resultado esperado:**  
Tienes resultados comparables para OpenAI, Gemini y Claude.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%208%20de%20un%20laboratorio%20de%20IA%20generativa.%20Cre%C3%A9%20un%20script%20comparativo%20que%20ejecuta%20OpenAI%2C%20Gemini%20y%20Claude%20sobre%20los%20mismos%20casos%20de%20uso%2C%20mide%20latencia%2C%20estima%20tokens%2C%20calcula%20costos%20y%20genera%20un%20archivo%20CSV%20con%20los%20resultados.)

---

# 🧩 Tarea 9. Construir la matriz técnica de decisión

## 🎯 Objetivo de la tarea

Convertir los resultados técnicos de las pruebas en una matriz profesional para comparar costo, latencia, contexto, privacidad y operación.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo de matriz

**📝 Descripción del paso:**  
Vas a crear una hoja de cálculo para documentar la comparación.

**⚙️ Contenido del paso:**  
Crea un archivo llamado:

```text
Laboratorio_1_Matriz_Seleccion_Modelos.xlsx
```

Crea las siguientes hojas:

```text
01_Resultados
02_Contexto_Privacidad
03_Riesgos
04_Recomendacion
```

**✅ Validación del paso:**  
Confirma que el archivo tenga las cuatro hojas.

**📌 Resultado esperado:**  
Tienes un archivo profesional para documentar la selección técnica.

---

### ✅ Paso 2. Importa los resultados del CSV

**📝 Descripción del paso:**  
Vas a cargar los resultados reales generados por Python.

**⚙️ Contenido del paso:**  
Abre `resultados_modelos.csv` en Excel o Google Sheets y copia la información a la hoja:

```text
01_Resultados
```

Asegúrate de incluir estas columnas:

| Columna |
|---|
| caso_id |
| caso_uso |
| proveedor |
| modelo |
| solicitudes_mensuales |
| tokens_entrada_estimados |
| tokens_salida_estimados |
| latencia_segundos |
| precio_entrada_1m |
| precio_salida_1m |
| costo_solicitud_usd |
| costo_mensual_usd |
| respuesta_muestra |

**✅ Validación del paso:**  
Confirma que existan 9 filas de resultados.

**📌 Resultado esperado:**  
La matriz contiene resultados medidos desde código.

---

### ✅ Paso 3. Evalúa ventana de contexto y privacidad

**📝 Descripción del paso:**  
Vas a complementar los resultados técnicos con criterios arquitectónicos.

**⚙️ Contenido del paso:**  
En la hoja `02_Contexto_Privacidad`, crea esta tabla:

| Caso de uso | Proveedor | Modelo | Ventana de contexto | ¿Soporta el caso? | Sensibilidad de datos | Riesgo de privacidad | Observaciones |
|---|---|---|---|---|---|---|---|

Usa esta guía para `¿Soporta el caso?`:

| Valor | Criterio |
|---|---|
| Sí | El contexto del modelo es suficiente |
| Parcial | Requiere dividir el documento o usar RAG |
| No | No es adecuado para el tamaño esperado |

Usa esta guía para `Riesgo de privacidad`:

| Valor | Criterio |
|---|---|
| Bajo | Datos no sensibles o controles empresariales adecuados |
| Medio | Datos internos con dependencia de proveedor externo |
| Alto | Datos sensibles, legales, personales o confidenciales |

**🔧 Qué debes cambiar:**  
Consulta la documentación actual del proveedor y actualiza la ventana de contexto real de cada modelo usado.

**✅ Validación del paso:**  
Cada combinación de modelo y caso de uso debe tener evaluación de contexto y privacidad.

**📌 Resultado esperado:**  
La matriz no se limita a costo y latencia; también considera riesgo.

---

### ✅ Paso 4. Evalúa riesgos operativos

**📝 Descripción del paso:**  
Vas a documentar los riesgos de operación por proveedor.

**⚙️ Contenido del paso:**  
En la hoja `03_Riesgos`, crea esta tabla:

| Proveedor | Modelo | Riesgo de costo | Riesgo de latencia | Riesgo de cuota/límites | Riesgo de dependencia | Riesgo de integración | Mitigación |
|---|---|---|---|---|---|---|---|

Ejemplos de mitigación:

```text
Definir presupuesto mensual y alertas de consumo.
Aplicar límites de max_output_tokens.
Usar caché para preguntas frecuentes.
Implementar RAG para reducir contexto innecesario.
Definir proveedor alternativo.
No enviar datos sensibles sin clasificación previa.
Registrar prompts y respuestas para auditoría.
```

**✅ Validación del paso:**  
Cada proveedor debe tener al menos una mitigación clara.

**📌 Resultado esperado:**  
Tienes una evaluación realista de operación.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 9 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%209%20de%20un%20laboratorio%20de%20IA%20generativa.%20Constru%C3%AD%20una%20matriz%20t%C3%A9cnica%20de%20decisi%C3%B3n%20con%20resultados%20reales%20del%20CSV%2C%20evalu%C3%A9%20ventana%20de%20contexto%2C%20privacidad%2C%20riesgos%20operativos%20y%20mitigaciones.)

---

# 🧩 Tarea 10. Recomendar un modelo por caso de uso

## 🎯 Objetivo de la tarea

Seleccionar el proveedor/modelo más adecuado para cada caso de uso usando los resultados de la prueba y la matriz técnica.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la tabla de recomendación

**📝 Descripción del paso:**  
Vas a documentar la decisión final.

**⚙️ Contenido del paso:**  
En la hoja `04_Recomendacion`, crea esta tabla:

| Caso de uso | Modelo recomendado | Proveedor | Razón principal | Costo mensual estimado | Latencia medida | Riesgo principal | Mitigación | Alternativa |
|---|---|---|---|---:|---:|---|---|---|

**✅ Validación del paso:**  
La tabla debe tener una fila por cada caso de uso.

**📌 Resultado esperado:**  
Tienes una estructura lista para justificar la decisión.

---

### ✅ Paso 2. Recomienda modelo para resumen de contenido

**📝 Descripción del paso:**  
Vas a seleccionar el modelo más adecuado para generar resúmenes.

**⚙️ Contenido del paso:**  
Considera:

1. Costo mensual.
2. Claridad del resumen.
3. Latencia.
4. Longitud del texto de entrada.
5. Calidad de estructura en la respuesta.

Ejemplo de justificación:

```text
Para resumen de contenido se recomienda [modelo] de [proveedor] porque ofrece una buena relación entre costo, latencia y calidad de síntesis. La prueba mostró una latencia de [x] segundos y un costo mensual estimado de [y] USD. El principal riesgo es [riesgo], mitigado mediante [mitigación].
```

**✅ Validación del paso:**  
La recomendación debe estar alineada con los datos de `01_Resultados`.

**📌 Resultado esperado:**  
Tienes una recomendación defendible para resumen.

---

### ✅ Paso 3. Recomienda modelo para soporte técnico

**📝 Descripción del paso:**  
Vas a seleccionar el modelo más adecuado para alto volumen y respuesta rápida.

**⚙️ Contenido del paso:**  
Considera:

1. Menor latencia.
2. Costo mensual bajo o moderado.
3. Respuestas claras y accionables.
4. Capacidad de integrarse con una base de conocimiento.
5. Consistencia en instrucciones.

Ejemplo de justificación:

```text
Para soporte técnico asistido se recomienda [modelo] de [proveedor] porque el escenario requiere alto volumen, baja latencia y respuestas breves. La prueba mostró [x] segundos de latencia y un costo mensual estimado de [y] USD. El principal riesgo es [riesgo], por lo que se recomienda [mitigación].
```

**✅ Validación del paso:**  
No selecciones un modelo con latencia alta si existe otra opción con calidad suficiente y menor tiempo de respuesta.

**📌 Resultado esperado:**  
Tienes una recomendación orientada a operación.

---

### ✅ Paso 4. Recomienda modelo para análisis documental

**📝 Descripción del paso:**  
Vas a seleccionar el modelo más adecuado para documentos largos y mayor sensibilidad.

**⚙️ Contenido del paso:**  
Considera:

1. Ventana de contexto.
2. Calidad del razonamiento.
3. Capacidad para seguir instrucciones complejas.
4. Privacidad.
5. Riesgo de costo por entradas largas.
6. Necesidad de auditoría o cumplimiento.

Ejemplo de justificación:

```text
Para análisis documental se recomienda [modelo] de [proveedor] porque el caso requiere mayor capacidad de razonamiento y soporte para entradas largas. El costo mensual estimado fue de [y] USD y la latencia medida fue de [x] segundos. El principal riesgo es [riesgo], mitigado mediante clasificación previa de documentos, reducción de contexto y controles de privacidad.
```

**✅ Validación del paso:**  
La recomendación debe considerar contexto y privacidad, no solo costo.

**📌 Resultado esperado:**  
Tienes una recomendación para el caso más exigente.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 10 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2010%20de%20un%20laboratorio%20de%20IA%20generativa.%20Recomend%C3%A9%20un%20modelo%20por%20caso%20de%20uso%20usando%20resultados%20reales%20de%20latencia%2C%20tokens%2C%20costo%2C%20calidad%20observada%2C%20ventana%20de%20contexto%2C%20privacidad%20y%20riesgos%20operativos.)

---

# 🧩 Tarea 11. Validar funcionamiento y entregar evidencias

## 🎯 Objetivo de la tarea

Confirmar que los scripts funcionan, que la matriz contiene datos consistentes y que la recomendación final está soportada por evidencia.

---

## 🛠️ Pasos

### ✅ Paso 1. Valida que todos los scripts compilen

**📝 Descripción del paso:**  
Vas a verificar que los archivos Python no tengan errores de sintaxis.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python -m py_compile 00_validar_entorno.py
python -m py_compile casos_prueba.py
python -m py_compile utils.py
python -m py_compile precios_modelos.py
python -m py_compile 01_probar_openai.py
python -m py_compile 02_probar_gemini.py
python -m py_compile 03_probar_claude.py
python -m py_compile 04_comparar_modelos.py
```

**✅ Validación del paso:**  
Ningún comando debe mostrar errores.

**📌 Resultado esperado:**  
Todos los scripts tienen sintaxis válida.

---

### ✅ Paso 2. Valida las variables de entorno

**📝 Descripción del paso:**  
Vas a comprobar que las claves siguen cargando correctamente.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python 00_validar_entorno.py
```

**✅ Validación del paso:**  
Debe mostrarse:

```text
Variables de entorno cargadas correctamente.
No se muestran las claves por seguridad.
```

**📌 Resultado esperado:**  
Las claves de API están disponibles para los scripts.

---

### ✅ Paso 3. Valida la prueba individual por proveedor

**📝 Descripción del paso:**  
Vas a confirmar que cada proveedor responde por separado.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python 01_probar_openai.py
python 02_probar_gemini.py
python 03_probar_claude.py
```

**✅ Validación del paso:**  
Cada script debe mostrar tres respuestas.

**📌 Resultado esperado:**  
OpenAI, Gemini y Claude funcionan de manera independiente.

---

### ✅ Paso 4. Valida la comparación completa

**📝 Descripción del paso:**  
Vas a confirmar que la comparación integrada funciona.

**⚙️ Contenido del paso:**  
Ejecuta:

```bash
python 04_comparar_modelos.py
```

**✅ Validación del paso:**  
Confirma que se genere el archivo:

```text
resultados_modelos.csv
```

**📌 Resultado esperado:**  
Tienes un archivo CSV con los resultados de la comparación.

---

### ✅ Paso 5. Valida la matriz final

**📝 Descripción del paso:**  
Vas a revisar que la matriz sea consistente.

**⚙️ Contenido del paso:**  
En `Laboratorio_1_Matriz_Seleccion_Modelos.xlsx`, valida:

1. Existen resultados para los tres proveedores.
2. Existen resultados para los tres casos de uso.
3. El costo mensual se calcula con solicitudes mensuales.
4. La latencia proviene de la ejecución real.
5. La recomendación final cita costo, latencia y riesgo.
6. Las ventanas de contexto fueron consultadas en documentación oficial.
7. Los precios fueron revisados antes de entregar.

**✅ Validación del paso:**  
Marca cada punto como cumplido.

**📌 Resultado esperado:**  
La matriz está lista para entrega.

---

### ✅ Paso 6. Guarda evidencias

**📝 Descripción del paso:**  
Vas a guardar los archivos que demuestran tu trabajo.

**⚙️ Contenido del paso:**  
Entrega los siguientes archivos:

```text
00_validar_entorno.py
casos_prueba.py
utils.py
precios_modelos.py
01_probar_openai.py
02_probar_gemini.py
03_probar_claude.py
04_comparar_modelos.py
resultados_modelos.csv
Laboratorio_1_Matriz_Seleccion_Modelos.xlsx
```

No entregues el archivo:

```text
.env
```

**✅ Validación del paso:**  
Confirma que `.env` no se incluya en la entrega.

**📌 Resultado esperado:**  
Tienes una entrega completa y segura.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 11 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2011%20de%20un%20laboratorio%20de%20IA%20generativa.%20Valid%C3%A9%20que%20los%20scripts%20compilan%2C%20que%20las%20variables%20de%20entorno%20cargan%2C%20que%20OpenAI%2C%20Gemini%20y%20Claude%20responden%2C%20que%20se%20genera%20el%20CSV%20comparativo%20y%20que%20la%20matriz%20final%20tiene%20datos%20consistentes.)

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Variables de entorno configuradas.
4. Script de validación de entorno.
5. Archivo con casos de uso y prompts.
6. Script individual para OpenAI.
7. Script individual para Gemini.
8. Script individual para Claude.
9. Script comparativo automatizado.
10. Archivo `resultados_modelos.csv`.
11. Matriz `Laboratorio_1_Matriz_Seleccion_Modelos.xlsx`.
12. Recomendación técnica por caso de uso.
13. Evidencia de latencia, tokens estimados y costo mensual.
14. Identificación de riesgos y mitigaciones.
15. Decisión final justificada.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del ambiente local | 10% |
| Configuración segura de API keys | 10% |
| Ejecución correcta de pruebas por proveedor | 20% |
| Medición de latencia y tokens | 15% |
| Cálculo de costo mensual | 15% |
| Construcción de matriz técnica | 10% |
| Evaluación de privacidad y riesgos | 10% |
| Recomendación final justificada | 10% |
| Total | 100% |

---

# ⚠️ Errores comunes que debes evitar

1. Escribir las API keys directamente en el código.
2. Subir el archivo `.env` a un repositorio.
3. Usar modelos sin confirmar que estén disponibles en la cuenta.
4. Comparar proveedores con prompts diferentes.
5. Mezclar tokens de entrada y salida.
6. Usar precios desactualizados.
7. Comparar solo costo e ignorar latencia.
8. Ignorar privacidad en análisis documental.
9. Considerar una sola ejecución como resultado definitivo.
10. Recomendar un modelo sin justificar la decisión.

---

# Cierre de la práctica

En este laboratorio construiste una comparación técnica aplicada entre OpenAI, Gemini y Claude. Preparaste un entorno local en Windows, configuraste claves de API de forma segura, ejecutaste pruebas reales con Python, mediste latencia, estimaste tokens, calculaste costos y documentaste riesgos.

El resultado más importante no es solo el costo mensual estimado, sino la capacidad de justificar una decisión técnica considerando caso de uso, privacidad, ventana de contexto, operación, calidad observada y dependencia del proveedor.

Esta práctica te prepara para tomar decisiones arquitectónicas más sólidas antes de implementar soluciones de IA generativa en escenarios reales.


---

<div align="center">
<strong>Fin del Laboratorio 1</strong><br>
IA Generativa · Selección técnica de modelos · OpenAI · Gemini · Claude
</div>
