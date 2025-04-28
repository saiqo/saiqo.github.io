---
date:
  created: 2025-03-13
  updated: 2025-03-13
authors:
  - saiqo
# slug: hello-world
categories:
    - Système
    - Serveur
tags:
    - Système
    - Serveur
    - HP
    - ILO
# links:
#   - docs/index.md
readtime: 15

---

# Modifier la vitesse des ventilateurs d’un serveur HP Proliant ML 350e Gen 8

Je possède un serveur HP ProLiant ML350e Gen8 v2 pour mon homelab, mais je le lance assez rarement, car les ventilateurs sont extrêmement bruyants. Même avec les ventilateurs à 30% de leur capacité, je pouvais entendre le serveur en étant dans une autre pièce de la maison avec la porte fermée.
<!-- more -->

Malheureusement, depuis l'interface de l'iLO 4 (Integrated Lights Out) du serveur, nous n'avons pas la possibilité de gérer la vitesse/la courbe des ventilateurs.

En explorant un peu les possibilités pour réduire le bruit, j'avais deux options. Soit je remplaçais les ventilateurs du serveur par des moins bruyant, soit je tentais d'utiliser un firmware d'iLO customisé, que j'ai trouvé sur un repo github, qui permet justement d'accéder.

Je suis parti sur cette deuxième option, bien qu'au départ, j'étais plutôt sceptique et contre l'idée d'installer un firmware customisé. De plus, si le flash du firmware se passe mal, je peux perdre l'iLO et donc me retrouver à devoir changer la carte mère du serveur car inutilisable. Mais je me suis laissé tenté par cette option pour pouvoir profiter du serveur, et je vais donc vous expliquer dans cet article, comment le faire.

Repo github du projet : [https://github.com/kendallgoto/ilo4_unlock/tree/main](https://github.com/kendallgoto/ilo4_unlock/tree/main)
<!--Content-->
## Disclaimer
L'installation d'un firmware personnalisé sur votre iLO 4 comporte des risques. J'ai réalisé cette opération dans un cadre personnel, avec un serveur qui m'appartient et qui n'est plus sous garantie.

Cet article est à titre informatif et pédagogique, je ne serais pas tenu responsable de tout dommage, ou dysfonctionnement résultant de l'installation de ce firmware.
De plus, l'utilisation des nouvelles fonctionnalités pour la gestion des ventilateurs peut entraîner une surchauffe du matériel et donc des dommages si celle-ci a été mal configuré.

Ne lancez pas cette installation si vous ne savez pas ce que vous faites, et soyez bien conscient des risques encourus. Prenez bien le temps de lire l'article étape par étape sans vouloir aller trop vite.

Pour ma part, j'ai testé ce firmware uniquement sur un serveur HPe ProLiant ML350e Gen8 v2 avec un iLO en version 2.77 (Version recommandé pour le patch).

Pour information, la version 2.77 est la dernière qui fonctionne avec le firmware customisé (à la date de la rédaction de cet article), toute version ultérieur à la 2.77 ne fonctionnera pas car HP à supprimé certaines fonctionnalités de contrôle des ventilateurs.

Je vous recommande donc de passer en **version 2.77**, comme recommandé par l'auteur du patch, pour être sûr que tout fonctionne correctement.


## Environnement

Nous allons construire l'image customisé de l'iLO et pour cela, il nous faut une clé usb bootable sur Linux, par exemple un live CD d'Ubuntu (J'ai utilisé Ubuntu 22.04 server).

Cette clé USB nous servira à construire et installer le patch sur notre serveur.

## Préparation

Dans un premier temps, il va falloir éteindre complètement le serveur et retirer le cordon d'alimentation.

Ensuite, il faut activer "iLO4 Security Override", il s'agit d'un petit jumper à passer à 1 sur la carte mère, le temps de l'installation.

Référez-vous au manuel de votre serveur. Vous pouvez aussi retrouver l'information sur le panneau latéral du boîtier du serveur.

Ensuite, démarrez le serveur et boot sur la clé USB contenant notre live CD Ubuntu.

## Flash du firmware
Une fois le LiveCD démarré, nous allons installer les programmes et fichiers nécessaires à la construction
de l'image patché.

### Installation des dépendances et modules nécessaires
```
sudo apt-add-repository universe
sudo apt update
sudo apt-get install python2-minimal git curl

curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py && sudo python2 get-pip.py

python2 -m pip install virtualenv

git clone --recurse-submodules https://github.com/kendallgoto/ilo4_unlock.git && cd ilo4_unlock

python2 -m virtualenv venv
source venv/bin/activate

pip install -r requirements.txt
```

### Build
Une fois toutes les dépendances installées, nous pouvons construire notre image grâce à :
```
./buid.sh init

./build.sh 277
```

Un dossier build a été créé avec les fichiers nécessaires.

### Vérifications
On s'assure qu'aucun module HP soient chargés :
```
sudo modprobe -r hpilo
```

### Flash
Nous allons maintenant copier les fichiers nécessaires dans un dossier puis lancer le flash.
``` bash
mkdir -p flash
cp binaries/flash_ilo4 binaries/CP027911.xml flash/
cp build/ilo4_277.bin.patched flash/ilo4_277.bin
cd flash

# Flash de l'iLO
./flash_ilo4 --direct
```

Les ventilateurs du serveur vont se mettre à tourner à 100%, cela risque d'être bruyant.
Si tout se passe bien, cela va durer 1 à 2 minutes.

Ne stoppez pas ou ne débranchez surtout pas le serveur pendant toute la durée du flash au risque de causer des problèmes.

Une fois le flash terminé (les ventilateurs ont dû se calmer), éteignez le serveur :
```
shutdown now
```

### Finalisations

Une fois le serveur éteint, débranchez le câble d'alimentation puis repasser le jumper du "iLO Security Override" sur la position initiale.

Vous pouvez maintenant démarrer le serveur.

Si tout s'est bien passé, vous pouvez vous connecter en SSH à l'iLO et vérifiez que l'utilitaire est disponible en lançant la commande `fan`


## Gestion des ventilateurs
Grâce à la commande `fan`, nous pouvons maintenant gérer à peu près tout ce qui concerne les ventilateurs du serveur

- Récupérer les infos des ventilateurs en temps réel
- Changer la vitesse MAX et MIN de chaque ventilateur
- Ajuster les seuils des capteurs de température
- ...

Quelques exemples :

Récupérer les infos sur les ventilateurs :
```
fan info
```
Fixer la valeur de rotation maximale à 50 :
```
fan p XX max 50
```
Avec `XX` le numéro du ventilateur, trouvable avec `fan info`

50 n'est pas le RPM ni un pourcentage du RPM, juste une valeur arbitraire entre 0 et X (X dépend de votre serveur, retrouvable via `fan info`).


Exemple pour mon serveur (affichage tronqué de `fan info`) :
```
PWMs:
No.  PWM   Min   Max  Hyst  BLOW LOCK   PCT   Status
 0    16    16   50     0   221    0      0   0x0001
 1    16    16   221    0   221    0      0   0x0001
 2    16    16   221    0   221    0      0   0x0001
 3    16    16   221    0   221    0      0   0x0001
 4    16    16   221    0   221    0      0   0x0001
 5    16    16   221    0   221    0      0   0x0001

```

J'ai défini la vitesse MIN à 16 et le MAX à 50 pour le ventilateur 0 et un MAX de 221 pour les autres (valeur par défaut).

À vous d'ajuster les valeurs en fonction de vos besoins.

Et pensez à vérifier les températures des différents éléments pour vous assurer que tout est correctement refroidi.

Pour information, après un redémarrage, la configuration que vous avez effectuée sur les ventilateurs est perdue, il faut donc entrer les commandes après chaque démarrage du serveur.


## Recommandations
Comme mentionné sur l'issue [https://github.com/kendallgoto/ilo4_unlock/issues/22](https://github.com/kendallgoto/ilo4_unlock/issues/22), après un redémarrage, nous perdons l'affichage des commandes, mais elles restent fonctionnelles

Par exemple `fan info` nous affiche rien.

Pour résoudre ce problème, il faut faire un "reset" de l'iLO (Menu Information > Diagnostics > Reset), sauf que cela est assez contraignant.

Je vous conseille donc de trouver la configuration qui vous convient puis de noter les commandes de côté et dès que vous démarrez votre serveur, vous avez juste à relancer ces commandes (possibilité d'utiliser des crontabs à chaque démarrage du serveur).

Vous trouverez également quelques exemples de script fait par la communauté sur le repo du projet pour régler les ventilateurs automatiquement.

La deuxième recommandation est de manipuler et régler les seuils des capteurs de température plutôt que fixer en dur une valeur maximale de vitesse de rotation des ventilateurs pour assurer un refroidissement plus "flexible". 