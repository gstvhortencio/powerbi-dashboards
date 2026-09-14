# Power BI Dashboards

[![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-01B8AA)](https://learn.microsoft.com/dax/)
[![License](https://img.shields.io/badge/License-MIT-lightgrey.svg)](LICENSE)

Reports built in Power BI during the **Data Analyst program at EBAC**.

> 🇧🇷 Versão em português: [README.md](README.md)

Each dashboard ships the `.pbix` file to open and interact with, the `.pbip` project in plain text so Git can diff versions, captures of every page, and documentation of the model.

---

## Sales — Latin America

A three-page report on a pet-supplies sales operation across seven Latin American countries (2014–2017, $73.65M revenue and $35.66M gross profit), on a model of one fact table and six dimensions.

![Dashboard Overview page](vendas-latam/img/01-visao-geral.png)

### What the report answers

| Page | Question it answers |
|---|---|
| **Visão Geral** (Overview) | How did revenue evolve over time, and how is it split across countries? |
| **Vendedores** (Sales reps) | Who sells most, at what margin, and how does the team rank? |
| **Insights** | What do those numbers imply for a decision? |

The third page is the core of the work: instead of more charts, it carries six analytical readings of the model — portfolio concentration, margin behaviour across markets, per-country anomalies, and the 2017 break in trend.

### Key findings

**Concentration** — Argentina and Colombia account for 59% of cumulative revenue. Two single points of commercial failure.

**Uniform margin** — the spread between worst and best country is 1.8 pp (47.45% in Brazil, 49.26% in Peru). With no differentiated pricing power by market, the profit lever is volume, not price.

**Brazil is the outlier** — the region's largest economy, yet only 6% of revenue and the portfolio's worst margin.

**2017** — a +36% jump over a three-year plateau, spread across every product band. The pattern points to a systemic factor rather than one product's success.

> The data is fictional, supplied as course material. These readings describe associations in the dataset, not causal relationships.

### Model figures

| Country | City | Total revenue | Gross profit | Margin |
|---|---|---:|---:|---:|
| Argentina | Buenos Aires | $23,411,629 | $11,342,507 | 48.45% |
| Colombia | Bogotá | $20,100,937 | $9,764,608 | 48.58% |
| Mexico | Mexico City | $7,588,171 | $3,649,907 | 48.10% |
| Uruguay | Montevideo | $7,504,662 | $3,639,775 | 48.50% |
| Chile | Santiago | $7,492,387 | $3,620,384 | 48.32% |
| Brazil | São Paulo | $4,440,619 | $2,107,273 | 47.45% |
| Peru | Lima | $3,113,213 | $1,533,463 | 49.26% |
| **Total** | | **$73,651,618** | **$35,657,916** | **48.41%** |

**Products** — seven items (biscuit, toy, collar, brush, bone, pet food, clothing) across two categories (Generic, Special).

**Team** — seven reps, allocated not one-to-one with countries: Ellen Viana covers Colombia alone; Argentina is split between Elena da Silva and Bruno Dias (whose revenues sum to exactly the country's $23,411,629); and Marcia Perucci carries both Brazil and Peru (an exact sum of $7,553,832). That arithmetic is where insight 6 comes from.

### The pages

![Sales reps page](vendas-latam/img/02-vendedores.png)

![Insights page](vendas-latam/img/03-insights.png)

### Documentation

| Document | Contents |
|---|---|
| [Insights](docs/vendas-latam/insights.md) | The six analytical readings, with the method note |
| [Data model](docs/vendas-latam/modelo-de-dados.md) | Tables, relationships, and why the product branch is a snowflake |
| [DAX measures](docs/vendas-latam/medidas-dax.md) | All ten measures, with each expression explained |
| [Critical review](docs/vendas-latam/revisao-critica.md) | Seven open points with their corrections, and what the report gets right |

*(documentation is in Portuguese)*

### The measures

Ten measures, all on `fact_Vendas`:

| Group | Measures |
|---|---|
| Base | `Receita Total`, `Custo Total`, `Lucro Bruto`, `Margem Bruta` |
| Time intelligence | `Receita YTD`, `Receita YTD LY`, `Receita MTD`, `Receita MAT` |
| Ranking and share | `Ranking de Vendedores`, `Market Share % (por país)` |

The two base measures are iterators, not plain sums:

```dax
Receita Total =
SUMX(
    fact_Vendas,
    fact_Vendas[Unidades]
        * RELATED(dim_Produto[PrecoVarejo])
        * (1 - fact_Vendas[DescontoDeReceita])
)
```

Because the discount varies per transaction, summing the product row by row is the only correct calculation — multiplying the sums would give a different number.

### The model

One fact table and six dimensions. Geography, calendar and sales rep link straight to the fact (star); the product hierarchy chains across two hops (snowflake):

```
dim_Calendario   dim_Geografia   dim_Representante
       └───────────────┼───────────────┘
                  fact_Vendas
                       │
                  dim_Produto  →  dim_SubCategoria  →  dim_Categoria
```

### How to open it

Requires **Power BI Desktop** ([free download](https://powerbi.microsoft.com/desktop/), Windows).

```
1. Download vendas-latam/powerbi/dashboard-vendas.pbix
2. Open it in Power BI Desktop
3. The data is embedded in the model — no data source to connect
```

**Reading the model without Power BI** — `vendas-latam/powerbi/pbip/` holds the same report in project format, where everything is text:

| File | Contents |
|---|---|
| `dashboard-vendas.SemanticModel/model.bim` | Tables, columns, relationships and all ten DAX measures |
| `dashboard-vendas.Report/report.json` | Visuals, fields and the three pages' layout |

The data cache (`.pbi/`) was stripped, so that folder describes the model but carries no data — for the figures, use the `.pbix`.

---

## Structure

```
powerbi-dashboards/
├── README.md / README.en.md
├── LICENSE
├── .gitattributes             # marks .pbix as binary for Git
├── vendas-latam/
│   ├── img/
│   │   ├── 01-visao-geral.png
│   │   ├── 02-vendedores.png
│   │   └── 03-insights.png
│   └── powerbi/
│       ├── dashboard-vendas.pbix
│       └── pbip/
│           ├── dashboard-vendas.pbip
│           ├── dashboard-vendas.Report/
│           └── dashboard-vendas.SemanticModel/
└── docs/
    └── vendas-latam/
        ├── insights.md
        ├── modelo-de-dados.md
        ├── medidas-dax.md
        └── revisao-critica.md
```

Further reports get their own subfolder as the course progresses.

## On version-controlling Power BI files

A `.pbix` is a binary container (a `.zip` with the compressed model inside). Git stores it but cannot diff it: every save becomes a full copy in history. Hence the `.gitattributes` marking `*.pbix` as binary, which stops Git from attempting merges.

That is also why each dashboard ships **both**: the `.pbix` for anyone who wants to open and interact with it, and the `.pbip` so Git can show what changed between commits. Enable `.pbip` under `File → Options → Preview features → Power BI Project (.pbip) save option`.

## Author

**Gustavo Hortêncio da Silva** — Economist (UFPB), training as a Data Analyst at EBAC.

[![GitHub](https://img.shields.io/badge/GitHub-gstvhortencio-181717?logo=github&logoColor=white)](https://github.com/gstvhortencio)

📊 Looker Studio dashboards: [looker-studio-dashboards](https://github.com/gstvhortencio/looker-studio-dashboards)
📚 Course notes: [ebac-analise-de-dados](https://github.com/gstvhortencio/ebac-analise-de-dados)

## Licence

Released under the MIT licence. See [`LICENSE`](LICENSE).
