# Design notes

How each subsystem of Revision A works & why it was designed that way. Faults found at
bring-up are in [errata.md](errata.md); fabrication detail is in
[manufacturing.md](manufacturing.md).

Contents: [Schematic](#schematic) · [Power](#power) · [USB](#usb) · [Ethernet](#ethernet) ·
[microSD](#microsd) · [DSI display](#dsi-display) · [Boot & reset](#boot--reset) ·
[Debug](#debug) · [LEDs](#leds) · [SoM pin mapping](#som-pin-mapping)

---

## Schematic

Hierarchical, 10 sheets. The SoM, U1 (`Core3506_multipart`), is an 8-unit symbol whose units
are placed on the sheets that use them.

| Page | Sheet |
|---|---|
| 1 | Contents & revisions |
| 2 | Block diagram |
| 3 | Power tree |
| 4 | Power regulation |
| 5 | USB interfaces |
| 6 | Ethernet |
| 7 | TF card |
| 8 | DSI |
| 9 | GPIO header & boot keys |
| 10 | Misc — test points, LEDs, fiducials |

## Power

- **Input:** 5 V from the OTG0 USB-C port (H5) through D3 (MBR230SFT Schottky) to
  `VCC5V0_SYS`. D3 stops OTG1 VBUS back-feeding OTG0.
- **`VCC5V0_SYS` TVS:** ED17 (ESD5451N, 5 V stand-off, 13 V clamp).
- **3.3 V:** supplied by the PMIC on the SoM. U2 (ME6217C33M5G LDO) is not fitted; it is
  reserved for a future Wi-Fi/Bluetooth module.
- **OTG1 VBUS:** U7 (TPS2553DBVR) power switch, current limit set to 500 mA by R4 (56 kΩ).
  ED1 (ESD5451N) protects its output. Does not switch on in Revision A — see errata E1.
- **Ethernet analogue supply:** FB1 (330 Ω at 100 MHz) isolates `VCCA3V3_RMII0` from `VCC_3V3`.

## USB

### OTG0 — device, flashing & power input (H5)

- CC pull-downs (Rd, i.e. USB-C sink): R136 & R124, 5.1 kΩ.
- D+/D− series resistors R110 & R114, 22 Ω — the same value that breaks OTG1 (errata E2).
- ESD: ED18 & ED2 (ESD5341N).
- VBUSDET (U1 pin 113) tied to `VCC_3V3` through R133 (10 kΩ); ID (pin 112) left
  unconnected. Both follow the Lyra Pi.

### OTG1 — host, 500 mA (H6)

- CC pull-ups (Rp, i.e. USB-C source): R1 & R2, 56 kΩ to OTG1 VBUS, advertising Default USB
  power.
- D+/D− series resistors R6 & R7, 22 Ω. **Wrong for high speed** — see errata E2.
- ESD: ED15 & ED16 (ESD5341N).

## Ethernet

### CH182H2 PHY (U5)

> **NOTE:** the CH182H2 is not strap-compatible with the CH182H1. The RMII-mode strap moved
> from pin 27 (COL) on the H1 to pin 8 (RXDV) on the H2. Use the H2 tables in datasheet
> CH182DS1; CH182H1 designs give the wrong strap assignments.

| Function | Setting | Parts |
|---|---|---|
| PHY address | 0x01 (PA0 = 1 via LED0 pull-up, PA1 = 0 via LED1 pull-down) | — |
| Mode | RMII: RXDV (pin 8) pulled high | R50, 4.7 kΩ |
| Clock | MAC drives 50 MHz REFCLK; CLKCTL pulled high | R38, 4.7 kΩ |
| 25 MHz XI | From SoM `ETH_CLK0_25M_OUT`; no crystal | R24, 0 Ω |
| LOS | Low — LED function, not wake-on-LAN | R61, 4.7 kΩ to GND |
| RSTB | Internal pull-up is enough | R52, 4.7 kΩ, not fitted |
| Idle state | CRS_DV & RXER pulled low | R45 & R68, 4.7 kΩ |

### RJ45 (J1, HR911105A)

- Integrated magnetics give galvanic isolation, so the MDI pairs carry no ESD diodes.
- Centre-tap decoupling: C2 & C3, 100 nF.
- Link & activity LEDs driven from the CH182H2 LED0 & LED1 pins.

### Routing

- MDI: 100 Ω differential; intra-pair match within 1.3 mm; ≥ 0.75 mm between pairs.
- RMII: 50 Ω single-ended; data matched to REF_CLK within 10 mm.

## microSD

- Socket: J2 (TF-110, LCSC C266613).
- Power: Q5 (SI2301CDS P-channel MOSFET) switches `VCC_3V3` to `VCC3V3_SD`, gate driven from
  `SDMMC_PWREN` through R70.
- Card detect: T2 (MMBT3906 PNP).
- ESD: ED8–ED14 (ESD5341N) on CLK, CMD, D0–D3 & card detect; D2 on `VCC3V3_SD`.
- Series resistors: R59 (4 × 22 Ω array) on D0–D3, R65 (22 Ω) on CMD. CLK has none.
- Routing: matched to CLK within about 7.5 mm, inside the tolerance for 50 MHz. CMD, D2 & D3
  change to B.Cu through two vias each.

The slot is electrically dead when the SoM has on-module eMMC — a limitation of the RK3506,
explained in errata E4.

## DSI display

Target display: Luckfox 7" DSI touchscreen (SKU 25265) — 800 × 480 IPS, 5-point capacitive
touch — through H4 (22-pin 0.5 mm FPC, LCSC C262668). Sold by
[Luckfox](https://www.luckfox.com/EN-7inch-DSI-Touchscreen) & Waveshare.

| FPC pin | Signal | Connection |
|---|---|---|
| 1 | `VCC_3V3` | Display power |
| 2 | `I2C2_SDA` | Touch; R15 22 Ω series, R11 3.3 kΩ pull-up |
| 3 | `I2C2_SCL` | Touch; R14 22 Ω series, R10 3.3 kΩ pull-up |
| 4, 7, 10, 13, 16, 19, 22 | GND | |
| 5 | `LCD_IO1` | GPIO1_C4 (U1 pin 34) — LCD reset, R12 10 kΩ pull-up |
| 6 | `LCD_IO0` | GPIO0_B7 (U1 pin 63) — backlight enable |
| 8, 9, 11, 12 | — | Not connected |
| 14–15 | `DSI_TX_CLKN/P` | U1 pins 1 & 2; 2.2 Ω series (R5 array) |
| 17–18 | `DSI_TX_D1N/P` | U1 pins 4 & 5; 2.2 Ω series (R5 array) |
| 20–21 | `DSI_TX_D0N/P` | U1 pins 7 & 8; 2.2 Ω series (R8, R9) |

Departures from the Lyra Pi:

- **No LCD power switch** (WPM3407-3/TR). The board is always powered from USB, & the DSI
  driver sequences reset independently of power.
- **No backlight driver** (TCS7191A). The display has its own; only an enable is needed.
- **2.2 Ω instead of 0 Ω** on all six DSI lines, for consistent damping.
- **No ESD on the FPC.** The connector is internal, & the DSI lanes are sensitive to added
  capacitance.

`I2C2_SDA` (U1 pin 43, GPIO1_B3) & `I2C2_SCL` (U1 pin 36, GPIO1_C2) are shared by the touch
panel, the 40-pin header (H1 pins 3 & 5) & the Qwiic connector, with no jumpers.

## Boot & reset

- **RESET** (Key1): pulls `NPOR_L` (U1 pin 80) low through R30 (100 Ω), ESD by ED3. The SoM
  PMIC drives `NPOR_L` push-pull, so there is no external pull-up — as on the Lyra Pi.
- **RECOVERY** (Key2): pulls `SARADC_IN1` low through R34 (100 Ω), ESD by ED5. A key read by
  software after boot; it does **not** enter maskrom (errata E5).
- **Maskrom strap:** `SARADC_IN0_BOOT` (U1 pin 114) reaches test point TP6 through R33
  (100 Ω), ESD by ED4. No button is fitted.

## Debug

Neither connector is fitted in Revision A.

- **J3** — UART0, 3-pin 2.54 mm header: U1 pins 56 & 57 (GPIO0_C7, GPIO0_C6) & GND. UART0 is
  also on the 40-pin header.
- **J4** — Qwiic/STEMMA QT (JST-SH 4-pin): `I2C2_SDA`, `I2C2_SCL`, `VCC_3V3`, GND.

## LEDs

Both green 0805, each switched low-side by an SS8050 NPN from a SoM GPIO.

| LED | Transistor | GPIO | Base resistor | LED resistor |
|---|---|---|---|---|
| D1 (SYS) | T1 | GPIO1_A0 (U1 pin 55) | R111, 1 kΩ | R101, 470 Ω |
| D5 (USER) | T3 | GPIO1_A6 (U1 pin 49) | R17, 1 kΩ | R16, 470 Ω |

## SoM pin mapping

| U1 unit | Sheet | Signals |
|---|---|---|
| 1 | Power regulation | `VCC5V0_SYS`, `VCC_3V3`, `VCC_1V8` |
| 2 | USB interfaces | OTG0 D+/D− (108–109), OTG1 D+/D− (110–111), ID (112), VBUSDET (113) |
| 3 | Ethernet | RMII0, `ETH_CLK0_25M_OUT`, MDC/MDIO |
| 4 | TF card | SDMMC (101–106) |
| 5 & 6 | GPIO header & boot keys | `NPOR_L` (80), SARADC, GPIO0_B1–B6, GPIO0_C6–C7, GPIO1_A3–A5, GPIO1_D2–D3 |
| 7 | DSI | DSI CLK, D0 & D1 (1–8), `LCD_IO0` (63), `LCD_IO1` (34) |
| 8 | Misc | LED drive & power |
