# Quebec Manufacturing & Sourcing Notes

## Local PCB / PCBA Partners (as of late 2026)

### Assembly & Turnkey
- **R.S. Electronique** (Montreal) — https://www.rspcbassembly.com  
  Turnkey PCBA, quick-turn prototypes, component procurement.
- **M2S Électronique** (Québec City) — https://m2selectronique.com  
  Full SMT, test, ISO certified.
- **Cancino Technologies** (Montreal) — https://www.cancinotec.com  
  Design + assembly.
- **Fabrique Manic** (St-Augustin-de-Desmaures, near Québec City)  
  Prototype-focused SMT.

### PCB Fabrication
- Qualieco Circuits (Canada-wide, serves Quebec)
- Local houses via the above assemblers
- For first prototypes: JLCPCB / PCBWay + Canadian shipping is often faster/cheaper

## Component Distributors (Canada)

- **FM Electroniques** (Montreal) — broad line card, good for hard-to-find parts
- Digi-Key Canada, Mouser Canada
- Arrow, Avnet
- LCSC (China) for RP2350 and passives — excellent price, ships to Canada

## Recommended Prototype Flow

1. Finalize KiCad + Gerbers + BOM (with LCSC or Digi-Key parts)
2. Order 5–10 PCBs from JLCPCB (or local) with optional SMT for passives
3. Hand-solder or use local PCBA for the RP2350 (QFN is tricky by hand)
4. Flash UF2 firmware via BOOTSEL
5. Test FIDO2 on webauthn.io / GitHub / etc.
6. Iterate enclosure (3D print)
7. Engage Quebec PCBA house for larger runs

## Cost Targets (rough, per unit at 100 pcs)

- PCB + components: ~$8–15 CAD
- Assembly: ~$5–10
- Enclosure: ~$2–5 (3D print) or less at volume
- Total BOM+assembly goal: under $25 CAD before packaging/certification

## Notes

- RP2350A is available via LCSC (search C42411118 or similar).
- Prefer RoHS parts.
- Document DFM rules early (clearances, thermal vias under QFN, etc.).
