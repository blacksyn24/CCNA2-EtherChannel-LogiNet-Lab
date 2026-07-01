# 🔗 CCNA2 — EtherChannel LACP | LogiNet Bénin

![Cisco](https://img.shields.io/badge/Cisco-CCNA2-blue?style=for-the-badge&logo=cisco&logoColor=white)
![PacketTracer](https://img.shields.io/badge/Packet%20Tracer-8.x-orange?style=for-the-badge&logo=cisco)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Protocol](https://img.shields.io/badge/Protocol-EtherChannel%20LACP-purple?style=for-the-badge)
![Switches](https://img.shields.io/badge/Switches-2-red?style=for-the-badge)
![PCs](https://img.shields.io/badge/PCs-4-yellow?style=for-the-badge)
![Bandwidth](https://img.shields.io/badge/Bandwidth-200%20Mbps-green?style=for-the-badge)

---

## 📋 Description

Ce TP simule le réseau de l'entreprise **LogiNet Bénin** 🇧🇯.
Le lien entre les deux switches était saturé à 100 Mbps en
permanence causant des lenteurs pour les employés.
L'objectif est de configurer **EtherChannel LACP** pour
agréger 2 liens physiques en 1 lien logique de **200 Mbps**.

### Objectifs
- ✅ Comprendre le rôle d'**EtherChannel**
- ✅ Configurer **LACP** (protocole standard 802.3ad)
- ✅ Créer un **Port-Channel** entre 2 switches
- ✅ Tester la **redondance automatique** en cas de panne d'un lien

---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🔌 Switch | Cisco 2960-24TT | SW1 | Switch principal |
| 🔌 Switch | Cisco 2960-24TT | SW2 | Switch secondaire |
| 💻 PC | PC-PT | PC7 | Employé bureau 1 |
| 💻 PC | PC-PT | PC8 | Employé bureau 2 |
| 💻 PC | PC-PT | PC9 | Employé bureau 3 |
| 💻 PC | PC-PT | PC10 | Employé bureau 4 |

---

## 🗺️ Topologie

```
PC7 ──Fa0/3──┐                    ┌──Fa0/3──PC9
PC8 ──Fa0/4──┤                    ├──Fa0/4──PC10
             │                    │
            [SW1]══Po1 (200Mbps)═[SW2]
            Fa0/1────────────────Fa0/1
            Fa0/2────────────────Fa0/2
                       ↑
                  EtherChannel
               2 liens = 200 Mbps
```

<p align="center">
  <img src="images/topologie.png" width="800">
  <br>
  <em>Capture Cisco Packet Tracer — Topologie LogiNet</em>
</p>

---

## 🔌 Câblage

| De | Port | Vers | Port | Rôle |
|----|------|------|------|------|
| SW1 | Fa0/1 | SW2 | Fa0/1 | EtherChannel lien 1 |
| SW1 | Fa0/2 | SW2 | Fa0/2 | EtherChannel lien 2 |
| SW1 | Fa0/3 | PC7 | Fa0 | PC bureau 1 |
| SW1 | Fa0/4 | PC8 | Fa0 | PC bureau 2 |
| SW2 | Fa0/3 | PC9 | Fa0 | PC bureau 3 |
| SW2 | Fa0/4 | PC10 | Fa0 | PC bureau 4 |

---

## 📊 Plan d'adressage

| PC | IP | Masque | Switch | Port |
|----|-----|--------|--------|------|
| PC7 | 192.168.1.10 | 255.255.255.0 | SW1 | Fa0/3 |
| PC8 | 192.168.1.11 | 255.255.255.0 | SW1 | Fa0/4 |
| PC9 | 192.168.1.12 | 255.255.255.0 | SW2 | Fa0/3 |
| PC10 | 192.168.1.13 | 255.255.255.0 | SW2 | Fa0/4 |

---

## ⚙️ Configuration complète

### 🔧 SW1 — Switch principal

```cisco
enable
configure terminal
hostname SW1

! Sélectionner les 2 ports à agréger
! Ces 2 ports formeront le Port-Channel 1
interface range fa0/1-2
channel-group 1 mode active
exit

! Configurer l'interface logique créée
! Le trunk se configure sur Po1, pas sur fa0/1 ou fa0/2
interface port-channel 1
switchport mode trunk
exit

! Ports des PC en mode accès
interface range fa0/3-4
switchport mode access
exit

end
write
```

---

### 🔧 SW2 — Switch secondaire

```cisco
enable
configure terminal
hostname SW2

! Même configuration que SW1
! active + active = LACP fonctionne ✅
interface range fa0/1-2
channel-group 1 mode active
exit

interface port-channel 1
switchport mode trunk
exit

interface range fa0/3-4
switchport mode access
exit

end
write
```

---

## 🔍 Commandes de vérification

```cisco
! Vue globale EtherChannel
SW1# show etherchannel summary

! Détails complets du canal
SW1# show etherchannel 1 detail

! État de l'interface logique
SW1# show interfaces port-channel 1

! Vérifier la config des ports physiques
SW1# show interfaces fa0/1 switchport
SW1# show interfaces fa0/2 switchport

! Voir si STP voit bien 1 seul lien
SW1# show spanning-tree

! Configuration complète
SW1# show running-config
```

---

### 📊 Résultat attendu — show etherchannel summary

```
Group  Port-channel  Protocol  Ports
------+-------------+---------+-----------
1      Po1(SU)       LACP      Fa0/1(P) Fa0/2(P)
```

| Symbole | Signification |
|---------|---------------|
| **Po1** | Port-Channel 1 = lien logique |
| **S** | Layer 2 (switch) |
| **U** | Up = fonctionne ✅ |
| **LACP** | Protocole utilisé |
| **Fa0/1(P)** | Port dans le bundle ✅ |
| **Fa0/2(P)** | Port dans le bundle ✅ |

---

## 🧪 Tests de connectivité

```
✅ PC7  → ping 192.168.1.12  (SW1 → SW2)
✅ PC7  → ping 192.168.1.13  (SW1 → SW2)
✅ PC9  → ping 192.168.1.10  (SW2 → SW1)
✅ PC10 → ping 192.168.1.11  (SW2 → SW1)
```

---

## 🎭 Scénario — Test de redondance

### Simuler la panne de Fa0/1

```cisco
SW1# configure terminal
SW1(config)# interface fa0/1
SW1(config-if)# shutdown
```

### Vérifier l'état du canal

```cisco
SW1# show etherchannel summary
```

**Résultat :**
```
1    Po1(SU)    LACP    Fa0/1(D) Fa0/2(P)

Fa0/1(D) = down ❌
Fa0/2(P) = toujours actif ✅
Po1(SU)  = canal toujours UP ✅
```

### Retester la connectivité

```
PC7 → ping 192.168.1.12
✅ Fonctionne encore !
→ Trafic passe sur Fa0/2 uniquement
→ 100 Mbps au lieu de 200 Mbps
→ Réseau toujours opérationnel !
```

### Réactiver le lien

```cisco
SW1(config)# interface fa0/1
SW1(config-if)# no shutdown
```

### Vérifier le rétablissement

```cisco
SW1# show etherchannel summary
→ Po1(SU) Fa0/1(P) Fa0/2(P)
→ 200 Mbps rétablis ✅
```

---

## 🛠️ Dépannage

| Problème | Cause | Solution |
|---------|-------|---------|
| `Po1(SD)` | Canal down | Vérifiez les câbles |
| `Fa0/1(I)` | Port isolé | Config différente des 2 côtés |
| `Fa0/1(s)` | Port suspendu | Conflits de configuration |
| Canal ne se forme pas | Mode incompatible | Vérifiez active+active |

```cisco
! Diagnostic détaillé
SW1# show etherchannel 1 detail

! Si erreur de configuration, réinitialiser
SW1(config)# interface range fa0/1-2
SW1(config-if-range)# no channel-group 1
SW1(config-if-range)# channel-group 1 mode active
```

---

## 💡 Points clés à retenir

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `interface range fa0/1-2` | Sélectionner les ports à agréger |
| `channel-group 1 mode active` | Créer le canal LACP |
| `interface port-channel 1` | Configurer le canal logique |
| `switchport mode trunk` | Trunk sur Po1, pas sur les ports physiques |
| `show etherchannel summary` | Vérifier l'état du canal |
| `show etherchannel 1 detail` | Diagnostic détaillé |

---

## 📊 Comparatif avant/après

| | Avant EtherChannel | Après EtherChannel |
|---|---|---|
| **Bande passante** | 100 Mbps | 200 Mbps ✅ |
| **Redondance** | ❌ Aucune | ✅ Automatique |
| **STP** | Bloque 1 lien | Voit 1 seul lien logique |
| **Panne câble** | Coupure réseau | Continuité automatique |

---

## 🛠️ Outils

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco%20Packet%20Tracer-8.x-orange?style=flat-square&logo=cisco)
![Cisco IOS](https://img.shields.io/badge/Cisco%20IOS-15.x-blue?style=flat-square)
![GitHub](https://img.shields.io/badge/GitHub-black?style=flat-square&logo=github)

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Libre d'utilisation pour l'apprentissage et la formation réseau.
