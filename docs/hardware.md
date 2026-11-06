# T-Display K230 hardware audit

## Purchase status

Defer the purchase until the radio SKU and PCB revision are matched to the schematic and Linux image. For a 915 MHz SX1262 bench test, the candidate is the separately listed **T-Display K230 915 MHz, K256-A**. Confirm that it ships with an SX1262, a matching antenna, and a board revision supported by the Linux BSP below. Confirm fit and electrical compatibility separately if ordering the cased keyboard base.

The cased kit lists **K256-03** as SX1262/868 MHz and **K277** as LR2021/830–940 MHz. K277 uses a different transceiver and would change the driver scope. Match the radio band to the intended MeshCore peer. The kit lists the keyboard as a separate option.

Sources: [cased kit variants](https://lilygo.cc/products/t-display-k230-kit.js), [standalone variants](https://lilygo.cc/products/t-display-k230.js), [cased kit shipping list](https://lilygo.cc/products/t-display-k230-kit).

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

## Other board interfaces

- The schematic shows a K230 and LPDDR4. Confirm installed RAM for the ordered revision. The Linux BSP maps SD boot/storage to GPIO54–59. Confirm the correct image and recovery steps for that revision.
- The BSP identifies the RM69A10 AMOLED on MIPI DSI with GPIO22 reset and GPIO25 enable. GT9895 touch uses I2C on GPIO36/37, with GPIO23 IRQ and GPIO24 reset.
- The optional keyboard base uses a TCA8418 at I2C address 0x34, GPIO42 IRQ, GPIO43 reset and GPIO52/PWM4 backlight. The BSP also maps BQ25896 charger and BQ27220 battery gauge support on the base. Confirm battery inclusion and the hardware populated on the selected keyboard variant.

Source: [Linux BSP hardware pin map](https://github.com/Xinyuan-LilyGO/T-Display-K230/blob/c27c58c07ca5e39d15f5dcf1b7103aead29282df/k230_bsp/docs/HARDWARE_PINMAP.md).

## Bench checks after SKU confirmation

1. Record PCB and module markings, boot the matching Linux image and verify the radio device and GPIO lines.
2. Verify radio power, reset and BUSY behavior, then initialize the SX1262 and check DIO1 on packet events.
3. Verify packet transmit and receive with a known-good peer on matching settings. MeshCore interoperability is a separate test tracked in [issue #6](https://github.com/traft00/meshcore-k230/issues/6).
