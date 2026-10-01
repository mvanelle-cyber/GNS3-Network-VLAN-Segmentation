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
