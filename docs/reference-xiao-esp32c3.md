# Référence technique — Seeed Studio XIAO ESP32C3

Source : https://wiki.seeedstudio.com/XIAO_ESP32C3_Getting_Started/

---

## Spécifications principales

- **Chip** : ESP32-C3 RISC-V single-core 32-bit, 160 MHz, pipeline 4 étages
- **RAM** : 400 KB SRAM
- **Flash** : 4 MB
- **WiFi** : 802.11 b/g/n (2.4 GHz)
- **Bluetooth** : BLE 5.0 / Mesh
- **Dimensions** : 21 × 17.8 mm
- **Température** : -40 °C à +85 °C

---

## Alimentation

| Paramètre | Valeur |
|-----------|--------|
| Tension entrée (USB) | 5 V |
| Tension entrée (batterie) | 3.7 V |
| Sortie 3.3 V max | 700 mA |
| Source max | 3 A |
| Charge rapide batterie | 380 mA |
| Charge trickle | 40 mA |

### Consommation

| Mode | Courant |
|------|---------|
| WiFi actif | 75 mA |
| WiFi modem-sleep | 25 mA |
| WiFi light-sleep | 4 mA |
| BLE modem-sleep | 27 mA |
| BLE light-sleep | 10 mA |
| Deep sleep | 44 µA |

### Pins d'alimentation

- **5V** : sortie USB ou entrée externe (diode série requise si alimentation externe)
- **3V3** : sortie régulée, 700 mA max
- **GND** : masse commune

---

## Brochage (pinout)

| Pin XIAO | Fonction | GPIO | ADC | Fonctions alternatives |
|----------|----------|------|-----|------------------------|
| D0 | Analog | GPIO2 | ADC1_CH2 | — |
| D1 | Analog | GPIO3 | ADC1_CH3 | — |
| D2 | Analog | GPIO4 | ADC1_CH4 | FSPIHD, MTMS (JTAG) |
| D3 | Analog | GPIO5 | ADC2_CH0 | FSPIWP, MTDI (JTAG) |
| D4 | SDA | GPIO6 | — | FSPICLK, MTCK (JTAG) |
| D5 | SCL | GPIO7 | — | FSPID, MTDO (JTAG) |
| D6 | TX | GPIO21 | — | U0TXD |
| D7 | RX | GPIO20 | — | U0RXD |
| D8 | SCK | GPIO8 | — | SPI Clock |
| D9 | MISO | GPIO9 | — | SPI Data (+ bouton BOOT) |
| D10 | MOSI | GPIO10 | — | FSPICS0 |

### Interfaces

- **UART** : 1 port (D6=TX / D7=RX), partagé avec USB-série
- **I2C** : 1 port (D4=SDA / D5=SCL)
- **SPI** : 1 port (D8=SCK / D9=MISO / D10=MOSI)
- **PWM** : disponible sur tous les GPIO
- **ADC** : 4 canaux (3 fiables sur ADC1, voir avertissement ci-dessous)

---

## Strapping pins (attention au boot)

**GPIO2, GPIO8 et GPIO9 sont des strapping pins.** Leur niveau logique au moment du reset détermine le mode de démarrage du chip. Un niveau incorrect peut empêcher le boot ou le flash du firmware.

| GPIO | Comportement |
|------|-------------|
| GPIO2 | Doit être flottant ou haut au boot |
| GPIO8 | Doit être haut au boot (mode normal) |
| GPIO9 | Haut = boot normal, Bas = mode download (bouton BOOT) |

**En pratique** : ces pins sont utilisables après le démarrage, mais le circuit externe ne doit pas forcer un niveau incompatible au moment du reset. Éviter d'y connecter des charges qui tirent vers le bas au démarrage.

---

## ADC — Avertissement

**D3 (GPIO5 / ADC2_CH0)** utilise ADC2, qui peut produire des lectures erronées (faux signaux d'échantillonnage). Pour des lectures analogiques fiables, utiliser **ADC1 uniquement** :
- D0 (GPIO2) — ADC1_CH2
- D1 (GPIO3) — ADC1_CH3
- D2 (GPIO4) — ADC1_CH4

---

## Deep sleep et réveil

- Consommation deep sleep : ~44 µA
- Sources de réveil supportées : GPIO et timer
- **Pins compatibles wake-up** : D0, D1, D2, D3 uniquement (GPIO2–5)

---

## Batterie (3.7 V lithium)

- Pads de soudure sur la face arrière (+ et −)
- Circuit de charge intégré (380 mA fast / 40 mA trickle)
- Mesure de tension batterie : pont diviseur externe (2 × 200 kΩ) sur A0, lecture via `analogReadMilliVolts()`

---

## Boutons intégrés

- **Reset** (CHIP_EN) : reset hardware du chip
- **Boot** (GPIO9) : maintenu au reset → mode download. Réutilisable comme bouton utilisateur après le démarrage

---

## Antenne

- Antenne PCB intégrée sur la carte
- Connecteur U.FL disponible pour antenne externe (nécessite de déplacer une résistance 0 Ω)
