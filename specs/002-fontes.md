# 002 — Fontes de dados

> **SPECIFY** · Atende ao critério **"Compreensão dos dados" (20%)** e à
> **"Ingestão e reprodutibilidade" (15%)**. Corresponde ao item 2 do enunciado.

## Fonte única

| Item | Descrição |
|---|---|
| **Nome** | Levantamento de Preços de Combustíveis — Série Histórica |
| **Órgão** | ANP — Agência Nacional do Petróleo, Gás Natural e Biocombustíveis |
| **Página** | https://www.gov.br/anp/pt-br/centrais-de-conteudo/dados-abertos/serie-historica-de-precos-de-combustiveis |
| **Licença** | Dados abertos do governo federal |
| **Formato** | CSV, separador `;`, decimal `,`, **UTF-8 com BOM** |
| **Granularidade** | 1 linha = 1 coleta de preço em 1 posto, em 1 data, para 1 produto |
| **Natureza** | Dado **estruturado** — schema fixo de 16 colunas |

### O que o dado é, de fato

A ANP não coleta o preço de todos os postos todos os dias. É uma **pesquisa
amostral presencial**: pesquisadores visitam postos selecionados e registram o
preço da bomba. Em junho/2025 houve **22 datas distintas** de coleta no mês.

> Isso tem consequência analítica: a base **não é um censo de preços**. Qualquer
> conclusão é sobre a amostra pesquisada, não sobre o mercado inteiro. Isso
> precisa estar dito na apresentação.

## Organização dos arquivos

Um arquivo por **mês** × **grupo de produto**:

| Grupo | Produtos contidos |
|---|---|
| `gasolina-etanol` | GASOLINA, GASOLINA ADITIVADA, ETANOL |
| `diesel-gnv` | DIESEL, DIESEL S10, GNV |
| `glp` | GLP (botijão de 13 kg) |

### Padrão de URL — muda ao longo do tempo ⚠️

```
Base: https://www.gov.br/anp/pt-br/centrais-de-conteudo/dados-abertos/arquivos/shpc/dsan

2023–2025:  /<ano>/precos-<grupo>-<MM>.csv
2026 (a):   /2026/<MM>-dados-abertos-precos-<grupo>.csv
2026 (b):   /2026/<MM>-dados-abertos-precos-2026-<MM>-<grupo>.csv
2026 (c):   /2026/02-cados-abertos-preco-gasolina-etanol.csv   ← com erro de digitação na origem
```

São **três padrões diferentes** mais um arquivo com typo (`cados` em vez de
`dados`, `preco` em vez de `precos`). A ingestão não pode assumir um padrão
único — ver `plan/arquitetura.md`.

### Acesso

O servidor da ANP responde **403 Forbidden** para requisições sem cabeçalho de
navegador. A ingestão precisa enviar `User-Agent` e `Referer`. Verificado em
2026-09-23.

## Volume

Medições reais (junho/2025):

| Grupo | Registros no mês |
|---|---|
| gasolina-etanol | 49.976 |
| diesel-gnv | 22.363 |
| glp | 11.460 |
| **Total/mês** | **≈ 83.800** |

Período proposto: **jan/2023 a ago/2026** = 44 meses, **menos abril/2026**
(ausente na origem) = **43 meses**.

> **Volume estimado: ≈ 3,6 milhões de registros.**
> Suficiente para justificar processamento distribuído no RA2.

## Dicionário de dados (16 colunas)

| # | Coluna original | Significado | Relevância |
|---|---|---|---|
| 1 | `Regiao - Sigla` | Macrorregião (N, NE, CO, SE, S) | **Alta** — agregação geográfica |
| 2 | `Estado - Sigla` | UF | **Alta** — tributação é estadual |
| 3 | `Municipio` | Município (sem acento, caixa alta) | **Alta** — concorrência é local |
| 4 | `Revenda` | Razão social do posto | Média — identifica redes |
| 5 | `CNPJ da Revenda` | CNPJ do posto | **Alta** — chave do posto |
| 6 | `Nome da Rua` | Logradouro | Baixa |
| 7 | `Numero Rua` | Número | Baixa |
| 8 | `Complemento` | Complemento | Nenhuma — 78% vazio |
| 9 | `Bairro` | Bairro | Baixa |
| 10 | `Cep` | CEP | Média — poderia refinar a geografia |
| 11 | `Produto` | Combustível pesquisado | **Alta** — 7 produtos |
| 12 | `Data da Coleta` | Data da pesquisa (`dd/mm/aaaa`) | **Alta** — eixo temporal / RA2 |
| 13 | `Valor de Venda` | Preço ao consumidor | **Alta** — variável de interesse |
| 14 | `Valor de Compra` | Preço de aquisição | Nenhuma — **100% vazio** |
| 15 | `Unidade de Medida` | Unidade do preço | **Crítica** — 3 unidades diferentes |
| 16 | `Bandeira` | Distribuidora (ou BRANCA) | **Alta** — pergunta 2 |

### Produtos e escalas de preço (junho/2025)

| Produto | Unidade | Registros | Mín | Mediana | Máx |
|---|---|---|---|---|---|
| ETANOL | R$ / litro | 16.092 | 3,19 | 4,39 | 6,49 |
| GASOLINA | R$ / litro | 19.122 | 4,99 | 6,19 | 8,99 |
| GASOLINA ADITIVADA | R$ / litro | 14.762 | 5,29 | 6,39 | 9,29 |
| DIESEL | R$ / litro | 7.659 | 5,29 | 5,98 | 8,19 |
| DIESEL S10 | R$ / litro | 13.016 | 5,19 | 5,99 | 8,99 |
| GNV | **R$ / m³** | 1.688 | 3,99 | 4,72 | 6,49 |
| GLP | **R$ / 13 kg** | 11.460 | 79,99 | 110,00 | 156,00 |

> ⚠️ **A coluna `Valor de Venda` mistura três unidades incompatíveis.** Calcular
> média sem separar por unidade produz um número sem significado. Este é o achado
> de qualidade mais importante da base — ver `003-achados-qualidade.md`.

## Relação com o problema

A base contém, na mesma linha, **o preço** e os três fatores que a pergunta
central investiga: **onde** (região/UF/município), **quem** (bandeira) e
**quando** (data da coleta). É o dado mínimo necessário e suficiente para as
perguntas 1 a 5 de `001-problema.md`, sem precisar de fonte externa.

## Fonte complementar (opcional, decidir depois)

Se a equipe quiser explicar a diferença entre estados, a alíquota de ICMS por UF
seria a fonte natural. **Fica fora do RA1** — só entra se o RA2 exigir.
