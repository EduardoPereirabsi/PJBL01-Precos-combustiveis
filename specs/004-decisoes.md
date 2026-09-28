# 004 — Decisões de limpeza e preparação (DECIDE)

> **SPECIFY'** · Atende ao critério **"Qualidade e preparação" (35%)** — é a peça
> que mais vale ponto. Corresponde ao item 5 do enunciado.
>
> ⚠️ **Status: proposta.** Cada linha precisa ser discutida e aceita pela equipe.
> Na apresentação, quem defender a decisão precisa concordar com ela.
>
> **Regra do projeto:** nenhuma transformação entra no código sem uma linha aqui.

## Tabela de decisões

| # | Achado | Decisão | Justificativa |
|---|---|---|---|
| A1 | `Valor de Venda` mistura 3 unidades | **Não converter.** Manter `unidade_medida` no dataset e impor no contrato que cada produto tenha exatamente uma unidade | Converter R$/13 kg e R$/m³ para "equivalente litro" exigiria premissas de densidade e poder calorífico que a base não tem. Inventar essa conversão seria criar dado. Tornar a unidade explícita e validá-la impede a agregação errada sem falsear nada |
| A2 | UTF-8 com BOM | Ler com `encoding="utf-8-sig"` | Resolve na leitura, sem alterar o arquivo bruto. `data/raw/` permanece intocado |
| A3 | Decimal com vírgula | `decimal=","` no `read_csv` | A variável de interesse do projeto precisa ser numérica. Converter depois com `str.replace` seria mais frágil |
| A4 | CNPJ com espaço e pontuação | Normalizar para **14 dígitos**, sem pontuação | O CNPJ é a chave do posto. Forma canônica única evita que o mesmo posto vire dois no agrupamento |
| A5 | `Valor de Compra` 100% vazia | **Remover a coluna** | Vazia em todos os anos e produtos verificados. Não é ausência parcial a tratar — é coluna sem dado. Mantê-la carrega peso morto para o RA2 e induz o leitor a achar que existe informação de margem |
| A6 | Colunas de endereço detalhado | **Remover** `nome_rua`, `numero_rua`, `complemento`, `bairro`. **Manter** `cep` | Nenhuma das 5 perguntas de `001` opera abaixo do nível de município. O `cep` fica porque é barato e permite georreferenciar no RA3 sem refazer a ingestão |
| A7 | `Numero Rua` com texto | Resolvido por A6 | Se a equipe decidir manter o endereço, a decisão passa a ser "manter como texto", pois `S/N` e `KM 1,7` são valores legítimos |
| A8 | Acentuação inconsistente | Normalizar `municipio`, `revenda` e `bandeira` para **maiúscula, sem acento, sem espaço duplicado** | A base já é majoritariamente sem acento; padronizar elimina o risco de a mesma entidade virar duas categorias. Custo: perde-se a grafia correta — aceitável, pois nenhuma pergunta depende da ortografia |
| A9 | `Municipio` não é chave | Documentar que a chave geográfica é (`uf`, `municipio`) e validar no contrato | `VALENCA` existe em RJ e BA. Agrupar só por município fundiria duas cidades. Não exige transformação, exige disciplina — e o contrato garante |
| A10 | Nomes de coluna com espaço/hífen | Renomear para `snake_case` sem acento | Permite notação de atributo, evita aspas em SQL e simplifica a migração para Spark no RA2 |
| A11 | Data em `dd/mm/aaaa` | `format="%d/%m/%Y"` explícito, **não** `dayfirst=True` | `dayfirst` é uma dica que o parser pode ignorar; `format` falha alto se o padrão mudar. Erro de data aqui corromperia silenciosamente o eixo temporal, que é a base do RA2 |
| A12 | Ingestão instável | Catálogo explícito de URLs + cabeçalhos HTTP + registro de ausências | Ver `plan/arquitetura.md`. Abril/2026 vira uma **lacuna declarada**, não um silêncio |

## Decisões de escopo

| Decisão | Justificativa |
|---|---|
| Período: **jan/2023 a ago/2026** | É o intervalo com granularidade mensal publicada. Antes disso os arquivos são semestrais e com outro padrão, o que dobraria o trabalho de ingestão sem beneficiar as perguntas |
| Manter os **7 produtos** | Excluir GLP e GNV simplificaria, mas jogaria fora o achado A1, que é o mais forte da base |
| **Não** imputar valores ausentes | Não há ausência nas colunas que importam. Imputar aqui seria inventar defeito e depois inventar correção |
| **Não** remover outliers de preço | Nenhum valor implausível foi encontrado (mín 3,19 / máx 9,29 por litro). Preço alto em região remota é fato de mercado, não erro. Remover distorceria justamente a dispersão que o projeto quer medir |

> As duas últimas linhas são decisões de **não fazer**. Elas valem tanto quanto as
> outras: mostram que a equipe verificou antes de agir. É exatamente o que a
> professora pede em *"não é necessário inventar defeitos"*.

## Pendências antes de fechar

Dependem das verificações abertas em `003-achados-qualidade.md`, seção C:

- [ ] **CNPJ que muda de bandeira entre meses** — decidir se é troca real (manter o histórico) ou erro de cadastro (padronizar pela mais frequente)
- [ ] **Duplicatas entre arquivos de meses diferentes** — decidir a regra, se existirem
- [ ] **Outliers por produto e UF** — revisitar a decisão de não remover, agora segmentada
