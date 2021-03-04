# rthomaz-iac-private-ns1

Servidor de DNS Primário de rede privada

[![CI](https://github.com/rthomaz-iac/rthomaz-iac-private-ns1/workflows/CI/badge.svg)](https://github.com/rthomaz-iac/rthomaz-iac-private-ns1/actions?query=workflow%3ACI)

|**hostname:**|ns1|
|---|---|
|**ip address:**|192.168.1.11|
|**subnet mask:**|255.255.255.0|
|**domain:**|corp.rthomaz.com.br|

## Começando

### Pré-requisitos

[Raspbian Instalação e Configuração (sem-Domínio)](https://github.com/rthomaz-iac/rthomaz-iac-wiki/wiki/Raspbian-Instala%C3%A7%C3%A3o-e-Configura%C3%A7%C3%A3o-(sem-Dom%C3%ADnio))

### Instalação

`$ sudo apt-get update`

 Instalando o BIND:
 
`$ sudo apt-get install bind9 bind9utils bind9-doc`

 Instalando utilitários de DNS:

`$ sudo apt-get install dnsutils`

### Configuração IPV4 para o bind

`$ sudo cp /etc/default/bind9 /etc/default/bind9.original`

`$ sudo nano /etc/default/bind9`

    #
    # run resolvconf?
    RESOLVCONF=no
    
    # startup options for the server
    OPTIONS="-u bind -4"

Agora reinicie o bind para aplicar as configurações.

`$ sudo systemctl restart bind9`

### Configurando as opções do bind

`$ sudo cp /etc/bind/named.conf.options /etc/bind/named.conf.options.original`

`$ sudo nano /etc/bind/named.conf.options`

    # IP addresses and network ranges allowed to run recursive queries
    acl "trusted" {
        127.0.0.1;       # ns1 - can be set to localhost
        192.168.1.0/24;
    };

    options {
            directory "/var/cache/bind";
                    
            recursion yes;                 # enables resursive queries
            allow-recursion { trusted; };  # allows recursive queries from "trusted" clients
            listen-on { 192.168.1.11; };   # ns1 private IP address - listen on private network only
            allow-transfer { none; };      # disable zone transfers by default
    
            forwarders {
                    8.8.8.8;
                    8.8.4.4;
            };
    };
    
### Configurando o arquivo local do bind

`$ sudo cp /etc/bind/named.conf.local /etc/bind/named.conf.local.original`

`$ sudo nano /etc/bind/named.conf.local`

    zone "corp.rthomaz.com.br" {
        type master;
        file "/etc/bind/zones/db.corp.rthomaz.com.br"; # zone file path
        allow-transfer { 192.168.1.12; };              # ns2 private IP address - secondary
    };
    
    zone "168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/zones/db.192.168";  # 192.168.1.0/24 subnet
        allow-transfer { 192.168.1.12; };   # ns2 private IP address - secondary
    };
    
### Configurando a zona de encaminhamento

`$ sudo mkdir /etc/bind/zones`

`$ sudo nano /etc/bind/zones/db.corp.rthomaz.com.br`

    $TTL    604800
    @       IN      SOA     ns1.corp.rthomaz.com.br. admin.corp.rthomaz.com.br. (
                      3     ; Serial
                 604800     ; Refresh
                  86400     ; Retry
                2419200     ; Expire
                 604800 )   ; Negative Cache TTL
    ;
    ; name servers - NS records
         IN      NS      ns1.corp.rthomaz.com.br.
         IN      NS      ns2.corp.rthomaz.com.br.
    
    ; name servers - A records
    ns1.corp.rthomaz.com.br.          IN      A       192.168.1.11
    ns2.corp.rthomaz.com.br.          IN      A       192.168.1.12
    
    ; 192.168.1.0/24 - A records
    jenkins.corp.rthomaz.com.br.      IN      A       192.168.1.15
    nexus.corp.rthomaz.com.br.        IN      A       192.168.1.16
    
### Configurando a zona reversa

`$ sudo nano /etc/bind/zones/db.192.168`

    $TTL    604800
    @       IN      SOA     corp.rthomaz.com.br. admin.corp.rthomaz.com.br. (
                                  3         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ; name servers
          IN      NS      ns1.corp.rthomaz.com.br.
          IN      NS      ns2.corp.rthomaz.com.br.
    
    ; PTR Records
    11.1    IN      PTR     ns1.corp.rthomaz.com.br.      ; 192.168.1.11
    12.1    IN      PTR     ns2.corp.rthomaz.com.br.      ; 192.168.1.12
    15.1    IN      PTR     jenkins.corp.rthomaz.com.br.  ; 192.168.1.15
    16.1    IN      PTR     nexus.corp.rthomaz.com.br.    ; 192.168.1.16
    
### Verificando a sintaxe de configuração do BIND

Execute o comando a seguir para verificar a sintaxe dos arquivos _named.conf*_:

`$ sudo named-checkconf`

###  Verificando a configuração da zona de encaminhamento 

Execute o seguinte comando:

`$ sudo named-checkzone corp.rthomaz.com.br /etc/bind/zones/db.corp.rthomaz.com.br`

### Verificando a configuração da zona reversa

Execute o seguinte comando:

`$ sudo named-checkzone 168.192.in-addr.arpa /etc/bind/zones/db.192.168`

### Aplicando as configurações

Agora reinicie o bind para aplicar as configurações.

`$ sudo systemctl restart bind9`

## Referências

- https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-18-04-pt
