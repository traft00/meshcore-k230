# T-Display K230 hardware audit

## Purchase status

Defer the purchase until the radio SKU and PCB revision are matched to the schematic and Linux image. For a 915 MHz SX1262 bench test, the candidate is the separately listed **T-Display K230 915 MHz, K256-A**. Confirm that it ships with an SX1262, a matching antenna, and a board revision supported by the Linux BSP below. Confirm fit and electrical compatibility separately if ordering the cased keyboard base.

The cased kit lists **K256-03** as SX1262/868 MHz and **K277** as LR2021/830–940 MHz. K277 uses a different transceiver and would change the driver scope. Match the radio band to the intended MeshCore peer. The kit lists the keyboard as a separate option.

Sources: [cased kit variants](https://lilygo.cc/products/t-display-k230-kit.js), [standalone variants](https://lilygo.cc/products/t-display-k230.js), [cased kit shipping list](https://lilygo.cc/products/t-display-k230-kit).

LILYGO's own Spark device manifest identifies the standalone T-Display K230 as SX1262 across all four listed bands, including 915 MHz, and links directly to the storefront where 915 MHz maps to K256-A. This is strong evidence that K256-A is specified with an SX1262. The manifest describes the product family and does not identify the PCB revision or production lot. The older CanMV RT repository also documents SX1262 support, but its RT firmware archives cannot establish compatibility with the current Linux SD image.

Sources: [LILYGO Spark device manifest](https://github.com/Xinyuan-LilyGO/LILYGO-Spark/blob/a88415a33a699dbdf64d90f5d38263f549d53017/firmware_manifest.json), [CanMV RT revision history](https://github.com/Xinyuan-LilyGO/T-Display-K230_canmv_rt/blob/main/README.md).

As of 2026-11-06, LILYGO's latest Linux SD image is **v0.2.9**. The launcher source at that release tag includes SX1262 detection and a factory 915 MHz profile. This confirms software support for the documented SX1262 design. The store data identifies K256-A by band only, and the release does not identify a tested K256-A PCB revision. Ask LILYGO to confirm the radio module and PCB revision in current K256-A stock and that it boots v0.2.9 and detects the onboard radio.

Sources: [v0.2.9 Linux image](https://github.com/Xinyuan-LilyGO/T-Display-K230/releases/tag/v0.2.9), [v0.2.9 radio source](https://github.com/Xinyuan-LilyGO/T-Display-K230/blob/v0.2.9/k230_launcher/k230_phone_ui/src/ui_lora.cpp).

## SX1262 wiring

The LoRa sheet in LILYGO's V1.0 schematic, dated 2026-06-26, agrees with the vendor Linux pin map and launcher configuration. These are K230 SoC GPIO numbers. U26 is the radio module footprint.

| Signal | K230 GPIO | Schematic path |
| --- | --- | --- |
| SPI chip select | 14 | R106 to U26 pin 13 |
| SPI clock | 15 | R107 to U26 pin 12 |
| SPI MOSI | 16 | R109 to U26 pin 11 |
| SPI MISO | 17 | R111 to U26 pin 10 |
| Radio reset | 5 | R105 to U26 pin 14 |
| BUSY | 19 | R113 to the module BUSY selection |
| SX1262 DIO1 interrupt | 20 | Module IRQ selection via R108/R110 |
| Radio power enable | 44 | LoRa_EN to U27 RT9080-33 enable, with R115 pull-down |

U27 generates Radio_3V3 from VDD_5V. The radio sheet routes U26's antenna output to the RF3 connector through a matching network. U26 supports several module pinouts. The selection resistors and module population on a shipped board need to match the SX1262 path.

The Linux device-tree patch enables SPI0 CS0 with a 4 MHz ceiling. The launcher opens `/dev/spidev0.0`, drives radio enable high, and passes DIO1, reset and BUSY to RadioLib. The vendor SX1262 initialization requests 3.3 V TCXO control and the bundled RadioLib initialization enables DIO2 RF-switch control. Those are software settings to validate against the actual module, including oscillator and RF switch population.

Sources: [V1.0 schematic, LoRa sheet](https://github.com/Xinyuan-LilyGO/T-Display-K230_canmv_rt/blob/abb07090ad8a666ed7a5e097b3c714b918731645/schematic/T-Display%20K230_V1.0_NEW.pdf), [SPI device-tree patch](https://github.com/Xinyuan-LilyGO/T-Display-K230/blob/c27c58c07ca5e39d15f5dcf1b7103aead29282df/k230_bsp/overlay/buildroot-overlay/linux/0054-riscv-dts-rm69a10-enable-lora-spidev.patch), [launcher radio code](https://github.com/Xinyuan-LilyGO/T-Display-K230/blob/c27c58c07ca5e39d15f5dcf1b7103aead29282df/k230_launcher/k230_phone_ui/src/ui_lora.cpp), [bundled RadioLib SX126x initialization](https://github.com/Xinyuan-LilyGO/T-Display-K230/blob/c27c58c07ca5e39d15f5dcf1b7103aead29282df/k230_launcher/k230_phone_ui/src/third_party/RadioLib/src/modules/SX126x/SX126x.cpp).

## CanMV RT repository history

The CanMV RT repository ships separate V1.2 and V1.3 RT-Smart SD images. The checked-in LoRa demo selects LR2021 by default, with a compile-time SX1262 branch that initializes at 915 MHz with a 3.3 V TCXO and BUSY on GPIO19. The RT branch passes no DIO1 GPIO to RadioLib. This is source evidence of a vendor 915 MHz SX1262 configuration, without a test record for the K256-A or Linux v0.2.9.

The bundled HPD16A datasheet lists two high-band HPD16A_89 SX1262 variants covering 830–945 MHz, one with a TCXO and one with a standard crystal. The datasheet does not identify which module is populated on K256-A. The historical schematic and current schematic both carry V1.0 on the LoRa sheet, but differ in the radio circuit. The older sheet shows the radio supplied by Radio_3V3 and no local U27 regulator/enable circuit, while the June 2026 sheet explicitly shows the GPIO44-enabled U27 path. This makes the schematic date and PCB marking important alongside the revision label.

Sources: [CanMV RT README](https://github.com/Xinyuan-LilyGO/T-Display-K230_canmv_rt/blob/abb07090ad8a666ed7a5e097b3c714b918731645/README.md), [RT radio demo](https://github.com/Xinyuan-LilyGO/T-Display-K230_canmv_rt/blob/abb07090ad8a666ed7a5e097b3c714b918731645/canmv_k230/src/rtsmart/mpp/userapps/sample/sample_display/ui_brookesia/src/app_examples/phone/lora/Lora.cpp), [historical schematic](https://github.com/Xinyuan-LilyGO/T-Display-K230_canmv_rt/blob/ed19e87f/schematic/T-Display%20K230%20V1.0.pdf), [module datasheet directory](https://github.com/Xinyuan-LilyGO/T-Display-K230_canmv_rt/tree/abb07090ad8a666ed7a5e097b3c714b918731645/datasheet).

## Other board interfaces

- The schematic shows a K230 and LPDDR4. Confirm installed RAM for the ordered revision. The Linux BSP maps SD boot/storage to GPIO54–59. Confirm the correct image and recovery steps for that revision.
- The BSP identifies the RM69A10 AMOLED on MIPI DSI with GPIO22 reset and GPIO25 enable. GT9895 touch uses I2C on GPIO36/37, with GPIO23 IRQ and GPIO24 reset.
- The BSP describes a TCA8418 keyboard at I2C address 0x34 on SDA47/SCL46, with IRQ42, reset43 and backlight52. The wiki's cased keyboard drawing instead labels SDA33/SCL32, IRQ46, reset43 and backlight47. The drawing also identifies BQ25896 charging, BQ27220 fuel gauging and an empty 21700 holder. Match the base PCB revision before relying on either pin map. An 868 MHz kit owner reported the mismatch under v0.2.8 in [vendor issue #10](https://github.com/Xinyuan-LilyGO/T-Display-K230/issues/10). See the [wiki keyboard drawing](https://wiki.lilygo.cc/products/t-display-series/t-display-k230/index/image/t-display-k230-keyboard-extension.jpg).

Source: [Linux BSP hardware pin map](https://github.com/Xinyuan-LilyGO/T-Display-K230/blob/c27c58c07ca5e39d15f5dcf1b7103aead29282df/k230_bsp/docs/HARDWARE_PINMAP.md).

The wiki's original V1.0 photo and cased pin map both corroborate the eight host-side radio lines above. The original 40-pin diagram additionally labels GPIO3 as LoRa TCXO enable and GPIO4 as LoRa IRQ, separately from GPIO20 DIO1. The schematic shows these extra nets at unpopulated selection resistors R103 and R104. Treat them as module-dependent and follow the SX1262 launcher and schematic path for GPIO20.

Sources: [original pin map](https://wiki.lilygo.cc/products/t-display-series/t-display-k230/index/image/t-display-k230-cn.jpg), [40-pin diagram](https://wiki.lilygo.cc/products/t-display-series/t-display-k230/index/image/t-display-k230-40pin.jpg), [cased pin map](https://wiki.lilygo.cc/products/t-display-series/t-display-k230/cased-version/image/t-display-k230-cased-pinmap.jpg).

## MeshCore installation

The MeshCore web flasher’s default device catalog has no T-Display K230 entry as of 2026-11-06. Its similarly named T-Display Pro entry is an ESP32 target. Use the matching LILYGO Linux image as the starting point for the K230 feasibility work.

Sources: [MeshCore web flasher catalog](https://flasher.meshcore.io/config.json), [LILYGO Linux releases](https://github.com/Xinyuan-LilyGO/T-Display-K230/releases).

## Bench checks after SKU confirmation

1. Record PCB and module markings, boot the matching Linux image and verify the radio device and GPIO lines.
2. Verify radio power, reset and BUSY behavior, then initialize the SX1262 and check DIO1 on packet events.
3. Verify packet transmit and receive with a known-good peer on matching settings. MeshCore interoperability is a separate test tracked in [issue #6](https://github.com/traft00/meshcore-k230/issues/6).
