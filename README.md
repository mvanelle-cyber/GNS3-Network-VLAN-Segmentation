# GNS3-Network-VLAN-Segmentation
Architecture de commutation et sécurité réseau sous GNS3 : Isolation de trafic inter-services par VLANs.
##  Objectif du Scénario
Sécuriser les flux d'une infrastructure d'entreprise sous GNS3 en isolant le trafic de deux services distincts (RH et Comptabilité) via une segmentation de Niveau 2 (VLANs), tout en résolvant un incident critique lié au pilote de capture de paquets.

##  Résolution d'Incident Système (Troubleshooting)
* **Incident rencontré :** Dysfonctionnement du moteur d'émulation Dynamips ("Could not determine the Dynamips version"). Blocage lié à l'absence ou la corruption de la bibliothèque dynamique "wpcap.dll" requise par Wireshark.
* **Résolution :** Déploiement et mise à niveau du pilote de capture réseau **Npcap 1.89** en mode d'administration, avec activation impérative de l'option de compatibilité API WinPcap afin de restaurer les hooks de capture et de simulation de paquets.

##  Configuration et Durcissement Réseau
* **VLAN 10 (Service RH) :** Configuration du port "Ethernet0" du Switch principal en mode Access dédié au VLAN 10. Attribution de l'IP fixe "192.168.1.1/24" sur PC1.
* **VLAN 20 (Service Comptabilité) :** Configuration du port "Ethernet1" du Switch principal en mode Access dédié au VLAN 20. Attribution de l'IP fixe "192.168.1.2/24" on PC2.

##  Validation de l'Étanchéité (Tests de Charge)
* Les tests d'interconnexion initiaux (VLAN par défaut) affichaient un routage fluide du trafic ICMP.
* Suite à l'application des masques de VLAN, l'isolation logique est validée avec succès : toute tentative de ping inter-VLAN (PC1 vers PC2) se heurte à un rejet automatique du commutateur avec l'erreur réglementaire : "host not reachable".
  
##  Laboratoire Complémentaire : Vulnérabilité des Infrastructures Combinées (Switch vs Hub)

### Descriptif de l'Audit Cyber
Mise en évidence de la surface d'attaque induite par l'utilisation d'équipements de Niveau 1 (Hubs) face à l'étanchéité des architectures commutées de Niveau 2 (Switches).

###  Topologie de Test
* **Zone Commutée (Sécurisée) :** `PC_RH` isolé de manière étanche sur le Switch Central.
* **Zone Partagée (Vulnérable) :** `PC_Compta` et un tiers malveillant (`PC_Espion`) cohabitant sur un Ethernet Hub.

###  Preuve d'Interception (Analyse Wireshark)
* **Constat technique :** Activation d'une sonde de capture passive (Sniffing) sur l'interface du `PC_Espion` (`192.168.20.99`).
* **Résultat empirique :** Capture réussie de l'intégralité du trafic ICMP/ARP circulant entre la comptabilité et le reste du réseau. Le Hub dupliquant aveuglément les trames, la confidentialité des données inter-services est rompue à l'échelle du segment de diffusion.

##  Laboratoire 7 : Architecture Multi-Commutateurs Résiliente & Marquage 802.1Q (Projet Avancé)

###  Descriptif de l'Infrastructure
Déploiement d'une topologie d'entreprise résiliente de Niveau 2/3 à trois commutateurs interconnectés (`SW-Core`, `SW-Etage1`, `SW-Etage2`) pour valider le transport de flux étanches via des liaisons dorsales d'agrégation.

###  Preuve de Concept & Analyse Wireshark (Trunking)
* **Mécanisme audité :** Capture de paquets sur la dorsale Trunk inter-switches reliant le cœur de réseau au commutateur d'accès.
* **Constat empirique :** Interception réussie d'une trame de diffusion ARP. L'analyse microscopique confirme l'injection et la présence de l'en-tête **`802.1Q Virtual LAN`** taguée avec le **`VLAN ID: 10`**. 
* **Sécurité validée :** L'isolation logique est parfaitement opérante; malgré le transport physique partagé sur la même dorsale, le trafic du VLAN 10 reste totalement invisible pour les segments du VLAN 20.


 Laboratoire 8 : Architecture Réseau Redondante & Haute Disponibilité (FHRP / HSRP-vrrp Simulation)

 Descriptif de l'Infrastructure
Conception et déploiement d'une topologie d'entreprise en losange à haute disponibilité (High Availability) combinant la segmentation de Niveau 2 (VLANs/Trunking) et les mécanismes de redondance de passerelle par défaut. L'objectif est de garantir une résilience totale de l'infrastructure face aux pannes matérielles critiques.

 Topologie & Plan d'Adressage Statique
L'infrastructure s'organise autour d'une double liaison montante stratégique :
* **Segment d'Accès Redondant :** Les clients du `VLAN 10` convergent vers le commutateur `SW-Acces` interconnecté simultanément à deux équipements de routage/passerelle.
* **Passerelle Virtuelle Commune :** `192.168.10.1/24` (FHRP Virtual Gateway).
* **Nœud Actif (Master) :** `R1-Principal` interceptant les flux nominaux.
* **Nœud Passif (Backup) :** `R2-Secours` prêt à reprendre le trafic de manière transparente.

 Déroulement du Test d'Audit (Simulation de Panne Critique)

1. **Phase Nominale :**
   Configuration des hôtes VPCS avec persistance des données (`save`) et ciblage de la passerelle virtuelle :
   ```bash
   PC-Client1> ip 192.168.10.10/24 192.168.10.1
   PC-Client1> ping 192.168.10.20 -t
   ```
   *Constat :* Transmission fluide et ininterrompue des trames ICMP à travers le flanc gauche de la topologie.

2. **Phase d'Incident (Injection du Crash) :**
   Coupure physique et logicielle instantanée du nœud prioritaire (`R1-Principal ➡️ Stop`). Les indicateurs de liaison virent au rouge.

3. **Phase de Résilience (Bascule HA) :**
   * *Comportement observateur (Sonde Wireshark / VPCS) :* Perte éphémère d'un unique paquet (Timeout < 2s).
   * *Résolution autonome :* Convergence immédiate du switch d'accès vers le second chemin physique. `R2-Secours` assume l'identité de la passerelle virtuelle `192.168.10.1`. Le flux de requêtes reprend automatiquement sans intervention humaine ni reconfiguration des clients.

 Compétences Validées
* Modélisation de réseaux tolérants aux pannes (Fault-Tolerant Network Architectures).
* Maîtrise conceptuelle des protocoles de redondance du premier saut (FHRP / HSRP / VRRP).
* Configuration d'interfaces Trunk 802.1Q distribuées sur des topologies de distribution multiniveaux.
