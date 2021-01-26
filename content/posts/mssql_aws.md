---
title: "AWS Elastic Beanstalk MSSQL"
date: 2020-10-15T18:50:00-03:00
draft: false
toc: false
images:
tags:
  - aws
  - mssql
  - php 
---

Novamente meu *job* foi aceito no portal *Freelancer*. Desta vez seria algo que não tenho muito conhecimento.
Tenho conhecimento com *AWS* principalmente com *S3* e *EC2*, mas nunca fiquei cara a cara com a orquestração *Elastic Beanstalk*.

Fui sincero na minha proposta, informando que minha área de maior conhecimento é Linux, mas tenho experiência com *Docker* e *Kubernetes*.

O empregador rebateu perguntando se eu já trabalhei com *EB*, informei que não, mas como é um container baseado em *CentOS*, eu conseguiria me virar.
Fiz uma proposta informando que eu iria escrever as configurações, e após o OK, ele aceitaria minha proposta.

Uma das melhores formas de aprender, é na marra!

## Poucas documentações
Me deparei com poucas documentações informando como instalar os drivers *MSSQL* no *PHP* para Linux.
A maioria já estava bem defasada. O que eu fiz? Instalar um *CentOS* virtualizado e ir procurando referências com o *pecl*.

### Instalando no CentOS virtualizado
Basicamente precisava instalar o *PHP*, *Pear*, os compiladores GCC e o *devel* do driver ODBC.
```shell
yum update -y
# Habilitando PHP7, pois o CentOS 7 vem com PHP5
yum install -y epel-release yum-utils
yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum-config-manager --enable remi-php74
yum install -y php php-common php-opcache php-mcrypt php-cli php-gd \
    php-curl php-mysqlnd php-pear php-devel gcc gcc-c++ unixODBC-devel
# Verifique a versão do PHP
php -v
```

E agora compilar o módulo e ativar as bibliotecas no *conf*
```shell
pecl install sqlsrv
pecl install pdo_sqlsrv
# Ativando as bibliotecas
echo extension=sqlsrv.so > /etc/php.d/sqlsrv.ini
echo extension=pdo_sqlsrv.so > /etc/php.d/pdo_sqlsrv.ini
```

Como eu não queria instalar o servidor *web*, vou rodar o *phpinfo()* pelo shell e abrir o html
```shell
echo "<?php
phpinfo();" > phpinfo.php
php phpinfo.php > resultado.txt
```

E no arquivo, fui procurar a parte *pdo_sqlsrv* para verificar se o módulo está ativo, e estava :)
```
pdo_sqlsrv

pdo_sqlsrv support => enabled
ExtensionVer => 5.8.1

Directive => Local Value => Master Value
pdo_sqlsrv.client_buffer_max_kb_size => 10240 => 10240
pdo_sqlsrv.log_severity => 0 => 0
pdo_sqlsrv.set_locale_info => 2 => 2
```

Agora é ler a documentação da AWS e verificar como faço o *deploy* do contêiner

## Começando a apanhar
Pela documentação eu só precisaria me preocupar com a pasta *.ebextensions*, com um arquivo *config* e com os blocos de comandos em *YAML*.

Todos os arquivos de configurações para o *deploy* devem ficar na pasta *.ebextensions*, então criei o arquivo **mssql.config**.
Percebi uma enorme diferença entre os comandos **commands** e **container_commands**. Sem me aprofundar, a chave **commands** é executada antes da inicialização do contêiner, e a chave **container_commands** é executada após a inicialização do contêiner.

Então comecei a montar o *YAML* ``.ebextensions/mssql.config``:
```yaml
commands:
  # Instalação do Pear, compiladores e os headers    
  00_yum:
    command: yum install -y php-pear gcc gcc-c++ unixODBC-devel
  # Compilando o driver MSSQL  
  01_sqlsrv:
    command: pecl install sqlsrv      
  # Compilando o PDO
  02_pecl_pdo:
    command: pecl install pdo_sqlsrv
  # Configurando os módulos nos confs
  03_files:
    command: |
      echo extension=sqlsrv.so > /etc/php.d/sqlsrv.ini
      echo extension=pdo_sqlsrv.so > /etc/php.d/pdo_sqlsrv.ini
```

Crie o arquivo ``index.php``:
```php
<?php
phpinfo();
```

Para enviar os arquivos para o *EB* é só compactar todos os arquivos como *ZIP* e fazer o *upload*

## Criando a instância
Para criar a instância faça 
- Vá em https://console.aws.amazon.com/elasticbeanstalk;
- Clique em **Create a new environment**;
- Deixe marcado **Web server environment** => **Select**;
- Em **Aplication name** informe o nome da aplicação;
- **Environment name** deixe como preencheu;
- **Platform** => **PHP** e deixe o restante como padrão;
- Em **Application code** => **Upload your code**;
- **Local file** => **Choose file** e envie o *.ZIP*;
- Clique em **Create environment** e aguarde o deploy.

Se aparecer um OK, funcionou..... só que não!

Clique em **Go to environment** e observe o *sqlsvr*.

## Primeiro problema
Me deparei com esse problema quando eu queria atualizar somente o arquivo *PHP*. Se pegar o mesmo arquivo *.ZIP* e enviar é apresentado erros no deploy, mas por quê?

Para enviar uma nova versão é ir em **Application versions** => **Upload** => selecionar a nova versão => **Actions** => Deploy.

Porque os comandos *pecl* quando executado novamente retornam erro, e o deploy finalizado com a saída de erro. A solução seria verificar se existem os arquivos *.SO* e não executar novamente
```yaml
commands:
  # Instalação do Pear, compiladores e os headers    
  00_yum:
    command: yum install -y php-pear gcc gcc-c++ unixODBC-devel
  # Compilando o driver MSSQL    
  01_sqlsrv:
    command: |
      if [ ! -f "/usr/lib64/php/modules/sqlsrv.so" ]; then
        pecl install sqlsrv
      fi
  # Compilando o PDO    
  02_pecl_pdo:
    command: |
      if [ ! -f "/usr/lib64/php/modules/pdo_sqlsrv.so" ]; then
        pecl install pdo_sqlsrv
      fi
  # Configurando os módulos nos confs  
  03_files:
    command: |
      echo extension=sqlsrv.so > /etc/php.d/sqlsrv.ini
      echo extension=pdo_sqlsrv.so > /etc/php.d/pdo_sqlsrv.ini
```

## Segundo problema
Agora seria testar a conexão, alterei o arquivo ``index.php`` para uma conexão com a base de dados e um retorno simples. O banco foi fornecido pelo próprio empregador
```php
<?php
$pdo = new PDO( "sqlsrv:server=example.com;database=db", 
                "username", "password");

$query = $pdo->query("SELECT * FROM contacts");

$assoc = $query->fetch(PDO::FETCH_ASSOC);

print_r($assoc);
```

Fiz novamente o *upload*, e o que acontece?
```
"SQLSTATE[IMSSP]: This extension requires the Microsoft ODBC Driver for 
SQL Server to communicate with SQL Server. Access the following URL to 
download the ODBC Driver for SQL Server for x64: 
https://go.microsoft.com/fwlink/?LinkId=163712"
```
Pela documentação da Microsoft, é necessário um *driver ODBC* para comunicar, então é ler, e adaptar ao *YAML*
```yaml
commands:
  # Instalação do Pear, compiladores e os headers    
  00_yum:
    command: yum install -y php-pear gcc gcc-c++ unixODBC-devel
  # Compilando o driver MSSQL    
  01_sqlsrv:
    command: |
      if [ ! -f "/usr/lib64/php/modules/sqlsrv.so" ]; then
        pecl install sqlsrv
      fi
  # Compilando o PDO    
  02_pecl_pdo:
    command: |
      if [ ! -f "/usr/lib64/php/modules/pdo_sqlsrv.so" ]; then
        pecl install pdo_sqlsrv
      fi
  # Configurando os módulos nos confs  
  03_files:
    command: |
      echo extension=sqlsrv.so > /etc/php.d/sqlsrv.ini
      echo extension=pdo_sqlsrv.so > /etc/php.d/pdo_sqlsrv.ini
  # Driver ODBC
  05_odbc:
    command: |
      curl https://packages.microsoft.com/config/rhel/7/prod.repo > \
        /etc/yum.repos.d/mssql-release.repo
      ACCEPT_EULA=Y yum install -y msodbcsql17
      ACCEPT_EULA=Y yum install -y mssql-tools
      export PATH="$PATH:/opt/mssql-tools/bin"
```

E após enviar tudo novamente, obtive a minha consulta, e finalizei o projeto!

### REFERÊNCIAS
**Documentação AWS Elastic Beanstalk**
https://docs.aws.amazon.com/pt_br/elasticbeanstalk/latest/dg/ebextensions.html
**Documentação Microsoft**
https://docs.microsoft.com/en-us/sql/connect/php/system-requirements-for-the-php-sql-driver?redirectedfrom=MSDN&view=sql-server-ver15
**Entendendo o AWS Elastic Beanstalk**
https://guilhermeteles.com.br/entendendo-o-aws-elastic-beanstalk/