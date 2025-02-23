---
title: Openvswitch
type: docs
prev: docs/reseau/

---

#### Afficher la configuration du switch :

```bash
ovs-vsctl show
```

#### Conf

```bash
# Création du bridge
ovs-vsctl add-br br0

# Port en mode trunk
ovs-vsctl add-port br0 ens4
ovs-vsctl add-port br0 ens6
ovs-vsctl set port eth0 vlan_mode=trunk # Alternative :

#Autoriser seulement qq vlans sur interface trunk
ovs-vsctl add-port br0 eth5 trunks=3,4,5

# Conf port vlan 99
vs-vsctl add-port br0 ens5 tag=99

# Conf port vlan 10
ovs-vsctl add-port br0 ens7 tag=10
ovs-vsctl add-port br0 ens8 tag=10

# Conf port vlan 20
ovs-vsctl add-port br0 ens9 tag=20
ovs-vsctl add-port br0 ens10 tag=20


# Creation d'une interface vlan
ovs-vsctl add-port br0 vlan99 tag=99 -- set interface vlan99 type=internal

# Ajout IP pour interface vlan
ip address add X.X.X.X/24 dev vlan99
sudo ip link set vlan99 up

# Supression
ovs-vsctl del-port br0 ens4
```

---

Ressources : 

[Basic Configuration — Open vSwitch 3.4.90 documentation](https://docs.openvswitch.org/en/latest/faq/configuration/)

[www.openvswitch.org](https://www.openvswitch.org/support/dist-docs/ovs-vsctl.8.html)
