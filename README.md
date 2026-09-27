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

### 3.2 Catálogo de Dados

O catálogo de dados apresenta a estrutura das principais tabelas utilizadas nas camadas Silver e Gold, incluindo os atributos, tipos de dados e suas respectivas descrições.

A camada Bronze preserva os 21 atributos provenientes do arquivo de origem, enquanto a camada Silver realiza a padronização dos nomes, tipagem dos dados e inclusão de atributos descritivos. A camada Gold contém as estruturas agregadas utilizadas para apoiar as análises de negócio.

#### 3.2.1 Silver — `absenteeism`

Tabela principal tratada e enriquecida do projeto.

| Atributo | Tipo | Descrição |
|---|---|---|
| `id` | INT | Identificador do colaborador. |
| `reason_code` | INT | Código correspondente ao motivo da ausência. |
| `month_code` | INT | Código numérico do mês da ausência. |
| `day_of_week_code` | INT | Código do dia da semana da ausência. |
| `season_code` | INT | Código da estação do ano. |
| `transportation_expense` | INT | Despesa de transporte do colaborador. |
| `distance_from_residence_to_work` | INT | Distância entre residência e trabalho, em quilômetros. |
| `service_time` | INT | Tempo de serviço do colaborador. |
| `age` | INT | Idade do colaborador. |
| `work_load_average_day` | DOUBLE | Indicador de carga média de trabalho por dia. |
| `hit_target` | INT | Indicador relacionado ao atingimento da meta. |
| `disciplinary_failure` | INT | Indicador de ocorrência disciplinar: 0 = não e 1 = sim. |
| `education_code` | INT | Código referente ao nível de escolaridade. |
| `son` | INT | Quantidade de filhos. |
| `social_drinker` | INT | Indicador de consumo social de álcool: 0 = não e 1 = sim. |
| `social_smoker` | INT | Indicador de tabagismo: 0 = não e 1 = sim. |
| `pet` | INT | Quantidade de animais de estimação. |
| `weight` | INT | Peso do colaborador. |
| `height` | INT | Altura do colaborador. |
| `body_mass_index` | INT | Índice de massa corporal (IMC). |
| `absenteeism_time_in_hours` | INT | Quantidade de horas de ausência registrada. |
| `month_name` | STRING | Descrição do mês da ausência. |
| `day_of_week_name` | STRING | Descrição do dia da semana da ausência. |
| `season_name` | STRING | Descrição da estação do ano. |
| `education_description` | STRING | Descrição do nível de escolaridade. |
| `reason_description` | STRING | Descrição do motivo da ausência. |

#### 3.2.2 Silver — `reason_lookup`

Tabela auxiliar utilizada para relacionar os códigos dos motivos de ausência às respectivas descrições.

| Atributo | Tipo | Descrição |
|---|---|---|
| `reason_code` | INT | Código do motivo da ausência. |
| `reason_description` | STRING | Descrição correspondente ao motivo da ausência. |

Os códigos documentados pela fonte compreendem os valores de **1 a 28**. O código `0`, identificado nos dados de origem, foi preservado na tabela principal e classificado no projeto como **"Não especificado na documentação"**.

#### 3.2.3 Gold — `agg_absenteeism_month`

Tabela agregada utilizada para analisar a distribuição do absenteísmo por mês.

| Atributo | Tipo | Descrição |
|---|---|---|
| `month_code` | INT | Código numérico do mês. |
| `month_name` | STRING | Descrição do mês. |
| `absence_occurrences` | LONG | Quantidade de registros de ausência no mês. |
| `total_absence_hours` | LONG | Total de horas de ausência no mês. |
| `average_absence_hours` | DOUBLE | Média de horas de ausência por registro no mês. |

#### 3.2.4 Gold — `agg_absenteeism_weekday`

Tabela agregada utilizada para analisar a distribuição do absenteísmo por dia da semana.

| Atributo | Tipo | Descrição |
|---|---|---|
| `day_of_week_code` | INT | Código do dia da semana. |
| `day_of_week_name` | STRING | Descrição do dia da semana. |
| `absence_occurrences` | LONG | Quantidade de registros de ausência no dia da semana. |
| `total_absence_hours` | LONG | Total de horas de ausência no dia da semana. |
| `average_absence_hours` | DOUBLE | Média de horas de ausência por registro. |

#### 3.2.5 Gold — `agg_absenteeism_reason`

Tabela agregada utilizada para analisar frequência e volume de horas por motivo de ausência.

| Atributo | Tipo | Descrição |
|---|---|---|
| `reason_code` | INT | Código do motivo da ausência. |
| `reason_description` | STRING | Descrição do motivo da ausência. |
| `absence_occurrences` | LONG | Quantidade de registros associados ao motivo. |
| `total_absence_hours` | LONG | Total de horas de ausência associado ao motivo. |
| `average_absence_hours` | DOUBLE | Média de horas de ausência por registro do motivo. |

#### 3.2.6 Gold — `agg_absenteeism_employee`

Tabela agregada utilizada para analisar recorrência, volume e concentração das ausências por colaborador.

| Atributo | Tipo | Descrição |
|---|---|---|
| `id` | INT | Identificador do colaborador. |
| `absence_occurrences` | LONG | Quantidade de registros de ausência do colaborador. |
| `total_absence_hours` | LONG | Total de horas de ausência do colaborador. |
| `average_absence_hours` | DOUBLE | Média de horas de ausência por registro do colaborador. |
| `percentage_total_hours` | DOUBLE | Participação percentual do colaborador no total de horas de ausência. |

## 4. Pipeline de Dados

O pipeline de dados foi desenvolvido no **Databricks Free Edition**, utilizando **PySpark** para realizar as etapas de ingestão, transformação, enriquecimento e agregação dos dados.

O processamento segue a arquitetura Medallion e foi implementado no notebook [`01_pipeline_medallion_absenteismo.ipynb`](./Notebooks/01_pipeline_medallion_absenteismo.ipynb).

### 4.1 Camada Bronze

A camada Bronze representa a entrada do pipeline. O arquivo CSV foi lido utilizando PySpark e persistido em formato Delta na tabela:

`workspace.bronze.absenteeism_raw`

Nesta etapa, foram preservados os nomes dos atributos e a estrutura proveniente da fonte, mantendo uma representação dos dados antes das transformações realizadas nas camadas posteriores.

### 4.2 Camada Silver

A camada Silver foi construída a partir dos dados da Bronze e concentra as principais transformações do pipeline.

Foram realizadas as seguintes etapas:

- padronização dos nomes dos atributos;
- conversão dos atributos para os tipos de dados definidos no projeto;
- criação das descrições de mês, dia da semana, estação e escolaridade;
- criação da tabela auxiliar `reason_lookup` com os códigos e descrições dos motivos de ausência;
- enriquecimento da base principal com a descrição dos motivos de ausência;
- tratamento dos códigos não descritos na documentação da fonte, preservando os valores originais.

Como resultado, foram persistidas as tabelas:

`workspace.silver.absenteeism`

`workspace.silver.reason_lookup`

### 4.3 Camada Gold

A camada Gold foi construída exclusivamente a partir dos dados tratados da camada Silver e tem como finalidade disponibilizar estruturas diretamente relacionadas às perguntas de negócio.

Foram criadas quatro tabelas agregadas:

- `workspace.gold.agg_absenteeism_month`
- `workspace.gold.agg_absenteeism_weekday`
- `workspace.gold.agg_absenteeism_reason`
- `workspace.gold.agg_absenteeism_employee`

As agregações apresentam indicadores como quantidade de ocorrências, total de horas de ausência e média de horas, considerando diferentes perspectivas de análise.

A tabela por colaborador também apresenta sua participação percentual no total de horas de ausência, permitindo avaliar a concentração do absenteísmo entre os colaboradores.

As tabelas da camada Gold são utilizadas como principal fonte do notebook [`02_analise_absenteismo.ipynb`](./Notebooks/02_analise_absenteismo.ipynb).
