# NLP: extracción de texto, búsqueda semántica y recomendador de libros

Tres trabajos encadenados de **Procesamiento de Lenguaje Natural** (Tecnicatura Universitaria en Inteligencia Artificial, FCEIA, UNR). Cada uno usa lo que produjo el anterior: el TP1 arma un dataset de libros, el TP2 busca sobre él por significado y el TP3 lleva lo aprendido a un catálogo grande para recomendar libros a un lector.

**Equipo:** Franco Renna · Micaela Gómez · Irma Carolina Ruiz · Franco Esparza

## Proyectos

| Trabajo | Qué hace | Resultado |
|---|---|---|
| [**TP1 — Extracción de texto**](tp1-extraccion/) | Scraper con Playwright que recorre la categoría *Ciencia ficción* de Lectulandia y arma un dataset con metadatos y sinopsis | Dataset de 150 libros, muestreado al azar sobre ~4.300 y documentado campo por campo |
| [**TP2 — Búsqueda semántica**](tp2-busqueda-semantica/) | Compara TF-IDF, Word2Vec y SBERT para buscar libros por significado; guarda los vectores en Postgres con pgvector | Evaluado con 18 consultas propias. Dividir los textos en fragmentos (chunking) hizo que una consulta difícil pasara de 0 a 2 libros relevantes en el top 5 |
| [**TP3 — Auditoría y recomendador**](tp3-recomendador/) | Audita un catálogo de 8.747 libros contaminado a propósito y aprende las preferencias de un lector para recomendarle libros | 380 registros sospechosos detectados. De cada 5 recomendaciones, 2 o 3 le interesaban al lector, frente a 1,5 al azar |

## Stack

Python · Playwright · spaCy · gensim (Word2Vec) · sentence-transformers (SBERT) · scikit-learn · PostgreSQL + pgvector (Supabase) · NLTK · langdetect

## Estructura

```
├── tp1-extraccion/           scraper, diseño de la extracción y dataset
├── tp2-busqueda-semantica/   notebook, consultas de evaluación, informe y vectores
├── tp3-recomendador/         notebook, script de valoración, informe, datos y resultados
├── docs/
│   ├── ejecucion.md          instalación y cómo correr cada trabajo (local y Colab)
│   └── modelo_de_datos.md    campos de cada archivo y tablas de la base
├── requirements.txt          dependencias de los tres trabajos
└── .env.example              plantilla para la conexión a la base (TP2)
```

## Cómo ejecutarlo

Requiere Python 3.12 o 3.13:

```bash
python -m venv .venv-NLP
.venv-NLP\Scripts\activate        # Windows (en Linux/macOS: source .venv-NLP/bin/activate)
pip install -r requirements.txt
```

Las instrucciones completas de cada trabajo (modelos a descargar, conexión a la base, ejecución en Google Colab) están en [docs/ejecucion.md](docs/ejecucion.md).

Desarrollado con asistencia de Claude Code.

---

**English:** Three chained NLP projects (AI degree, Universidad Nacional de Rosario, Argentina): a Playwright scraper that builds a 150-book dataset; semantic search comparing TF-IDF, Word2Vec and SBERT with vectors stored in PostgreSQL + pgvector; and a data-quality audit of an 8,747-book catalog (380 suspicious records flagged) followed by a personalized embedding-based book recommender.
