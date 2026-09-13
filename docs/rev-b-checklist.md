# Revision B checklist

Changes planned for the respin. Causes & reworks for the faults are in [errata.md](errata.md).

## Faults — must fix

- [ ] **E1 — OTG1 VBUS switch.** Connect R3 pin 1 to U7 pin 1 (IN), not pin 6 (OUT).
- [ ] **E2 — USB series resistors.** R6 & R7 → 0 Ω or removed; the same for R110 & R114 on OTG0.
- [ ] **E3 — USB-C spacing.** Move H5 & H6 apart so two cables fit at once.

## Improvements

- [ ] **MASKROM button** from `SARADC_IN0_BOOT` to GND through R33 & ED4, labelled MASKROM.
      It is the only input that makes the RK3506 BootROM enter maskrom (errata E5).
- [ ] **Key2** — relabel from RECOVERY to USER or KEY, or remove it if no firmware reads
      `SARADC_IN1`.
- [ ] **U7 EN to a free GPIO**, instead of a fixed pull-up. Linux can then power-cycle OTG1, &
      `vcc5v0_otg1` gets a `gpio` property instead of `regulator-fixed` with `always-on`.
- [ ] **U7 FAULT# to a free interrupt-capable GPIO** (open-drain). Linux can then log OTG1
      over-current & thermal shutdown instead of browning out silently.
- [ ] **Silkscreen warning beside J2:** "SD slot non-functional with on-module eMMC SoM
      (Core3506-0808)" (errata E4).
- [ ] **Silkscreen:** align the 40-pin header numbering on the bottom side, & raise its text
      thickness to 0.15 mm.
- [ ] **Mounting holes:** add four, one in each corner.
- [ ] **Header pins** for `SARADC_IN0_BOOT` & `NPOR_L`, which are test points only today.
