---
title: "Documentación de Ubuntu-Mikrotik IPsec - Español"
date: 2021-02-02T07:49:18-03:00
draft: true
toc: false
images:
tags:
  - mikrotik
  - ubuntu
  - ipsec
  - vpn
---

### Configuração do Servidor
#### Guión
| IP Servidor | Subnet GCP | Subnet LAN |
| :---: | :---: | :---: |
| 35.184.47.124 | 10.128.0.0/24 | 192.168.0.0/24 |

En el servidor de Ubuntu, realice la instalación de StrongSwan:
```shell
apt-get install strongswan
```

Edita el archivo ``/etc/ipsec.config``
```shell
version 2

config setup
        charondebug="ike 2, knl 2"

conn Mikrotik-01
        left=%any
        # Informar a la subnet de los servidores
        leftsubnet=10.128.0.0/24
        right=%any
        # Informar a la subnet de lo Mikrotik
        # Debe ser único para cada Mikrotik
        rightsubnet=192.168.0.0/24
        forceencaps=yes
        ike=sha512-aes256-modp4096
        esp=sha512-aes256-modp4096
        authby=secret
        auto=start
```

Edita el archivo ``/etc/ipsec.secrets``
```shell
%any %any : PSK 'Contrasena_larga_segura'
```

### Configuración de Mikrotik
- IP / IPsec / Profiles
![Profiles](/ipsec/profiles.png)

- IP / IPsec / Peers  
Atención al campo **Address**, debe informar la IP externa del servidor.  
![Peers](/ipsec/peer.png)

- IP / IPsec / Identifies  
Atención al campo **Secret**, porque debe contener la misma contraseña informada en el archivo ``/etc/ipsec.secrets``.
![Identifies](/ipsec/identifies.png)

- IP / IPsec / Proposal
![Proposal](/ipsec/proposal.png)

- IP / IPsec / Policy  
Atención al campos **Src. Address** e **Dst. Address**, porque debe informar a la subred del servidor y la subred de Mikrotik, respectivamente.
![Policy-01](/ipsec/policy-01.png)
![Policy-01](/ipsec/policy-02.png)

- Y ahora compruebe si la conexión se ha estabilizado.  
IP / IPsec / Active Peers
![Active Peers](/ipsec/active.png)

  

### Añadiendo el segundo Mikrotik
#### Guión
| IP Servidor | Subnet GCP | Subnet LAN 2 |
| :---: | :---: | :---: |
| 35.184.47.124 | 10.128.0.0/24 | 192.168.88.0/24 |

En el servidor, agregue las líneas siguientes al archivo ``/etc/ipsec.config``
```shell
conn Mikrotik-02
        left=%any
        leftsubnet=10.128.0.0/24
        right=%any
        # Informar a la subnet de lo Mikrotik
        # Debe ser único para cada Mikrotik
        rightsubnet=192.168.88.0/24
        forceencaps=yes
        ike=sha512-aes256-modp4096
        esp=sha512-aes256-modp4096
        authby=secret
        auto=start
```

Y en Mikrotik sigue la misma configuración que la anterior, solo presta atención a la configuración de la Política, que debería cambiar el campo **Dst. Address**:
![Policy-Mikrotik-2](/ipsec/policy-mikrotik-2.png)

##### Y FÍN