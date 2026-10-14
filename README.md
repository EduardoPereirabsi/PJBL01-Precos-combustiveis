# PjBL — Preços de Combustíveis no Brasil

Projeto da disciplina de **Data Analytics** (PUCPR).

**Equipe:** Eduardo Pereira, Patrick de Oliveira, Roberto Zhou, Matheus Fraiz de Camargo Antonello

## Contexto

O preço de um mesmo combustível muda bastante de um posto para outro, mesmo dentro da mesma cidade e no mesmo dia. Essa diferença depende de vários fatores: imposto estadual, custo de transporte, bandeira do posto e concorrência local.

A ANP acompanha esses preços pelo *Levantamento de Preços de Combustíveis*, uma pesquisa feita em postos de todo o país. Neste projeto usamos esses dados para estudar a variação de preço entre os postos brasileiros.

## Problema e perguntas

**Pergunta central:** quais fatores estão associados à variação do preço dos combustíveis entre os postos brasileiros?

Perguntas que pretendemos responder nas próximas etapas:

1. Qual é a diferença de preço entre regiões e estados para o mesmo produto?
2. Postos sem bandeira (bandeira branca) cobram preços diferentes dos postos com bandeira?
3. Quanto o preço varia entre postos de uma mesma cidade?
4. Em quais estados o etanol compensa mais que a gasolina (regra dos 70%)?
5. Como os preços evoluíram de 2023 a 2026?

As perguntas 1 e 5 são **descritivas** (o que aconteceu) e as perguntas 2, 3 e 4 são **diagnósticas** (o que está associado ao preço).

## Fonte dos dados

| | |
|---|---|
| Fonte | ANP — Levantamento de Preços de Combustíveis (Série Histórica) |
| Link | https://www.gov.br/anp/pt-br/centrais-de-conteudo/dados-abertos/serie-historica-de-precos-de-combustiveis |
| Formato | CSV, um arquivo por mês para cada grupo de produtos (129 arquivos) |
| Produtos | Gasolina, gasolina aditivada, etanol, diesel, diesel S10, GNV e GLP |
| Período | janeiro/2023 a agosto/2026 (43 meses; abril/2026 não foi publicado pela ANP) |
| Registros | 3.717.753 recebidos · 3.715.016 após a preparação |
| Tipo de dado | Estruturado |

## O que este repositório entrega

Todo o desenvolvimento está no notebook `PJBL_precos_combustiveis.ipynb`, organizado no template da disciplina:

- **RA1 — Ingestão, limpeza e análise inicial:** download dos 129 arquivos da ANP, exploração, limpeza com decisões justificadas, validação e transformação dos dados. O resultado é a base `data/processed/precos_combustiveis.parquet`.
- **RA2 — Streaming e janelas temporais:** simulação da chegada de novas coletas de preço da gasolina em Curitiba, processadas com Spark Structured Streaming e agregadas em janelas Tumbling de 1 hora (postos pesquisados, preço médio, menor e maior preço).

Principais problemas encontrados e tratados (os detalhes e as justificativas estão no notebook):

- 2.713 linhas totalmente vazias no fim de dois arquivos de janeiro/2023;
- 16 linhas duplicadas e um posto com dois preços diferentes para o mesmo produto no mesmo dia;
- preço e data guardados como texto, com vírgula decimal e data no formato `dd/mm/aaaa`;
- a coluna de preço mistura três unidades de medida (litro, m³ e botijão de 13 kg);
- a mesma distribuidora aparece com nomes diferentes ao longo do tempo (ex.: `VIBRA ENERGIA` e `VIBRA`);
- a coluna `Valor de Compra` está vazia em todo o período.


## Como executar

O notebook baixa os dados direto do site da ANP, então não é preciso baixar nada manualmente.

**No Google Colab:** abra o notebook pelo GitHub (`Arquivo → Abrir notebook → GitHub`, colando o link deste repositório) e execute todas as células. A seção do RA2 é independente: para demonstrar só o streaming, basta executar a partir da seção 8.

**Localmente:**

```bash
git clone https://github.com/EduardoPereirabsi/PJBL01-Precos-combustiveis.git
cd PJBL01-Precos-combustiveis
pip install -r requirements.txt
jupyter lab PJBL_precos_combustiveis.ipynb
```

Na primeira execução o download leva alguns minutos (cerca de 630 MB). Nas seguintes, os arquivos já baixados são reaproveitados. O RA2 usa PySpark, que precisa de Java 17 ou superior (o Colab já tem).

## Estrutura

```
PJBL_precos_combustiveis.ipynb         notebook com o RA1 e o RA2
data/raw/                              arquivos baixados da ANP (não vão para o GitHub)
data/controle_ingestao.csv             lista dos arquivos baixados, URL de origem e número de linhas
data/processed/                        base preparada
requirements.txt                       bibliotecas usadas
```
