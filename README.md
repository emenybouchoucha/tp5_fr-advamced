Configuration avancée du Frame Relay (hub and spoke)

TP de l'atelier Technologies WAN – RSI3, ISET Nabeul (année 2025/2026). Réalisé sous GNS3 avec des routeurs Cisco c7200 et la GNS3 VM.

Objectif

Mettre en place un WAN Frame Relay en topologie hub and spoke entre un site central et deux agences, avec un lien de secours PPP :

Partie 1 : PVC sur interfaces physiques, ARP inverse, routage statique avec distance administrative 50 pour la route de secours. Observation du problème quand une interface tombe.
Partie 2 : solution avec des sous-interfaces point-to-point (une par PVC) et du routage dynamique RIPv2.
Topologie
                 C2 (192.168.1.10)
                  |
                COMM2
                  |
               fa0/0 .254
             [SiteCentral]  192.168.1.0/24
          s0/0 .1       s0/1 .1
            |              |
  172.16.1.0/24 (PPP)   192.168.0.0/24 (Frame Relay)
            |              |
         s0/0 .2         port 1
        [Agence1]  s0/1 .2 -- port 2 [ FR ] port 3 -- s0/0 .3 [Agence2]
        fa0/0 .254                                      fa0/0 .254
            |                                               |
          COMM1                                           COMM3
            |                                               |
   C1 (192.168.2.10)                              C3 (192.168.3.10)
   192.168.2.0/24                                 192.168.3.0/24
Contenu du dépôt
frame-relay-avance/
├── README.md
├── partie1/
│   └── TP_FrameRelay_avance_partie1_VM.gns3project
├── partie2/
│   └── TP_FrameRelay_avance_partie2_VM.gns3project
└── configs/
    └── TP_FrameRelay_avance_configs.txt

Les fichiers .gns3project sont des archives binaires (zip) : ils s'importent directement dans GNS3. Le fichier configs contient les configurations en texte, utiles pour les relire dans un diff.

Prérequis
GNS3 2.2 ou plus récent, avec la GNS3 VM démarrée.
Image IOS c7200 : c7200-adventerprisek9-mz.153-3.XB12.image (à adapter dans les propriétés des routeurs si le nom diffère).
Wireshark (optionnel, pour les captures).
Utilisation
Démarrer la GNS3 VM, puis GNS3.
File → Import portable project et choisir le projet de la partie voulue, sous un nouveau nom.
▶ Start all devices, puis attendre environ 2 minutes.
Ouvrir les consoles et taper enable sur chaque routeur.

Les configurations sont chargées au démarrage : il ne faut rien coller dans les consoles.

Correspondance des interfaces

Le c7200 remplace les interfaces des énoncés :

Énoncé	Projet
fa0/0	FastEthernet0/0
s0/0	Serial1/0
s0/1	Serial1/1
s0/1.16	Serial1/1.16
Plan d'adressage
Équipement	Interface	Adresse (partie 1)	Remarque
SiteCentral	FastEthernet0/0	192.168.1.254/24	LAN de C2
SiteCentral	Serial1/0	172.16.1.1/24	PPP vers Agence1
SiteCentral	Serial1/1	192.168.0.1/24	Frame Relay (port 1 du commutateur)
Agence1	FastEthernet0/0	192.168.2.254/24	LAN de C1
Agence1	Serial1/0	172.16.1.2/24	PPP vers SiteCentral
Agence1	Serial1/1	192.168.0.2/24	Frame Relay (port 2)
Agence2	FastEthernet0/0	192.168.3.254/24	LAN de C3
Agence2	Serial1/0	192.168.0.3/24	Frame Relay (port 3)

Hôtes (VPCS) : C1 192.168.2.10/24, C2 192.168.1.10/24, C3 192.168.3.10/24, passerelle en .254.

Adressage de la partie 2 (sous-interfaces point-to-point) :

Équipement	Sous-interface	Adresse	DLCI
SiteCentral	Serial1/1.16	192.168.0.1/30	16
SiteCentral	Serial1/1.17	192.168.100.1/30	17
Agence1	Serial1/1.16	192.168.0.2/30	16
Agence2	Serial1/0.16	192.168.100.2/30	16
Circuits virtuels du commutateur Frame Relay
PVC	Port source / DLCI	Port destination / DLCI
SiteCentral – Agence1	port 1 / 16	port 2 / 16
SiteCentral – Agence2	port 1 / 17	port 3 / 16

LMI de type ANSI sur toutes les interfaces Frame Relay. L'encapsulation PPP est utilisée sur le WAN 172.16.1.0/24.

Configurations clés
Partie 1 : routage statique

SiteCentral :

interface Serial1/1
 ip address 192.168.0.1 255.255.255.0
 encapsulation frame-relay
 frame-relay lmi-type ansi
ip route 192.168.2.0 255.255.255.0 192.168.0.2
ip route 192.168.2.0 255.255.255.0 172.16.1.2 50
ip route 192.168.3.0 255.255.255.0 192.168.0.3

Agence1 :

ip route 192.168.1.0 255.255.255.0 192.168.0.1
ip route 192.168.1.0 255.255.255.0 172.16.1.1 50

Agence2 :

ip route 192.168.1.0 255.255.255.0 192.168.0.1

Le suffixe 50 est la distance administrative : la route par le lien PPP n'est utilisée que si la route par le commutateur disparaît de la table.

Partie 2 : sous-interfaces et RIPv2

SiteCentral :

interface Serial1/1
 no ip address
 encapsulation frame-relay
 frame-relay lmi-type ansi
interface Serial1/1.16 point-to-point
 description PVC-avec-agence1
 ip address 192.168.0.1 255.255.255.252
 frame-relay interface-dlci 16
interface Serial1/1.17 point-to-point
 description PVC-avec-agence2
 ip address 192.168.100.1 255.255.255.252
 frame-relay interface-dlci 17
router rip
 version 2
 no auto-summary
 network 192.168.1.0
 network 192.168.0.0
 network 192.168.100.0
 network 172.16.0.0

Les configurations complètes des trois routeurs sont dans configs/TP_FrameRelay_avance_configs.txt.

Démonstration
Partie 1
Où	Commande	À observer
SiteCentral	show frame-relay pvc	DLCI 16 et 17 en ACTIVE
SiteCentral	show interface serial1/0 / serial1/1	PPP, puis Frame Relay avec LMI ANSI
SiteCentral	show ip route	S 192.168.2.0/24 [1/0] via 192.168.0.2 (la route de distance 50 est masquée)
Agence1, Agence2	show ip route	route statique vers 192.168.1.0/24 via 192.168.0.1
C3, puis C1	ping 192.168.1.10	réponses
Agence1	interface Serial1/1 puis shutdown, puis show ip route	la route vers 192.168.1.0/24 bascule sur le lien PPP
SiteCentral	show interface serial1/1, show ip route	état de l'interface et route vers 192.168.2.0/24
C2	ping 192.168.2.10	observer le résultat : c'est le problème que la partie 2 corrige
Partie 2
Où	Commande	À observer
SiteCentral	show ip interface brief, show ip route	sous-interfaces up, routes R (RIP)
C2	ping 192.168.3.10, ping 192.168.2.10	réponses
Agence1	interface Serial1/1 puis shutdown, puis show ip route	route vers 192.168.1.0/24 uniquement par le lien PPP
SiteCentral	show ip interface brief	Serial1/1 up, sous-interface Serial1/1.16 à observer
C2	ping 192.168.2.10, trace 192.168.2.10	chemin de secours par le lien PPP
