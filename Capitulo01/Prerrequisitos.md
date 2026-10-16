# Prerrequisitos

Este archivo es para quien prepara el equipo **antes** del taller. La persona participante no instala programas ni ejecuta los comandos de esta página.

El taller se realiza en **Windows Server**, con **Visual Studio Code** ya abierto en ese equipo (por ejemplo, por Escritorio remoto). Todo el ejercicio corre en el propio servidor: no usa base de datos, contenedor, API ni servicios externos.

## Equipo

| Condición | Valor |
|---|---|
| Sistema operativo | Windows Server con escritorio |
| Procesador | 64 bits, al menos 2 núcleos a 2.0 GHz |
| Memoria | Mínimo 8 GB; se recomiendan 16 GB |
| Disco libre | Al menos 5 GB |
| Pantalla | Resolución mínima de 1366x768 |
| Internet | Solo durante esta preparación |
| Permisos | La cuenta puede instalar programas para sí misma y crear carpetas en su perfil |

## Programas que deben quedar instalados

| Programa | Versión |
|---|---|
| Visual Studio Code | Actual |
| Python | 3.12.1 |
| pip | 23.3.2 |
| NumPy | 1.26.4 |
| pandas | 2.2.0 |
| matplotlib | 3.8.2 |
| scikit-learn | 1.4.0 |
| ipykernel | 6.29.0 |

Extensiones de Visual Studio Code, para la misma cuenta:

- Python (Microsoft)
- Jupyter (Microsoft)
- Spanish Language Pack (recomendado, para que los menús coincidan con la guía)

`python` debe responder en PowerShell sin activar un entorno. Al escribir `python --version` debe aparecer `Python 3.12.1`.

## Instalación

Haga esta preparación una sola vez, con la cuenta que usará el participante. Los comandos se escriben en **PowerShell**. Un comando es una instrucción de una línea: se escribe, se pulsa Enter y se espera a que el cursor vuelva a quedar libre antes de escribir la siguiente.

### 1. Abrir PowerShell

1. En el escritorio del servidor, abra el menú Inicio.
2. Escriba `PowerShell`.
3. Abra **Windows PowerShell**.

El título de la ventana debe decir PowerShell. Si dice Símbolo del sistema, ciérrela y abra PowerShell.

### 2. Permitir la descarga segura

Este comando le indica a Windows que use una conexión cifrada actual al descargar los instaladores. No instala nada.

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

No muestra un resultado. Si el cursor queda libre y no aparece texto en rojo, continúe.

### 3. Instalar Python 3.12.1

Este comando descarga el instalador oficial de Python 3.12.1 y lo guarda en la carpeta temporal del usuario.

```powershell
Invoke-WebRequest -Uri "https://www.python.org/ftp/python/3.12.1/python-3.12.1-amd64.exe" -OutFile "$env:TEMP\python-3.12.1-amd64.exe" -UseBasicParsing
```

Espere a que termine. La ventana no muestra una barra de avance; el cursor vuelve cuando el archivo ya está guardado.

Este comando instala Python solo para el usuario actual, incluye pip y agrega `python` a la ruta de programas. No pide confirmar pantallas.

```powershell
Start-Process -FilePath "$env:TEMP\python-3.12.1-amd64.exe" -ArgumentList "/quiet", "InstallAllUsers=0", "PrependPath=1", "Include_pip=1", "Include_test=0" -Wait
```

La instalación puede tardar unos minutos. Cuando el cursor vuelva, Python ya está en el equipo.

Este comando actualiza, en la ventana ya abierta, la lista de programas disponibles. Así PowerShell reconoce `python` sin cerrar la ventana.

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path", "User") + ";" + [System.Environment]::GetEnvironmentVariable("Path", "Machine")
```

Compruebe la versión. Debe imprimir `Python 3.12.1`.

```powershell
python --version
```

Si PowerShell dice que no reconoce `python`, cierre la ventana, ábrala de nuevo y repita `python --version`.

### 4. Instalar pip 23.3.2 y las librerías

pip es el programa que agrega librerías a Python. Este comando deja pip en la versión 23.3.2.

```powershell
python -m pip install --upgrade pip==23.3.2
```

Al final debe verse una línea parecida a `Successfully installed pip-23.3.2`.

Este comando instala las librerías que usa el cuaderno: tablas, números, gráficos, el separador de datos y el motor que ejecuta las celdas.

```powershell
python -m pip install numpy==1.26.4 pandas==2.2.0 matplotlib==3.8.2 scikit-learn==1.4.0 ipykernel==6.29.0
```

Espere hasta ver `Successfully installed`. Si alguna librería ya estaba en esa versión, el mensaje puede decir que el requisito ya está cumplido. En ambos casos puede seguir.

### 5. Registrar Python en Visual Studio Code

Este comando publica el Python instalado con el nombre **Python 3.12**, que es el que el cuaderno debe usar.

```powershell
python -m ipykernel install --user --name nlp-bancario-py312 --display-name "Python 3.12"
```

Debe aparecer una ruta que termina en `nlp-bancario-py312` y un texto de instalación correcta.

Si PowerShell responde que no puede ejecutar scripts porque la ejecución está deshabilitada, aplique esta permiso solo en la ventana actual y vuelva a lanzar el comando anterior:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Ese cambio se pierde al cerrar la ventana. No cambia la política del servidor.

### 6. Instalar Visual Studio Code

Este comando descarga el instalador de Visual Studio Code para el usuario actual.

```powershell
Invoke-WebRequest -Uri "https://update.code.visualstudio.com/latest/win32-x64-user/stable" -OutFile "$env:TEMP\VSCodeUserSetup.exe" -UseBasicParsing
```

Este comando lo instala sin abrir el editor al terminar. También agrega el comando `code`, que sirve para instalar extensiones desde PowerShell.

```powershell
Start-Process -FilePath "$env:TEMP\VSCodeUserSetup.exe" -ArgumentList "/VERYSILENT", "/NORESTART", "/MERGETASKS=!runcode" -Wait
```

Vuelva a cargar la lista de programas y compruebe que el editor responde:

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path", "User") + ";" + [System.Environment]::GetEnvironmentVariable("Path", "Machine")
code --version
```

La primera línea debe ser un número de versión, por ejemplo `1.95.0`. Cualquier versión reciente de la edición estable sirve.

### 7. Instalar las extensiones

Cada comando agrega una extensión. Espere el texto `was successfully installed` antes de pasar al siguiente.

La extensión Python permite elegir el intérprete y ejecutar código:

```powershell
code --install-extension ms-python.python
```

La extensión Jupyter abre los cuadernos `.ipynb` y muestra el botón de ejecutar celda:

```powershell
code --install-extension ms-toolsai.jupyter
```

El paquete de idioma deja los menús en español, como en la guía del taller:

```powershell
code --install-extension MS-CEINTL.vscode-language-pack-es
```

### 8. Dejar los menús en español

1. Abra Visual Studio Code desde el menú Inicio.
2. Pulse Ctrl+Mayús+P.
3. Escriba `Configure Display Language` y pulse Enter.
4. Elija **español (es)**.
5. Pulse **Restart** cuando lo pida.

Al volver a abrir, los menús deben estar en español. Confirme que existe **Terminal > Nueva terminal**.

## Comprobar que el equipo está listo

Cierre PowerShell, ábralo de nuevo con la misma cuenta y ejecute una línea cada vez:

```powershell
python --version
python -m pip --version
python -c "import numpy, pandas, matplotlib, sklearn; print('librerias listas')"
```

Resultado esperado:

- `Python 3.12.1`
- una línea de pip que mencione `23.3.2`
- el texto `librerias listas`

Después, en Visual Studio Code:

1. **Archivo > Nuevo archivo**.
2. Guárdelo como `prueba.ipynb` en el escritorio.
3. Pulse **Seleccionar kernel** y elija **Python 3.12**.
4. Cree una celda de código, pegue `print("listo")` y pulse el triángulo de ejecutar.

Debe aparecer `listo` debajo de la celda. Borre `prueba.ipynb` al terminar. Si el kernel no aparece, repita el paso 5 de la instalación y vuelva a abrir Visual Studio Code.

## Qué conviene haber visto en el curso

No hace falta saber programar ni administrar el servidor. Durante el taller se copian bloques ya escritos y se observa el resultado.

Conviene reconocer estas ideas del curso:

- Frecuencia y probabilidad de una palabra.
- Probabilidad condicional.
- Distancia mínima de edición.
- Modelos de lenguaje con n-gramas.
- Suavizado aditivo.
- Palabras fuera del vocabulario.
- Perplejidad.

## Seguridad de la información

El taller usa solo mensajes inventados. En el equipo, el cuaderno y las carpetas del ejercicio no deben copiarse mensajes reales de clientes, números de cuenta, tarjetas, identificadores, contraseñas, códigos de un solo uso, importes ni información operativa.
