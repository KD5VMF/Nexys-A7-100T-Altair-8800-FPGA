<div align="center">

# MILO Altair 8800 FPGA

### A hardware-verified Intel 8080 / MITS Altair 8800 computer for the Digilent Nexys A7-100T

**Stage 7.5 · WRITE-FIX-3 · GOLD**

[![FPGA](https://img.shields.io/badge/FPGA-Nexys%20A7--100T-0A66C2?style=for-the-badge)](#hardware)
[![CPU](https://img.shields.io/badge/CPU-Intel%208080%20Compatible-6E40C9?style=for-the-badge)](#architecture)
[![OS](https://img.shields.io/badge/OS-CP%2FM%202.2-2E8B57?style=for-the-badge)](#what-it-does)
[![Release](https://img.shields.io/badge/Release-Stage%207.5%20GOLD-D4AF37?style=for-the-badge)](#gold-release)
[![UART](https://img.shields.io/badge/UART-115200%208N1-555?style=for-the-badge)](#console)
[![Boot](https://img.shields.io/badge/Boot-QSPI%20SPIx4-008080?style=for-the-badge)](#boot-flow)

**A real FPGA appliance, not a software emulator.**  
Power it on, let the authentic MITS-style disk bootstrap execute at 2 MHz, and arrive at CP/M on a serial terminal.

[Quick Start](docs/QUICK_START.md) ·
[Complete Manual](docs/MILO-Altair8800-Stage7.5-Complete-Manual.pdf) ·
[Architecture](docs/ARCHITECTURE.md) ·
[Build & Flash](docs/BUILD_AND_FLASH.md) ·
[Recovery](docs/RECOVERY.md)

</div>

---

![MILO Altair 8800 FPGA Architecture](docs/images/architecture.png)

## What this project is

**MILO Altair 8800 FPGA** turns a Digilent **Nexys A7-100T** into a dedicated Intel 8080 / Altair-style computer with a hardware CPU core, FPGA RAM, MITS-compatible console and disk interfaces, persistent microSD storage, autonomous QSPI configuration, and a live address-bus display on the board LEDs.

The design is intentionally appliance-like:

> **Power on → initialize storage → validate the boot disk → run the MITS DBL path → boot CP/M → use the machine.**

No host PC is required after the FPGA and microSD have been prepared.

This repository freezes the **hardware-validated Stage 7.5 / WRITE-FIX-3 gold release**. The source, prebuilt FPGA images, SD image, reports, documentation, recovery tools, and published hashes are kept together so the exact working machine can be reproduced and preserved.

---

## Highlights

- **Intel 8080-compatible CPU in FPGA logic** using the `vm80a` core.
- **Authentic 2 MHz bootstrap behavior** for the MITS disk boot path.
- **Fixed 8 MHz production runtime** after CP/M begins console output.
- **56K CP/M 2.2 environment**.
- **64 KB FPGA RAM** with protected high-memory boot/PROM region.
- **MITS 88-2SIO-compatible serial console**.
- **MITS 88-DCDD-compatible disk controller**.
- **Native Nexys A7 microSD storage** over SPI.
- **Persistent CP/M writes** verified on physical hardware.
- **Autonomous QSPI SPIx4 boot** from the board's S25FL128S configuration flash.
- **Live A0-A15 address-bus display** on the 16 red LEDs.
- **Blue RGB disk/activity indicator**.
- **Ready-to-write 80 MiB gold SD image**.
- **Prebuilt `.bit` and `.mcs` files** for immediate hardware use.
- **Batch Vivado build and QSPI programming flows**.
- **Full recovery, validation, troubleshooting, and provenance documentation**.
- **SHA256-locked gold artifacts** so a known-good release cannot be confused with an experimental build.

---

# Gold release

## Stage 7.5 / WRITE-FIX-3

Stage 7.5 is the first release in this project where normal CP/M file creation and persistence were validated on a physical Nexys A7-100T using ordinary CP/M tools including **PIP** and **ED**.

The release solves three independent write-path problems.

| Fix | Problem | Resolution |
|---|---|---|
| **WRITE-FIX-1** | Safe STEP/head commands could be lost while an SD `CMD24` write commit was still finishing. | Mechanical controls remain accepted; only a conflicting new write is blocked. |
| **WRITE-FIX-2** | Tracks 2-5 were accidentally treated as immutable even though CP/M directory data lives there. | Six boot tracks remain readable from embedded material, but only tracks 0-1 are write-protected. |
| **WRITE-FIX-3** | System-format tracks produce 134 physical bytes while normal data tracks produce 137; older RTL waited for 137 everywhere. | Expected write length is selected from the active track region: **134 bytes on tracks 0-5, 137 bytes on tracks 6+**. |

### Hardware proof

The gold build successfully completed normal CP/M create/close/read workflows such as:

```text
A0>PIP TEST.TXT=CON:
MILO WRITE FIX TEST
^Z

A0>DIR TEST.TXT
A: TEST TXT

A0>ED HI.TXT
NEW FILE
*i
Hello World!
*e

A0>TYPE HI.TXT
Hello World!
```

The important part is not the text itself. These operations exercise the **real CP/M directory update and persistent disk-write path** rather than a synthetic raw-sector test.

See [WRITE_FIX_3.md](docs/WRITE_FIX_3.md) for the full technical explanation.

---

# Gold identity

These four SHA256 values define the authoritative release.

| Artifact | SHA256 |
|---|---|
| `stage7/src/altair_dcdd_sd.sv` | `6765a46ece31864903d664d7952efc5c9e5678a16104a5322c85d07bd481fece` |
| `release/milo-altair8800-stage7-final.bit` | `68deae429c52cf4711baa7ad417bd6970961beb3d8a0fd33cfeb9b28e52cc828` |
| `release/milo-altair8800-stage7-final-qspi-x4.mcs` | `c22c47d6fe426312d4dfd9e2d4916c74cbf9bbe0e39b05958ad3fd59006e3085` |
| `release/MILO-ALTAIR8800-STAGE7.5-WRITE-FIX-3.img` | `34abb0d6c387f37304379c562c4abb5d5a563348677ea5aa34e44e5cbe6d42c7` |

Linux:

```bash
./tools/verify-gold.sh
```

Windows PowerShell:

```powershell
powershell -ExecutionPolicy Bypass -File .\tools\verify-release.ps1
```

### Source-of-truth rule

**Do not silently replace any of the four gold artifacts in place.**

Any HDL modification, rebuilt bitstream, regenerated MCS, or changed SD image is a **new build** and must receive new hashes and a new release identity.

---

# What it does

Once configured, the Nexys A7 behaves like a standalone Altair/CP/M appliance.

```text
POWER ON
   │
   ▼
FPGA loads automatically from QSPI
   │
   ▼
Power-on delay / reset sequencing
   │
   ▼
microSD initialization
   │
   ▼
A: boot-disk preflight
   │
   ▼
8080 released at 2 MHz
   │
   ▼
Authentic MITS DBL boot path
   │
   ▼
CP/M begins transmitting on 88-2SIO
   │
   ▼
Runtime switches to fixed 8 MHz
   │
   ▼
A0>
```

![Boot flow](docs/images/boot-flow.png)

There is **no active setup menu in the Stage 7.5 production profile**. Runtime is fixed at 8 MHz and persistent disk writes are enabled.

---

# Architecture

| Subsystem | Gold implementation |
|---|---|
| FPGA | Xilinx Artix-7 XC7A100T |
| Board | Digilent Nexys A7-100T |
| CPU | `vm80a` Intel 8080-compatible core |
| Bootstrap clock | 2 MHz |
| Production runtime | 8 MHz fixed |
| RAM | 64 KB FPGA RAM |
| CP/M environment | 56K CP/M 2.2 |
| Console | MITS 88-2SIO-compatible |
| Console ports | `10h` status / `11h` data |
| UART | 115200 baud, 8N1 |
| Disk controller | MITS 88-DCDD-compatible |
| Disk ports | `08h`, `09h`, `0Ah` |
| Storage | Native microSD over SPI |
| SD init clock | 250 kHz |
| SD run clock | 12.5 MHz |
| FPGA boot | S25FL128S QSPI, SPIx4 |
| Front panel | 16 red LEDs = sampled A0-A15 |
| Activity | RGB LED16 blue |
| 7-segment | Disabled intentionally |

The front-panel LEDs display a sampled view of the **actual address bus**. They are not a decorative counter.

---

# Storage

The gold system exposes four Altair-style disk slots.

| Drive | Base LBA | Geometry | Intended role |
|---|---:|---|---|
| **A:** | 2048 | 2048 tracks × 32 sectors | System / boot |
| **B:** | 67584 | 2048 tracks × 32 sectors | Programs / working storage |
| **C:** | 133120 | 77 tracks × 32 sectors | Legacy / games |
| **D:** | 137216 | 77 tracks × 32 sectors | Legacy / work |

![Storage map](docs/images/storage-map.png)

The supplied release image is exactly **80 MiB** and contains the known-good storage state for the gold release.

> Historical note: the byte-identical gold SD image originally existed on the development system under an older Stage 7.4 filename. Its SHA256 matched the Stage 7.5 gold identity exactly. The repository uses the correct Stage 7.5 release name while preserving the proven bytes.

See [STORAGE_LAYOUT.md](docs/STORAGE_LAYOUT.md).

---

# Hardware

## Required

- Digilent **Nexys A7-100T**
- microSD card of at least 80 MiB capacity
- USB connection for JTAG / FPGA programming
- USB-UART connection for the serial console
- Computer capable of running Vivado Hardware Manager for initial setup

## FPGA target

```text
XC7A100T-1CSG324C
```

The gold build was produced with **Vivado 2026.1**.

See [HARDWARE_PINOUT.md](docs/HARDWARE_PINOUT.md) for the exact board connections.

---

# Quick start

## 1. Verify the release

Linux:

```bash
./tools/verify-gold.sh
```

Do not proceed if a gold hash fails.

## 2. Write the microSD image

```bash
sudo ./tools/write-gold-sd.sh /dev/sdX
```

**This overwrites the selected device.**

The helper writes the complete 80 MiB image and performs readback verification.

## 3. JTAG-test the FPGA first

```bash
./tools/jtag-load-gold.sh
```

This loads the proven `.bit` temporarily. It does **not** alter permanent QSPI configuration.

## 4. Open the console

Use:

```text
115200 baud
8 data bits
no parity
1 stop bit
no flow control
```

## 5. Verify CP/M operation

Reach the `A0>` prompt and perform a normal file create/read test.

Example:

```text
A0>PIP TEST.TXT=CON:
TEST
^Z

A0>TYPE TEST.TXT
TEST
```

## 6. Program QSPI only after the JTAG test passes

```bash
./tools/flash-gold-qspi.sh
```

Wait for the programming flow to report verification success.

Then remove power, reconnect power, and confirm the board boots autonomously.

For the complete procedure, use [QUICK_START.md](docs/QUICK_START.md) and [BUILD_AND_FLASH.md](docs/BUILD_AND_FLASH.md).

---

# Console

The machine is UART-first by design.

```text
115200 8N1
```

The CP/M console is presented through an 88-2SIO-compatible interface:

```text
10h  status
11h  data
```

Stage 7.5 boots automatically. There is no normal requirement for VGA, USB keyboard, switches, or seven-segment interaction.

---

# Building from source

The exact gold development snapshot contains three `$readmemh` paths referencing:

```text
/home/milo/fpga/mits-altair8800/software/...
```

For the closest reproduction of the proven build, place the repository at:

```text
~/fpga/mits-altair8800
```

Then:

```bash
source /data/fpga/AMD/2026.1/Vivado/settings64.sh
cd ~/fpga/mits-altair8800
vivado -mode batch -source stage7/build/build.tcl
```

The build flow generates implementation reports and is designed to fail closed when timing is not acceptable.

### Important

A fresh rebuild will normally produce a **different binary hash** even when functionally equivalent. The supplied prebuilt BIT/MCS remain the authoritative hardware-validated gold artifacts.

---

# QSPI programming

The gold permanent FPGA image is:

```text
release/milo-altair8800-stage7-final-qspi-x4.mcs
```

Configuration flash:

```text
S25FL128S
SPIx4
```

Use:

```bash
./tools/flash-gold-qspi.sh
```

The underlying Vivado flow identifies the XC7A100T, attaches the S25FL128S configuration-memory device, erases it, programs it, and performs verification.

---

# Front-panel behavior

The Nexys A7 itself becomes the visible front panel.

### Red LEDs

```text
LED[15:0] = sampled 8080 A[15:0]
```

The display is sampled slowly enough to be visible to a human while leaving the CPU free to run normally.

### RGB LED

The blue RGB channel indicates disk/storage activity.

### Seven-segment display

Intentionally disabled in the production design.

The result is a clean, minimal machine: **address activity, disk activity, and a serial console**.

---

# Repository layout

```text
MILO-Altair8800-NexysA7/
│
├── README.md
├── RELEASE_NOTES.md
├── CHANGELOG.md
├── VERSION
├── SHA256SUMS.txt
├── SHA256SUMS-GOLD.txt
├── LICENSE-NOTICE.md
│
├── stage7/
│   ├── src/                 # Gold RTL + Nexys A7 constraints
│   └── build/               # Vivado build/JTAG/QSPI Tcl flows
│
├── software/
│   ├── RAM_INIT.mem
│   ├── CPM8_A_BOOT_ROM.mem
│   ├── STAGE7_BOOT_MENU.mem
│   └── bootrom/
│
├── release/
│   ├── milo-altair8800-stage7-final.bit
│   ├── milo-altair8800-stage7-final-qspi-x4.mcs
│   └── MILO-ALTAIR8800-STAGE7.5-WRITE-FIX-3.img
│
├── tools/
│   ├── verify-gold.sh
│   ├── verify-release.ps1
│   ├── write-gold-sd.sh
│   ├── jtag-load-gold.sh
│   ├── flash-gold-qspi.sh
│   └── build-gold.sh
│
├── reports/                 # Timing / DRC / utilization evidence
│
├── provenance/              # Original pull manifests and environment
│
└── docs/
    ├── MILO-Altair8800-Stage7.5-Complete-Manual.pdf
    ├── QUICK_START.md
    ├── ARCHITECTURE.md
    ├── BUILD_AND_FLASH.md
    ├── STORAGE_LAYOUT.md
    ├── WRITE_FIX_3.md
    ├── VALIDATION.md
    ├── RECOVERY.md
    ├── TROUBLESHOOTING.md
    ├── HARDWARE_PINOUT.md
    └── images/
```

---

# Documentation

| Document | Purpose |
|---|---|
| **[Complete Manual](docs/MILO-Altair8800-Stage7.5-Complete-Manual.pdf)** | Full project manual |
| **[Quick Start](docs/QUICK_START.md)** | Fast path from files to a running machine |
| **[Architecture](docs/ARCHITECTURE.md)** | CPU, memory, I/O, storage, and boot design |
| **[Build & Flash](docs/BUILD_AND_FLASH.md)** | Vivado build, JTAG, and QSPI procedures |
| **[Storage Layout](docs/STORAGE_LAYOUT.md)** | A/B/C/D geometry and SD layout |
| **[WRITE-FIX-3](docs/WRITE_FIX_3.md)** | Persistent-write failure analysis and solution |
| **[Validation](docs/VALIDATION.md)** | Gold hardware proof and acceptance checks |
| **[Recovery](docs/RECOVERY.md)** | Restoring FPGA and storage |
| **[Troubleshooting](docs/TROUBLESHOOTING.md)** | Field diagnosis |
| **[Hardware Pinout](docs/HARDWARE_PINOUT.md)** | Nexys A7 signals and connections |
| **[Source of Truth](docs/SOURCE_OF_TRUTH.md)** | Rules for preserving gold identity |

---

# Validation philosophy

The release is not considered valid merely because Vivado produces a bitstream.

A complete validation chain includes:

1. Gold hashes match.
2. SD image writes and reads back correctly.
3. JTAG BIT starts correctly.
4. UART reaches CP/M.
5. Normal CP/M file creation succeeds.
6. The created file can be read back.
7. QSPI programming verifies.
8. The machine boots after a cold power cycle.
9. The created CP/M file survives reboot.

That final persistence test covers the complete chain:

```text
8080 CPU
   ↓
CP/M BDOS/BIOS
   ↓
88-DCDD emulation
   ↓
sector write engine
   ↓
SD SPI controller
   ↓
microSD media
   ↓
cold reboot
   ↓
readback
```

---

# Known cold-start note

On the gold hardware, a cold start can occasionally appear stalled with the A0-A15 LEDs dark/static.

If that happens:

1. reset/reboot or power-cycle,
2. watch the address LEDs,
3. once they begin moving, allow the normal boot sequence to continue.

This is documented so a user does **not** unnecessarily rewrite the microSD or reflash QSPI because of a startup retry.

See [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md).

---

# Recovery

This repository deliberately includes multiple recovery layers:

- exact gold HDL,
- exact gold `.bit`,
- exact gold SPIx4 `.mcs`,
- exact gold 80 MiB SD image,
- build scripts,
- programming scripts,
- SHA256 manifests,
- implementation reports,
- original pull provenance,
- complete documentation.

The goal is simple:

> **Years from now, the machine should still be recoverable from the repository alone.**

See [RECOVERY.md](docs/RECOVERY.md).

---

# Project philosophy

This project intentionally favors a **known-good hardware appliance** over an endless collection of experimental builds.

The gold release is therefore treated like firmware:

- preserve it,
- hash it,
- document it,
- validate it on real hardware,
- never silently mutate it,
- branch new experiments away from it.

That makes Stage 7.5 useful not only as a working machine, but as a dependable historical snapshot of the project.

---

# Third-party software and publishing

This repository contains or interoperates with historically significant software and third-party-compatible components.

Before redistributing CP/M binaries, historical ROM material, disk images, or other third-party content, review:

**[Publishing and Third-Party Material](docs/PUBLISHING_AND_THIRD_PARTY.md)**  
**[LICENSE-NOTICE.md](LICENSE-NOTICE.md)**

The FPGA project documentation does not claim ownership over third-party material.

---

# Contributing

The Stage 7.5 gold artifacts are intentionally frozen.

Improvements are welcome, but changes should be treated as a **new development release**, not silently substituted for Stage 7.5.

See [CONTRIBUTING.md](CONTRIBUTING.md).

A useful contribution should ideally include:

- source change,
- reason for the change,
- successful Vivado implementation,
- timing/DRC results,
- hardware test results,
- new hashes,
- regression verification against the Stage 7.5 behavior.

---

# Credits

Built as part of the **MILO Project**.

This release combines FPGA implementation work, hardware bring-up, CP/M storage debugging, persistent-write validation, recovery engineering, and release documentation around the Digilent Nexys A7-100T.

Special credit belongs to the original authors and maintainers of the historical Altair/8080/CP/M software and compatible open hardware cores on which this project builds.

---

<div align="center">

## MILO Altair 8800 FPGA

**Nexys A7-100T · Intel 8080 · CP/M 2.2 · MITS-style I/O · Persistent microSD · QSPI autonomous boot**

### Stage 7.5 / WRITE-FIX-3 / GOLD

**Preserve it. Boot it. Use it.**

</div>
