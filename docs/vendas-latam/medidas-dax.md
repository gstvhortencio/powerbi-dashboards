# Medidas DAX

Dez medidas no modelo, todas na tabela `fact_Vendas`. As expressões abaixo foram extraídas do `model.bim` do projeto `.pbip` — são o código real, não uma reconstrução.

---

## Medidas base

### Receita Total

```dax
Receita Total =
SUMX(
    fact_Vendas,
    fact_Vendas[Unidades]
        * RELATED(dim_Produto[PrecoVarejo])
        * (1 - fact_Vendas[DescontoDeReceita])
)
```

Iterador linha a linha. `SUMX` percorre cada venda, calcula unidades × preço de varejo × (1 − desconto) e soma os resultados. Somar o produto é diferente de multiplicar as somas: com descontos variando por transação, só a iteração linha a linha dá o valor correto.

`RELATED` busca o preço em `dim_Produto` seguindo o relacionamento muitos-para-um — é o que permite usar uma coluna da dimensão dentro de um iterador sobre a fato.

**Formato:** `\$#,0.00;(\$#,0.00);\$#,0.00` — moeda, duas casas, negativos entre parênteses.

### Custo Total

```dax
Custo Total =
SUMX(
    fact_Vendas,
    fact_Vendas[Unidades]
        * RELATED(dim_Produto[CustoPadrao])
        * fact_Vendas[PercentualDoCustoPadrão]
)
```

Mesma estrutura, aplicada ao custo. O `PercentualDoCustoPadrão` por linha ajusta o custo padrão do produto ao custo efetivo daquela transação.

### Lucro Bruto

```dax
Lucro Bruto = [Receita Total] - [Custo Total]
```

Composição de medidas: reutiliza as duas anteriores em vez de repetir a iteração. Se a regra de receita mudar, muda em um lugar só.

### Margem Bruta

```dax
Margem Bruta = DIVIDE([Lucro Bruto], [Receita Total], 0)
```

`DIVIDE` em vez do operador `/`: o terceiro argumento define o retorno quando o denominador é zero. Com `/`, uma receita zerada produziria erro ou infinito na visualização.

---

## Inteligência temporal

Todas dependem de `dim_Calendario[Data]` como tabela de datas marcada.

### Receita YTD

```dax
Receita YTD = TOTALYTD([Receita Total], dim_Calendario[Data])
```

Acumulado do início do ano até a data do contexto atual.

### Receita YTD LY

```dax
Receita YTD LY =
CALCULATE([Receita YTD], SAMEPERIODLASTYEAR(dim_Calendario[Data]))
```

O mesmo acumulado, deslocado um ano para trás. Serve para comparação ano contra ano.

### Receita MTD

```dax
Receita MTD = TOTALMTD([Receita Total], dim_Calendario[Data])
```

Acumulado do mês.

### Receita MAT (últimos 12 meses)

```dax
Receita MAT (últimos 12 meses) =
CALCULATE(
    [Receita Total],
    DATESINPERIOD(dim_Calendario[Data], MAX(dim_Calendario[Data]), -12, MONTH)
)
```

*Moving Annual Total* — janela móvel de doze meses contados para trás a partir da última data do contexto. Ao contrário do YTD, não reinicia em janeiro, o que remove o degrau artificial na virada do ano.

---

## Ranking e participação

### Ranking de Vendedores

```dax
Ranking de Vendedores =
RANKX(ALL(dim_Representante[NomeRepresentante]), [Receita Total])
```

`ALL` remove o filtro da coluna de representante, criando a lista completa contra a qual cada um é classificado — sem isso, cada linha da tabela conteria só a si mesma e o ranking seria sempre 1. A ordem padrão de `RANKX` é decrescente, então a maior receita recebe a posição 1.

### Market Share % (por país)

```dax
Market Share % (por país) =
DIVIDE([Receita Total], CALCULATE([Receita Total], ALL(dim_Geografia)), 0)
```

Divide a receita do contexto atual pela receita com o filtro de geografia inteiramente removido. O numerador respeita o país selecionado; o denominador é sempre o total geral.

---

## Onde cada medida é usada

| Medida | Página | Visualização |
|---|---|---|
| `Receita Total` | Todas | Linha, colunas, tabelas, barras, área empilhada |
| `Receita YTD` | Visão Geral | Cartão |
| `Margem Bruta` | Visão Geral, Vendedores | Cartão, tabela |
| `Lucro Bruto` | Insights | Tabela |
| `Ranking de Vendedores` | Vendedores | Tabela |
| `Custo Total` | — | Usada apenas como componente de `Lucro Bruto` |
| `Receita YTD LY` | — | Não utilizada em nenhuma visualização |
| `Receita MTD` | — | Não utilizada em nenhuma visualização |
| `Receita MAT` | — | Não utilizada em nenhuma visualização |
| `Market Share %` | — | Não utilizada em nenhuma visualização |

As quatro últimas estão definidas mas não aparecem no relatório. Ver [revisão crítica](revisao-critica.md) sobre formatação e sobre medidas não utilizadas.
