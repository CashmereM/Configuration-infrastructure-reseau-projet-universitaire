<img src=SchemaReseau.jpg alt="" width="800"/>


# Configuration réseau

* IP INTERNE  : `10.0.24.24/24`
* IP EXTERNE  : `192.168.24.1/24`
* IP ROUTEUR  : `10.0.24.254/24` et `192.168.24.24/24`
* IP DHCP     : `10.0.24.1/24`
* IP DNS     : `10.0.24.5/24`

_A noter, pour les téléchargement de paquets comme le service bind9, vous devez lancer la commande une fois `apt update` juste avant d'effectuer les téléchargements de paquets_

# ROUTEUR
## Mise en place de la VLAN 
* On active l'interface eth0 : `ip link set up eth0`
* On crée l'id de la VLAN : `ip link add link eth0 name eth0.24 type vlan id 24`
* On active l'interface eht0.24 de la VLAN : `ip link set up eth0.24`
* On ajoute l'ip sur l'interface de la VLAN : `ip a add 10.0.24.254/24 dev eth0.24`

# DNS

## Configuration du DNS

* On installe le DNS avec bind9 sur le Routeur : `apt-get install bind9`
* On ajoute la zone "max-ibra.com" dans /etc/bind/named.conf.local à l'aide de la commande nano ( ou autre editeur de texte) : 

```
zone "max-ibra.com" {
    type master;
    file "/etc/bind/maxi.db";
};
```

* On crée le fichier de zone `/etc/bind/maxi.db`
* Enfin on peut redémarrer le DNS : `systemctl restart bind9`

Le serveur DNS est placé sur le ROUTEUR car il a une visibilité sur les 2 réseaux. Grace à cela, le DNS peut donner les noms de domaine sur les 2 réseaux.

## Mise en place de la VLAN 
* On active l'interface eth0 : `ip link set up eth0`
* On crée l'id de la VLAN : `ip link add link eth0 name eth0.24 type vlan id 24`
* On active l'interface eht0.24 de la VLAN : `ip link set up eth0.24`
* On ajoute l'ip sur l'interface de la VLAN : `ip a add 10.0.24.5/24 dev eth0.24`

# DHCP
## Mise en place de la VLAN 

* On active l'interface eth0 : `ip link set up eth0`
* On crée l'id de la VLAN : `ip link add link eth0 name eth0.24 type vlan id 24`
* On active l'interface eht0.24 de la VLAN : `ip link set up eth0.24`
* On ajoute l'ip sur l'interface de la VLAN : `ip a add 10.0.24.1/24 dev eth0.24`

Notre VLAN est maintenant en place sur la machine DHCP. ( On a segmenté le réseau 10.0.24.0 et on va configurer notre serveur DHCP dans ce segment de réseau )

## Configuration du serveur DHCP 

* On installe DHCP : `apt install isc-dhcp-server`
* On modifie la configuration du fichier `/etc/dhcp/dhcpd.conf` à l'aide de la commande nano, afin qu'il puisse donner des adresses sur le reseau INTRANET : 
* A noter que, l'adresse MAC diffère selon les machines donc doit être changer obligatoirement

```
option domain-name "max-ibra.com";
option domain-name-servers 10.0.24.5, 172.21.0.203, 172.21.0.204;
option domain-search "ad.iut-nantes.univ-nantes.prive";
option domain-search "iut-nantes.univ-nantes.prive";


authorative;
default-lease-time 600;
max-lease-time 7200;

subnet 10.0.24.0 netmask 255.255.255.0 {
  range 10.0.24.75 10.0.24.100;
  option routers 10.0.24.254;
}
host I{
  hardware ethernet 3e:41:90:ea:2b:a3; #A changer en fonction de l'adresse MAC de la machine Interne
  fixed-address 10.0.24.24;
  option routers 10.0.24.254;
}

host DNS{
  hardware ethernet de:15:9a:7f:4d:c0; #A changer en fonction de l'adresse MAC de la machine DNS
  fixed-address 10.0.24.5;
  option routers 10.0.24.254;
}

host ROUTEUR{
  hardware ethernet a6:dc:42:23:dc:50; #A changer en fonction de l'adresse MAC de la machine Routeur
  fixed-address 10.0.24.254;
}



```

* On modifie la configuration du `/etc/default/isc-dhcp-server` toujours avec la commande nano, afin qu'il attribue les adresses dans la sous-interface réseau eth0.24

```
INTERFACESv4="eth0.24"
INTERFACESv6=""
```

* ENfin, on démarre le DHCP : `systemctl start isc-dhcp-server`

Notre serveur DHCP est maintenant en place. Il peut donc attribuer les adresses IP en connaissant les différents routages 

# INTERNE

## Mise en place de la VLAN 

* On active l'interface eth0 de la VLAN : `ip link set up eth0`
* On crée l'id de la VLAN : `ip link add link eth0 name eth0.24 type vlan id 24`
* On active l'interface eth0.24 de la VLAN : `ip link set up eth0.24`
* On execute la commande : `dhclient` pour que le serveur nous attribue notre adresse

Notre VLAN est maintenant en place sur la machine INTERNE.

##  Configuration du HTTP 

* On installe `apache2` : `apt install apache2`
* On ajoute notre site mvc au répertoire `/var/www/html` ( recuperer le site dans l'archive 'site.tar' du git )

* On lance le mode rewrite : `a2enmod rewrite`
* On modifie le fichier du site : `nano /etc/apache2/sites-available/000-default.conf`
```                                   
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        RewriteEngine On
        RewriteRule ^$ /../Home [R=301,L]

        <Directory /var/www/html/ >
                AllowOverride All
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* On relance apache2 : `service apache2 restart`

# ROUTEUR
* On configure le nat pour que le serveur réponde sur le port 80 et soit accessible via http://ROUTEUR:8080 depuis EXTERNE
`iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to 10.0.24.24` #A tester



