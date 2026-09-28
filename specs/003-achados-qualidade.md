# 003 — Achados de qualidade (INSPECT)

> **EXPLORE** · Atende ao critério **"Qualidade e preparação" (35%)** — metade
> dele. A outra metade é `004-decisoes.md`. Corresponde ao item 4 do enunciado.

## Como estes números foram obtidos

Amostra inspecionada em **2026-09-23**, com leitura direta dos CSVs:

| Arquivo | Registros |
|---|---|
| `gasolina-etanol` 2023-06 | 49.041 |
| `gasolina-etanol` 2025-06 | 49.976 |
| `gasolina-etanol` 2026-08 | 51.856 |
| `diesel-gnv` 2025-06 | 22.363 |
| `glp` 2025-06 | 11.460 |

Salvo indicação em contrário, os percentuais referem-se a `gasolina-etanol`
2025-06 (49.976 linhas). **Estes números precisam ser reproduzidos pelo notebook
sobre o período completo antes da entrega** — ver `plan/tasks.md`.

---

## A. Problemas encontrados

Classificados pelas categorias do slide 8 da professora.

### A1 — `Valor de Venda` mistura três unidades de medida
**Categoria:** valores inválidos / formatos inconsistentes · **Gravidade: alta**

| Unidade | Registros (jun/2025) | Faixa de preço |
|---|---|---|
| `R$ / litro` | 70.651 | 3,19 – 9,29 |
| `R$ / m³` (GNV) | 1.688 | 3,99 – 6,49 |
| `R$ / 13 kg` (GLP) | 11.460 | 79,99 – 156,00 |

Por que é problema: os valores estão na **mesma coluna numérica** mas em escalas
que diferem em até 50×. Qualquer média, soma ou comparação que não segmente por
unidade produz um número sem significado — e o erro é silencioso, não gera
exceção. É o achado mais importante da base.

### A2 — Arquivo em UTF-8 **com BOM**
**Categoria:** formato inconsistente · **Gravidade: alta**

O byte order mark (`﻿`) precede o cabeçalho. Lido com `encoding="utf-8"`,
a primeira coluna passa a se chamar `﻿Regiao - Sigla` e **qualquer seleção
por nome falha**. Confirmado nos 5 arquivos inspecionados.

### A3 — Decimal com vírgula
**Categoria:** tipo de dado incorreto · **Gravidade: alta**

`Valor de Venda` vem como `6,02`. Sem `decimal=","`, o pandas infere `object` e
a coluna de interesse do projeto vira **texto**.

### A4 — `CNPJ da Revenda` com espaço à esquerda em 100% das linhas
**Categoria:** formato inconsistente · **Gravidade: média**

Valor bruto: `' 03.659.452/0006-82'`. Afeta **49.976 de 49.976** linhas. Como o
CNPJ é a chave natural do posto, o espaço quebra agrupamento e junção.
Distintos após limpeza: **5.645 postos**.

### A5 — `Valor de Compra` 100% vazia
**Categoria:** valores ausentes · **Gravidade: média**

Vazia em **todos** os arquivos e produtos verificados (2023, 2025, 2026;
gasolina, diesel e GLP). Não é ausência parcial — é **coluna morta**.

### A6 — `Complemento` 78,1% vazia
**Categoria:** valores ausentes · **Gravidade: baixa**

39.020 de 49.976 vazias. Campo de texto livre sem valor analítico.

### A7 — `Numero Rua` é texto em campo aparentemente numérico
**Categoria:** tipo de dado incorreto · **Gravidade: baixa**

**6.164 linhas (12,3%)** não são números:
`S/N`, `SN`, `S/ NR`, `437A`, `1083-A`, `KM-01`, `KM 1,7`, `1 PLL`.

### A8 — Acentuação inconsistente entre colunas de texto
**Categoria:** formato inconsistente · **Gravidade: média**

| Coluna | Valores distintos | Com acento |
|---|---|---|
| `Municipio` | 385 | **0 (0,0%)** |
| `Bairro` | 2.870 | 87 (3,0%) |
| `Nome da Rua` | 4.012 | 90 (2,2%) |
| `Revenda` | 5.006 | 167 (3,3%) |

`Municipio` é integralmente sem acento (`MACAPA`, `ARAXA`), mas outras colunas do
**mesmo arquivo** preservam acento. Impede junção com bases externas (IBGE) e
cria categorias duplicadas se as colunas forem combinadas.

### A9 — `Municipio` não é chave sozinho
**Categoria:** categorias inconsistentes · **Gravidade: média**

`VALENCA` aparece em **RJ e BA**. Agrupar só por município funde duas cidades
diferentes. A chave geográfica correta é **(`Estado - Sigla`, `Municipio`)**.

### A10 — Nomes de coluna não utilizáveis como identificadores
**Categoria:** formato inconsistente · **Gravidade: baixa**

`Regiao - Sigla`, `Estado - Sigla`, `CNPJ da Revenda`, `Data da Coleta` — têm
espaço, hífen e maiúscula. Impedem notação de atributo e complicam o RA2.

### A11 — `Data da Coleta` em `dd/mm/aaaa`
**Categoria:** tipo de dado incorreto · **Gravidade: média**

Formato brasileiro. Sem `dayfirst=True`, `02/06/2025` é lido como 6 de fevereiro.
Erro silencioso que corrompe todo o eixo temporal — e o eixo temporal é o que
sustenta o RA2.

### A12 — Problemas na camada de ingestão
**Categoria:** outros · **Gravidade: alta (vale 15% da rubrica)**

| Problema | Evidência |
|---|---|
| Três padrões de nome de arquivo | `precos-X-MM.csv` · `MM-dados-abertos-precos-X.csv` · `MM-dados-abertos-precos-AAAA-MM-X.csv` |
| Erro de digitação na origem | `02-cados-abertos-preco-gasolina-etanol.csv` (fev/2026) |
| **Mês ausente** | **Abril/2026 não existe** para nenhum dos 3 grupos |
| Servidor rejeita robôs | HTTP **403** sem `User-Agent` e `Referer` |

---

## B. Verificações que NÃO encontraram problema

> A professora pede explicitamente para documentar a verificação mesmo sem achado.
> Estas linhas valem ponto e **não devem ser omitidas** da apresentação.

| Verificação | Resultado |
|---|---|
| Linhas 100% idênticas | **0** |
| Chave (CNPJ, Produto, Data) repetida | **0** |
| `Valor de Venda` não numérico | **0** |
| `Valor de Venda` ≤ 0 | **0** |
| `Valor de Venda` implausível (> R$ 15/litro) | **0** |
| `Data da Coleta` fora de `dd/mm/aaaa` | **0** |
| `Cep` fora de `00000-000` | **0** de 49.976 |
| `Bandeira` com grafias variantes | **0** — 55 valores distintos, 55 após normalizar |
| Mudança de schema entre anos | **0** — 16 colunas idênticas em 2023, 2025 e 2026 |
| Mudança de schema entre produtos | **0** — gasolina, diesel e GLP têm o mesmo cabeçalho |

> Observação honesta: esta base é **limpa** nos eixos clássicos (duplicatas,
> categorias, outliers). A discussão de qualidade aqui é sobre **tipos, unidades,
> chaves e ingestão** — problemas mais estruturais e menos óbvios. Esse é o
> argumento a sustentar na apresentação.

---

## C. Verificações ainda pendentes

Precisam rodar sobre o período completo antes de fechar `004-decisoes.md`:

- [ ] O mesmo CNPJ muda de `Bandeira` entre meses? (troca real ou erro de cadastro)
- [ ] O mesmo CNPJ aparece com `Municipio` ou `Revenda` diferentes entre meses?
- [ ] Existem duplicatas **entre arquivos** de meses distintos?
- [ ] `Valor de Venda` tem outliers por produto e UF (e não no geral)?
- [ ] Há saltos ou lacunas na cobertura de datas além de abril/2026?
- [ ] O conjunto de UFs cobertas é estável ao longo dos meses?
