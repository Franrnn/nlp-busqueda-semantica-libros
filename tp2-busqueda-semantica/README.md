# TP2 — Embeddings y búsqueda semántica

[← Volver al README principal](../README.md)

**Objetivo:** encontrar libros por el significado de su sinopsis, no solo por las palabras que comparten con la consulta, y medir qué representación funciona mejor.

## Qué hace

1. **Preprocesa** las 150 sinopsis del TP1 en dos versiones: una limpia (para TF-IDF) y una cruda (para SBERT).
2. **Compara cuatro representaciones:** TF-IDF, Word2Vec entrenado sobre el corpus, Word2Vec pre-entrenado (SBW) y SBERT.
3. **Guarda los vectores en PostgreSQL con pgvector** (Supabase), con un índice HNSW, y busca por similitud con consultas SQL.
4. **Evalúa** los modelos con un conjunto propio de 18 consultas en lenguaje natural (`queries.json`), incluidas dos diseñadas para no compartir ninguna palabra con sus libros relevantes.
5. **Parte avanzada:** divide las sinopsis largas en fragmentos (chunking) para no perder el texto que SBERT trunca.

## Resultados

- El chunking hizo que una consulta difícil pasara de 0 a 2 libros relevantes entre los 5 primeros resultados (precision@5 de 0 a 0,4).

El análisis completo está en el notebook y en [informe.pdf](informe.pdf).

## Archivos

| Archivo | Contenido |
|---|---|
| `TP2_Renna_Gomez_Ruiz_Esparza.ipynb` | Notebook con todo el desarrollo |
| `queries.json` | 18 consultas de evaluación con sus libros relevantes |
| `informe.pdf` | Informe del trabajo |
| `data/` | Sinopsis preprocesadas, vectores y fragmentos |

## Ejecución

Ver [docs/ejecucion.md](../docs/ejecucion.md#tp2-notebook). La conexión a la base no está en el repositorio: sin ella, el notebook corre igual y omite las celdas que la usan.
