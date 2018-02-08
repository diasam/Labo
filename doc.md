---
author:
- Samuel, Aramis
title: 
- Progetto m146
subtitle: 
- Fase intermedia
version:
- 1.0.0
date:
- 08.02.2018
monofont:
- Menlo
mainfont:
- Palatino
sansfont:
- Helvetica
fontsize:
- 12pt
---

# Schema visio

# Ambiente di sviluppo

Per poter lavorare da casa ho dovuto simulare la rete interna
del firewall.

Per fare ciò ho creato una rete `NAT` in `virtualbox`.

Con il seguente comando  da terminale si può creare una rete NAT con 
la rete `10.10.10.0/24`.

``` {.bash .numberLines }
VBoxManage natnetwork add --netname m146 --network "10.10.10.0/24" --enable
```

# Macchine virtuali

## DNS e DHCP

Per configurare il server dns ho utilizzato `dhcpd`. 

Per installare `dhcpd` si utilizza il seguente comando.

``` { .bash .numberLines }
apk add acf-dhcp
```

Per configurarlo bisogna creare il file `dhcpd.conf` nella directory `/etc/dhcp/`.

In seguito il file di configurazione che ho fatto per il server dhcp.

``` { .bash .numberLines }
# Configurazione standard
default-lease-time 302400;
max-lease-time 604800;
ddns-update-style none;
log-facility local7;
authoritative;

subnet 10.10.10.0 netmask 255.255.255.0
{
   range "10.10.10.50 10.2.0.200";
   option domain-name-servers 10.10.10.254;
   option routers 10.10.10.1;
   option domain-name "m146.ch";
}
```

Infine i seguenti comandi per far partire il servizio dhcp e per farlo partire
a boot-time.

``` { .bash .numberLines }
rc-service dhcpd start 
rc-update add dhcpd
```

Per il server dns ho utilizzato `unbound`.

Per installarlo si utilizza il comando 

``` {.bash .numberLines }
apk add unbound
```

Per configuarlo si deve modificare il file `/etc/unbound/unbound.conf`.

```
server:
        verbosity: 1
## Specify the interface address to listen on:
        interface: 10.10.10.254
## To listen on all interfaces use:
#       interface: 0.0.0.0
        do-ip4: yes
        do-ip6: yes
        do-udp: yes
        do-tcp: yes
        do-daemonize: yes
        access-control: 0.0.0.0/0 allow
## Other access control examples
#access-control: 192.168.1.0/24 action
## 'action' should be replaced by any one of:
#deny (drop message)
#refuse (sends  a  DNS  rcode REFUSED error message back)
#allow (recursive ok)
#allow_snoop (recursive and nonrecursive ok).
## Minimum lifetime of cache entries in seconds.  Default is 0.
#cache-min-ttl: 60
## Maximum lifetime of cached entries. Default is 86400 seconds (1  day).
#cache-max-ttl: 172800
## enable to not answer id.server and hostname.bind queries. 
        hide-identity: yes
## enable to not answer version.server and version.bind queries. 
        hide-version: yes
## default is to use syslog, which will log to /var/log/messages.
use-syslog: yes
## to log elsewhere, set 'use-syslog' to 'no' and set the log file location below:
#logfile: /var/log/unbound
python:
remote-control:
        control-enable: no
## Note for forward zones, the destination servers must be able to handle recursion to other DNS server
## Forward all *.example.com queries to the server at 192.168.1.1
#forward-zone:
#        name: "example.com"
#        forward-addr: 192.168.1.1
## Forward all other queries to the Verizon DNS servers
forward-zone:      
        name: "."
## Level3 Verizon
        forward-addr: 9.9.9.9
        forward-addr: 9.9.9.9
```

Dopodichè farlo partire e fare in modo che si avvii a boot-time tramite i seguenti comandi.

``` { .bash .numberLines }
/etc/init.d/unbound start
rc-update add unbound
```

## WebServer

Il webserver è stato installato su un server con una distro di linux di nome `Alpine`.

Il webserver installato si chiama `lighttpd`, che è molto sicuro, performante e semplice.

Per installarlo basterà eseguire il seguente comando.

``` { .bash .numberLines }
apk add lighttpd
```

Rispettivamente, per avviarlo, fermarlo o riavviarlo si possono utilizzare i seguenti comandi

``` {.bash .numberLines }
rc-service lighttpd start
rc-service lighttpd stop
rc-service lighttpd restart
```

Infine per impostarlo a runlevel, cioè che si avvii automaticamente
all'accensione del server, si utilizza il seguente comando.

``` { .bash .numberLines }
rc-update add lighttpd default
```

Se si vogliono configurare dei parametri si deve modificare il file di configurazione
al seguente percorso.

```
/etc/lighttpd/lighttpd.conf
```

Mentre il percorso di default per l'htdocs si trova al seguente percorso.

```
/var/www/localhost/htdocs/
```

## FTP

## FTPS

# Firewall

# Test

+----------------------+---------------------------------------------------------------------------+
|    **Test Case**     |                                  TC-001                                   |
+======================+===========================================================================+
| **Nome**             | Webserver                                                                 |
+----------------------+---------------------------------------------------------------------------+
| **Descrizione**      | Testa il corretto funzionamento del webserver, se risponde alle richieste |
+----------------------+---------------------------------------------------------------------------+
| **Prerequisiti**     |                                                                           |
+----------------------+---------------------------------------------------------------------------+
| **Procedura**        | In una `bash`, utilizzare il comando `wget 10.10.10.251`                  |
+----------------------+---------------------------------------------------------------------------+
| **Risultati attesi** | Il file index.html viene salvato nella directory attuale                  |
+----------------------+---------------------------------------------------------------------------+

+----------------------+---------------------------------------------------------------------+
|    **Test Case**     |                               TC-002                                |
+======================+=====================================================================+
| **Nome**             | DHCP                                                                |
+----------------------+---------------------------------------------------------------------+
| **Descrizione**      | Testa il corretto funzionamento server dhcp                         |
+----------------------+---------------------------------------------------------------------+
| **Prerequisiti**     |                                                                     |
+----------------------+---------------------------------------------------------------------+
| **Procedura**        | Collegare una macchina virtuale alla rete virtuale NAT.  In seguito |
|                      | utilizzare il comando `ifconfig` e                                  |
+----------------------+---------------------------------------------------------------------+
| **Risultati attesi** | controllare che la interfaccia abbia unindirizzo IP compreso tra    |
|                      | `10.10.10.50` e `10.10.10.200`                                      |
+----------------------+---------------------------------------------------------------------+

