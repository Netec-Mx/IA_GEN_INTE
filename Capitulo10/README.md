<div align="center">

# 🧪 Laboratorio 10

## Evaluación de fidelidad de un chatbot con Golden Dataset, ROUGE, BLEU y G-Eval

![Nivel](https://img.shields.io/badge/Nivel-Intermedio%20Avanzado-2563EB?style=flat-square)
![Sistema](https://img.shields.io/badge/Sistema-Windows-0F766E?style=flat-square)
![Editor](https://img.shields.io/badge/Editor-VS%20Code-7C3AED?style=flat-square)
![Terminal](https://img.shields.io/badge/Terminal-Git%20Bash-475569?style=flat-square)
![Lenguaje](https://img.shields.io/badge/Lenguaje-Python-CA8A04?style=flat-square)
![Evaluación](https://img.shields.io/badge/Evaluaci%C3%B3n-ROUGE%20%7C%20BLEU%20%7C%20G--Eval-DB2777?style=flat-square)

</div>

---

> [!IMPORTANT]
> En este laboratorio vas a construir un framework para evaluar la fidelidad de respuestas de un chatbot contra un **Golden Dataset**. Primero ejecutarás una ruta sin costo usando **fixtures locales** y métricas léxicas. Después, de forma opcional, podrás activar generación con API y evaluación tipo **G-Eval** con un modelo juez. No uses datos reales de clientes, credenciales, tickets internos ni información sensible.

<table>
<tr>
<td width="25%"><strong>🎯 Enfoque</strong><br>Evaluación de fidelidad de respuestas</td>
<td width="25%"><strong>⏱️ Duración</strong><br>40 minutos</td>
<td width="25%"><strong>🧠 Bloom</strong><br>Aplicar, analizar, evaluar y crear</td>
<td width="25%"><strong>📦 Entregable</strong><br>Golden Dataset + reportes Markdown/HTML/JSON</td>
</tr>
</table>

---

## 🧭 Sección 1. Información general de la práctica

### 📌 Descripción general

En esta práctica vas a construir una aplicación de evaluación llamada `chatbot_evaluator.py`. La aplicación compara respuestas generadas por un chatbot contra respuestas de referencia definidas en un **Golden Dataset**.

El laboratorio trabaja con tres niveles de evaluación:

1. **Evaluación con fixtures locales:** usa respuestas simuladas para ejecutar el flujo sin consumir API.
2. **Métricas léxicas:** calcula ROUGE-1, ROUGE-2, ROUGE-L y BLEU para medir similitud textual.
3. **Evaluación semántica opcional:** usa un modelo juez con un patrón tipo G-Eval para calificar fidelidad, relevancia, coherencia y fluencia.

También generarás reportes en Markdown, HTML y JSON. Estos reportes te permitirán revisar promedios, mejores respuestas, peores respuestas, divergencias entre métricas y casos que requieren revisión humana.

La práctica original propone crear un Golden Dataset, evaluar respuestas con ROUGE/BLEU, ejecutar G-Eval opcional y generar reportes profesionales. Esta versión mantiene esa base, pero la organiza con una estructura progresiva, detallada y adecuada para Windows, Visual Studio Code y Git Bash.

---

### 🎯 Objetivos de aprendizaje

Al finalizar esta práctica, tú serás capaz de:

1. Preparar un entorno local en Windows para evaluar respuestas generadas por chatbots.
2. Crear un Golden Dataset con preguntas, respuestas de referencia, categorías y dificultad.
3. Crear fixtures locales para ejecutar una evaluación sin consumir API.
4. Implementar modelos de datos con Pydantic para validar entradas y resultados.
5. Calcular métricas ROUGE-1, ROUGE-2, ROUGE-L y BLEU.
6. Interpretar diferencias entre similitud textual y fidelidad factual.
7. Implementar una evaluación tipo G-Eval con salida JSON estructurada.
8. Separar el modelo generador del modelo juez.
9. Usar cache para evitar llamadas repetidas y reducir costo.
10. Generar reportes Markdown, HTML y JSON.
11. Analizar divergencias entre métricas automáticas.
12. Complementar la evaluación automática con revisión humana.

---

### ✅ Prerrequisitos

Antes de iniciar, asegúrate de cumplir con lo siguiente:

1. Tener conocimientos básicos de Python.
2. Saber ejecutar comandos desde Git Bash.
3. Saber crear y activar entornos virtuales.
4. Conocer la estructura básica de archivos JSON.
5. Comprender qué es un chatbot y qué significa evaluar una respuesta.
6. Conocer la diferencia conceptual entre evaluación léxica y evaluación semántica.
7. Tener Visual Studio Code instalado.
8. Tener Python 3.11 o superior instalado.
9. Tener acceso a internet para instalar dependencias desde PyPI.
10. Tener API key de OpenAI solo si vas a ejecutar la ruta opcional con generación o G-Eval.

> [!NOTE]
> No necesitas API key para la ruta base del laboratorio. La ejecución principal puede hacerse con `--use-fixtures --skip-geval`, sin llamadas a modelos externos.

---

### 💻 Hardware

| Recurso | Requisito mínimo | Recomendado |
|---|---:|---:|
| Equipo | Laptop o PC con Windows | Laptop o PC con Windows 11 |
| CPU | 2 núcleos | 4 núcleos o más |
| RAM | 8 GB | 16 GB |
| Almacenamiento libre | 500 MB | 1 GB |
| GPU | No requerida | No requerida |
| Internet | Requerido para instalación | 10 Mbps o superior |

---

### 🧰 Software

| Software / Paquete | Uso |
|---|---|
| Visual Studio Code | Edición de código |
| Git Bash | Ejecución de comandos |
| Python 3.11 o superior | Runtime de la práctica |
| pip | Instalación de dependencias |
| `openai` | Generación opcional y juez LLM opcional |
| `rouge-score` | Cálculo de métricas ROUGE |
| `nltk` | Tokenización y cálculo de BLEU |
| `pandas` | Análisis tabular de resultados |
| `pydantic` | Validación de estructuras de datos |
| `python-dotenv` | Carga de variables de entorno |
| `jinja2` | Plantilla para reporte HTML |
| `markdown` | Conversión de Markdown a HTML |
| `tabulate` | Tablas Markdown desde pandas |
| `tenacity` | Reintentos con backoff |

---

### 📋 Datos generales de la práctica

| Elemento | Detalle |
|---|---|
| Duración estimada | 40 minutos |
| Complejidad | Media - Alta |
| Nivel de Bloom | Aplicar, analizar, evaluar y crear |
| Modalidad | Individual o equipos de 2 personas |
| Sistema operativo | Windows |
| Editor | Visual Studio Code |
| Terminal | Git Bash |
| Lenguaje | Python |
| Costo base | $0 USD usando fixtures locales y `--skip-geval` |
| Costo opcional | Bajo, si ejecutas generación con API o G-Eval |
| Entregable principal | `chatbot_evaluator.py` y reportes de evaluación |
| Entregable secundario | `golden_dataset.json`, `fixtures_responses.json`, cache y checklist de revisión |

---

## 🛡️ Consideraciones para estudiantes

<table>
<tr>
<td><strong>🔐 Seguridad</strong><br>No compartas claves ni subas `.env`.</td>
<td><strong>💸 Costo</strong><br>La ruta base no consume API.</td>
<td><strong>📊 Evaluación</strong><br>Una métrica alta no siempre implica verdad factual.</td>
</tr>
</table>

1. No escribas tu API key dentro del código.
2. No entregues el archivo `.env`.
3. No uses datos reales de clientes en el Golden Dataset.
4. No incluyas correos reales, teléfonos, tokens, contraseñas ni información sensible.
5. Ejecuta primero la ruta sin API con `--use-fixtures --skip-geval`.
6. Ejecuta G-Eval solo después de validar que el pipeline base funciona.
7. Usa cache para evitar costos repetidos.
8. No tomes ROUGE o BLEU como verdad absoluta; son métricas de similitud textual.
9. BLEU puede ser bajo en respuestas abiertas aunque la respuesta sea correcta.
10. La revisión humana sigue siendo necesaria en casos extremos, críticos o divergentes.

---

## 🏗️ Arquitectura del laboratorio

```text
Golden Dataset
      │
      ├── Fixtures locales ───────────────┐
      │                                   │
      └── Modelo generador opcional ──────┤
                                          ▼
                                 Respuestas evaluadas
                                          │
             ┌────────────────────────────┼────────────────────────────┐
             ▼                            ▼                            ▼
        ROUGE/BLEU                  G-Eval opcional              Revisión humana
      métricas léxicas              LLM-as-a-Judge               casos extremos
             └────────────────────────────┼────────────────────────────┘
                                          ▼
                         Reporte Markdown + HTML + JSON
```

---

## 🚀 Sección 2. Desarrollo de la práctica

---

# 🧩 Tarea 1. Preparar el proyecto local en Windows

## 🎯 Objetivo de la tarea

Crear la carpeta del laboratorio, abrirla en Visual Studio Code, crear el entorno virtual, instalar dependencias y preparar los archivos base de configuración.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea la carpeta del laboratorio

**📝 Descripción del paso:**  
Vas a crear una carpeta nueva para el laboratorio. Esta carpeta será la raíz del proyecto y ahí guardarás todos los archivos: datasets JSON, script principal, reportes, cache, configuración y entorno virtual. Ejecuta estos comandos desde Git Bash; no necesitas crear archivos manualmente en este paso.

**⚙️ Contenido del paso:**

```bash
mkdir -p ~/labs-ia-gen/lab-10-evaluacion-chatbot
cd ~/labs-ia-gen/lab-10-evaluacion-chatbot
```

**✅ Validación del paso:**

```bash
pwd
```

**📌 Resultado esperado:**

```text
/c/Users/TU_USUARIO/labs-ia-gen/lab-10-evaluacion-chatbot
```

---

### ✅ Paso 2. Abre el proyecto en Visual Studio Code

**📝 Descripción del paso:**  
Vas a abrir en Visual Studio Code la carpeta `lab-10-evaluacion-chatbot` que acabas de crear. A partir de este punto, todos los archivos nuevos del laboratorio deben crearse dentro de esta carpeta.

**⚙️ Contenido del paso:**

```bash
code .
```

Si `code .` no funciona, abre VS Code manualmente y selecciona:

```text
File > Open Folder > labs-ia-gen > lab-10-evaluacion-chatbot
```

**✅ Validación del paso:**  
Confirma que VS Code muestre la carpeta `lab-10-evaluacion-chatbot`.

**📌 Resultado esperado:**  
El proyecto está abierto en Visual Studio Code.

---

### ✅ Paso 3. Crea y activa el entorno virtual

**📝 Descripción del paso:**  
Vas a crear un entorno virtual llamado `.venv` dentro de la carpeta del laboratorio y después lo vas a activar. Este entorno evita mezclar dependencias del laboratorio con otros proyectos de Python en tu equipo.

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
La ruta de Python debe apuntar a `.venv/Scripts/python` o incluir `.venv`.

---

### ✅ Paso 4. Crea `requirements.txt`

**📝 Descripción del paso:**  
Vas a crear el archivo `requirements.txt` en la raíz del proyecto. Este archivo define las librerías necesarias para calcular métricas, validar datos, generar reportes, cargar configuración y ejecutar llamadas opcionales a modelos.

**⚙️ Contenido del paso:**

```bash
cat > requirements.txt << 'REQ'
openai>=1.90,<2
rouge-score==0.1.2
nltk>=3.9,<4
pandas>=2.2,<3
pydantic>=2.10,<3
python-dotenv>=1.0,<2
jinja2>=3.1,<4
markdown>=3.6,<4
tabulate>=0.9,<1
tenacity>=8.5,<10
REQ
```

**✅ Validación del paso:**

```bash
cat requirements.txt
```

**📌 Resultado esperado:**  
El archivo contiene todas las dependencias del laboratorio.

---

### ✅ Paso 5. Instala dependencias

**📝 Descripción del paso:**  
Vas a instalar dentro del entorno virtual activo todas las dependencias declaradas en `requirements.txt`. Antes de ejecutar este paso, confirma que Git Bash muestre `(.venv)` al inicio de la línea.

**⚙️ Contenido del paso:**

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**✅ Validación del paso:**

```bash
python -c "import openai, nltk, pandas, pydantic, dotenv, markdown, jinja2, tabulate, tenacity; print('Dependencias instaladas correctamente')"
```

**📌 Resultado esperado:**

```text
Dependencias instaladas correctamente
```

---

### ✅ Paso 6. Descarga recursos de NLTK

**📝 Descripción del paso:**  
Vas a descargar los recursos de tokenización que usa NLTK para calcular BLEU. Este paso crea archivos en el directorio local de datos de NLTK del usuario, no dentro del proyecto. Algunas versiones de NLTK pueden pedir también `punkt_tab`; por eso se incluye una validación adicional.

**⚙️ Contenido del paso:**

```bash
python - << 'PY'
import nltk
nltk.download('punkt')
try:
    nltk.download('punkt_tab')
except Exception:
    pass
print('✅ Recursos NLTK listos')
PY
```

**✅ Validación del paso:**

```bash
python - << 'PY'
import nltk
print('NLTK disponible:', nltk.__version__)
PY
```

**📌 Resultado esperado:**  
NLTK queda listo para tokenizar texto durante el cálculo de BLEU.

---

### ✅ Paso 7. Crea carpetas de trabajo

**📝 Descripción del paso:**  
Vas a crear las carpetas `reports/` y `cache/`. La carpeta `reports/` almacenará los reportes Markdown, HTML y JSON. La carpeta `cache/` guardará respuestas generadas y resultados G-Eval para evitar llamadas repetidas a la API.

**⚙️ Contenido del paso:**

```bash
mkdir -p reports cache
```

**✅ Validación del paso:**

```bash
ls -la
```

**📌 Resultado esperado:**

```text
reports/
cache/
```

---

### ✅ Paso 8. Crea `.env.example`, `.env` y `.gitignore`

**📝 Descripción del paso:**  
Vas a crear tres archivos en la raíz del proyecto. `.env.example` documenta las variables esperadas, `.env` guarda configuración local y `.gitignore` evita subir credenciales, entorno virtual, cache y reportes generados.

**⚙️ Contenido del paso:**

```bash
cat > .env.example << 'ENV'
# Copia este archivo como .env si vas a usar la ruta con API.
OPENAI_API_KEY=sk-tu_clave_aqui
OPENAI_MODEL_GENERATION=gpt-4o-mini
OPENAI_MODEL_JUDGE=gpt-4o-mini
ENV

cat > .env << 'ENV'
# Ruta base sin API: puedes dejar esta clave vacía si usarás --use-fixtures --skip-geval
OPENAI_API_KEY=
OPENAI_MODEL_GENERATION=gpt-4o-mini
OPENAI_MODEL_JUDGE=gpt-4o-mini
ENV

cat > .gitignore << 'GIT'
.env
.venv/
__pycache__/
*.pyc
*.pyo
reports/
cache/
GIT
```

**🔧 Qué debes cambiar:**  
Solo agrega tu API key en `.env` si vas a ejecutar generación con modelo o G-Eval. Para la ruta base no necesitas modificar `OPENAI_API_KEY`.

**✅ Validación del paso:**

```bash
grep -q '^.env$' .gitignore && echo '✅ .env está protegido'
```

**📌 Resultado esperado:**

```text
✅ .env está protegido
```

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 1 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%201%20del%20Laboratorio%2010.%20Prepar%C3%A9%20un%20proyecto%20local%20en%20Windows%20con%20VS%20Code%2C%20Git%20Bash%2C%20entorno%20virtual%2C%20requirements.txt%2C%20carpetas%20reports%20y%20cache%2C%20.env.example%2C%20.env%20y%20.gitignore%20para%20evaluar%20la%20fidelidad%20de%20un%20chatbot.)

---

# 🧩 Tarea 2. Crear el Golden Dataset

## 🎯 Objetivo de la tarea

Crear un dataset de referencia con preguntas, respuestas esperadas, categoría y dificultad. Este archivo será la fuente de verdad contra la que compararás las respuestas generadas.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `golden_dataset.json`

**📝 Descripción del paso:**  
Vas a crear el archivo `golden_dataset.json` en la raíz del proyecto. Este archivo contiene 20 ejemplos de evaluación. Cada ejemplo incluye un identificador, una pregunta, una respuesta de referencia, una categoría y un nivel de dificultad. No edites otro archivo en este paso.

**⚙️ Contenido del paso:**

```bash
cat > golden_dataset.json << 'JSON'
[
  {"id": 1, "pregunta": "¿Quién es considerado una figura fundacional de la computación moderna?", "respuesta_referencia": "Alan Turing es considerado una figura fundacional de la computación moderna. En 1936 propuso la máquina de Turing, un modelo matemático abstracto que ayudó a formalizar qué significa computar.", "categoria": "factual", "nivel_dificultad": "easy"},
  {"id": 2, "pregunta": "¿Qué fue ENIAC y cuándo se completó?", "respuesta_referencia": "ENIAC fue una de las primeras computadoras electrónicas digitales de propósito general. Fue completada en 1945 en la Universidad de Pensilvania por John Mauchly y J. Presper Eckert.", "categoria": "factual", "nivel_dificultad": "easy"},
  {"id": 3, "pregunta": "¿Cuál fue la contribución de Grace Hopper al desarrollo del software?", "respuesta_referencia": "Grace Hopper contribuyó al desarrollo de compiladores y lenguajes de alto nivel. Participó en el desarrollo del compilador A-0 y fue una figura clave en la evolución de COBOL, un lenguaje orientado a aplicaciones de negocio.", "categoria": "factual", "nivel_dificultad": "medium"},
  {"id": 4, "pregunta": "¿Qué es la Ley de Moore y cuál es su relevancia actual?", "respuesta_referencia": "La Ley de Moore es una observación formulada por Gordon Moore en 1965 sobre el crecimiento del número de transistores en los circuitos integrados. Ha servido como referencia para la industria, aunque actualmente enfrenta límites físicos, económicos y de fabricación.", "categoria": "inferencia", "nivel_dificultad": "medium"},
  {"id": 5, "pregunta": "¿Cuál fue el papel de Ada Lovelace en la historia de la programación?", "respuesta_referencia": "Ada Lovelace es frecuentemente reconocida como la primera programadora. Trabajó sobre la Máquina Analítica de Charles Babbage y escribió notas que incluían un algoritmo destinado a ser ejecutado por una máquina.", "categoria": "factual", "nivel_dificultad": "easy"},
  {"id": 6, "pregunta": "¿Qué diferencia fundamental existe entre la arquitectura Von Neumann y la arquitectura Harvard?", "respuesta_referencia": "La arquitectura Von Neumann usa una memoria compartida para datos e instrucciones, lo que puede crear un cuello de botella. La arquitectura Harvard separa memoria y buses de datos e instrucciones, permitiendo accesos simultáneos en ciertos sistemas.", "categoria": "inferencia", "nivel_dificultad": "hard"},
  {"id": 7, "pregunta": "¿Qué fue ARPANET y cómo se relaciona con Internet?", "respuesta_referencia": "ARPANET fue una red temprana de conmutación de paquetes financiada por ARPA en Estados Unidos. Es un antecedente directo de Internet; posteriormente, la adopción de TCP/IP en 1983 fue clave para la evolución hacia la red moderna.", "categoria": "factual", "nivel_dificultad": "easy"},
  {"id": 8, "pregunta": "¿Qué impacto tuvo el microprocesador Intel 4004?", "respuesta_referencia": "El Intel 4004, lanzado en 1971, fue uno de los primeros microprocesadores comerciales integrados en un chip. Ayudó a reducir tamaño y costo de sistemas computacionales y abrió camino a la expansión de dispositivos programables y computadoras personales.", "categoria": "inferencia", "nivel_dificultad": "medium"},
  {"id": 9, "pregunta": "¿Cuáles fueron lenguajes importantes de programación de la década de 1950?", "respuesta_referencia": "Entre los lenguajes importantes de la década de 1950 se encuentran FORTRAN, orientado a cálculo científico; LISP, asociado al procesamiento simbólico e inteligencia artificial; y COBOL, diseñado para aplicaciones de negocio.", "categoria": "factual", "nivel_dificultad": "medium"},
  {"id": 10, "pregunta": "¿Por qué UNIX fue significativo para la computación moderna?", "respuesta_referencia": "UNIX fue significativo por su diseño modular, portabilidad y filosofía de herramientas pequeñas que pueden combinarse. Influyó directamente en sistemas posteriores como Linux, BSD y macOS, además de consolidar prácticas importantes de software de sistemas.", "categoria": "inferencia", "nivel_dificultad": "hard"},
  {"id": 11, "pregunta": "¿Qué es el test de Turing?", "respuesta_referencia": "El test de Turing, propuesto por Alan Turing en 1950, evalúa si una máquina puede producir respuestas conversacionales indistinguibles de las de una persona para un evaluador humano bajo ciertas condiciones.", "categoria": "factual", "nivel_dificultad": "easy"},
  {"id": 12, "pregunta": "¿Cuál fue la importancia del lenguaje C en software de sistemas?", "respuesta_referencia": "El lenguaje C, creado por Dennis Ritchie, fue importante porque combinó eficiencia cercana al hardware con mayor portabilidad que el ensamblador. Permitió reescribir gran parte de UNIX y se volvió base de muchos sistemas operativos y herramientas.", "categoria": "inferencia", "nivel_dificultad": "medium"},
  {"id": 13, "pregunta": "¿Qué fue la burbuja puntocom?", "respuesta_referencia": "La burbuja puntocom fue un periodo de especulación financiera alrededor de empresas de Internet, principalmente entre mediados de los años 1990 y 2001. Muchas compañías fueron sobrevaloradas pese a no tener modelos de negocio sostenibles.", "categoria": "inferencia", "nivel_dificultad": "hard"},
  {"id": 14, "pregunta": "¿Qué es el código abierto y cuál fue su impacto?", "respuesta_referencia": "El código abierto es un modelo en el que el código fuente puede ser estudiado, modificado y distribuido bajo ciertas licencias. Su impacto fue enorme en infraestructura, desarrollo colaborativo y adopción de proyectos como Linux, Apache, Python y muchas herramientas modernas.", "categoria": "resumen", "nivel_dificultad": "medium"},
  {"id": 15, "pregunta": "¿Por qué los transistores reemplazaron a los tubos de vacío?", "respuesta_referencia": "Los transistores reemplazaron a los tubos de vacío porque eran más pequeños, consumían menos energía, generaban menos calor y eran más confiables. Esto permitió construir computadoras más compactas, rápidas y eficientes.", "categoria": "factual", "nivel_dificultad": "medium"},
  {"id": 16, "pregunta": "¿Cuál fue la contribución de Tim Berners-Lee?", "respuesta_referencia": "Tim Berners-Lee propuso y desarrolló la World Wide Web mientras trabajaba en el CERN. Su trabajo integró ideas como HTTP, HTML, URL y el primer navegador/editor, facilitando el acceso global a información enlazada en Internet.", "categoria": "factual", "nivel_dificultad": "easy"},
  {"id": 17, "pregunta": "¿Qué lecciones dejó Multics para el diseño de sistemas operativos?", "respuesta_referencia": "Multics mostró el potencial de sistemas multiusuario avanzados, pero también los riesgos de una complejidad excesiva. Algunas de sus lecciones influyeron en UNIX, que adoptó una filosofía más simple, modular y orientada a herramientas pequeñas.", "categoria": "inferencia", "nivel_dificultad": "hard"},
  {"id": 18, "pregunta": "Resume hitos importantes de la evolución de lenguajes de programación entre 1950 y 2000.", "respuesta_referencia": "Entre 1950 y 2000 destacan el uso de ensamblador, FORTRAN y COBOL en los años 1950, LISP para IA, C y Pascal en programación estructurada, Smalltalk y C++ en orientación a objetos, Java en los años 1990 y lenguajes de scripting como Python y JavaScript.", "categoria": "resumen", "nivel_dificultad": "hard"},
  {"id": 19, "pregunta": "¿Por qué la interfaz gráfica de usuario impulsó la adopción masiva de computadoras?", "respuesta_referencia": "La interfaz gráfica redujo la dependencia de comandos de texto y permitió interactuar mediante ventanas, iconos y punteros. Ideas desarrolladas en Xerox PARC e impulsadas comercialmente por sistemas como Macintosh y Windows facilitaron el uso por personas no técnicas.", "categoria": "inferencia", "nivel_dificultad": "medium"},
  {"id": 20, "pregunta": "¿Qué diferencia a la computación cuántica de la computación clásica?", "respuesta_referencia": "La computación clásica usa bits que representan 0 o 1. La computación cuántica usa qubits, que pueden aprovechar superposición y entrelazamiento. Esto puede ofrecer ventajas en ciertos problemas, como simulación cuántica y algunos algoritmos especializados.", "categoria": "factual", "nivel_dificultad": "hard"}
]
JSON
```

**✅ Validación del paso:**

```bash
python - << 'PY'
import json
from collections import Counter

with open('golden_dataset.json', encoding='utf-8') as f:
    data = json.load(f)

print(f'Total de entradas: {len(data)}')
print('Categorías:', Counter(d['categoria'] for d in data))
print('Dificultades:', Counter(d['nivel_dificultad'] for d in data))

assert len(data) == 20
assert all('id' in d for d in data)
assert all('pregunta' in d and 'respuesta_referencia' in d for d in data)
assert all('categoria' in d and 'nivel_dificultad' in d for d in data)
print('✅ Golden Dataset válido')
PY
```

**📌 Resultado esperado:**

```text
Total de entradas: 20
✅ Golden Dataset válido
```

---

### ✅ Paso 2. Inspecciona una muestra del Golden Dataset

**📝 Descripción del paso:**  
Vas a revisar los primeros registros para confirmar que el archivo se puede leer y que las claves principales tienen sentido. Este paso no modifica archivos; solo imprime una vista rápida del dataset.

**⚙️ Contenido del paso:**

```bash
python - << 'PY'
import json
with open('golden_dataset.json', encoding='utf-8') as f:
    data = json.load(f)
for item in data[:3]:
    print('-' * 70)
    print('ID:', item['id'])
    print('Pregunta:', item['pregunta'])
    print('Categoría:', item['categoria'])
    print('Dificultad:', item['nivel_dificultad'])
PY
```

**✅ Validación del paso:**  
Debes ver los primeros 3 ejemplos con pregunta, categoría y dificultad.

**📌 Resultado esperado:**  
El Golden Dataset es legible y está listo para ser usado por el evaluador.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 2 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%202%20del%20Laboratorio%2010.%20Cre%C3%A9%20un%20Golden%20Dataset%20en%20JSON%20con%20preguntas%2C%20respuestas%20de%20referencia%2C%20categor%C3%ADas%20y%20niveles%20de%20dificultad%20para%20evaluar%20fidelidad%20factual%20de%20un%20chatbot.)

---

# 🧩 Tarea 3. Crear fixtures para evaluación sin API

## 🎯 Objetivo de la tarea

Crear respuestas simuladas para ejecutar el pipeline completo sin consumir tokens ni depender de una API externa.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea `fixtures_responses.json`

**📝 Descripción del paso:**  
Vas a crear el archivo `fixtures_responses.json` en la raíz del proyecto. Este archivo contiene una respuesta generada simulada para cada ID del Golden Dataset. Las respuestas incluyen casos buenos, parciales y uno intencionalmente problemático para observar cómo se comportan las métricas.

**⚙️ Contenido del paso:**

```bash
cat > fixtures_responses.json << 'JSON'
[
  {"id": 1, "respuesta_generada": "Alan Turing es una de las figuras centrales de la computación moderna. Su máquina teórica de 1936 formalizó la idea de algoritmo y computación."},
  {"id": 2, "respuesta_generada": "ENIAC fue una computadora electrónica digital de propósito general terminada en 1945 en la Universidad de Pensilvania por Mauchly y Eckert."},
  {"id": 3, "respuesta_generada": "Grace Hopper fue clave en la creación de compiladores y en la evolución de COBOL, ayudando a acercar la programación a lenguajes más legibles."},
  {"id": 4, "respuesta_generada": "La Ley de Moore dice que la capacidad de los chips crece con el tiempo. Hoy sigue siendo una referencia, aunque ya no se cumple de forma tan directa por límites físicos y costos."},
  {"id": 5, "respuesta_generada": "Ada Lovelace trabajó con Charles Babbage y escribió notas para la Máquina Analítica. Por ello suele ser reconocida como una de las primeras personas en describir un programa."},
  {"id": 6, "respuesta_generada": "Von Neumann separa memoria de datos e instrucciones, mientras Harvard usa una sola memoria compartida. Esto hace a Von Neumann más rápida en todos los casos."},
  {"id": 7, "respuesta_generada": "ARPANET fue una red temprana de conmutación de paquetes y se considera un antecedente importante de Internet. La adopción de TCP/IP fue decisiva para su evolución."},
  {"id": 8, "respuesta_generada": "El Intel 4004 ayudó a integrar procesamiento en un chip comercial, reduciendo tamaño y costo, y abrió el camino a dispositivos programables más pequeños."},
  {"id": 9, "respuesta_generada": "En los años 1950 destacaron FORTRAN para cálculo científico, LISP para procesamiento simbólico y COBOL para aplicaciones empresariales."},
  {"id": 10, "respuesta_generada": "UNIX influyó por su portabilidad, su diseño modular y su filosofía de herramientas pequeñas. Su legado aparece en Linux, BSD, macOS y muchos entornos modernos."},
  {"id": 11, "respuesta_generada": "El test de Turing evalúa si una máquina puede conversar de manera que un evaluador no distinga claramente si responde una persona o una máquina."},
  {"id": 12, "respuesta_generada": "C fue importante para software de sistemas porque ofrecía eficiencia y portabilidad. Permitió escribir sistemas operativos y herramientas cercanas al hardware sin depender totalmente de ensamblador."},
  {"id": 13, "respuesta_generada": "La burbuja puntocom fue una etapa de inversión especulativa en empresas de Internet. Muchas compañías crecieron en valoración sin ingresos o modelos sostenibles y luego colapsaron."},
  {"id": 14, "respuesta_generada": "El código abierto permite estudiar, modificar y compartir código bajo licencias específicas. Su impacto se ve en Linux, Apache, Python y la colaboración global de software."},
  {"id": 15, "respuesta_generada": "Los transistores sustituyeron a los tubos de vacío por ser más pequeños, más eficientes, menos calientes y más confiables."},
  {"id": 16, "respuesta_generada": "Tim Berners-Lee creó la World Wide Web en el CERN, integrando HTML, HTTP y URLs para facilitar el acceso a información enlazada."},
  {"id": 17, "respuesta_generada": "Multics mostró ideas avanzadas, pero también una complejidad que dificultó su implementación. UNIX aprendió de esto y favoreció simplicidad y modularidad."},
  {"id": 18, "respuesta_generada": "De 1950 a 2000 hubo una evolución desde ensamblador y FORTRAN hasta COBOL, LISP, C, Pascal, C++, Java, Python y JavaScript, con cambios hacia abstracción, objetos y scripting."},
  {"id": 19, "respuesta_generada": "La GUI facilitó usar computadoras sin memorizar comandos. Ventanas, iconos y punteros hicieron la interacción más intuitiva y favorecieron la adopción masiva."},
  {"id": 20, "respuesta_generada": "La computación cuántica usa qubits y fenómenos como superposición, mientras la clásica usa bits. Puede aportar ventajas para problemas específicos, aunque no reemplaza todo cómputo clásico."}
]
JSON
```

**✅ Validación del paso:**

```bash
python - << 'PY'
import json
with open('fixtures_responses.json', encoding='utf-8') as f:
    data = json.load(f)
assert len(data) == 20
assert all('id' in d and 'respuesta_generada' in d for d in data)
print('✅ Fixtures válidos para ejecución sin API')
PY
```

**📌 Resultado esperado:**

```text
✅ Fixtures válidos para ejecución sin API
```

---

### ✅ Paso 2. Verifica correspondencia entre Golden Dataset y fixtures

**📝 Descripción del paso:**  
Vas a confirmar que cada entrada del Golden Dataset tiene una respuesta fixture con el mismo ID. Este paso evita errores posteriores donde una pregunta quede sin respuesta generada.

**⚙️ Contenido del paso:**

```bash
python - << 'PY'
import json
with open('golden_dataset.json', encoding='utf-8') as f:
    golden = json.load(f)
with open('fixtures_responses.json', encoding='utf-8') as f:
    fixtures = json.load(f)

golden_ids = {item['id'] for item in golden}
fixture_ids = {item['id'] for item in fixtures}

print('IDs Golden:', len(golden_ids))
print('IDs Fixtures:', len(fixture_ids))
print('Faltantes:', sorted(golden_ids - fixture_ids))
print('Sobrantes:', sorted(fixture_ids - golden_ids))

assert golden_ids == fixture_ids
print('✅ Cada pregunta tiene una respuesta fixture')
PY
```

**📌 Resultado esperado:**  
No deben aparecer IDs faltantes ni sobrantes.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 3 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%203%20del%20Laboratorio%2010.%20Cre%C3%A9%20fixtures%20locales%20para%20evaluar%20respuestas%20sin%20consumir%20API%20y%20comparar%20respuestas%20generadas%20contra%20referencias%20usando%20m%C3%A9tricas%20autom%C3%A1ticas.)

---

# 🧩 Tarea 4. Implementar `chatbot_evaluator.py`

## 🎯 Objetivo de la tarea

Crear el framework principal de evaluación con carga de datos, validación de estructuras, generación opcional, métricas ROUGE/BLEU, G-Eval opcional, cache y generación de reportes.

---

## 🛠️ Pasos

### ✅ Paso 1. Crea el archivo principal `chatbot_evaluator.py`

**📝 Descripción del paso:**  
Vas a crear el archivo `chatbot_evaluator.py` en la raíz del proyecto. Este será el script principal del laboratorio. Cópialo completo como un solo bloque para evitar errores de indentación, imports faltantes o funciones incompletas.

**⚙️ Contenido del paso:**

```bash
cat > chatbot_evaluator.py << 'PY'
from __future__ import annotations
import argparse, json, os, re, time
from datetime import datetime
from pathlib import Path
from typing import Optional
import markdown, nltk, pandas as pd
from dotenv import load_dotenv
from jinja2 import Template
from nltk.translate.bleu_score import SmoothingFunction, sentence_bleu
from openai import OpenAI
from pydantic import BaseModel, Field, field_validator
from rouge_score import rouge_scorer
from tenacity import retry, retry_if_exception_type, stop_after_attempt, wait_random_exponential
try:
    from openai import APIError, RateLimitError
except Exception:
    APIError = Exception
    RateLimitError = Exception
load_dotenv()
MODEL_GENERATION=os.getenv("OPENAI_MODEL_GENERATION","gpt-4o-mini")
MODEL_JUDGE=os.getenv("OPENAI_MODEL_JUDGE","gpt-4o-mini")
OPENAI_API_KEY=os.getenv("OPENAI_API_KEY","").strip()
CACHE_DIR=Path("cache"); REPORTS_DIR=Path("reports"); CACHE_DIR.mkdir(exist_ok=True); REPORTS_DIR.mkdir(exist_ok=True)
class GoldenEntry(BaseModel): id:int; pregunta:str; respuesta_referencia:str; categoria:str; nivel_dificultad:str
class FixtureResponse(BaseModel): id:int; respuesta_generada:str
class RougeScores(BaseModel): rouge1_precision:float; rouge1_recall:float; rouge1_f1:float; rouge2_precision:float; rouge2_recall:float; rouge2_f1:float; rougeL_precision:float; rougeL_recall:float; rougeL_f1:float
class GEvalJudgeOutput(BaseModel):
    fidelidad:float=Field(ge=0,le=5); relevancia:float=Field(ge=0,le=5); coherencia:float=Field(ge=0,le=5); fluencia:float=Field(ge=0,le=5); justificacion:str
    @field_validator("justificacion")
    @classmethod
    def validar_justificacion(cls,v:str)->str: return v.strip() or "Sin justificación disponible."
class GEvalResult(GEvalJudgeOutput):
    score_promedio:float; judge_model:str
    @classmethod
    def from_output(cls,o:GEvalJudgeOutput,judge_model:str)->"GEvalResult": return cls(**o.model_dump(),score_promedio=round((o.fidelidad+o.relevancia+o.coherencia+o.fluencia)/4,4),judge_model=judge_model)
class EvaluationEntry(BaseModel): golden:GoldenEntry; respuesta_generada:str; rouge_scores:RougeScores; bleu_score:float; geval_result:Optional[GEvalResult]=None
class EvaluationResults(BaseModel): entries:list[EvaluationEntry]; model_generation:str; model_judge:str; used_fixtures:bool; used_geval:bool; timestamp:str
def get_openai_client()->OpenAI:
    if not OPENAI_API_KEY: raise RuntimeError("OPENAI_API_KEY no está configurada. Usa --use-fixtures --skip-geval para ejecutar sin API.")
    return OpenAI(api_key=OPENAI_API_KEY)
def load_golden_dataset(path:str="golden_dataset.json")->list[GoldenEntry]:
    data=[GoldenEntry(**e) for e in json.loads(Path(path).read_text(encoding="utf-8"))]
    if len({e.id for e in data})!=len(data): raise ValueError("IDs duplicados en Golden Dataset")
    return data
def load_fixture_responses(path:str="fixtures_responses.json")->dict[int,str]:
    return {FixtureResponse(**x).id:FixtureResponse(**x).respuesta_generada for x in json.loads(Path(path).read_text(encoding="utf-8"))}
def load_or_none(path:Path): return json.loads(path.read_text(encoding="utf-8")) if path.exists() else None
def save_json(path:Path,data)->None: path.write_text(json.dumps(data,ensure_ascii=False,indent=2),encoding="utf-8")
@retry(wait=wait_random_exponential(multiplier=1,min=2,max=30),stop=stop_after_attempt(4),retry=retry_if_exception_type((RateLimitError,APIError)),reraise=True)
def call_generation_model(client:OpenAI,question:str,model:str)->str:
    r=client.chat.completions.create(model=model,messages=[{"role":"system","content":"Eres un asistente experto en historia de la computación. Responde de forma precisa, clara y concisa en español."},{"role":"user","content":question}],temperature=0.2,max_tokens=320)
    return (r.choices[0].message.content or "").strip()
def get_predictions(dataset,use_fixtures,refresh_cache,model):
    cache_path=CACHE_DIR/f"generated_responses_{model}.json"
    if use_fixtures:
        print("✅ Usando fixtures locales. No se consumirán tokens para generación."); fx=load_fixture_responses(); return [fx.get(e.id,"") for e in dataset]
    if not refresh_cache and (cached:=load_or_none(cache_path)): print(f"✅ Respuestas cargadas desde cache: {cache_path}"); return [i["respuesta_generada"] for i in sorted(cached,key=lambda x:x["id"])]
    client=get_openai_client(); preds=[]
    for i,e in enumerate(dataset,1): print(f"[{i:02d}/{len(dataset)}] {e.pregunta[:70]}..."); preds.append(call_generation_model(client,e.pregunta,model)); time.sleep(.3)
    save_json(cache_path,[{"id":e.id,"pregunta":e.pregunta,"respuesta_generada":p} for e,p in zip(dataset,preds)]); return preds
def normalize_text(t:str)->str: return re.sub(r"\s+"," ",re.sub(r"[^\wáéíóúñü\s]"," ",t.lower().strip(),flags=re.I))
def tokenize(t:str)->list[str]:
    try: return nltk.word_tokenize(normalize_text(t),language="spanish")
    except Exception: return normalize_text(t).split()
def calculate_rouge_scores(predictions,references):
    scorer=rouge_scorer.RougeScorer(["rouge1","rouge2","rougeL"],use_stemmer=True); out=[]
    for p,r in zip(predictions,references):
        if not p.strip(): out.append(RougeScores(rouge1_precision=0,rouge1_recall=0,rouge1_f1=0,rouge2_precision=0,rouge2_recall=0,rouge2_f1=0,rougeL_precision=0,rougeL_recall=0,rougeL_f1=0)); continue
        s=scorer.score(r,p); out.append(RougeScores(rouge1_precision=round(s["rouge1"].precision,4),rouge1_recall=round(s["rouge1"].recall,4),rouge1_f1=round(s["rouge1"].fmeasure,4),rouge2_precision=round(s["rouge2"].precision,4),rouge2_recall=round(s["rouge2"].recall,4),rouge2_f1=round(s["rouge2"].fmeasure,4),rougeL_precision=round(s["rougeL"].precision,4),rougeL_recall=round(s["rougeL"].recall,4),rougeL_f1=round(s["rougeL"].fmeasure,4)))
    return out
def calculate_bleu_scores(predictions,references):
    sm=SmoothingFunction().method1; return [0.0 if not p.strip() else round(sentence_bleu([tokenize(r)],tokenize(p),weights=(.25,.25,.25,.25),smoothing_function=sm),4) for p,r in zip(predictions,references)]
GEVAL_SYSTEM_PROMPT="""Eres un evaluador experto. Evalúa con escala 0 a 5 en fidelidad, relevancia, coherencia y fluencia. Devuelve JSON válido: fidelidad, relevancia, coherencia, fluencia, justificacion."""
@retry(wait=wait_random_exponential(multiplier=1,min=2,max=30),stop=stop_after_attempt(4),retry=retry_if_exception_type((RateLimitError,APIError)),reraise=True)
def call_judge_model(client,prompt,model):
    r=client.chat.completions.create(model=model,messages=[{"role":"system","content":GEVAL_SYSTEM_PROMPT},{"role":"user","content":prompt}],temperature=0,max_tokens=500,response_format={"type":"json_object"}); return r.choices[0].message.content or "{}"
def evaluate_with_geval(entry,prediction,model,client):
    prompt=f"Pregunta:\n{entry.pregunta}\n\nReferencia:\n{entry.respuesta_referencia}\n\nRespuesta generada:\n{prediction}\n\nEvalúa la respuesta."
    try: return GEvalResult.from_output(GEvalJudgeOutput.model_validate_json(call_judge_model(client,prompt,model)),model)
    except Exception as exc: return GEvalResult.from_output(GEvalJudgeOutput(fidelidad=0,relevancia=0,coherencia=0,fluencia=0,justificacion=f"Error durante G-Eval: {exc}"),model)
def get_geval_results(dataset,predictions,skip_geval,refresh_cache,model):
    if skip_geval: print("✅ G-Eval omitido. Solo se calcularán métricas léxicas."); return None
    cache_path=CACHE_DIR/f"geval_results_{model}.json"
    if not refresh_cache and (cached:=load_or_none(cache_path)): print(f"✅ Resultados G-Eval cargados desde cache: {cache_path}"); return [GEvalResult(**x) for x in cached]
    client=get_openai_client(); res=[evaluate_with_geval(e,p,model,client) for e,p in zip(dataset,predictions)]; save_json(cache_path,[x.model_dump() for x in res]); return res
def build_results_dataframe(results):
    rows=[]
    for e in results.entries:
        row={"id":e.golden.id,"categoria":e.golden.categoria,"dificultad":e.golden.nivel_dificultad,"pregunta":e.golden.pregunta,"rouge1_f1":e.rouge_scores.rouge1_f1,"rouge2_f1":e.rouge_scores.rouge2_f1,"rougeL_f1":e.rouge_scores.rougeL_f1,"bleu4":e.bleu_score}
        if e.geval_result: row.update({"geval_fidelidad":e.geval_result.fidelidad,"geval_relevancia":e.geval_result.relevancia,"geval_coherencia":e.geval_result.coherencia,"geval_fluencia":e.geval_result.fluencia,"geval_promedio":e.geval_result.score_promedio,"geval_justificacion":e.geval_result.justificacion})
        rows.append(row)
    return pd.DataFrame(rows)
def generate_evaluation_report(results):
    df=build_results_dataframe(results); has_geval="geval_promedio" in df.columns; sort_col="geval_promedio" if has_geval else "rouge1_f1"; metrics=["rouge1_f1","rouge2_f1","rougeL_f1","bleu4"]+(["geval_promedio"] if has_geval else [])
    lines=["# Reporte de Evaluación de Fidelidad del Chatbot\n",f"**Fecha:** {results.timestamp}  ",f"**Modelo evaluado:** `{results.model_generation}`  ",f"**Modelo juez:** `{results.model_judge}`  ",f"**Uso de fixtures:** `{results.used_fixtures}`  ",f"**Uso de G-Eval:** `{results.used_geval}`  ",f"**Total de ejemplos:** {len(results.entries)}\n","---\n","## 1. Resumen global\n",df[metrics].agg(["mean","min","max"]).T.round(4).to_markdown(),"\n","## 2. Análisis por categoría\n",df.groupby("categoria")[metrics].mean().round(4).to_markdown(),"\n","## 3. Análisis por dificultad\n",df.groupby("dificultad")[metrics].mean().round(4).to_markdown(),"\n"]
    for title,data in [("## 4. Top 3 mejores respuestas",df.nlargest(3,sort_col)),("## 5. Top 3 peores respuestas",df.nsmallest(3,sort_col))]:
        lines.append(title+"\n")
        for _,row in data.iterrows():
            ent=next(x for x in results.entries if x.golden.id==row["id"]); lines += [f"### ID {ent.golden.id} — score `{row[sort_col]:.4f}`\n",f"**Pregunta:** {ent.golden.pregunta}\n",f"**Referencia:** {ent.golden.respuesta_referencia}\n",f"**Respuesta generada:** {ent.respuesta_generada}\n",f"**ROUGE-1:** {ent.rouge_scores.rouge1_f1} | **ROUGE-L:** {ent.rouge_scores.rougeL_f1} | **BLEU-4:** {ent.bleu_score}\n"]
            if ent.geval_result: lines.append(f"**G-Eval:** {ent.geval_result.score_promedio}/5 — {ent.geval_result.justificacion}\n")
    lines += ["## 6. Tabla completa de resultados\n",df[["id","categoria","dificultad","rouge1_f1","rouge2_f1","rougeL_f1","bleu4"]+(["geval_promedio"] if has_geval else [])].round(4).to_markdown(index=False),"\n","## 7. Matriz de interpretación\n","| Caso | Interpretación | Acción recomendada |\n|---|---|---|\n| ROUGE alto + G-Eval alto | Respuesta cercana y correcta | Mantener |\n| ROUGE bajo + G-Eval alto | Parafraseo correcto | No penalizar automáticamente |\n| ROUGE alto + G-Eval bajo | Texto similar con posible error factual | Revisar manualmente |\n| ROUGE bajo + G-Eval bajo | Respuesta incorrecta o irrelevante | Corregir prompt, datos o modelo |\n","## 8. Conclusiones\n",f"- ROUGE-1 F1 promedio: `{df['rouge1_f1'].mean():.4f}`.",f"- BLEU-4 promedio: `{df['bleu4'].mean():.4f}`.","- Recomendación: usa métricas léxicas como filtro económico y G-Eval/revisión humana para casos críticos o divergentes."]
    return "\n".join(lines)
def export_html_report(markdown_content,output_path):
    body=markdown.markdown(markdown_content,extensions=["tables","fenced_code"]); html=f"<!DOCTYPE html><html lang='es'><head><meta charset='UTF-8'><title>Reporte</title></head><body>{body}</body></html>"; Path(output_path).write_text(html,encoding="utf-8")
def run_pipeline(use_fixtures,skip_geval,refresh_cache):
    print("\n"+"="*72); print("FRAMEWORK DE EVALUACIÓN DE FIDELIDAD DE CHATBOT"); print("="*72)
    data=load_golden_dataset(); print(f"✅ Golden Dataset cargado: {len(data)} entradas"); preds=get_predictions(data,use_fixtures,refresh_cache,MODEL_GENERATION); refs=[e.respuesta_referencia for e in data]
    print("✅ Calculando ROUGE..."); rouge=calculate_rouge_scores(preds,refs); print("✅ Calculando BLEU..."); bleu=calculate_bleu_scores(preds,refs); geval=get_geval_results(data,preds,skip_geval,refresh_cache,MODEL_JUDGE)
    entries=[EvaluationEntry(golden=g,respuesta_generada=p,rouge_scores=r,bleu_score=b,geval_result=geval[i] if geval else None) for i,(g,p,r,b) in enumerate(zip(data,preds,rouge,bleu))]
    results=EvaluationResults(entries=entries,model_generation=MODEL_GENERATION if not use_fixtures else "fixtures-locales",model_judge=MODEL_JUDGE if not skip_geval else "no-aplica",used_fixtures=use_fixtures,used_geval=not skip_geval,timestamp=datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
    report=generate_evaluation_report(results); md=REPORTS_DIR/"evaluation_report.md"; html=REPORTS_DIR/"evaluation_report.html"; js=REPORTS_DIR/"evaluation_results.json"; md.write_text(report,encoding="utf-8"); export_html_report(report,str(html)); save_json(js,results.model_dump())
    df=build_results_dataframe(results); print(f"ROUGE-1 F1 promedio: {df['rouge1_f1'].mean():.4f}"); print(f"BLEU-4 promedio: {df['bleu4'].mean():.4f}"); print(f"✅ {md}\n✅ {html}\n✅ {js}")
if __name__=="__main__":
    parser=argparse.ArgumentParser(description="Evalúa fidelidad de un chatbot contra un Golden Dataset."); parser.add_argument("--use-fixtures",action="store_true"); parser.add_argument("--skip-geval",action="store_true"); parser.add_argument("--refresh-cache",action="store_true"); args=parser.parse_args(); run_pipeline(args.use_fixtures,args.skip_geval,args.refresh_cache)

PY
```

**✅ Validación del paso:**

```bash
python -m py_compile chatbot_evaluator.py && echo '✅ Sintaxis OK'
```

**📌 Resultado esperado:**

```text
✅ Sintaxis OK
```

---

### ✅ Paso 2. Revisa la ayuda del script

**📝 Descripción del paso:**  
Vas a ejecutar el script con `--help` para confirmar que expone correctamente sus opciones de ejecución. Este paso no evalúa datos todavía; solo valida la interfaz de línea de comandos.

**⚙️ Contenido del paso:**

```bash
python chatbot_evaluator.py --help
```

**✅ Validación del paso:**  
Debes ver las opciones `--use-fixtures`, `--skip-geval` y `--refresh-cache`.

**📌 Resultado esperado:**  
El script está listo para ejecutar la ruta base y las rutas opcionales.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 4 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%204%20del%20Laboratorio%2010.%20Implement%C3%A9%20chatbot_evaluator.py%20con%20Pydantic%2C%20carga%20de%20datos%2C%20generaci%C3%B3n%20opcional%2C%20ROUGE%2C%20BLEU%2C%20G-Eval%20opcional%2C%20cache%20y%20reportes%20Markdown%2C%20HTML%20y%20JSON.)

---

# 🧩 Tarea 5. Ejecutar evaluación sin API

## 🎯 Objetivo de la tarea

Validar todo el pipeline base usando fixtures locales y omitiendo G-Eval para no consumir tokens.

---

## 🛠️ Pasos

### ✅ Paso 1. Ejecuta con fixtures y sin G-Eval

**📝 Descripción del paso:**  
Vas a ejecutar `chatbot_evaluator.py` usando `--use-fixtures --skip-geval`. Con esto el script leerá `golden_dataset.json`, cargará respuestas desde `fixtures_responses.json`, calculará ROUGE y BLEU, y generará reportes sin llamar a ningún modelo externo.

**⚙️ Contenido del paso:**

```bash
python chatbot_evaluator.py --use-fixtures --skip-geval
```

**✅ Validación del paso:**  
La salida debe indicar que se usaron fixtures, que se calcularon ROUGE y BLEU, y que G-Eval fue omitido.

**📌 Resultado esperado:**

```text
✅ Golden Dataset cargado: 20 entradas
✅ Usando fixtures locales. No se consumirán tokens para generación.
✅ Calculando ROUGE...
✅ Calculando BLEU...
✅ G-Eval omitido. Solo se calcularán métricas léxicas.
```

---

### ✅ Paso 2. Verifica archivos generados

**📝 Descripción del paso:**  
Vas a listar la carpeta `reports/` para confirmar que el evaluador generó los tres archivos principales: reporte Markdown, reporte HTML y resultados JSON.

**⚙️ Contenido del paso:**

```bash
ls -la reports/
```

**✅ Validación del paso:**  
Debes encontrar:

```text
evaluation_report.md
evaluation_report.html
evaluation_results.json
```

**📌 Resultado esperado:**  
La ruta base del laboratorio generó evidencias completas sin usar API.

---

### ✅ Paso 3. Valida el JSON de resultados

**📝 Descripción del paso:**  
Vas a abrir `reports/evaluation_results.json` con Python para verificar que contiene 20 entradas evaluadas y que el archivo registra correctamente si se usaron fixtures y si G-Eval fue omitido.

**⚙️ Contenido del paso:**

```bash
python - << 'PY'
import json
with open('reports/evaluation_results.json', encoding='utf-8') as f:
    data = json.load(f)
entries = data['entries']
assert len(entries) == 20
print('Total evaluado:', len(entries))
print('Usó fixtures:', data['used_fixtures'])
print('Usó G-Eval:', data['used_geval'])
print('✅ JSON de resultados válido')
PY
```

**📌 Resultado esperado:**

```text
Total evaluado: 20
Usó fixtures: True
Usó G-Eval: False
✅ JSON de resultados válido
```

---

### ✅ Paso 4. Abre el reporte Markdown

**📝 Descripción del paso:**  
Vas a abrir el reporte Markdown en VS Code para revisar el resumen global, análisis por categoría, análisis por dificultad, mejores respuestas, peores respuestas y tabla completa de resultados.

**⚙️ Contenido del paso:**

```bash
code reports/evaluation_report.md
```

**✅ Validación del paso:**  
El archivo debe abrirse y mostrar secciones de análisis generadas automáticamente.

**📌 Resultado esperado:**  
Puedes revisar la calidad de las respuestas fixture sin consumo de API.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 5 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%205%20del%20Laboratorio%2010.%20Ejecut%C3%A9%20la%20evaluaci%C3%B3n%20sin%20API%20usando%20fixtures%20locales%20y%20--skip-geval%2C%20generando%20reportes%20en%20Markdown%2C%20HTML%20y%20JSON.)

---

# 🧩 Tarea 6. Analizar métricas léxicas ROUGE y BLEU

## 🎯 Objetivo de la tarea

Interpretar las métricas de similitud textual y reconocer sus límites al evaluar respuestas abiertas de chatbots.

---

## 🛠️ Pasos

### ✅ Paso 1. Revisa el resumen global del reporte

**📝 Descripción del paso:**  
Vas a abrir `reports/evaluation_report.md` y localizar la sección `## 1. Resumen global`. Ahí encontrarás promedios, mínimos y máximos para ROUGE-1, ROUGE-2, ROUGE-L y BLEU.

**⚙️ Contenido del paso:**

```bash
code reports/evaluation_report.md
```

**✅ Validación del paso:**  
Busca la sección:

```text
## 1. Resumen global
```

**📌 Resultado esperado:**  
Puedes identificar qué métricas son más altas y cuáles son más estrictas.

---

### ✅ Paso 2. Interpreta ROUGE

**📝 Descripción del paso:**  
Vas a interpretar qué observa cada variante de ROUGE. Este paso es analítico: no modifica archivos. Úsalo para explicar por qué una respuesta puede tener buen ROUGE-1 pero menor ROUGE-2.

**⚙️ Contenido del paso:**

| Métrica | Qué observa | Cómo interpretarla |
|---|---|---|
| ROUGE-1 | Coincidencia de palabras individuales | Útil para cobertura general |
| ROUGE-2 | Coincidencia de pares de palabras | Más estricta, suele ser menor |
| ROUGE-L | Subsecuencia común más larga | Tolera algo de reordenamiento |

> [!NOTE]
> En este laboratorio se usa `use_stemmer=True` como aproximación. No es un stemmer especializado para español; en producción conviene usar normalización específica del idioma.

**✅ Validación del paso:**  
Explica con tus palabras por qué ROUGE-2 puede ser menor que ROUGE-1.

**📌 Resultado esperado:**  
Comprendes que ROUGE mide superposición textual, no necesariamente verdad factual.

---

### ✅ Paso 3. Interpreta BLEU

**📝 Descripción del paso:**  
Vas a revisar BLEU como métrica de coincidencia de n-gramas. BLEU fue diseñado originalmente para traducción automática, por lo que puede penalizar respuestas correctas que estén redactadas de forma diferente a la referencia.

**⚙️ Contenido del paso:**

| Valor BLEU | Interpretación didáctica |
|---:|---|
| 0.00 - 0.05 | Baja coincidencia de n-gramas |
| 0.05 - 0.15 | Normal en respuestas libres |
| 0.15 - 0.35 | Coincidencia textual razonable |
| > 0.35 | Respuesta muy cercana a la referencia |

**✅ Validación del paso:**  
Identifica si BLEU promedio es menor que ROUGE-1.

**📌 Resultado esperado:**  
Comprendes por qué BLEU puede ser bajo aunque una respuesta sea aceptable.

---

### ✅ Paso 4. Extrae métricas desde JSON

**📝 Descripción del paso:**  
Vas a leer `reports/evaluation_results.json` para calcular manualmente el promedio de ROUGE-1 y BLEU. Esto valida que el reporte no es una caja negra: los datos base están disponibles en JSON.

**⚙️ Contenido del paso:**

```bash
python - << 'PY'
import json
with open('reports/evaluation_results.json', encoding='utf-8') as f:
    data = json.load(f)
rouge1 = [e['rouge_scores']['rouge1_f1'] for e in data['entries']]
bleu = [e['bleu_score'] for e in data['entries']]
print(f'ROUGE-1 promedio: {sum(rouge1)/len(rouge1):.4f}')
print(f'BLEU-4 promedio: {sum(bleu)/len(bleu):.4f}')
PY
```

**✅ Validación del paso:**  
Los valores deben coincidir o ser consistentes con los del reporte Markdown.

**📌 Resultado esperado:**  
Puedes extraer métricas desde el JSON para análisis externo.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 6 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%206%20del%20Laboratorio%2010.%20Analic%C3%A9%20m%C3%A9tricas%20l%C3%A9xicas%20ROUGE%20y%20BLEU%20para%20interpretar%20similitud%20textual%2C%20cobertura%20y%20limitaciones%20en%20respuestas%20abiertas%20de%20chatbots.)

---

# 🧩 Tarea 7. Ejecutar G-Eval opcional con cache

## 🎯 Objetivo de la tarea

Evaluar respuestas con un modelo juez usando criterios de fidelidad, relevancia, coherencia y fluencia.

> [!WARNING]
> Esta tarea consume API. Ejecútala solo después de completar correctamente la ruta sin API.

---

## 🛠️ Pasos

### ✅ Paso 1. Configura `.env` para usar API

**📝 Descripción del paso:**  
Vas a abrir el archivo `.env` en VS Code y agregar tu API key real. Este paso es necesario solo si vas a ejecutar G-Eval o generación con modelo. No modifiques `.env.example`; ese archivo solo documenta la estructura.

**⚙️ Contenido del paso:**

```env
OPENAI_API_KEY=sk-tu_clave_real
OPENAI_MODEL_GENERATION=gpt-4o-mini
OPENAI_MODEL_JUDGE=gpt-4o-mini
```

**✅ Validación del paso:**

```bash
python - << 'PY'
from dotenv import load_dotenv
import os

load_dotenv(dotenv_path=".env")

print('API key configurada:', bool(os.getenv('OPENAI_API_KEY')))
print('Modelo juez:', os.getenv('OPENAI_MODEL_JUDGE'))
PY
```

**📌 Resultado esperado:**

```text
API key configurada: True
```

---

### ✅ Paso 2. Ejecuta G-Eval sobre fixtures

**📝 Descripción del paso:**  
Vas a mantener las respuestas locales de `fixtures_responses.json`, pero usarás un modelo juez para calificarlas. Esta ruta permite probar G-Eval sin pagar generación de respuestas.

**⚙️ Contenido del paso:**

```bash
python chatbot_evaluator.py --use-fixtures
```

**✅ Validación del paso:**  
La salida debe mostrar que se ejecuta G-Eval y que se generan resultados del juez.

**📌 Resultado esperado:**  
El reporte se regenera con columnas o secciones relacionadas con G-Eval.

---

### ✅ Paso 3. Verifica cache de G-Eval

**📝 Descripción del paso:**  
Vas a revisar la carpeta `cache/`. El script guarda los resultados del juez para evitar repetir llamadas si vuelves a ejecutar el mismo flujo.

**⚙️ Contenido del paso:**

```bash
ls -la cache/
```

**✅ Validación del paso:**  
Debes ver un archivo similar a:

```text
geval_results_gpt-4o-mini.json
```

**📌 Resultado esperado:**  
La evaluación G-Eval quedó cacheada.

---

### ✅ Paso 4. Repite la ejecución usando cache

**📝 Descripción del paso:**  
Vas a ejecutar nuevamente el mismo comando. Esta vez el script debe cargar resultados desde cache en lugar de hacer llamadas nuevas al juez, siempre que no uses `--refresh-cache`.

**⚙️ Contenido del paso:**

```bash
python chatbot_evaluator.py --use-fixtures
```

**✅ Validación del paso:**  
Debe aparecer un mensaje indicando que los resultados G-Eval se cargaron desde cache.

**📌 Resultado esperado:**  
Evitas llamadas repetidas y reduces costo.

---

### ✅ Paso 5. Fuerza regeneración de cache solo si lo necesitas

**📝 Descripción del paso:**  
Vas a ejecutar con `--refresh-cache` únicamente si quieres recalcular resultados del juez. Este paso puede consumir API otra vez.

**⚙️ Contenido del paso:**

```bash
python chatbot_evaluator.py --use-fixtures --refresh-cache
```

**✅ Validación del paso:**  
El script vuelve a evaluar los 20 ejemplos con el modelo juez.

**📌 Resultado esperado:**  
La cache se actualiza con resultados nuevos.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 7 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%207%20del%20Laboratorio%2010.%20Configur%C3%A9%20G-Eval%20opcional%20con%20API%20key%2C%20us%C3%A9%20un%20modelo%20juez%2C%20activ%C3%A9%20cache%20y%20compar%C3%A9%20fidelidad%2C%20relevancia%2C%20coherencia%20y%20fluencia.)

---

# 🧩 Tarea 8. Ejecutar ruta completa con modelo generador opcional

## 🎯 Objetivo de la tarea

Generar respuestas con un modelo, evaluarlas contra el Golden Dataset y comparar resultados con o sin G-Eval.

> [!WARNING]
> Esta tarea consume API porque genera respuestas. El costo dependerá del modelo, el número de ejemplos y si también ejecutas G-Eval.

---

## 🛠️ Pasos

### ✅ Paso 1. Ejecuta generación y métricas léxicas sin G-Eval

**📝 Descripción del paso:**  
Vas a pedir al modelo configurado en `.env` que genere respuestas para las 20 preguntas del Golden Dataset. Después calcularás ROUGE y BLEU, pero omitirás G-Eval para reducir costo.

**⚙️ Contenido del paso:**

```bash
python chatbot_evaluator.py --skip-geval
```

**✅ Validación del paso:**  
La salida debe indicar que el modelo generó respuestas y que se calcularon ROUGE/BLEU.

**📌 Resultado esperado:**  
Se genera o actualiza `cache/generated_responses_gpt-4o-mini.json` y los reportes en `reports/`.

---

### ✅ Paso 2. Ejecuta pipeline completo

**📝 Descripción del paso:**  
Vas a ejecutar el flujo completo: generación de respuestas, cálculo de métricas léxicas y G-Eval con modelo juez. Hazlo solo cuando ya tengas controlado el costo y hayas validado la ruta base.

**⚙️ Contenido del paso:**

```bash
python chatbot_evaluator.py
```

**✅ Validación del paso:**  
Debe generarse un reporte con métricas ROUGE/BLEU y G-Eval.

**📌 Resultado esperado:**  
Tienes una evaluación completa de respuestas generadas por modelo.

---

### ✅ Paso 3. Revisa cache

**📝 Descripción del paso:**  
Vas a listar la carpeta `cache/` para confirmar que existen archivos de respuestas generadas y, si ejecutaste G-Eval, resultados del juez.

**⚙️ Contenido del paso:**

```bash
ls -la cache/
```

**✅ Validación del paso:**  
Debes ver cache de generación y cache del juez si ejecutaste el pipeline completo.

**📌 Resultado esperado:**  
El pipeline puede repetirse sin recalcular todo, salvo que uses `--refresh-cache`.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 8 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%208%20del%20Laboratorio%2010.%20Ejecut%C3%A9%20la%20ruta%20completa%20con%20modelo%20generador%20opcional%2C%20calcul%C3%A9%20m%C3%A9tricas%20y%20evalu%C3%A9%20resultados%20con%20o%20sin%20G-Eval%20usando%20cache.)

---

# 🧩 Tarea 9. Analizar divergencias entre métricas

## 🎯 Objetivo de la tarea

Identificar casos donde ROUGE/BLEU y G-Eval no coinciden, para entender cuándo una respuesta puede ser correcta aunque no se parezca textualmente a la referencia, o viceversa.

---

## 🛠️ Pasos

### ✅ Paso 1. Abre el reporte

**📝 Descripción del paso:**  
Vas a abrir `reports/evaluation_report.md` y revisar las secciones de correlación y divergencias. Si ejecutaste con `--skip-geval`, esas secciones pueden no aparecer porque no existe evaluación con juez.

**⚙️ Contenido del paso:**

```bash
code reports/evaluation_report.md
```

**✅ Validación del paso:**  
Busca estas secciones:

```text
## 6. Correlación entre métricas
## 7. Casos de divergencia
```

**📌 Resultado esperado:**  
Puedes ubicar los casos donde las métricas discrepan.

---

### ✅ Paso 2. Interpreta patrones de divergencia

**📝 Descripción del paso:**  
Vas a clasificar cada divergencia usando una matriz de interpretación. Este paso es analítico; no modifica archivos automáticamente.

**⚙️ Contenido del paso:**

| Patrón | Significado probable |
|---|---|
| ROUGE bajo + G-Eval alto | La respuesta parafrasea correctamente |
| ROUGE alto + G-Eval bajo | La respuesta usa palabras similares pero puede tener errores factuales |
| BLEU bajo + G-Eval alto | Normal en respuestas abiertas |
| Todo bajo | Respuesta deficiente o fuera de tema |

**✅ Validación del paso:**  
Selecciona al menos un caso del reporte y clasifícalo con esta tabla.

**📌 Resultado esperado:**  
Puedes explicar por qué no basta con una sola métrica.

---

### ✅ Paso 3. Genera lista de casos extremos

**📝 Descripción del paso:**  
Vas a leer `reports/evaluation_results.json` y calcular diferencias entre G-Eval normalizado y ROUGE-1. Este paso solo mostrará resultados si ejecutaste G-Eval.

**⚙️ Contenido del paso:**

```bash
python - << 'PY'
import json
with open('reports/evaluation_results.json', encoding='utf-8') as f:
    data = json.load(f)

rows = []
for e in data['entries']:
    r1 = e['rouge_scores']['rouge1_f1']
    g = e.get('geval_result')
    if g:
        rows.append((e['golden']['id'], r1, g['score_promedio'], g['score_promedio']/5 - r1))

if not rows:
    print('No hay resultados G-Eval. Ejecuta sin --skip-geval para analizar divergencias semánticas.')
else:
    for row in sorted(rows, key=lambda x: abs(x[3]), reverse=True)[:5]:
        print(f'ID={row[0]} ROUGE1={row[1]:.4f} GEVAL={row[2]:.2f} DIV={row[3]:.4f}')
PY
```

**✅ Validación del paso:**  
Si existe G-Eval, deben aparecer los IDs con mayor divergencia.

**📌 Resultado esperado:**  
Identificas casos prioritarios para revisión humana.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 9 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%209%20del%20Laboratorio%2010.%20Analic%C3%A9%20divergencias%20entre%20ROUGE%2C%20BLEU%20y%20G-Eval%20para%20detectar%20par%C3%A1frasis%20correctas%2C%20errores%20factuales%2C%20respuestas%20incompletas%20o%20limitaciones%20de%20m%C3%A9tricas.)

---

# 🧩 Tarea 10. Realizar revisión humana y cerrar evidencias

## 🎯 Objetivo de la tarea

Complementar métricas automáticas con criterio humano y confirmar que los entregables finales están completos y seguros.

---

## 🛠️ Pasos

### ✅ Paso 1. Revisa los peores casos

**📝 Descripción del paso:**  
Vas a abrir `reports/evaluation_report.md` y revisar la sección `Top 3 peores respuestas`. El objetivo es determinar si los peores casos realmente son incorrectos o si fueron penalizados por redacción diferente.

**⚙️ Contenido del paso:**

```bash
code reports/evaluation_report.md
```

Busca:

```text
## 5. Top 3 peores respuestas
```

**✅ Validación del paso:**  
Completa en tus notas una tabla como esta:

| ID | Problema detectado | ¿La respuesta es factualmente correcta? | Acción recomendada |
|---|---|---|---|
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

**📌 Resultado esperado:**  
Tienes una interpretación humana de los casos débiles.

---

### ✅ Paso 2. Revisa casos divergentes

**📝 Descripción del paso:**  
Si ejecutaste G-Eval, vas a revisar los casos donde ROUGE/BLEU y G-Eval no coinciden. Esto ayuda a distinguir entre paráfrasis correcta, error factual, respuesta incompleta o limitación de una métrica.

**⚙️ Contenido del paso:**

```text
## 7. Casos de divergencia
```

Clasifica cada caso:

| ID | ROUGE | G-Eval | Interpretación | Acción |
|---|---:|---:|---|---|
|  |  |  | Parafraseo correcto / error factual / irrelevante |  |

**✅ Validación del paso:**  
Al menos un caso divergente queda clasificado.

**📌 Resultado esperado:**  
La evaluación automática queda complementada con criterio humano.

---

### ✅ Paso 3. Valida entregables

**📝 Descripción del paso:**  
Vas a ejecutar una revisión rápida para confirmar que los archivos principales existen y no están vacíos. Esta validación ayuda a preparar la entrega final.

**⚙️ Contenido del paso:**

```bash
python - << 'PY'
from pathlib import Path
files = [
    'golden_dataset.json',
    'fixtures_responses.json',
    'chatbot_evaluator.py',
    'reports/evaluation_report.md',
    'reports/evaluation_report.html',
    'reports/evaluation_results.json',
]
for f in files:
    p = Path(f)
    print(('✅' if p.exists() else '❌'), f, p.stat().st_size if p.exists() else '')
PY
```

**📌 Resultado esperado:**  
Todos los archivos obligatorios existen y tienen contenido.

---

### ✅ Paso 4. Entrega evidencias seguras

**📝 Descripción del paso:**  
Vas a separar archivos seguros de entrega y archivos que no deben compartirse. No entregues `.env`, `.venv/` ni cache si contiene resultados de API que no quieras compartir.

**⚙️ Contenido del paso:**

Puedes entregar:

```text
golden_dataset.json
fixtures_responses.json
chatbot_evaluator.py
requirements.txt
reports/evaluation_report.md
reports/evaluation_report.html
reports/evaluation_results.json
```

No entregues:

```text
.env
.venv/
cache/ si contiene resultados privados o costosos
```

**✅ Validación del paso:**

```bash
grep -r "sk-" . --include="*.py" --include="*.json" --include="*.md" 2>/dev/null   && echo "⚠️ Posibles claves encontradas"   || echo "✅ No se encontraron claves en archivos entregables"
```

**📌 Resultado esperado:**  
Tu entrega no contiene credenciales ni información sensible.

---

## 💬 Prompt de apoyo para explicar lo realizado

[Explicar la Tarea 10 en ChatGPT](https://chatgpt.com/?q=Expl%C3%ADcame%20qu%C3%A9%20hice%20en%20la%20Tarea%2010%20del%20Laboratorio%2010.%20Realic%C3%A9%20revisi%C3%B3n%20humana%20de%20casos%20extremos%2C%20valid%C3%A9%20entregables%20y%20cerr%C3%A9%20evidencias%20seguras%20del%20laboratorio%20de%20evaluaci%C3%B3n%20de%20fidelidad.)

---

# 🏁 Resultado final esperado del laboratorio

Al finalizar la práctica, debes contar con:

1. Proyecto local creado en Windows.
2. Entorno virtual Python funcional.
3. Dependencias instaladas correctamente.
4. Archivo `.env` protegido por `.gitignore`.
5. Golden Dataset `golden_dataset.json` con 20 entradas.
6. Fixtures locales `fixtures_responses.json` con 20 respuestas.
7. Script principal `chatbot_evaluator.py`.
8. Carpeta `reports/` con reportes Markdown, HTML y JSON.
9. Carpeta `cache/` para resultados generados y G-Eval opcional.
10. Métricas ROUGE-1, ROUGE-2, ROUGE-L y BLEU calculadas.
11. Ruta base sin API validada.
12. Ruta opcional con G-Eval documentada.
13. Casos de divergencia identificados si se ejecutó G-Eval.
14. Checklist de revisión humana aplicado.
15. Evidencias seguras listas para entregar.

---

# ✅ Validación integral del laboratorio

Ejecuta esta validación después de completar el laboratorio base:

```bash
cat > validation_test.py << 'PY'
import json
from pathlib import Path

print('=' * 70)
print('VALIDACIÓN INTEGRAL — Laboratorio 10')
print('=' * 70)

required = [
    'golden_dataset.json',
    'fixtures_responses.json',
    'chatbot_evaluator.py',
    'reports/evaluation_report.md',
    'reports/evaluation_report.html',
    'reports/evaluation_results.json',
]

failed = 0

for item in required:
    path = Path(item)
    if path.exists() and path.stat().st_size > 0:
        print(f'✅ {item}')
    else:
        print(f'❌ {item}')
        failed += 1

if failed:
    print(f'\n❌ Validación fallida: faltan {failed} archivo(s) requerido(s).')
    raise SystemExit(1)

with open('reports/evaluation_results.json', encoding='utf-8') as f:
    data = json.load(f)

entries = data['entries']

assert len(entries) == 20, f'Se esperaban 20 entradas, hay {len(entries)}'

rouge1 = [e['rouge_scores']['rouge1_f1'] for e in entries]
bleu = [e['bleu_score'] for e in entries]

assert sum(rouge1) / len(rouge1) > 0, 'ROUGE-1 promedio debe ser > 0'
assert sum(bleu) / len(bleu) >= 0, 'BLEU debe ser >= 0'

print('\nResumen:')
print(f'Entradas: {len(entries)}')
print(f'ROUGE-1 promedio: {sum(rouge1) / len(rouge1):.4f}')
print(f'BLEU promedio: {sum(bleu) / len(bleu):.4f}')
print('\n🎉 Validación completada')
PY
```
```bash
python validation_test.py
```

**📌 Resultado esperado:**  
La validación confirma que los archivos obligatorios existen, que hay 20 entradas evaluadas y que las métricas fueron calculadas.

---

# 📊 Criterios de evaluación sugeridos

| Criterio | Ponderación |
|---|---:|
| Preparación correcta del entorno local | 10% |
| Configuración segura de `.env` y `.gitignore` | 10% |
| Golden Dataset válido | 10% |
| Fixtures locales sin API | 10% |
| Script `chatbot_evaluator.py` funcional | 20% |
| Métricas ROUGE/BLEU calculadas | 15% |
| Reportes Markdown/HTML/JSON generados | 10% |
| G-Eval opcional con cache documentado | 10% |
| Revisión humana y entrega segura | 5% |
| Total | 100% |

---

# ⚠️ Errores comunes que debes evitar

1. Ejecutar G-Eval antes de probar la ruta sin API.
2. Subir `.env` a un repositorio.
3. Usar datos reales de clientes en el Golden Dataset.
4. Confundir ROUGE alto con respuesta factualmente correcta.
5. Confundir BLEU bajo con respuesta necesariamente incorrecta.
6. Borrar `reports/` y pensar que la evaluación no funcionó.
7. Leer resultados antiguos desde cache sin darte cuenta.
8. Ejecutar `--refresh-cache` muchas veces y consumir API innecesariamente.
9. Usar un modelo juez sin documentar cuál fue.
10. Omitir la revisión humana de los casos extremos.

---

# 🧯 Solución de problemas

## Problema 1. `ModuleNotFoundError`

**Causa probable:**  
El entorno virtual no está activo o no instalaste dependencias.

**Solución:**

```bash
source .venv/Scripts/activate
pip install -r requirements.txt
python -c "import rouge_score, nltk, pandas, pydantic; print('✅ Dependencias OK')"
```

---

## Problema 2. `DataFrame.to_markdown()` falla

**Causa probable:**  
Falta la librería `tabulate`.

**Solución:**

```bash
pip install 'tabulate>=0.9,<1'
python chatbot_evaluator.py --use-fixtures --skip-geval
```

---

## Problema 3. NLTK solicita `punkt_tab`

**Causa probable:**  
Algunas versiones recientes de NLTK separan recursos adicionales para tokenización.

**Solución:**

```bash
python - << 'PY'
import nltk
nltk.download('punkt')
nltk.download('punkt_tab')
print('✅ Recursos NLTK listos')
PY
```

---

## Problema 4. G-Eval falla por API key

**Causa probable:**  
`.env` no tiene API key, tiene un valor incorrecto o el entorno no cargó variables.

**Solución:**

```bash
cat .env
python - << 'PY'
from dotenv import load_dotenv
import os
load_dotenv()
key = os.getenv('OPENAI_API_KEY')
print('Key configurada:', bool(key))
PY
```

---

## Problema 5. Resultados no cambian aunque modificaste el código

**Causa probable:**  
Estás leyendo resultados desde `cache/`.

**Solución:**

```bash
python chatbot_evaluator.py --use-fixtures --refresh-cache
rm -f cache/*.json
```

---

# 🧹 Limpieza del entorno

Ejecuta estos comandos si deseas limpiar archivos generados:

```bash
rm -rf reports/
rm -rf cache/
rm -f validation_test.py
```

Para desactivar el entorno virtual:

```bash
deactivate
```

Para eliminar el entorno virtual completo:

```bash
rm -rf .venv/
```

Antes de compartir el proyecto, valida que no haya claves en archivos de entrega:

```bash
grep -r "sk-" . --include="*.py" --include="*.json" --include="*.md" 2>/dev/null   && echo "⚠️ Posibles claves encontradas"   || echo "✅ No se encontraron claves en archivos entregables"
```

---

# 📚 Resumen conceptual

En este laboratorio construiste un framework completo para evaluar la fidelidad de respuestas de un chatbot contra un Golden Dataset. La arquitectura final separa responsabilidades:

| Capa | Archivo / técnica | Función |
|---|---|---|
| Dataset de referencia | `golden_dataset.json` | Define preguntas y respuestas esperadas |
| Respuestas simuladas | `fixtures_responses.json` | Permite evaluar sin API |
| Métricas léxicas | ROUGE / BLEU | Miden similitud textual |
| Evaluación semántica | G-Eval opcional | Evalúa fidelidad, relevancia, coherencia y fluencia |
| Persistencia operativa | `cache/` | Reduce costo y llamadas repetidas |
| Evidencia | `reports/` | Guarda resultados Markdown, HTML y JSON |

La clave del diseño está en no depender de una sola métrica. ROUGE y BLEU son útiles como filtros económicos, pero pueden penalizar paráfrasis correctas. G-Eval aporta una evaluación más semántica, pero también debe revisarse críticamente. Por eso, una práctica profesional combina métricas automáticas, análisis de divergencias y revisión humana.
