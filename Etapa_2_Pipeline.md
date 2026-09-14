**Etapa 2 — Execução do Pipeline**

## Ingestão dos dados

A primeira etapa do trabalho foi realizar a ingestão do arquivo avaliacao_transactions.csv para o ambiente de processamento. Como a base foi disponibilizada de forma consolidada em um único arquivo e não existe chegada contínua de novas transações durante a execução da atividade, optei por realizar a leitura em batch utilizando PySpark. A escolha do PySpark também foi feita pensando na continuidade do pipeline. Em vez de utilizar uma ferramenta apenas para carregar os dados e outra para transformá-los, preferimos manter a mesma tecnologia nas etapas seguintes. Isso facilitou a leitura do arquivo, a definição dos tipos das colunas, a aplicação das regras de qualidade e a criação das camadas Bronze, Silver e Gold. Após a ingestão, realizei uma contagem inicial para confirmar se todos os registros haviam sido carregados corretamente. O resultado foi de 30.000 transações, exatamente a quantidade esperada para a base fornecida. Nesse primeiro momento, não foi identificado problema relacionado à perda de registros ou falha na leitura do arquivo. O principal cuidado foi garantir que o CSV fosse interpretado corretamente antes de iniciar qualquer transformação.

## Criação da camada Raw e definição do schema 

Depois da ingestão, mantivemos os dados em uma camada Raw, representando a informação mais próxima possível da fonte original. A intenção dessa camada é preservar o dado recebido e criar um ponto de referência para eventuais conferências ou reprocessamentos. Uma decisão importante nessa etapa foi definir explicitamente o schema da base, em vez de depender apenas da inferência automática do Spark. Campos como transaction_id e customer_id foram definidos como inteiros; amount e risk_score como numéricos; timestamp como data e hora; e is_fraud como booleano. Essa definição foi importante porque as etapas posteriores dependem diretamente da correta interpretação desses campos. Por exemplo, o timestamp precisa estar corretamente tipado para que seja possível extrair ano, mês e hora da transação. Da mesma forma, o campo is_fraud precisa ser interpretado corretamente para calcular quantidade e taxa de fraude nas agregações. Também aproveiteitamos essa etapa para verificar a estrutura geral da base. Não foram encontradas duplicidades no transaction_id, nem valores nulos nas colunas analisadas. Isso mostrou que o conjunto de dados já apresentava uma boa qualidade inicial, mas ainda assim considerei importante manter as regras de validação nas etapas seguintes.

## Aplicação da estratégia de particionamento

A estratégia de particionamento definida na Etapa 1 foi aplicada utilizando as colunas ano e mês, derivadas do campo timestamp. A escolha dessa granularidade foi feita pensando em consultas temporais, que são comuns em análises de transações e fraude como vistso em sala. Em um cenário real, a área de risco pode precisar analisar um mês específico, comparar períodos ou investigar um comportamento ocorrido em determinada janela de tempo. Por exemplo, caso seja necessário consultar apenas as transações fraudulentas ocorridas em outubro de 2025, a estrutura particionada permite acessar diretamente os dados daquele período, em vez de percorrer toda a base. Também consideramos outras possibilidades, como particionar por cliente, transação, dia ou hora, mas descartamos essas opções porque poderiam gerar muitas partições pequenas. Isso aumentaria a complexidade do armazenamento e poderia prejudicar o desempenho. Como a base atual contém somente registros de 2025, a coluna mês é a divisão mais relevante neste momento. Ainda assim, mantive ano e mês para deixar a estrutura preparada para uma futura inclusão de dados de outros anos.

## Camada Bronze — limpeza, validação e padronização 

Na camada Bronze, o objetivo foi garantir que os dados estivessem consistentes antes de qualquer enriquecimento ou análise. Foram aplicadas regras para identificar duplicidades de transação, valores ausentes, identificadores fora do intervalo esperado, valores financeiros inválidos, scores fora dos limites e problemas no campo de data. Também foram realizadas padronizações nos campos textuais, como remoção de espaços extras e conversão dos textos para um mesmo padrão de escrita. 

As principais regras utilizadas foram: 

transaction_id maior que zero; 
customer_id dentro do intervalo esperado; 
amount maior que zero; 
risk_score entre 0 e 100; 
credit_score entre 300 e 900; 
timestamp preenchido; 
remoção de duplicidades com base no transaction_id. 

Após aplicar todas essas regras, a camada Bronze permaneceu com 30.000 registros. Esse resultado foi interessante porque, inicialmente, esperavamos que alguma linha pudesse ser descartada durante a limpeza. No entanto, todos os registros atenderam aos critérios definidos. Por isso, considero importante destacar que a Bronze não foi apenas uma cópia da Raw. As validações realmente foram executadas, mas nenhuma inconsistência crítica foi encontrada. O resultado dessa etapa foi, portanto, a confirmação de que os dados estavam adequados para seguir para a camada Silver.

## Camada Silver — enriquecimento dos dados 

Depois de validarmos os registros, passamos para a camada Silver, cujo objetivo foi enriquecer os dados e criar novas variáveis que facilitassem as análises. Criamos uma variável chamada faixa_valor, que agrupou as transações de acordo com o valor financeiro. Isso permite avaliar, por exemplo, se faixas de valores maiores apresentam comportamento diferente em relação à fraude. Também extraímos a hora da transação a partir do timestamp e consideramos criar uma variável chamada periodo_dia, classificando as transações em madrugada, manhã, tarde e noite. Outra variável criada foi faixa_risco, derivada do risk_score, classificando as transações em baixo, médio e alto risco. Essas novas colunas foram criadas pensando diretamente nas análises que seriam realizadas posteriormente. 

Uma decisão importante foi manter na Silver as variáveis channel e merchant_category. consideramos importante manter channel porque o canal utilizado pode influenciar o comportamento da fraude. Uma transação realizada pelo aplicativo pode apresentar características diferentes de uma transação realizada via POS ou ATM. Da mesma forma, mantive merchant_category porque o tipo de estabelecimento também pode apresentar padrões distintos. Essa decisão acabou se mostrando relevante quando os dados foram agregados na camada Gold, pois foram encontradas diferenças consideráveis tanto entre canais quanto entre categorias. A Silver também ficou com 30.000 registros, pois essa etapa teve como objetivo enriquecer os dados, e não realizar uma nova exclusão de linhas.

## Camada Gold — criação das agregações analíticas

Na camada Gold, o objetivo foi transformar os dados detalhados da Silver em informações mais próximas da necessidade de análise. Para isso, criamos duas tabelas agregadas principais. A primeira agregação foi feita por canal de transação. Nessa tabela, calculamos o total de transações, quantidade de fraudes, valor total movimentado, risco médio e taxa de fraude para cada canal. 

Essa agregação permitiu observar diferenças relevantes entre os canais. O app apresentou 12.021 transações e 455 fraudes, resultando em uma taxa de fraude de aproximadamente 3,79%. Os demais canais apresentaram taxas menores: 

ATM: aproximadamente 1,80%; 
Web: aproximadamente 1,74%; 
POS: aproximadamente 1,42%. 

Esse resultado indica que o aplicativo merece maior atenção nas análises posteriores, já que apresentou uma taxa de fraude proporcionalmente superior aos demais canais. A segunda agregação foi feita por categoria de estabelecimento. Nessa tabela, foram calculados o total de transações, quantidade de fraudes, risco médio, ticket médio e taxa de fraude por categoria. A categoria viagem apresentou o resultado mais elevado, com 3.010 transações, 160 fraudes e uma taxa de aproximadamente 5,32%. As demais categorias apresentaram taxas mais próximas de 2%. Esse resultado mostrou uma diferença importante: varejo possui um volume maior de fraudes em números absolutos, mas viagem apresenta uma taxa proporcionalmente superior. Por esse motivo, consideramos importante trabalhar tanto com valores absolutos quanto com taxas. As duas agregações foram criadas pensando diretamente na Etapa 3, em que esses dados serão utilizados para gráficos, indicadores e análises mais detalhadas.

## Síntese para o questionário 

1. Quantas linhas sobreviveram da Bronze em diante? 
Alguma foi descartada? 

As 30.000 transações permaneceram na camada Bronze e seguiram para a Silver. Nenhuma linha foi descartada, pois todos os registros atenderam às regras de qualidade aplicadas. Foram verificadas duplicidades, valores nulos, identificadores inválidos, valores financeiros menores ou iguais a zero, scores fora das faixas esperadas e problemas de data. O fato de nenhuma linha ter sido removida não significa que a limpeza não tenha sido realizada. Pelo contrário, as regras foram executadas e serviram para confirmar que a base estava consistente.

2. A Silver utiliza channel e merchant_category? 

Sim. Decidimos manter as duas variáveis porque elas são relevantes para entender o contexto da transação. A variável channel permite comparar o comportamento da fraude entre app, web, POS e ATM. A variável merchant_category permite comparar categorias como varejo, viagem, alimentação, saúde, serviços e eletrônico. Essa decisão foi importante porque as análises posteriores mostraram diferenças relevantes entre esses grupos, principalmente no app e na categoria viagem.

3. Quais agregações foram colocadas na Gold e por quê? 

Foram criadas duas agregações principais. A primeira foi por canal, com total de transações, fraudes, valor total, risco médio e taxa de fraude. A segunda foi por categoria de estabelecimento, com total de transações, fraudes, risco médio, ticket médio e taxa de fraude. Escolhemos essas duas agregações porque elas permitem responder perguntas importantes para a análise de fraude em quais canais a taxa de fraude é maior e em quais categorias de estabelecimento apresentam maior risco proporcional? 

Ao final da Etapa 2, conseguimos executar todo o fluxo proposto: Ingestão → Raw → Bronze → Silver → Gold. 

A base começou com 30.000 registros e manteve os mesmos 30.000 após as validações da Bronze e o enriquecimento da Silver. A principal conclusão dessa etapa foi que a base fornecida possui boa qualidade inicial, mas ainda assim foi necessário aplicar regras de validação para garantir a consistência dos dados. Além disso, as primeiras agregações já mostraram alguns padrões que merecem investigação na Etapa 3, principalmente a maior taxa de fraude observada no app, com 3,79%, e na categoria viagem, com 5,32%. Esse resultado mostra que o pipeline não serviu apenas para organizar os dados, mas também começou a revelar informações relevantes para a análise de risco e fraude.
