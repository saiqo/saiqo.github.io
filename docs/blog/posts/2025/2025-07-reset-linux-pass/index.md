---
date:
  created: 2025-07-20
  updated: 2025-07-20
authors:
  - saiqo
# slug: hello-world
categories:
    - Linux
    - Système
tags:
    - Linux
    - Système

# links:
#   - docs/index.md
readtime: 5

---

# Réinitialiser un mot de passe perdu sur un système Linux
Dans cet article, nous allons voir 2 méthodes pour réinitialiser un mot de passe perdu sur un système Linux. La première méthode utilise GRUB, et la seconde méthode utilise un Live CD/USB.
<!-- more -->

!!! Note "Disclaimer"
    Cette méthode fonctionne sur un Linux utilisant `systemd` ou `OpenRC`. De plus, si votre disque dur est chiffré, ou que GRUB est protégé par un mot de passe, il vous faudra d'abord le déchiffrer et/ou déverrouiller GRUB.

## Méthode 1 : Réinitialiser le mot de passe en utilisant GRUB
###  1) Accès à un shell
Dans un premier temps, nous allons éditer les paramètres de démarrage de GRUB pour pouvoir accéder à un shell sur la machine.

Lorsque vous démarrez votre machine, vous devriez voir l'écran de GRUB. 

???+ Warning "Attention"
    Il est possible que l'écran de GRUB ne s'affiche pas par défaut ou si votre système démarre trop rapidement.
    Dans ce cas, vous pouvez maintenir la touche `Shift` ou `Esc` juste après le démarrage pour afficher le menu de GRUB.

Appuyez sur la touche `e` pour éditer les paramètres de démarrage.
Recherchez la ligne qui commence par `linux`, voici un exemple :

``` bash
linux /boot/vmlinuz-6.8.0-51 root=UUID=XXXX ro quiet splash
```

Ajoutez `init=/bin/bash` à la fin de cette ligne, ce qui devrait ressembler à ceci :

``` bash
linux /boot/vmlinuz-6.8.0-51 root=UUID=XXXX ro quiet splash init=/bin/bash
```

Ensuite, appuyez sur `Ctrl + X` ou `F10` pour démarrer avec ces nouveaux paramètres.

Vous devriez maintenant être dans un shell ressemblant à ceci :

``` bash
root@(none):/# 
```

### 2) Remonter le système de fichiers en écriture
Par défaut, le système de fichiers est monté en lecture seule. Pour pouvoir modifier le mot de passe, nous devons le remonter en écriture. Exécutez la commande suivante :

``` bash
$ mount -o remount,rw /
```

Vous pouvez vérifier que le système de fichiers est monté en écriture en exécutant :

``` bash
$ mount
```

Vous devriez voir une ligne similaire à celle-ci :

``` bash
/dev/sda1 on / type ext4 (rw,relatime)
```

### 3) Réinitialiser le mot de passe
Maintenant que le système de fichiers est monté en écriture, vous pouvez réinitialiser le mot de passe de n'importe quel utilisateur (également root). Par exemple, pour réinitialiser le mot de passe de l'utilisateur `user`, exécutez :

``` bash
$ passwd user
```

On peut maintenant continuer le démarrage du système, avec notre nouveau mot de passe en exécutant :

``` bash
$ exec /sbin/init
```

## Méthode 2 : Réinitialiser le mot de passe avec un Live CD/USB

Pour le bon fonctionnement de cette méthode, il vous faudra une clé usb avec un Live CD/USB pour pouvoir démarrer dessus et modifier la partition de votre système.

### 1) Identifier et monter la partition racine
Une fois que vous avez démarré sur votre Live CD/USB, ouvrez un terminal et listez les partitions avec :

```bash
$ lsblk
```

Repérez la partition racine de votre système (Par exemple `/dev/sda2` ou `/dev/sda3`). Montez-la dans un dossier temporaire :

```bash
$ mkdir /mnt/mapartition
$ mount /dev/sda3 /mnt/mapartition
```
Remplacez `/dev/sda3` par la partition de votre système.

On peut vérifier que la partition est bien montée en regardant le contenu du dossier monté :

```bash
$ ls /mnt/mapartition
```

Vous devriez voir les fichiers de votre système, comme `etc`, `home`, etc.

### 2) Chroot
Pour accéder à votre système comme si vous étiez dessus, entrez dans un chroot sur la partition montée :

```bash
$ chroot /mnt/mapartition
```

### 3) Réinitialisation le mot de passe
Dans le chroot, réinitialisez le mot de passe de l’utilisateur souhaité (par exemple `user` ou un autre utilisateur) :

```bash
$ passwd user
```

### 4) Démonter la partition et redémarrer
Sortez du chroot et démontez la partition :

```bash
$ exit
$ umount /mnt/mapartition
```

Redémarrez votre ordinateur, retirez la clé USB et démarrez sur votre système. Vous devriez maintenant pouvoir vous connecter avec le nouveau mot de passe.

