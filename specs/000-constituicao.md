# 000 — Constituição do projeto

> Regras que não mudam. Toda spec e todo código respondem a este documento.

## Entrega

**PjBL – RA1: Ingestão e Preparação dos Dados**
Vencimento: **21/10/2026, 18:00** · 10 pontos · avaliação na **apresentação em sala**.

## Escopo do RA1

```
Fontes → Ingestão → Compreensão → Avaliação da Qualidade → Limpeza → Dados Preparados
```

### Está DENTRO
1. Definição do problema e perguntas analíticas
2. Descrição das fontes de dados
3. Ingestão reprodutível por código
4. Compreensão e avaliação da qualidade
5. Limpeza e preparação com decisões justificadas
6. Conjunto de dados resultante

### Está FORA (é RA2/RA3)
Transformações analíticas, agregações, construção de indicadores, visualizações
e conclusões. **Gráfico só é permitido como evidência de qualidade** (ex.: barra
de percentual de ausentes), nunca como análise do problema.

## Rubrica — todo entregável aponta para um critério

| Critério | Peso | Onde é atendido |
|---|---|---|
| Problema e perguntas analíticas | 20% | `001-problema.md` |
| Compreensão dos dados | 20% | `002-fontes.md` + seção "Conhecer" do notebook |
| **Qualidade e preparação** | **35%** | `003-achados-qualidade.md` + `004-decisoes.md` |
| Ingestão e reprodutibilidade | 15% | `src/ingestao.py` + `002-fontes.md` |
| Organização e documentação | 10% | `README.md` + estrutura do repositório |

Escala por critério: 4 Excelente · 3 Adequado · 2 Parcial · 1 Insuficiente · 0 Não apresentado.

## Princípios (vindos da própria professora)

1. **`INSPECT → DECIDE → TRANSFORM → VALIDATE`** — nenhuma transformação entra no
   código sem uma linha correspondente em `004-decisoes.md`.
2. **Limpeza não é só código.** O que vale ponto é a justificativa, não o comando.
3. **Não inventar defeitos.** Só tratamos o que existe de fato na base.
4. **Verificação sem achado também é entrega.** Se procuramos duplicatas e não
   havia, isso é documentado com o número encontrado (zero).
5. **Quantidade de técnica não é mérito.** Decisão adequada ao dado é.

## Regras técnicas

- O dado bruto é **imutável**: `data/raw/` nunca é editado, e não vai para o git.
- Fluxo em camadas: `raw/` → `interim/` → `processed/`.
- Nomes de coluna em `snake_case`, sem acento e sem espaço.
- Toda decisão de limpeza registra **antes e depois** com número.
- O notebook precisa rodar **do zero**, em máquina limpa e no Google Colab.
- `005-contrato-dados.md` é executável: vira `assert` no fim do notebook.

## Definição de pronto (RA1)

- [ ] As 6 seções do enunciado estão documentadas
- [ ] O notebook roda de ponta a ponta sem erro, partindo de `data/` vazio
- [ ] Toda transformação tem justificativa escrita
- [ ] Todas as regras do contrato de dados passam
- [ ] O `README.md` permite um terceiro reproduzir tudo
- [ ] Cada integrante consegue defender qualquer decisão na apresentação
