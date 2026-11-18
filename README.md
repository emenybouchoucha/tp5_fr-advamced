
# Configuration Avancée du Frame Relay (Hub-and-Spoke)

Atelier **Technologies WAN** (**RSI3**, **ISET Nabeul**, 2025/2026) sous **GNS3** avec routeurs Cisco c7200.

## 🎯 Objectif
Mettre en place un WAN Frame Relay en topologie hub-and-spoke avec un lien de secours PPP (haute disponibilité).
* **Partie 1 :** Interfaces physiques, ARP inverse, routage statique (métrique de secours = 50) et observation des limites.
* **Partie 2 :** Sous-interfaces `point-to-point` par PVC et routage dynamique **RIPv2**.

## 📂 Contenu du Dépôt
* `README.md` : Ce fichier.
* `partie1/` : Projet GNS3 de la Partie 1.
* `partie2/` : Projet GNS3 de la Partie 2.

## 🛠️ Utilisation
1. Ouvrir **GNS3** avec la GNS3 VM démarrée.
2. Importer le projet souhaité dans GNS3.
3. Démarrer tous les équipements (**Start all devices**) et patienter ~2 minutes.
4. Ouvrir les consoles des routeurs et taper `enable` (les configurations sont chargées automatiquement).
