# Dicionário do conjunto preparado

Arquivo: `dados/preparados/faixas_preparadas.csv`, CSV UTF-8, separador vírgula, sem índice. Uma linha representa uma ocorrência de faixa em um chart e uma data de coleta. A chave é `data_coleta`, `id_chart` e `id_faixa`.

O retrato de 16/10/2026 produziu 2.800 registros e 16 atributos. Nenhum campo selecionado tem valor nulo nesses 2.800 registros. Evidências: notebook §5, células "Seleção e organização dos atributos", "Verificações finais do conjunto preparado" e "Gravação e releitura do conjunto preparado".

| Atributo | Origem | Significado e uso | Tipo em memória | Domínio ou unidade |
|---|---|---|---|---|
| `id_faixa` | `id` | Identificador da faixa; P1–P4 e RA2 | int64 | Inteiro positivo, repetível em charts distintos |
| `titulo` | `title` | Título para identificação da faixa | string | Texto não vazio, sem espaços nas pontas |
| `id_chart` | `chart_id_genero` | Identificador do grupo das P1–P4 | int64 | Inteiro não negativo; 0 é o chart geral |
| `genero` | `chart_nome_genero` | Nome do chart de origem; grupo das perguntas e evento RA2 | string | Texto não vazio; All identifica o geral na fonte |
| `tipo_chart` | Derivado de `id_chart` | Separação entre geral e charts de gênero | string | geral quando id_chart é 0; genero nos demais |
| `posicao_chart` | `position` | Posição da faixa no chart; P4 | int64 | Inteiro positivo, relativo ao chart de origem |
| `id_artista` | `artist_id` | Identificador do artista; P3 e P4 | int64 | Inteiro positivo |
| `artista` | `artist_name` | Nome para identificação e eventos do RA2 | string | Texto não vazio; agrupar artistas pelo id |
| `id_album` | `album_id` | Identificador do álbum para rastreabilidade | int64 | Inteiro positivo |
| `duracao_s` | `duration` | Duração da faixa; P1 e limite da duração ouvida no RA2 | int64 | Inteiro positivo, segundos |
| `rank` | `rank` | Popularidade da faixa; P3 e peso do sorteio do RA2 | int64 | Inteiro não negativo; não é a posição no chart |
| `numero_fas` | `detalhe_artista_nb_fan` | Quantidade de fãs do artista; P3 | int64 | Inteiro não negativo |
| `data_lancamento` | `detalhe_faixa_release_date` | Data de lançamento da faixa; P2 | datetime64 ns | Data, serializada no CSV como AAAA-MM-DD; valores futuros são preservados e sinalizados |
| `data_coleta` | `metadados.inicio_utc` | Data UTC do retrato e parte da chave | datetime64 ns | Data, serializada como AAAA-MM-DD |
| `lancamento_posterior_coleta` | Comparação entre as duas datas | Sinalização de qualidade temporal | bool | True quando o lançamento é posterior à coleta; False nos demais |
| `duracao_fora_limites_iqr` | Regra IQR da seção 3 sobre `duration` | Sinalização para inspeção de extremos | bool | True fora dos limites; não indica erro confirmado |

As 12 colunas selecionadas do bruto foram acrescidas de quatro campos: tipo_chart, data_coleta e as duas sinalizações. O nome rank foi preservado porque já é esperado pelo gerador de eventos. Evidências: notebook §5, células "Seleção e organização dos atributos" e "Padronização dos nomes e tipos".

## Leitura e limites de uso

O CSV não guarda tipos. Ao reler, informar int64 para as oito colunas inteiras, string para os quatro textos e bool para as duas sinalizações; converter data_lancamento e data_coleta para datas. A célula "Gravação e releitura do conjunto preparado" do notebook §5 contém a leitura validada.

Nos eventos por gênero do RA2, selecionar somente tipo_chart igual a genero. O geral continua no arquivo para a comparação geral da P3. Não excluir participações de uma faixa em outros charts por repetição de id_faixa.

O lançamento posterior à coleta não representa lançamento já ocorrido na janela retrospectiva da P2. A sinalização IQR não autoriza exclusão automática nem substituição pela média. Dados não selecionados continuam no retrato bruto, sem alteração.
