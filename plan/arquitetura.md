# Arquitetura do pipeline (PLAN)

> Atende ao critério **"Ingestão e reprodutibilidade" (15%)** e ao item 3 do enunciado.

## Camadas

```
ANP (web)  →  data/raw/  →  data/interim/  →  data/processed/
              intocado      consolidado       validado e entregue
              (fora do git) (fora do git)     (no git)
```

| Camada | Conteúdo | Regra |
|---|---|---|
| `raw/` | CSV exatamente como baixado, um arquivo por mês × grupo | **Nunca é editado.** Se for preciso rebaixar, apaga-se e roda de novo |
| `interim/` | Os arquivos concatenados, já com nomes de coluna normalizados e tipos corrigidos — mas **sem** decisões de limpeza aplicadas | Separar as duas etapas permite mostrar o "antes" e o "depois" na apresentação |
| `processed/` | O dataset final, validado contra `specs/005-contrato-dados.md` | Único que vai para o git, em Parquet |

O dado bruto fica fora do git de propósito: **a reprodutibilidade é garantida
pelo código de ingestão, não por commitar 300 MB de CSV.** Isso é exatamente o
que o enunciado pede em *"o processo de ingestão deverá ser reprodutível por
meio de código"*.

## Módulo de ingestão (`src/ingestao.py`)

### Problema a resolver

A ANP não publica com padrão único. A ingestão precisa lidar com:

1. **Três padrões de nome de arquivo** (ver `specs/002-fontes.md`)
2. **Um arquivo com erro de digitação na origem** (`02-cados-abertos-preco-...`)
3. **Um mês inexistente** (abril/2026)
4. **HTTP 403** para requisições sem cabeçalho de navegador

### Desenho

**Catálogo explícito, não adivinhação.** Um dicionário mapeia
`(ano, mês, grupo) → URL`, construído a partir dos links reais da página da ANP.
Tentar deduzir o nome do arquivo por regra falharia em fevereiro e em junho de 2026.

```
CATALOGO = {
    (2025, 6, "gasolina-etanol"): ".../2025/precos-gasolina-etanol-06.csv",
    (2026, 2, "gasolina-etanol"): ".../2026/02-cados-abertos-preco-gasolina-etanol.csv",
    ...
}
```

**Responsabilidades:**

| Função | O que faz |
|---|---|
| `baixar(ano, mes, grupo)` | Baixa com `User-Agent` e `Referer`; pula se o arquivo já existe em `raw/` (cache) |
| `baixar_periodo(inicio, fim)` | Percorre o catálogo e registra o que **não** existe |
| `manifesto()` | Gera `data/raw/_manifesto.csv` com arquivo, URL, bytes, linhas, data do download e hash |

> O **manifesto** é o que torna a ingestão auditável: quem clonar o repositório
> consegue provar que baixou exatamente os mesmos arquivos. Atende diretamente ao
> "permitindo reproduzir o fluxo do RA1" da rubrica.

**Ausências são declaradas, não silenciosas.** Abril/2026 aparece no manifesto
com status `AUSENTE_NA_ORIGEM`. Um mês que some sem aviso é um erro de dados;
um mês que some com registro é uma característica documentada da base.

## Módulo de preparação (`src/preparacao.py`)

Uma função por decisão de `specs/004-decisoes.md`, cada uma recebendo e
devolvendo um DataFrame, e cada uma retornando também o **impacto** (linhas ou
células afetadas) para o relatório antes/depois.

| Função | Decisão |
|---|---|
| `normalizar_colunas(df)` | A10 |
| `normalizar_cnpj(df)` | A4 |
| `normalizar_texto(df)` | A8 |
| `converter_tipos(df)` | A3, A11 |
| `remover_colunas_vazias(df)` | A5, A6 |

## Módulo de validação (`src/validacao.py`)

Uma função por regra `R1`–`R12` de `specs/005-contrato-dados.md`. Devolve uma
tabela `regra | esperado | obtido | status` e levanta `AssertionError` se
qualquer regra falhar.

> É aqui que a spec deixa de ser documento e vira teste. O `VALIDATE` do SDD e o
> `VALIDATE` da professora passam a ser a mesma linha de código.

## Notebook (`notebooks/ra1_preparacao.ipynb`)

A estrutura espelha a do notebook da professora, porque é o formato que ela
avalia. O notebook **chama** os módulos de `src/` em vez de repetir a lógica —
assim a mesma função serve ao notebook e ao RA2.

| Seção | Conteúdo | Enunciado |
|---|---|---|
| 1. Carregar | Ingestão + manifesto | item 3 |
| 2. Conhecer | Dicionário de dados, o que cada coluna significa | item 2 |
| 3. Inspecionar | `shape`, tipos, ausentes, duplicatas, distribuições | item 4 |
| 4. Identificar problemas | Os achados **com números medidos** | item 4 |
| 5. Decidir | A tabela de `004-decisoes.md` renderizada | item 5 |
| 6. Transformar | Cada tratamento com antes → depois → impacto | item 5 |
| 7. Validar | As 12 regras, com ✓/✗ | item 6 |
| 8. Dataset resultante | `shape` final, amostra, gravação em Parquet | item 6 |

> **Não existe seção 9 de análise.** O notebook da professora tem uma, mas o
> enunciado do RA1 exclui explicitamente análise, agregação e visualização
> analítica. Terminar no dataset validado é o comportamento correto.

## Compatibilidade com Google Colab

A professora usa Colab. O notebook precisa rodar nos dois ambientes: a primeira
célula detecta o Colab, clona o repositório se necessário e ajusta os caminhos.
Sem isso, a apresentação vira uma sessão de depuração de caminho de arquivo.

## Ponte para o RA2

Decisões tomadas agora pensando na próxima entrega:

- **Parquet em vez de CSV** — o Spark lê nativamente e com tipos preservados
- **`data_coleta` como timestamp real** — é o eixo do stream processing
- **Uma função por transformação** — migram para PySpark sem reescrever a lógica
- **Particionamento por `ano/mes`** ao gravar — aproveitável direto no Spark
