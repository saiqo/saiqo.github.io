---
title: Modifier la vitesse des ventilateurs d’un serveur HP Proliant ML 350e Gen 8
date: 2025-03-05
description: Installation d'un firmaware customisé sur HP iLO 4 afin de régler manuellement la vitesse des ventilateurs d'un serveur ProLiant ML350e Gen8 v2
authors:
  - name: Saiqo
    link: https://github.com/saiqo
    image: https://github.com/saiqo.png
tags:
  - HP
  - ILO
  - System
excludeSearch: false
draft: true
---
<!--Intro-->

Je possède un serveur HP ProLiant ML350e Gen8 v2 pour mon homelab, mais je le lance assez rarement car les ventilateurs sont extrêmement bruyant. Même avec les ventilateurs à 30% de leur capacité, je pouvais entendre le serveur en étant dans une autre pièce de la maison avec la porte fermé. 

Malheureusement, depuis l'interface de l'iLO 4 (Integrated Lights Out) du serveur, nous n'avons pas la possibilité de gérer la vitesse / la courbe des ventilateurs.

En explorant un peu les possibiltés pour réduire le bruit, j'avais deux options. Soit je remplacais les ventilateurs du serveur par des moins bruyant, soit je tentais d'utiliser un firmware d'iLO customisé, que j'ai trouvé sur un repo github, qui permet justemment d'accéder 

Je suis parti sur cette deuxième option, bien que au départ, j'étais plutot sceptique et contre l'idée d'installer un firmware customisé. De plus, si le flash du firmware se passe mal, je peux potentiellement perdre l'iLO et donc me retrouver à devoir changer la carte mère du serveur. Mais je me suis laissé tenté par cette option pour pouvoir profiter du serveur, et je vais donc vous expliquer dans cet article, comment le faire.

Repo github du projet : https://github.com/kendallgoto/ilo4_unlock/tree/main

<!--Content-->
## Disclaimer
L'installation d'un firmware personnalisé sur votre iLO 4 comporte des risques potentiels. Je le fait dans un cadre personnel, avec un serveur qui m'apartient et qui n'est plus sous garantie.

Cet article est à titre informatif et pédagogique, et je ne suis pas tenu responsable de tout dommage, ou dysfonctionnement résultant de l'installation de ce firmware.
De plus, l'utilisation des nouvelles fonctionalités pour la gestion des ventilateurs peut entrainer une surchauffe si celle ci a été mal configuré.

Ne lancez pas cette installation si vous ne savez pas ce que vous faites, et soyez bien conscient des risques encourues.

Pour ma part, j'ai testé ce firmware uniquement sur un serveur HPe ProLiant ML350e Gen8 v2 en version 2.77.

Pour information, la version 2.77 est la dernière qui fonctionne avec le firmware customisé, toutes versions ultérieurs à la 2.77 ne fonctionneront pas car HP à supprimer certains outils qui étaient utilisés par le firmware custom.

Je vous recommande donc d'upgrade ou downgrade en version 2.77 pour être sur que tout fonctionne correctement.

## Environnement
Dans cette étape, nous allons build l'image customisé de l'iLO et pour cela, il va nous falloir une machine pour compiler les fichiers.

J'ai utilisé une VM Ubuntu 22.04, avec python2 d'installé.

Il nous faudra aussi une clé usb bootable sur Linux, par exemple un live CD d'Ubuntu.

## Préparation

Nous allons maintenant télécharger les fichiers nécessaires et compiler le firmware sur notre VM Ubuntu 

### Installation des dépendances
```
sudo apt-add-repository universe
sudo apt update
sudo apt-get install python2-minimal git curl
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py && sudo python2 get-pip.py
python2 -m pip install virtualenv
git clone --recurse-submodules https://github.com/kendallgoto/ilo4_unlock.git
cd ilo4_unlock
python2 -m virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Build
```
./buid.sh init

./build.sh 277
```

## Installation
Première étape, il va falloir éteindre complètement le serveur et retirer le cordon d'alimentation.

Ensuite il faut activé "iLO4 Security Override", il s'agit d'un petit jumper à passer à 1 sur la carte mère, le temps de l'installation.

Ensuite, démarrez le serveur et boot sur la clé USB contenant notre live CD Ubuntu.