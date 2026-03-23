# Étude de faisabilité — ha-cbx3s-gate-module

Ce document recense les options techniques envisagées pour réaliser les fonctionnalités décrites dans `specs/besoins-fonctionnels.md`. Chaque section correspond à un choix ouvert.

---

## Choix du microcontrôleur

**Contraintes** : WiFi intégré, GPIO suffisants (2 relais + 1 entrée contact sec + 1 entrée détection secteur), ADC si option TBTS, alimentation 3.3V, faible coût.

| Option | Pour | Contre |
|--------|------|--------|
| **ESP32** (3.3V) | WiFi+BT intégrés, écosystème riche (ESPHome, Arduino, MicroPython), GPIO suffisants, ADC intégré, faible coût (~5€) | Consommation WiFi en pointes, ADC non linéaire sur certaines plages |
| STM32 + module WiFi | Meilleur ADC, plus de GPIO, industriel | Plus complexe, coût supérieur, 2 composants |
| Arduino + shield WiFi | 5V natif (simplifie relais) | Encombrant, écosystème WiFi moins mature |

**Tendance** : ESP32 est le candidat naturel. Compatible ESPHome pour une intégration directe Home Assistant/MQTT.

**Statut** : À arbitrer.

---

## Commande du portail (F01, F02)

**Principe** : Fermeture brève (~0.5–1s) d'un contact sec entre bornes 30+31 (total) ou 32+31 (piéton).

**Solution retenue** : 2 relais pilotés par GPIO du MCU.

| Option | Pour | Contre |
|--------|------|--------|
| Module relais 2 canaux (type SRD-05VDC) | Simple, isolé, fiable | Encombrant, consomme ~70mA par relais |
| Relais reed ou SSR | Compact | Moins courant en hobby |
| Optocoupleur + MOSFET | Ultra compact, silencieux | Contact sec = pas de tension à commuter, relais mécanique plus direct |

**Tendance** : Module 2 relais, simple et éprouvé. Attention : si MCU en 3.3V, vérifier la compatibilité du module relais (certains nécessitent 5V sur la commande).

**Statut** : Quasi décidé (module relais), à confirmer avec le choix MCU.

---

## Lecture de l'état portail (F05)

**Source** : Sortie auxiliaire bornes 7/8, avec P15=1 (témoin portail).

**Comportement P15=1** :
- Contact **ouvert** → portail fermé
- Contact **clignotant** → portail en mouvement
- Contact **fermé en continu** → portail ouvert

**Implémentation** :
- GPIO en entrée avec pull-up (interne ou externe 10kΩ vers 3.3V)
- Lecture périodique (ex: toutes les 100ms)
- Algorithme de détection du clignotement par comptage de transitions sur une fenêtre temporelle (ex: 2–3s)

**Question ouverte** : La fréquence de clignotement est-elle documentée ? À mesurer sur site.

**Statut** : Faisable. Nécessite P15=1 et mesure du signal réel.

---

## Détection secteur / batterie (F03)

Deux approches principales. Le module doit rester alimenté via la batterie de secours pour signaler la coupure.

### Approche TBTS — Surveillance du 24V (bornes 19/20)

Avantage : tout en basse tension, simple. Inconvénient : la tension 24V reste présente sur batterie, il faut détecter la *variation* de tension.

#### Option A — Lecture ADC

- Pont diviseur R1=180kΩ, R2=27kΩ → V_ADC ≈ 3.12V à 24V, courant ~0.116mA
- Protections : R série 1kΩ + zener 3.6V ou TVS 5V + RC (10kΩ/100nF)
- Seuil logiciel : ≥23.5V = secteur, <23.0V pendant >3s = batterie
- Hystérésis logiciel ~0.5V + anti-rebond 2–5s
- **+** Ultra simple, flexible | **−** Dépend de la précision ADC, nécessite calibration

#### Option B — Comparateur à hystérésis

- Comparateur rail-to-rail (LMV331, TLV3701, MCP6541) + référence (TL431 à 2.50V)
- Pont diviseur : R1=82kΩ (E12), R2=10kΩ → seuil ~23.5V
- Rfeedback ≈ 1MΩ pour hystérésis ±0.5V
- Sortie TOR directe vers GPIO
- **+** Seuil net, insensible au bruit, pas d'ADC requis | **−** Un CI supplémentaire

#### Option C — Optocoupleur avec seuil zener

- Pont pour ~5–6V à 24V + zener 4.7–5.1V en parallèle LED opto
- Rlim pour 2–3mA, pull-up 10kΩ vers 3.3V côté transistor
- **+** Isolation galvanique, sortie TOR propre | **−** Seuil moins précis

### Approche 230V — Surveillance directe (bornes 1/2)

Avantage : détection franche (présent/absent). Inconvénient : manipulation haute tension, isolation obligatoire.

#### Option 1 — Module détecteur 230V (recommandé)

- Module "AC mains detect" (entrée 85–265VAC, sortie opto isolée 3.3/5V)
- Entrée → bornes 1/2, sortie → GPIO avec pull-up
- Mots-clés achat : "AC 230V mains presence detector optocoupler output"

#### Option 2 — Mini-relais bobine 230VAC

- Bobine sur bornes 1/2, contact NO/COM lu en TBTS
- Ajouter fusible 100–200mA + varistor 275VAC
- **+** Contact sec, pas d'électronique fine | **−** Conso ~0.3–1W

#### Option 3 — Optocoupleur maison

- Pont de diodes 600V + 2×330kΩ 0.5W + opto (PC817) + RC filtrage
- Courant LED opto ~0.5mA, dissipation ~0.2W
- Fusible + varistor 275VAC côté secteur
- **+** Très faible conso | **−** Réservé aux habitués (normes, isolement)

### Conseils transverses

- Toujours prendre le signal sur **1/2** (entrée secteur), pas sur 5/6 (sortie éclairage)
- Filtrage temporel : "secteur perdu" si signal absent >2–5s
- Mesurer les tensions réelles (secteur vs batterie, repos vs mouvement) avant de fixer les seuils
- Protéger par TVS 33V côté 24V si environnement bruité
- Séparer physiquement pistes HT et TBTS

**Statut** : À arbitrer après mesures sur site (tensions réelles 24V secteur vs batterie).

---

## Interfaces matérielles — Récapitulatif

| Fonction | Bornes CBX | Type signal | GPIO MCU |
|----------|-----------|-------------|----------|
| Commande ouverture totale | 30 + 31 | Contact sec (relais) | 1 sortie digitale |
| Commande ouverture piéton | 32 + 31 | Contact sec (relais) | 1 sortie digitale |
| État portail (P15=1) | 7 + 8 | Contact sec (lecture) | 1 entrée digitale + pull-up |
| Alimentation module | 19 + 20 | 24V DC | Via buck → 3.3V |
| Détection secteur | 1/2 ou 19/20 | Selon option | 1 entrée (digitale ou ADC) |

**Total GPIO minimum** : 2 sorties + 2 entrées = 4 GPIO (largement dans les capacités d'un ESP32).

---

## Détection source de commande (F06, F07)

**Problème** : La CBX 3S io ne signale pas *qui* a déclenché le mouvement. La sortie P15=1 indique seulement l'état résultant.

**Approche envisagée** :
1. Le module sait quand *il* a envoyé une commande (il contrôle les relais)
2. Si le portail passe en mouvement sans commande récente du module → "commande externe détectée"
3. La distinction total/piéton pourrait venir du nombre de vantaux en mouvement, mais P15=1 ne semble pas différencier les deux

**Faisabilité** : Partielle. On peut détecter qu'une commande externe a eu lieu, mais pas forcément distinguer total de piéton. À approfondir avec des mesures du signal P15 en conditions réelles.

**Statut** : Optionnel, à étudier après MVP.

---

## Décisions ouvertes

| # | Décision | Dépend de | Priorité |
|---|----------|-----------|----------|
| D1 | Choix MCU | — | Haute |
| D2 | Méthode détection secteur/batterie | Mesures sur site | Haute |
| D3 | Compatibilité module relais avec 3.3V | D1 | Moyenne |
| D4 | Fréquence clignotement P15=1 | Mesure sur site | Moyenne |
| D5 | Distinction total/piéton via P15 | Mesure sur site | Basse |

## Mesures à réaliser sur site

- [ ] Tension bornes 19/20 en fonctionnement normal (secteur)
- [ ] Tension bornes 19/20 en fonctionnement batterie
- [ ] Tension bornes 19/20 pendant un mouvement du portail
- [ ] Signal sur bornes 7/8 (P15=1) : niveaux, fréquence de clignotement, durée
- [ ] Signal P15=1 lors d'une ouverture totale vs piéton (différence ?)
- [ ] Couverture WiFi au niveau du boîtier CBX
- [ ] Valeur actuelle de P15 et P37
- [ ] Type de batterie installée (9,6V ou 24V)
