# Mesures sur site — Control Box 3S io

Document à imprimer pour effectuer les relevés et vérifications sur le boîtier CBX 3S io.
Matériel nécessaire : **multimètre**, **chronomètre** (ou téléphone), **stylo**.

---

## 1. Lire les paramètres de la CBX

### Procédure de lecture (sans modification)

1. Appui long sur **PROG** (~0.5s) → l'écran affiche **P01**
2. Naviguer avec les touches **+** / **−** jusqu'au paramètre souhaité
3. La valeur affichée **fixe** = valeur actuellement configurée
4. **Ne pas appuyer sur OK** (sinon la valeur serait validée/modifiée)
5. Pour sortir : attendre ~30s sans toucher, l'afficheur revient à l'écran normal (C1)

### Paramètres à relever

| Paramètre | Fonction                                  | Valeur relevée | Valeur souhaitée          |
|-----------|-------------------------------------------|:--------------:|:-------------------------:|
| **P15**   | Mode sortie auxiliaire (bornes 7/8)       |    ________    | **1** (témoin portail)    |
| **P37**   | Mode commandes filaires (bornes 30/32)    |    ________    | **0** (cycle total/piéton)|
| P01       | Mode de fonctionnement cycle total        |    ________    | — (noter pour info)       |
| P07       | Entrée sécurité cellules                  |    ________    | — (noter pour info)       |
| P13       | Mode sortie éclairage de zone             |    ________    | — (noter pour info)       |
| P16       | Temporisation sortie auxiliaire           |    ________    | — (noter pour info)       |

> **P15** et **P37** sont critiques pour le projet. Les autres sont à relever pour information.

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
| Portail à l'arrêt, alimentation secteur         |   __________ V  |
| Portail en mouvement (ouverture), secteur       |   __________ V  |
| Portail en mouvement (fermeture), secteur       |   __________ V  |
| Portail à l'arrêt, alimentation batterie (*)    |   __________ V  |
| Portail en mouvement, alimentation batterie (*) |   __________ V  |

(*) Pour passer sur batterie : couper le disjoncteur dédié au portail. Vérifier que la CBX reste alimentée (afficheur allumé, code Hc1 ou Hu1).


### Identification du type de batterie

Quand le 230V est coupé, l'afficheur indique :
- **Hc1** → batterie 9,6V
- **Hu1** → batterie 24V
- Pas de batterie installée si l'afficheur s'éteint

Code affiché sur batterie : __________

---

## 3. Caractérisation de la sortie auxiliaire (bornes 7/8)

⚠ **Prérequis** : P15 doit être à **1**. Si ce n'est pas le cas, il faudra le modifier (voir section 5).

Multimètre en mode **continuité/résistance** ou **DC** avec pull-up externe, pointes sur borne 7 et borne 8.

### Test portail fermé

État du contact 7/8 (ouvert ou fermé ?) : __________

### Test portail en mouvement

Déclencher une ouverture totale et observer le contact 7/8 :

| Mesure                                   | Relevé          |
|------------------------------------------|:---------------:|
| Le contact clignote-t-il ? (oui / non)   |   __________    |
| Fréquence approximative du clignotement  |   __________ Hz |
| Durée ON approximative                   |   __________ ms |
| Durée OFF approximative                  |   __________ ms |

> Pour la fréquence : compter le nombre de changements en 10s et diviser par 10.

### Test portail ouvert (arrêté en position ouverte)

État du contact 7/8 (ouvert ou fermé ?) : __________

### Comparaison ouverture totale vs piéton

| Test                    | Comportement du contact 7/8 |
|-------------------------|-----------------------------|
| Ouverture **totale**    | ____________________________|
| Ouverture **piéton**    | ____________________________|
| Différence ? (oui/non)  | ____________________________|

---

## 4. Modification de P15 (si nécessaire)

**Uniquement si P15 n'est pas à 1** et que tu souhaites le modifier sur place.

1. Appui long sur **PROG** (~0.5s) → l'écran affiche **P01**
2. Naviguer avec **+** / **−** jusqu'à **P15**
3. Appuyer sur **OK** → la valeur clignote
4. Utiliser **+** / **−** pour sélectionner **1**
5. Appuyer sur **OK** pour valider → la valeur arrête de clignoter
6. Attendre ~30s pour sortir du mode paramétrage

> Après modification de P15, refaire un cycle d'ouverture/fermeture pour vérifier le comportement de la sortie 7/8.
