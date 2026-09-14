**Etapa 3 — Análise e Dashboard**


## Visão geral dos dados

Antes de aprofundar as análises, calculamos alguns indicadores gerais da base para entender a dimensão do problema. A base possui 30.000 transações, sendo 751 classificadas como fraude, o que representa uma taxa geral de aproximadamente 2,50%. O valor financeiro associado às transações fraudulentas foi de aproximadamente R$ 119.367,93.

Esses indicadores foram utilizados no dashboard porque ajudam a dar uma referência para as análises seguintes. Por exemplo, quando encontro um grupo com taxa de fraude de 5%, conseguimos perceber que esse resultado é aproximadamente o dobro da média geral da base.

## Análise por segmento

A primeira análise foi feita utilizando a variável segment, comparando a quantidade de transações e a taxa de fraude entre Premium, Standard e High-Risk.

| Segmento  | Transações | Fraudes | Taxa de fraude |
| --------- | ---------: | ------: | -------------: |
| High-Risk |      2.915 |     282 |      **9,67%** |
| Standard  |     10.645 |     312 |      **2,93%** |
| Premium   |     16.440 |     157 |      **0,95%** |

O resultado que mais chamou atenção foi o segmento High-Risk. Apesar de representar uma parcela menor das transações, apresentou uma taxa de fraude de aproximadamente 9,67%, muito superior aos demais grupos.

Finding → Insight → Ação

Finding: o segmento High-Risk apresentou taxa de fraude de 9,67%, enquanto o Premium ficou abaixo de 1%.
Insight: a classificação de segmento parece separar grupos com comportamentos bastante diferentes em relação à fraude. O cliente classificado como High-Risk apresenta uma incidência proporcional muito maior.
Ação: a área de risco poderia utilizar essa informação para priorizar o monitoramento das transações desse segmento, combinando o segmento com outras variáveis antes de tomar decisões mais restritivas. 

## Risco por canal

Na segunda análise, comparei os diferentes canais utilizados nas transações: app, web, POS e ATM.
Os resultados foram: 

| Canal | Transações | Fraudes | Taxa de fraude |
| ----- | ---------: | ------: | -------------: |
| App   |     12.021 |     455 |      **3,79%** |
| ATM   |      3.007 |      54 |          1,80% |
| Web   |      9.004 |     157 |          1,74% |
| POS   |      5.968 |      85 |          1,42% |

O app apresentou tanto o maior número de fraudes quanto a maior taxa proporcional entre os canais.
É importante separar essas duas interpretações. Um canal com muitas fraudes pode simplesmente possuir maior volume de transações, mas neste caso o app também possui uma taxa de fraude superior aos demais.
Aproximadamente 3,79% das transações realizadas pelo app foram classificadas como fraude, enquanto nos outros canais a taxa ficou entre aproximadamente 1,4% e 1,8%.
Existe uma concentração maior de fraude nas transações realizadas pelo aplicativo. Isso pode estar relacionado às características das operações digitais ou ao próprio perfil das transações realizadas nesse canal.
Priorizariamos o app em análises mais detalhadas, verificando quais categorias, horários e segmentos concentram as fraudes dentro desse canal.
Um ponto adicional foi que o risk_score médio ficou próximo de 50 em todos os canais. Portanto, apenas comparar a média do score por canal não explica sozinho a diferença encontrada na taxa de fraude.

## Risco por categoria de estabelecimento 

| Categoria   | Transações | Fraudes | Taxa de fraude |
| ----------- | ---------: | ------: | -------------: |
| Viagem      |      3.010 |     160 |      **5,32%** |
| Saúde       |      2.951 |      73 |          2,47% |
| Alimentação |      5.976 |     138 |          2,31% |
| Varejo      |      9.047 |     203 |          2,24% |
| Serviços    |      4.576 |      90 |          1,97% |
| Eletrônico  |      4.440 |      87 |          1,96% |

Também analisamos a variável merchant_category para identificar se determinadas categorias apresentavam maior ocorrência de fraude.

A categoria viagem apresentou a maior taxa de fraude, com aproximadamente 5,32%.
Esse resultado é interessante porque varejo possui o maior número absoluto de fraudes, com 203 ocorrências, mas também possui mais de 9 mil transações. Quando analiso proporcionalmente, viagem se torna o principal destaque.
A categoria viagem apresentou taxa de fraude de 5,32%, mais que o dobro da taxa geral de aproximadamente 2,50%.
transações relacionadas a viagem apresentam um comportamento mais crítico proporcionalmente, mesmo não sendo a categoria com maior volume.
Utilizaria essa categoria como um dos critérios para aprofundar a análise, principalmente quando combinada com canal, segmento e horário.

## Análise temporal 

Para verificar se havia algum comportamento de fraude relacionado ao tempo, analisamos inicialmente a taxa mensal.Os resultados permaneceram relativamente estáveis ao longo do ano, variando aproximadamente entre 2,08% e 2,80%. O menor resultado ocorreu em setembro, com cerca de 2,08%, enquanto dezembro apresentou aproximadamente 2,80%.
Isso indica que não existe uma mudança mensal muito brusca no conjunto analisado. Portanto, apenas o mês não parece ser suficiente para identificar um padrão forte de fraude.
Quando passamos para uma análise mais detalhada por hora, encontrei uma diferença mais interessante.
A faixa de 1 hora da manhã apresentou taxa de fraude de aproximadamente 4,46%, a maior entre os horários analisados.
Outros horários da madrugada, como 3h e 4h, também apareceram entre os valores mais altos.
A interpretação é que o horário parece trazer uma informação mais útil do que o mês para esse conjunto de dados. Mesmo assim, eu evitaria afirmar que qualquer operação de madrugada é suspeita. O horário deve ser analisado junto com outras características da transação.

Após essa analise, nos questionamos em quais combinações de canal e categoria de estabelecimento a TechPay deveria concentrar primeiro sua investigação de fraude?

O principal resultado foi:
App + Viagem
1.237 transações
99 fraudes
taxa de fraude de aproximadamente 8,00%

Esse resultado chamou bastante atenção porque a combinação apresenta uma taxa mais de três vezes superior à média geral da base. As 99 fraudes desse grupo representam aproximadamente 13,2% de todas as fraudes encontradas, apesar de essas transações representarem pouco mais de 4% da base total.
As transações da categoria viagem realizadas pelo app apresentaram taxa de fraude próxima de 8%. Quando canal e categoria são analisados juntos, aparece um grupo mais crítico do que quando essas variáveis são vistas separadamente. Priorizriamos esse grupo para regras adicionais de autenticação ou revisão. Não aplicaria bloqueio automático apenas por pertencer à combinação app + viagem, mas trataria essas operações com atenção maior.

## Estrutura do Dash 

O dashboard foi desenvolvido para transformar essas análises em uma visualização mais fácil de acompanhar.
Ele possui os quatro tipos de bloco exigidos no enunciado:
KPI: total de transações e taxa geral de fraude; Tendência: evolução mensal da taxa de fraude; Composição: comparação das taxas de fraude por canal e por segmento; 
Detalhe: matriz que cruza canal e categoria de estabelecimento.
A ideia foi permitir que o usuário comece com uma visão geral e depois consiga identificar rapidamente quais grupos apresentam comportamento diferente.

## Descrição da ideia do BI

Para quem é o dashboard? Pensamos o painel principalmente para a equipe de risco e prevenção a fraudes da TechPay.
Uma liderança da área também poderia utilizá-lo, mas o principal usuário seria o analista responsável por acompanhar o comportamento das transações e identificar mudanças ou grupos que mereçam investigação.

Que decisão ele ajuda a tomar? O dashboard ajuda principalmente a decidir onde concentrar o esforço de investigação.Em uma base com milhares de transações, não seria possível analisar cada operação manualmente. O painel ajuda a mostrar quais canais, segmentos, categorias ou combinações estão apresentando taxas de fraude mais altas.Assim, ele não serviria diretamente para bloquear clientes, mas para orientar priorização, regras de monitoramento e investigações.

Escolhemos como principais indicadores:
total de transações;
quantidade e taxa de fraude;
valor associado às fraudes;
comportamento por canal;
comportamento por segmento;
categoria;
comportamento temporal.

A taxa de fraude foi um indicador especialmente importante porque permite comparar grupos de tamanhos diferentes.
Por exemplo, varejo tem mais fraudes em valores absolutos do que viagem, mas também possui muito mais transações. Apenas olhando o número de fraudes, eu poderia chegar a uma conclusão incompleta. Não colocamos o risco médio como um dos principais KPIs do dashboard porque as médias encontradas por canal e categoria ficaram muito próximas de 50. Nesse nível agregado, esse indicador não apresentou tanta capacidade de diferenciar os grupos quanto a taxa real de fraude.
Quem seria o dono do painel?
Eu colocaria como responsável pela rotina do dashboard a área de Risco/Fraude.
Essa equipe poderia acompanhar o painel semanalmente e avaliar mudanças nas taxas de fraude, novos grupos críticos e possíveis ajustes nas regras de monitoramento.
Caso fosse identificada uma mudança mais relevante, as informações poderiam então ser levadas para gestores ou outras áreas responsáveis pelas decisões de negócio.
