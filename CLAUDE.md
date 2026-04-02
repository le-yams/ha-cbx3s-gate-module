# ha-cbx3s-gate-module

## Description

Module électronique à concevoir pour piloter et superviser un portail battant équipé d'un boîtier **Somfy Control Box 3S io**, intégré à **Home Assistant** via **MQTT over WiFi**.

## Phase actuelle

**Étude des besoins et faisabilité** — Pas encore de code.

## Prochaines étapes

1. **Mesures sur site** — prérequis, débloque tout le reste → `specs/mesures-sur-site.md`
2. **Arbitrages techniques** — détection secteur, relais → décisions D2–D5 dans `specs/etude-faisabilite.md`
3. **Conception matérielle** — schéma, composants, PCB, boîtier
4. **Implémentation firmware** — découpage en tâches à partir du MVP (F01–F06)
5. **Fonctionnalités optionnelles** — F07–F08, après MVP fonctionnel

## Structure du projet

```
docs/                           Documentation technique de référence
  reference-cbx-3s-io.md        Synthèse du bornier, paramètres et comportements de la CBX 3S io
  reference-xiao-esp32c3.md     Pinout, strapping pins, ADC, alimentation, deep sleep du XIAO ESP32C3
  *.pdf                          Notices officielles Somfy (Control Box, aide-mémoire)
specs/                          Spécifications et étude
  besoins-fonctionnels.md       Cas d'usage MVP et optionnels, contraintes
  etude-faisabilite.md          Options techniques, arbitrages ouverts, mesures à faire
```

## Conventions

- Les fonctionnalités sont identifiées par un code (F01, F02, etc.) défini dans `specs/besoins-fonctionnels.md`
- Les décisions techniques ouvertes sont numérotées (D1, D2, etc.) dans `specs/etude-faisabilite.md`
- La référence technique (`docs/reference-cbx-3s-io.md`) est une synthèse vérifiée des PDFs Somfy — la consulter plutôt que les PDFs sauf besoin de vérification

## MCU — Seeed Studio XIAO ESP32C3

- **Chip** : ESP32-C3 RISC-V single-core 160 MHz, 400 KB SRAM, 4 MB flash
- **Connectivité** : WiFi 802.11 b/g/n + BLE 5.0
- **GPIO** : 11 pins (8 libres + 3 strapping), 5 utilisés par le projet, 6 libres
- **Format** : 21 × 17.8 mm
- **Documentation** :
  - Référence locale (pinout, strapping, ADC, power) : `docs/reference-xiao-esp32c3.md`
  - Wiki Seeed (source) : https://wiki.seeedstudio.com/XIAO_ESP32C3_Getting_Started/
  - Datasheet ESP32-C3 : https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf
  - ESP-IDF Programming Guide : https://docs.espressif.com/projects/esp-idf/en/stable/esp32c3/
  - Arduino ESP32 core : https://docs.espressif.com/projects/arduino-esp32/en/latest/

## Contexte technique clé

- La CBX 3S io se pilote via **contacts secs filaires** (bornes 30/31/32), pas via le protocole radio io-homecontrol
- L'état du portail est disponible via la **sortie auxiliaire** (bornes 7/8) à condition que **P15=1**
- Le module est alimenté par le **24V DC** de la CBX (bornes 19/20), qui reste disponible sur batterie de secours
- Le paramètre **P37** détermine si les entrées filaires fonctionnent en mode cycle (total/piéton) ou directionnel (ouverture/fermeture)
- Les boutons physiques externes sont câblés **en parallèle des relais** sur les bornes CBX (pas de GPIO consommé)
