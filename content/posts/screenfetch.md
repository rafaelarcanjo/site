---
title: "Instalação do ScreenFetch"
date: 2021-02-02T15:28:18-03:00
draft: false
toc: false
images:
tags:
  - screenfetch
  - ubuntu
  - opensuse
---

Finalmente fazendo a documentação do ScreenFetch, sempre que me deparo com a configuração, fico meio perdido.

Abaixo irei descrever como fiz no Ubuntu 20.04 e no openSUSE 15.2.

## Ubuntu 20.04
Baixando e instalando:
```shell
sudo wget -c https://raw.github.com/KittyKatt/screenFetch/master/screenfetch-dev -O /usr/bin/screenfetch
chmod +x /usr/bin/screenfetch
```

Pelo *motd* do Ubuntu ser meio poluído, na minha opinião, eu prefiro usar os arquivos que eu editei com o tempo, então vou fazer *backup* dos arquivos originais e configurar com os meus.
```shell
mkdir ~/bkp_motd
mv /etc/update-motd.d/* ~/bkp_motd
```

- Arquivo ``00-header``  
```shell
cat > /etc/update-motd.d/00-header
#!/bin/sh
[ -r /etc/lsb-release ] && . /etc/lsb-release
if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
    DISTRIB_DESCRIPTION=$(lsb_release -s -d)
fi
screenfetch
```

- Arquivo ``10-sysinfo``  
```shell
cat > /etc/update-motd.d/10-sysinfo
#!/bin/bash
date=`date`
load=`cat /proc/loadavg | awk '{print $1}'`
root_usage=`df -h / | awk '/\// {print $(NF-1)}'`
memory_usage=`free -m | grep Mem: | awk '{printf("%3.1f%%", $3/$2*100)}'`
swap_usage=`free -m | awk '/Swap/ { printf("%3.1f%%", $3/$2*100) }'`
users=`users | wc -w`
time=`uptime | grep -ohe 'up .*' | sed 's/,/\ hours/g' | awk '{ printf $2" "$3 }'`
processes=`ps aux | wc -l`
ip=$(curl -s http://whatismyip.akamai.com)
echo "Informação do sistema em: $date"
echo
printf "Carga:\t\t%s\tIP:\t\t%s\n" $load $ip
printf "Memória:\t%s\tUptime:\t\t%s\n" $memory_usage "$time"
printf "Uso em /:\t%s\tSwap:\t\t%s\n" $root_usage $swap_usage
printf "Usuários:\t%s\tProcessos:\t%s\n" $users $processes
echo
```

- Arquivo ``90-footer``  
```shell
cat > /etc/update-motd.d/90-footer
#!/bin/sh
[ -f /etc/motd.tail ] && cat /etc/motd.tail || true
```
- Alterando as permissões
```shell
chmod +x /etc/update-motd.d/*
```

E pronto, no Ubuntu ele já está pré configurado.

## openSUSE 15.2
Já no openSUSE, é necessário configurar algumas coisas e fazer uma gambiarra.

Os arquivos devem ser os mesmos do Ubuntu, porém a pasta deve ser criada.
```shell
mkdir /etc/update-motd.d/
```
Agora é só criar os arquivos ``00-header``, ``10-sysinfo`` e ``90-footer`` e alterar a permissão, conforme procedimento do Ubuntu.

- Instalando o ScreenFetch
```shell
sudo zypper in screenfetch
```

- Alterar o arquivo ``/etc/pam.d/sshd``
```shell
...
session optional pam_keyinit.so force revoke
session optional pam_motd.so motd=/run/motd.dynamic
```

- Adicionar a linha para gerar o *motd* no *cron*, não quero sair alterando vários arquivos do *SO*.
```shell
crontab -e
* * * * * find /etc/update-motd.d/ -type f -executable -exec {} \; > /run/motd.dynamic
```

E fim!

### REFERÊNCIAS
**Lançado ScreenFetch para Ubuntu** https://diolinux.com.br/noticias/como-instalar-o-screenfetch-no-ubuntu.html  
**Luca Critelli** https://github.com/lucacri/motd/blob/master/00-header