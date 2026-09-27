# MVP_Engenharia_de_Dados_Absenteismo
MVP da Sprint de Engenharia de Dados do curso de Data Science e Analytics da PUC-Rio

# Análise de absenteísmo em uma empresa brasileira de serviços de courier

## 1. Contexto de Negócios e Perguntas

O absenteísmo corresponde às ausências dos colaboradores ao trabalho e pode gerar impactos significativos no dia a dia das organizações, afetando a rotina operacional e ocasionando custos relacionados ao remanejamento de pessoal, realização de horas extras e possíveis impactos nos prazos das atividades. Dessa forma, seu acompanhamento torna-se relevante para a gestão da força de trabalho.

A análise dos registros de absenteísmo permite compreender a distribuição das ausências, identificar os motivos mais representativos e reconhecer possíveis padrões de concentração. Apesar de parecer um tema simples, o absenteísmo possui diferentes dimensões que podem ser exploradas, permitindo avaliar não apenas a frequência das ocorrências, mas também o volume de horas de ausência e possíveis fatores associados ao seu comportamento.

A partir dessas informações, torna-se possível fornecer subsídios para que a área de gestão de pessoas compreenda melhor o fenômeno e identifique oportunidades de atuação que contribuam para a gestão do absenteísmo e da força de trabalho.

### 1.1 Objetivo

O objetivo deste projeto é desenvolver um pipeline de dados em ambiente cloud utilizando a arquitetura Medallion, estruturada nas camadas Bronze, Silver e Gold, para realizar a ingestão, tratamento, organização e disponibilização de dados relacionados ao absenteísmo.

A partir dos dados tratados, será realizada uma análise exploratória buscando compreender os principais padrões associados às horas de ausência registradas.

### 1.2 Perguntas de negócio

O MVP busca responder às seguintes perguntas:

**1. Como o absenteísmo se distribui ao longo dos meses e dias da semana? Existem períodos de maior concentração?**

**2. Quais são os motivos de ausência mais frequentes e quais representam o maior volume de horas de absenteísmo?**

**3. O volume de horas de ausência está concentrado em determinados colaboradores ou distribuído entre a força de trabalho?**

**4. Os colaboradores com maior recorrência de ausências são também aqueles que apresentam maior volume de horas de absenteísmo?**

## 2. Fonte e Carga dos Dados

O conjunto de dados utilizado neste projeto é o **Absenteeism at Work**, disponibilizado pelo UCI Machine Learning Repository.

A base foi construída a partir de registros de absenteísmo de uma empresa de courier no Brasil, contemplando o período de julho de 2007 a julho de 2010.

O conjunto de dados possui **740 registros e 21 atributos**, contendo informações relacionadas aos motivos das ausências, características dos colaboradores, aspectos relacionados ao trabalho e quantidade de horas de absenteísmo.

### 2.1 Fonte dos dados

- **Dataset:** [Absenteeism at Work](https://github.com/BrunoSennin/MVP_Engenharia_de_Dados_Absenteismo/blob/main/Absenteeism_at_work.csv)
- **Fonte:** UCI Machine Learning Repository
- **Período dos dados:** julho de 2007 a julho de 2010
- **Quantidade de registros:** 740
- **Quantidade de atributos:** 21
- **Formato utilizado:** CSV
- **Documentação auxiliar:** [Attribute Information](https://github.com/BrunoSennin/MVP_Engenharia_de_Dados_Absenteismo/blob/main/documentation/Attribute%20Information.md)

### 2.2 Carga dos dados

Os arquivos de origem foram carregados manualmente no ambiente **Databricks Free Edition**, utilizando um Volume do Unity Catalog para armazenamento dos arquivos utilizados no projeto.

A leitura do arquivo CSV foi realizada utilizando **PySpark**, preservando inicialmente a estrutura e os nomes dos atributos provenientes da fonte.

Após a ingestão, os dados foram persistidos em formato **Delta** na camada Bronze, na tabela:

`workspace.bronze.absenteeism_raw`

Essa tabela representa a entrada do pipeline e mantém os dados brutos antes das etapas de tratamento, padronização e enriquecimento realizadas na camada Silver.

O processo de ingestão e construção das camadas está disponível no notebook [`01_pipeline_medallion_absenteismo.ipynb`](Nootbooks/01_pipeline_medallion_absenteismo.ipynb).

## 3. Modelagem e Catálogo de Dados

### 3.1 Modelagem dos dados

A modelagem do projeto foi estruturada seguindo a arquitetura **Medallion**, com a organização dos dados nas camadas Bronze, Silver e Gold.

A escolha dessa arquitetura permite separar as diferentes etapas do processamento, mantendo os dados brutos preservados na camada inicial e disponibilizando, nas camadas seguintes, dados progressivamente tratados e preparados para análise.

A estrutura adotada foi:

- **Bronze:** armazenamento dos dados brutos provenientes do arquivo de origem, sem aplicação das regras de tratamento utilizadas nas etapas posteriores.
- **Silver:** dados padronizados, tipados e enriquecidos com descrições que facilitam sua interpretação. Nesta camada também foi criada uma tabela auxiliar contendo a descrição dos motivos de ausência.
- **Gold:** tabelas agregadas construídas a partir dos dados da camada Silver e direcionadas às perguntas de negócio definidas no projeto.

A camada Silver foi mantida em uma estrutura predominantemente **flat**, adequada às características e ao volume do conjunto de dados utilizado. A camada Gold, por sua vez, foi estruturada em tabelas agregadas específicas para cada dimensão de análise.

As tabelas que compõem o pipeline são:

| Camada | Tabela | Finalidade |
|---|---|---|
| Bronze | `absenteeism_raw` | Preservação dos dados provenientes do arquivo de origem. |
| Silver | `absenteeism` | Base principal tratada, tipada, padronizada e enriquecida. |
| Silver | `reason_lookup` | Tabela auxiliar com os códigos e descrições dos motivos de ausência. |
| Gold | `agg_absenteeism_month` | Agregação das ocorrências e horas de ausência por mês. |
| Gold | `agg_absenteeism_weekday` | Agregação das ocorrências e horas de ausência por dia da semana. |
| Gold | `agg_absenteeism_reason` | Agregação das ocorrências e horas de ausência por motivo. |
| Gold | `agg_absenteeism_employee` | Agregação das ocorrências e horas de ausência por colaborador. |

### 3.2 Arquitetura e linhagem dos dados

O fluxo dos dados ao longo do pipeline pode ser representado da seguinte forma:

```mermaid
flowchart LR
    A[Arquivo CSV] --> B[Bronze<br/>absenteeism_raw]
    B --> C[Silver<br/>absenteeism]
    C --> D[Gold<br/>agg_absenteeism_month]
    C --> E[Gold<br/>agg_absenteeism_weekday]
    C --> F[Gold<br/>agg_absenteeism_reason]
    C --> G[Gold<br/>agg_absenteeism_employee]
    C --> H[Análises e<br/>Visualizações]
    D --> H
    E --> H
    F --> H
    G --> H
