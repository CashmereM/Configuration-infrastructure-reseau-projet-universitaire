# Configuration réseau


* IP INTERNE : `10.0.24.24/24`
* IP EXTERNE : `192.168.24.1/24`
* IP ROUTEUR : `10.0.24.254/24` et `192.168.24.24/24`

## Configuration du DNS

* On installe le DNS avec bind9 sur le Routeur : `apt-get install bind9`
* On ajoute la zone DNS dans /etc/bind/named.conf.local à l'aide de nano : 

```
zone "max-ibra.com" {
    type master;
    file "/etc/bind/maxi.db";
}
```

* On crée le fichier de sauvegarde `/etc/bind/maxi.db`
* Enfin on peut redémarrer le DNS : `systemctl restart bind9`

Le serveur DNS est placé sur le ROUTEUR car il a une visibilité sur les 2 réseaux. Grace à cela, le DNS peut donner les noms de domaine sur les 2 réseaux.

## Mise en place de la VLAN 

* On crée l'id de la VLAN : `ip link add link eth0 name eth0.24 type vlan id 24`
* On ajoute l'ip sur l'interface de la VLAN : `ip a add 10.0.24.24/24 dev eth0.24`
* On active l'ip de la VLAN : `ip link set up eth0.24`
* On test la connexion : `ping 192.168.18.1 `

Notre VLAN est maintenant en place sur la machine INTERNE car c'est la partie privée du réseau qui est ségmentée.


## Configuration du serveur DHCP 

* On installe DHCP : `apt install isc-dhcp-server`
* On modifie la configuration du fichier `/etc/dhcp/dhcpd.conf` à l'aide de la commande nano, afin qu'il puisse donner des adresses sur le reseau INTRANET : 

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
  hardware ethernet 26:87:1c:97:9e:5e;
  fixed-address 10.0.24.24;
}

```

* ENfin, on démarre le DHCP : `systemctl start isc-dhcp-server`

Notre serveur DHCP est maintenant en place sur le ROUTEUR. Il peut donc attribuer les adresses IP en connaissant les différents routages 


##  Configuration du HTTP 

* On installe 'apache2' : 'apt install apache2' 
* On configure le nat pour que le serveur réponde sur le port 80 et soit accessible via http://ROUTEUR:8080 depuis EXTERNE




