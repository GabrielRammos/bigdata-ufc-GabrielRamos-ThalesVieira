# Etapa 4 — Modelo de Machine Learning para Detecção de Fraude

**Disciplina:** Big Data  
**Alunos:** Gabriel Ramos e Thales Vieira  

## 1. Objetivo

Como etapa adicional do trabalho, desenvolvemos um modelo de Machine Learning para verificar se as informações disponíveis na base da TechPay poderiam ser utilizadas para identificar transações com maior probabilidade de fraude.

Optamos pela Regressão Logística por ser um modelo relativamente simples e, principalmente, por permitir interpretar os coeficientes das variáveis. Dessa forma, além de avaliar o desempenho do modelo, também conseguimos observar quais características contribuíram para aumentar ou reduzir a probabilidade estimada de fraude.

O objetivo desta etapa não foi desenvolver um modelo pronto para ser utilizado em produção, mas criar uma primeira aplicação de classificação utilizando os dados trabalhados nas etapas anteriores.

## 2. Preparação da base

A variável utilizada como resposta foi `is_fraud`, que identifica se determinada transação foi ou não classificada como fraude.

Foram utilizadas as seguintes variáveis no modelo:

- valor da transação (`amount`);
- score de risco (`risk_score`);
- score de crédito (`credit_score`);
- hora da transação;
- dia da semana;
- tipo de transação (`transaction_type`);
- canal (`channel`);
- categoria do estabelecimento (`merchant_category`);
- segmento do cliente (`segment`).

Consideramos importante manter `channel` e `merchant_category` porque as análises realizadas na Etapa 3 mostraram diferenças relevantes na ocorrência de fraude entre canais e categorias. Portanto, retirar essas informações faria o modelo perder parte do contexto que já havia se mostrado importante na análise exploratória.

A hora e o dia da semana foram derivados do `timestamp`, permitindo incorporar informações relacionadas ao momento em que a transação ocorreu.

## 3. Divisão entre treino e teste

A base foi dividida em 80% para treinamento e 20% para teste.

Utilizamos `random_state=123` para permitir a reprodução do experimento e a estratificação pela variável `is_fraud` para manter aproximadamente a mesma proporção de fraudes nos conjuntos de treino e teste.

Essa decisão foi importante porque a base é desbalanceada. Das 30.000 transações, somente 751 são fraudulentas, correspondendo a aproximadamente 2,5% do total.

Por esse motivo, a acurácia isoladamente não seria uma boa métrica para avaliar o modelo. Um modelo que classificasse praticamente todas as transações como legítimas poderia apresentar uma acurácia alta e, mesmo assim, ter pouca utilidade para detectar fraude.

## 4. Tratamento das variáveis

As variáveis numéricas foram padronizadas antes do treinamento. Já as variáveis categóricas foram transformadas utilizando One-Hot Encoding.

No One-Hot Encoding utilizamos `drop='first'`, fazendo com que uma categoria de cada variável seja utilizada como referência. Essa escolha também facilita a interpretação posterior dos coeficientes da Regressão Logística.

Como a quantidade de fraudes é muito menor que a quantidade de transações legítimas, utilizamos `class_weight='balanced'`. Dessa forma, o algoritmo atribui maior peso à classe menos frequente durante o treinamento.

## 5. Avaliação do modelo

A principal métrica utilizada para avaliar a capacidade geral de discriminação do modelo foi a AUC-ROC.

O modelo apresentou uma AUC próxima de 0,77. Esse resultado indica que existe capacidade de diferenciação entre transações fraudulentas e legítimas, embora ainda exista espaço para melhoria.

Por se tratar de uma Regressão Logística relativamente simples e de uma primeira aplicação sobre a base, consideramos o resultado adequado como referência inicial.

Além da AUC, analisamos Precision, Recall, F1-score e a matriz de confusão, pois essas métricas ajudam a entender melhor o comportamento do modelo diante do desbalanceamento da base.

## 6. Matriz de confusão e threshold

Para a primeira avaliação utilizamos o threshold padrão de 0,50. Isso significa que uma transação foi classificada como fraude quando a probabilidade estimada pelo modelo foi igual ou superior a 50%.

A matriz de confusão permitiu separar os resultados em quatro situações:

- **True Negative (TN):** transações legítimas classificadas corretamente;
- **False Positive (FP):** transações legítimas sinalizadas como fraude;
- **False Negative (FN):** fraudes que o modelo não conseguiu identificar;
- **True Positive (TP):** fraudes identificadas corretamente.

O resultado mostrou que o modelo conseguiu identificar uma parcela relevante das fraudes, apresentando recall próximo de 67%.

Por outro lado, também ocorreu uma quantidade considerável de falsos positivos. Esse comportamento pode ser explicado, em parte, pelo uso de `class_weight='balanced'`, já que o modelo passa a dar maior importância à identificação da classe minoritária.

Nesse caso, existe uma troca entre aumentar a capacidade de encontrar fraudes e aumentar a quantidade de transações legítimas encaminhadas incorretamente para análise.

Por isso, entendemos que o threshold de 0,50 não deveria ser adotado automaticamente em uma situação real.

## 7. Interpretação do threshold para o negócio

Em uma operação real da TechPay, a escolha do threshold dependeria principalmente do custo associado aos dois tipos de erro.

Um falso negativo pode representar uma fraude que passou pelo sistema sem ser identificada. Já um falso positivo pode representar uma transação legítima encaminhada para revisão ou até mesmo bloqueada incorretamente.

Se a prioridade fosse identificar o maior número possível de fraudes, seria possível trabalhar com um threshold mais baixo, aumentando o recall. Entretanto, isso provavelmente aumentaria também a quantidade de falsos positivos.

Por outro lado, um threshold mais alto reduziria a quantidade de alertas, mas poderia fazer com que mais fraudes deixassem de ser identificadas.

Assim, a escolha deveria considerar tanto o desempenho estatístico quanto a capacidade da equipe de risco para analisar os alertas gerados.

## 8. Interpretação dos coeficientes

Uma das razões para escolhermos a Regressão Logística foi a possibilidade de analisar seus coeficientes.

De forma simplificada, um coeficiente positivo indica que o aumento daquela variável, ou a presença daquela categoria, está associado a um aumento na chance estimada de fraude. Um coeficiente negativo indica uma associação no sentido contrário.

Também calculamos o Odds Ratio a partir dos coeficientes. Valores superiores a 1 indicam aumento das chances estimadas, enquanto valores inferiores a 1 indicam redução.

No caso das variáveis categóricas, essa interpretação deve ser feita sempre em comparação com a categoria utilizada como referência pelo modelo.

É importante destacar que os coeficientes não representam necessariamente relações de causa e efeito. Eles mostram apenas como as variáveis contribuíram para as previsões dentro da base utilizada e considerando as demais características presentes no modelo.

## 9. Relação com as análises anteriores

Os resultados do modelo complementam o que foi observado na Etapa 3.

Na análise exploratória, verificamos que a fraude não ocorre com a mesma frequência em todos os grupos. O canal `app`, por exemplo, apresentou uma taxa de fraude superior aos demais canais, enquanto a categoria `viagem` também se destacou.

Quando cruzamos as duas características, a combinação entre `app` e `viagem` apresentou uma taxa de fraude ainda maior.

Por esse motivo, consideramos importante que canal e categoria também fizessem parte do modelo de Machine Learning. A análise mostra que observar somente uma variável, como o `risk_score`, pode não ser suficiente para representar todo o comportamento relacionado à fraude.

## 10. Conclusão

A aplicação da Regressão Logística mostrou que as variáveis disponíveis na base possuem capacidade de contribuir para a identificação de transações fraudulentas.

A AUC próxima de 0,77 indica uma capacidade razoável de separação entre as duas classes. Ao mesmo tempo, a matriz de confusão mostrou uma questão importante: aumentar a capacidade de identificar fraudes também pode gerar um número elevado de falsos positivos.

Por isso, não entendemos o modelo desenvolvido como uma solução final para a TechPay. Ele funciona como uma primeira referência que poderia ser comparada posteriormente com outros algoritmos e diferentes configurações.

Como próximos passos, seria possível testar outros thresholds, comparar a Regressão Logística com modelos mais complexos e avaliar os resultados considerando custos de negócio. Dessa forma, a decisão não dependeria somente da melhor métrica estatística, mas também do impacto que os erros do modelo poderiam gerar na operação.