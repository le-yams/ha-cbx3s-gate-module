# Référence technique — Somfy Control Box 3S io

Synthèse de la documentation officielle Somfy pour la Control Box 3S io.
Sources : `5128270bcontrol_box_3s_io-fr-en-tr-ar.pdf`, `aidememoire_cbx_3s_io.pdf`.

## Bornier complet

### Alimentation secteur 230V AC

| Borne | Fonction |
|-------|----------|
| 1     | Phase (L) — entrée secteur |
| 2     | Neutre (N) — entrée secteur |
| 3, 4  | Terre (pour éclairage classe 1) |

### Sortie éclairage de zone 230V

| Borne | Fonction |
|-------|----------|
| 5     | Neutre (N) — sortie éclairage de zone |
| 6     | Phase (L) — sortie éclairage de zone |

- Protection : fusible 5A
- Mode configurable via **P13** (0=inactive, 1=pilotée par télécommande, 2=automatique+pilotée)
- Temporisation configurable via **P14** (valeur × 10s, défaut 60s)
- **Non disponible sur batterie de secours**
- Ne pas confondre avec le feu clignotant (flash) qui est sur les bornes 17/18

### Sortie auxiliaire — Contact sec TBTS

| Borne | Fonction |
|-------|----------|
| 7     | Contact (NO) |
| 8     | Commun |

- Contact sec libre de potentiel, **24V max, 2A max**, TBTS
- La carte ne fournit pas de tension : elle ouvre/ferme un relais interne
- Comportement configurable via **P15** (voir section Paramétrage P15)

### Batterie de secours

| Borne | Fonction |
|-------|----------|
| 9     | Batterie (+) |
| 10    | Batterie (−) |

- Raccordement du module batterie Somfy optionnel
- Codes afficheur : **Hc1** = batterie 9,6V, **Hu1** = batterie 24V

### Moteurs

| Borne | Fonction |
|-------|----------|
| 11    | Moteur 1 (+) |
| 12    | Moteur 1 (−) |
| 13    | Fin de course Moteur 1 (Ixengo uniquement) |
| 14    | Moteur 2 (+) |
| 15    | Moteur 2 (−) |
| 16    | Fin de course Moteur 2 (Ixengo uniquement) |

> Les bornes 13 et 16 ne sont utilisées qu'avec les moteurs Ixengo (fin de course magnétique/mécanique). Inutilisées avec SGS, Elixo, Evolvia, etc.

### Feu clignotant (flash)

| Borne | Fonction |
|-------|----------|
| 17    | Flash (+) |
| 18    | Flash (−) |

- Feu orange de signalisation de mouvement du portail
- Préavis configurable via **P12** (0=sans préavis, 1=préavis 2s avant mouvement)

### Alimentation 24V DC (périphériques)

| Borne | Fonction |
|-------|----------|
| 19    | +24V DC |
| 20    | 0V (GND) |

- Alimente cellules photo, capteurs, accessoires TBTS
- ⚠ **Coupé sur batterie de secours** (mesure sur site : 0V au repos ET en mouvement, batterie fonctionnelle par ailleurs). Les périphériques TBTS branchés sur 19/20 ne sont donc **pas alimentés** pendant une coupure secteur.

### Cellules photoélectriques (sécurité)

| Borne | Fonction |
|-------|----------|
| 21, 23, 24 | Alimentation 24V DC dédiée cellules (mode selon P07) |
| 22          | Entrée contact sec cellule (sécurité gate) |

### Serrure / Sécurité programmable / Test

| Borne | Fonction |
|-------|----------|
| 25    | Sortie serrure (+) |
| 26    | Sortie serrure (−) |
| 27    | Commun — entrée sécurité 2 programmable |
| 28    | Contact — entrée sécurité 2 programmable |
| 29    | Sortie test sécurité |

- Sortie serrure : 24V ou 12V selon **P17** (0=24V impulsionnel, 1=12V impulsionnel)
- Entrée sécurité 2 : configurable via **P09** (mode), **P10** (fonction), **P11** (action)
- Borne 29 : utilisée pour l'auto-test des dispositifs de sécurité

### Commandes filaires (contact sec)

| Borne | Fonction |
|-------|----------|
| 30    | Entrée commande TOTAL / OUVERTURE |
| 31    | Commun (partagé entre 30 et 32) |
| 32    | Entrée commande PIETON / FERMETURE |

- Comportement dépendant du **paramètre P37** :
  - **P37=0** (défaut) : borne 30 = cycle total (2 vantaux), borne 32 = cycle piéton (1 vantail)
  - **P37=1** : borne 30 = ouverture seulement, borne 32 = fermeture seulement

### Antenne radio

| Borne | Fonction |
|-------|----------|
| 33    | Âme |
| 34    | Tresse |

---

## Paramétrage P15 — Sortie auxiliaire (bornes 7/8)

| Valeur P15 | Mode | Description |
|------------|------|-------------|
| 0 | Inactive | Sortie non utilisée |
| 1 | Témoin portail | Éteint = fermé, clignote = mouvement, allumé = ouvert |
| 2 | Bistable temporisé | Active pendant mouvement + temporisation (durée = P16 × 10s) |
| 3 | Impulsionnel | Impulsion au début du mouvement uniquement |
| 4 | Bistable piloté radio | ON/OFF via télécommande mémorisée |
| 5 | Impulsionnel piloté radio | Impulsion à chaque activation radio |
| 6 | Bistable temporisé piloté radio | Activation radio, désactivation auto après P16 × 10s |

### Fonctions des touches (source : notice Somfy §2.4.2)

| Touche       | Action                                                                                   |
|--------------|------------------------------------------------------------------------------------------|
| SET (0,5 s)  | Entrée **et** sortie du menu paramétrage (PXX)                                           |
| SET (2 s)    | Déclenchement de l'auto-apprentissage (depuis H0)                                        |
| SET (7 s)    | Effacement de l'auto-apprentissage et des paramètres (reset)                             |
| + / −        | Navigation dans la liste des paramètres (appui bref = pas à pas, maintenu = rapide) ; en mode valeur : modification de la valeur |
| OK           | Validation de la sélection d'un paramètre ; validation de la valeur ; lancement auto-apprentissage (depuis H1) ; marche forcée |
| PROG (2 s)   | Mémorisation des télécommandes (menu FXX)                                                |
| PROG (7 s)   | Effacement de toutes les télécommandes                                                   |

### Affichage de la valeur d'un paramètre (source : notice §7.2)

- **Fixe** : valeur actuellement sélectionnée pour ce paramètre (lecture).
- **Clignotant** : valeur sélectionnable, en cours de choix (pas encore validée).

### Lecture d'un paramètre (sans modification)

1. Appui **0,5 s sur SET** → l'afficheur affiche **P01**
2. Naviguer avec **+ / −** (appui bref pas à pas, maintenu pour défilement rapide) jusqu'au paramètre souhaité (ex : P15)
3. Appuyer sur **OK** → la valeur actuelle s'affiche **fixe**
4. Ne toucher ni à **+ / −** (ça passerait en modification clignotante), ni à **OK** (revient au nom du paramètre)
5. Sortir du menu par un **appui 0,5 s sur SET**

### Modification d'un paramètre (ex : passer P15 de 6 à 1)

1. Appui **0,5 s sur SET** → P01
2. **+ / −** jusqu'à **P15**
3. **OK** → valeur actuelle affichée fixe
4. **+ / −** → la valeur devient clignotante ; sélectionner la nouvelle valeur (1)
5. **OK** → la valeur devient fixe (validée)
6. Appui **0,5 s sur SET** → sortie du menu paramétrage

> ⚠ Ne pas confondre **SET** (menu paramètres PXX) et **PROG** (menu télécommandes FXX).

---

## Paramétrage P37 — Entrées de commande filaire (bornes 30/32)

| Valeur P37 | Mode | Description |
|------------|------|-------------|
| 0 | Cycle total / piéton | Borne 30 = cycle total, borne 32 = cycle piéton |
| 1 | Ouverture / fermeture | Borne 30 = ouverture seulement, borne 32 = fermeture seulement |

---

## Batterie de secours

- Module batterie Somfy (réf. 2400965)
- Autonomie : ~10 manœuvres complètes
- Deux variantes selon le module :
  - **Batterie 9,6V** → code afficheur **Hc1**
  - **Batterie 24V** → code afficheur **Hu1**
- En mode batterie :
  - Logique de commande + radio io : **opérationnels**
  - Sortie 24V (bornes 19/20) : **coupée** (0V mesuré sur site, au repos comme en mouvement)
  - Cellules photo (21/23/24) : comportement à vérifier si utilisées (probablement alimentées uniquement pendant un cycle)
  - Sortie éclairage 230V (bornes 5/6) : **indisponible**
  - Feu clignotant (bornes 17/18) : **indisponible** (230V)
- **Aucune sortie native ne signale le passage sur batterie**
- L'afficheur de la carte distingue les deux cas (Hc1/Hu1), mais aucune borne ne reporte cette information

---

## Codes afficheur (principaux)

| Code | Désignation |
|------|-------------|
| C1   | Attente de commande (fonctionnement normal) |
| C2   | Ouverture en cours |
| C3   | Attente de refermeture |
| C4   | Fermeture en cours |
| C6   | Détection sur sécurité cellule |
| H0   | Attente de réglage |
| Hc1  | Attente de réglage + batterie 9,6V |
| Hu1  | Attente de réglage + batterie 24V |
| F0–F3 | Modes mémorisation télécommande |

---

## Résumé des paramètres pertinents

| Paramètre | Fonction | Valeurs clés |
|-----------|----------|-------------|
| P01 | Mode de fonctionnement cycle total | 0=séquentiel (défaut), 1=séq.+tempo, 2=semi-auto, 3=auto, 5=homme mort |
| P07 | Entrée sécurité cellules | 0=inactive, 1=active, 2=auto-test, 3=auto-test commutation, 4=bus |
| P12 | Préavis feu orange | 0=sans, 1=2s avant mouvement |
| P13 | Sortie éclairage de zone | 0=inactive, 1=pilotée, 2=auto+pilotée |
| P14 | Temporisation éclairage | 0–60 (×10s), défaut 6 (60s) |
| P15 | Sortie auxiliaire | 0–6 (voir section dédiée) |
| P16 | Temporisation sortie auxiliaire | 0–60 (×10s), défaut 6 (60s) |
| P17 | Sortie serrure | 0=24V impulsionnel, 1=12V impulsionnel |
| P37 | Entrées commande filaire | 0=cycle total/piéton, 1=ouverture/fermeture |
