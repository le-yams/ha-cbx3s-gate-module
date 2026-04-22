# Architecture bloc consolidée — ha-cbx3s-gate-module

Synthèse intégrant les décisions D1 (MCU), D2 (détection secteur), D3 (photo-MOSFET) et D6 (super-cap). Sert de référence pour la phase de conception matérielle.

Le « pourquoi » de chaque décision (D1–D6) et l'analyse des alternatives écartées restent dans [`etude-faisabilite.md`](etude-faisabilite.md).

---

## Vue d'ensemble

```
┌──────────────────────────────┐                      ┌──────────────────┐
│      CBX 3S io (Somfy)       │                      │ Boutons externes │
│                              │                      │       (F05)      │
│  19  20  30  31  32  7   8   │                      └────┬─────────┬───┘
└───┬───┬───┬───┬───┬──┬───┬───┘                           │         │
    │   │   │   │   │  │   │                               │         │
    │   │   ▲   ▲   ▲  │   │       (boutons câblés en parallèle sur  │
    │   │   │   │   │  │   │        bornes 30-31 et 32-31, via le    │
    │   │   │   └───┼──│───│─────── bornier du module comme simple   │
    │   │   │       │  │   │        passerelle) ─────────────────────┘
    │   │   │       │  │   │
    ▼   ▼   │       │  ▼   ▼
 ╔══════════╪═══════╪══════════════════════════════════════════════════╗
 ║          │       │                     MODULE (PCB custom)          ║
 ║          │       │                                                  ║
 ║  ┌──[D1]─┴──[Buck 24→5V]──[SuperCap 3F/5,5V]──[Buck-boost 5→3,3V]──┐║
 ║  │ Schottky        │                                      │        │║
 ║  │                 │                              +3,3V rail       │║
 ║  │                 ▼                                      │        │║
 ║  │       [Pont div 180k/27k + TVS 5V + RC]                │        │║
 ║  │                 │                                      │        │║
 ║  │                 └──────────────────────── ADC ─────────┤        │║
 ║  │                                                        │        │║
 ║  │   ┌─────────────────────────────────────────┐          │        │║
 ║  │   │        XIAO ESP32-C3                    │ ◀── 3,3V ┘        │║
 ║  │   │                                         │                   │║
 ║  │   │  GPIO5 ← ADC secteur (F03)              │                   │║
 ║  │   │  GPIO6 ← État portail (F06)             │                   │║
 ║  │   │  GPIO3 → TLP241A total  (F01)           │                   │║
 ║  │   │  GPIO4 → TLP241A piéton (F02)           │                   │║
 ║  │   │  GPIO9 ← Bouton BOOT config AP (F04)    │                   │║
 ║  │   │                                         │                   │║
 ║  │   │  WiFi ∽∽▶ MQTT ▶ Home Assistant         │                   │║
 ║  │   └──┬──────────┬────────────┬──────────────┘                   │║
 ║  │      │ GPIO3    │ GPIO4      │ GPIO6                            │║
 ║  │      │ +R 470Ω  │ +R 470Ω    │ +pull-up 10kΩ                    │║
 ║  │      ▼          ▼            ▲  +RC anti-rebond                 │║
 ║  │   [LED]       [LED]          │                                  │║
 ║  │  TLP241A    TLP241A          │                                  │║
 ║  │   #1 total  #2 piéton        │                                  │║
 ║  │   (MOSFET)  (MOSFET)         │                                  │║
 ║  └──────┬─────┬────┬─────┬──────┴──────────────────────────────────┘║
 ║         │     │    │     │                                          ║
 ╚═════════│═════│════│═════│══════════════════════════════════════════╝
           ▼     ▼    ▼     ▼
          30    31   32    31     (commandes vers CBX, contacts secs)
          ↑ remontée 7/8 en entrée côté module (flèche du haut sur le schéma)
```

## Blocs fonctionnels

| Bloc | Composants clés | Rôle | F couverts |
|------|----------------|------|-----------|
| **Alimentation principale** | Schottky D1, buck 24→5 V | Convertit 24 V CBX en rail 5 V | Tous (prérequis) |
| **Réserve d'énergie** | Super-cap 3 F / 5,5 V + soft-start | Maintien alim ~15-30 s après coupure 19/20 | F03 last-gasp |
| **Régulation MCU** | Buck-boost TPS63020 5→3,3 V | Extrait énergie super-cap jusqu'à 2 V | Tous (prérequis) |
| **Détection secteur** | Pont 180k/27k + TVS 5 V + RC | Attaque ADC GPIO5 avec image du 24 V | F03 |
| **Commande totale** | TLP241A #1 + R 470 Ω | SSR piloté par GPIO3 vers bornes 30-31 | F01 |
| **Commande piéton** | TLP241A #2 + R 470 Ω | SSR piloté par GPIO4 vers bornes 32-31 | F02 |
| **Lecture état** | Pull-up 10 kΩ + RC anti-rebond | Conditionne 7/8 vers GPIO6 | F06 |
| **Config utilisateur** | Bouton BOOT intégré XIAO | GPIO9 pour activer mode AP | F04 |
| **Connectivité** | WiFi intégré XIAO | MQTT vers broker Home Assistant | F01-F07 |
| **Boutons externes** | 2× contacts secs sur bornier module | Passent en parallèle vers 30/32-31 CBX | F05 |

## Interfaces physiques du module (bornier)

| Bornier module | Nb broches | Câblage |
|----------------|-----------|---------|
| **Entrée alimentation** | 2 | Vers CBX 19 (+24 V) et 20 (GND) |
| **Sortie commande totale** | 2 | Vers CBX 30 et 31 |
| **Sortie commande piéton** | 2 | Vers CBX 32 et 31 (mutualisable avec précédent) |
| **Entrée état portail** | 2 | Vers CBX 7 et 8 |
| **Bouton externe total** | 2 | En parallèle sortie commande totale |
| **Bouton externe piéton** | 2 | En parallèle sortie commande piéton |

Total : 6 borniers 2 points (soit ~12 broches). Envisageable de mutualiser les retours 31 et les GND CBX (20, 8) sur un même plan de masse pour réduire à ~8 points.

## Flux d'information

```
Home Assistant ─(MQTT)─▶ XIAO ─(GPIO3/4)─▶ TLP241A ─▶ CBX (commande)
HA ◀─(MQTT)──── XIAO ◀─(GPIO6)── 7/8 CBX           (état)
HA ◀─(MQTT)──── XIAO ◀─(GPIO5 ADC)── 19/20         (secteur)
Boutons externes ────── direct CBX 30/32-31         (hors MCU, F05)
```

## Budget énergie — fonctionnement nominal

Hypothèses : WiFi associé en modem-sleep (MQTT keep-alive 60 s), publications épisodiques, portail actionné <10 fois/jour.

| Mode | Courant @ 3,3 V | Temps / jour | Énergie |
|------|-----------------|--------------|---------|
| Modem-sleep (WiFi associé, idle) | ~1 mA moy. | ~23 h | ~83 J |
| Pic WiFi publication MQTT | ~150 mA × 200 ms | ~100 fois/j | ~10 J |
| Activation photo-MOSFET (1 s) | +3 mA × 2 | ~10 fois/j | ~0,2 J |
| Lecture P15 en cours de mouvement (~1 min) | ~20 mA (CPU réveillé) | ~10 min/j | ~40 J |
| **Total journalier** | — | — | **~130 J** |

→ Courant moyen ≈ **~0,5 mA à 3,3 V**. Le buck 24→5 V puis buck-boost 5→3,3 V doivent être choisis pour un bon rendement à faible charge (I_q < 50 µA pour le buck-boost si possible).

## Budget énergie — coupure secteur (validation super-cap)

Cf. [`etude-faisabilite.md` §D6](etude-faisabilite.md) : ~4 J worst-case pour last-gasp, cap 3 F fournit ~27 J utiles → marge ×6,8.

## Contraintes d'encombrement estimées

| Composant | Surface approx. | Hauteur |
|-----------|----------------|---------|
| XIAO ESP32-C3 | 21 × 18 mm | 3 mm |
| Super-cap 3 F radial | Ø13 × ×13 mm pad | 25 mm |
| Buck 24→5V (module ou CMS) | 15 × 15 mm | 8 mm |
| Buck-boost 5→3,3V (IC + inductance + caps) | 15 × 15 mm | 3 mm |
| 2× TLP241A | 5 × 5 mm chacun | 3 mm |
| Bornier à vis 6 × 2 pts | 30 × 10 mm | 12 mm |
| Passifs, diodes, TVS, RC | ~10 × 20 mm zone | < 3 mm |

→ PCB estimé **~60 × 50 mm**, hauteur ~25 mm (dominée par le super-cap). Compatible avec un boîtier DIN 3 modules ou un boîtier IP54 standard pour domotique.

## Checklist de cohérence

Vérifications avant de passer à la conception matérielle détaillée :

- [x] Alimentation principale : source 24 V disponible (19/20 mesuré à ~27 V)
- [x] Réserve d'énergie : super-cap dimensionné pour worst-case F03 (×6,8 de marge)
- [x] Rendement alimentation : chaîne Schottky + buck + buck-boost cohérente
- [x] Commandes sorties : TLP241A compatibles 3,3 V GPIO (LED Vf 1,17 V, 5 mA)
- [x] Entrée ADC : pont diviseur 180k/27k produit 3,1 V à 24 V (sous le 3,3 V max ESP32-C3 avec TVS)
- [x] Entrée digitale P15 : pull-up + RC compatible lecture par GPIO
- [x] Allocation GPIO : 5 utilisés sur 8 non-strapping, 3 libres pour évolutions
- [x] Pas de conflit entre alimentation super-cap et détection secteur (D1 isole)
- [x] Boutons externes (F05) indépendants du MCU (câblage direct CBX)
- [x] Connectivité WiFi interne au XIAO, pas d'antenne externe à prévoir (à vérifier couverture sur site)
- [ ] Caractéristiques électriques bornes 30/31/32 CBX compatibles avec TLP241A (V_off < 60 V, I_on < 1 A) → à mesurer sur site
- [ ] Fréquence de clignotement P15=1 compatible avec période de sample GPIO (100 ms prévue) → à mesurer sur site
- [ ] Couverture WiFi au point de montage → à mesurer sur site

Les 3 cases à cocher restantes dépendent toutes d'une session sur site (après passage P15 de 6 à 1).

## Composants non encore figés

| Poste | Candidats | Décision reportée à |
|-------|-----------|---------------------|
| Buck 24→5 V | LMR33630 (IC), LM2596 (module), MP2315 | Phase conception matérielle |
| Buck-boost 5→3,3 V | TPS63020, TPS63900 (plus low-Iq) | Phase conception matérielle |
| Super-cap (ref exacte) | EATON PHV-5R4H305-R, Panasonic EECS5R5H305 | Phase approvisionnement |
| TVS détection secteur | SMAJ5.0A, PESD5V0S1UL | Phase conception matérielle |
| Boîtier | DIN 3 modules, IP54 hobby | Phase intégration |
