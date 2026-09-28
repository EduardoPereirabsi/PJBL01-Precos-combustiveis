# PjBL — RA1: Ingestão e Preparação dos Dados

**Disciplina:** Data Analytics · PUCPR
**Tema:** Variação do preço de combustíveis nos postos brasileiros
**Fonte:** ANP — Levantamento de Preços de Combustíveis (Série Histórica)

> **Equipe:** _(preencher com os nomes dos integrantes)_

---

## O problema

O mesmo combustível, no mesmo dia, custa valores diferentes dependendo de onde o
consumidor abastece — inclusive entre postos da mesma cidade. Essa variação não é
transparente e não está descrita de forma estruturada.

**Pergunta central:** quais fatores estão associados à variação do preço dos
combustíveis entre os postos brasileiros?

As perguntas completas estão em [`specs/001-problema.md`](specs/001-problema.md).

## O que este repositório entrega

Este é o **RA1**: a etapa de obtenção, ingestão, avaliação de qualidade, limpeza e
preparação dos dados. A entrega é um **conjunto de dados confiável e documentado**,
não uma análise. As respostas às perguntas acima são o RA2 e o RA3.

## Estrutura

```
specs/                        A especificação — fonte da verdade do projeto
  000-constituicao.md           regras, escopo e rubrica
  001-problema.md               problema e perguntas analíticas
  002-fontes.md                 origem, formato, dicionário de dados e volume
  003-achados-qualidade.md      o que a inspeção encontrou, com números
  004-decisoes.md               problema | decisão | justificativa
  005-contrato-dados.md         schema e regras de validação do dataset final

plan/
  arquitetura.md                desenho do pipeline
  tasks.md                      backlog rastreado até a rubrica

src/                          implementação
  ingestao.py                   download reprodutível a partir da ANP
  preparacao.py                 uma função por decisão de 004
  validacao.py                  uma função por regra de 005

notebooks/
  ra1_preparacao.ipynb          o fluxo completo, de ponta a ponta

data/
  raw/                          como baixado — não versionado
  interim/                      consolidado — não versionado
  processed/                    dataset final validado
```

## Como reproduzir

Requer Python 3.11 ou superior.

```bash
git clone <url-do-repositorio>
cd pjbl-precos-combustiveis
python -m venv .venv
```

Ative o ambiente:

```bash
source .venv/Scripts/activate
```

Instale as dependências:

```bash
pip install -r requirements.txt
```

Execute a ingestão (baixa os arquivos da ANP para `data/raw/`):

```bash
python -m src.ingestao --inicio 2023-01 --fim 2026-08
```

Depois abra `notebooks/ra1_preparacao.ipynb` e execute todas as células. O
notebook termina imprimindo o resultado das 12 regras de validação — todas
precisam passar.

> O download é de aproximadamente 300 MB e fica em cache em `data/raw/`. Execuções
> seguintes não rebaixam os arquivos.

## Sobre os dados

Os CSVs brutos **não são versionados** por decisão de projeto: a reprodutibilidade
é garantida pelo código de ingestão e pelo manifesto (`data/raw/_manifesto.csv`),
que registra URL, tamanho, número de linhas e hash de cada arquivo baixado.

Detalhes da fonte, incluindo o dicionário das 16 colunas originais, estão em
[`specs/002-fontes.md`](specs/002-fontes.md).

## Método

O projeto segue o ciclo **INSPECT → DECIDE → TRANSFORM → VALIDATE**.

A consequência prática é que **nenhuma transformação existe no código sem uma
justificativa escrita** em [`specs/004-decisoes.md`](specs/004-decisoes.md), e
toda decisão é verificada automaticamente pelas regras de
[`specs/005-contrato-dados.md`](specs/005-contrato-dados.md).

Isso inclui as decisões de **não fazer**: a base não recebeu imputação de valores
ausentes nem remoção de outliers, porque a inspeção mostrou que esses problemas
não existem nela. A verificação está documentada mesmo sem achado.
