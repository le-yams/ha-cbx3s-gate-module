# Mesures sur site — Control Box 3S io

Document à imprimer pour effectuer les relevés et vérifications sur le boîtier CBX 3S io.

## Matériel nécessaire

- **Multimètre** (Vdc, mA, Ω, continuité)
- **Chronomètre** / téléphone (avec mode **slow-motion vidéo** pour fréquence de clignotement)
- **Résistances** 1 kΩ et 10 kΩ (pour mesure de courant par shunt)
- **Téléphone** avec application **WiFi Analyzer** (ou équivalent : RSSI, scan SSID)
- **Fils de test** avec pinces crocodiles ou dominos
- **Stylo**

## Statut des mesures

| # | Section | Statut | Note |
|---|---------|:------:|------|
| 1 | Paramètres CBX (P01, P15, P37…) | ✅ partiel | P15=6, P37=0, P01=1, P13=2, P16=6 relevés. P07 non relevé (facultatif). |
| 2 | Tensions 19/20 | ✅ secteur, ⏳ batterie | 27-27,5 V secteur OK. Code Hc1/Hu1 reste à relever. |
| 3 | Caractérisation sortie 7/8 (P15=1) | ⏳ à faire | **Nécessite P15=1** (cf. section 4 pour modification préalable) |
| 4 | Modification P15 (6 → 1) | ⏳ à faire | **À faire en premier** lors de la prochaine session |
| 5 | Caractéristiques bornes 30/31/32 | ⏳ à faire | Validation choix TLP241A (D3) |
| 6 | Couverture WiFi au point de montage | ⏳ à faire | Check faisabilité WiFi depuis l'emplacement prévu |

## Ordre recommandé pour la prochaine session

1. **Section 4** — Modifier P15 (6 → 1) **en premier**
2. **Section 2 (résiduel)** — Relever code Hc1/Hu1 (couper disjoncteur)
3. **Section 3** — Caractériser signal 7/8 avec P15=1 (ouverture totale + piéton)
4. **Section 5** — Mesurer bornes 30/31/32
5. **Section 6** — Test couverture WiFi

---

## 1. Lire les paramètres de la CBX

### Procédure de lecture (sans modification)

Source : notice Somfy §2.4.2 et §7.1.

1. **Appui 0,5 s sur SET** → l'écran affiche **P01** (entrée dans le menu paramétrage)
2. Naviguer avec **+** / **−** jusqu'au paramètre souhaité
   - appui bref = défilement paramètre par paramètre
   - appui maintenu = défilement rapide
3. Appuyer sur **OK** → la valeur actuelle s'affiche **fixe** → c'est la valeur à relever
4. **Ne toucher ni à + / − ni à OK** (sinon on entre en modification : la valeur passerait en clignotant)
5. Pour sortir : **appui 0,5 s sur SET** → retour à l'écran normal (C1)

> ⚠ **SET 0,5 s** ouvre/ferme le menu paramètres (PXX).
> **SET 2 s** lance l'auto-apprentissage (depuis H0) — à ne pas utiliser.
> **SET 7 s** efface tous les paramètres — à ne pas utiliser.
> **PROG 2 s** ouvre le menu de mémorisation des télécommandes (FXX) — à ne pas utiliser ici.

### Lecture de l'affichage (source : notice §7.2)

- **Fixe** = valeur actuellement sélectionnée (ce qu'on veut relever)
- **Clignotant** = valeur sélectionnable / en cours de modification (pas encore validée)

### Paramètres à relever

| Paramètre | Fonction                                  | Valeur relevée | Valeur souhaitée          |
|-----------|-------------------------------------------|:--------------:|:-------------------------:|
| **P15**   | Mode sortie auxiliaire (bornes 7/8)       |     **6** (à changer en 1) | **1** (témoin portail)    |
| **P37**   | Mode commandes filaires (bornes 30/32)    |     **0** ✅    | **0** (cycle total/piéton)|
| P01       | Mode de fonctionnement cycle total        |     **1**       | — (noter pour info)       |
| P07       | Entrée sécurité cellules                  |    ________    | — (noter pour info)       |
| P13       | Mode sortie éclairage de zone             |     **2**       | — (noter pour info)       |
| P16       | Temporisation sortie auxiliaire           |     **6** (60s) | — (noter pour info)       |

> **P15** et **P37** sont critiques pour le projet. Les autres sont à relever pour information.
>
> ⚠ **P15 = 6** (bistable temporisé radio) au relevé initial → à **modifier en 1** (voir section 5) pour permettre la lecture d'état portail via 7/8 (F06).

### Rappel des valeurs P15

| Valeur  | Mode                                                                  |
|---------|-----------------------------------------------------------------------|
| 0       | Inactive                                                              |
| **1**   | **Témoin portail** (éteint=fermé, clignote=mouvement, allumé=ouvert)  |
| 2       | Bistable temporisé                                                    |
| 3       | Impulsionnel                                                          |
| 4       | Bistable piloté radio                                                 |
| 5       | Impulsionnel piloté radio                                             |
| 6       | Bistable temporisé piloté radio                                       |

### Rappel des valeurs P37

| Valeur  | Mode                                                           |
|---------|----------------------------------------------------------------|
| **0**   | **Borne 30 = cycle total, borne 32 = cycle piéton**            |
| 1       | Borne 30 = ouverture seulement, borne 32 = fermeture seulement |

---

## 2. Mesures de tension sur bornes 19/20 (24V DC)

Multimètre en **DC**, pointes sur borne 19 (+) et borne 20 (−/GND).

| Condition                                       | Tension relevée |
|-------------------------------------------------|:---------------:|
| Portail à l'arrêt, alimentation secteur         | **27 à 27,5 V** ✅ |
| Portail en mouvement (ouverture), secteur       | **26,2 à 27,5 V** ✅ (varie selon sollicitation vérins) |
| Portail en mouvement (fermeture), secteur       | **26,2 à 27,5 V** ✅ (idem) |
| Portail à l'arrêt, alimentation batterie (*)    | **0 V** ✅ (sortie coupée par la CBX) |
| Portail en mouvement, alimentation batterie (*) | **0 V** ✅ (confirmé, reste à 0 V) |

(*) Pour passer sur batterie : couper le disjoncteur dédié au portail. Vérifier que la CBX reste alimentée (afficheur allumé, code Hc1 ou Hu1).


### Identification du type de batterie

Quand le 230V est coupé, l'afficheur indique :
- **Hc1** → batterie 9,6V
- **Hu1** → batterie 24V
- Pas de batterie installée si l'afficheur s'éteint

Code affiché sur batterie : __________

---

## 3. Caractérisation de la sortie auxiliaire (bornes 7/8)

⚠ **Prérequis** : P15 doit être à **1**. Si ce n'est pas le cas, le modifier au préalable (voir section 5).

**Objectif** : clôturer D4 (fréquence de clignotement) et D5 (distinction totale vs piéton) — cf. `specs/etude-faisabilite.md`.

### 3.1 Préparation

Monter un circuit de test pour visualiser et mesurer le contact 7/8 :

```
   +24V (19) ──┬── R pull-up 10 kΩ ──┬── Borne 7
               │                      │
               │                  (point de mesure)
               │                      │
   GND  (20) ──┴──────────────────────┴── Borne 8
```

- Multimètre Vdc sur le point de mesure : lit ~24 V quand contact ouvert, ~0 V quand contact fermé
- Alternative : LED + R série (1 kΩ) entre le point de mesure et GND pour visualiser directement
- Pour mesurer la fréquence de clignotement : filmer la LED en **slow-motion** avec le téléphone (120 ou 240 fps) → compter les frames par période

### 3.2 Test portail fermé (CBX affiche C1)

| Mesure | Relevé |
|--------|--------|
| Tension sur point de mesure | ______ V |
| État du contact 7/8 (ouvert/fermé) | ______ |

Attendu selon doc : contact **ouvert** → tension **~24 V** (pull-up actif).

### 3.3 Test portail en mouvement (ouverture totale)

Déclencher une ouverture totale (télécommande ou bouton filaire), observer/filmer pendant ~10 s :

| Mesure | Relevé |
|--------|--------|
| Le contact clignote-t-il ? (oui/non) | ______ |
| Durée du clignotement total (s) | ______ s |
| Nombre de cycles ON/OFF comptés | ______ |
| Fréquence calculée (cycles / durée) | ______ Hz |
| Durée ON d'un cycle (approx.) | ______ ms |
| Durée OFF d'un cycle (approx.) | ______ ms |
| Rapport cyclique (ON / période) | ______ % |

> **Mesure fiable** : filmer en slow-motion puis compter les frames ON vs OFF entre 2 fronts montants, en sachant le FPS de l'enregistrement.

> Si la fréquence est inférieure à ~2 Hz, comptage visuel direct fonctionne (chronomètre + compteur).

### 3.4 Test portail ouvert (position ouverte stable)

| Mesure | Relevé |
|--------|--------|
| Tension sur point de mesure | ______ V |
| État du contact 7/8 (ouvert/fermé) | ______ |

Attendu : contact **fermé** → tension **~0 V**.

### 3.5 Test portail en mouvement (ouverture piéton)

Même protocole que 3.3 mais avec une commande piéton :

| Mesure | Relevé |
|--------|--------|
| Le contact clignote-t-il ? (oui/non) | ______ |
| Fréquence calculée | ______ Hz |
| Rapport cyclique | ______ % |

### 3.6 Comparaison totale vs piéton (D5)

| Aspect | Totale | Piéton | Différence exploitable ? |
|--------|:------:|:------:|:------------------------:|
| Fréquence clignotement | ______ Hz | ______ Hz | oui / non |
| Rapport cyclique | ______ % | ______ % | oui / non |
| Durée totale du mouvement | ______ s | ______ s | oui / non |

> Si une différence est mesurable et stable, F07/F08 (distinction total/piéton) devient faisable. Sinon, on détecte seulement "mouvement externe" sans pouvoir dire lequel.

---

## 4. Modification de P15 (si nécessaire)

**Uniquement si P15 n'est pas à 1** et que tu souhaites le modifier sur place.

Source : notice Somfy §2.4.2 et §7.1.

1. **Appui 0,5 s sur SET** → l'écran affiche **P01**
2. Naviguer avec **+** / **−** jusqu'à **P15**
3. Appuyer sur **OK** → la valeur actuelle s'affiche fixe
4. Appuyer sur **+** / **−** → la valeur passe en clignotant ; sélectionner **1**
5. Appuyer sur **OK** pour valider → la valeur redevient fixe (nouvelle valeur enregistrée)
6. **Appui 0,5 s sur SET** pour sortir du menu paramétrage

> Après modification de P15, refaire un cycle d'ouverture/fermeture pour vérifier le comportement de la sortie 7/8.

---

## 5. Caractéristiques électriques des bornes 30/31/32

**Objectif** : valider que les contacts secs de la CBX sur les bornes de commande sont compatibles avec le photo-MOSFET **TLP241A** retenu pour D3 (V_off ≤ 60 V, I_on ≤ 1 A).

⚠ **Prérequis** : débrancher tout câble actuellement connecté sur 30, 31, 32 (télécommande filaire, interphone, etc.) pour mesurer la CBX seule. À noter : déclencher un contact sur ces bornes **commande un mouvement du portail** — se placer en zone sûre, portail dégagé.

### 5.1 Tension en circuit ouvert (mesure non-invasive)

Multimètre en **Vdc**, gamme auto ou 50 V, CBX à l'arrêt (affiche C1).

| Mesure | Relevé | Interprétation |
|--------|--------|----------------|
| V entre borne **30** et borne **31** | ______ V | Tension de détection côté total |
| V entre borne **32** et borne **31** | ______ V | Tension de détection côté piéton |
| Polarité : la pointe **(+)** du multimètre était sur quelle borne ? (30 ou 31) | ______ | Indique le sens du pull-up interne CBX |

> Tension attendue : typiquement **5-24 V** (pull-up interne CBX vers un rail de détection).
> Si V > 60 V → incompatible TLP241A → bascule sur photo-MOSFET 200 V ou SSR dédié.

### 5.2 Courant en circuit fermé (mesure par shunt de résistance connue)

**Avertissement** : cette mesure déclenche un cycle du portail. Prévenir l'entourage, dégager la zone, prévoir d'annuler le cycle si besoin (via télécommande).

Placer une résistance **R_shunt = 1 kΩ** entre borne 30 et borne 31 pendant ~0,5 s, mesurer la tension V_R aux bornes de la résistance pendant ce temps (avec le multimètre branché en parallèle sur R, en Vdc).

| Mesure | Relevé | Calcul |
|--------|--------|--------|
| V_R avec R_shunt = 1 kΩ sur 30-31 | ______ V | I_30 = V_R / 1000 = ______ mA |
| V_R avec R_shunt = 1 kΩ sur 32-31 | ______ V | I_32 = V_R / 1000 = ______ mA |

> Si V_R ≈ V_oc (mesurée en 5.1), la résistance 1 kΩ ne tire pas assez de courant pour activer → refaire avec **R_shunt = 100 Ω** (attention, cela triggera plus probablement le cycle CBX).

### 5.3 Estimation du courant en fermeture franche (si déclenchement OK)

Pour estimer le courant qui traverserait le photo-MOSFET à l'état ON (R_on ~ 0,6 Ω) :

$$I_{ON} \approx \frac{V_{oc}}{R_{source}}$$

où $R_{source} = \frac{V_{oc} - V_R}{V_R / R_{shunt}}$.

| Calcul | Valeur |
|--------|--------|
| R_source côté 30-31 | ______ Ω |
| R_source côté 32-31 | ______ Ω |
| I_ON estimé (≈ V_oc / R_source) | ______ mA |

> Pour validation TLP241A : I_ON doit être **< 1000 mA**. Typique : quelques dizaines de mA.

### 5.4 Verdict

- [ ] V_oc < 60 V ? (OK TLP241A)
- [ ] I_ON < 1 A ? (OK TLP241A)
- [ ] Comportement symétrique entre 30 et 32 ? (si non, alerte de câblage)

Si un des critères n'est pas rempli → voir alternatives (AQY211EH, AQY212EH 200 V, ou SSR mécanique à coil 3 V).

---

## 6. Couverture WiFi au point de montage

**Objectif** : vérifier que le module pourra établir une connexion WiFi stable depuis l'emplacement prévu pour son boîtier (à proximité de la CBX).

### 6.1 Préparation

- Positionner le téléphone **à l'emplacement prévu du module**, pas à l'entrée du garage ou à distance
- Application recommandée : **WiFi Analyzer** (Android/iOS) ou équivalent
- Identifier le **SSID** de la box internet / point d'accès cible

### 6.2 Mesure RSSI

| Mesure | Relevé | Interprétation |
|--------|--------|----------------|
| SSID cible | ______ | — |
| RSSI au point de montage | ______ dBm | Voir grille ci-dessous |
| Canal WiFi utilisé | ______ | 1-13 (2,4 GHz) |
| Nombre d'autres réseaux sur le même canal | ______ | Congestion |

### Grille d'interprétation RSSI

| RSSI | Qualité | Verdict |
|------|---------|---------|
| > −60 dBm | Excellente | ✅ MQTT stable sans souci |
| −60 à −70 dBm | Bonne | ✅ OK |
| −70 à −80 dBm | Acceptable | ⚠ Reconnexions ponctuelles possibles, OK avec retry MQTT |
| −80 à −85 dBm | Marginale | ⚠ Envisager antenne externe sur XIAO ou répéteur WiFi |
| < −85 dBm | Mauvaise | ❌ Antenne externe impérative OU déport du boîtier |

### 6.3 Test de stabilité (optionnel mais recommandé)

Depuis un ordinateur portable ou téléphone placé au même endroit, lancer un ping de 1 min vers la box :

```
ping -n 60 <IP_box>          # Windows
ping -c 60 <IP_box>          # Linux/Mac
```

| Mesure | Relevé |
|--------|--------|
| Paquets envoyés | 60 |
| Paquets perdus | ______ |
| Temps de ping moyen | ______ ms |
| Temps de ping max | ______ ms |

> Attendu : < 2 % de perte, ping max < 100 ms pour une connexion MQTT fiable.

### 6.4 Verdict

- [ ] RSSI ≥ −80 dBm ? (antenne interne XIAO OK)
- [ ] Pas de coupures significatives sur 1 min de ping ?
- [ ] Si non → option **antenne externe** du XIAO ESP32-C3 (connecteur U.FL disponible sur la version "Antenna")

---
