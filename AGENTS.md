# OpenRX Mono

Open-source dual-band ExpressLRS receiver. An ESP32-C3 controls one LR1121;
2.4 GHz and sub-GHz RF paths share one U.FL antenna through an RF switch. A
separate ceramic antenna serves the ESP32-C3 Wi-Fi interface.

## Architecture

The ESP32-C3 (U1) runs ExpressLRS, communicates over UART0/CRSF, and drives the
WS2812B status LED (D1). It controls the LR1121 (U3) over SPI. At 2.4 GHz,
`RFIO_HF` passes through FL1, the RFX2401C PA/LNA (U4), and SKY13373 switch (U5)
to U.FL J1. Sub-GHz transmit uses the LR1121 high-power output through T1 and
U5; receive returns through U5 and T1 to `RFI_P/N_LF`. T1's low-power TX port is
unused. LR1121 DIO5/DIO6 drive U4 RXEN/TXEN and DIO7/DIO8 drive U5 V1/V2.

## Power

```text
5V pad (TP3)
└── TLV75533PDQNR (U2), 3.3 V
    ├── ESP32-C3 (U1) and status LED (D1)
    ├── LR1121 (U3) and 32 MHz TCXO (OSC1)
    └── RFX2401C (U4) and SKY13373 (U5)
```

## Key parts

| Function | Ref | Part | LCSC | Note |
|---|---|---|---|---|
| MCU | U1 | ESP32-C3 | | QFN-32 |
| 3.3 V LDO | U2 | TLV75533PDQNR | C2861882 | 500 mA |
| Dual-band radio | U3 | LR1121IMLTRT | C7498014 | |
| 2.4 GHz PA/LNA | U4 | RFX2401C | C19213 | |
| RF switch | U5 | SKY13373-460LF | C150853 | Shared antenna |
| Sub-GHz IPD | T1 | 0900PC16J0042001E | C19842466 | |
| 2.4 GHz filter | FL1 | 2450FM07D0034T | C2651081 | |
| ELRS connector | J1 | U.FL-R-SMT-1(80) | C88374 | |
| Wi-Fi antenna | AE1 | 2450AT18A100E | C89334 | ESP32-C3 only |
| Radio TCXO | OSC1 | OW7EL89CENUYO3YLC-32M | C22381772 | 32 MHz |
| MCU crystal | X1 | CJ17-400001010B20 | C2875272 | 40 MHz |
| Status LED | D1 | XL-1010RGBC-WS2812B | C5349953 | |

## Connectors and I/O

| Pad | Net | ESP32-C3 | Function |
|---|---|---|---|
| RX (TP1) | U0RXD | GPIO20 | CRSF serial input |
| TX (TP2) | U0TXD | GPIO21 | CRSF telemetry output |
| 5V (TP3) | +5V | - | Supply input |
| GND (TP4) | GND | - | Ground |
| BOOT (TP5) | BOOT | GPIO9 | UART download mode when held low at power-up |

Radio SPI is SCK/MOSI/MISO on GPIO6/4/5; NSS/RST/BUSY/DIO1 are GPIO7/2/3/1.
The RGB status LED is GPIO8 in GRB order.

## Layout rules

Preserve both controlled-impedance RF paths, the ground-via fences around the
front end, and the Wi-Fi antenna keepout. Keep the LR1121, IPD, PA/LNA, RF
switch and U.FL interconnects short; do not disturb the exposed-pad via arrays.

## Firmware

ExpressLRS target `Unified_ESP32C3_LR1121_RX`, platform `esp32-c3`, minimum
version 3.5.0. Upload methods are UART, Wi-Fi and Betaflight passthrough. The
authoritative GPIO, RF-switch table and power values are in
`firmware/OpenRX Mono LR1121.json`.

## Repo

| | |
|---|---|
| Maintainer | @bastian2001 |
| KiCad project | `hardware/OpenRX-Mono.kicad_pro` |
| Root schematic | `hardware/OpenRX-Mono.kicad_sch`; radio/core sheet `hardware/esp32c3_lr1121_mono.kicad_sch` |
| Board | `hardware/OpenRX-Mono.kicad_pcb`, 6 layers, 1.0 mm |
| Local library | `hardware/lib.pretty/` and `hardware/lib.3dshapes/`, nickname `OpenRX-Shared` retained by the design; `shared/libs/OpenRX-Shared.3dshapes` is a compatibility link for embedded legacy model paths |
| Design rules | `hardware/OpenRX-Mono.kicad_dru` |
| Firmware target | `firmware/OpenRX Mono LR1121.json` and `firmware/targets_entries.json` |

## Rules

The shared [Board rules](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules) apply. Only what differs for this board is listed here.

## Revisions

| Rev | Date | Change |
|---|---|---|
| rev2 | 2026-08-14 | Fab set re-exported and verified; small batch ordered. |
| rev1 | 2026-03-23 | First released OpenRX Mono design. |

