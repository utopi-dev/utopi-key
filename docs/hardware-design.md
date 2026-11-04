# Hardware Design — utopi-key

## Design Philosophy

Start from the official Raspberry Pi RP2350A Minimal reference design and adapt it into a compact USB security key form factor. Prioritize:

1. **Correctness** of power, clock, and USB (follow official guidance closely)
2. **Manufacturability** in Quebec (standard 2-layer or 4-layer, common packages)
3. **Openness** and auditability
4. **Small size** while remaining hand-assemblable for prototypes

## Block Diagram

```
[USB-A] --5V--> [LDO 3.3V] --+--> [RP2350A IOVDD / USB / QSPI]
                             |
                             +--> [Internal Switcher] --1.1V--> [DVDD]
                                      |
                                      +-- 3.3µH inductor

[RP2350A]
  - USB_DP/DM --27Ω--> USB D+/D-
  - QSPI --─────────> [W25Q128 Flash]
  - XIN/XOUT -----> [12 MHz Crystal + 15pF]
  - GPIO ---------> [User Button]
  - QSPI_SS -------> [BOOTSEL Button]
  - Optional GPIO -> [LED]
```

## Key Circuits (Textual Schematic)

### 1. Power Supply

```
USB_VBUS (5V) ----+---- C_in (10µF) ---- GND
                  |
                  +---- LDO_IN (NCP1117-3.3)
                         |
                         +---- LDO_OUT (3.3V) ----+---- C_out (10µF)
                                                  |
                                                  +---- to all 3.3V rails

Internal 1.1V regulator (critical layout):
VREG_VIN ---- C6 (4.7µF) ---- GND
VREG_LX  ---- L1 (3.3µH) ----+---- C7 (4.7µF) ---- DVDD
                             |
VREG_FB  --------------------+
VREG_PGND ---- (star to main GND, short)
VREG_AVDD ---- R3 (33Ω) ----+---- C9 (4.7µF) ---- 3.3V
```

**Layout notes**: Follow Raspberry Pi guidance exactly for the switcher. Keep the high-current loop (VIN → LX → inductor → output cap → PGND) as small as possible.

### 2. Crystal

```
XIN ----+---- C_load (15pF) ---- GND
        |
        +---- Crystal (ABM8-272-T3) ----+---- XOUT
                                        |
                                        +---- C_load (15pF) ---- GND
```

Place crystal and caps very close to the RP2350.

### 3. USB

```
USB_D+ ---- R (27Ω) ---- USB_DP (RP2350)
USB_D- ---- R (27Ω) ---- USB_DM (RP2350)
```

Place series resistors close to the MCU. Add optional TVS (e.g. PESD5V0L1BA or similar) on VBUS/D+/D- for ESD.

### 4. Flash + BOOTSEL

```
QSPI_SS ----+---- Flash_CS
            |
            +---- R (1k) ---- Button ---- GND   (BOOTSEL)

QSPI_SCLK / SD0 / SD1 / SD2 / SD3 ---- Flash
Flash_VCC ---- 3.3V + 100nF
```

### 5. Buttons

- **User Presence**: GPIO (with internal pull-up) ← Button ← GND
- **BOOTSEL**: as above (required for initial UF2 flashing)

## Recommended Starting Point

1. Download the official Minimal KiCad design:  
   https://datasheets.raspberrypi.com/rp2350/Minimal-KiCAD.zip (or search for current RP2350A Minimal)
2. Copy the power, crystal, flash, and USB sections.
3. Remove the large GPIO headers.
4. Add USB-A edge connector (or discrete plug) and user button.
5. Shrink the board to stick form factor.
6. Add mounting features / case alignment holes.

## PCB Recommendations

- Layers: 2-layer for prototype, 4-layer preferred for better GND
- Thickness: 1.6 mm (standard) or 2.0 mm for better USB insertion feel
- Finish: ENIG (better for USB contacts)
- Impedance: USB differential ~90 Ω (not critical on short runs)

## Future Enhancements

- NFC antenna + controller (like Nitrokey 3 / Solo 2+)
- Dedicated secure element (e.g. for higher certification)
- Capacitive touch instead of mechanical button
- Metal enclosure option
- USB-C version

---

This document will be updated as the KiCad schematic is created.
