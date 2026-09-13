# MILO Altair 8800 FPGA
## Authentic 8080/CP/M Computer on the Digilent Nexys A7-100T

**Short description:** A self-booting MITS Altair 8800-compatible FPGA appliance for the Digilent Nexys A7-100T: vm80a Intel 8080 core, authentic MITS DBL bootstrap, 56K CP/M 2.2 with 8 MB virtual storage, persistent microSD, autonomous QSPI boot, UART console, selectable 2–25 MHz runtime speeds, and live A0–A15 address LEDs.

This repository is the clean, current recovery/build package for the MILO Altair 8800 FPGA project. It intentionally contains the files required to understand, build, flash, recover, and operate the current machine — not superseded experiments or old Stage builds.

## What it is

The goal is an appliance-like Altair 8800 implementation: power it on, watch the authentic MITS disk bootstrap run at 2 MHz, and arrive at CP/M on a serial terminal. No VGA console, no USB keyboard, no normal-operation switches, and no 7-segment display are required.

### Current hardware behavior

- Digilent Nexys A7-100T (`XC7A100T-1CSG324C`)
- vm80a Intel 8080-compatible CPU core
- 64 KB FPGA RAM
- Authentic MITS DBL bootstrap
- MITS 88-2SIO-compatible serial console
- MITS 88-DCDD-style disk interface
- UART: **115200 8N1**
- QSPI x4 autonomous FPGA boot
- microSD persistent disk/settings storage
- CPU choices: **2 / 4 / 8 / 16 / 25 MHz**
- Bootstrap stays at authentic **2 MHz**
- Selected runtime speed applies after CP/M starts serial activity
- 16 red LEDs show **inverted live A0–A15 address state**
- Blue RGB LED = disk/storage activity
- 7-segment display intentionally disabled

## Current storage

The current Stage 7.4 MAX STORAGE system boots:

```text
56K CP/M 2.2b v1.0
For Altair 8Mb Virtual Drive
```

Observed on the current machine:

```text
A: 7940k free
B: 8168k free
```

A: and B: are the large 8 MB-class virtual disks. C: and D: are retained legacy disks in the current build.

## Setup menu

During the five-second startup window, press **DEL**:

```text
1  2 MHz AUTHENTIC
2  4 MHz
3  8 MHz
4  16 MHz
5  25 MHz TURBO
W  Toggle SD safe/write mode
S  Show system status
X  Save & Exit
Q  Discard & Exit
```

Settings are stored persistently on the microSD configuration block. QSPI stores the FPGA configuration itself.

## Repository layout

```text
rtl/          Current HDL only
constraints/  Nexys A7 XDC
memory/       Current FPGA memory-init files
build/        Vivado build + QSPI flash Tcl
release/      Current BIT/MCS and compact implementation reports
recovery/     Exact first-80-MiB recovery image when available
tools/        Small build/flash/recovery helpers
docs/         Build, operation, recovery and architecture notes
```

## Quick build

From the repository root on the Linux build host:

```bash
./tools/build.sh
```

The repository is portable; it no longer depends on Adam's original `/home/milo/fpga/mits-altair8800` source-tree path. Generated Vivado files and new BIT/MCS outputs go into `out/`.

A valid current build must have non-negative timing slack. The known write-fix build reached:

```text
STAGE7_WNS_NS=0.313
STAGE 7 FINAL BUILD: PASS
```

## Flash QSPI

Connect Nexys **J6 PROG/UART**. The onboard Digilent FT2232 should appear as USB `0403:6010`.

```bash
./tools/flash-qspi.sh
```

The flash helper prefers a freshly rebuilt `out/` MCS when present; otherwise it programs the packaged timing-clean release MCS.

Expected finish:

```text
Program/Verify Operation successful.
STAGE7_QSPI_FLASH_VERIFY=PASS
```

Cold power-cycle after programming.

## Using CP/M

Terminal: **115200 8N1**.

Typical storage check:

```text
A>STAT A:
A>B:
B>STAT B:
```

To receive a file into B: with the supplied CP/M PCGET utility:

```text
B>A:PCGET PRIME.HEX
```

Then start XMODEM Send on the PC.

## Current milestone

The Stage 7.4 MAX STORAGE system is the **95% perfect known-good milestone**:

- autonomous boot works
- authentic DBL starts at 2 MHz
- CP/M 2.2 56K boots
- 8 MB A: works
- 8 MB B: is visible
- setup persistence works
- 16 MHz runtime is visibly faster and works
- current HDL includes the SD CMD24 write-response polling fix
- current HDL uses inverted A0–A15 LEDs

Remaining validation: occasional cold-boot lockup and extended hardware write-path stress testing.

## Redistribution note

Some historical Altair/CP/M software or generated recovery images may contain third-party copyrighted material. Before publishing binary disk/recovery images in a public repository, verify that you have the right to redistribute them. The HDL/build portions can be kept separate from private recovery media if necessary.
