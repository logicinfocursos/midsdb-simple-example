# mysql - dicas

## acessar e executar comandos no container
para ver os containers existentes
```
$ docker-compose ps
```
o resultado é:
```
time="2025-11-28T09:29:37-03:00" level=warning msg="A:\\workspaces\\alwaysON\\dev\\learning\\mindsDB\\docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion"
NAME                IMAGE                    COMMAND                  SERVICE   CREATED          STATUS          PORTS
mindsdb_container   mindsdb/mindsdb:latest   "bash -c 'python -Im…"   mindsdb   36 minutes ago   Up 35 minutes   0.0.0.0:47334-47335->47334-47335/tcp, [::]:47334-47335->47334-47335/tcp
mindsdb_mysql       mysql:8.0                "docker-entrypoint.s…"   mysql     35 minutes ago   Up 35 minutes   0.0.0.0:3307->3306/tcp, [::]:3307->3306/tcp
```

nesse caso o service é **mysql**

para entrar no terminal:
```
$ docker-compose exec mysql bash
```
o resultado é:
```
time="2025-11-28T09:31:15-03:00" level=warning msg="A:\\workspaces\\alwaysON\\dev\\learning\\mindsDB\\docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion"
bash-5.1# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 14
Server version: 8.0.44 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

pronto, você está no terminal do mysql
```

## Preparar os dados de consulta (caso você ainda não tenha os dados)
Para criar dados de exemplo para uso no mindsdb, você pode usar o script "mindsdb.sql", que cria a tabela e insere 50registos. Para tanto:
1) Copie o arquivo para o container:
```
docker cp a:/workspaces/alwaysON/dev/learning/mindsDB/mindsdb.sql mindsdb_mysql:/mindsdb.sql
```
2) Acesse o terminal do container MySQL:
```
docker-compose exec mysql bash
```
3) Execute o script SQL:
```
mysql -u mindsdb -pmindsdb mindsdb < /mindsdb.sql
```


4) para listar os bancos de dados existentes:
```
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mindsdb            |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)
```


5) para ver as tabelas do banco de dados mindsdb:
```
mysql> use mindsdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_mindsdb |
+-------------------+
| vendas            |
+-------------------+
1 row in set (0.01 sec)
```

6) para ver a estrutura da tabela vendas
```
mysql> DESCRIBE vendas;
+----------------+---------------+------+-----+---------+-------+
| Field          | Type          | Null | Key | Default | Extra |
+----------------+---------------+------+-----+---------+-------+
| id             | int           | NO   | PRI | NULL    |       |
| data           | date          | YES  |     | NULL    |       |
| produto        | varchar(20)   | YES  |     | NULL    |       |
| quantidade     | int           | YES  |     | NULL    |       |
| preco_unitario | decimal(10,2) | YES  |     | NULL    |       |
| total_vendas   | decimal(10,2) | YES  |     | NULL    |       |
+----------------+---------------+------+-----+---------+-------+
6 rows in set (0.00 sec)
```

7) para listar os registros (os 10 primeiros)

```
mysql> select * from vendas limit 10;
+----+------------+-----------+------------+----------------+--------------+
| id | data       | produto   | quantidade | preco_unitario | total_vendas |
+----+------------+-----------+------------+----------------+--------------+
|  1 | 2025-01-01 | Produto C |         15 |          16.71 |       250.65 |
|  2 | 2025-01-02 | Produto D |         19 |          98.82 |      1877.58 |
|  3 | 2025-01-03 | Produto A |         12 |          79.50 |       954.00 |
|  4 | 2025-01-04 | Produto C |          3 |          27.88 |        83.64 |
|  5 | 2025-01-05 | Produto C |          5 |          10.50 |        52.50 |
|  6 | 2025-01-06 | Produto D |         19 |          83.39 |      1584.41 |
|  7 | 2025-01-07 | Produto A |          7 |          73.62 |       515.34 |
|  8 | 2025-01-08 | Produto A |          9 |          75.61 |       680.49 |
|  9 | 2025-01-09 | Produto C |          7 |          79.41 |       555.87 |
| 10 | 2025-01-10 | Produto B |         18 |          16.66 |       299.88 |
+----+------------+-----------+------------+----------------+--------------+
10 rows in set (0.00 sec)
```