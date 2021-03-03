# rthomaz-iac-private-ns1

Servidor de DNS Primário de rede privada

[![CI](https://github.com/rthomaz-iac/rthomaz-iac-private-ns1/workflows/CI/badge.svg)](https://github.com/rthomaz-iac/rthomaz-iac-private-ns1/actions?query=workflow%3ACI)


## Começando

### Pré-requisitos

[Raspberry Pi 3 - Criando (sem-Domínio)](https://github.com/rthomaz-iac/rthomaz-iac-wiki/wiki/Raspberry-Pi-3-Criando-(sem-Dom%C3%ADnio))

### Instalação

`$ sudo apt-get install bind9 bind9utils bind9-doc`

### Configuração

`$ sudo cp /etc/default/bind9 /etc/default/bind9.original`

`$ sudo nano /etc/default/bind9`

`$ sudo cp /etc/bind/named.conf.options /etc/bind/named.conf.options.original`

`$ sudo nano /etc/bind/named.conf.options`

`$ sudo cp /etc/bind/named.conf.local /etc/bind/named.conf.local.original`

`$ sudo nano /etc/bind/named.conf.local`

`$ sudo mkdir /etc/bind/zones`

`$ sudo nano /etc/bind/zones/db.corp.rthomaz.com.br`

`$ sudo nano /etc/bind/zones/db.192.168`

## Usando

### Windows

### Linux

## Referências

- https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-18-04-pt
