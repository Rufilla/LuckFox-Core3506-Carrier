# Foxbridge — Luckfox Core3506 carrier board

A carrier board for the [Luckfox Core3506](https://wiki.luckfox.com/Core3506/Introduction/)
system-on-module (SoM), for anyone who wants the RK3506 on a Raspberry Pi Model B-sized board
(85 × 56 mm) with Ethernet, USB-C, a DSI display & a 40-pin header. Designed in KiCad 10 by
Rufilla Ltd., from the [Luckfox Lyra Pi](https://www.luckfox.com/Luckfox-Lyra-Pi) reference
design.

![Foxbridge carrier board](images/luckfox_core3506_adapter.png)

> **Revision A has been built & brought up with three hardware faults.** USB host (OTG1) does
> not work without two reworks. Read the [errata](docs/errata.md) before ordering boards.

## Features

- **USB-C ×2** — OTG0 (device, flashing & 5 V power input) & OTG1 (host, 500 mA current limit)
- **10/100 Ethernet** — CH182H2 PHY on RMII0, HR911105A RJ45 with integrated magnetics
- **MIPI DSI** — 2-lane, 22-pin 0.5 mm FPC for the Luckfox 7" DSI touchscreen (SKU 25265)
- **40-pin header** — Raspberry Pi-compatible pinout
- **microSD** — TF-110 socket with power switching; see errata E4 for the eMMC SoM
- **RESET & RECOVERY buttons**; UART0 header & Qwiic connector (both not fitted)
- **4-layer, 1.6 mm, impedance-controlled** — built for JLCPCB Economic PCBA

## Open the design

Requirements:

- KiCad 10.0 or later. Checked with 10.0.6.
- Third-party libraries, needed only to update a part from its library:

| Nickname in the design | Library | Install from |
|---|---|---|
| `PCM_SparkFun-*` | SparkFun KiCad Libraries | Plugin & Content Manager (PCM) |
| `PCM_*_AKL` | [Alternate KiCad Library](https://github.com/DawidCislo/Alternate-KiCad-Library) | PCM |
| `PCM_JLCPCB*` | [JLCPCB KiCad Library](https://github.com/CDFER/JLCPCB-Kicad-Library) | PCM, after adding CD_FER's repository |
| `digikey-footprints`, `dk_*` | [Digi-Key KiCad Library](https://github.com/Digi-Key/digikey-kicad-library) | Clone & register by hand (J3 only) |

PCM libraries resolve only with the default nickname prefix `PCM_` (**Preferences → Packages
& Updates**).

Open `luckfox_core3506_adapter.kicad_pro`. The schematic & board carry a copy of every symbol
& footprint, so the design opens, plots & runs ERC/DRC without the libraries above. Parts
specific to this board are in the project library `lib/` (nickname `Foxbridge`), with their
3D models embedded.

## Repository layout

| Path | Contents |
|---|---|
| `luckfox_core3506_adapter.kicad_pro` | KiCad project; the root schematic, 10 sheets & board sit beside it |
| `lib/` | Project symbol library & footprint library |
| `production/` | Revision A files as ordered from JLCPCB: Gerbers & drill (`.zip`), BOM, placement, IPC netlist |
| `docs/` | Design notes, errata, manufacturing notes & the Revision B checklist |
| `images/` | Board render & rework photographs |

## Documentation

- [Design notes](docs/design-notes.md) — how each subsystem works & why
- [Errata](docs/errata.md) — Revision A faults, reworks & limitations
- [Manufacturing](docs/manufacturing.md) — stack-up, ordering from JLCPCB, ERC/DRC status
- [Revision B checklist](docs/rev-b-checklist.md) — changes planned for the respin

## What it does not do

- No on-board 3.3 V regulator is fitted; the SoM supplies `VCC_3V3`. U2 is reserved for a
  future Wi-Fi/Bluetooth module.
- No Wi-Fi or Bluetooth.
- Power comes only from 5 V on the OTG0 USB-C port — no barrel jack & no PoE.
- The microSD slot does not work with the Core3506 variant that has on-module eMMC.
- No MASKROM button & no mounting holes in Revision A.

## Contributing

Report problems & suggestions through GitHub issues.

## Licence

Copyright 2026 Rufilla Ltd. Licensed under the [Apache License 2.0](LICENSE).

Symbols, footprints & 3D models copied from third-party libraries remain under their own
licences.
