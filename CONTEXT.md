# Contexto do projeto

> Leia este arquivo **antes de qualquer tarefa**. Ele resume o que está decidido e aponta onde está o detalhe. É curto de propósito: abra só o que a sua tarefa pede (tabela da seção 9).

## 1. Disciplina e entregas

- **Disciplina:** Data Analytics, Projeto Baseado em Problemas (PjBL), BSI PUCPR, 2026. **Professora:** Marina de Lara.
- **Equipe** (nomes exatamente assim na capa do documento e na identificação do notebook; a turma não entra): Eduardo Fabri, João Correa, Rodrigo Alves e João Pedro Cardoso. Áreas e logins: [plano de tarefas](entregas/plano-de-tarefas.md), seção 1.
- **Entregas cobertas por este repositório:** RA1 (ingestão e preparação dos dados) e RA2 (processamento em streaming com janelas temporais). O RA3 (análise e visualização) virá depois e usará o mesmo notebook e os mesmos dados.
- **Prazo do RA1 e do RA2:** **21/10/2026** (data que aparece nas duas tarefas do Canvas). O RA1 é avaliado na apresentação da equipe em sala; o RA2 tem demonstração oral de 5 minutos.
- **Enunciados transcritos:** [enunciados/ra1.md](enunciados/ra1.md) e [enunciados/ra2.md](enunciados/ra2.md). Os PDFs originais ficam fora do repositório (ver [enunciados/README.md](enunciados/README.md)).

**O RA1 e o RA2 vão no mesmo arquivo:** um PDF único, `output/entrega-ra1-ra2.pdf`, enviado nas duas tarefas do Canvas ([ADR-0009](docs/adr/0009-entrega-unica-em-pdf.md)). A tarefa do RA2 recebe o arquivo; a do RA1 só aceita URL, então nela vai o link do mesmo PDF no repositório, na tag `entrega` (`https://github.com/Jcliz/data-analytics/blob/entrega/output/entrega-ra1-ra2.pdf`).

| Parte do PDF único | O que é | Onde está a fonte | Decisão |
|---|---|---|---|
| 1. Documento do RA1 | Template `.docx` preenchido e exportado em PDF, com a URL do repositório na capa | `documentacao/ra1/` | [ADR-0003](docs/adr/0003-documento-ra1-gerado-do-template.md) |
| 2. Notebook | Notebook inteiro executado (identificação, RA1 nas seções 1 a 7, RA2 nas seções 8 a 12), exportado em PDF; o `.ipynb` executado fica no repositório | `notebook/secoes/` | [ADR-0002](docs/adr/0002-notebook-montado-por-secoes.md), [ADR-0005](docs/adr/0005-ra2-cenario-a-simulacao.md) |

Como o PDF deve ficar: [entregas/orientacoes-documento-final.md](entregas/orientacoes-documento-final.md).

## 2. Fonte de dados

- **API pública da Deezer** (`https://api.deezer.com`), sem autenticação e sem chave. É a mesma API usada pelo app [rdsalvesPUC/dataaudio](https://github.com/rdsalvesPUC/dataaudio), que serve de referência ("muleta") para os endpoints e o formato dos objetos. Decisão: [ADR-0001](docs/adr/0001-fonte-de-dados-api-deezer.md).
- **Endpoints já usados pelo dataaudio** [Fato, `lib/services/deezer_service.dart`]: `/chart/0/tracks` (paginado por `index` e `limit`), `/search?q=` e `/track/{id}`.
- **Reprodutibilidade:** a resposta da API muda todo dia (o chart é ao vivo), então a ingestão grava um **retrato bruto** em `dados/brutos/` e o notebook lê dele por padrão. Decisão: [ADR-0004](docs/adr/0004-retrato-bruto-versionado.md).
- Detalhe da API, limites e problemas de qualidade esperados: [base/api-deezer.md](base/api-deezer.md).

## 3. Problema e perguntas analíticas

**Problema:** o que caracteriza as faixas que estão no topo dos charts da Deezer e como esse perfil muda entre gêneros. Unidade de observação: a faixa; período: a data da coleta do retrato bruto (chart geral e charts por gênero). Decisão: [ADR-0010](docs/adr/0010-problema-e-perguntas.md); texto completo em [documentacao/ra1/01-problema-e-perguntas.md](documentacao/ra1/01-problema-e-perguntas.md).

| ID | Pergunta | Dados |
|---|---|---|
| P1 | Como a duração das faixas varia entre os gêneros? | `duration`, gênero do chart |
| P2 | Que parte das faixas de cada gênero foi lançada nos 12 meses anteriores à coleta? | `release_date`, gênero do chart |
| P3 | Faixas de artistas com mais fãs têm `rank` mais alto, no geral e em cada gênero? | `rank`, `nb_fan`, gênero do chart |
| P4 | Quantos artistas distintos ocupam cada chart de gênero e que fatia das posições fica com os 5 com mais faixas? | `artist.id`, `position`, gênero do chart |

O gênero do chart de origem é o grupo de todas as perguntas: a deduplicação de faixas precisa preservá-lo.

## 4. Glossário mínimo

| Termo | Significado no projeto |
|---|---|
| **Faixa** | Uma música da Deezer (objeto `track`). É a unidade de observação ([ADR-0010](docs/adr/0010-problema-e-perguntas.md)). Como a mesma faixa pode estar em mais de um chart, uma linha do conjunto preparado é uma **ocorrência** da faixa num chart, e não a faixa sozinha ([dicionário](dados/preparados/dicionario-faixas.md)). |
| **Chart** | Ranking de faixas mais populares devolvido por `/chart/{id_genero}/tracks`. |
| **Retrato bruto** | Cópia fiel das respostas da API em JSON, gravada com a data da coleta em `dados/brutos/`. Nunca é editada à mão. |
| **Conjunto preparado** | Saída do RA1: `dados/preparados/faixas_preparadas.csv`, já limpo e documentado, com uma linha por ocorrência faixa-chart (chave `data_coleta`, `id_chart`, `id_faixa`). Entrada do RA2. |
| **Evento de reprodução** | Registro artificial do RA2: "a faixa X foi tocada no instante T". Gerado a partir do conjunto preparado (cenário A). |
| **Janela tumbling / sliding** | Janela temporal sem sobreposição / com sobreposição, usada na agregação do RA2. |
| **Q1…Qn** | IDs das verificações de qualidade (documento do RA1, seção 03). |
| **D1…Dn** | IDs das decisões de limpeza (documento do RA1, seção 04). Não confundir com as decisões do projeto, que são ADRs. |
| **P1…Pn** | IDs das perguntas analíticas (documento do RA1, seção 01). |

## 5. Decisões vigentes

| | Decisão | Status | ADR |
|---|---|---|---|
| 0001 | Fonte de dados: API pública da Deezer | aceita | [0001](docs/adr/0001-fonte-de-dados-api-deezer.md) |
| 0002 | Um notebook único, montado por script a partir de um arquivo por seção | proposta | [0002](docs/adr/0002-notebook-montado-por-secoes.md) |
| 0003 | Documento do RA1 escrito em Markdown e gerado no template `.docx` por script; PDF exportado do `.docx` | proposta | [0003](docs/adr/0003-documento-ra1-gerado-do-template.md) |
| 0004 | Retrato bruto da API versionado; notebook lê do retrato por padrão | proposta | [0004](docs/adr/0004-retrato-bruto-versionado.md) |
| 0005 | RA2 no cenário A: eventos de reprodução simulados a partir do conjunto preparado | proposta | [0005](docs/adr/0005-ra2-cenario-a-simulacao.md) |
| 0006 | Ferramentas: pandas no RA1, PySpark 4.0.3 no RA2, Google Colab como ambiente de referência | proposta | [0006](docs/adr/0006-ferramentas-e-ambiente.md) |
| 0007 | Nenhum commit com Claude (ou outra IA) como coautor | aceita | [0007](docs/adr/0007-commits-sem-coautoria-de-ia.md) |
| 0008 | Divisão do trabalho em quatro áreas, uma por integrante | proposta | [0008](docs/adr/0008-divisao-em-quatro-areas.md) |
| 0009 | Entrega única: um PDF com o documento do RA1 e o notebook, enviado no RA1 e no RA2 | aceita | [0009](docs/adr/0009-entrega-unica-em-pdf.md) |
| 0010 | Problema e perguntas: perfil das faixas populares nos charts da Deezer por gênero (P1 a P4) | proposta | [0010](docs/adr/0010-problema-e-perguntas.md) |

"Proposta" quer dizer que o texto já orienta o trabalho, mas o grupo ainda pode mudar. Mudou? Atualize o ADR para "aceita" ou "substituída".

## 6. Contrato entre as seções do notebook

O notebook é executado de cima para baixo, e cada seção é de uma área. Para uma seção não quebrar a outra, os nomes abaixo são fixos. Quem precisar de outro nome combina na issue `pendencia-cruzada` da área dona.

| Nome | Tipo | Criado em | Usado em | Conteúdo |
|---|---|---|---|---|
| `RAIZ` | `pathlib.Path` | 00b (ambiente) | todas | Raiz do repositório. |
| `PASTA_BRUTOS`, `PASTA_PREPARADOS`, `PASTA_STREAMING` | `Path` | 00b | 02, 04, 05, 09, 10, 11 | `dados/brutos`, `dados/preparados`, `dados/streaming`. |
| `BAIXAR_DA_API` | `bool` | 00b | 02 | `False` por padrão: lê o retrato bruto já versionado. `True` coleta de novo na API. |
| `df_bruto` | `pandas.DataFrame` | 02 | 03, 04 | Faixas como vieram da API, sem limpeza: uma linha por item dos charts (com o chart de origem), já com os campos de `/track/{id}`, `/album/{id}` e `/artist/{id}` juntados pelo id (prefixos `detalhe_faixa_`, `detalhe_album_` e `detalhe_artista_`). |
| `df_preparado` | `pandas.DataFrame` | 04 e 05 | 05, 07, 09 | Conjunto preparado. Também gravado em `dados/preparados/faixas_preparadas.csv`. |
| `spark` | `SparkSession` | 09 | 09, 10, 11 | Sessão do Spark do RA2. |
| `PASTA_ENTRADA`, `PASTA_PUBLICACAO`, `PASTA_CHECKPOINT` | `Path` | 09 | 09, 10, 11 | Subpastas `entrada/`, `publicacao/` e `checkpoint/` de `PASTA_STREAMING`. |
| `SEMENTE`, `INICIO_SIMULACAO` | `int`, `datetime` | 09 | 09, 11 | Semente do sorteio e instante do primeiro evento, fixos para a execução final repetir os números. |
| `esquema_eventos` | `StructType` | 09 | 09 | Esquema do evento de reprodução `{id_evento, id_faixa, genero, artista, duracao_ouvida_s, instante}`. |
| `gerar_eventos(n, inicio)` | função | 09 | 09, 11 | Sorteia `n` eventos de `df_preparado`, com instantes crescentes a partir de `inicio`. |
| `publicar_lote(eventos)` | função | 09 | 09, 11 | Grava um lote em JSON em `PASTA_PUBLICACAO` e move para `PASTA_ENTRADA`. |
| `eventos` | `DataFrame` em streaming | 09 | 09, 10 | `spark.readStream` sobre `PASTA_ENTRADA`. |
| `TABELA_JANELAS` | `str` | 10 | 11 | Nome da tabela em memória da consulta com a janela (`queryName`). |
| `consulta` | `StreamingQuery` | 10 | 11 | Consulta em streaming com a janela. Encerrada na última célula da seção 11. |

## 7. Regras para quem escreve (pessoas e IAs)

1. **Precedência:** enunciados ([enunciados/](enunciados/)) > decisões ([docs/adr/](docs/adr/README.md)) > base de conhecimento ([base/](base/README.md)) > rascunhos.
2. **Não contradiga uma decisão.** Se algo pedir mudança, registre a dúvida na issue ou com o grupo em vez de mudar o texto por conta própria.
3. **O RA1 não tem análise.** Agregações analíticas, indicadores, visualizações e conclusões ficam para os próximos RAs (enunciado do RA1, seção 6). No RA1 entram só exploração, qualidade, limpeza e preparação.
4. **Toda afirmação sobre os dados tem evidência:** a célula do notebook ou o arquivo que a mostra, com a contagem. "Muitos valores ausentes" não vale; "312 de 1.950 faixas (16,0%) com `bpm` = 0, notebook §3, célula "Bpm igual a zero"" vale. A célula é citada pelo comentário-título que abre o código dela.
5. **Separe [Fato] de [Inferência]** quando o texto não for óbvio, e cite a fonte pelo número de [base/fontes.md](base/fontes.md). No documento do RA1 e nas respostas do notebook, não use as marcas entre colchetes (no documento, colchete é campo não preenchido): diga no texto quando algo é estimativa ou suposição.
6. **Tudo em português do Brasil:** textos, nomes de variáveis novas, colunas do conjunto preparado, commits, issues e PRs. Nomes que vêm da API (`title`, `duration`, `rank`…) mantêm o original no retrato bruto e são renomeados para português no conjunto preparado.
7. **Commits sem coautoria de IA** ([ADR-0007](docs/adr/0007-commits-sem-coautoria-de-ia.md)). O autor é sempre a pessoa do grupo.
8. **Nenhuma declaração de uso de IA** em nenhuma entrega: documento do RA1, notebook e PDFs não mencionam assistentes de IA. Decisão do grupo (João, 15/10/2026); não acrescente essa seção nem quando um template, um modelo de outro projeto ou o seu padrão sugerir.
9. **Nada da organização interna no texto entregue:** o texto que vai para a entrega (documento do RA1, células e saídas do notebook, PDFs) não cita área, tarefa (T01…T14), issue, PR, branch nem kanban; a evidência cita a seção, a célula ou o arquivo. Essa organização é do repositório, não do trabalho entregue (João, 18/10/2026; critério X.9). As linhas `#>` e os comentários `<!-- -->` podem citar, porque os scripts os descartam. `python scripts/verificar_entrega.py` acusa o que sobrar.

## 8. Onde fica cada coisa

- **Mapa do repositório e fluxo de trabalho:** [README.md](README.md)
- **Plano de tarefas, áreas e prazos:** [entregas/plano-de-tarefas.md](entregas/plano-de-tarefas.md)
- **Critérios de aceite (definição de pronto):** [entregas/criterios-de-aceite.md](entregas/criterios-de-aceite.md)
- **Como o PDF final deve ficar:** [entregas/orientacoes-documento-final.md](entregas/orientacoes-documento-final.md)
- **Base de conhecimento:** [base/README.md](base/README.md)
- **Decisões:** [docs/adr/README.md](docs/adr/README.md)

## 9. O que ler para cada tarefa

| Tarefa | Leia | Decisões |
|---|---|---|
| Problema e perguntas (A) | [enunciados/ra1.md](enunciados/ra1.md) §1, [sugestoes-de-perguntas](base/sugestoes-de-perguntas.md), [api-deezer](base/api-deezer.md) §1–2 | 0001 |
| Ingestão e reprodutibilidade (B) | [api-deezer](base/api-deezer.md), [material de aula](base/material-de-aula.md) (ingestão) | 0001, 0004, 0006 |
| Compreensão e qualidade (C) | [qualidade-de-dados](base/qualidade-de-dados.md), [api-deezer](base/api-deezer.md) §4 | 0004 |
| Limpeza, preparação e conjunto resultante (D) | [qualidade-de-dados](base/qualidade-de-dados.md), [material de aula](base/material-de-aula.md) (preparação × transformação) | 0004 |
| RA2, qualquer seção | [enunciados/ra2.md](enunciados/ra2.md), [streaming-e-janelas](base/streaming-e-janelas.md) | 0005, 0006 |
| Consolidação e PDF | [orientacoes-documento-final](entregas/orientacoes-documento-final.md), [criterios-de-aceite](entregas/criterios-de-aceite.md) | 0002, 0003, 0009 |
