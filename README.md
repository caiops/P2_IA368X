# Projeto 2 - Predizendo Prognóstico de Mortalidade com Dados Sintéticos

## Apresentação

O presente projeto foi originado no contexto das atividades da disciplina de pós-graduação [Ciência e Visualização de Dados em Saúde](https://ds4h.org/), oferecida no primeiro semestre de 2022, na Unicamp.

| Nome | RA | Especialização |
| --- | --- | --- |
| Caio Pinheiro Santana | 218653 | Elétrica |
| Bruno Rangel Balbino dos Santos | 218450 | Elétrica |

## Contextualização da Proposta

<!-- Apresentação da proposta de predição indicando os parâmetros adotados para a mesma com a justificativa (por que esses parâmetros foram adotados?). O ideal é que a proposta seja apresentada como uma pergunta de pesquisa. -->

De forma geral, prognóstico significa prever ou estimar a probabilidade ou risco de condições futuras. Na medicina, em especial, o prognóstico geralmente está associado à probabilidade ou risco de um indivíduo desenvolver um determinado estado de saúde (um desfecho, como morte ou complicações) em um período específico, com base em seu perfil clínico e não clínico. Tal perfil pode incluir a idade, sexo, história, sintomas, sinais e outros resultados de testes do paciente[^1].

É interessante notar que essa é uma das poucas áreas da medicina que utiliza ferramentas e modelos baseados em aprendizagem de máquina no dia-a-dia clínico e não apenas de forma experimental para pesquisas. Pode-se citar, por exemplo, o APACHE[^2] (do inglês, _Acute Physiology And Chronic Health Evaluation_) e o SAPS[^3] (do inglês, _Simplified Acute Physiology Score_), modelos utilizados para prever a mortalidade hospitalar em pacientes críticos. Dessa forma, o prognóstico acaba se tornando um tema bastante interessante para a prática do ferramental relacionado à predição a partir da aprendizagem de máquina no contexto da saúde.

No entanto, existe toda uma problemática relacionada ao uso de dados reais de pacientes, especialmente em termos éticos. Uma alternativa possível para contornar esse problema e ainda utilizar dados próximos ao mundo real é a utilização de dados sintéticos de pacientes - dados fictícios gerados com base em dados reais que permitem a realização da prática desejada. Em especial, este projeto utiliza dados gerados a partir do [Synthea](https://github.com/synthetichealth/synthea), um gerador de pacientes sintéticos e realistas (mas não reais) que modela o histórico médico desses pacientes - incluindo ciclos de vida do nascimento à morte, dados de encontros, condições, medicamentos, observações, procedimentos, entre outros.

Assim, o objetivo geral deste projeto é montar um ou mais modelos de prognóstico que realizem a predição de mortalidade de pacientes sintéticos gerados em cenários fictícios. Dentre as diversas condições e doenças disponíveis nos conjuntos de dados disponibilizados, optou-se por explorar os casos de neutropenia febril, um tema que já havia sido explorado no primeiro projeto da disciplina.

Frequentemente, os pacientes que passam por quimioterapia apresentam uma diminuição na contagem de células sanguíneas. Em especial, a diminuição da contagem dos neutrófilos - glóbulos brancos responsáveis pela defesa contra bactérias e fungos - se denomina neutropenia. Tal quadro aumenta o risco de infecção e febre, podendo gerar o que se conhece como neutropenia febril (NF).

A NF é definida por uma temperatura oral maior do que 38.5°C ou duas medidas consecutivas acima de 38°C por 2 horas, além de uma contagem absoluta de neutrófilos abaixo de 500/uL ou que se espera que caia abaixo desse valor. Mesmo com avanços na prevenção e tratamento, a NF continua sendo uma das complicações mais preocupantes da quimioterapia, sendo uma das principais causas de morbidade, uso de recursos de saúde e de redução de sua eficácia devido a atrasos e reduções de dose na quimioterapia.

Apesar dos grandes avanços na prevenção e tratamento, a NF continua sendo uma das complicações mais preocupantes da quimioterapia do câncer, sendo uma das principais causas de morbidade, uso de recursos de saúde e eficácia comprometida devido a atrasos e reduções de dose na quimioterapia[^4]. Dessa forma, o objetivo principal desse projeto é criar modelos de prognóstico para tentar predizer a morte de pacientes sintéticos que foram diagnósticados com NF, ou seja, predizer se tais pacientes virão a óbito ou não em decorrência da NF. Define-se, então, a seguinte questão de pesquisa:

<!-- Comentar aqui quais os parâmetros que serão utilizados para a predição? Comentar que é um modelo de classificação pq a ideia é definir se o paciente vai morrer ou não (categórico) em decorrência da NF? -->

- É possível criar modelos de prognóstico para predizer a morte em decorrência da NF a partir dos dados sintéticos disponibilizados?
- Quais os resultados obtidos pelos modelos? Qual deles se desempenhou melhor?
- O modelo que apresentou os melhores resultados é robusto o suficiente para ser aplicado em um conjunto diferente de dados?

### Ferramentas

- Jupyter Notebooks (linguagem Python) através do Google Colab - utilizado para explorar os dados e extrair as _features_ a serem utilizadas pelo modelo;
- Orange Workflows (versão 3.31.1) - utilizado para treinar/validar/testar o modelo.

<!-- Listagem das ferramentas utilizadas (na forma de itens). -->

## Metodologia

<!-- Abordagem adotada pelo projeto na predição. Justificar as escolhas e (opcionalmente) apresentar fundamentos teóricos. -->

### Bases Adotadas para o Estudo

Como citado anteriormente, este projeto utiliza dados sintéticos gerados através do [Synthea](https://github.com/synthetichealth/synthea). Foram disponibilizados quatro diferentes conjuntos de dados, variando principalmente em relação ao número de pacientes disponíveis. Neste projeto, utilizou-se apenas as bases abaixo por possuírem um menor número de pacientes, facilitando sua exploração e manipulação.

- scenario01 - inclui dados de 1174 pacientes;
- scenario02 - inclui dados de 1121 pacientes.

Para cada uma das bases, 18 arquivos .csv foram disponibilizados. Maiores detalhes sobre as tabelas e os dados contidos em cada uma podem ser encontrados na [Wiki do GitHub do Synthea](https://github.com/synthetichealth/synthea/wiki/CSV-File-Data-Dictionary).

Nem todas as tabelas foram utilizadas neste projeto. Exploramos apenas 9 delas, as quais julgamos possivelmente terem maior relação com a condição de interesse. Foram elas:

- Pacientes (patients.csv) - por conter as informações básicas sobre os pacientes. Informações como idade e sexo, por exemplo, podem ser muito úteis para o modelo de predição;
- Condições (conditions.csv) - por conter as condições e diagnósticos dos pacientes. Não apenas apresenta a condição de interesse (NF), mas possívelmente outras condições que podem estar relacionadas e impactar na predição de mortalidade;
- Encontros (encounters.csv) - por conter os encontros pelos quais os pacientes passaram. Neste caso, a ideia foi de que os tipos e frequências dos encontros poderiam trazer informações úteis sobre a saúde dos pacientes.
- Medicamentos (medications.csv) - a utilização de algum medicamento, especialmente ligado diretamente ao diagnóstico, pode influenciar a mortalidade desses pacientes.
- Planos de cuidado (careplans.csv)
- Imunizações (immunizations.csv)
- Procedimentos (procedures.csv)
- Observações (observations.csv)
- Alergias (allergies.csv)

Todas as tabelas utilizadas podem ser encontradas no diretório /data/external, separadas por cenário. A única exceção é a tabela de observações. Devido ao seu tamanho, não foi possível colocá-la diretamente no GitHub. Ao invés disso, disponibilizamos as tabelas de observações já filtradas para conter apenas os dados dos pacientes de interesse em /data/interim/interest/ - tabelas observations01_i.csv e observations02_i.csv.

### Exploração dos Dados e Extração das _Features_



### Criação dos Modelos

Como pretende-se prever uma resposta binária - a morte ou sobrevivência do paciente em decorrência da NF - o modelo 



## Resultados Obtidos

## Evolução do Projeto

## Discussão

## Conclusão

### Principais Desafios Enfrentados e Lições Aprendidas

### Trabalhos Futuros

## Referências Bibliográficas

[^1]: MOONS, Karel GM et al. Prognosis and prognostic research: what, why, and how?. Bmj, v. 338, 2009.

[^2]: KNAUS, William A. et al. The APACHE III prognostic system: risk prediction of hospital mortality for critically III hospitalized adults. Chest, v. 100, n. 6, p. 1619-1636, 1991.

[^3]: LE GALL, Jean-Roger; LEMESHOW, Stanley; SAULNIER, Fabienne. A new simplified acute physiology score (SAPS II) based on a European/North American multicenter study. Jama, v. 270, n. 24, p. 2957-2963, 1993.

[^4] DE NAUROIS, J. et al. Management of febrile neutropenia: ESMO clinical practice guidelines. Annals of Oncology, v. 21, p. v252-v256, 2010.
