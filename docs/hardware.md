# Hardware audit

Target the LILYGO T-Display K230. Verify the exact SKU, board revision, and radio band against vendor documentation before selecting a unit.

- Identify the SoC, available memory and storage, boot media, and recovery mechanism.
- Trace the SX1262 SPI bus, chip select, reset, BUSY and interrupt signals. Check RF switch, oscillator control, power enable and antenna requirements against the schematic.
- Identify the display, touch, keyboard, battery charging and power control interfaces. Map each to its Linux driver or userspace interface where possible.
- Record schematic and SDK revisions together so pin assignments can be checked against the correct board files.

Board-level compatibility remains an open question. Add evidence and unresolved items here as the audit progresses.
