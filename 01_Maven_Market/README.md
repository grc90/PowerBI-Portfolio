# Maven Market — Marketing Campaign Analysis

## Fuente y atribución

El dataset ([`data/raw/Marketing Campaign.xlsx`](./data/raw/Marketing%20Campaign.xlsx)) y el brief de proyecto original ([`docs/Project Brief - Marketing Campaign.pdf`](./docs/Project%20Brief%20-%20Marketing%20Campaign.pdf)) provienen de un caso de estudio público desarrollado originalmente por **Shahid Khan**:

- LinkedIn: https://www.linkedin.com/in/shahid-khan-791b78239/
- Kaggle: https://www.kaggle.com/shahidkhan01174
- Peerlist: https://peerlist.io/shahidkhan

Este proyecto retoma ese dataset para construir un modelo de datos y un análisis propio en Power BI, incluyendo una revisión crítica de algunas conclusiones del brief y análisis original (por ejemplo, re-evaluar qué campaña tuvo mejor desempeño usando tasas en vez de conteos brutos, y re-verificar si el canal Catálogo realmente está bajo rendimiento). Ver [`docs/business_questions.md`](./docs/business_questions.md).

## Objetivo

Analizar los datos de 2,240 clientes de una campaña de marketing — segmentación de clientes, rendimiento de canales y campañas, y desempeño de productos — para responder un set concreto de preguntas de negocio mediante un modelo de datos y dashboard en Power BI.

## Dataset

- **Fuente:** `data/raw/Marketing Campaign.xlsx`, hoja `marketing_data`.
- **Tamaño:** 2,240 clientes × 28 columnas.
- **Contenido:** demografía (edad, educación, estado civil, ingresos, país), gasto por categoría de producto (vinos, frutas, carne, pescado, dulces, oro), respuesta a 6 campañas de marketing, compras por canal (Web, Catálogo, Tienda).

## Modelo de datos

Esquema estrella/constelación propio, documentado con justificación de cada decisión de diseño:

- `Dim_Customer` (2,240 filas) — atributos demográficos y de comportamiento
- `Fact_Spend` (13,440 filas) — gasto por cliente × categoría de producto
- `Fact_Channel` (6,720 filas) — compras por cliente × canal
- `Fact_Campaign` (13,440 filas) — respuesta por cliente × campaña
- `Dim_Campaign` (6 filas), `Dim_Date`

Ver detalle completo en [`docs/data_model.md`](./docs/data_model.md).

## Limpieza y transformación de datos

- `Income` = 0 (24 filas) y outlier de $666,666 (1 fila) → recodificados a nulo
- `Year_Birth` implausible / edad 126+ (3 filas) → recodificado a nulo, fila retenida
- `Marital_Status` normalizado ("Alone" → "Single"; "YOLO"/"Absurd" → "Other")
- Resultado: 2,240 filas retenidas, 0 eliminadas

Ver detalle completo en [`docs/data_remediation.md`](./docs/data_remediation.md).

## Preguntas de negocio / páginas de dashboard planificadas

1. **Executive Summary** — KPIs generales: clientes totales, ingresos totales, tasa de respuesta a campañas, ingreso y edad promedio
2. **Customer Segmentation** — perfil demográfico, comportamiento de gasto por ingreso y presencia de hijos, segmentos más receptivos a campañas
3. **Campaign & Channel Performance** — campaña con mayor tasa de aceptación, canal con más compras por cliente, comportamiento de compradores con descuento
4. **Product Performance** — categorías con mayor gasto total y promedio, concentración de gasto por segmento

Ver detalle completo en [`docs/business_questions.md`](./docs/business_questions.md).

## KPIs o métricas

No verificables todavía: el archivo `.pbix` tiene el modelo de datos cargado, pero el reporte visual aún no tiene gráficos ni medidas construidas (una sola página en blanco).

## Herramientas y skills demostradas

- Power BI (modelado de datos)
- Modelado de datos (esquema estrella/constelación)
- Limpieza y transformación de datos
- Definición y priorización de preguntas de negocio

## Archivos relevantes

- `powerbi/maven_market.pbix` — archivo Power BI (modelo de datos cargado; reporte visual pendiente)
- `data/raw/Marketing Campaign.xlsx` — dataset original
- `docs/business_questions.md`, `docs/data_model.md`, `docs/data_remediation.md`
- `docs/Project Brief - Marketing Campaign.pdf` — brief original (Shahid Khan)

## Estado

🔧 Modelo de datos completo — dashboard visual en progreso.

Pendiente: medidas DAX (`docs/dax_measures.md`), diccionario de datos (`docs/data_dictionary.md`), conclusiones (`docs/conclusions.md`) y capturas de pantalla (`images/`) — actualmente vacíos, se completarán junto con el reporte visual.

## Cómo visualizarlo

Abrir `powerbi/maven_market.pbix` con [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (gratuito, Windows) para explorar el modelo de datos. Las capturas de pantalla se añadirán aquí cuando el reporte visual esté construido.
