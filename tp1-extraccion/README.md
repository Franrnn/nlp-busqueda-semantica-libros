# TP1 — Extracción de texto

[← Volver al README principal](../README.md)

**Objetivo:** armar un dataset de libros de ciencia ficción, con sus metadatos y su sinopsis, a partir del sitio Lectulandia.

## Qué hace

- `scraper.py` recorre las 182 páginas del listado del género (~4.300 libros), descarta duplicados y toma una **muestra aleatoria de 150 libros** con semilla fija, para que el resultado sea representativo y reproducible.
- `Libro.py` define la estructura de cada registro.
- El resultado se guarda en `data/libros.csv`, con un campo por dato (título, autores, géneros, serie, sinopsis, URL, fecha de extracción).

El criterio de muestreo, los campos y las decisiones de limpieza están explicados en [diseno_extraccion.md](diseno_extraccion.md). La descripción de cada campo está en [docs/modelo_de_datos.md](../docs/modelo_de_datos.md).

## Archivos

| Archivo | Contenido |
|---|---|
| `scraper.py` | Scraper con Playwright |
| `Libro.py` | Clase que representa un libro |
| `diseno_extraccion.md` | Diseño y criterios de la extracción |
| `data/libros.csv` | Dataset resultante (150 libros) |

## Ejecución

Desde la raíz del repositorio, con el entorno activado:

```bash
playwright install chromium
python tp1-extraccion/scraper.py
```

Tarda varios minutos y sobrescribe `data/libros.csv`.
