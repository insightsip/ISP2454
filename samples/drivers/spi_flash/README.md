# SPI Flash

Standalone adaptation of Zephyr's `samples/drivers/spi_flash` to bring up and
verify the external SPI-NOR flash of the ISP2454 test boards, without
building/flashing a full Matter application.

## Supported boards

| Board target                       | Module      | External flash |
|------------------------------------|-------------|----------------|
| `isp2454_lx_tb_mx/nrf54l15/cpuapp` | ISP2454-LX  | MX25R3235F     |
| `isp2454_lx_tb_rn/nrf54l15/cpuapp` | ISP2454-LX  | AT25FF321A     |
| `isp2454_lp_tb_mx/nrf54l15/cpuapp` | ISP2454-LP  | MX25R3235F     |
| `isp2454_lp_tb_rn/nrf54l15/cpuapp` | ISP2454-LP  | AT25FF321A     |

Both chips are 32 Mbit (4 MiB) parts.

The flash is wired to `spi00`:

| Signal | Pin                |
|--------|--------------------|
| SCK    | P2.01              |
| MOSI   | P2.02              |
| MISO   | P2.04              |
| CS     | P2.05 (active low) |

## Build and flash

From the repository root:

```shell
west build -b isp2454_lx_tb_rn/nrf54l15/cpuapp samples/drivers/spi_flash -p
west flash
```

This builds a single application image: no MCUboot is added.

With twister (on hardware connected to the host):

```shell
west twister -T samples/drivers/spi_flash -p isp2454_lx_tb_rn/nrf54l15/cpuapp --device-testing --device-serial <COMx>
```

## What it does

> [!WARNING]
> The test erases and rewrites the **last 4 KB sector** of the external flash
> (offset `0x3ff000`). This area is outside the MCUboot secondary slot used by
> the Matter samples, but any other data stored there is lost.

On boot it runs a single-sector test and prints the result on the serial
console (`uart20`, TX on P1.04, 115200 baud):

1. Erases the sector and checks that it reads back as erased (`0xff`).
2. Writes 8 known bytes, reads them back and compares.

A clean run looks like:

```text
at25ff321a@0 SPI flash testing
==========================

Perform test on single sector
Test 1: Flash erase
Flash erase succeeded!

Test 2: Flash write
Attempting to write 8 bytes
Data read matches data written. Good!!
```

(the device name is `mx25r3235f@0` on `*_mx` boards).
