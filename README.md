Introducció
===========

Playbook que configura una màquina. Fa el següent:

* Apt-get update
* Instal·la usuaris
* Hostname, NTP i hosts


# Rols i variables a definir

## Rol ddclient

Instal·la un ddclient per sincronitzar hostnames amb IPs dinàmiques.

Variables:
* `ddclient_server` : servidor dyndns. Exemple: qui.guifi.net
* `ddclient_username` : Secret. username ddclient
* `ddclient_password` : Secret. password ddclient
* `ddclient_name` : hostname donat d'alta al ddclient

## Rol kvm host

Instal·la el KVM i modifica el fitxer de la xarxa (compte)

Variables:
* `kvmhost_ipaddress` : adreça ip a assignar al host. Ex: 10.0.2.3
* `kvmhost_netmask` : netmask a assignar al host. Ex: 255.255.255.0
* `kvmhost_gateway` : gateway a assignar. Ex: 10.0.2.1
* `kvmhost_dnsservers` : llista servidors DNS. Ex: 1.1.1.1 8.8.8.8
* `kvmhost_user` : usuari a crear per gestionar el kvm
* `kvmhost_main_iface` : interfície principal. Ex: eno1

## Rol server

Instal·la un servidor.

Variables:
* `server_hostname` : hostname
* `server_hostnamefqdn` : hostname fqdn. Ex. mariamayol2.guifi.gotes.org
* `server_ntpserver` : servidor ntp. Ex. localhost.
* `server_sshconfig_contents` : contingut de sshconfig de l'usuari root
* `server_users`: llista d'estructures d'usuaris que es crearan al sistema. S'ha de definir:
 * `name`: nom de l'usuari
 * `sudo`: True o False si està al grup sudo
 * `key`: clau pública de l'usuari
* `server_secret_users_pass`: Secret. llista d'estructures d'usuaris amb el digest de la seva password. Es copia a l'/etc/shadow. S'ha de definir:
 * `name`: nom de l'usuari
 * `pass`: digest de l'usuari

Exemple de server_users:

```
users:
-
   name: usu1
   sudo: True
   key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDV/TsZEBg4QVUOQPGQ5xO7XvMW8tcBpi9Ksa7qoAxXVvZG2XyzktzcObTqItEMWSBCoF3ikQYhj5YI7sA3IT5YP+knk+xLtIy2UP4Qo4vCfx82sPP/Onva4hvdfe7gbWr2TQM9Iy8W6G1zTMpB45huxo0S9gTLTKDdh6zMjDq1isHxGWLlol3QnVTygBSufdMXaT0koajLPjDtEmK79Uwx3Tfq+TSCwdUCXR+cE4BL8rUK8jWrijOf+po0RxXEsLyLTxKmEpKgSe2xijcOq95SBO1f64voGPw6h07S7aqK/ZMjhDTKUKcpskhOriJ+jqnBTmJnHnPtMFct6Q0H+FUd usu1@myhost"

-
   name: usu2
   sudo: True
   key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDV/TsZEBg4QVUOQPGQ5xO7XvMW8tcBpi9Ksa7qoAxXVvZG2XyzktzcObTqItEMWSBCoF3ikQYhj5YI7sA3IT5YP+knk+xLtIy2UP4Qo4vCfx82sPP/Onva4hvdfe7gbWr2TQM9Iy8W6G1zTMpB45huxo0S9gTLTKDdh6zMjDq1isHxGWLlol3QnVTygBSufdMXaT0koajLPjDtEmK79Uwx3Tfq+TSCwdUCXR+cE4BL8rUK8jWrijOf+po0RxXEsLyLTxKmEpKgSe2xijcOq95SBO1f64voGPw6h07S7aqK/ZMjhDTKUKcpskhOriJ+jqnBTmJnHnPtMFct6Q0H+FUd usu2@myhost"
```

Exemple de server_secret_users_pass:

```
server_secret_users_pass:
- 
   name: usu1
   pass: "$6$......"
```

Fitxers opcionals:
* `roles/server/files/id_rsa` : clau privada servidor
* `roles/server/files/id_rsa.pub` : clau pública servidor

# Rol zabbix agent. 

Instal·la un agent de zabbix.

Variables:
* `zabbixagent_hostpsk` : preshared key de zabbix. Exemple: bfda6f927b1c17098566597db6372f69097b33f41f9355887feb9abc29cc5f45
* `zabbixagent_zabbix_server` : servidor zabbix. Exemple: zabbix.organization.org



# Operacions

## Desplegar server

1. Generar inventari
1. Generar clau SSH (opcional): `ssh-keygen -t rsa -f ./roles/server/files/id_rsa -P ""
1. Copiar clau publica SSH on calgui. Concretament, al gitlab (opcional)
1. ansible-playbook  --vault-id one@vault -vvvvv -i inventory.yaml glinux.yml

## Amb rol Zabbix agent

Si hem de desplegar amb zabbix agent i variables secretes, introduim la PSK manualment:

```
openssl rand -hex 32
ansible-playbook -K --vault-id one@prompt -e host_psk=<key> -i inventory -l myserver glinux.yml
```

o bé si només volem instal·lar zabbix-agent


```
openssl rand -hex 32
ansible-playbook -K -e host_psk=<key> -i inventory zabbix-agent.yml 
```
