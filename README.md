Configuration Avancée du Frame Relay (Topologie Hub-and-Spoke)Ce dépôt présente un TP pratique réalisé dans le cadre de l'atelier Technologies WAN (RSI3, ISET Nabeul, année universitaire 2025/2026). L'environnement de simulation s'appuie sur GNS3 (version 2.2+) couplé à la GNS3 VM et utilise des routeurs Cisco c7200.🎯 Objectif du TPMettre en place un réseau WAN en topologie Hub-and-Spoke reliant un site central à deux agences via un commutateur Frame Relay, tout en intégrant un lien de secours PPP pour assurer la haute disponibilité.Le TP est divisé en deux parties progressives :Partie 1 : Utilisation des interfaces physiques, de l'ARP inverse, et du routage statique avec une distance administrative de 50 pour la route de secours (mise en évidence des limites de cette configuration).Partie 2 : Résolution des problèmes de la partie 1 grâce à l'utilisation de sous-interfaces point-to-point (une par PVC) et l'activation du protocole de routage dynamique RIPv2.📂 Contenu du Dépôtframe-relay-avance/
├── README.md
├── partie1/
│   └── TP_FrameRelay_avance_partie1_VM.gns3project
├── partie2/
│   └── TP_FrameRelay_avance_partie2_VM.gns3project
└── configs/
    └── TP_FrameRelay_avance_configs.txt
Fichiers .gns3project : Archives binaires à importer directement dans GNS3 (File > Import portable project).Fichier de configurations (configs/) : Contient l'intégralité des configurations textuelles des routeurs pour consultation et analyse de type diff.🛠️ Prérequis et Mise en routeEnvironnement : GNS3 2.2 ou supérieur avec la GNS3 VM active.Image IOS : Cisco c7200 (c7200-adventerprisek9-mz.153-3.XB12.image — à adapter selon votre installation dans les propriétés du routeur).Lancement :Importer le projet choisi dans GNS3.Démarrer tous les équipements (Start all devices) et patienter environ 2 minutes.Ouvrir les consoles des routeurs et saisir enable.⚠️ Les configurations de démarrage sont déjà injectées ; il ne faut rien coller manuellement dans les consoles.🔌 Correspondance des Interfaces (Cisco c7200)Pour ce TP, le routeur c7200 remplace les interfaces des énoncés théoriques :ÉnoncéÉquipement Réel (c7200)FastEthernet0/0FastEthernet0/0Serial0/0Serial1/0Serial0/1Serial1/1Serial0/1.16Serial1/1.16📋 Plan d'AdressageÉquipements et LANsC1 (Agence1) : 192.168.2.10/24 — Passerelle : 192.168.2.254C2 (SiteCentral) : 192.168.1.10/24 — Passerelle : 192.168.1.254C3 (Agence2) : 192.168.3.10/24 — Passerelle : 192.168.3.254Partie 1 : Interfaces Physiques et Routage StatiqueSiteCentral (Serial1/0 - PPP) : 172.16.1.1/24 $\rightarrow$ Agence1 (172.16.1.2/24)SiteCentral (Serial1/1 - Frame Relay) : 192.168.0.1/24Agence1 (Serial1/1 - Frame Relay) : 192.168.0.2/24Agence2 (Serial1/0 - Frame Relay) : 192.168.0.3/24LMI : Type ANSI sur toutes les interfaces Frame Relay.Partie 2 : Sous-interfaces Point-to-Point (DLCI)ÉquipementSous-interfaceAdresse IPDLCIRôle du PVCSiteCentralSerial1/1.16192.168.0.1/3016Vers Agence1 (Port 1 $\leftrightarrow$ Port 2)SiteCentralSerial1/1.17192.168.100.1/3017Vers Agence2 (Port 1 $\leftrightarrow$ Port 3)Agence1Serial1/1.16192.168.0.2/3016Vers SiteCentralAgence2Serial1/0.16192.168.100.2/3017Vers SiteCentral⚙️ Extraits de Configurations ClésPartie 1 : Routage Statique avec Secours (Exemple : SiteCentral)interface Serial1/1
 ip address 192.168.0.1 255.255.255.0
 encapsulation frame-relay
 frame-relay lmi-type ansi
!
! Route principale via Frame Relay
ip route 192.168.2.0 255.255.255.0 192.168.0.2
! Route de secours via PPP (Distance Administrative = 50)
ip route 192.168.2.0 255.255.255.0 172.16.1.2 50
ip route 192.168.3.0 255.255.255.0 192.168.0.3
Partie 2 : Sous-interfaces et RIPv2 (Exemple : SiteCentral)interface Serial1/1
 no ip address
 encapsulation frame-relay
 frame-relay lmi-type ansi
!
interface Serial1/1.16 point-to-point
 description PVC-avec-agence1
 ip address 192.168.0.1 255.255.255.252
 frame-relay interface-dlci 16
!
interface Serial1/1.17 point-to-point
 description PVC-avec-agence2
 ip address 192.168.100.1 255.255.255.252
 frame-relay interface-dlci 17
!
router rip
 version 2
 no auto-summary
 network 192.168.1.0
 network 192.168.0.0
 network 192.168.100.0
 network 172.16.0.0
🔍 Procédure de Test et Démonstration🧪 Partie 1 : Validation et Limites du StatiqueVérification des PVCs : Sur SiteCentral, tapez show frame-relay pvc (les DLCI 16 et 17 doivent être à l'état ACTIVE).Vérification des tables de routage : Tapez show ip route sur SiteCentral. Observez que la route de secours (métrique 50) est masquée tant que la liaison principale fonctionne.Test de panne : Coupez l'interface principale de l'Agence1 (shutdown sur Serial1/1).Résultat attendu : La route bascule automatiquement sur le lien PPP (redondance fonctionnelle).Problème identifié : Sur les interfaces physiques Frame Relay, des dysfonctionnements de routage/joignabilité apparaissent entre les agences en raison de la nature non-diffusante du mode multipoint (split-horizon).🚀 Partie 2 : Résolution (Sous-interfaces & RIPv2)Vérification des interfaces : Tapez show ip interface brief et show ip route sur SiteCentral pour valider que les sous-interfaces sont up et que les routes dynamiques R (RIP) sont apprises.Tests de connectivité : Depuis C2, effectuez des pings vers C3 (192.168.3.10) et C1 (192.168.2.10).Test de bascule propre : Désactivez l'interface Serial de l'Agence1 (shutdown). La route bascule de manière transparente sur le lien PPP et le traceur (traceroute) confirme le nouveau chemin de secours.
