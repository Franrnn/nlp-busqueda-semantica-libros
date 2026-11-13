# Modelo de datos

[← Volver al README](../README.md)

### `tp1-extraccion/data/libros.csv` (salida del TP1)

Una fila por libro, 150 filas.

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `id` | string | Identificador de la ficha en Lectulandia (atributo `data-book` del sitio). Si el sitio no lo trae, se genera un `fallback-<n>`. |
| `titulo` | string | Título del libro. |
| `autores` | array (JSON) | Autor o autores, ej. `["Isaac Asimov"]`. Se guarda como string JSON y se recupera con `json.loads`. |
| `generos` | array (JSON) | Géneros que la ficha declara, ej. `["Ciencia ficción", "Novela"]`. Mismo formato que `autores`. |
| `serie` | string | Serie a la que pertenece el libro; cadena vacía si no pertenece a ninguna. |
| `sinopsis` | string | Texto completo de la sinopsis, sin saltos de línea ni espacios repetidos. |
| `url_libro` | string | Dirección de la ficha del libro en Lectulandia. |
| `categoria_origen` | string | Categoría por la que el libro ingresó al dataset (siempre "Ciencia ficción"). |
| `fecha_extraccion` | string (fecha) | Fecha y hora en que se obtuvo el registro, formato `YYYY-MM-DD HH:MM:SS`. |
| `url_portada` | string | Dirección de la imagen de portada. |

### `tp2-busqueda-semantica/data/libros_preprocesados.csv`

Las mismas filas y columnas de `libros.csv`, más dos:

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `sinopsis_emb` | string | Versión **cruda**: la sinopsis original, solo con los espacios normalizados. Es la entrada del modelo de oración (SBERT). |
| `sinopsis_limpio` | string | Versión **limpia**: minúsculas, sin puntuación, sin números, sin stopwords (lista de español de NLTK) y sin palabras de una letra. Es la entrada de TF-IDF. |

### `tp2-busqueda-semantica/data/libros_vectores.pkl`

DataFrame de pandas (se lee con `pd.read_pickle`) con las columnas anteriores y los vectores de cada libro. En este archivo `autores` y `generos` son listas de Python.

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `vec_propio` | array de 100 `float32` | Promedio de los vectores de palabra del Word2Vec entrenado sobre el corpus, con norma 1. |
| `cob_propio` | float (0 a 1) | Fracción de las palabras de la sinopsis (sin stopwords) que tienen vector en ese modelo. |
| `vec_sbw` | array de 300 `float32` | Promedio de los vectores de palabra de `SBW-vectors-300-min5`, con norma 1. |
| `cob_sbw` | float (0 a 1) | Lo mismo que `cob_propio`, con el vocabulario de SBW. |
| `n_tokens_sbert` | entero | Cantidad de tokens de la sinopsis según el tokenizador de SBERT. Si supera 128, el modelo la trunca. |
| `vec_sbert` | array de 512 `float32` | Embedding de la sinopsis cruda con `distiluse-base-multilingual-cased-v1`, con norma 1. |

`tp2-busqueda-semantica/data/emb_sbert.npy` es la misma información de `vec_sbert` como matriz de NumPy de 150 × 512, con las filas en el orden de `libros_preprocesados.csv`.

### `tp2-busqueda-semantica/data/chunks.pkl` y `tp2-busqueda-semantica/data/emb_sbert_chunks.npy`

Parte avanzada del TP2. `chunks.pkl` es un DataFrame con una fila por fragmento de sinopsis (322 en total); `emb_sbert_chunks.npy` es la matriz de 322 × 512 con sus embeddings, en el mismo orden.

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `libro_id` | string | `id` del libro al que pertenece el fragmento. |
| `titulo`, `generos` | string, lista | Copiados del libro. |
| `chunk_idx` | entero | Número de fragmento dentro del libro, empezando en 0. |
| `texto` | string | Oraciones consecutivas de la sinopsis que juntas no superan los 128 tokens de SBERT. |
| `n_tokens` | entero | Tokens del fragmento según el tokenizador de SBERT. |
| `vec` | array de 512 `float32` | Embedding del fragmento, con norma 1. |

### `tp2-busqueda-semantica/queries.json`

Conjunto de evaluación del TP2: 18 consultas en lenguaje natural y, para cada una, los libros del corpus que se consideran relevantes.

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `id` | string | Identificador de la consulta (`q01` a `q18`). |
| `consulta` | string | Texto de la consulta. |
| `criterio` | string | Qué se consideró relevante para esa consulta y qué no. |
| `sin_palabras_en_comun` | booleano | `true` en las consultas diseñadas para no compartir ninguna palabra con sus libros relevantes (`q17` y `q18`). |
| `relevantes` | array de objetos | Libros relevantes, cada uno con `id` (el mismo de `libros.csv`) y `titulo`. |

### Tablas en Supabase (Postgres + `pgvector`)

Una tabla por modelo. `libros_word2vec` y `libros_sbert` tienen las mismas columnas y 150 filas; solo cambia la dimensión del vector.

| Columna | Tipo | Descripción |
| --- | --- | --- |
| `id` | `text` (clave primaria) | Mismo `id` que en `libros.csv`. |
| `titulo` | `text` | Título del libro. |
| `generos` | `text[]` | Géneros del libro, como array para poder filtrar en SQL. |
| `embedding` | `vector(300)` en `libros_word2vec`, `vector(512)` en `libros_sbert` | Promedio de vectores de palabra de SBW y embedding de SBERT, respectivamente. |

`libros_sbert_chunks` guarda un vector por fragmento (322 filas):

| Columna | Tipo | Descripción |
| --- | --- | --- |
| `libro_id`, `chunk_idx` | `text`, `int` (clave primaria compuesta) | Libro y número de fragmento. |
| `titulo`, `generos` | `text`, `text[]` | Copiados del libro. |
| `texto` | `text` | Texto del fragmento. |
| `embedding` | `vector(512)` | Embedding de SBERT del fragmento. |

Las tres tablas tienen un índice HNSW sobre `embedding` con la opclass `vector_cosine_ops`. El notebook las borra y las vuelve a crear en cada ejecución.

### Archivos del TP3

Entradas que entrega la cátedra:

| Archivo | Qué contiene |
| --- | --- |
| `tp3-recomendador/data/catalogo_tp3.jsonl` | 8747 libros, uno por línea. Campos: `source_url`, `slug`, `title`, `authors`, `genres`, `seed_genre`, `series`, `series_number`, `cover_url`, `synopsis`, `synopsis_chars`, `scraped_at`. Algunos registros están contaminados y no se indica cuáles. |
| `tp3-recomendador/data/muestra_etiquetada.csv` | 500 libros del mismo catálogo con `contaminado` (1 o 0). En los contaminados, `fragmento` trae el texto que se insertó. El 20 % está contaminado, bastante más que el catálogo completo. |

Salidas que genera el notebook:

| Archivo | Qué contiene |
| --- | --- |
| `tp3-recomendador/salidas/catalogo_paso1.csv` | Los 8592 libros que quedan después del Paso 1, con `slug`, `title`, `authors`, `genres`, `series` y `synopsis`. Se regenera en segundos, así que no se sube al repositorio. |
| `tp3-recomendador/sospechosos.csv` | Los 380 libros que la auditoría marca, con `slug` y `motivo`. El motivo combina con `+` los detectores que lo marcaron: `regex`, `normalizacion`, `idioma`, `clasificador`, `casi_duplicado`. |
| `tp3-recomendador/catalogo_limpio.csv` | Los 8212 libros que quedan al sacar los sospechosos. **Es la entrada de los pasos 3 a 8**, y conviene no regenerarlo: el Paso 3 lo mezcla con `random_state=42` y el lector valora los libros en ese orden. |
| `tp3-recomendador/salidas/deteccion_idioma_catalogo.csv` | Cache del detector de idioma: `slug` y `marcado`. |
| `tp3-recomendador/salidas/deteccion_clasificador_catalogo.csv` | Cache del clasificador: `slug` y la `probabilidad` más alta entre las oraciones de cada sinopsis. |
| `tp3-recomendador/salidas/valoraciones_log.csv` | Lo que respondió el lector en `valorar.py`, una fila por libro visto: `orden` (posición en el recorrido), `slug`, `respuesta` (`1`, `0`, vacío o `s`) y `momento`. Se escribe de a una fila, para poder cortar y retomar. |
| `tp3-recomendador/preferencias.csv` | Un libro por fila con `slug`, `title`, `synopsis` e `interes` (1, 0 o vacío). El Paso 4 le agrega la columna `conjunto`. Las abstenciones se conservan: un libro sin valorar tiene preferencia desconocida, no es un 0. |
| `tp3-recomendador/grupo11_sospechosos.csv` | Copia de `sospechosos.csv` con el nombre que pide la entrega. |
| `tp3-recomendador/grupo11_preferencias.csv` | Copia de `preferencias.csv` con la columna `conjunto` (`entrenamiento` o `prueba`; vacía en las abstenciones), con el nombre que pide la entrega. |
| `tp3-recomendador/salidas/embeddings_candidatos.npy` | Cache del Paso 8: embeddings (8112 × 512) de los libros del catálogo limpio que el lector no valoró. `embeddings_candidatos_slugs.csv` guarda sus `slug` en el mismo orden; si no coinciden con los candidatos, la celda los recalcula (unos 4 minutos). |

Las celdas que leen un cache comprueban antes que sus `slug` coincidan con los libros actuales; si no coinciden, cortan con un error (idioma y clasificador) o lo recalculan (embeddings del Paso 8).
