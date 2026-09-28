# 005 — Contrato do dataset preparado (VALIDATE)

> Atende ao item 6 do enunciado e fecha o ciclo **VALIDATE**.
> Este documento é **executável**: cada regra vira um `assert` no fim do notebook.
> Se uma regra falhar, o dataset não está pronto para entrega.

## Identificação

| Item | Valor |
|---|---|
| Arquivo | `data/processed/precos_combustiveis.parquet` |
| Formato | Parquet (preserva tipos; CSV perderia a tipagem e o RA2 teria de reinferir) |
| Granularidade | 1 linha = 1 coleta de preço, em 1 posto, 1 data, 1 produto |
| Chave natural | (`cnpj_revenda`, `produto`, `data_coleta`) |
| Chave geográfica | (`uf`, `municipio`) |

## Schema esperado

| Coluna | Tipo | Aceita nulo? | Origem |
|---|---|---|---|
| `regiao` | `category` | não | `Regiao - Sigla` |
| `uf` | `category` | não | `Estado - Sigla` |
| `municipio` | `string` | não | `Municipio` (normalizado) |
| `cnpj_revenda` | `string` | não | `CNPJ da Revenda` (14 dígitos) |
| `revenda` | `string` | não | `Revenda` (normalizado) |
| `cep` | `string` | sim | `Cep` |
| `produto` | `category` | não | `Produto` |
| `data_coleta` | `datetime64[ns]` | não | `Data da Coleta` |
| `valor_venda` | `float64` | não | `Valor de Venda` |
| `unidade_medida` | `category` | não | `Unidade de Medida` |
| `bandeira` | `category` | não | `Bandeira` |

**11 colunas.** Removidas: `Valor de Compra`, `Nome da Rua`, `Numero Rua`,
`Complemento`, `Bairro` — ver decisões A5 e A6 em `004-decisoes.md`.

## Regras de validação

| # | Regra | Rastreia |
|---|---|---|
| R1 | O schema tem exatamente as 11 colunas acima, nos tipos declarados | A10 |
| R2 | Nenhum nulo em `regiao`, `uf`, `municipio`, `cnpj_revenda`, `produto`, `data_coleta`, `valor_venda`, `unidade_medida`, `bandeira` | A5 |
| R3 | `cnpj_revenda` tem exatamente 14 dígitos em 100% das linhas | A4 |
| R4 | `uf` pertence ao conjunto das 27 unidades federativas | A9 |
| R5 | **Cada `produto` tem exatamente uma `unidade_medida`** | **A1** |
| R6 | `valor_venda > 0` em 100% das linhas | — |
| R7 | `data_coleta` está entre 2023-01-01 e 2026-08-31 | A11 |
| R8 | Nenhuma duplicata na chave (`cnpj_revenda`, `produto`, `data_coleta`) | — |
| R9 | `municipio` não contém caractere acentuado | A8 |
| R10 | `municipio` não tem espaço no início/fim nem espaço duplo | A8 |
| R11 | Os meses ausentes são **apenas** os declarados em `002-fontes.md` (abril/2026) | A12 |
| R12 | A contagem final bate com as linhas lidas de `data/raw/` menos as remoções registradas | — |

> **R5 é a regra mais importante.** É ela que impede automaticamente o erro que o
> achado A1 descreve: misturar R$/litro com R$/13 kg numa mesma média.

## Relatório de entrega

O notebook precisa imprimir, ao final:

- linhas antes × depois, com total removido e o motivo de cada remoção
- contagem de nulos por coluna (esperado: zero nas colunas obrigatórias)
- período coberto e lista dos meses ausentes
- número de postos distintos, municípios distintos e UFs cobertas
- tabela `produto × unidade_medida × contagem` (evidência visual de R5)
- resultado de cada regra R1–R12, com ✓ ou ✗

## Critério de aceite

> O RA1 está pronto quando as 12 regras passam, partindo de `data/` vazio, em uma
> execução única do notebook, do começo ao fim.
