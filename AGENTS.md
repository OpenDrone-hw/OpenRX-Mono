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

EU build (target key `mono_eu`): `firmware/OpenRX Mono LR1121 EU.json`, the same
pins with one power level per band: 2.4 GHz chip setting -17 dBm into the
RFX2401C (5 dBm conducted), 868 MHz chip setting 12 dBm (10 dBm conducted).
Built with `Regulatory_Domain_EU_CE_2400` and domain `eu_868`. Both conducted
levels are to be confirmed by measurement.

## Repo

| | |
|---|---|
| Maintainer | @bastian2001 |
| KiCad project | `hardware/OpenRX-Mono.kicad_pro` |
| Root schematic | `hardware/OpenRX-Mono.kicad_sch`; radio/core sheet `hardware/esp32c3_lr1121_mono.kicad_sch` |
| Board | `hardware/OpenRX-Mono.kicad_pcb`, 6 layers, 1.0 mm |
| Local library | `hardware/lib.pretty/` and `hardware/lib.3dshapes/`, nickname `OpenRX-Shared` retained by the design; `shared/libs/OpenRX-Shared.3dshapes` is a compatibility link for embedded legacy model paths |
| Shared library | `hardware/KiCad-Library/`, submodule of [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), nickname `OpenDrone`; exact component datasheets resolve through the project text variable `OPENDRONE_LIB` |
| Design rules | `hardware/OpenRX-Mono.kicad_dru` |
| Firmware target | `firmware/OpenRX Mono LR1121.json`, `firmware/OpenRX Mono LR1121 EU.json` and `firmware/targets_entries.json` |

## Parts and datasheets

- **Per-repository part index:** `hardware/OpenRX-Mono.kicad_sch` and its
  listed sub-sheet are authoritative for what this board fits. Export the
  netlist with the command below when a script-readable board index is needed;
  do not maintain a second hand-written BOM.
- **Proven shared parts:**
  `hardware/KiCad-Library/PARTS-USED.md` is the catalogue index. Its `Boards`
  column identifies every repository using each LCSC part; filter it for
  `OpenRX-Mono` to get this repository's proven shared-part view.
- **Local-only parts:** inspect `hardware/lib.pretty/` and
  `hardware/lib.3dshapes/`, then verify supplier fields in the board
  schematic. Do not duplicate a shared part or datasheet.

## Environment

```sh
kicad-cli sch erc hardware/OpenRX-Mono.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones hardware/OpenRX-Mono.kicad_pcb
kicad-cli sch export netlist --format kicadsexpr -o /tmp/OpenRX-Mono.net hardware/OpenRX-Mono.kicad_sch
```

This board has no board-specific scripts; any it gets go in a `tools` folder under `hardware`.

## Rules

The [Board rules](#board-rules) at the end of this file apply, a CI-checked copy of the [hardware-template](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules) section. Only what differs for this board is listed here.

## Revisions

| Rev | Date | Change |
|---|---|---|
| rev2 | 2026-08-14 | Fab set re-exported and verified. |
| rev1 | 2026-03-23 | First released OpenRX Mono design. |

## By task

- Render the board for the README: `$KPY <hardware-tooling>/hardware/kicad/render_board.py hardware/OpenRX-Mono.kicad_pcb --outdir images`, KiCad closed.

## Board rules

Shared by every OpenDrone KiCad board repository, byte for byte, and checked
in CI against [hardware-template](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules).
Edit them there, never in a board.

### Files

Every board has this layout; its Repo table lists only what differs.

| | |
|---|---|
| Designed in | KiCad 10 |
| Status | The `status-*` topic on the repository, never written in a file |
| Local library | `hardware/lib.kicad_sym`, `hardware/lib.pretty/`, `hardware/lib.3dshapes/`, nickname `lib` |
| Shared library | `hardware/KiCad-Library/`, submodule of [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), nickname `OpenDrone`; 3D models and exact component datasheets resolve through the project text variable `OPENDRONE_LIB` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| Board setup | 6 layers, 0.09 mm clearance and track, via 0.35 on 0.20 drill |
| License | CERN-OHL-S-2.0 |

A repository holds design files only. Sourcing (suppliers, prices, quotes, RFQs, contacts) is handled by Incutec and never lives in OpenDrone repositories.

### Parts and datasheets

- **Per-repository part index:** the root schematic and its listed sub-sheets
  are authoritative for what this board fits. Export the netlist with the
  command below when a script-readable board index is needed; do not maintain
  a second hand-written BOM.
- **Proven shared parts:**
  `hardware/KiCad-Library/PARTS-USED.md` is the catalogue index. Its `Boards`
  column identifies every repository using each LCSC part; filter that column
  for the repository name to get its proven shared-part view.
- **Exact datasheets:**
  `hardware/KiCad-Library/datasheet/manifest.json` maps shared symbols to the
  committed PDFs and their SHA-256 hashes. The PDFs live in
  `hardware/KiCad-Library/datasheet/`, and symbol links resolve there through
  `OPENDRONE_LIB`.
- **Local-only parts:** inspect `hardware/lib.kicad_sym`,
  `hardware/lib.pretty/`, and `hardware/lib.3dshapes/`, then verify supplier
  fields in the board schematic. Do not duplicate a shared part or datasheet.

### Commands

`<name>` is the KiCad project named in the board's Repo table.

```sh
# schematic and board checks
kicad-cli sch erc hardware/<name>.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones hardware/<name>.kicad_pcb

# netlist, for scripted analysis
kicad-cli sch export netlist --format kicadsexpr -o /tmp/<name>.net hardware/<name>.kicad_sch
```

On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`, and `pcbnew` imports
only under KiCad's bundled Python, `KPY`:
`/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3`.
Reusable scripts for renders, STEP export and packaging art come from Incutec
hardware tooling. The OpenDrone release standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).
Board-specific scripts, where a board has any, live in `hardware/tools/`.

### Rules

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
  kicad-skip cannot round-trip KiCad 10 schematics: use a full S-expression
  parser, `kicad-cli sch upgrade --force`, then an ERC and netlist diff.
- **Metadata yes, connections no.** An agent may write BOM and documentation
  fields (MPN, Manufacturer, LCSC, Cost, Datasheet, text variables). An agent
  may not change nets, wiring, routing, placement, footprint assignment, or any
  value that changes the circuit.
- **Close KiCad before any write to a KiCad file.** KiCad caches library tables
  at process start and overwrites files on save.
- **Reuse before you draw.** Check the `OpenDrone` library and its
  `PARTS-USED.md` first. If the part is there we have already sourced,
  footprinted and shipped it, and its symbol links to the exact committed
  datasheet: place it from `OpenDrone`. Draw a new part into `lib` only when
  the catalogue has nothing that fits, imported with
  `easyeda2kicad` from its LCSC number. Pulling a newer catalogue is a
  deliberate, reviewed commit: `git submodule update --remote
  hardware/KiCad-Library`, then DRC.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **Run ERC and DRC before every pull request.** Existing approved findings
  may remain; a new type or increased count must be reviewed before merge.
  The commands are under Commands.

### By task

- Check the design: run the ERC and DRC commands in Commands before every pull request.
- Add a part: place it from the `OpenDrone` library if `hardware/KiCad-Library/PARTS-USED.md` lists it; otherwise import it into `lib` with `$KPY <hardware-tooling>/hardware/kicad/import_part.py` (read `--help` first), KiCad closed.
- Render the board for the README: `$KPY <hardware-tooling>/hardware/kicad/render_board.py hardware/<name>.kicad_pcb --outdir images`, KiCad closed.
- Analyse the netlist: export it with the netlist command in Commands, then read it with a script; never hand-write a second BOM.
- Update the shared library: `git submodule update --remote hardware/KiCad-Library`, run DRC, commit as its own reviewed change.
