# Construcción de un asistente probabilístico para corregir y autocompletar mensajes bancarios

## Metadatos

| Campo | Valor |
|---|---|
| Duración | 105 minutos |
| Complejidad | Media |
| Nivel de Bloom | Crear |

## Descripción general

En esta práctica construirá una línea base reproducible de un asistente probabilístico para mensajes bancarios sintéticos. El asistente detectará errores ortográficos simples mediante frecuencia de palabras y distancia mínima de edición, y sugerirá la siguiente palabra mediante modelos unigramas y bigramas con suavizado aditivo.

El sistema se ejecutará completamente en local, sin API, base de datos, contenedor ni servidor externo. Las sugerencias serán demostrativas: no deben utilizarse para comunicar estados operativos, importes, plazos o decisiones bancarias reales.

## Objetivos de aprendizaje

Al finalizar la práctica, podrá:

- [ ] Preparar, normalizar, tokenizar y separar reproduciblemente un corpus sintético de mensajes bancarios.
- [ ] Construir un corrector basado en candidatos a una edición y probabilidades unigramas.
- [ ] Implementar modelos unigramas y bigramas con marcadores `<s>`, `</s>`, suavizado de Laplace y `<UNK>`.
- [ ] Calcular probabilidad de secuencias, cobertura de vocabulario, exactitud de sugerencias y perplejidad.
- [ ] Integrar corrección y autocompletado en la función `asistente_bancario(mensaje, top_k=3)` y documentar limitaciones de uso.

## Requisitos previos

### Conocimientos

- Frecuencias y probabilidades de palabras.
- Probabilidad condicional.
- Distancia mínima de edición.
- Modelos de lenguaje n-grama.
- Suavizado aditivo.
- Manejo de palabras fuera del vocabulario.
- Perplejidad.
- Python básico: listas, diccionarios, funciones, comprensión de listas y arreglos.
- Uso elemental de JupyterLab.

### Acceso y condiciones

- Computadora de 64 bits con al menos 2 núcleos a 2.0 GHz.
- Memoria RAM mínima de 8 GB; se recomiendan 16 GB.
- Al menos 5 GB libres en disco.
- Resolución mínima de 1366x768.
- Conexión a Internet solo para la instalación inicial.
- Python 3.12.1 y Git 2.43.0 instalados o disponibles en su organización.
- Permisos de escritura en el directorio personal del usuario.

> **Nota de seguridad:** esta práctica usa exclusivamente mensajes sintéticos. No copie mensajes reales de clientes, números de cuenta, tarjetas, identificadores, contraseñas, códigos OTP, importes ni información operativa en el corpus, notebook, resultados o repositorio Git.

## Entorno de la práctica

### Software objetivo

| Componente | Versión objetivo |
|---|---:|
| Python | 3.12.1 |
| pip | 23.3.2 |
| venv | 3.12.1 |
| JupyterLab | 4.1.0 |
| notebook | 7.1.0 |
| ipykernel | 6.29.0 |
| NumPy | 1.26.4 |
| pandas | 2.2.0 |
| matplotlib | 3.8.2 |
| scikit-learn | 1.4.0 |
| Git | 2.43.0 |

### Parámetros no aplicables

| Parámetro | Estado |
|---|---|
| Base de datos | No aplicable |
| Puerto de base de datos | No aplicable |
| Contenedor Docker | No aplicable |
| Servidor externo | No aplicable |
| API de terceros | No aplicable |
| Acceso remoto a JupyterLab | Prohibido |

### Preparación inicial del proyecto

Todos los comandos y notebooks deben ejecutarse desde el directorio raíz obligatorio.

#### Linux o macOS

```bash
mkdir -p ~/taller-pln-probabilistico
cd ~/taller-pln-probabilistico

mkdir -p data/raw data/processed notebooks src results tests docs

python3.12 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip==23.3.2
python -m pip install \
  jupyterlab==4.1.0 \
  notebook==7.1.0 \
  ipykernel==6.29.0 \
  numpy==1.26.4 \
  pandas==2.2.0 \
  matplotlib==3.8.2 \
  scikit-learn==1.4.0

python -m ipykernel install --user \
  --name nlp-bancario-py312 \
  --display-name "Python 3.12 (nlp-bancario-py312)"

git init
```

#### Windows PowerShell

```powershell
mkdir $env:USERPROFILE\taller-pln-probabilistico
cd $env:USERPROFILE\taller-pln-probabilistico

mkdir data, data\raw, data\processed, notebooks, src, results, tests, docs

py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip==23.3.2
python -m pip install `
  jupyterlab==4.1.0 `
  notebook==7.1.0 `
  ipykernel==6.29.0 `
  numpy==1.26.4 `
  pandas==2.2.0 `
  matplotlib==3.8.2 `
  scikit-learn==1.4.0

python -m ipykernel install --user `
  --name nlp-bancario-py312 `
  --display-name "Python 3.12 (nlp-bancario-py312)"

git init
```

Si PowerShell bloquea la activación del entorno, ejecute una vez:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Cree además un archivo `.gitignore` desde el directorio raíz:

```bash
cat > .gitignore <<'EOF'
.venv/
__pycache__/
.ipynb_checkpoints/
*.pyc
results/*.png
EOF
```

En Windows PowerShell, puede crear el mismo archivo con:

```powershell
@"
.venv/
__pycache__/
.ipynb_checkpoints/
*.pyc
results/*.png
"@ | Set-Content .gitignore
```

## Procedimiento paso a paso

### Paso 1. Verificar la estructura y crear el notebook principal

**Objetivo:** confirmar que el proyecto cumple la estructura obligatoria y crear el notebook principal de la práctica.

**Instrucciones:**

1. Desde la raíz `~/taller-pln-probabilistico` o `%USERPROFILE%\taller-pln-probabilistico`, verifique el intérprete activo:

   ```bash
   python --version
   ```

2. Verifique que el entorno virtual esté activo. La consola debe mostrar normalmente el prefijo `(.venv)`.

3. Cree el notebook obligatorio:

   ```bash
   jupyter lab --ip=127.0.0.1 --port=8888 --no-browser
   ```

4. Abra en el navegador la URL local:

   ```text
   http://127.0.0.1:8888/lab
   ```

5. En JupyterLab, cree un notebook Python dentro de `notebooks` con el nombre exacto:

   ```text
   01_asistente_probabilistico_bancario.ipynb
   ```

6. Seleccione el kernel:

   ```text
   Python 3.12 (nlp-bancario-py312)
   ```

7. En la primera celda Markdown del notebook, escriba:

   ```markdown
   # Asistente probabilístico bancario

   Corpus sintético. No contiene ni debe contener información bancaria real.
   ```

8. Guarde el notebook.

**Resultado esperado:**

- JupyterLab se ejecuta solamente en `127.0.0.1:8888`.
- Existe el archivo `notebooks/01_asistente_probabilistico_bancario.ipynb`.
- El kernel seleccionado es `nlp-bancario-py312`.

**Verificación:**

Ejecute esta celda en el notebook:

```python
from pathlib import Path
import sys

ROOT = Path.cwd().resolve()

if ROOT.name != "taller-pln-probabilistico":
    raise RuntimeError(
        f"Ejecute JupyterLab desde la raíz del proyecto. Directorio actual: {ROOT}"
    )

for ruta in [
    "data/raw",
    "data/processed",
    "notebooks",
    "src",
    "results",
    "tests",
    "docs",
]:
    assert (ROOT / ruta).exists(), f"Falta el directorio obligatorio: {ruta}"

print("Python:", sys.version.split()[0])
print("Raíz del proyecto:", ROOT)
print("Estructura obligatoria verificada.")
```

La salida debe indicar Python 3.12.x y la ruta raíz del proyecto.

---

### Paso 2. Crear y cargar el corpus sintético

**Objetivo:** crear un corpus local, sintético y no sensible de mensajes bancarios para entrenar y evaluar el asistente.

**Instrucciones:**

1. Inserte una nueva celda de código en el notebook.
2. Ejecute el siguiente código para crear `data/raw/mensajes_bancarios.csv`.

```python
import pandas as pd
from pathlib import Path

RAW_DIR = ROOT / "data" / "raw"
RAW_DIR.mkdir(parents=True, exist_ok=True)

mensajes = [
    "Su transferencia fue recibida correctamente.",
    "Su transferencia fue procesada correctamente.",
    "La transferencia está en proceso de validación.",
    "La transferencia fue rechazada por información incompleta.",
    "Puede consultar el estado de su transferencia.",
    "Hemos recibido su solicitud de transferencia.",
    "La solicitud fue registrada correctamente.",
    "El saldo disponible es visible en la aplicación.",
    "Puede consultar su saldo disponible.",
    "El saldo de su cuenta fue actualizado.",
    "Su tarjeta fue activada correctamente.",
    "La tarjeta está disponible para compras.",
    "Puede bloquear su tarjeta desde la aplicación.",
    "La solicitud de tarjeta fue recibida.",
    "El pago fue recibido correctamente.",
    "El pago está en proceso de confirmación.",
    "Puede consultar el estado de su pago.",
    "El pago fue aplicado a su cuenta.",
    "El retiro fue procesado correctamente.",
    "Puede consultar los límites de retiro.",
    "El retiro está sujeto a validación.",
    "No comparta su contraseña con otras personas.",
    "No comparta códigos de seguridad.",
    "Verifique la información antes de confirmar la operación.",
    "La operación requiere una validación adicional.",
    "Nuestro equipo de atención está disponible.",
    "Puede contactar al equipo de atención al cliente.",
    "Hemos recibido su mensaje correctamente.",
    "La consulta fue enviada al equipo responsable.",
    "Recibirá una actualización cuando finalice el proceso.",
]

# Se duplican variantes de forma controlada para obtener frecuencias suficientes.
corpus = mensajes + [
    mensaje.replace(".", " hoy.") if "correctamente" in mensaje else mensaje
    for mensaje in mensajes
]

df_corpus = pd.DataFrame(
    {
        "id_mensaje": range(1, len(corpus) + 1),
        "mensaje": corpus,
        "origen": "sintetico",
    }
)

ruta_corpus = RAW_DIR / "mensajes_bancarios.csv"
df_corpus.to_csv(ruta_corpus, index=False, encoding="utf-8")

print(f"Corpus creado: {ruta_corpus}")
print(f"Mensajes: {len(df_corpus)}")
display(df_corpus.head())
```

3. Cargue el archivo desde disco, en lugar de continuar trabajando con el objeto creado en memoria:

```python
df = pd.read_csv(
    ROOT / "data" / "raw" / "mensajes_bancarios.csv",
    encoding="utf-8"
)

assert set(["id_mensaje", "mensaje", "origen"]).issubset(df.columns)
assert df["origen"].eq("sintetico").all()
assert df["mensaje"].notna().all()

print("Forma del corpus:", df.shape)
display(df.sample(5, random_state=42))
```

4. Explore longitudes y duplicados:

```python
df["longitud_caracteres"] = df["mensaje"].str.len()
df["duplicado"] = df["mensaje"].duplicated(keep=False)

display(df[["longitud_caracteres", "duplicado"]].describe())
print("Mensajes duplicados controlados:", int(df["duplicado"].sum()))
```

**Resultado esperado:**

- Se crea `data/raw/mensajes_bancarios.csv`.
- El corpus contiene al menos 50 mensajes sintéticos.
- Las columnas `id_mensaje`, `mensaje` y `origen` están presentes.
- Todos los registros tienen `origen = sintetico`.

**Verificación:**

Ejecute:

```python
assert len(df) >= 50
assert not df["mensaje"].str.contains(
    r"\b\d{12,19}\b", regex=True
).any(), "Posible identificador numérico sensible detectado."

print("Verificación de corpus sintético completada.")
```

> El detector anterior es deliberadamente simple. En un entorno real, los filtros de seguridad deben ser mucho más completos y estar alineados con las políticas de protección de datos de la organización.

---

### Paso 3. Normalizar, tokenizar y separar entrenamiento/prueba

**Objetivo:** preparar el texto preservando caracteres relevantes del español y signos básicos, y crear una separación reproducible de entrenamiento y prueba.

**Instrucciones:**

1. Añada una celda con las funciones de normalización y tokenización.

```python
import re
import unicodedata
from sklearn.model_selection import train_test_split

TOKEN_RE = re.compile(
    r"\d+(?:[.,]\d+)?|[a-záéíóúüñ]+|[.,;:¿?!]",
    flags=re.IGNORECASE
)

SIGNOS = {".", ",", ";", ":", "¿", "?", "!", "…"}

def normalizar_texto(texto: str) -> str:
    """Normaliza Unicode, minúsculas y espacios sin eliminar acentos."""
    texto = unicodedata.normalize("NFC", str(texto).lower())
    texto = re.sub(r"\s+", " ", texto).strip()
    return texto

def tokenizar(texto: str) -> list[str]:
    """Conserva palabras españolas, números y signos definidos."""
    return TOKEN_RE.findall(normalizar_texto(texto))

def tokens_lexicos(tokens: list[str]) -> list[str]:
    """Devuelve solo palabras alfabéticas para el modelo lingüístico."""
    return [token for token in tokens if token.isalpha()]
```

2. Aplique las funciones al corpus:

```python
df["texto_normalizado"] = df["mensaje"].map(normalizar_texto)
df["tokens"] = df["texto_normalizado"].map(tokenizar)
df["tokens_lexicos"] = df["tokens"].map(tokens_lexicos)

display(df[["mensaje", "texto_normalizado", "tokens"]].head(8))
```

3. Separe los datos con una semilla fija:

```python
train_df, test_df = train_test_split(
    df,
    test_size=0.25,
    random_state=42,
    shuffle=True,
)

train_df = train_df.copy()
test_df = test_df.copy()

print("Mensajes de entrenamiento:", len(train_df))
print("Mensajes de prueba:", len(test_df))
```

4. Guarde las particiones procesadas:

```python
PROCESSED_DIR = ROOT / "data" / "processed"
PROCESSED_DIR.mkdir(parents=True, exist_ok=True)

train_df.to_csv(PROCESSED_DIR / "train.csv", index=False, encoding="utf-8")
test_df.to_csv(PROCESSED_DIR / "test.csv", index=False, encoding="utf-8")
```

5. Registre la convención usada en `docs/convenciones.md`:

```python
convenciones = """# Convenciones de la práctica

- Corpus: exclusivamente sintético y local.
- Normalización: Unicode NFC, minúsculas y espacios simples.
- Tokenización: palabras con caracteres españoles, números y signos básicos.
- Signos: se conservan durante tokenización; el modelo bigrama usa solo tokens léxicos.
- Inicio de oración: `<s>`.
- Fin de oración: `</s>`.
- Palabra desconocida: `<UNK>`.
- Umbral de rareza: frecuencia menor o igual a 1 en entrenamiento.
- Suavizado: aditivo de Laplace con k=1.0.
- Separación: 75 % entrenamiento y 25 % prueba con random_state=42.
"""

(ROOT / "docs" / "convenciones.md").write_text(convenciones, encoding="utf-8")
print("Convenciones documentadas.")
```

**Resultado esperado:**

- Los textos están en minúsculas.
- Los acentos y la `ñ` se preservan.
- Los signos se representan como tokens independientes.
- La separación se puede reproducir usando `random_state=42`.
- Existen los archivos `data/processed/train.csv`, `data/processed/test.csv` y `docs/convenciones.md`.

**Verificación:**

```python
ejemplo = "¿Su transferencia está procesada correctamente?"
tokens_ejemplo = tokenizar(ejemplo)

print(tokens_ejemplo)

assert "transferencia" in tokens_ejemplo
assert "está" in tokens_ejemplo
assert "¿" in tokens_ejemplo
assert "?" in tokens_ejemplo
assert len(train_df) + len(test_df) == len(df)

print("Normalización, tokenización y partición verificadas.")
```

---

### Paso 4. Construir vocabulario, frecuencias y corrector ortográfico

**Objetivo:** implementar un corrector que genere candidatos mediante eliminación, inserción, sustitución y transposición, y los ordene por probabilidad unigram.

**Instrucciones:**

1. Cree frecuencias a partir de los tokens léxicos del conjunto de entrenamiento:

```python
from collections import Counter

tokens_train = [
    token
    for tokens in train_df["tokens_lexicos"]
    for token in tokens
]

frecuencias_palabras = Counter(tokens_train)
vocabulario_corrector = set(frecuencias_palabras)
total_tokens_train = sum(frecuencias_palabras.values())

probabilidades_unigrama = {
    palabra: frecuencia / total_tokens_train
    for palabra, frecuencia in frecuencias_palabras.items()
}

print("Tamaño del vocabulario del corrector:", len(vocabulario_corrector))
print("Tokens léxicos de entrenamiento:", total_tokens_train)
print("Palabras más frecuentes:")
print(frecuencias_palabras.most_common(10))
```

2. Implemente la distancia de Levenshtein con transposición. La transposición permite tratar casos como `trasnferencia` → `transferencia`.

```python
def distancia_damerau_levenshtein(a: str, b: str) -> int:
    """Distancia mínima con inserción, eliminación, sustitución y transposición."""
    filas = len(a) + 1
    columnas = len(b) + 1
    matriz = [[0] * columnas for _ in range(filas)]

    for i in range(filas):
        matriz[i][0] = i

    for j in range(columnas):
        matriz[0][j] = j

    for i in range(1, filas):
        for j in range(1, columnas):
            costo = 0 if a[i - 1] == b[j - 1] else 1

            matriz[i][j] = min(
                matriz[i - 1][j] + 1,
                matriz[i][j - 1] + 1,
                matriz[i - 1][j - 1] + costo,
            )

            if (
                i > 1
                and j > 1
                and a[i - 1] == b[j - 2]
                and a[i - 2] == b[j - 1]
            ):
                matriz[i][j] = min(
                    matriz[i][j],
                    matriz[i - 2][j - 2] + 1,
                )

    return matriz[-1][-1]
```

3. Implemente las operaciones de edición a una distancia:

```python
ALFABETO_ESP = "abcdefghijklmnopqrstuvwxyzáéíóúüñ"

def ediciones_uno(palabra: str) -> set[str]:
    """Genera candidatos por eliminación, inserción, sustitución y transposición."""
    divisiones = [(palabra[:i], palabra[i:]) for i in range(len(palabra) + 1)]

    eliminaciones = {
        izquierda + derecha[1:]
        for izquierda, derecha in divisiones
        if derecha
    }

    transposiciones = {
        izquierda + derecha[1] + derecha[0] + derecha[2:]
        for izquierda, derecha in divisiones
        if len(derecha) > 1
    }

    sustituciones = {
        izquierda + letra + derecha[1:]
        for izquierda, derecha in divisiones
        if derecha
        for letra in ALFABETO_ESP
    }

    inserciones = {
        izquierda + letra + derecha
        for izquierda, derecha in divisiones
        for letra in ALFABETO_ESP
    }

    return eliminaciones | transposiciones | sustituciones | inserciones
```

4. Cree funciones para obtener candidatos conocidos y proponer una corrección:

```python
def es_token_protegido(token: str) -> bool:
    """Evita modificar números, códigos o tokens no alfabéticos."""
    return (not token.isalpha()) or any(caracter.isdigit() for caracter in token)

def candidatos_conocidos(palabra: str) -> set[str]:
    """Filtra candidatos a una edición que existan en el vocabulario."""
    if palabra in vocabulario_corrector:
        return {palabra}

    return {
        candidato
        for candidato in ediciones_uno(palabra)
        if candidato in vocabulario_corrector
    }

def sugerir_correccion(palabra: str, top_k: int = 3) -> list[tuple[str, float]]:
    """Ordena candidatos por probabilidad unigram y luego alfabéticamente."""
    palabra = normalizar_texto(palabra)

    if es_token_protegido(palabra):
        return [(palabra, 1.0)]

    candidatos = candidatos_conocidos(palabra)

    if not candidatos:
        return [(palabra, 0.0)]

    ordenados = sorted(
        candidatos,
        key=lambda candidato: (
            -probabilidades_unigrama[candidato],
            distancia_damerau_levenshtein(palabra, candidato),
            candidato,
        ),
    )

    return [
        (candidato, probabilidades_unigrama[candidato])
        for candidato in ordenados[:top_k]
    ]

def corregir_token(token: str) -> str:
    """Propone el candidato principal; conserva tokens sin candidato."""
    return sugerir_correccion(token, top_k=1)[0][0]
```

5. Pruebe las erratas solicitadas:

```python
for errata in ["trasnferencia", "tarjetaa", "disponibel", "prosesada"]:
    print(f"{errata:15} -> {sugerir_correccion(errata, top_k=3)}")
```

**Resultado esperado:**

- `trasnferencia` debe proponer `transferencia` como primer candidato si la palabra aparece en entrenamiento.
- `tarjetaa` debe proponer `tarjeta`.
- `disponibel` debe proponer `disponible`.
- Los candidatos se ordenan usando frecuencia/probabilidad unigram, no solo similitud ortográfica.

**Verificación:**

```python
assert distancia_damerau_levenshtein("trasnferencia", "transferencia") == 1
assert distancia_damerau_levenshtein("tarjetaa", "tarjeta") == 1
assert sugerir_correccion("123456")[0][0] == "123456"

print("Corrector básico verificado.")
```

> **Limitación importante:** el corrector no comprende la intención. Si dos palabras válidas son cercanas, la frecuencia puede favorecer una corrección incorrecta. En producción, un candidato debe combinarse con contexto, reglas de negocio y revisión humana.

---

### Paso 5. Construir el modelo bigrama con `<UNK>` y suavizado

**Objetivo:** entrenar un modelo bigrama capaz de asignar probabilidades a secuencias y manejar palabras raras o no observadas.

**Instrucciones:**

1. Defina el vocabulario del modelo de lenguaje. Las palabras con frecuencia menor o igual a 1 se reemplazarán por `<UNK>`:

```python
UMBRAL_RAREZA = 1
K_SUAVIZADO = 1.0

vocabulario_lm = {
    palabra
    for palabra, frecuencia in frecuencias_palabras.items()
    if frecuencia > UMBRAL_RAREZA
}
vocabulario_lm.add("<UNK>")

vocabulario_prediccion = vocabulario_lm | {"</s>"}

print("Vocabulario LM:", len(vocabulario_lm))
print("Vocabulario de predicción:", len(vocabulario_prediccion))
```

2. Cree una función para reemplazar palabras raras o desconocidas:

```python
def mapear_unk(palabra: str) -> str:
    return palabra if palabra in vocabulario_lm else "<UNK>"

def preparar_secuencia_lm(tokens: list[str]) -> list[str]:
    """Convierte tokens léxicos en secuencia con marcadores de inicio y fin."""
    palabras = tokens_lexicos(tokens)
    palabras_mapeadas = [mapear_unk(palabra) for palabra in palabras]
    return ["<s>"] + palabras_mapeadas + ["</s>"]
```

3. Calcule conteos unigramas de contexto y bigramas:

```python
conteos_contexto = Counter()
conteos_bigramas = Counter()

for tokens in train_df["tokens"]:
    secuencia = preparar_secuencia_lm(tokens)

    for anterior, actual in zip(secuencia[:-1], secuencia[1:]):
        conteos_contexto[anterior] += 1
        conteos_bigramas[(anterior, actual)] += 1

print("Cantidad de contextos:", len(conteos_contexto))
print("Cantidad de bigramas observados:", len(conteos_bigramas))
print("Ejemplos de bigramas:", conteos_bigramas.most_common(10))
```

4. Implemente probabilidades con y sin suavizado:

```python
def probabilidad_bigrama(
    anterior: str,
    actual: str,
    k: float = K_SUAVIZADO,
) -> float:
    """Calcula P(actual | anterior) con suavizado aditivo."""
    anterior = mapear_unk(anterior) if anterior != "<s>" else "<s>"
    actual = mapear_unk(actual) if actual != "</s>" else "</s>"

    numerador = conteos_bigramas[(anterior, actual)] + k
    denominador = conteos_contexto[anterior] + k * len(vocabulario_prediccion)

    return numerador / denominador

def probabilidad_bigrama_sin_suavizado(anterior: str, actual: str) -> float:
    """Calcula P(actual | anterior) sin suavizado."""
    anterior = mapear_unk(anterior) if anterior != "<s>" else "<s>"
    actual = mapear_unk(actual) if actual != "</s>" else "</s>"

    denominador = conteos_contexto[anterior]
    if denominador == 0:
        return 0.0

    return conteos_bigramas[(anterior, actual)] / denominador
```

5. Compare un bigrama observado y uno no observado:

```python
ejemplos_bigramas = [
    ("su", "transferencia"),
    ("transferencia", "procesada"),
    ("transferencia", "astronómica"),
]

for anterior, actual in ejemplos_bigramas:
    print(
        f"P({actual} | {anterior}) sin suavizado = "
        f"{probabilidad_bigrama_sin_suavizado(anterior, actual):.6f}"
    )
    print(
        f"P({actual} | {anterior}) con Laplace = "
        f"{probabilidad_bigrama(anterior, actual):.6f}"
    )
    print()
```

**Resultado esperado:**

- El token `<UNK>` forma parte del vocabulario del modelo.
- Un bigrama no observado tiene probabilidad `0.0` sin suavizado.
- Con `k=1.0`, todo bigrama posible del vocabulario de predicción recibe una probabilidad positiva.
- Los marcadores `<s>` y `</s>` participan en la probabilidad de cada mensaje.

**Verificación:**

```python
assert "<UNK>" in vocabulario_lm
assert "</s>" in vocabulario_prediccion
assert probabilidad_bigrama("transferencia", "palabra_inexistente") > 0
assert probabilidad_bigrama_sin_suavizado(
    "transferencia", "palabra_inexistente"
) == 0.0

print("Modelo bigrama y suavizado verificados.")
```

---

### Paso 6. Calcular probabilidad, perplejidad y autocompletado

**Objetivo:** medir la plausibilidad de secuencias y sugerir las palabras más probables dado un contexto final.

**Instrucciones:**

1. Implemente la probabilidad de una secuencia usando logaritmos para evitar desbordamiento numérico:

```python
import math
import numpy as np

def log_probabilidad_mensaje(tokens: list[str], k: float = K_SUAVIZADO) -> float:
    secuencia = preparar_secuencia_lm(tokens)
    log_prob = 0.0

    for anterior, actual in zip(secuencia[:-1], secuencia[1:]):
        prob = probabilidad_bigrama(anterior, actual, k=k)
        log_prob += math.log(prob)

    return log_prob

def probabilidad_mensaje(tokens: list[str], k: float = K_SUAVIZADO) -> float:
    return math.exp(log_probabilidad_mensaje(tokens, k=k))

def perplejidad_mensaje(tokens: list[str], k: float = K_SUAVIZADO) -> float:
    secuencia = preparar_secuencia_lm(tokens)
    numero_predicciones = len(secuencia) - 1

    if numero_predicciones == 0:
        return float("inf")

    log_prob = log_probabilidad_mensaje(tokens, k=k)
    return math.exp(-log_prob / numero_predicciones)
```

2. Implemente la perplejidad sin suavizado para comparar el efecto de bigramas no observados:

```python
def perplejidad_sin_suavizado(tokens: list[str]) -> float:
    secuencia = preparar_secuencia_lm(tokens)
    probabilidades = [
        probabilidad_bigrama_sin_suavizado(anterior, actual)
        for anterior, actual in zip(secuencia[:-1], secuencia[1:])
    ]

    if any(prob == 0 for prob in probabilidades):
        return float("inf")

    log_prob = sum(math.log(prob) for prob in probabilidades)
    return math.exp(-log_prob / len(probabilidades))
```

3. Implemente el autocompletado. El modelo devuelve las palabras más probables después del último token léxico:

```python
def ultimo_token_lexico(texto: str) -> str:
    palabras = tokens_lexicos(tokenizar(texto))
    return palabras[-1] if palabras else "<s>"

def sugerir_siguiente_palabra(contexto: str, top_k: int = 3) -> list[tuple[str, float]]:
    ultimo = ultimo_token_lexico(contexto)
    ultimo = mapear_unk(ultimo) if ultimo != "<s>" else "<s>"

    candidatos = [
        palabra
        for palabra in vocabulario_prediccion
        if palabra not in {"<UNK>", "</s>"}
    ]

    ranking = sorted(
        (
            (palabra, probabilidad_bigrama(ultimo, palabra))
            for palabra in candidatos
        ),
        key=lambda elemento: (-elemento[1], elemento[0]),
    )

    return ranking[:top_k]
```

4. Pruebe el autocompletado:

```python
for contexto in [
    "su transferencia fue",
    "el saldo disponible",
    "puede consultar",
    "nuestro equipo de atención",
]:
    print(f"\nContexto: {contexto!r}")
    for palabra, probabilidad in sugerir_siguiente_palabra(contexto, top_k=3):
        print(f"  {palabra:15} {probabilidad:.4f}")
```

5. Evalúe cobertura de vocabulario y perplejidad sobre prueba:

```python
def cobertura_vocabulario(df_evaluacion: pd.DataFrame) -> float:
    tokens = [
        token
        for lista in df_evaluacion["tokens_lexicos"]
        for token in lista
    ]

    if not tokens:
        return 0.0

    conocidos = sum(token in vocabulario_lm for token in tokens)
    return conocidos / len(tokens)

resultados_perplejidad = test_df[["id_mensaje", "mensaje", "tokens"]].copy()
resultados_perplejidad["perplejidad_laplace"] = resultados_perplejidad["tokens"].map(
    perplejidad_mensaje
)
resultados_perplejidad["perplejidad_sin_suavizado"] = resultados_perplejidad["tokens"].map(
    perplejidad_sin_suavizado
)

cobertura = cobertura_vocabulario(test_df)

print(f"Cobertura de vocabulario en prueba: {cobertura:.2%}")
display(resultados_perplejidad.head())
```

**Resultado esperado:**

- Cada contexto genera tres sugerencias ordenadas por probabilidad descendente.
- La perplejidad con Laplace es finita para todos los mensajes no vacíos.
- La perplejidad sin suavizado puede ser infinita si aparece al menos un bigrama no observado.
- La cobertura está entre 0 y 1.

**Verificación:**

```python
assert 0.0 <= cobertura <= 1.0
assert np.isfinite(resultados_perplejidad["perplejidad_laplace"]).all()

sugerencias = sugerir_siguiente_palabra("su transferencia fue", top_k=3)
assert len(sugerencias) == 3
assert all(probabilidad > 0 for _, probabilidad in sugerencias)

print("Probabilidades, perplejidad y autocompletado verificados.")
```

---

### Paso 7. Integrar el asistente y generar pruebas controladas

**Objetivo:** integrar corrección y autocompletado en una única función y evaluar resultados con al menos diez mensajes alterados.

**Instrucciones:**

1. Añada una función para reconstruir texto conservando el espaciado básico de signos:

```python
def reconstruir_texto(tokens: list[str]) -> str:
    texto = ""

    for token in tokens:
        if not texto:
            texto = token
        elif token in {".", ",", ";", ":", "?", "!"}:
            texto += token
        elif token == "¿":
            texto += " " + token
        else:
            texto += " " + token

    return texto.strip()
```

2. Implemente la función integrada requerida:

```python
def asistente_bancario(mensaje: str, top_k: int = 3) -> dict:
    """
    Propone correcciones ortográficas simples y sugerencias de continuación.

    No valida hechos bancarios, importes, plazos ni estados operativos.
    """
    tokens_originales = tokenizar(mensaje)
    tokens_corregidos = [
        corregir_token(token) if token.isalpha() else token
        for token in tokens_originales
    ]

    mensaje_corregido = reconstruir_texto(tokens_corregidos)
    sugerencias = sugerir_siguiente_palabra(mensaje_corregido, top_k=top_k)

    return {
        "mensaje_original": mensaje,
        "mensaje_corregido": mensaje_corregido,
        "sugerencias_siguiente_palabra": [palabra for palabra, _ in sugerencias],
        "probabilidades_siguiente_palabra": [prob for _, prob in sugerencias],
        "requiere_revision_humana": True,
        "advertencia": (
            "Sugerencia lingüística basada en corpus sintético; "
            "no confirma estados, importes, fechas ni condiciones bancarias."
        ),
    }
```

3. Cree una tabla de pruebas con diez mensajes alterados:

```python
pruebas_controladas = [
    {
        "mensaje": "Su trasnferencia fue procesada correctamente.",
        "palabra_esperada": "transferencia",
    },
    {
        "mensaje": "La tarjetaa está disponible para compras.",
        "palabra_esperada": "tarjeta",
    },
    {
        "mensaje": "El saldo disponibel es visible.",
        "palabra_esperada": "disponible",
    },
    {
        "mensaje": "El pago fue resibido correctamente.",
        "palabra_esperada": "recibido",
    },
    {
        "mensaje": "Puede konsultar el estado de su pago.",
        "palabra_esperada": "consultar",
    },
    {
        "mensaje": "La solisitud fue registrada correctamente.",
        "palabra_esperada": "solicitud",
    },
    {
        "mensaje": "El retiro fue prosesado correctamente.",
        "palabra_esperada": "procesado",
    },
    {
        "mensaje": "No comparta su contrasena con otras personas.",
        "palabra_esperada": "contraseña",
    },
    {
        "mensaje": "La operacion requiere validación adicional.",
        "palabra_esperada": "operación",
    },
    {
        "mensaje": "Nuestro equipo de atencion está disponible.",
        "palabra_esperada": "atención",
    },
]

filas_prueba = []

for caso in pruebas_controladas:
    salida = asistente_bancario(caso["mensaje"], top_k=3)
    filas_prueba.append(
        {
            "mensaje_alterado": caso["mensaje"],
            "palabra_esperada": caso["palabra_esperada"],
            "mensaje_corregido": salida["mensaje_corregido"],
            "sugerencias": ", ".join(salida["sugerencias_siguiente_palabra"]),
            "correccion_detectada": (
                caso["palabra_esperada"] in salida["mensaje_corregido"].split()
            ),
        }
    )

tabla_pruebas = pd.DataFrame(filas_prueba)
display(tabla_pruebas)
```

4. Calcule exactitud de corrección sobre las diez pruebas:

```python
exactitud_corrector = tabla_pruebas["correccion_detectada"].mean()

print(f"Exactitud de corrección controlada: {exactitud_corrector:.2%}")
```

5. Evalúe exactitud de autocompletado `@3` sobre bigramas de prueba:

```python
def evaluar_autocompletado_top_k(
    df_evaluacion: pd.DataFrame,
    top_k: int = 3,
) -> tuple[float, int]:
    aciertos = 0
    total = 0

    for tokens in df_evaluacion["tokens"]:
        palabras = [mapear_unk(token) for token in tokens_lexicos(tokens)]

        for anterior, real in zip(palabras[:-1], palabras[1:]):
            ranking = sugerir_siguiente_palabra(anterior, top_k=top_k)
            predicciones = [palabra for palabra, _ in ranking]

            if real in predicciones:
                aciertos += 1

            total += 1

    exactitud = aciertos / total if total else 0.0
    return exactitud, total

exactitud_top3, total_predicciones = evaluar_autocompletado_top_k(test_df, top_k=3)

print(f"Predicciones evaluadas: {total_predicciones}")
print(f"Exactitud de autocompletado @3: {exactitud_top3:.2%}")
```

6. Guarde los resultados de evaluación:

```python
RESULTS_DIR = ROOT / "results"
RESULTS_DIR.mkdir(parents=True, exist_ok=True)

tabla_pruebas.to_csv(
    RESULTS_DIR / "pruebas_corrector.csv",
    index=False,
    encoding="utf-8",
)

metricas = {
    "cobertura_vocabulario_prueba": float(cobertura),
    "exactitud_corrector_controlado": float(exactitud_corrector),
    "exactitud_autocompletado_top3": float(exactitud_top3),
    "perplejidad_laplace_promedio": float(
        resultados_perplejidad["perplejidad_laplace"].mean()
    ),
    "mensajes_entrenamiento": int(len(train_df)),
    "mensajes_prueba": int(len(test_df)),
    "suavizado_k": K_SUAVIZADO,
    "umbral_rareza": UMBRAL_RAREZA,
}

pd.Series(metricas).to_json(
    RESULTS_DIR / "metricas.json",
    indent=2,
    force_ascii=False,
)

print("Resultados guardados en results/.")
```

7. Ejecute ejemplos integrados:

```python
ejemplos_asistente = [
    "Su trasnferencia fue procesada",
    "El saldo disponibel es",
    "La tarjetaa está disponible",
]

for mensaje in ejemplos_asistente:
    salida = asistente_bancario(mensaje, top_k=3)
    print("\nMensaje original:", salida["mensaje_original"])
    print("Propuesta:", salida["mensaje_corregido"])
    print("Continuaciones:", salida["sugerencias_siguiente_palabra"])
    print("Revisión humana:", salida["requiere_revision_humana"])
```

**Resultado esperado:**

- Existe una tabla con diez casos alterados.
- La función `asistente_bancario` devuelve una corrección propuesta y tres sugerencias.
- Las métricas quedan almacenadas en `results/metricas.json`.
- El asistente siempre indica que requiere revisión humana.

**Verificación:**

```python
salida = asistente_bancario("Su trasnferencia fue procesada", top_k=3)

assert "mensaje_corregido" in salida
assert "sugerencias_siguiente_palabra" in salida
assert len(salida["sugerencias_siguiente_palabra"]) == 3
assert salida["requiere_revision_humana"] is True
assert (RESULTS_DIR / "pruebas_corrector.csv").exists()
assert (RESULTS_DIR / "metricas.json").exists()

print("Integración del asistente verificada.")
```

## Validación y pruebas

Ejecute las siguientes comprobaciones finales en una celda del notebook:

```python
import json

with open(RESULTS_DIR / "metricas.json", encoding="utf-8") as archivo:
    metricas_finales = json.load(archivo)

print(json.dumps(metricas_finales, indent=2, ensure_ascii=False))

assert metricas_finales["mensajes_entrenamiento"] > 0
assert metricas_finales["mensajes_prueba"] > 0
assert 0.0 <= metricas_finales["cobertura_vocabulario_prueba"] <= 1.0
assert 0.0 <= metricas_finales["exactitud_corrector_controlado"] <= 1.0
assert 0.0 <= metricas_finales["exactitud_autocompletado_top3"] <= 1.0
assert metricas_finales["perplejidad_laplace_promedio"] > 0

print("Todas las validaciones finales fueron superadas.")
```

Como evidencia mínima de entrega, confirme que existen los siguientes archivos:

```text
data/raw/mensajes_bancarios.csv
data/processed/train.csv
data/processed/test.csv
notebooks/01_asistente_probabilistico_bancario.ipynb
docs/convenciones.md
results/pruebas_corrector.csv
results/metricas.json
```

Documente al final del notebook, en una celda Markdown, estas limitaciones:

1. El corpus es pequeño, sintético y no representa todos los productos, canales ni situaciones bancarias.
2. La frecuencia unigram puede favorecer palabras comunes aunque no sean correctas para el contexto.
3. El corrector solo considera candidatos a una edición y puede fallar ante errores múltiples, nombres propios o abreviaturas.
4. El bigrama considera únicamente una palabra previa; no comprende intención, políticas, estados transaccionales ni restricciones regulatorias.
5. El suavizado evita probabilidades cero, pero puede asignar probabilidad positiva a secuencias poco plausibles.
6. Una frase probable no es necesariamente verdadera, autorizada ni apta para comunicación externa.
7. Está prohibido entrenar, probar o registrar este ejercicio con información bancaria real o datos personales.
8. La aceptación, edición o descarte de toda sugerencia corresponde a una persona usuaria autorizada.

## Resolución de problemas

### 1. JupyterLab muestra otro kernel o aparece `ModuleNotFoundError`

**Síntoma:** el notebook no muestra `Python 3.12 (nlp-bancario-py312)` o falla al importar `pandas`, `numpy` o `sklearn`.

**Causa probable:** JupyterLab se abrió fuera del entorno `.venv`, las dependencias se instalaron en otro intérprete o el kernel no fue registrado correctamente.

**Solución:**

1. Detenga JupyterLab con `Ctrl+C`.
2. Active de nuevo el entorno virtual desde la raíz del proyecto.
3. Verifique el intérprete:

   ```bash
   python --version
   python -m pip --version
   ```

4. Reinstale el kernel:

   ```bash
   python -m ipykernel install --user \
     --name nlp-bancario-py312 \
     --display-name "Python 3.12 (nlp-bancario-py312)"
   ```

5. Inicie JupyterLab nuevamente con:

   ```bash
   jupyter lab --ip=127.0.0.1 --port=8888 --no-browser
   ```

6. En el notebook, use **Kernel > Change Kernel** y seleccione `Python 3.12 (nlp-bancario-py312)`.

### 2. El corrector no propone una palabra esperada o la perplejidad sin suavizado es infinita

**Síntoma:** una errata conserva su forma original, no aparece el candidato esperado o se observa `inf` en `perplejidad_sin_suavizado`.

**Causa probable:** la palabra correcta no quedó en el conjunto de entrenamiento, no pertenece al vocabulario del corrector, el error necesita más de una edición o el bigrama de prueba no fue observado durante entrenamiento.

**Solución:**

1. Verifique que la palabra exista en el vocabulario:

   ```python
   print("transferencia" in vocabulario_corrector)
   print(frecuencias_palabras["transferencia"])
   ```

2. Confirme que usa la misma semilla de partición:

   ```python
   random_state=42
   ```

3. Ejecute nuevamente todas las celdas en orden con **Run > Run All Cells**.
4. Mantenga `k=1.0` para calcular la perplejidad final; una perplejidad infinita sin suavizado es un comportamiento esperado ante bigramas no observados.
5. No agregue datos reales para aumentar cobertura. Si necesita ampliar el ejercicio, agregue únicamente mensajes sintéticos revisados.

## Limpieza

1. Guarde el notebook y confirme que los resultados requeridos permanecen en `results/`.
2. Opcionalmente, registre la línea base en Git:

   ```bash
   git add .
   git commit -m "Lab 01 asistente probabilistico bancario"
   ```

3. Detenga JupyterLab con `Ctrl+C` en la terminal.
4. Desactive el entorno virtual:

   ```bash
   deactivate
   ```

5. No elimine `data/`, `notebooks/`, `docs/` ni `results/`, ya que constituyen la línea base reutilizable para prácticas posteriores.

## Resumen

En esta práctica creó un asistente probabilístico local para mensajes bancarios sintéticos. Preparó un corpus con normalización y tokenización reproducibles, construyó un corrector ortográfico basado en frecuencia y distancia de edición, y entrenó un modelo bigrama con `<UNK>`, `<s>`, `</s>` y suavizado de Laplace.

También evaluó cobertura de vocabulario, exactitud de corrección, exactitud de autocompletado `@3` y perplejidad. El resultado es una línea base interpretable y auditable, útil para comprender modelos probabilísticos, pero insuficiente para tomar decisiones financieras o emitir comunicaciones sin reglas de negocio, filtros de seguridad y revisión humana.

### Recursos opcionales

- Jurafsky y Martin, *Speech and Language Processing*: https://web.stanford.edu/~jurafsky/slp3/
- NIST AI Risk Management Framework: https://www.nist.gov/itl/ai-risk-management-framework
- Reglamento General de Protección de Datos: https://eur-lex.europa.eu/eli/reg/2016/679/oj
- OWASP Top 10 para aplicaciones de modelos de lenguaje: https://owasp.org/www-project-top-10-for-large-language-model-applications/
