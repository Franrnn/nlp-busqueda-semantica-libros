# Instalación y ejecución

[← Volver al README](../README.md)

## Instalación

Requiere Python 3.12 o 3.13.

```powershell
python -m venv .venv-NLP
.\.venv-NLP\Scripts\Activate.ps1
pip install -r requirements.txt
```

`requirements.txt` incluye las dependencias de los tres trabajos. Para volver a ejecutar el scraper del TP1 hace falta además el navegador de Playwright:

```powershell
playwright install chromium
```

El TP3 usa un modelo de spaCy en español, que no se instala con `requirements.txt`:

```powershell
python -m spacy download es_core_news_lg
```

## Ejecución

### TP1: scraper

```powershell
python tp1-extraccion/scraper.py
```

Abre un navegador Chromium, recorre el listado del género, extrae una muestra aleatoria de 150 libros y genera `tp1-extraccion/data/libros.csv`. Para ejecutarlo sin mostrar la ventana, cambiar la constante `HEADLESS` a `True` al inicio de `tp1-extraccion/scraper.py`. Sobrescribe el CSV existente y tarda varios minutos.

### TP2: notebook

**En local (VS Code o Jupyter).** Abrir `tp2-busqueda-semantica/TP2_Renna_Gomez_Ruiz_Esparza.ipynb`, elegir el kernel `.venv-NLP` y ejecutar todas las celdas en orden. También se puede ejecutar completo desde la terminal:

```powershell
jupyter nbconvert --to notebook --execute --inplace tp2-busqueda-semantica/TP2_Renna_Gomez_Ruiz_Esparza.ipynb
```

**En Google Colab.**

1. Subir el notebook a Colab (Archivo → Subir notebook).
2. Abrir el panel *Secrets* (ícono de llave en la barra izquierda), crear el secreto `DATABASE_URL` con la cadena de conexión y activar "Acceso desde el notebook".
3. Ejecutar las celdas en orden. La primera instala lo que Colab no trae.
4. Cuando las celdas lo pidan, subir `tp1-extraccion/data/libros.csv` (al principio) y `tp2-busqueda-semantica/queries.json` (en la Parte D).

La primera ejecución descarga el modelo `SBW-vectors-300-min5` (1,1 GB, a la carpeta `models/`) y el modelo SBERT (unos 500 MB), y tarda varios minutos. El resultado es el mismo en cada ejecución: se usa una semilla fija.

#### TP2: conexión a la base de datos (Supabase)

Las partes E y F y la parte avanzada del TP2 usan una base Postgres en Supabase. La cadena de conexión **no está en el repositorio**: se entrega por separado.

- **En local:** copiar `.env.example` como `.env` en la raíz del repo y pegar ahí la cadena (`DATABASE_URL=...`). El archivo `.env` está en `.gitignore`.
- **En Colab:** cargarla como secreto `DATABASE_URL` (paso 2 de arriba).

Usar la cadena **Session pooler**, que funciona en cualquier red y en Colab. La de conexión directa (`db.<proyecto>.supabase.co`) solo funciona en redes con IPv6.

Sin la cadena, el notebook se ejecuta igual: las celdas que usan la base avisan que se omiten y el resto corre completo. El notebook entregado ya tiene guardadas las salidas de esas celdas.

### TP3: notebook

Abrir `tp3-recomendador/TP3_grupo11.ipynb`, elegir el kernel `.venv-NLP` y ejecutar las celdas en orden. También se puede ejecutar completo desde la terminal:

```powershell
jupyter nbconvert --to notebook --execute --inplace tp3-recomendador/TP3_grupo11.ipynb
```

Los archivos de entrada ya están en el repositorio: `tp3-recomendador/data/catalogo_tp3.jsonl` (el catálogo de la cátedra, 8747 libros) y `tp3-recomendador/data/muestra_etiquetada.csv` (500 libros con la etiqueta `contaminado`, que el Paso 2 usa para calibrar y medir los detectores).

La primera ejecución descarga el modelo de oración del TP2 (`distiluse-base-multilingual-cased-v1`, unos 500 MB). El TP3 no usa la base de datos.

**La carpeta `tp3-recomendador/salidas/`** guarda resultados intermedios para no recalcularlos. Los dos detectores lentos del Paso 2 (el de idioma y el clasificador, que recorren unas 60.000 oraciones) dejan ahí su resultado: la primera ejecución tarda unos 45 minutos y las siguientes, unos pocos. Esos archivos están en el repositorio, así que quien clone no tiene que esperar esa primera vez. Si se cambia algo del Paso 1 y la cantidad de libros deja de coincidir, las celdas cortan con un error y hay que borrarlos.

Todas las divisiones usan `random_state=42` y la semilla de `langdetect` queda fija, así que cada ejecución da el mismo resultado.

#### TP3 en Google Colab

El notebook busca sus archivos con rutas relativas a su carpeta, así que en Colab hay que subir la carpeta `tp3-recomendador` completa, con `salidas/` incluida para no esperar los 45 minutos de los detectores.

1. En la PC, desde la carpeta `TUIA-NLP`, comprimir la carpeta (queda un zip de unos 40 MB):

   ```powershell
   Compress-Archive -Path tp3-recomendador\* -DestinationPath TP3.zip -Force
   ```

2. En Colab, abrir el notebook: Archivo → Subir notebook → `tp3-recomendador/TP3_grupo11.ipynb`. Alcanza con el entorno de CPU.
3. Subir `TP3.zip` desde el panel de archivos (ícono de carpeta en la barra izquierda).
4. Insertar una celda de código al principio del notebook y ejecutarla. Descomprime el zip, se para en la carpeta del TP3 e instala lo que Colab no trae (spaCy, scikit-learn y sentence-transformers ya vienen instalados):

   ```python
   # Descomprime TP3.zip, busca la carpeta donde quedó el catálogo y se para en ella
   import glob, os
   !unzip -q -o /content/TP3.zip -d /content/TP3
   carpeta = os.path.dirname(glob.glob("/content/TP3/**/catalogo_tp3.jsonl", recursive=True)[0])
   %cd {carpeta}
   !pip install -q thefuzz==0.22.1 python-Levenshtein langdetect==1.0.9
   !python -m spacy download es_core_news_lg
   ```

   Al terminar, spaCy siempre avisa "Restart to reload dependencies". No hace falta reiniciar, salvo que la celda que carga spaCy (Paso 2.4) diga `Can't find model 'es_core_news_lg'`: en ese caso, reiniciar la sesión y ejecutar todo de nuevo.

5. Entorno de ejecución → Ejecutar todas. La primera vez descarga además el modelo de oración (unos 500 MB). Las celdas de los detectores del Paso 2 y la de embeddings del Paso 8 tienen que decir que leyeron su resultado de `salidas/`.

Para comprobar que corrió bien: el Paso 2 marca 380 sospechosos y deja 8212 libros limpios, el Paso 4 imprime `80 20` y la celda de código del Paso 7 da precision@5 de 0,4, 0,6 y 0,4 (azar 0,30). Por diferencias de redondeo, el clasificador del Paso 2 puede marcar uno o dos libros más o menos, pero los sospechosos tienen que seguir siendo 380.

Los archivos que genera el notebook en Colab se borran al cerrar la sesión. El notebook descargado de Colab no reemplaza al del repositorio, porque incluye la celda de preparación.

### TP3: valorar los libros (Paso 3)

El Paso 3 necesita que **una sola persona del grupo** valore los libros a mano. Eso no se hace en el notebook sino con un script aparte, porque lleva alrededor de una hora y conviene poder cortar y retomar:

```powershell
.\.venv-NLP\Scripts\python.exe tp3-recomendador\valorar.py
```

Muestra un libro por pantalla, con su título y su sinopsis, y se responde con una tecla:

| Tecla | Significado |
| --- | --- |
| `1` | me interesaría leerlo |
| `0` | no me interesaría leerlo |
| `Enter` | no me decido; el libro queda sin valorar |
| `s` | es spam que el Paso 2 no detectó |
| `q` | salir y seguir más tarde |

Recorre el catálogo limpio en el mismo orden que el notebook (`sample(frac=1, random_state=42)`) y **guarda cada respuesta en el momento** en `tp3-recomendador/salidas/valoraciones_log.csv`, así que se puede cortar en cualquier momento y al volver a abrirlo sigue donde había quedado. Arriba muestra cuántas respuestas faltan para llegar a las 100 válidas y a las 20 de cada clase que pide la consigna.

Cuando el objetivo esté cumplido, hay que volver a ejecutar el Paso 3 del notebook: de ahí sale `tp3-recomendador/preferencias.csv`.
