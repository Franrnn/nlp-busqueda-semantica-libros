# TP3 — Auditoría de un catálogo y recomendador personalizado

[← Volver al README principal](../README.md)

**Objetivo:** limpiar un catálogo de 8.747 libros que la cátedra contaminó a propósito, sin saber qué registros estaban alterados, y después recomendarle libros a un lector real según sus gustos.

## Qué hace

1. **Auditoría de calidad de datos.** Combina varios detectores: expresiones regulares, normalización de texto, detección de idioma, NER con spaCy, un clasificador de oraciones y búsqueda de casi duplicados. Se calibran con una muestra de 500 libros etiquetados.
2. **Valoración.** Un integrante del grupo valoró a mano más de 100 libros con `valorar.py`, un script de consola que guarda cada respuesta y permite cortar y retomar.
3. **Recomendador.** Representa cada sinopsis con embeddings de oración, aprende las preferencias del lector y recomienda libros del catálogo que no valoró, con un resumen automático de cada uno.

## Resultados

- **380 registros sospechosos** detectados; quedan 8.212 libros limpios.
- **De cada 5 libros recomendados, 2 o 3 le interesaban al lector**, frente a 1,5 eligiendo al azar (precision@5 de 0,4 a 0,6 contra 0,30).

El análisis completo está en el notebook y en [informe.pdf](informe.pdf).

## Archivos

| Archivo / carpeta | Contenido |
|---|---|
| `TP3_grupo11.ipynb` | Notebook con todo el desarrollo |
| `valorar.py` | Script para valorar los libros a mano (Paso 3) |
| `informe.pdf` | Informe del trabajo |
| `data/` | Entradas de la cátedra: catálogo y muestra etiquetada |
| `salidas/` | Resultados intermedios guardados para no recalcularlos |
| `sospechosos.csv`, `catalogo_limpio.csv`, `preferencias.csv` | Resultados del notebook |
| `grupo11_sospechosos.csv`, `grupo11_preferencias.csv` | Copias con los nombres que pedía la entrega |

## Ejecución

Ver [docs/ejecucion.md](../docs/ejecucion.md#tp3-notebook). La primera ejecución tarda unos 45 minutos; con los resultados guardados en `salidas/`, las siguientes tardan unos pocos.
