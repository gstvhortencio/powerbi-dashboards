# Revisão crítica do relatório

Avaliação do próprio trabalho: o que o relatório resolve bem e o que ainda tem a melhorar. Cada ponto aberto vem com a correção correspondente.

---

## Pontos em aberto

### 1. `Market Share % (por país)` sem formato percentual

```dax
Market Share % (por país) =
DIVIDE([Receita Total], CALCULATE([Receita Total], ALL(dim_Geografia)), 0)
```

A medida não tem `formatString` definido. Colocada em qualquer visualização, exibirá `0,3179` em vez de `31,79%` — o nome promete percentual e o valor entrega decimal.

**Correção** — Ferramentas de medida → Formato → Porcentagem, duas casas.

### 2. Formato inconsistente entre medidas monetárias

`Receita Total` e `Lucro Bruto` usam formato de moeda. `Custo Total`, `Receita YTD`, `Receita YTD LY`, `Receita MTD` e `Receita MAT` estão com `0.00` — número simples, sem cifrão nem separador de milhar.

No relatório atual isso passa despercebido: o único visual que usa uma delas é um cartão com unidade de exibição em milhões. Mas qualquer tabela que venha a usá-las vai exibir `22890123.45` ao lado de `$23.411.629,13`.

**Correção** — aplicar o mesmo formato de moeda a todas as medidas monetárias.

### 3. Quatro medidas definidas e não utilizadas

`Receita YTD LY`, `Receita MTD`, `Receita MAT` e `Market Share %` não aparecem em nenhuma visualização.

Não é um erro — pode ser preparação deliberada para uma próxima versão. Mas medida não utilizada é código morto: ninguém a testa, e ela envelhece junto com o modelo. Ou entram no relatório (uma comparação ano contra ano com `Receita YTD LY` seria um acréscimo natural à página *Visão Geral*), ou saem.

### 4. Tabela de data automática ligada

O modelo carrega uma `DateTableTemplate_…` gerada pelo recurso **Data/hora automática**, que cria uma tabela de datas oculta por coluna de data — mesmo já existindo uma `dim_Calendario` própria. Aumenta o tamanho do arquivo sem que nenhuma medida a utilize.

**Correção** — `Arquivo → Opções → Carregamento de Dados → desmarcar "Data/hora automática"`.

### 5. Hierarquia de produto em floco de neve

`dim_Categoria` → `dim_SubCategoria` → `dim_Produto` → `fact_Vendas`: dois saltos a mais de propagação de filtro do que um modelo estrela puro exigiria. Ver [modelo-de-dados.md](modelo-de-dados.md).

Neste volume de dados não há impacto perceptível de desempenho. É questão de manutenção — vale saber que existe e por que a recomendação padrão é achatar em uma única `dim_Produto`.

### 6. Rótulo "Receita YTD" sem período

O cartão mostra `22,89 Mi` sem indicar de que ano. Como a base termina em 2017, é o acumulado daquele ano — mas isso só é óbvio para quem conhece a base.

**Correção** — título dinâmico com a medida de ano, ou subtítulo fixo com o período coberto.

### 7. Eixo Y truncado no gráfico de receita mensal

O eixo da página *Visão Geral* vai de $5,4 Mi a $6,8 Mi, não do zero. A variação real entre o menor e o maior mês é de cerca de 24%; o gráfico a faz parecer uma montanha-russa.

Truncar o eixo em série temporal é defensável — partir do zero achataria a série e esconderia o padrão. Mas a escolha **amplifica a percepção de volatilidade**, e um leitor apressado tira uma conclusão mais forte do que os dados sustentam.

---

## O que está bem resolvido

**A página de Insights.** É o que separa este relatório de um exercício de formatação. A maioria dos dashboards de portfólio para na descrição do gráfico; este chega à implicação — risco de concentração, ausência de poder de precificação, anomalia brasileira.

**O insight 6 é aritmeticamente verificável.** A afirmação de que Marcia Perucci cobre Brasil e Peru enquanto a Argentina tem dois vendedores não é impressão: a receita de Marcia ($7.553.832,3735) é a soma exata de Brasil ($4.440.619,2765) e Peru ($3.113.213,097), e as receitas de Elena da Silva e Bruno Dias somam exatamente a da Argentina ($23.411.629,1315).

**As medidas base usam `SUMX`, não `SUM`.** Com desconto variando por transação, somar o produto linha a linha é o único cálculo correto — multiplicar as somas daria outro número. A escolha demonstra entendimento de contexto de linha, que é onde a maioria dos iniciantes em DAX tropeça.

**`DIVIDE` em vez de `/`.** Tratamento explícito de divisão por zero nas duas medidas de razão.

**Escolha de visual coerente com o dado.** Ranking de vendedores em barras ordenadas e produtos em colunas — comprimento, que o olho compara bem — reservando a rosca para a única comparação de duas categorias, onde ela funciona.

**A distinção entre observação e hipótese.** O insight 5 aponta o salto de 2017 e oferece candidatos a explicação (câmbio, reajuste, novo canal) sem afirmar qual é — postura correta, já que o modelo não contém as variáveis para testar nenhuma delas.
