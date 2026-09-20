# MVP_Engenharia_de_Dados_Absenteismo
MVP da Sprint de Engenharia de Dados do cruso de Data Sciencie e Analytics da PUC-Rio
# Análise de Absenteísmo no Trabalho utilizando Arquitetura Medallion

## 1. Contexto de Negócios e Perguntas

O absenteísmo corresponde às ausências dos colaboradores ao trabalho e seu acompanhamento é relevante para a gestão da força de trabalho. A análise desses registros permite compreender a distribuição das ausências, identificar os motivos mais representativos e reconhecer possíveis padrões de concentração.

### 1.1 Objetivo

O objetivo deste projeto é desenvolver um pipeline de dados em ambiente cloud utilizando a arquitetura Medallion, estruturada nas camadas Bronze, Silver e Gold, para realizar a ingestão, tratamento, organização e disponibilização de dados relacionados ao absenteísmo.

A partir dos dados tratados, será realizada uma análise exploratória buscando compreender os principais padrões associados às horas de ausência registradas.

### 1.2 Perguntas de negócio

O projeto busca responder às seguintes perguntas:

1. Quais motivos apresentam maior frequência de ocorrências e quais geram maior volume de horas de ausência?
2. Como o absenteísmo se distribui ao longo dos meses e dias da semana? Existem períodos de maior concentração?
3. O volume de horas de ausência está concentrado em determinados colaboradores ou distribuído entre os colaboradores da organização?
4. Quais características disponíveis na base apresentam associação com maiores níveis de absenteísmo?
## 2. Fonte e Carga dos Dados

O conjunto de dados utilizado neste projeto é o **Absenteeism at Work**, disponibilizado pelo UCI Machine Learning Repository.

A base foi construída a partir de registros de absenteísmo de uma empresa de courier no Brasil, contemplando o período de julho de 2007 a julho de 2010.

O conjunto de dados possui **740 registros e 21 atributos**, contendo informações relacionadas aos motivos das ausências, características dos colaboradores, aspectos relacionados ao trabalho e quantidade de horas de absenteísmo.

### 2.1 Fonte dos dados

- **Dataset:** Absenteeism at Work
- **Fonte:** UCI Machine Learning Repository
- **Período dos dados:** julho de 2007 a julho de 2010
- **Quantidade de registros:** 740
- **Quantidade de atributos:** 21
- **Formato utilizado:** CSV
