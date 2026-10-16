# Construcción de un asistente probabilístico para corregir y autocompletar mensajes bancarios

## Metadatos

| Campo | Valor |
|---|---|
| Duración | 105 minutos |
| Complejidad | Media |
| Nivel de Bloom | Crear |
| Sistema | Windows Server |
| Herramienta | Visual Studio Code |

## Descripción general

En esta práctica construirá un asistente de ejemplo para mensajes bancarios inventados. El asistente detecta errores de escritura sencillos, propone la palabra corregida y sugiere cómo podría continuar la frase.

Para corregir, compara la palabra escrita con las palabras que ya vio y mide cuántos cambios harían falta para llegar a una palabra conocida. Para sugerir la continuación, calcula qué palabra suele ir después de la anterior.

Todo ocurre en el Windows Server, dentro de Visual Studio Code. No se consulta internet, ni una base de datos, ni otro sistema. Las sugerencias sirven para aprender: no confirman saldos, pagos, plazos ni decisiones del banco.

Los programas ya están instalados. La lista está en [Prerrequisitos.md](../Capitulo01/Prerrequisitos.md). Durante el taller no se instala software.

## Objetivos de aprendizaje

Al finalizar la práctica, podrá:

- [ ] Preparar un conjunto de mensajes inventados, ordenarlos en palabras y separarlos en dos grupos repetibles: uno para enseñar al asistente y otro para probarlo.
- [ ] Construir un corrector que proponga palabras conocidas a un solo cambio de distancia y las ordene por qué tan frecuentes son.
- [ ] Calcular qué tan probable es una palabra y qué tan probable es que una palabra siga a otra, incluyendo palabras nuevas o poco vistas.
- [ ] Medir si el asistente cubre las palabras de prueba, cuántas correcciones acierta y qué tan "sorprendido" queda ante una frase.
- [ ] Unir corrección y sugerencia de la siguiente palabra en una sola función, y dejar escritas las limitaciones de uso.

## Cómo está organizado el trabajo

Trabajará con tres piezas. Conviene reconocerlas antes de empezar:

| Pieza | Qué es | Dónde la verá |
|---|---|---|
| Explorador | La lista de carpetas y archivos | Barra izquierda de Visual Studio Code |
| Terminal | El panel donde se escriben comandos cortos | Parte inferior. El título debe decir PowerShell |
| Cuaderno | Un archivo donde se alternan explicaciones, código y resultados | `01_asistente_probabilistico_bancario.ipynb` |

Un **comando** es una instrucción de una línea. Se escribe en la terminal y se confirma con Enter. En este taller los comandos solo crean la carpeta de trabajo y comprueban que Python responde.

El **código Python** no se escribe en la terminal. Se pega en una celda del cuaderno y se ejecuta con el triángulo que aparece a la izquierda de esa celda.

Una **celda** es un recuadro del cuaderno. Hay celdas de texto (Markdown) y celdas de código (Python).

## Cómo ejecutar cada bloque de código

Repita estos cinco pasos con cada bloque Python de la guía:

1. En la barra del cuaderno, pulse **+ Código** (+ Code).
2. Copie el bloque de la guía.
3. Péguelo en la celda vacía.
4. Pulse el triángulo **Ejecutar celda** a la izquierda de la celda. También vale Mayús+Enter (Shift+Enter).
5. Lea el resultado debajo de la celda.

Ejecute las celdas de arriba hacia abajo. Cada una usa lo que calcularon las anteriores. Si debajo de una celda aparece un texto en rojo, deténgase y vaya a [Resolución de problemas](#resolución-de-problemas). No continúe con las celdas siguientes hasta que esa celda muestre un resultado normal.

> **Nota de seguridad:** use solo los mensajes de esta guía. No copie mensajes reales de clientes, números de cuenta, tarjetas, identificadores, contraseñas, códigos de un solo uso, importes ni información operativa.

## Procedimiento paso a paso

### Paso 1. Crear la carpeta de trabajo y el cuaderno

**Objetivo:** dejar una carpeta con el nombre esperado y abrir el cuaderno en Visual Studio Code.

**Instrucciones:**

1. Abra Visual Studio Code en el Windows Server.

2. Abra la terminal. En el menú superior elija **Terminal > Nueva terminal** (Terminal > New Terminal).

   Si el panel no dice **PowerShell**, pulse la flecha que está junto al signo **+** del panel y elija **PowerShell**.

3. El siguiente comando crea la carpeta del taller dentro de su usuario. No instala nada. Escríbalo y pulse Enter.

   ```powershell
   mkdir $env:USERPROFILE\taller-pln-probabilistico
   ```

   Si el mensaje indica que la carpeta ya existe, continúe con el paso siguiente.

4. Este comando entra en esa carpeta. Después de Enter, la ruta de la terminal debe terminar en `taller-pln-probabilistico`.

   ```powershell
   cd $env:USERPROFILE\taller-pln-probabilistico
   ```

5. Este comando crea las carpetas internas del ejercicio: datos, cuaderno, resultados y notas.

   ```powershell
   mkdir data, data\raw, data\processed, notebooks, src, results, tests, docs
   ```

6. Este comando pregunta qué Python hay en el equipo. Debe responder `Python 3.12.1`. Si el texto es otro, o si PowerShell dice que no reconoce `python`, avise a quien preparó el servidor. No instale nada.

   ```powershell
   python --version
   ```

7. Abra esa carpeta en Visual Studio Code: **Archivo > Abrir carpeta** (File > Open Folder). Elija `taller-pln-probabilistico`, dentro de su carpeta de usuario. Si aparece una pregunta sobre confiar en los archivos, pulse **Sí** (Yes). Es la carpeta que acaba de crear.

8. En el explorador de la izquierda debe ver `data`, `notebooks`, `src`, `results`, `tests` y `docs`. Si no aparecen, pulse el icono de actualizar del explorador.

9. Pulse con el botón derecho la carpeta `notebooks` y elija **Nuevo archivo** (New File). Escriba exactamente este nombre y pulse Enter:

   ```text
   01_asistente_probabilistico_bancario.ipynb
   ```

   Visual Studio Code abre el cuaderno. Arriba a la derecha pulse **Seleccionar kernel** (Select Kernel) y elija **Python 3.12**. El kernel es el programa que ejecuta las celdas. Si no aparece, avise a quien preparó el equipo.

10. Pulse **+ Markdown** para crear una celda de texto. Escriba:

    ```markdown
    # Asistente probabilístico bancario

    Mensajes inventados. No contienen ni deben contener información bancaria real.
    ```

11. Guarde con **Archivo > Guardar** o con Ctrl+S.

**Resultado esperado:**

- El explorador muestra la carpeta `taller-pln-probabilistico` y sus subcarpetas.
- Existe el archivo `notebooks/01_asistente_probabilistico_bancario.ipynb`.
- El kernel del cuaderno es Python 3.12.

**Verificación:**

Pegue este bloque en una celda de código y ejecútelo. Comprueba que el cuaderno está dentro de la carpeta correcta. En Visual Studio Code la celda a veces se ejecuta desde `notebooks`; el bloque sube un nivel cuando hace falta.

```python
from pathlib import Path
import sys

ROOT = Path.cwd().resolve()
if ROOT.name == "notebooks":
    ROOT = ROOT.parent

if ROOT.name != "taller-pln-probabilistico":
    raise RuntimeError(
        "Abra en Visual Studio Code la carpeta taller-pln-probabilistico. "
        f"Directorio actual: {Path.cwd()}"
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

Debe imprimir una versión `3.12` y una ruta de Windows que termine en `taller-pln-probabilistico`.

---

### Paso 2. Crear y cargar los mensajes de ejemplo

**Objetivo:** guardar en un archivo una lista de mensajes bancarios inventados. Esos mensajes son el material con el que el asistente aprende qué palabras son frecuentes.

**Instrucciones:**

1. Cree una celda de código y ejecute este bloque. Crea el archivo `data/raw/mensajes_bancarios.csv`.

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

Debajo de la celda debe verse la ruta del archivo, la cantidad de mensajes y una tabla con las primeras filas. La tabla es la forma en que el cuaderno muestra filas y columnas.

2. Este bloque vuelve a leer el archivo desde el disco, para trabajar con lo guardado y no solo con lo que está en memoria.

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

3. Este bloque cuenta caracteres y marca los mensajes repetidos a propósito.

```python
df["longitud_caracteres"] = df["mensaje"].str.len()
df["duplicado"] = df["mensaje"].duplicated(keep=False)

display(df[["longitud_caracteres", "duplicado"]].describe())
print("Mensajes duplicados controlados:", int(df["duplicado"].sum()))
```

**Resultado esperado:**

- Existe `data/raw/mensajes_bancarios.csv`. Puede abrirlo desde el explorador para ver el texto.
- Hay al menos 50 mensajes.
- Cada fila tiene `id_mensaje`, `mensaje` y `origen`.
- En todas las filas, `origen` vale `sintetico`.

**Verificación:**

```python
assert len(df) >= 50
assert not df["mensaje"].str.contains(
    r"\b\d{12,19}\b", regex=True
).any(), "Posible identificador numérico sensible detectado."

print("Verificación de corpus sintético completada.")
```

> Esta revisión solo busca números muy largos. En un proceso real, los controles de seguridad son más amplios y siguen la política de protección de datos de la organización.

---

### Paso 3. Pasar los mensajes a palabras y separarlos en dos grupos

**Objetivo:** dejar el texto en minúsculas, conservar tildes y eñes, separar palabras y signos, y guardar dos grupos fijos: entrenamiento y prueba.

Entrenamiento es el grupo con el que el asistente cuenta palabras. Prueba es el grupo que se reserva para medir después, sin usarlo para enseñar.

**Instrucciones:**

1. Añada una celda con las funciones que limpian y separan el texto.

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

2. Aplique esas funciones a todos los mensajes:

```python
df["texto_normalizado"] = df["mensaje"].map(normalizar_texto)
df["tokens"] = df["texto_normalizado"].map(tokenizar)
df["tokens_lexicos"] = df["tokens"].map(tokens_lexicos)

display(df[["mensaje", "texto_normalizado", "tokens"]].head(8))
```

3. Separe los mensajes. El número `42` es una semilla: hace que la separación sea la misma cada vez que se ejecuta el cuaderno de arriba hacia abajo.

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

4. Guarde los dos grupos en archivos:

```python
PROCESSED_DIR = ROOT / "data" / "processed"
PROCESSED_DIR.mkdir(parents=True, exist_ok=True)

train_df.to_csv(PROCESSED_DIR / "train.csv", index=False, encoding="utf-8")
test_df.to_csv(PROCESSED_DIR / "test.csv", index=False, encoding="utf-8")
```

5. Guarde una nota con las reglas usadas, para que otra persona pueda repetir el ejercicio:

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

- Los textos quedan en minúsculas.
- Las tildes y la `ñ` se mantienen.
- Los signos quedan como piezas separadas.
- La separación se repite gracias a `random_state=42`.
- Existen `data/processed/train.csv`, `data/processed/test.csv` y `docs/convenciones.md`.

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

### Paso 4. Contar palabras y proponer correcciones

**Objetivo:** armar un corrector que genere palabras parecidas (quitando, agregando, cambiando o intercambiando una letra) y se quede con las que ya conoce, ordenadas de la más frecuente a la menos frecuente.

**Instrucciones:**

1. Cuente las palabras del grupo de entrenamiento:

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

La lista final son las diez palabras que más se repiten. El corrector prefiere esas palabras cuando hay varias correcciones posibles.

2. Mida la distancia entre dos palabras. Un intercambio de letras vecinas cuenta como un solo cambio. Así, `trasnferencia` está a un cambio de `transferencia`.

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

3. Genere todas las palabras a un solo cambio:

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

4. Quédese solo con las palabras conocidas y ordénelas:

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

5. Pruebe cuatro errores de escritura:

```python
for errata in ["trasnferencia", "tarjetaa", "disponibel", "prosesada"]:
    print(f"{errata:15} -> {sugerir_correccion(errata, top_k=3)}")
```

**Resultado esperado:**

- `trasnferencia` propone primero `transferencia`, si esa palabra quedó en el grupo de entrenamiento.
- `tarjetaa` propone `tarjeta`.
- `disponibel` propone `disponible`.
- El orden sale de la frecuencia, no solo del parecido de las letras.

**Verificación:**

```python
assert distancia_damerau_levenshtein("trasnferencia", "transferencia") == 1
assert distancia_damerau_levenshtein("tarjetaa", "tarjeta") == 1
assert sugerir_correccion("123456")[0][0] == "123456"

print("Corrector básico verificado.")
```

> El corrector no entiende la intención de la frase. Si dos palabras válidas se parecen, puede elegir la más frecuente aunque no sea la adecuada. En un uso real, la propuesta se combina con el contexto, con reglas del negocio y con la revisión de una persona.

---

### Paso 5. Calcular qué palabra suele seguir a otra

**Objetivo:** estimar la probabilidad de que una palabra venga después de otra. Las palabras que aparecieron una sola vez, o que no aparecieron, se tratan como una misma etiqueta: `<UNK>` (desconocida). El inicio de la frase se marca con `<s>` y el final con `</s>`.

Sin un ajuste, una combinación nunca vista vale cero y anula toda la frase. El suavizado de Laplace suma una cantidad pequeña (`k = 1`) para que toda combinación posible conserve una probabilidad mayor que cero.

**Instrucciones:**

1. Defina el vocabulario. Las palabras con frecuencia 1 o menor pasan a `<UNK>`:

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

2. Prepare cada frase con las marcas de inicio, fin y palabra desconocida:

```python
def mapear_unk(palabra: str) -> str:
    return palabra if palabra in vocabulario_lm else "<UNK>"

def preparar_secuencia_lm(tokens: list[str]) -> list[str]:
    """Convierte tokens léxicos en secuencia con marcadores de inicio y fin."""
    palabras = tokens_lexicos(tokens)
    palabras_mapeadas = [mapear_unk(palabra) for palabra in palabras]
    return ["<s>"] + palabras_mapeadas + ["</s>"]
```

3. Cuente cada par de palabras seguidas en el grupo de entrenamiento:

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

Un bigrama es un par: la palabra anterior y la palabra actual. La lista final muestra los pares más repetidos.

4. Calcule la probabilidad del par con suavizado y sin suavizado:

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

5. Compare un par habitual con uno que el entrenamiento no vio:

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

- `<UNK>` forma parte del vocabulario.
- Un par nunca visto vale `0.0` sin suavizado.
- Con `k = 1.0`, ese mismo par vale más que cero.
- `<s>` y `</s>` entran en la cuenta de cada mensaje.

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

### Paso 6. Medir las frases y sugerir la palabra siguiente

**Objetivo:** asignar una probabilidad a una frase completa, resumir esa probabilidad con la perplejidad y proponer las tres continuaciones más probables.

La perplejidad es un resumen: un número bajo indica que la frase se parece a las que el asistente ya vio; un número alto indica que la frase le resulta poco habitual. Sin suavizado, una sola combinación nunca vista deja la perplejidad en infinito.

**Instrucciones:**

1. Calcule la probabilidad y la perplejidad con suavizado. Se usan logaritmos para que el producto de muchas probabilidades pequeñas no se vuelva cero por un límite del computador.

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

2. Calcule la misma medida sin suavizado, para ver el efecto de un par nunca visto:

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

3. Proponga la palabra siguiente a partir de la última palabra de la frase:

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

4. Pruebe cuatro inicios de frase:

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

Cada inicio debe mostrar tres palabras. La primera es la continuación más probable según los mensajes de entrenamiento.

5. Mida, en el grupo de prueba, qué parte de las palabras ya se conocía y cuál es la perplejidad de cada mensaje:

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

- Cada frase de prueba ofrece tres sugerencias, de mayor a menor probabilidad.
- La perplejidad con Laplace es un número finito en todos los mensajes que tienen palabras.
- La perplejidad sin suavizado puede salir como `inf` cuando aparece un par no visto. Eso es el comportamiento previsto.
- La cobertura es un porcentaje entre 0 % y 100 %.

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

### Paso 7. Unir corrección y sugerencia, y probar diez frases

**Objetivo:** dejar una sola función que reciba un mensaje, proponga el texto corregido y tres palabras para continuar. Después se prueba con diez frases alteradas a propósito.

**Instrucciones:**

1. Reconstruya la frase cuidando el espacio de los signos:

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

2. Defina la función del asistente:

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

3. Pruebe diez mensajes con un error cada uno:

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

4. Calcule qué porcentaje de esas diez correcciones apareció en el texto propuesto:

```python
exactitud_corrector = tabla_pruebas["correccion_detectada"].mean()

print(f"Exactitud de corrección controlada: {exactitud_corrector:.2%}")
```

5. Mida cuántas veces la palabra real del grupo de prueba está entre las tres sugerencias:

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

El texto `@3` significa "entre las tres primeras sugerencias".

6. Guarde la tabla y las medidas en la carpeta `results`:

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

7. Vea el asistente completo con tres ejemplos:

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

- La tabla tiene diez frases alteradas.
- `asistente_bancario` devuelve un texto propuesto y tres continuaciones.
- Las medidas quedan en `results/metricas.json`.
- Todas las respuestas indican que hace falta revisión humana.

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

Ejecute esta última comprobación en una celda de código:

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

En el explorador, confirme que existen estos archivos:

```text
data/raw/mensajes_bancarios.csv
data/processed/train.csv
data/processed/test.csv
notebooks/01_asistente_probabilistico_bancario.ipynb
docs/convenciones.md
results/pruebas_corrector.csv
results/metricas.json
```

Al final del cuaderno, cree una celda Markdown y copie estas limitaciones:

1. Los mensajes son pocos, inventados, y no cubren todos los productos, canales ni situaciones del banco.
2. La palabra más frecuente puede ganar aunque no encaje en la frase.
3. El corrector solo prueba un cambio. Falla si hay varios errores, nombres propios o abreviaturas.
4. La sugerencia mira solo la palabra anterior. No conoce la intención, las políticas, el estado de una transacción ni las normas.
5. El suavizado evita el cero, y por eso también da probabilidad a frases poco reales.
6. Una frase probable no es una frase verdadera, autorizada ni lista para enviarse.
7. Está prohibido enseñar, probar o guardar este ejercicio con información bancaria real o con datos personales.
8. Aceptar, editar o descartar una sugerencia corresponde a una persona autorizada.

## Resolución de problemas

### 1. El cuaderno no ejecuta el código o no encuentra pandas

**Qué se ve:** el botón de kernel dice otra versión de Python, o una celda muestra en rojo `ModuleNotFoundError` al usar `pandas`, `numpy` o `sklearn`.

**Qué suele pasar:** Visual Studio Code está usando un Python distinto del que preparó tecnología.

**Qué hacer:**

1. Arriba a la derecha del cuaderno, pulse el nombre del kernel y elija **Python 3.12**.
2. En la terminal de PowerShell, dentro de la carpeta del taller, compruebe el programa disponible:

   ```powershell
   python --version
   ```

   Debe decir `Python 3.12.1`.

3. Vuelva a ejecutar las celdas desde la primera. En el cuaderno, abra el menú **...** de la barra y elija **Ejecutar todo** (Run All).

Si Python 3.12 no aparece en la lista, detenga el taller y pida la revisión del equipo. La instalación está descrita en [Prerequisitos.md](../Prerequisitos.md). No ejecute instaladores durante la práctica.

### 2. El corrector deja la palabra igual, o la perplejidad sin suavizado sale infinita

**Qué se ve:** una palabra con error no cambia, no aparece la corrección esperada, o en la columna `perplejidad_sin_suavizado` se lee `inf`.

**Qué suele pasar:** la palabra correcta quedó en el grupo de prueba y el corrector no la conoce; el error necesita más de un cambio; o el par de palabras de la frase de prueba no apareció en el entrenamiento. Un valor `inf` sin suavizado es esperable en ese último caso.

**Qué hacer:**

1. Compruebe si la palabra entró al vocabulario del corrector. Pegue esto en una celda de código:

   ```python
   print("transferencia" in vocabulario_corrector)
   print(frecuencias_palabras["transferencia"])
   ```

2. Confirme que la separación usa la semilla `random_state=42`, como en el paso 3.
3. Ejecute de nuevo todas las celdas en orden con **Ejecutar todo** (Run All).
4. Use la perplejidad con Laplace (`k = 1.0`) como resultado del taller. Conserve el infinito sin suavizado como contraste, no como fallo.
5. Amplíe el ejercicio solo con mensajes inventados y revisados. No agregue datos reales para subir la cobertura.

## Cierre

1. Guarde el cuaderno con Ctrl+S.
2. En el explorador, confirme que los archivos de `results` siguen ahí.
3. Cierre la pestaña del cuaderno. Puede cerrar Visual Studio Code.
4. Conserve las carpetas `data`, `notebooks`, `docs` y `results`. Son el resultado del taller y sirven para repetir la práctica.

## Resumen

En esta práctica construyó, en Windows Server y con Visual Studio Code, un asistente local para mensajes bancarios inventados. Preparó esos mensajes, los separó en palabras y en dos grupos repetibles, armó un corrector por frecuencia y por cambios de letras, y calculó la palabra siguiente con pares de palabras, la etiqueta `<UNK>` y el suavizado de Laplace.

También midió la cobertura del vocabulario, los aciertos del corrector, los aciertos de las tres sugerencias y la perplejidad. El asistente ayuda a ver cómo funciona un modelo probabilístico. Una sugerencia suya no reemplaza las reglas del negocio, los controles de seguridad ni la revisión de una persona.

### Lecturas opcionales

- Jurafsky y Martin, *Speech and Language Processing*: https://web.stanford.edu/~jurafsky/slp3/

