# Quick Start

## Use the packaged release immediately

1. Put the Stage 7.4 microSD in the Nexys A7.
2. Connect J6 PROG/UART to the Linux host.
3. Run:

```bash
./tools/flash-qspi.sh
```

If `out/` does not contain a freshly built MCS, the flash script automatically uses the packaged release MCS.

Cold power-cycle the Nexys after a successful QSPI verify.

Open the UART at **115200 8N1**. Normal boot requires no input. Press **DEL** during the five-second setup window if desired.

## Rebuild from source

Known build environment: Ubuntu 24.04 + AMD Vivado 2026.1.

```bash
./tools/build.sh
```

The build is fail-closed: a negative WNS prevents release BIT/MCS generation. Outputs appear in `out/`.

Then flash the freshly built image:

```bash
./tools/flash-qspi.sh
```

## CP/M

The current system boots 56K CP/M 2.2b for the Altair 8 MB Virtual Drive.

```text
A0>STAT A:
A0>B:
B0>STAT B:
```

Known observed free space:

- A: 7940k
- B: 8168k

Receive a file into B: using PCGET:

```text
B0>A:PCGET PRIME.HEX
```

Then start XMODEM Send on the PC.
