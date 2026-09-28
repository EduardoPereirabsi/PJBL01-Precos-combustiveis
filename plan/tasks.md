# Backlog do RA1 (TASKS)

> Prazo: **21/10/2026, 18:00** · Hoje: 2026-09-23 · **4 semanas**
> Cada tarefa aponta para o critério da rubrica que ela alimenta.

## Legenda

`[ ]` aberta · `[~]` em andamento · `[x]` concluída · **(E)** decisão da equipe, não minha

---

## Bloco 0 — Fundação

- [x] Transcrever o enunciado e a rubrica — `docs/enunciado-RA1.md`
- [x] Ler e sintetizar o material da professora — `docs/entendimento-disciplina.md`
- [x] Avaliar bases candidatas com medição real — `docs/opcoes-de-tema.md`
- [x] Escolher o tema **(E)** — ANP, preços de combustíveis
- [x] Criar estrutura do repositório e `git init`
- [x] Escrever specs 000 a 005
- [ ] **(E)** Criar o repositório no GitHub e adicionar os integrantes
- [ ] **(E)** Registrar os nomes da equipe no `README.md`
- [ ] Primeiro commit

## Bloco 1 — Problema e perguntas · *20%*

- [ ] **(E)** Revisar `001-problema.md` em equipe — as perguntas precisam ser de vocês
- [ ] **(E)** Escolher **uma** pergunta central e defendê-la (hoje há 5 candidatas)
- [ ] **(E)** Confirmar o tipo de analytics: descritivo + diagnóstico
- [ ] **(E)** Definir quem apresenta cada parte

## Bloco 2 — Fontes e ingestão · *20% + 15%*

- [ ] Montar o catálogo completo de URLs (2023-01 a 2026-08, 3 grupos, 129 arquivos)
- [ ] Implementar `src/ingestao.py` com cabeçalhos HTTP e cache local
- [ ] Gerar `data/raw/_manifesto.csv` com hash e contagem de linhas
- [ ] Confirmar que abril/2026 é a **única** lacuna do período
- [ ] Medir o volume real total (estimativa atual: ~3,6 milhões de linhas)
- [ ] Atualizar `002-fontes.md` com os números medidos, não estimados

## Bloco 3 — Qualidade · *35% (maior peso)*

- [ ] Reproduzir todos os achados de `003` sobre o **período completo**, não só a amostra
- [ ] Verificação pendente: o mesmo CNPJ muda de `Bandeira` entre meses?
- [ ] Verificação pendente: o mesmo CNPJ muda de `Municipio` ou `Revenda`?
- [ ] Verificação pendente: existem duplicatas **entre** arquivos de meses diferentes?
- [ ] Verificação pendente: outliers de preço **por produto e UF** (não no geral)
- [ ] Verificação pendente: a cobertura de UFs é estável mês a mês?
- [ ] Atualizar `003-achados-qualidade.md` com os números definitivos
- [ ] **(E)** Revisar e aceitar cada linha de `004-decisoes.md`
- [ ] **(E)** Fechar as 3 pendências de `004`

## Bloco 4 — Implementação

- [ ] Criar o ambiente virtual e instalar as dependências
- [ ] Implementar `src/preparacao.py` — uma função por decisão
- [ ] Implementar `src/validacao.py` — uma função por regra R1–R12
- [ ] Montar `notebooks/ra1_preparacao.ipynb` nas 8 seções de `plan/arquitetura.md`
- [ ] Cada transformação imprime antes → depois → impacto
- [ ] Gravar `data/processed/precos_combustiveis.parquet`
- [ ] Fazer o notebook rodar também no Google Colab

## Bloco 5 — Validação e entrega · *10%*

- [ ] Executar as 12 regras do contrato; todas precisam passar
- [ ] **Teste de mesa limpa:** apagar `data/`, rodar do zero, tudo funciona
- [ ] Escrever o `README.md` de reprodução
- [ ] Revisar a cobertura: os 6 itens do enunciado estão documentados?
- [ ] Revisar a cobertura: os 5 critérios da rubrica têm entregável apontando?

## Bloco 6 — Apresentação

- [ ] **(E)** Montar o roteiro a partir das specs
- [ ] **(E)** Ensaiar a defesa das decisões de **não fazer** (não imputar, não remover outliers) — é onde a professora costuma pressionar
- [ ] **(E)** Preparar a resposta para *"por que vocês não converteram as unidades?"* — é a pergunta mais provável
- [ ] **(E)** Cada integrante precisa saber explicar qualquer decisão, não só a sua parte

---

## Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| A ANP muda a URL dos arquivos durante o projeto | Alto | O manifesto com hash detecta; `data/raw/` fica em backup local |
| A base é limpa demais nos eixos clássicos | Médio | Já mapeado: o argumento se sustenta em tipos, unidades e ingestão. A seção B de `003` vira ponto forte, não fraqueza |
| A equipe não revisar `004` a tempo | **Alto** | É o item de 35%. Precisa ser a primeira reunião, não a última |
| Download de ~300 MB na hora da apresentação | Médio | Cache em `data/raw/` + rodar antes; o notebook não deve baixar ao vivo |

## Caminho crítico

```
Revisar 001 (E)  →  ingestão  →  achados completos  →  aceitar 004 (E)  →  implementar  →  validar
```

O gargalo não é técnico, é **decisão de equipe**. As tarefas marcadas **(E)**
bloqueiam tudo o que vem depois — e são as que mais valem ponto.
