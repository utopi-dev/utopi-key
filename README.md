# utopi-key

**Open-source hardware FIDO2 USB security key** — a transparent, auditable alternative to YubiKey / Google Titan, designed for local manufacturing in Quebec, Canada.

> Goal: A compact USB-A (or USB-C) FIDO2/WebAuthn authenticator with physical user-presence button, open schematics, open firmware, and a clear path to small-batch production in Quebec.

## Status

**Planning / Early Design** — Hardware architecture defined. Full KiCad schematic and PCB layout to be added. Firmware will target existing open-source stacks (pico-fido / RS-Key style or Trussed).

## Why this project?

- YubiKey and Titan are closed-source hardware and firmware.
- Existing open options (Nitrokey 3, SoloKeys, RP2350 DIY keys) prove the concept works.
- Quebec has capable PCB fab/assembly houses and component distributors — we want a design that can be built locally.
- Emphasis on **open hardware** (CERN-OHL or CC-BY-SA) + **open firmware**.

## Target Specifications

| Feature                  | Target                                      |
|--------------------------|---------------------------------------------|
| Form factor              | USB-A male stick (~45 × 18 × 5 mm assembled) or USB-C |
| Protocol                 | FIDO2 / CTAP2.1 + U2F (CTAP1) + WebAuthn    |
| User presence            | Physical tactile button (or capacitive)     |
| Status LED               | Optional RGB or single LED                  |
| MCU                      | **Raspberry Pi RP2350A** (QFN-60)           |
| Security features        | Secure boot, OTP, TrustZone, signed firmware |
| Flash                    | External 8–16 MB QSPI (W25Q64/128)          |
| Power                    | USB 5 V → 3.3 V LDO + internal 1.1 V switcher |
| Enclosure                | 3D-printable (PETG) + optional injection mold |
| Certification target     | FIDO Alliance (future)                      |
| License                  | Hardware: CERN-OHL-S-2.0 or CC-BY-SA-4.0<br>Firmware: Apache-2.0 / MIT |

### Why RP2350A?

- Modern dual Cortex-M33 with hardware security features (secure boot, OTP, SHA-256 accelerator, TrustZone).
- Mature open-source FIDO2 firmware already exists (pico-fido lineage, RS-Key, etc.).
- Excellent official hardware design guide and Minimal reference schematic.
- Lower cost and more available than dedicated secure elements for a first design.
- Future upgrade path: pair with a Common Criteria secure element if desired.

## High-Level Architecture

```
USB-A Connector
    ↓
5 V VBUS ──── LDO (3.3 V) ────┬──── IOVDD / USB_OTP_VDD / QSPI_IOVDD
                        │
                        └──── Internal switcher (1.1 V DVDD) via 3.3 µH inductor

RP2350A
  ├─ USB_DP / USB_DM (via 27 Ω series)
  ├─ QSPI Flash (W25Q128)
  ├─ 12 MHz crystal (ABM8-272-T3) + 15 pF load caps
  ├─ BOOTSEL button (pulls QSPI_SS low)
  ├─ User Presence button (GPIO)
  ├─ Optional LED
  └─ SWD debug header (for development)
```

## Detailed Hardware Design (based on official RP2350 Minimal)

The design follows the official Raspberry Pi *Hardware design with RP2350* Minimal schematic (Appendix A) with the following adaptations for a security key:

### Power Supply
- USB VBUS (5 V) → NCP1117-3.3 or AMS1117-3.3 LDO → 3.3 V rail
- Decoupling: 10 µF on input and output of LDO
- Internal 1.1 V switcher:
  - Inductor: Abracon AOTA-B201610S3R3-101-T (3.3 µH)
  - Input cap (VREG_VIN): 4.7 µF
  - Output cap (DVDD): 4.7 µF
  - AVDD filter: 33 Ω + 4.7 µF
- Multiple 100 nF ceramic decoupling on IOVDD, DVDD, USB_OTP_VDD, QSPI_IOVDD

### Clock
- Crystal: Abracon ABM8-272-T3 (12 MHz, 10 ppm, 10 pF)
- Load capacitors: 15 pF (0402)

### Flash
- Winbond W25Q128JVS (or W25Q64) SOIC-8 or WSON
- Connected to QSPI_SS, QSPI_SCLK, QSPI_SD0–SD3
- BOOTSEL button on QSPI_SS via 1 kΩ (for DFU / UF2 flashing)

### USB
- USB-A male edge connector (PCB-integrated gold fingers) **or** discrete USB-A plug
- Series resistors: 27 Ω on D+ and D− (close to RP2350)
- Optional ESD protection (TVS diodes)

### User Interface
- User Presence tactile switch (e.g. 4×3×2.5 mm, 180 g force)
- Optional RGB LED (WS2812B or discrete)
- BOOTSEL button (can be internal or accessible via pinhole)

### Debug (development only)
- SWD header (SWCLK, SWDIO, GND, 3V3)
- RUN pin accessible

## Bill of Materials (BOM) — Core Components

| Designator | Description                  | Part / MPN                          | Footprint     | Qty | Notes / Source          |
|------------|------------------------------|-------------------------------------|---------------|-----|-------------------------|
| U1         | MCU                          | RP2350A                             | QFN-60        | 1   | Raspberry Pi / LCSC     |
| U2         | 3.3 V LDO                    | NCP1117-3.3 or AMS1117-3.3          | SOT-223       | 1   | Common                  |
| U3         | QSPI Flash                   | W25Q128JVS                          | SOIC-8        | 1   | Winbond                 |
| Y1         | 12 MHz Crystal               | ABM8-272-T3                         | 3225          | 1   | Abracon                 |
| L1         | 3.3 µH Inductor              | AOTA-B201610S3R3-101-T              | 0806          | 1   | Abracon                 |
| SW1        | User Presence Button         | Tactile 4×3 mm                      |             | 1   |                         |
| SW2        | BOOTSEL Button               | Tactile                             |             | 1   |                         |
| C*         | Decoupling capacitors        | 100 nF, 4.7 µF, 10 µF X5R/X7R      | 0402/0603     | ~15 |                         |
| R*         | Resistors                    | 27 Ω, 1 kΩ, 33 Ω, 0 Ω             | 0402          | ~6  |                         |
| J1         | USB-A Male                   | PCB edge or discrete plug           |             | 1   |                         |
| D1         | Optional LED                 | WS2812B or 0603 LED                 |             | 1   |                         |

Full detailed BOM with LCSC part numbers will be generated once the KiCad design is finalized. Components are widely available from Digi-Key, Mouser, LCSC, and Canadian distributors (FM Electroniques, etc.).

## Quebec Manufacturing Path

### PCB Fabrication & Assembly
- **R.S. Electronique** (Montreal) — turnkey PCBA, quick-turn prototypes
- **M2S Électronique** (Québec City) — SMT + through-hole, ISO certified
- **Qualieco Circuits** — PCB fab + assembly
- **Cancino Technologies** (Montreal) — design + assembly
- **Fabrique Manic** (near Québec City) — prototype-focused

For first prototypes, JLCPCB / PCBWay (with Canadian shipping) or local houses are both viable. Target 2-layer or 4-layer, 1.6 mm (or 2.0 mm for better USB contact).

### Component Sourcing
- Local: FM Electroniques (Montreal), other Canadian distributors
- Global: LCSC (for RP2350 and passives), Digi-Key, Mouser

### Enclosure
- 3D print first (PETG or resin)
- Later: local injection molding or CNC aluminum for premium versions

## Next Steps

1. [ ] Create full KiCad schematic starting from official RP2350A Minimal design
2. [ ] Layout compact USB-stick PCB (target < 45 mm long)
3. [ ] Generate Gerbers + BOM with LCSC/JLCPCB parts
4. [ ] Order first prototype PCBs
5. [ ] Port / adapt open FIDO2 firmware (pico-fido or RS-Key)
6. [ ] Design 3D enclosure
7. [ ] Document manufacturing package for Quebec partners

## References & Inspiration

- [Raspberry Pi Hardware design with RP2350](https://datasheets.raspberrypi.com/rp2350/hardware-design-with-rp2350.pdf) (official Minimal schematic)
- [Nitrokey FIDO2 hardware](https://github.com/Nitrokey/nitrokey-fido2-hardware)
- [SoloKeys solo2-hw](https://github.com/solokeys/solo2-hw)
- [CyberTrust (RP2350A)](https://github.com/Aahil78/CyberTrust)
- [PicoKey](https://github.com/Keyaan-07/PicoKey)
- [RS-Key firmware](https://github.com/TheMaxMur/RS-Key)
- [pico-fido](https://github.com/librekeys/pico-fido2)

## License

Hardware designs will be released under **CERN-OHL-S-2.0** (or dual with CC-BY-SA-4.0).  
Firmware under Apache-2.0 / MIT.

---

**This is a community project.** Contributions welcome — especially KiCad layout, mechanical design, firmware porting, and Quebec manufacturing contacts.

Built with the goal of digital sovereignty and transparent hardware security.
