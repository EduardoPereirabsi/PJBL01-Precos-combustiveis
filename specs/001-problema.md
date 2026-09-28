# 001 — Problema e perguntas analíticas

> **SPECIFY** · Atende ao critério **"Problema e perguntas analíticas" (20%)**
> e ao item 1 do enunciado.
> ⚠️ Rascunho para a equipe revisar — as perguntas precisam ser **de vocês**,
> porque cada integrante vai defendê-las na apresentação.

## Contexto

O preço final da gasolina, do etanol e do GLP no Brasil não é definido por um
único agente. Ele combina custo de produção, tributação estadual, custo de
distribuição, política de bandeira e concorrência local. O resultado é que
**o mesmo produto, no mesmo dia, custa valores diferentes** dependendo de onde
o consumidor abastece — inclusive entre postos da mesma cidade.

A ANP acompanha isso desde 2004 através do *Levantamento de Preços de
Combustíveis*, uma pesquisa presencial que registra o preço praticado por posto,
com data, endereço e bandeira.

## Problema

> Existe dispersão relevante no preço dos combustíveis entre localidades e entre
> postos, mas essa variação não é transparente para o consumidor nem está
> descrita de forma estruturada. **Não se sabe, a partir do dado bruto, quanto da
> diferença de preço está associada a região, bandeira ou concorrência local.**

## Perguntas analíticas

### Pergunta central
> **Quais fatores estão associados à variação do preço dos combustíveis entre os
> postos brasileiros?**

### Perguntas de apoio

| # | Pergunta | Tipo |
|---|---|---|
| 1 | Qual a diferença de preço médio entre regiões e estados para o mesmo produto e período? | Descritivo |
| 2 | Postos de bandeira **BRANCA** praticam preços diferentes dos bandeirados? | Diagnóstico |
| 3 | Qual a **dispersão de preço dentro de um mesmo município** — ou seja, quanto o consumidor economiza escolhendo o posto? | Diagnóstico |
| 4 | O etanol é vantajoso frente à gasolina (regra dos 70%) e em quais estados? | Diagnóstico |
| 5 | Como os preços evoluíram entre 2023 e 2026, e os aumentos chegaram igualmente a todas as regiões? | Descritivo |

> As perguntas 3 e 4 são as mais fortes: têm resposta objetiva, significado
> prático para o consumidor e exigem tratamento real do dado.

## Classificação (slide 18 da professora)

| Item | Resposta |
|---|---|
| **Contexto** | Economia / consumo / energia |
| **Tipo de dado** | **Estruturado** — CSV tabular, schema fixo de 16 colunas |
| **Tipo de analytics** | **Descritivo** (perguntas 1 e 5) e **Diagnóstico** (2, 3 e 4) |
| **Evolução possível** | Preditivo no RA3 (projeção de preço por região) |

## Por que esta base atende ao checklist do slide 19

| Critério | Situação |
|---|---|
| Relacionada à pergunta | ✅ o preço é a própria variável de interesse |
| Variáveis compreensíveis | ✅ 16 colunas de significado direto |
| Dados suficientes | ✅ ~50 mil registros/mês só em gasolina e etanol |
| **Tem data/hora** | ✅ `Data da Coleta` |
| **Exige transformação** | ✅ confirmado por medição — ver `003-achados-qualidade.md` |
| Serve ao semestre | ✅ cada coleta é um evento datado → streaming no RA2 |

## Fora de escopo neste RA1

Responder às perguntas acima. O RA1 entrega **a base confiável para respondê-las**.
As respostas são RA2 e RA3.
