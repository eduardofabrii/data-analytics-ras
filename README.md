# Perfil das faixas populares nos charts da Deezer por gênero

Projeto de Data Analytics da PUCPR, desenvolvido por Eduardo Fabri, João Correa, Rodrigo Alves e João Pedro Cardoso, sob orientação da professora Marina de Lara.

O notebook reúne a ingestão, avaliação da qualidade e preparação dos dados do RA1 e a simulação de reproduções com Spark Structured Streaming do RA2. A fonte é a [API pública da Deezer](https://api.deezer.com), sem chave ou autenticação.

## Arquivos do projeto

| Caminho | Conteúdo |
|---|---|
| [notebook/ra1-ra2-deezer.ipynb](notebook/ra1-ra2-deezer.ipynb) | Código, explicações e resultados salvos do RA1 e do RA2 |
| [dados/brutos/2026-10-16/](dados/brutos/2026-10-16/) | Respostas originais da API e metadados da coleta |
| [dados/preparados/faixas_preparadas.csv](dados/preparados/faixas_preparadas.csv) | Conjunto preparado pelo RA1 e utilizado na simulação do RA2 |
| [dados/preparados/dicionario-faixas.md](dados/preparados/dicionario-faixas.md) | Significado, origem, tipos e limites de uso das colunas do CSV |
| [requirements.txt](requirements.txt) | Dependências para execução local |

## Execução no Google Colab

1. [Abra o notebook no Colab](https://colab.research.google.com/github/eduardofabrii/data-analytics-ras/blob/main/notebook/ra1-ra2-deezer.ipynb).
2. Use um ambiente Python com CPU. Em uma célula antes do início do notebook, prepare as bibliotecas do RA1:

   ```python
   %pip install -q "pandas>=2.2,<3" "requests>=2.31"
   ```

3. Reinicie a sessão após a instalação, para carregar as versões instaladas, e execute o notebook de cima para baixo.

A célula "Preparação do ambiente: bibliotecas, pastas do projeto e origem dos dados", a primeira célula de código da seção 2, baixa este repositório para a pasta `data-analytics-ras`. A seção 9 instala o PySpark 4.0.3 no Colab. Para iniciar o Spark, o ambiente precisa ter Java 17 ou mais recente; confira com `!java -version`. Se o Java não estiver disponível, prepare-o antes da seção 9 ou use o ambiente local descrito abaixo.

## Execução local

Requisitos: Git, Python 3.12 e Java 17 ou mais recente. O ambiente de conferência utilizou Python 3.12.14, pandas 2.3.3 e PySpark 4.0.3. O comando `java -version` deve funcionar no mesmo terminal usado para iniciar o Jupyter.

```bash
git clone https://github.com/eduardofabrii/data-analytics-ras.git
cd data-analytics-ras
python -m venv .venv
```

Ative o ambiente virtual:

```powershell
# Windows (PowerShell)
.\.venv\Scripts\Activate.ps1
```

```bash
# Linux ou macOS
source .venv/bin/activate
```

Instale as dependências e abra o notebook:

```bash
python -m pip install -r requirements.txt
python -m jupyter notebook notebook/ra1-ra2-deezer.ipynb
```

Execute todas as células na ordem. O notebook pode ser iniciado com o diretório de trabalho na raiz do clone ou em sua pasta `notebook/`.

## Dados e resultados esperados

Mantenha `BAIXAR_DA_API = False` para ler a coleta já incluída no repositório. Assim, a reprodução não depende de a API estar disponível nem de seus rankings atuais. Uma nova coleta, ativada com `True`, pode produzir resultados diferentes e levar mais tempo.

A seção 5, célula "Gravação e releitura do conjunto preparado", valida e grava o CSV com **2.800 linhas e 16 colunas**, mantendo uma ocorrência de faixa por chart e data de coleta. O [dicionário](dados/preparados/dicionario-faixas.md) explica a chave e os cuidados de interpretação.

Na seção 11, as células "Segundo lote: atualização de uma janela existente e abertura de novos intervalos" e "Terceiro lote: novas reproduções e comparação das contagens acumuladas" mostram a evolução dos totais de **30 para 60 e 90 reproduções simuladas**, agrupadas por gênero em janelas de cinco minutos. Esses eventos são artificiais e não representam audiência real da Deezer.

Para demonstrar outro lote, execute `publicar_proximo_lote(10);` em uma célula antes de "Encerramento da consulta e limpeza das subpastas da simulação". A célula de encerramento, no fim da seção 11, encerra a consulta e remove as subpastas temporárias de entrada, publicação e checkpoint, preservando os dados brutos e preparados. Para repetir a simulação, execute novamente as seções 9, 10 e 11, nessa ordem.
