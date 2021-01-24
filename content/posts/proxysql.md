---
title: "Configurando ProxySQL"
date: 2020-12-22T17:01:18-03:00
draft: false
toc: false
images:
tags:
  - proxysql
  - mysql
  - ha
---

Fui premiado em um job, aonde a ideia é criar um HA (alta disponibilidade) do MySQL. Estudei com HAProxy, mas no meio da pesquisa, surgiu o projeto ProxySQL. Procurei a documentação, falhas, críticas, e gostei da ideia.

Aqui deixo o meu relato e como foi a implementação.

Por eu ter assinado um contrato de sigilo, não posso falar nomes, IPs, hostnames e várias outras coisas. Será somente a parte documental.

O ambiente atual é um MySQL rodando em um Debian 9, virtualizado, que fica protegido por um Firewall (não sei especificar qual é), que responde para várias aplicações web (PHP, Node e Ruby).

O meu serviço será configurar um outro MySQL (Oracle Linux) no DC 2, replicando todas as informações do MySQL em produção (DC 1), e após isso, configurar o ProxySQL (distribuição de minha escolha, mas gostei muito de trabalhar com o Oracle Linux), em um outro servidor (DC 1) para realizar o balanceamento de carga.

Talvez futuramente, e eu espero que sim, será criar um HA do próprio ProxySQL fazendo um over fail no próprio Firewall (isso será com o pessoal interno, minha responsabilidade será somente a configuração do segundo ProxySQL, mas espero eu que me deixem participar).

Momento de arregaçar as mangas, pegar um café e bora lá!

## Desenhando o cenário

| Função | DC | IP |
| :---: | :---: | :---: |
| ProxySQL | 01 | 192.168.3.100 |
| MySQL-01 | 01 | 192.168.3.110 |
| MySQL-02 | 02 | 192.168.3.111 |

##### O MySQL-01 está em produção, já o 02 será instalado via Docker em um servidor já existente.

## Configurando os bancos e a replicação
### Configurando o MySQL-01 em Produção
- Realizando o *dump* do banco de dados
```shell
root@mysql-01:~#
# Escolhi a flag --databases para não pegar os bancos de 
# controle do MySQL
mysqldump -uroot -psenha_fornecida_pela_seguranca \
    --databases banco01 banco02 banco03 > mysqldump.sql
```

- Preparando a replicação, **atenção com o IP, dever ser do MySQL-02**
```shell
root@mysql-01:~#
vim /etc/mysql/mariadb.conf.d/50-server.cnf
[mysqld]
...
server-id=1
log-bin=/var/log/mysql/bin-log
auto-increment-increment = 1
auto-increment-offset = 2
...
systemctl restart mysql.service
```
```sql
mysql>
CREATE USER 'replica'@'192.168.3.111' 
    IDENTIFIED BY 'senha_aleatoria';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'192.168.3.111';
SHOW MASTER STATUS;
-- Anote o File e Position, se não retornar nada, deu ruim
```

### Configurando o MySQL-02 no Oracle Linux
- Instalando o MySQL no Oracle Linux
```shell
[root@mysql-02 ~]#
yum install -y dnf-utils zip unzip
yum config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce --nobest
systemctl enable docker.service
systemctl start docker.service
# Verificando se deu bonito
systemctl status docker.service
```

- Montando o container do MySQL, **atenção com a pasta mysql**, **importante as que as versões sejam iguais**
```shell
[root@mysql-02 ~]#
docker run \
    --name mysql-02 \
    --restart always \
    -v data:/var/lib/mysql \
    -v <CAMINHO_ABSOLUTO>/mysql:/etc/mysql/conf.d \
    -e MYSQL_ROOT_PASSWORD=senha_fornecida_pela_seguranca \
    -p 3306:3306 \
    -d mysql:5.5.30
docker exec -it mysql-02 mysql -uroot \
    -psenha_fornecida_pela_seguranca
```
```sql
mysql>
show variables like 'server%';
-- Se o Value de server_id for igual a 2, deu bom
```

- Restaurando o *dump*
```shell
[root@mysql-02 ~]#
scp root@192.168.3.110:/root/mysqldump.sql .
cat mysqldump.sql | docker exec -i mysql-02 mysql -uroot \
    -psenha_fornecida_pela_seguranca
```

- Criando o usuário de acesso ao sistema
```sql
mysql>
CREATE USER 'user_sistema'@'%' IDENTIFIED BY 'user_sistema';
GRANT ALL PRIVILEGES ON *.* TO 'user_sistema'@'%';
FLUSH PRIVILEGES;
```

- Preparando a replicação, **atenção com a senha do usuário replica, deve ser a mesma dos dois bancos**
```sql
mysql>
CREATE USER 'replica'@'192.168.3.110' 
    IDENTIFIED BY 'senha_aleatoria';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'192.168.3.110';
FLUSH PRIVILEGES;
    CHANGE MASTER TO MASTER_HOST='192.168.3.110',
    MASTER_USER='replica',
    MASTER_PASSWORD='senha_aleatoria',
    MASTER_LOG_FILE='<File_do_MASTER_STATUS>',
    MASTER_LOG_POS=<Position_do_MASTER_STATUS>;
START SLAVE;
SHOW SLAVE STATUS;
-- Verifica se tem algum erro, se não, continue a nadar
SHOW MASTER STATUS;
-- Anote o File e Position, se não retornar nada, deu ruim
```

### Finalizando a replicação no MySQL-01 em Produção
- Finalizando a replicação
```sql
root@mysql-01:~# 
mysql>
CHANGE MASTER TO MASTER_HOST='192.168.3.111',
    MASTER_USER='replica',
    MASTER_PASSWORD='senha_aleatoria',
    MASTER_LOG_FILE='<File_do_MASTER_STATUS>',
    MASTER_LOG_POS=<Position_do_MASTER_STATUS>;
START SLAVE;
SHOW SLAVE STATUS;
-- Verifica se tem algum erro, se não, continue a nadar
```

- Em teoria, está funcionando a replicação, bora testar
```sql
root@mysql-01:~# 
mysql>
CREATE DATABASE TESTE;
use TESTE;
CREATE TABLE testando (id INT NULL);
[root@mysql-02 ~]# 
mysql>
show databases;
-- Se tiver o DB TESTE, tá lindo!
use TESTE;
show tables;
-- Se tiver a tabela testando, funcionou perfeitamente a 
-- replicação do MySQL-01 para o MySQL-02
-- Agora testar a replicação inversa
DROP DATABASE TESTE;
root@mysql-01:~# 
mysql>
show databases;
-- Se não tiver a base de dados, está funcionando perfeitamente, 
-- agora configurar o ProxySQL
```

- Criando o usuário de monitoramento, **atenção ao IP, deve ser o mesmo do ProxySQL**
```sql
-- Como está replicado, pode executar em qualquer um dos nodes
mysql>
CREATE USER 'monitor'@'192.168.3.100' 
    IDENTIFIED BY 'senha_aleatoria';
GRANT SELECT ON *.* TO 'monitor'@'192.168.3.100';
FLUSH PRIVILEGES;
```

## ProxySQL
- Instalando o ProxySQL
```shell
[root@proxy-sql ~]#
echo '[ProxySQL_Repo]
name=ProxySQL YUM repository
baseurl=https://repo.proxysql.com/ProxySQL/proxysql-2.0.x/centos/8/
gpgcheck=1
gpgkey=https://repo.proxysql.com/ProxySQL/repo_pub_key' > \
    /etc/yum.repos.d/proxysql.repo
yum install -y proxysql mysql
systemctl enable proxysql.service
systemctl start proxysql.service
mysql -uadmin -padmin -h 127.0.0.1 -P6032
# Se logou, o serviço está rodando
```

- Trocando a senha do administrador
```sql
mysql>
UPDATE global_variables SET 
    variable_value='admin:senha_fornecida_pela_seguranca'
    WHERE variable_name='admin-admin_credentials';
LOAD ADMIN VARIABLES TO RUNTIME;
SAVE ADMIN VARIABLES TO DISK;
```

- Alterando algumas configurações
```sql
mysql>
UPDATE global_variables SET variable_value=5 WHERE 
    variable_name='admin-stats_mysql_connection_pool';
UPDATE global_variables SET variable_value=5 WHERE 
    variable_name='admin-stats_mysql_connections';
UPDATE global_variables SET variable_value=5 WHERE 
    variable_name='admin-stats_mysql_query_cache';
UPDATE global_variables SET variable_value=10 WHERE 
    variable_name='mysql-ping_timeout_server';
UPDATE global_variables SET variable_value=10 WHERE 
    variable_name='mysql-connect_timeout_server';
UPDATE global_variables SET variable_value=10 WHERE 
    variable_name='mysql-poll_timeout';
-- Carregando as configurações
LOAD MYSQL VARIABLES TO RUNTIME;
LOAD MYSQL SERVERS TO RUNTIME;
-- Persistindo no banco de dados
SAVE MYSQL VARIABLES TO DISK;
SAVE MYSQL SERVERS TO DISK;
UPDATE global_variables SET variable_value='0.0.0.0:3306' 
    WHERE variable_name='mysql-interfaces';
SAVE MYSQL VARIABLES TO DISK;
PROXYSQL RESTART
```

- Inserindo os *nodes* para monitoramento
```sql
[root@proxy-sql ~]#
mysql -uadmin -h 127.0.0.1 -P6032 -psenha_fornecida_pela_seguranca
mysql>
INSERT INTO mysql_servers(hostgroup_id,hostname,port) 
    VALUES (1,'192.168.3.110',3306),(1,'192.168.3.111',3306);
```

- Configurando o usuário do monitoramento, a senha deve ser a mesma informada na criação
```sql
mysql>
UPDATE global_variables SET variable_value='monitor' 
    WHERE variable_name='mysql-monitor_username';
UPDATE global_variables SET variable_value='senha_aleatoria' 
    WHERE variable_name='mysql-monitor_password';
-- Carregando as configurações
LOAD MYSQL VARIABLES TO RUNTIME;
LOAD MYSQL SERVERS TO RUNTIME;
-- Persistindo no banco de dados
SAVE MYSQL VARIABLES TO DISK;
SAVE MYSQL SERVERS TO DISK;
```
```shell
[root@proxy-sql ~]#
systemctl restart proxysql.service
```
```sql
mysql>
-- Verificando o status do monitoramento, se retornar NULL 
-- em ping_error, está funcionando
SELECT * FROM monitor.mysql_server_ping_log;
```

- Criando usuário do sistema, é prudente manter a mesma senha, para se tornar transparente
```sql
mysql>
INSERT INTO mysql_users (username,password,default_hostgroup) 
    VALUES ('user_ibri','user_ibri',1);
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
```

- Vamos aos testes
```shell
# Logue no ProxySQL, lembrando que a porta 6032 é para 
# administração, e a 6033 é de uso dos bancos
[root@proxy-sql ~]#
mysql -uuser_sistema -h 127.0.0.1 -P6033 -puser_sistema
```
```sql
mysql>
-- Faça uma Query simples
show databases;
use banco_de_dados;
SELECT * FROM tabela;
-- Agora logue no MySQL-01 e pare o banco de dados
root@mysql-01:~#
systemctl stop mysql.service
-- Repita a consulta anterior e observe a mágina
-- Agora inverta, suba o MySQL-01 e pare o MySQL-02
-- E repita a consulta
```

- Verificando o status dos servidores
```sql
mysql>
SELECT srv_host,status FROM stats.stats_mysql_connection_pool;
```

## FIM
Todos os arquivos estão no repositório do [Github](https://github.com/rafaelarcanjo/ProxySQL)