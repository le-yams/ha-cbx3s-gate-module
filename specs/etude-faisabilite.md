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

**Décision** : **Seeed Studio XIAO ESP32C3** (déjà en stock).

- ESP32-C3 RISC-V single-core 160 MHz, 400 KB SRAM, 4 MB flash
- WiFi 802.11 b/g/n + BLE 5.0
- 11 GPIO (8 libres + 3 strapping), 4 ADC (3 fiables sur ADC1)
- Format ultra-compact : 21 × 17.8 mm
- Deep sleep 44 µA, sortie 3.3V / 500 mA
- Prix ~5 €

**Statut** : ✅ Décidé.

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

### Contexte simplifié par les mesures sur site et D6

Deux éléments rendent F03 quasi-trivial :

1. **Mesure sur site** : les bornes 19/20 passent de ~27 V à **0 V** dès la bascule CBX sur batterie (pas de tension intermédiaire). Le signal à détecter est donc **binaire** (24 V présent ou absent), pas analogique avec seuil fin.
2. **Décision D6** (super-cap 3 F / 5,5 V) : le module dispose de ~15-30 s d'autonomie après la chute du 24 V → largement de quoi détecter, filtrer les micro-coupures (spec F03 : 2-5 s), publier MQTT, se déconnecter proprement.

Conséquence : **la surveillance directe de 19/20 suffit**. L'ancienne approche "surveillance directe du 230 V sur bornes 1/2" devient inutile (aucun gain fonctionnel pour un surcoût en isolation HT).

### Architecture retenue

```
24V (19/20) ──[D1]──┬──[buck 24→5V]──[super-cap 3F]──[buck-boost 5→3,3V]── ESP32-C3
                    │
                    └──[pont diviseur + protections]── GPIO (détection)
```

- **D1 (diode Schottky ou MOSFET idéal)** : empêche le super-cap de rétro-alimenter les bornes 19/20 et de fausser la détection. Chute de 0,2-0,4 V acceptable côté buck.
- **Point de mesure** : en amont de D1 (côté 19/20) pour voir la vraie tension CBX, pas celle maintenue par le super-cap.

### Deux variantes d'implémentation

Avec un signal binaire, la précision d'ADC n'est plus critique. Deux options viables :

#### Option A — Lecture ADC sur pont diviseur (préférée)

- Pont diviseur R1 = 180 kΩ, R2 = 27 kΩ → V_ADC ≈ 3,12 V à 24 V, courant ~0,12 mA
- Protection : TVS 5V (ou zener 3,6 V) + RC (10 kΩ / 100 nF) pour lisser les transitoires
- Seuil logiciel large : ≥ 10 V = secteur, < 5 V = coupure (marge 5 V autour d'un signal qui transite 27 V ↔ 0 V)
- Anti-rebond logiciel : confirmer l'état sur 2-5 s (exigence spec F03) avant publication
- **+** Un seul composant réactif (ADC interne), flexible, diagnostic facile (valeur lue en mV)
- **−** Utilise 1 pin ADC (GPIO5 déjà prévu)

#### Option B — Détection TOR par optocoupleur

- 19/20 via résistance de limitation (R ~= 10 kΩ, ~2,3 mA dans la LED opto) vers PC817 ou équivalent
- Sortie transistor avec pull-up 10 kΩ vers 3,3 V
- GPIO lit niveau haut (24 V absent) ou bas (24 V présent)
- **+** Isolation galvanique, signal propre sans traitement, insensible au bruit
- **−** Un composant de plus, pas de valeur analogique pour diagnostic

**Choix de référence : Option A (ADC)**, pour la simplicité mécanique (un pont diviseur + un TVS) et la visibilité diagnostic. L'isolation galvanique apportée par l'option B n'est pas critique ici : tout est en TBTS sur le même rail GND que 19/20.

### Rejet de l'approche 230V (bornes 1/2)

L'approche "détection directe du secteur 230 V" était motivée par le fait que 19/20 restait supposément à 24 V sur batterie. La mesure sur site infirme cette hypothèse → l'approche 230 V n'a plus d'intérêt :

- Mêmes informations (secteur présent ou non) disponibles en TBTS sur 19/20
- Évite la manipulation de 230 V, les contraintes d'isolation, le fusible/varistor
- Supprime un câble supplémentaire vers les bornes 1/2

Conservé uniquement comme alternative documentaire si jamais le comportement de la CBX change (ex : CBX avec 19/20 maintenu partiel sur batterie) : module "AC mains detect" sur 1/2, sortie opto vers GPIO.

### Séquence de détection côté firmware

1. ADC échantillonné périodiquement (ex : 10 Hz) sur GPIO5
2. État instantané : `secteur` si V > 10 V, `coupure` si V < 5 V, zone morte entre (ignorée)
3. Filtrage : changement d'état confirmé uniquement si stable sur 2-5 s (anti-rebond micro-coupures)
4. À la transition `secteur → coupure` confirmée :
   - Publier MQTT `portail/status/secteur = off` (retained, QoS 1)
   - Publier état portail courant (snapshot avant extinction)
   - Déconnexion MQTT et WiFi propres
   - Deep sleep → le super-cap se vide → extinction
5. Au démarrage (retour secteur) : publier `portail/status/secteur = on` + état portail courant

### Conseils pratiques

- TVS 33 V côté 19/20 si environnement électriquement bruité (proximité moteurs, flash)
- Condensateur de découplage 100 nF au plus près de l'ADC
- Vérifier en prototype que la chute de tension sur 19/20 est franche (pas de plateau intermédiaire au moment de la bascule batterie)
- Pas de séparation HT/TBTS à gérer puisqu'on reste entièrement en TBTS

**Statut** : Option A (ADC sur pont diviseur 180 k / 27 k) retenue comme référence → clôt **D2**. Seuils et timings à affiner en prototype.

---

## Source d'énergie locale du module (D6)

### Contexte

Les bornes 19/20 passent à **0 V** dès que la CBX bascule sur batterie de secours (mesure sur site confirmée). Sans source d'énergie locale, le module s'éteint brutalement à chaque coupure secteur. Pour réaliser F03 (détection coupure) de façon fiable et publier une notification MQTT explicite, il faut une réserve d'énergie suffisante pour tenir la séquence "last-gasp" avant extinction.

### Options étudiées

| Option | Autonomie coupure | Couvre quoi ? | Complexité | Verdict |
|--------|-------------------|---------------|------------|---------|
| Aucune (module meurt) | 0 s | F03 inférée par timeout MQTT (ambigu) | Nulle | Insuffisant pour F03 |
| **Super-cap 3 F / 5,5 V** | ~15-30 s utiles | F03 explicite + filtrage micro-coupures + disconnect MQTT propre | Faible (un cap + buck-boost) | ✅ **Retenu** |
| LiPo + TP4056 | Heures | F01/F02/F06/F07/F08 aussi pendant coupure | Moyenne (charge, protection, vieillissement) | Surdimensionné : la box internet est typiquement hors-ligne en même temps |

**Décision** : super-condensateur **3 F / 5,5 V**, à valider en prototype.

### Budget énergie à couvrir (worst-case F03)

| Étape | Durée worst-case | Courant @ 3,3 V |
|-------|-------------------|------------------|
| Détection + filtrage anti-rebond (spec F03 : 2-5 s) | 5 s | 40-80 mA (modem-sleep WiFi) |
| Publish MQTT retained `secteur=off` | 2-3 s (retry TCP) | 150 mA pointe |
| Publish état final portail + LWT explicite | 1 s | 150 mA |
| Déconnexion MQTT + WiFi propre | 1 s | 100 mA |
| Transition deep sleep | <10 ms | — |

**Total worst-case** : ~10 s à 100 mA moyen sur 3,3 V → ~3,3 J côté rail 3,3 V → **~4 J** à fournir côté super-cap (rendement convertisseur ~85 %).

### Formule de dimensionnement

$$E_{utile} = \tfrac{1}{2} \cdot C \cdot (V_{start}^2 - V_{cutoff}^2) \cdot \eta_{conv}$$

Avec $V_{start}$ = 5,0 V (marge sous 5,5 V), $V_{cutoff}$ = 2,0 V (seuil bas buck-boost), $\eta_{conv}$ ≈ 0,85 :

$$E_{utile} \approx 8{,}9 \cdot C \text{ (J, avec C en F)}$$

| Capacité | Énergie utile | Autonomie @ 100 mA/3,3V moy. | Marge worst-case |
|----------|---------------|------------------------------|------------------|
| 0,5 F | ~4,5 J | ~14 s | ×1,1 (limite) |
| 1 F | ~9 J | ~27 s | ×2,3 (minimum viable) |
| **3 F** | **~27 J** | **~80 s** | **×6,8 (sweet spot)** |
| 5 F | ~45 J | ~135 s | ×11 (marge confortable) |
| 10 F | ~89 J | ~270 s | ×22 (surdimensionné) |

### Topologie recommandée

```
24V (19/20) ──[buck 24→5V]──┬── super-cap 3F/5,5V ──[buck-boost 5→3,3V]── ESP32-C3
                            │
                  [R soft-start 10-47Ω + diode idéale/MOSFET PFET]
```

- **Super-cap sur rail 5 V** : cap unique 5,5 V, pas de stack à équilibrer
- **Buck-boost en aval** (ex : TPS63020, TPS63900) : extrait ~90 % de l'énergie jusqu'à 2 V en entrée
- **Soft-start** : résistance + MOSFET idéal qui bypasse une fois chargé, évite l'inrush à la mise sous tension
- Alternative LDO 3,3V possible mais n'exploite que ~50 % de l'énergie → nécessiterait ~2× plus de capacité

### Fonctionnalités couvertes par cette option

| F | Secteur | Coupure avec super-cap 3F | Coupure sans (module mort) |
|---|---------|----------------------------|----------------------------|
| F01/F02 commande MQTT | ✅ | ❌ après ~15-30 s | ❌ dès coupure |
| **F03 détection coupure** | — | ✅ **notification explicite + filtrage** | ⚠ inférée par timeout MQTT |
| F04 config AP | ✅ | n/a | n/a |
| F05 boutons externes | ✅ | ✅ (câblés direct CBX) | ✅ |
| F06 état portail | ✅ | ❌ après ~15-30 s | ❌ dès coupure |
| F07/F08 notif. sources ext. | ✅ partiel | ❌ pendant coupure | ❌ pendant coupure |

Autrement dit, le super-cap ne prolonge pas la disponibilité du module pendant une coupure : il permet uniquement une **notification propre** avant extinction, et le **filtrage des micro-coupures** qui sinon provoqueraient des reboot répétés.

### Limitations acceptées

1. **Trou d'observation pendant la coupure** : si le portail est manipulé (télécommande radio, bouton filaire) pendant la coupure, HA ne le saura jamais. Au retour secteur, le module reboote et publie l'état *courant* lu sur 7/8, sans historique.
2. **F01/F02/F06/F07/F08 indisponibles pendant la coupure prolongée** (au-delà des ~15-30 s de super-cap).
3. **Pas de resynchronisation "à la volée"** : si la coupure dure > autonomie super-cap, le module crashe brutalement (à la fin).

Ces limitations sont acceptées car, pendant une coupure secteur, la box internet et le routeur WiFi sont typiquement eux aussi hors-ligne — même avec un LiPo, le module n'aurait personne à qui parler en MQTT. Le "last-gasp" juste avant extinction est donc la meilleure valeur accessible.

### Contraintes pratiques

- **ESR** : privilégier < 200 mΩ. Les pics WiFi (~300-400 mA) génèrent sinon une chute de tension gênante.
- **Fuite** : 5-15 µA typique pour 3 F / 5,5 V → absorbée par le buck en régime nominal, invisible.
- **Temps de charge initial** : à ~50 mA via soft-start → $3 \cdot 5 / 0{,}05$ ≈ **5 min** à la première mise sous tension.
- **Durée de vie** : calendaire 10-15 ans à 25 °C, cyclage > 500 000 cycles → largement supérieur à la durée du projet.
- **Encombrement** : radial Ø12-14 mm × H25 mm typique → à prévoir dans le dessin du boîtier.

### Composants candidats

| Référence | Capa / V | ESR | Format | Prix ind. |
|-----------|----------|-----|--------|-----------|
| **EATON PHV-5R4H305-R** | 3 F / 5,4 V | 85 mΩ | Radial Ø12,5×25 mm | ~3 € |
| **Panasonic EECS5R5H305** | 3 F / 5,5 V | 100 mΩ | Radial Ø13,5×25 mm | ~4 € |
| Vinatech VEC5R5107QG | 10 F / 5,5 V | 75 mΩ | Radial Ø21×45 mm | ~7 € |
| Maxwell BCAP0003 (stack 2×2,7 V) | 3 F / 5,4 V | < 100 mΩ | Plus compact, équilibrage requis | ~5 € |

Buck-boost candidat : **TPS63020** (0,5-5,5 V in, 1,2-5,5 V out, 96 % de rendement pic) ou **TPS63900** (ultra-low Iq pour extraire un maximum d'énergie).

### Protocole de validation en prototype

1. Charger le cap avec 24 V présent → mesurer temps de charge effectif
2. Couper brutalement le 24 V → mesurer le temps réel avant brownout de l'ESP32-C3
3. Instrumenter le firmware : horodater chaque étape (détection, publish, MQTT ACK, disconnect)
4. Vérifier que le worst-case (WiFi déconnecté, reconnect, publish, disconnect) tient dans la fenêtre
5. Mesurer à vide (modem-sleep) vs en activité WiFi pour caler l'estimation
6. Si 3 F trop juste → passer à 5 F (même footprint radial, sans redesign) ; si largement excédentaire → redescendre à 1 F

**Statut** : Choix de référence **3 F / 5,5 V**. À valider / ajuster au prototypage.

---

## Interfaces matérielles — Récapitulatif

| Fonction | Bornes CBX | Type signal | GPIO MCU |
|----------|-----------|-------------|----------|
| Commande ouverture totale | 30 + 31 | Contact sec (relais) | 1 sortie digitale |
| Commande ouverture piéton | 32 + 31 | Contact sec (relais) | 1 sortie digitale |
| État portail (P15=1) | 7 + 8 | Contact sec (lecture) | 1 entrée digitale + pull-up |
| Alimentation module | 19 + 20 | 24V DC (secteur uniquement — 0V sur batterie) | Buck 24→5V + super-cap 3F/5,5V + buck-boost 5→3,3V (voir D6) |
| Détection secteur | 1/2 ou 19/20 | Selon option | 1 entrée (digitale ou ADC) |
| Boutons externes total/piéton | 30/32 + 31 | Contact sec | Aucun (câblage direct) |
| Bouton config AP (F04) | — | Bouton poussoir | Bouton BOOT (GPIO9) |

### Câblage des boutons externes (F05)

Les boutons physiques externes sont câblés **en parallèle des relais**, directement sur les bornes CBX :

```
Borne 31 (commun) ──┬── Relais total NO  ──┬── Borne 30
                     └── Bouton total     ──┘

Borne 31 (commun) ──┬── Relais piéton NO ──┬── Borne 32
                     └── Bouton piéton    ──┘
```

Avantages :
- Pas de GPIO consommé (un bouton poussoir est un contact sec, comme le relais)
- Fonctionne même si le MCU est hors service (circuit purement électrique)
- La détection de commande externe (F07/F08) repose sur l'analyse du retour P15, pas sur la lecture des boutons

### Allocation GPIO — XIAO ESP32C3

| GPIO | Pin XIAO | Affectation | Fonction |
|------|----------|-------------|----------|
| GPIO3 | D1 | Relais total | F01 — sortie digitale |
| GPIO4 | D2 | Relais piéton | F02 — sortie digitale |
| GPIO5 | D3/A3 | Détection secteur | F03 — entrée ADC ou digitale |
| GPIO6 | D4 | État portail | F06 — entrée digitale + pull-up |
| GPIO9 | D9 | Bouton config AP | F04 — bouton BOOT réutilisé |
| GPIO7 | D5 | *Libre* | |
| GPIO21 | D6 | *Libre* | |
| GPIO20 | D7 | *Libre* | |
| GPIO10 | D10 | *Libre* | |
| GPIO2 | D0 | *Libre (strapping)* | |
| GPIO8 | D8 | *Libre (strapping)* | |

**Total : 5 GPIO utilisés, 4 libres sans contrainte, 2 libres avec précautions (strapping).**

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
| D1 | ~~Choix MCU~~ → **XIAO ESP32C3** | — | ✅ Décidé |
| D2 | ~~Méthode détection secteur/batterie~~ → **ADC sur pont diviseur 180 k / 27 k (19/20)** | ✅ Mesures 19/20 + D6 | ✅ Décidé |
| D3 | Compatibilité module relais avec 3.3V | ✅ D1 | Moyenne |
| D4 | Fréquence clignotement P15=1 | Mesure sur site | Moyenne |
| D5 | Distinction total/piéton via P15 | Mesure sur site | Basse |
| D6 | Source d'énergie locale → **super-cap 3 F / 5,5 V** (choix de référence, à valider au prototype) | ✅ Mesure 19/20 | ✅ Décidé (référence) |

## Mesures à réaliser sur site

- [x] Tension bornes 19/20 en fonctionnement normal (secteur) — **27 à 27,5 V** au repos, **26,2 à 27,5 V** pendant un mouvement (variation selon sollicitation vérins)
- [x] Tension bornes 19/20 en fonctionnement batterie — **0 V** (sortie coupée par la CBX)
- [x] Tension bornes 19/20 pendant un mouvement du portail (secteur) — voir ci-dessus
- [ ] Signal sur bornes 7/8 (P15=1) : niveaux, fréquence de clignotement, durée — à faire après passage P15 de 6 à 1
- [ ] Signal P15=1 lors d'une ouverture totale vs piéton (différence ?) — à faire après passage P15=1
- [ ] Couverture WiFi au niveau du boîtier CBX
- [x] Valeur actuelle de P15 et P37 — **P15=6** (bistable temporisé radio, à changer en 1), **P37=0** (cycle total/piéton, OK)
- [ ] Type de batterie installée (9,6V ou 24V) — code Hc1 ou Hu1 à relever lors du prochain passage sur batterie
- [ ] Prototype super-cap 3 F : mesurer temps de charge, autonomie réelle en coupure brutale, calibrer filtrage micro-coupures (voir D6)
