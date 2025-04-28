---
date:
  created: 2025-02-20
  updated: 2025-02-20
authors:
  - saiqo
categories:
  - Système
tags:
  - Microcore
  - OS

readtime: 10
---

# Installation de Micro Core Linux

Micro Core est une version encore plus minimaliste que Tiny Core (et oui, c'est possible). Avec une image pesant seulement 17 MB, vous pouvez avoir un système d'exploitation complet, fonctionnel et très rapide.
<!-- more -->
Cette distribution à la particularité d'être ultra-légère et peu consommatrice en termes de  ressources matérielles, ce qui la rend intéressante pour l'utiliser sur des machines peu puissantes ou encore pour déployer plusieurs machines virtuelles de tests sans être contraint par la puissance de votre ordinateur.

À noter que par défaut, Micro Core est conçu pour  être un système non-persistant, ce qui signifie qu'au prochain démarrage, toute modifications effectués sur la machine seront perdus. 

Nous allons voir dans ce tutoriel, comment le rendre persistant.

## Environnement
Pour des questions de simplicité, l'installation sera faite sur une machine virtuelle avec Vmware Workstation 17, mais il est tout à fait possible de se servir de ce tutoriel pour l'installer directement sur une machine.

## Installation minimale

### Création de la machine virtuelle

Téléchargez l’image (Core : 17 MB) : http://tinycorelinux.net/downloads.html

Ensuite, dans VMware workstation :

- File > New Virtual Machine > Next
- Installer disc image > Choisir l'image "core"
- Choisir Other > Other *
- 100 MB Ram
- 100 MB Disk
- Choisir un Réseau NAT


Démarrez la VM

Lorsque l’on arrive sur le premier menu `boot:` faites entrer

### Clavier en azerty temporaire

Pour simplifier l'installation, nous passons le clavier en français temporairement.

```bash
~ tce-load -wi kmaps.tcz
~ sudo loadkmap < /usr/share/kmap/azerty/fr-pc.kmap
```

Le clavier sera en Français (Jusqu’au prochain redémarrage)


### Script d'installation Micro Core

```bash
~ tce-load -wi tc-install.tcz
```

Cette commande va télécharger un script d’installation de MicroCore en CLI.

On exécute le script (en root): 

```bash
~ sudo tc-install.sh
```

Ici, plusieurs options vont nous être demandées :

- Boot from : CD-ROM (c)
- Install type : Frugal (f)
- Whole disk or partition (1)
- select disk sda (2 )
- Install a bootloader : y
- Type : enter
- ext4
- Option de boot : laissez les choix par défaut

On peut maintenant faire un `reboot`

Et voilà Micro Core est installé !

Cependant, notre système d'exploitation n'est pas encore persistant. La suite du tutoriel vous montrera comment le rendre persistant et installer quelques outils utiles.


## Installation avancée

### Clavier Azerty permanent

Afin de garder un clavier en français à chaque démarrage, nous devons faire quelques modifications à notre système.

On repasse le clavier en azerty 
```
~ tce-load -wi kmaps.tcz
~ sudo loadkmap < /usr/share/kmap/azerty/fr-pc.kmap
```

On modifie ensuite le fichier de boot  `/mnt/sda1/tce/boot/extlinux/extlinux.conf`

En rajoutant `kmap=azerty/fr-pc` après le paramètre `quiet`

``` bash title="extlinux.conf"
APPEND quiet kmap=/azerty/fr-pc waitusb ...
```

On termine avec un `reboot` et le clavier doit maintenant être en Azerty de façon permanente !


### Rendre un dossier persistant

Pour cela, nous avons deux options : 

- **Option 1 :** Utiliser un script qui sauvegarde les données dans un fichier compressé puis le charge à chaque démarrage. L’inconvénient est qu'il faut bien penser à sauvegarder avant chaque fois que vous allez éteindre/redémarrer la machine.

- **Option 2 :** Faire un montage de partition du dossier que l'on veut (Ex /home, /opt ...)

Personnellement, je vous recommande l'option 2 qui est plus pratique et plus fiable et évite les oublis de sauvegarde.


#### Option 1
Dans le dossier /opt/.filetoolst , renseignez tous les dossiers que vous souhaitez sauvegarder.

Exemple :
```bash title=".filetoolst"
/opt
/home
```

Ensuite, lancez la commande `bootsync.sh -b`

Si tout s'est bien passé, vous devriez avoir un fichier nommé `mydata.tgz`. Au démarrage, ce fichier va être décompressé et copié dans les dossiers adéquats. 


#### Option 2

On modifie le fichier de boot  `/mnt/sdXX/tce/boot/extlinux/extlinux.conf` :

En rajoutant `home=sdXX` dans les options de démarrage, comme ceci :

```bash title="extlinux.conf"
APPEND quiet kmap=/azerty/fr home=sda1 waitusb ...
```

Ce bootcode va permettre de lier automatiquement `/home/tc` avec `/mnt/sdXX/home/tc`

Ensuite :

1.  Supprimez le contenu du fichier `/opt/.filetool.lst` 

2. Vérifiez aussi qu’il n’y a pas de fichier **“mydata.tgz”** dans `/mnt/sdXX/tce`. S'il existe, supprimez-le.

Vous pouvez maintenant tester le bon fonctionnement de la persistance en créant un fichier dans le home et en redémarrant la machine

## Bonus

### Support d'IPv6

Par défaut, Micro Core ne supporte pas IPv6, il nous faut installer un module et le charger.

```
~ tce-load -wi ipv6-netfilter-6.6.8-tinycore.tcz
~ sudo modprobe ipv6
```

!!! info
    Le nom du paquet IPv6 à installer peut varier en fonction de votre version, pour le trouver : [http://tinycorelinux.net/16.x/](http://tinycorelinux.net/16.x/)

Cette configuration n'est que temporaire, il faut maintenant que la commande modprobe soit lancé à chaque démarrage. 

Pour cela, on peut rajouter la ligne suivante dans `/opt/bootsync.sh`

```bash title="bootsync.sh"
~ modprobe ipv6 & # Lancement du module en tache de fond
~ sleep 1 # On laisse le temps au module de charger
```


!!! note
    Le fichier `/opt/bootsync.sh` est un script chargé à chaque démarrage de la machine. Donc, depuis ce fichier, vous pouvez rajouter toutes les commandes et scripts que vous voulez lancer au démarrage.


### Pour aller plus loin
Pour plus de détails, je vous recommande le livre expliquant le fonctionnement détaillé de Tiny Core : [corebook](http://tinycorelinux.net/corebook.pdf) 


