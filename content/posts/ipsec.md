---
title: "Documentação IPsec Ubuntu-Mikrotik"
date: 2021-02-02T07:49:18-03:00
draft: false
toc: false
images:
tags:
  - mikrotik
  - ubuntu
  - ipsec
  - vpn
---

### Configuração do Servidor
#### Cenário
| IP Servidor | Subnet GCP | Subnet Local |
| :---: | :---: | :---: |
| 35.184.47.124 | 10.128.0.0/24 | 192.168.0.0/24 |

No servidor Ubuntu, executar a instalação do StrongSwan:
```shell
apt-get install strongswan
```

Editar o arquivo ``/etc/ipsec.config``
```shell
version 2

config setup
        charondebug="ike 2, knl 2"

conn Mikrotik-01
        left=%any
        # Informar a subnet dos servidores
        leftsubnet=10.128.0.0/24
        right=%any
        # Informar a subnet do Mikrotik
        # Deve ser única para cada Mikrotik
        rightsubnet=192.168.0.0/24
        forceencaps=yes
        ike=sha512-aes256-modp4096
        esp=sha512-aes256-modp4096
        authby=secret
        auto=start
```

Editar o arquivo ``/etc/ipsec.secrets``
```shell
%any %any : PSK 'senha_segura_longa'
```

### Configuração do Mikrotik
- IP / IPsec / Profiles
![Profiles](/ipsec/profiles.png)

- IP / IPsec / Peers  
Atenção ao campo **Address**, deve informar o IP externo do servidor.  
![Peers](/ipsec/peer.png)

- IP / IPsec / Identifies  
Atenção ao campo **Secret**, pois deve conter a mesma senha informada no arquivo ``/etc/ipsec.secrets``.
![Identifies](/ipsec/identifies.png)

- IP / IPsec / Proposal
![Proposal](/ipsec/proposal.png)

- IP / IPsec / Policy  
Atenção aos campos **Src. Address** e **Dst. Address**, pois deve informar a subnet do servidor e a subnet do Mikrotik, respectivamente.
![Policy-01](/ipsec/policy-01.png)
![Policy-01](/ipsec/policy-02.png)

- E agora verificar se a conexão estabilizou.  
IP / IPsec / Active Peers
![Active Peers](/ipsec/active.png)

  

### Adicionando o segundo Mikrotik
#### Cenário
| IP Servidor | Subnet GCP | Subnet Local 2 |
| :---: | :---: | :---: |
| 35.184.47.124 | 10.128.0.0/24 | 192.168.88.0/24 |

No servidor, adicionar as linhas abaixo no arquivo ``/etc/ipsec.config``
```shell
conn Mikrotik-02
        left=%any
        leftsubnet=10.128.0.0/24
        right=%any
        # Informar a subnet do Mikrotik
        # Deve ser única para cada Mikrotik
        rightsubnet=192.168.88.0/24
        forceencaps=yes
        ike=sha512-aes256-modp4096
        esp=sha512-aes256-modp4096
        authby=secret
        auto=start
```

E no Mikrotik seguir as mesmas configurações do anterior, só atenção na configuração do Policy, que deve alterar o campo **Dst. Address**:
![Policy-Mikrotik-2](/ipsec/policy-mikrotik-2.png)

##### E FIM