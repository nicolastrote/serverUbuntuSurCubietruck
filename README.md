# serverUbuntuSurCubietruck

Installation d'un serveur Ubuntu Xenial sur le mini-pc CubieTruck

## Installation cubietruck armbian
Sources
https://docs.armbian.com/User-Guide_Fine-Tuning/#how-to-change-network-configuration
https://wiki.debian.org/fr/WiFi/HowToUse#WPA-PSK_et_WPA2-PSK
https://samhobbs.co.uk/2015/01/dynamic-dns-ddclient-raspberry-pi-and-ubuntu


Name: http://www.mysticmarguerite.com/WebDocs/Texts/LoveAsteroids.html

## Sommaire

Sources        1
Sommaire        1
Préparation de la carte SD pour l’installation        3
Démarrage de la cubietruck sur la carte SD        3
Comment s’identifier?        3
Comment installer Ubuntu sur la NAND?        3
Comment se connecter au wifi?        4
Renommer Cubietruck        4
Mise à jour du système        4
Connexion via SSH depuis un DynDNS        5
Sécuriser avec Fail2ban        5
Création d’une clé publique et privée        5
Configuration de votre routeur maison        6
Création d’un compte chez NO-IP        6
Utilisation de ddclient        6
Comment se connecter localement avec l’adresse de No-ip?        7
Si des erreurs de langues locales surviennent        7
Sécuriser avec le firewall UFW        7





## Préparation de la carte SD pour l’installation

Aller sur https://www.armbian.com/cubietruck/ pour télécharger l’image Ubuntu Legacy 3.4.112 (Desktop) actuellement. Attention la version debian ne reconnaît pas les claviers bluetooth!
En ligne de commande:
$ cd ~/Téléchargement
$ wget https://www.armbian.com/donate/?f=https://image.armbian.com/Armbian_5.20_Cubietruck_Ubuntu_xenial_3.4.112.7z
$ sudo apt-get install p7zip p7zip-full
$ 7z x  https://www.armbian.com/donate/?f=https://image.armbian.com/Armbian_5.20_Cubietruck_Ubuntu_xenial_3.4.112.7z

Insérez votre carte SD et repérer son nom avec la commande
$ sudo fdisk -l

Reponse /dev/sdb pour moi
Création de la carte SD
$ sudo dd bs=1M if=~/Téléchargements/Armbian_5.20_Cubietruck_Ubuntu_xenial_3.4.112/Armbian_5.20_Cubietruck_Ubuntu_xenial_3.4.112.img  of=/dev/sdb  && sudo sync 

Le média est prêt, il peut être insérer dans la cubietruck.

## Démarrage de la cubietruck sur la carte SD

Insérez la carte SD dans le lecteur et allumez la cubietruck. Le premier démarrage dure 3 min environ puis la carte redémarre et vous devez attendre une minute avant de vous identifier. 

## Comment s’identifier?

A l’idetification renseignez “root” pour le login et “1234” pour le mot de passe. Après cela, il vous sera demandé de changer le mot de passe root et de créer un compte utilisateur qui fait parti du groupe sudo. Attention le clavier est en QWERTY par défaut.

## Comment installer Ubuntu sur la NAND?

Lancez la commande :
$ nand-sata-install

Plusieurs options existent mais dans mon cas j’ai choisi d’installer Ubuntu sur la NAND pour avoir un système rapide, à faible consommation d’énergie et silencieux.

## Comment se connecter au wifi?

	.	Il faut restreindre les permissions d'accès au fichier /etc/network/interfaces, pour éviter de divulguer la clef mot de passe (PSK) (sinon, utilisez un fichier de configuration séparé tel que /etc/network/interfaces.d/wlan0 sur les versions de Debian les plus récentes)
	.	# chmod 0600 /etc/network/interfaces
	.	
	.	Utilisez la phrase secrète WPA pour calculer la clé de hachage WPA PSK correcte pour votre SSID en modifiant l'exemple suivant :
	.	$ wpa_passphrase myssid my_very_secret_passphrase
	.	Si vous n'entrez pas la phrase secrète sur la ligne de commande, elle vous sera demandée. La commande ci-dessus donne cette sortie :
	.	network={        ssid="myssid"        #psk="my_very_secret_passphrase"        psk=ccb290fd4fe6b22935cbae31449e050edd02ad44627b16ce0151668f5f53c01b }
	.	Vous devrez copier entre « psk= » et la fin de la ligne dans le fichier /etc/network/interfaces.
	.	Ouvrez /etc/network/interfaces dans un éditeur de texte :
	.	# vim /etc/network/interfaces
	.	
	.	Entrez les données de votre réseau sans fil, la SSID et la Clef WPA (PSK). Par exemple :
	.	auto wlan0 iface wlan0 inet dhcp    wpa-ssid NomRéseau    wpa-psk ccb290fd4fe6b22935cbae31449e050edd02ad44627b16ce0151668f5f53c01b
	.	La commande auto lancera automatiquement l'interface sans fil au démarrage du système. Vous devez commenter ou supprimer cette ligne si vous ne désirez pas ce fonctionnement.
	.	Sauvegardez et sortez de l'éditeur.
	.	Démarrez votre interface. Wpa_suppliquant démarrera en tâche de fond
	.	# ifup wlan0

## Renommer Cubietruck

Renommer cubietruck en cupido
$ sudo nano /etc/hostname
$ sudo nano /etc/hosts

## Mise à jour du système

$ sudo apt-get update && sudo apt-get -y upgrade && sudo apt-get autoremove && sudo apt-get autoclean
apt-get install console-data

## Connexion via SSH depuis un DynDNS

Open-ssh est déjà installé par défaut. Il est donc possible de se connecter à son pc à distance localement. Il vous faudra juste connaître votre ip (un  “sudo ifconfig” dans le terminal vous donnera l’adresse).
Pour tester à distance lancez : 
$ ssh utilisateur@adresse-ip-cubietruck -p 22

## Sécuriser avec Fail2ban

Sécuriser son SSH face aux attaques “bruteforce” par exemple, avec Fail2ban:
        $ sudo apt-get install fail2ban

Puis éditez le fichier de configuration:
$ sudo nano  /etc/ssh/sshd_config

Modifiez-le ainsi :
PermitEmptyPasswords no
PermitRootLogin no
UsePAM no
uniquement après avoir créer les clef changez le port 22 pour un port 443 par exemple
uniquement après avoir créer les clef changez : PasswordAuthentication no

MaxStartups 5:15:20      
Le 5 correspond au nombre de tentatives avant d'avoir droit à une nouvelle chance. 
Le 15 correspond au pourcentage de chance pour avoir une nouvelle tentative. 
Et le 20 correspond au nombre de connexions maximum. 

Puis redémarrez ssh avec la commande: 
$ sudo /etc/init.d/ssh restart

## Création d’une clé publique et privée

Lancez dans le terminal la commande ( /!\ pas en root afin qu’elle soit attribuée à $USER /!\ ): 
$ ssh-keygen

La clef publique est dans /home/$USER/.ssh/id_rsa.pub et la clef privée dans /home/$USER/.ssh/id_rsa

Pour envoyer sa clé publique sur l’ordi distant, lancez la commande : 
$ ssh-copy-id     -i    ~/.ssh/id_rsa.pub      utilisateur@adresse-ip-cubietruck

Puis redémarrez ssh avec : 
$ sudo /etc/init.d/ssh restart

Connectez vous à votre machine distante : 
$ ssh utilisateur@adresse-ip-cubietruck -p 443

Notez que j’ai entre temps j’ai changé le port de connexion SSH qui est ici 443 via le fichier de configuration.

## Configuration de votre routeur maison

Pour vous connecter depuis n’importe quel endroit à votre serveur, la première étape et d’indiquer avec routeur maison (i.e.: la box de votre FAI) que vous vous connecterez via le port 443 (dans mon cas) sur la machine 192.168.0.101.
Connectez vous à votre routeur, qui classiquement se trouve à l’adresse http://192.168.0.1 et rendez-vous sur la page de redirection des ports en indiquant (à compléter suivant le progiciel du routeur):
        Nom de la redirection :        SSH        
Type de connexion:                 les deux        
Port entrant:                         443
IP de la machine cible:        192.168.0.101
Nom de la machine:                cupido
Adresse MAC:                        00:22:f4:fb:8a:21
Port sortant:                        443

## Création d’un compte chez NO-IP

La seconde étape est de connaître à tout moment l’IP de votre routeur par lequel vous devez passer pour se connecter à la cubietruck. Pour cela un petit nom est plus simple! Allez sur le site de http://www.noip.com/ pour vous inscrire. Une fois l’inscription faite, allez sur la page “Dynamic DNS“, cliquez sur “Add Hostname” et ajoutez un nom qui définira une adresse pour votre routeur sur internet.
Exemple : gtxcloud.ddns.net

## Utilisation de ddclient

La dernière étape est de renseigner régulièrement No-ip l’adresse IP de votre routeur sur internet. Pour cela, nous utiliserons ddclient et ssl pour sécuriser la transaction:
        $ sudo apt-get install ddclient libio-socket-ssl-perl

Puis éditez le fichier de configuration de ddclient ainsi : 
        $ sudo nano /etc/ddclient.conf

Modifiez le fichier ainsi:
protocol=dyndns2
use=if, if=wlan0
ssl=yes    # permet de ne pas afficher en claire le mot de passe
server=dynupdate.no-ip.com
login=loginSurNoIp                                # à adapter
password='monMotDePasseNoIp'                # à adapter en gardant les guillemets
gtxcloud.ddns.net                                # à adapter

Puis redémarrez le service ainsi:
$ sudo service ddclient restart        

Vous pouvez à tout moment re-configurer ddclient avec la commande:
$ sudo dpkg-reconfigure ddclient

Si tout est bien renseigné, rendez-vous sur la page https://my.noip.com/#!/dynamic-dns , vous devriez voir l’adresse IP de votre routeur apparaître:
Gtxcloud.ddns.net    No Dynamic Update Detected   70.80.245.139

## Comment se connecter localement avec l’adresse de No-ip?

Pour utiliser localement l’adresse gtxcloud.ddns.net, renseignez-la dans le fichier hosts : 
$ sudo nano  /etc/hosts

Ajoutez en fin de fichier :          192.168.0.101                gtxcloud.ddns.net
Si des erreurs de langues locales surviennent

Il faut reconfigurer les locales avec : 
$ sudo dpkg-reconfigure locales

Dans mon cas, je dois faire mes choix pour obtenir ce qui met demandé dans le log d’erreur :
LANGUAGE = "en_US.UTF-8",
            LC_ALL = (unset),
            LC_MESSAGES = "en_US.UTF-8",
            LANG = "fr_CA.UTF-8"

## Sécuriser avec le firewall UFW

  à venir....











