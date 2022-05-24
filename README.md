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

## Metodologia

<!-- Abordagem adotada pelo projeto na predição. Justificar as escolhas e (opcionalmente) apresentar fundamentos teóricos. -->

### Bases Adotadas para o Estudo

Como citado anteriormente, este projeto utiliza dados sintéticos gerados através do [Synthea](https://github.com/synthetichealth/synthea). Foram disponibilizados quatro diferentes conjuntos de dados, variando principalmente em relação ao número de pacientes disponíveis. Neste projeto, utilizou-se apenas as bases abaixo por possuírem um menor número de pacientes, facilitando sua exploração e manipulação.

- scenario01 - inclui dados de 1174 pacientes;
- scenario02 - inclui dados de 1121 pacientes.

Para cada uma das bases, 18 arquivos .csv foram disponibilizados. Maiores detalhes sobre as tabelas e os dados contidos em cada uma podem ser encontrados na [Wiki do GitHub do Synthea](https://github.com/synthetichealth/synthea/wiki/CSV-File-Data-Dictionary).

Nem todas as tabelas foram utilizadas neste projeto. Exploramos apenas 8 delas, as quais julgamos possivelmente terem maior relação com a condição de interesse. Foram elas:

- Pacientes (patients.csv);
- Condições (conditions.csv);
- Encontros (encounters.csv);
- Medicamentos (medications.csv);
- Planos de cuidados (careplans.csv);
- Imunizações (immunizations.csv);
- Procedimentos (procedures.csv);
- Observações (observations.csv).

Todas as tabelas utilizadas podem ser encontradas no diretório /data/external, separadas por cenário. A única exceção é a tabela de observações. Devido ao seu tamanho, não foi possível colocá-la diretamente no GitHub. Ao invés disso, disponibilizamos as tabelas de observações já filtradas para conter apenas os dados dos pacientes de interesse em /data/interim/interest/ - tabelas observations01_i.csv e observations02_i.csv.

### Exploração dos Dados e Extração das _Features_

A exploração dos dados e extração das _features_ foi realizada por meio dos notebooks que podem ser encontrados no diretório /notebooks. Optamos por dividir os processamentos realizados em alguns notebooks, de modo a facilitar sua manipulação e entendimento. Eles foram criados e executados na ordem em que serão apresentados a seguir. Para permitir sua execução de forma independente, algumas tabelas de dados processados foram salvas no diretório /data/interim/interest, enquanto as tabelas de _features_ geradas por cada notebook foram salvas no diretório /data/interim/features. Dessa forma, os notebooks que se utilizam de processamentos anteriores podem ser executados diretamente utilizando esses dados intermediários, sem a necessidade de que todos os processamentos sejam refeitos em cada notebook. Ainda, todos eles foram criados e configurados de modo a permitir sua execução por qualquer pessoa através do Google Colab.

O primeiro notebook (definition_and_basic_features.ipynb) foi utilizado para uma exploração inicial dos dados e geração das primeiras _features_. As tabelas de condições e encontros foram analisadas para verificar quais as possibilidades disponíveis e definir qual seria a proposta do projeto. Após definir que iríamos trabalhar com a NF, passamos a explorar os dados relacionados a essa condição. Em especial, buscamos o diagóstico de algum tipo de câncer, uma vez que a NF é atrelada ao tratamento quimioterápico para câncer.

Por fim, começamos a criar as tabelas de _features_ a serem utilizadas para construção do modelo. Nesta etapa, extraímos apenas dados básicos sobre os pacientes (através da tabela patients.csv), o desfecho se o paciente havia morrido ou não em decorrência da NF (a informação que o modelo deve prever) e a idade do paciente na data de início da NF (obtida ao relacionar sua data de nascimento e a data de início da NF e convertida em anos ao considerar um ano como 365 dias).

Em seguida, o notebook features_conditions_encounters.ipynb foi utilizado para explorar um pouco mais as tabelas de condições e encontros e possivelmente extrair mais algumas _features_ interessantes. Vale notar que tomamos o cuidado de filtrar as condições e encontros de forma a considerar apenas aqueles que se iniciaram antes ou no início da NF. Afinal, não faria sentido considerar para a predição alguma condição ou encontro que ocorreu após a NF, o que possivelmente enviesaria o modelo criado. Também verificamos se alguns dados que nos pareceram interessantes estavam atrelados ao desfecho dos pacientes. Ainda, em alguns casos, verificamos se a idade dos pacientes parecia ter relação com a potencial _feature_ sob análise, na tentativa de evitar utilizar duas _features_ muito correlacionadas, o que poderia prejudicar a performance do modelo.

Então, exploramos a tabela de medicamentos através do notebook features_medications.ipynb. Novamente, filtramos a data de início da administração dos medicamentos para considerar apenas aqueles que começaram a ser administrados antes ou no início da NF e verificamos se alguns dados que nos pareceram interessantes estavam atrelados ao desfecho dos pacientes. Em especial, buscamos por medicamentos administrados em razão da leucemia mielóide aguda (que, como apresentado nos resultados obtidos, estava atrelada à NF).

Ainda, com o notebook exploring_careplans_immunizations_observations_procedures.ipynb exploramos as tabelas de planos de cuidados, imunizações, procedimentos e observações, na tentativa de obter mais algumas _features_ de interesse. Novamente, filtramos as datas para considerar apenas os dados anteriores ou iguais ao início da NF. No entanto, no caso das observações e dos procedimentos, acabamos notando que utilizar esse limite para as datas acarretava na perda de informações importantes e ele foi descartado - tomando-se os devidos cuidados para evitar possíveis vieses.

O último notebook (features_analyses_final.ipynb) foi utilizado para realizar algumas análises do conjunto final de _features_ selecionadas, bem como para gerar as tabelas finais, prontas para serem utilizadas no Orange. Essas tabelas podem ser encontradas no diretório /data/processed e incluem os dados de cada cenário separadamente (scenario01.csv e scenario02.csv), bem como uma tabela com os dados dos dois cenários em conjunto (all.csv).

### Criação dos Modelos

Como pretende-se prever uma resposta binária - a morte ou sobrevivência do paciente em decorrência da NF - o modelo 



## Resultados Obtidos

### Exploração dos Dados e Extração das _Features_

#### definition_and_basic_features.ipynb

A partir dos processamentos realizados, obtivemos 139 pacientes com NF no cenário 01 e 117 no cenário 02. Cada paciente apresentou a condição apenas uma vez. Destes, 31 pacientes haviam morrido no cenário 01 e 21 no cenário 02. No entanto, apenas 26 e 13 pacientes, respectivamente, morreram em decorrência da NF e todos eles morreram no mesmo dia em que a NF começou (com exceção de um paciente do cenário 01 que morreu no dia seguinte ao início da NF). Ainda, para os pacientes que morreram, mas não em decorrência da NF, a condição se encerrou no mesmo dia em que se iniciou e eles vieram a óbito no mínimo 202 dias após a NF.

Todos esses pacientes também apresentaram o diagnóstico de leucemia mielóide aguda (acute myeloid leukemia disease). Novamente, a única exceção se deu para um paciente do cenário 01, que não apresentou dados para essa condição nem para algum outro tipo de câncer. (ATRELAR COM A QUESTÃO DOS PROCEDIMENTOS DEPOIS, QUE CONFIRMOU QUE ERA UM ERRO).

Dentre os dados básicos disponíveis sobre os pacientes, optamos por utilizar como _features_ apenas sua raça, etnia e gênero. Estes fatores podem ser importantes por questões socio-econômicas - como diferentes níveis de acesso a serviços de saúde - ou até mesmo por questões genéticas - como algum tipo de predisposição ao desenvolvimento da NF. Ainda, descartamos a possibilidade de utilizar dados relacionados ao endereço dos pacientes, uma vez que todos os pacientes de cada cenário residem em um mesmo estado, mas que difere entre os cenários (Massachusetts no 01 e Alaska no 02). Um modelo treinado utilizando dados desse tipo como _features_ poderia acabar se tornando muito específico para os pacientes de regiões específicas, dificultando sua generalização para pacientes de outras regiões.

A idade do paciente no início da NF também foi utilizada como _feature_, uma vez que esse fator pode influenciar suas chances de sobrevivência. Por exemplo, pacientes idosos apresentam maior risco de neutropenia febril após quimioterapia, com piores taxas de morbidade e mortalidade[^4]. Em ambos os cenários, as idades dos pacientes variaram de 0 a 21 anos. Vale notar que consideramos utilizar também a idade dos pacientes no momento do diagnóstico de leucemia mielóide aguda. No entanto, todos os pacientes receberam esse diagnóstico no mesmo dia da NF ou apenas um dia antes.

#### features_conditions_encounters.ipynb

Dentre as condições analisadas, a bacteremia - presença de bactéria no sangue - chamou nossa atenção por aparecer um número razoável de vezes e em proporção maior nos pacientes que morreram em decorrência da NF. De fato, o prognóstico da NF tende a ser pior em pacientes com bacteremia[^4]. Observamos ainda que cada paciente apresentou essa condição apenas uma vez e exatamente no dia de início da NF. Dessa forma, adicionamos a presença ou não de bacteremia como uma _feature_ a ser utilizada para criar o modelo de predição.

Quanto aos encontros, nos chamaram a atenção os do tipo _wellness_ e os ambulatoriais. No primeiro caso, imaginamos que, possivelmente, pacientes que passaram por mais visitas de rotina na infância teriam uma menor probabilidade de morrer em decorrência da NF, já que eles foram mais bem acompanhados. No entanto, encontramos o oposto, com uma maior proporção de mortos entre aqueles que passaram por mais encontros do tipo _wellness_. Ainda assim, a contagem de encontros deste tipo para cada paciente foi adotada como _feature_ para o modelo.

Já no segundo caso, acabamos utilizando como _feature_ a contagem de encontros ambulatóriais relacionados a sintomas. Não apenas os encontros relacionados a sintomas foram os mais comuns, como eles nos pareceram ter mais relação com a NF. Encontros ambulatoriais não atrelados a sintomas incluiam encontros por fraturas e queimaduras, por exemplo, enquanto os atrelados a sintomas incluiam principalmente sinusite viral, faringite viral aguda, bronquite aguda, entre outros. Nossa ideia foi de que pacientes que passaram por mais encontros deste tipo possivelmente possuíam um sistema imunológico mais debilitado - levando a presença dessas condições com maior frequência e possivelmente tendo algum impacto no caso da NF.

#### features_medications.ipynb

Dentre os medicamentos disponíveis, encontramos apenas um relacionado à leucemia mielóide aguda - levofloxacin 500 MG Oral Tablet, utilizado para tratar uma variedade de infecções bacterianas[^5]. Ainda, em todos os casos, esse medicamento foi administrado no dia de início da NF. Dessa forma, adicionamos a administração ou não desse remédio como uma _feature_ a ser considerada.

#### exploring_careplans_immunizations_observations_procedures.ipynb

Ao explorar os planos de cuidados e as imunizações, acabamos tendo dificuldade para relacionar os dados encontrados com a condição de interesse e acabamos não utilizando nenhuma informação dessas tabelas como _features_. Ainda, em relação aos procedimentos realizados, não encontramos dados que nos pareceram interessantes para utilizar como _features_. No entanto, foi possível confirmar que todos os pacientes de interesse realmente passaram pela quimioterapia e que o número de pacientes que morreram no hospital foi igual ao número de pacientes que morreram em decorrência da NF que havíamos obtido nas análises anteriores.

Já em relação à tabela de observações, encontramos duas observações diretamente relacionadas à NF: a contagem de neutrófilos no sangue e a temperatura corporal dos pacientes. Em ambos os casos, cada paciente apresentou a observação apenas uma vez, mesmo desconsiderando a data limite relacionada ao início da NF. Assim, ambos os dados de observações foram adicionados às _features_.

### Análise das _Features_ Selecionadas

Após a exploração dos dados, foram extraídas um total de 10 _features_: raça, etnia e gênero do paciente; idade do paciente na data de início da NF; se apresentou ou não bacteremia; contagem de encontros do tipo _wellness_ e de encontros do tipo ambulatorial relacionados a sintomas até a data de início da NF; se o medicamento levofloxacin 500 MG Oral Tablet foi administrado ou não ao paciente; a contagem de neutrófilos e a temperatura corporal do paciente.



### Resultados de Predição

## Evolução do Projeto

## Discussão



## Conclusão

### Principais Desafios Enfrentados e Lições Aprendidas

### Trabalhos Futuros

## Referências Bibliográficas

[^1]: MOONS, Karel GM et al. Prognosis and prognostic research: what, why, and how?. Bmj, v. 338, 2009.

[^2]: KNAUS, William A. et al. The APACHE III prognostic system: risk prediction of hospital mortality for critically III hospitalized adults. Chest, v. 100, n. 6, p. 1619-1636, 1991.

[^3]: LE GALL, Jean-Roger; LEMESHOW, Stanley; SAULNIER, Fabienne. A new simplified acute physiology score (SAPS II) based on a European/North American multicenter study. Jama, v. 270, n. 24, p. 2957-2963, 1993.

[^4]: DE NAUROIS, J. et al. Management of febrile neutropenia: ESMO clinical practice guidelines. Annals of Oncology, v. 21, p. v252-v256, 2010.

[^5]: Levofloxacin - Uses, Side Effects, and More. Disponível em <https://www.webmd.com/drugs/2/drug-14495-8235/levofloxacin-oral/levofloxacin-oral/details>. Último acesso em 23/05/2022.
