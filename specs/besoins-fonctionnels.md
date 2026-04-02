# Besoins fonctionnels — ha-cbx3s-gate-module

## Contexte

Module domotique à concevoir pour piloter et superviser un portail battant équipé d'un boîtier **Somfy Control Box 3S io**. Le module s'intègre à **Home Assistant** via **MQTT over WiFi**.

## Architecture d'intégration

```
Boutons externes ──filaire──> Module domotique <──filaire──> Portail (CBX 3S io)
                                    │
                                WiFi/MQTT
                                    │
                              Home Assistant
```

- Le module est le **point de câblage unique** entre la CBX et le monde extérieur
- Toutes les commandes (MQTT ou boutons physiques externes) passent par le module, qui relaie les contacts secs vers la CBX
- Cela minimise les interventions sur la Control Box : un seul câblage initial sur les bornes 30/31/32
- Le module communique avec Home Assistant via **MQTT** sur le réseau WiFi local
- Le protocole radio io-homecontrol de Somfy n'est **pas** utilisé par le module

---

## MVP — Fonctionnalités essentielles

### F01 — Commander l'ouverture totale du portail

- **Description** : Depuis Home Assistant, déclencher l'ouverture des deux vantaux du portail
- **Déclencheur** : Commande MQTT (ex: `portail/cmd/total`)
- **Action matérielle** : Fermeture brève d'un contact sec sur bornes 30+31 (impulsion relais ~0.5–1s)
- **Prérequis CBX** : P37=0 (mode cycle) ou P37=1 (mode ouverture)
- **Comportement attendu** : Le portail s'ouvre complètement (2 vantaux)

### F02 — Commander l'ouverture piéton du portail

- **Description** : Depuis Home Assistant, déclencher l'ouverture d'un seul vantail
- **Déclencheur** : Commande MQTT (ex: `portail/cmd/pieton`)
- **Action matérielle** : Fermeture brève d'un contact sec sur bornes 32+31 (impulsion relais ~0.5–1s)
- **Prérequis CBX** : P37=0 (mode cycle)
- **Comportement attendu** : Le portail s'ouvre partiellement (1 vantail)

### F03 — Détecter une coupure de courant (230V off)

- **Description** : Le module détecte la perte de l'alimentation secteur et en informe Home Assistant
- **Publication MQTT** : Topic d'état (ex: `portail/status/secteur`) avec valeur `on` / `off`
- **Contrainte** : Le module doit rester alimenté sur batterie de secours (via 24V bornes 19/20) pour pouvoir émettre l'alerte
- **Filtrage** : Micro-coupures <2–5s ignorées (anti-rebond temporel)

### F04 — Configurer le module (WiFi et MQTT)

- **Description** : Permettre la configuration initiale du module (credentials WiFi, paramètres MQTT) sans recompilation
- **Mécanisme** : Le module démarre en mode **Access Point** avec portail captif
  - Activation : automatiquement si aucun WiFi n'est configuré, ou sur **appui long** (5s) d'un bouton poussoir sur le module
  - Le portail captif permet de saisir : SSID, mot de passe WiFi, adresse du broker MQTT, port, credentials MQTT
  - Timeout : l'AP se coupe automatiquement après 5 min sans connexion
- **Stockage** : Les paramètres sont sauvegardés en flash (NVS) et persistent aux redémarrages
- **Sécurité** : L'AP n'est actif que sur action physique ou premier démarrage (pas d'AP permanent)

### F05 — Commandes externes (boutons physiques)

- **Description** : Des boutons poussoirs externes permettent de commander l'ouverture totale et piéton
- **Câblage** : Les boutons sont câblés **en parallèle des relais**, directement sur les bornes CBX (30+31 et 32+31). Un bouton poussoir est un contact sec, exactement comme un relais — le câblage parallèle fonctionne nativement
- **Pas de GPIO nécessaire** : Le circuit est purement électrique, indépendant du MCU
- **Résilience** : Les boutons fonctionnent même si le MCU est hors service (crash, mise à jour, etc.)
- **Intérêt** : Le module reste le point de câblage unique — les borniers du module offrent un point de raccordement pratique pour les boutons, sans nécessiter de câblage supplémentaire sur la CBX
- **Note** : La détection du déclenchement d'une commande externe (F07/F08) repose sur le retour d'état de la CBX (sortie auxiliaire bornes 7/8), pas sur la lecture des boutons

### F06 — Connaître l'état du portail

- **Description** : Home Assistant connaît l'état courant du portail
- **États possibles** :
  - `ferme` — portail fermé
  - `ouvert` — portail ouvert
  - `mouvement` — portail en cours de mouvement
- **Source matérielle** : Sortie auxiliaire bornes 7/8 avec **P15=1** (témoin portail)
  - Contact ouvert = fermé
  - Contact clignotant = mouvement
  - Contact fermé = ouvert
- **Publication MQTT** : Topic d'état (ex: `portail/status/etat`)
- **Si possible** : Distinguer le sens du mouvement (ouverture vs fermeture). Cela nécessite d'analyser les transitions : fermé→mouvement = ouverture, ouvert→mouvement = fermeture

---

## Optionnel — Fonctionnalités souhaitées

### F07 — Notifier le déclenchement d'une ouverture totale

- **Description** : Lorsque l'ouverture totale est déclenchée (par n'importe quelle source : télécommande, clavier, module, etc.), Home Assistant est informé
- **Difficulté** : La CBX 3S io ne fournit pas d'information sur la source de la commande. La détection ne peut se faire qu'indirectement via l'analyse de l'état portail (F06) combinée au fait que le module n'a pas émis la commande lui-même
- **Approche possible** : Si le portail passe en mouvement sans commande du module → notification "ouverture externe détectée". La distinction total/piéton pourrait être déduite du comportement (2 vantaux vs 1)

### F08 — Notifier le déclenchement d'une ouverture piéton

- Même principe que F07, mais pour l'ouverture piéton
- La distinction entre total et piéton depuis la sortie auxiliaire P15=1 reste à étudier (le signal est-il différent ?)

---

## Contraintes techniques identifiées

| Contrainte | Impact |
|-----------|--------|
| Alimentation depuis la CBX (24V bornes 19/20) | Le module doit fonctionner en 24V → régulateur buck vers 3.3V ou 5V |
| Fonctionnement sur batterie de secours | Le module doit continuer à opérer quand le 230V est coupé (F03) |
| WiFi requis pour MQTT | Le MCU doit avoir un module WiFi intégré (ESP32 probable) |
| Commandes par contact sec | Le module a besoin d'au moins 2 relais (ouverture totale + piéton) et 2 entrées contact sec (commandes externes) |
| Lecture état portail (P15=1) | Le module doit lire un contact sec (GPIO avec pull-up) et détecter le clignotement |
| Sécurité électrique | Isolation entre 230V et TBTS si détection secteur sur bornes 1/2 |

---

## Prérequis matériels à vérifier sur site

- [ ] Valeur actuelle de **P15** (doit être 1 pour le retour d'état)
- [ ] Valeur actuelle de **P37** (mode des commandes filaires)
- [ ] Type de batterie installée (9,6V ou 24V)
- [ ] Tensions réelles sur bornes 19/20 (secteur vs batterie, repos vs mouvement)
- [ ] Couverture WiFi au niveau du boîtier de la CBX 3S io
- [ ] Espace disponible dans ou à proximité du boîtier pour le module
