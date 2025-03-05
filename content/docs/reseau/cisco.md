---
title: Cisco
type: docs
prev: docs/reseau/

---
## Liste de commandes utiles pour équipements Cisco

#### Capture de traffic sur une interface
```bash
# Créer une capture de paquet sur l'interface Gi1/0/1 :
monitor capture test_capture interface Gi1/0/1 both

# Capturer depuis any en IPv4 vers 8.8.8.8 :
monitor capture test_capture match ipv4 any 8.8.8.8/32

# Démarrer la capture :
monitor capture test_capture start

# Arrêter la capture en cours :
monitor capture test_capture stop

# Afficher dans la console ce qui a été capturé :
show monitor capture test_capture buffer brief

# Enregistrer la capture sur le switch:
monitor capture test_capture export location flash:test_capture.pcap

# Supprimer une capture :
no monitor capture test_capture

# Récupérer la capture sur un PC (commande sur pc)
scp -O toto@switch:flash:/fichier.pcap. 
```

#### Mise à jour d’un switch depuis une clé USB
```bash
copy usbflash0:<.bin file> flash:
install add file flash:<new .bin file> activate commit
install remove inactive # Pour supprimer les images inactives
```

#### Afficher les détails d’un module SFP (ex: specs du SFP)

```bash
show idprom interface TwentyFiveGigE 1/1/4
```

#### Autoriser l’utilisation de SFP/GBIC non cisco
```
service unsupported-transceiver
no errdisable detect cause gbic-invalid
```

#### Test TDR cable
```
test cable-diagnostics tdr interface gigabitEthernet 1/0/1
show cable-diagnostics tdr interface gigabitEthernet 1/0/1
```

#### Ne pas avoir de pagination
```
terminal length 0
```

#### Remettre la configuration par défaut d’un élément (ex: interface)
```bash
default interface gigabitEthernet 1/0/2
```
#### Configurer l'historique de commande entrées sur le switch
```bash
archive 
log config
logging enable
```

#### Afficher l’historique de commandes tapées
```bash
show archive log config all
```

#### Afficher les informations sur un stack de switch
```bash
show switch details

show platform
```

#### Afficher des détails sur les câbles de stack (permet aussi de voir si câble défaillant) 
```bash
show switch stack-ports summary
```

#### Réinitialiser un switch sans disposer du mot de passe
Dans un premier temps, il faut rentrer dans le bootloader

Deux méthodes pour entrer dans le bootloader :
- Démarrer le switch. Pressez immédiatement et maintenir le boutton mode. Maintenir pressé le boutton jusqu'a ce que la led status passe en orange

- Démarrer le switch. Puis presser plusieurs fois le boutton mode jusqu'a l'affichage du bootloader

Une fois dans le bootloader, le menu doit ressembler à cela :
```
Switch:
```

Entrez la ligne suivante, qui va permettre d'ignorer la startup-config au démarrage du switch
```
Switch: SWITCH_IGNORE_STARTUP_CFG=1
```

Démarrez le switch
```
Switch: boot
```

Une fois que le switch à démarré, copier la startup-config dans la running-config
```
Switch# copy startup-config running-config
```

Changez vos mot de passe puis supprimer la variable définie précédemment dans le boot loader avec :
```
Switch(config)# no system ignore startupconfig switch all
```

Sauvegarde de la configuration
```
Switch# copy running-config startup-config
```
