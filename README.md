# ESP32 + Dual nRF24L01+ PA/LNA — Carrier PCB

A custom carrier board that holds:
- **ESP32-WROOM-32** (soldered on the board)
- **2× nRF24L01+ PA/LNA** modules (plug into 2×4 sockets) — antennas face **up / out**
- **TP4056 USB-C** charger — on the **back**, USB-C at the board edge
- **18650 cell** holder — on the **back**
- **ON/OFF slide switch** + **3.3 V regulator** + decoupling + a UART header for flashing

---

## Files in this folder

| File | What it is | How to use |
|---|---|---|
| `esp32_dual_nrf24.net` | KiCad **netlist** (all connections) | In KiCad Pcbnew: **File → Import → Netlist** → places all parts with ratsnest |
| `schematic.svg` | Visual pin map + placement | Open in any browser / image viewer |
| `BOM.csv` | Bill of materials | Open in Excel / Google Sheets |
| `gen.py` | Script that made the netlist/BOM | For editing pins later |

---

## Pin map (ESP32 → nRF24)

nRF24 socket order (2×4): `1 GND · 2 VCC · 3 CE · 4 CSN · 5 SCK · 6 MOSI · 7 MISO · 8 IRQ`

| Signal | nRF #1 (VSPI) | nRF #2 (HSPI) |
|---|---|---|
| CE   | IO17 | IO26 |
| CSN  | IO5  | IO15 |
| SCK  | IO18 | IO14 |
| MOSI | IO23 | IO13 |
| MISO | IO19 | IO27 |
| IRQ  | IO16 | IO25 |

VCC = 3.3 V, GND shared. Pins avoid ESP32 boot/strapping/flash pins, so the board boots and flashes cleanly. You can change them in firmware (ESP32 routes SPI to any GPIO).

---

## Board spec (give these to the PCB maker)

- **Size:** ~70 mm × 45 mm (fits an 18650 on the back)
- **Layers:** 2
- **Thickness:** 1.6 mm
- **Copper:** 1 oz
- **Surface finish:** HASL (cheapest) or ENIG
- **Min track / clearance:** 6 mil / 6 mil (JLCPCB default — easy)
- **Min hole:** 0.3 mm
- **Fill both layers with a GND copper pour**, stitched with vias.

### Placement rules (important)
1. Put both nRF24 sockets on **one edge** so the antennas stick **out past the board** — nothing (no battery, no copper pour under the SMA area) behind the antenna.
2. **ESP32-WROOM antenna end must sit at the board edge** too — keep copper and parts away from the WROOM antenna keep-out zone (it is marked on the module).
3. 18650 holder + TP4056 go on the **back**, with the **USB-C port at the board edge** so you can plug the cable.
4. Each nRF24 gets its own **10 µF** cap right on its VCC pin — this is what makes PA/LNA range actually work. Add the **470 µF** bulk cap near the ESP32.

### ⚠ Power note (the one real risk)
One 18650 (3.0–4.2 V) feeding the ESP32 **and** two PA radios is heavy. A plain AMS1117 LDO drops out when the battery gets low. For a board that works down to a low battery, use a **buck-boost regulator (TPS63020 / similar)** set to 3.3 V instead of the LDO. The BOM lists the LDO as the simple option and the buck-boost as the "do it right" option.

---

## How to finish and order the PCB

**Easiest path (recommended for a first board): EasyEDA + JLCPCB**
1. Go to **https://easyeda.com** (free, runs in the browser).
2. Draw the schematic from the pin map above (or **File → Import → import the `.net`** if you stay in KiCad).
3. Click **Convert to PCB**, arrange the parts as in `schematic.svg`, route the tracks, add a GND pour.
4. **Generate Gerber → Order at JLCPCB** (button is built in). ~\$2 for 5 boards + shipping.

**KiCad path (what you picked):**
1. Install **KiCad** (free): **https://www.kicad.org/download/**
2. New project → open **Pcbnew** → **File → Import → Netlist** → pick `esp32_dual_nrf24.net`.
3. Place parts (follow `schematic.svg`), route, add GND pour, run **DRC**.
4. **File → Fabrication Outputs → Gerbers + drill**, zip them.
5. Upload the zip to **https://jlcpcb.com** or **https://www.pcbway.com** and order.

---

*Generated 2026-10-06. Pin map and BOM are editable in `gen.py`.*
