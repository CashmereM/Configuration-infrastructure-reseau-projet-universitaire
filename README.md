# Configuration réseau

* IP INTERNE  : `10.0.24.24/24`
* IP EXTERNE  : `192.168.24.1/24`
* IP ROUTEUR  : `10.0.24.254/24` et `192.168.24.24/24`
* IP DHCP     : `10.0.24.1/24`

_A noter, pour les téléchargement de paquets comme le service bind9, vous devez lancer la commande une fois `apt update` juste avant d'effectuer les téléchargements de paquets_

# ROUTEUR

## Configuration du DNS

* On installe le DNS avec bind9 sur le Routeur : `apt-get install bind9`
* On ajoute la zone "max-ibra.com" dans /etc/bind/named.conf.local à l'aide de la commande nano ( ou autre editeur de texte) : 

```
zone "max-ibra.com" {
    type master;
    file "/etc/bind/maxi.db";
}
```

* On crée le fichier de zone `/etc/bind/maxi.db`
* Enfin on peut redémarrer le DNS : `systemctl restart bind9`

Le serveur DNS est placé sur le ROUTEUR car il a une visibilité sur les 2 réseaux. Grace à cela, le DNS peut donner les noms de domaine sur les 2 réseaux.

# DHCP
## Mise en place de la VLAN 

* On active l'interface eth0 : `ip link set up eth0`
* On crée l'id de la VLAN : `ip link add link eth0 name eth0.24 type vlan id 24`
* On active l'interface eht0.24 de la VLAN : `ip link set up eth0.24`
* On ajoute l'ip sur l'interface de la VLAN : `ip a add 10.0.24.1/24 dev eth0.24`
* On test la connexion : `ping 192.168.18.1`

Notre VLAN est maintenant en place sur la machine DHCP. ( On a segmenté le réseau 10.0.24.0 et on va configurer notre serveur DHCP dans ce segment de réseau )

## Configuration du serveur DHCP 

* On installe DHCP : `apt install isc-dhcp-server`
* On modifie la configuration du fichier `/etc/dhcp/dhcpd.conf` à l'aide de la commande nano, afin qu'il puisse donner des adresses sur le reseau INTRANET : 
* A noter que, l'adresse MAC diffère selon les machines donc doit être changer obligatoirement

```
option domain-name "max-ibra.com";
option domain-name-servers 192.168.24.24;

authorative;
default-lease-time 600;
max-lease-time 7200;

subnet 10.0.24.0 netmask 255.255.255.0 {
  range 10.0.24.75 10.0.24.100;
  option routers 10.0.24.254;
}
host I{
  hardware ethernet 26:87:1c:97:9e:5e; #A changer en fonction de l'adresse MAC de la machine Interne
  fixed-address 10.0.24.24;
}

```

* On modifie la configuration du `/etc/default/isc-dhcp-server` toujours avec la commande nano, afin qu'il attribue les adresses dans la sous-interface réseau eth0.24

```
INTERFACESv4="eth0.24"
INTERFACESv6=""
```

* ENfin, on démarre le DHCP : `systemctl start isc-dhcp-server`

Notre serveur DHCP est maintenant en place sur le ROUTEUR. Il peut donc attribuer les adresses IP en connaissant les différents routages 

# INTERNE

## Mise en place de la VLAN 

* On active l'interface eth0 de la VLAN : `ip link set up eth0`
* On crée l'id de la VLAN : `ip link add link eth0 name eth0.24 type vlan id 24`
* On active l'interface eth0.24 de la VLAN : `ip link set up eth0.24`
* On execute la commande : `dhclient` pour que le serveur nous attribue notre adresse
* On test la connexion : `ping 192.168.18.1 `

Notre VLAN est maintenant en place sur la machine INTERNE.

##  Configuration du HTTP 

* On installe 'apache2' : 'apt install apache2' 
* On configure le nat pour que le serveur réponde sur le port 80 et soit accessible via http://ROUTEUR:8080 depuis EXTERNE




