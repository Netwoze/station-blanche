# procédure configuration d'une station blanche sous linux.
apt-get -y install clamav



1) Créer une règle dans /etc/udev/rules.d/pour lancer un script au branchement de la clé USB

ACTION=="add", KERNEL=="sd[b-z][1-9]*", RUN+="/home/scan.sh $name"

ACTION : permet d’appliquer la règle pour tout ajout de périphérique
KERNEL : permet de spécifier le nom de la clé de correspondance, celle-ci doit commencer
RUN : permet de lancer une, ici c’est un script et $name permet de récupérer le nom de la clé de correspondance.


2) Création du script pour monter la clé USB et faire le scan
"""
#!/bin/bash
mkdir -p /media/$1/usb # création du dossier de l'arborescence
mount /dev/$1 /media/$1/usb # Monter la clé USB dans le répertoire créé précédemment

clamscan --no-summary -r /media/$1/usb/> /tmp/scan.txt # scan tous les répertoires et enregistre la sortie dans le fichier scan.txt
"""


3) Pour le bon fonctionnement et permettre de voir la clé USB monter
Faire la commande systemctl edit systemd-udevd puis inscrire :
[Service] MountFlags=shared PrivateMounts=no

"""
#!/bin/bash
# Créer l'arborescence pour la clé USB branché, le $1 permet de récupérer le nom de l'interface
mkdir -p /media/$1/usb
# Monter la clé USB dans le fichier créé précédemment
mount /dev/$1 /media/$1/usb
# Effectuer le scan de tous les répertoires et enregistrer la sortie dans le fichier scan.txt
clamscan --no-summary -r /media/$1/usb/ > /tmp/scan.txt # scan tous les répertoires et enregistre la sortie dans le fichier scan.txt
# création des fichiers ok et quarantaine
mkdir /media/$1/usb/ok
mkdir /media/$1/usb/quarantaine
# Boucle while qui s'arrête lorsqu'il n'y a plus de ligne
while read line
do
    # Si la fin de ligne du fichier contient 'FOUND' alors faire l'instruction ci-dessous
    if [[ "$line" *"FOUND" ]]; then
        :
    else
        :
    fi
    # garde essentiellement la partie avant ':' pour récupérer l'arborescence du fichier
    # déplace le fichier infecté dans le dossier 'quarantaine'
    mv `echo "$line" | cut -d ':' -f 1` /media/$1/usb/quarantaine/
    # garde essentiellement la partie avant ':' pour récupérer l'arborescence du fichier
    # déplace le fichier safe dans le dossier 'ok'
    mv `echo "$line" | cut -d ':' -f 1` /media/$1/usb/ok/
# Permet de prendre entré le ficher scan.txt dans lequel est enregistré la sortie du scan
done < '/tmp/scan.txt'
# Démonte la clé USB
umount /dev/$1
# Supprime l'arborescence créée
rm -r/media/$1
# Supprime le fichier de sortie du scan
rm /tmp/scan.txt

""""
