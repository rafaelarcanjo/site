+++
title = "Rafael Arcanjo"
date = "2021-01-27"
aliases = ["sobre","curriculo","curriculum"]
[ author ]
  name = "Rafael Arcanjo"
+++
#### FORMAÇÃO
* **Especialização em Projetos de Redes e Computação em Nuvens - Cruzeiro do Sul**  
  01 DE 2020 - Em andamento 
* **Tecnólogo em Redes de Computadores - Estácio de Sá**  
  01 DE 2014 - 01 DE 2016
* **Tecnólogo em Análise e Desenvolvimento de Sistemas - Universidade Anhanguera**  
  06 DE 2009 - 06 DE 2016
* **Curso Técnico em Informática - Colégio Técnico de Minas Gerais (COTEMIG)**  
  02 DE 2004 - 02 DE 2007

---

#### EXPERIÊNCIA
* **Analista de Suporte @ Transport Serviços Internacionais LTDA**  
    Objetivo: Atividades de Backup, Administração de Servidores Windows (Windows 2008 e 2012), Linux (SUSE Enterprise e CentOS) e FreeBSD (FreeNAS e pfSense), Implementação de Solução de Gerência de Chamados (GLPI), Administração de Servidor de e-mail (Exchange 2007), Administração de AD (Windows 2012 e Samba 3), Administração de Firewall (pfSense e IPTables) e Administração de DNS com Bind9.  
    Projetos executados:  
    **=>** Migração de firewall descentralizado (Squid, SquidGuard, IPTables e OpenVPN) para pfSense;  
    **=>** Interligação de filial em Mário Campos com Matriz em BH, utilizando pfSense com IPsec;  
    **=>** Solução de backup com Bacula e FreeNAS com daemon Bacula-SD, substituindo robô de fita com trocas manuais;  
    **=>** Substituição do software pago (PaperCut) por CUPS para recebimento de impressões remotas do Siscomex;  **=>** Migração do Active Directory do Windows 2008 para Windows 2012.

* **Técnico de Suporte @ PRODABEL – Empresa de Informática e Informação do Município de Belo Horizonte.**  
  Objetivo: Responsável pelo suporte técnico presencial e remoto ao gabinete do Prefeito do Município de Belo Horizonte. Auxílio ao gabinete do Prefeito Municipal de Belo Horizonte nas tomadas de decisões na área de tecnologia.

* **Técnico de Redes @ Prefeitura Municipal de Santa Luzia - Minas Gerais.**  
  Objetivo: Atividades de Backup, Administração de Servidores Linux (CentOS e Ubuntu Server) e FreeBSD (FreeNAS e pfSense), Implementação de Solução de Gerência de Chamados (GLPI), Administração de Firewall (pfSense), Administração de Equipamentos de Rede (Switches Datacom e Mikrotik) e Implementação de serviços em Containers.
  Projetos executados:  
  **=>** Criação de portal Hotspot para atender os munícipes de Santa Luzia, usando pfSense, FreeRADIUS e PHP  
  **=>** Interligação de unidades com internet terceirizada, utilizando pfSense e Mikrotik com IPsec, distribuindo Hotspot com replicação no Captive Portal do pfSense;  
  **=>** Centralização de logs com Graylog;  
  **=>** Implantação de Backup com Bacula em Storage;  
  **=>** Migração de serviços físicos para Docker, reduzindo consumo de recursos.

* **Contrato pelo portal Freelancer.com @ Empresa sigilosa**  
  Cenário: A Empresa possui dois servidores PowerEdge R740XD, em localidades distintas, e solicitou configurar alta disponibilidade, utilizando os dois equipamentos.  
  Objetivo: Migrar e Replicar Serviços para Nuvem, Configurar Alta Disponibilidade.

  Projetos executados:  
  **=>** Realizado replicação e failover do MySQL utilizando Proxy SQL, entre os servidores físicos. Detalhes das configurações no Github https://github.com/rafaelarcanjo/ProxySQL;  
  **=>** Configurado NGINX com EC2 para prover alta disponibilidade dos serviços web;  
  **=>** Redundância de backup utilizando o Backup com S3;  
  **=>** Configurado DNS com Bind9.  
  Projeto em andamento:  
  **=>** Migração do e-mail hospedado na Locaweb para solução em Zimbra.  
  Projetos em estudo:  
  **=>** Migração do Microsoft Hyper-V para VMware ESXi ou OpenStack.

* **Contrato pelo portal Freelancer.com @ Laura Assis - Empresa Sigilosa**  
  Cenário: Migrar serviços físicos para nuvem (Digital Ocean) utilizando orquestração de Containers.  
  Objetivo: Migrar serviços web, bancos de dados e apresentar soluções de backup.

  Projetos executados:  
  **=>** Realizado migrações dos serviços web e bancos de dados (MySQL e PostgreSQL) para containers com Docker;  
  **=>** Orquestração dos containers com Docker Compose;  
  **=>** Configuração do NGINX para alta disponibilidade e Memcached para otimizar o desempenho;  
  **=>** Implementação do Bacula para backup diário das bases de dados em storage local e em SOS (Spaces Object Storage - Digital Ocean);  
  **=>** Orientação sobre o Backup Automático de Droplet.  

* **Contrato pelo Portal Freelancer.com @ Jefferson T - AzureWEB do Brasil**  
  Objetivo: Implementar um serviço de atualização automática de DNS.  

  Projetos executados:  
  **=>** Implementação de atualização automática de registro diretamente no Bind9;  
  **=>** Implementação de atualização automática pelo CPanel, utilizando Perl e Shell Script, fazendo chamadas na API do CPanel. Escolhida pelo cliente, detalhes no Github https://github.com/rafaelarcanjo/azurewebr

* **Contrato pelo portal 99Freelas @ Marcelo Maia - Empresa CopywriterLAB**
  Cenário: Site em WordPress sofreu ataque por Apache CVE-2017-5638.  
  Objetivo: Fornecer uma camada extra de segurança.

  Projetos executados:  
  **=>** Migração para NGINX, realizando Proxy Pass para o PHP-FPM diretamente, habilitando configuração de “split_path_info”;  
  **=>** Alteração das configurações de SSH, para prover uma maior segurança.
---

#### CERTIFICAÇÕES
* LPI Linux Essentials
---

#### COMPETÊNCIAS
* Linux (principais distribuições: Debian, openSUSE, CentOS e Oracle Linux);
* MikroTik (RouterOS);
* FreeBSD (pfSense e FreeNAS);
* Switches Datacom;
* Windows Server (intermediário);
* Soluções de e-mail (Postfix, Dovecot e Zimbra);
* Bacula;
* Virtualização (KVM, estudando OpenStack e Hyper-v) ;
* GLPI;
* Domínio (Active Directory e Samba);
* Firewall (pfSense, UFW e IPTables);
* Bind9;
* Docker e Docker Compose;
* Monitoramento (Graylog e Zabbix);
* Banco de Dados (MySQL, PostgreSQL, SQL Server e Hasura GraphQL);
* Serviços em nuvens, como AWS, Google Cloud, Azure e Digital Ocean (conhecimentos intermediários e em estudos);
* Serviços web (NGINX, Apache e HAProxy);
* Shell Script.

#### Contatos
**E-mail:** [rafael@libre.tec.br](mailto:rafael@libre.tec.br)  
**Github:** [https://github.com/rafaelarcanjo](https://github.com/rafaelarcanjo)