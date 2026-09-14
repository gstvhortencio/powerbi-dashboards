# Modelo de dados

O modelo tem uma tabela fato e seis dimensões. A maior parte segue **esquema estrela**, mas a hierarquia de produto é **floco de neve** (*snowflake*) — as dimensões se encadeiam em vez de ligarem direto à fato.

```
   dim_Calendario        dim_Geografia        dim_Representante
         │                     │                     │
         └──────────┬──────────┴──────────┬──────────┘
                    │                     │
                ┌───┴─────────────────────┴───┐
                │        fact_Vendas          │
                └──────────────┬──────────────┘
                               │
                          dim_Produto
                               │            ← ramo em floco de neve
                         dim_SubCategoria
                               │
                          dim_Categoria
```

## Relacionamentos

Todos muitos-para-um, com filtro em direção única (da dimensão para a fato).

| De | Para | Observação |
|---|---|---|
| `fact_Vendas[Data]` | `dim_Calendario[Data]` | Eixo de tempo |
| `fact_Vendas[Localização]` | `dim_Geografia[Localização]` | País e cidade |
| `fact_Vendas[RepresentanteID]` | `dim_Representante[RepresentanteID]` | Equipe comercial |
| `fact_Vendas[ProdutoID]` | `dim_Produto[ProdutoID]` | Produto |
| `dim_Produto[SubcategoriaID]` | `dim_SubCategoria[SubCategoriaKey]` | ⚠️ dimensão → dimensão |
| `dim_SubCategoria[CategoriaKey]` | `dim_Categoria[CategoriaKey]` | ⚠️ dimensão → dimensão |

As duas últimas são o que caracteriza o floco de neve: para filtrar a fato por Categoria, o mecanismo precisa propagar o filtro por dois saltos (`dim_Categoria` → `dim_SubCategoria` → `dim_Produto` → `fact_Vendas`) em vez de um.

**Alternativa** — achatar os três níveis em uma única `dim_Produto` com as colunas `NomeProduto`, `SubCategoria` e `Categoria`. Reduz os saltos de propagação, simplifica a escrita de medidas e é a recomendação padrão para modelos do Power BI. Neste volume de dados a diferença de desempenho é imperceptível; a questão é de manutenção, não de velocidade.

## Tabelas

| Tabela | Colunas | Papel |
|---|---:|---|
| `fact_Vendas` | 8 | Transações; base de todas as medidas |
| `dim_Produto` | 6 | Produto, preço de varejo, custo padrão |
| `dim_Calendario` | 5 | Eixo de tempo; sustenta YTD, MTD e MAT |
| `dim_Geografia` | 4 | País e cidade |
| `dim_SubCategoria` | 3 | Nível intermediário da hierarquia |
| `dim_Representante` | 2 | Nome do representante |
| `dim_Categoria` | 2 | Nível superior da hierarquia |

`dim_Produto` carrega `PrecoVarejo` e `CustoPadrao`, buscados por `RELATED` dentro dos iteradores de receita e custo — por isso as medidas base são `SUMX` sobre a fato e não somas simples de colunas.

## Por que uma tabela calendário dedicada

As funções de inteligência temporal (`TOTALYTD`, `TOTALMTD`, `SAMEPERIODLASTYEAR`, `DATESINPERIOD`) exigem uma tabela de datas contígua, sem lacunas, marcada como tabela de datas. Usar a coluna de data da própria fato não funciona: se não houver venda em um dia, aquele dia não existe na tabela, e o acumulado fica errado.

## Tabela de data automática

O modelo contém uma tabela `DateTableTemplate_7856f7f5-…`, gerada pelo recurso **Data/hora automática** do Power BI. Ele cria uma tabela de datas oculta para cada coluna de data do modelo — inclusive quando já existe uma `dim_Calendario` própria, como aqui.

O efeito é aumento do tamanho do arquivo sem benefício, já que nenhuma medida usa essas tabelas ocultas.

**Correção** — `Arquivo → Opções → Carregamento de Dados → desmarcar "Data/hora automática para novos arquivos"` (e também na seção do arquivo atual).
