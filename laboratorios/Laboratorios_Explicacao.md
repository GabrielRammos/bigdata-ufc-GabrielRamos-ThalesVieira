# Laboratorios - Explicação da Execução

# Primeiro Grande Problema da Rota A - 1 laboratório
Em relação a criação da estrutura de pastas no estilo `raw/bronze/silver/gold`, inicialmente tentou-se usar a Rota A com cluster real. Os códigos foram executados e funcionou até o passo 6 que contou as linhas direto do HDFS. Porém, o grande problema foi a replicação dos arquivos, o resumo não indicou a replicação dos 3 arquivos a mensagem que saiu foi:  

```bash
# "Average block replication: 1" e "Minimally replicated blocks:1"
```

Então depois de pesquisa e consulta ao Geminin, obteve-se um código para forçar a replicação dos blocos:" hdfs dfs -setrep -R 3". O código funcionou parcialmente pois houve a replicação em 3 blocos, mas ao consultar novamente o resumo apontou:

```bash
# Total blocks (validated):      1 (avg. block size 8139648 B)
# Minimally replicated blocks:   1 (100.0 %)
# Under-replicated blocks:       1 (100.0 %)
# Average block replication:     1.0
# Missing replicas:              2 (66.666664 %)
```

Ou seja o sistema não estava identificando os arquivos replicados, eles foram tratados como "missing replicas". Continuei pesquisando e descobri que o problema o "Live Datanodes (1)", era necessário configurar para simular 3 DataNodes numa mesma máquina e assim conseguir comportamento de replicação (fator 3). Na ocasião decidiu-se seguir para o segundo laboratório e depois ver essa questão.

# Segundo Grande Problema da Rota A - 2 laboratório
No segundo laboratório os códigos foram executados até o ponto 3.5, o grande problema foi MySQL no `ERROR 1698 (28000): Access denied for user 'root'@'localhost', nenhum código do laboratório funcionou. Tentou-se usar os seguintes códigos:

```bash
# ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'SUASENHA';
# FLUSH PRIVILEGES;

# ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'SUASENHA';
# FLUSH PRIVILEGES
```

Os códigos davam erro, quando funcionava o MySQL apresentava a seguinte mensagem de erro: 

```bash
# ERROR 1819 (HY000): Your password does not satisfy the current policy requirements #
```
Se tentou mudar a senha do subsistem linux, desinstala e reinstalar o MYSQL, mas nada funcionava.

# Motivo do uso da Rota B - Python
 Dado que já tinha dado problema no 1 laboratório com o Live DataNodes do Hadoop e esse problema do MySQl não se resolvia, então optou-se por seguir pela Rota B - sem Admin e usando o Python para simular o processo. Todos os códigos foram executados no prompt de comando do windows e executando WSL2 (Windows Subsystem for Linux) até por motivos de teste do uso da linguagem linux. Os códigos resultantes foram copiados para arquivos de texto txt. As tentativas do uso da Rota A até o segundo laboratório foram mantidas no trabalho.
 Uma pequena observação, a execução desses laboratórios foi realizada num computador de terceiros (parente familiar) por possuir maior capacidade de processamento e ser possível rodar os programas de Big Data e por isso o "Claudlx". Mesmo assim, o ambiente de Big Data se mostrou mais complexo do que se imaginava.
