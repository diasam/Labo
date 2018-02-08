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

Per configurare il server dns ho utilizzato `dhcpd` mentre per il server dns ho utilizzato `unbound`.

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

