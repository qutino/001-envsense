# 001_env-sense — Schematic TODO

Status (2026-11-25): the 9 main parts are placed, but there are no wires, power symbols, resistors or capacitors yet.
Values are typical datasheet values, so check each against its datasheet.

**Suggested order:** wire the power path first (sections 1–5). Then do the ESP32 basics (6) and the sensor and display (7–9). Run the rules check (ERC) after each block.

## 1. USB-C input (J1)
- [ ] CC1 and CC2 each get their own 5.1 kΩ resistor to GND (without them, USB-C chargers won't supply 5 V; don't share one resistor)
- [ ] Join the two D+ pins together and the two D− pins together
- [ ] Wire D+ to GPIO19 and D− to GPIO18 on the ESP32 (built-in USB, so you can program and debug over USB-C)
- [ ] ESD protection on VBUS, D+ and D− (e.g. USBLC6-2SC6)
- [ ] Shield to GND, directly or through 1 MΩ in parallel with 4.7 nF
- [ ] 10 µF on VBUS

## 2. Charger: TP4056 (U2)
- [ ] PROG resistor sets charge current: 10 kΩ ≈ 130 mA, 2 kΩ ≈ 580 mA, 1.2 kΩ ≈ 1 A (pick about 0.5 × the cell's capacity)
- [ ] TEMP to GND if you're not using a battery thermistor
- [ ] Tie CE to VCC
- [ ] CHRG and STDBY: an LED with a 1 kΩ resistor on each, or wire them to ESP32 GPIOs with pull-ups
- [ ] 10 µF on VCC (input) and 10 µF on BAT
- [ ] Exposed pad (ESOP-8) to GND with thermal vias

## 3. Battery protection: DW01A (U3)
- [ ] **Add the dual N-MOSFET (e.g. FS8205A). It's missing, and the DW01A does nothing without it.**
- [ ] VCC: 100 Ω from battery +, and 0.1 µF to battery −
- [ ] CS: about 1–2 kΩ to the system-side ground (P−)
- [ ] Connect OD and OC to the two MOSFET gates
- [ ] Keep battery − (B−) and system ground (P−) as two separate nets, with the MOSFETs between them

## 4. Optional: USB powers the board while charging
- [ ] P-MOSFET plus Schottky diode so the board runs from USB when plugged in (keeps the TP4056's end-of-charge detection working)

## 5. 3.3 V regulator: AP2112K-3.3 (U5)
- [ ] 1 µF on the input, 1 µF on the output (2.2–10 µF on the output is better)
- [ ] EN to VIN, or to a GPIO
- [ ] Dropout: Wi-Fi transmit spikes of about 350 mA make the 3.3 V rail sag below about 3.5 V on the battery. Set a firmware cutoff of about 3.5 V, or use a lower-dropout regulator or a buck-boost
- [ ] Battery voltage sensing: 100 kΩ/100 kΩ divider to an ADC1 pin (GPIO0–4), with 100 nF on the ADC pin

## 6. ESP32-C3-WROOM-02 (U1)
- [ ] 10 µF and 0.1 µF on 3V3, close to the module
- [ ] EN: 10 kΩ pull-up and 1 µF to GND; optional RESET button to GND
- [ ] GPIO9: 10 kΩ pull-up and a BOOT button to GND
- [ ] GPIO8: pull-up (must read high at boot); GPIO2: pull-up
- [ ] Don't use the boot pins for anything that holds them low at power-up
- [ ] Antenna at the board edge, with no copper on any layer and no parts under or near it
- [ ] GND pad with plenty of vias
- [ ] I2C pins: e.g. SDA = GPIO4, SCL = GPIO5

## 7. BME280 sensor (U6)
- [ ] 0.1 µF on VDD and 0.1 µF on VDDIO
- [ ] Tie CSB to VDDIO (selects I2C mode)
- [ ] SDO to GND gives address 0x76 (SDO to VDD gives 0x77)
- [ ] Layout: at the board edge, away from the ESP32 and regulator, with a slot cut around it

## 8. OLED display (U7)
- [ ] Decide between the bare panel and a ready-made module. The bare ER-OLEDM0.91 panel needs the SSD1306's capacitors and resistor (VCOMH, IREF, VDD caps) from its datasheet; for a ready-made module, swap in a 4-pin header
- [ ] I2C address 0x3C, so no clash with the BME280
- [ ] Put the display to sleep with the SSD1306's sleep command to save battery

## 9. I2C bus
- [ ] 4.7 kΩ pull-ups on SDA and SCL (ready-made OLED modules often already have them, so check to avoid doubling up)

## 10. TLP222A-2 solid-state relay (U4)
- [ ] Decide whether you need it; if not, delete it
- [ ] If you keep it: about 330 Ω from the GPIO to each input LED (roughly 6 mA)
- [ ] Add an output connector for whatever it switches

## 11. Before starting the PCB
- [ ] GND, +3V3 and VBAT power symbols, plus a PWR_FLAG on each supply
- [ ] Clean rules check (ERC)
- [ ] A footprint for every part
- [ ] Test points: VBUS, VBAT, 3V3, GND, SDA, SCL
- [ ] Mounting holes
