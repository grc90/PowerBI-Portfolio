# Power BI Portfolio — Gregorio Ruiz

Portfolio de proyectos de análisis de datos y Business Intelligence, centrado en Power BI.
*Data analysis & Business Intelligence portfolio, focused on Power BI.*

## Sobre mí / About me

**ES** — Analista con background en marketing y más de una década transformando datos en decisiones. Experiencia construyendo dashboards, automatizando reportes y midiendo KPIs (ROI, CAC, CPL) con Excel, Power BI, Looker Studio y SQL. Orientado a resultados, con foco en storytelling de datos para negocio.

**EN** — Analyst with a marketing background and over a decade turning data into decisions. Experienced in building dashboards, automating reports, and tracking KPIs (ROI, CAC, CPL) using Excel, Power BI, Looker Studio, and SQL. Results-driven, with a focus on data storytelling for business.

**Ubicación / Location:** Barcelona, España. Disponibilidad presencial e híbrido en Barcelona, remoto en EMEA. · *Available on-site/hybrid in Barcelona, remote across EMEA.*

## Estado del portfolio / Portfolio status

**ES** — Este repositorio está en fase de construcción. El primer proyecto (Maven Market) ya tiene modelo de datos, limpieza documentada y análisis de negocio; el dashboard visual todavía se está construyendo. Los demás proyectos son carpetas reservadas sin contenido aún. Este README se irá actualizando a medida que cada proyecto avance.

**EN** — This repository is a work in progress. The first project (Maven Market) already has a data model, documented data cleaning, and business analysis; the visual dashboard is still being built. The other projects are placeholder folders with no content yet. This README will be updated as each project progresses.

## Proyectos / Projects

| Proyecto | Carpeta | Estado | Notas |
|---|---|---|---|
| Maven Market | [`01_Maven_Market/`](./01_Maven_Market/) | 🔧 Modelo de datos completo — dashboard visual en progreso | Modelado de datos, limpieza y preguntas de negocio documentados; visuales del reporte aún no construidos en el `.pbix` |
| Customer Churn | [`02_Customer_Churn/`](./02_Customer_Churn/) | 📁 Reservado | Carpeta creada, sin contenido aún |
| Supply Chain | [`03_Supply_Chain/`](./03_Supply_Chain/) | 📁 Reservado | Carpeta creada, sin contenido aún |
| Marketing Analytics | [`04_Marketing_Analytics/`](./04_Marketing_Analytics/) | 📁 Reservado | Carpeta creada, sin contenido aún |

---

## 01 — Maven Market

- **Objetivo:** Analizar los datos de 2,240 clientes de una campaña de marketing (segmentación, rendimiento de canales y campañas, y desempeño de productos) para responder un set concreto de preguntas de negocio mediante un modelo de datos y dashboard en Power BI.
- **Fuente / atribución:** El dataset (`data/raw/Marketing Campaign.xlsx`) y el brief original ([`docs/Project Brief - Marketing Campaign.pdf`](./01_Maven_Market/docs/Project%20Brief%20-%20Marketing%20Campaign.pdf)) provienen de un caso de estudio público desarrollado originalmente por **Shahid Khan** ([LinkedIn](https://www.linkedin.com/in/shahid-khan-791b78239/), [Kaggle](https://www.kaggle.com/shahidkhan01174)). Este proyecto retoma ese dataset para construir un modelo de datos y análisis propio en Power BI, incluyendo una revisión crítica de algunas conclusiones del brief original (ver `docs/business_questions.md`).
- **Dataset:** 2,240 clientes × 28 columnas (hoja `marketing_data`), con demografía, gasto por categoría de producto, respuesta a 6 campañas y compras por canal (Web/Catálogo/Tienda).
- **Modelo de datos:** Esquema estrella/constelación propio — `Dim_Customer`, `Fact_Spend`, `Fact_Channel`, `Fact_Campaign`, `Dim_Campaign`, `Dim_Date` — con justificación de cada decisión de diseño. Ver [`docs/data_model.md`](./01_Maven_Market/docs/data_model.md).
- **Limpieza de datos:** Nulos en `Income` (24 filas en 0 + 1 outlier de $666,666), `Year_Birth` implausible (3 filas), normalización de `Marital_Status`. 2,240 filas retenidas, 0 eliminadas. Ver [`docs/data_remediation.md`](./01_Maven_Market/docs/data_remediation.md).
- **Preguntas de negocio / páginas de dashboard planificadas:** Executive Summary, Customer Segmentation, Campaign & Channel Performance, Product Performance. Ver [`docs/business_questions.md`](./01_Maven_Market/docs/business_questions.md).
- **KPIs o métricas:** No verificables todavía — el reporte visual del `.pbix` aún no tiene gráficos ni KPIs construidos (solo el modelo de datos).
- **Herramientas y skills demostradas:** Power BI (modelado de datos), modelado de datos (esquema estrella), limpieza y transformación de datos, definición de preguntas de negocio.
- **Archivos relevantes:** [`01_Maven_Market/`](./01_Maven_Market/) — `powerbi/maven_market.pbix`, `data/raw/Marketing Campaign.xlsx`, `docs/`
- **Estado:** 🔧 Modelo de datos completo — dashboard visual en progreso (el `.pbix` tiene el modelo cargado pero el reporte todavía tiene una sola página en blanco, sin visuales).
- **Cómo visualizarlo:** Abrir `01_Maven_Market/powerbi/maven_market.pbix` con [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (gratuito, Windows) para explorar el modelo de datos. Las capturas de pantalla se añadirán a `images/` cuando el reporte visual esté construido.

## Próximos proyectos / Upcoming projects

`02_Customer_Churn`, `03_Supply_Chain` y `04_Marketing_Analytics` son carpetas reservadas para futuros proyectos. Aún no contienen archivos de trabajo, por lo que no se describe su alcance hasta tener contenido real.

*`02_Customer_Churn`, `03_Supply_Chain`, and `04_Marketing_Analytics` are placeholder folders for upcoming projects. They don't contain any working files yet, so their scope isn't described until there's real content to show.*

## Skills

Herramientas y competencias confirmadas por mí para este portfolio:

- Power BI
- Excel
- Looker Studio
- SQL
- Modelado de datos (esquema estrella/constelación)
- Limpieza y transformación de datos
- Construcción de dashboards
- Automatización de reportes
- Análisis y seguimiento de KPIs (ROI, CAC, CPL)
- Storytelling de datos para negocio

> Nota: DAX y Power Query se documentarán como skills confirmadas cuando el reporte visual y las medidas del proyecto Maven Market estén construidos en el `.pbix` (actualmente solo el modelo de datos está cargado).

## Cómo visualizar los proyectos / How to view the projects

**ES** — Los proyectos usan archivos `.pbix` de Power BI. Para abrirlos se necesita [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (gratuito, disponible para Windows). Mientras los archivos de cada proyecto se completan, se irán añadiendo capturas de pantalla en la carpeta `images/` de cada proyecto como alternativa de visualización rápida sin necesidad de instalar Power BI.

**EN** — Projects use Power BI `.pbix` files. Opening them requires [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free, Windows only). As each project's files are completed, screenshots will be added to its `images/` folder as a quick way to view the work without installing Power BI.

## Contacto / Contact

- **Email:** grc90.rc@gmail.com
- **LinkedIn:** [linkedin.com/in/gregorioruiz](https://www.linkedin.com/in/gregorioruiz/)
- **GitHub:** [github.com/grc90](https://github.com/grc90)
- **Ubicación:** Barcelona, España — presencial/híbrido en Barcelona, remoto en EMEA
