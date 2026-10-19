# ISP2454

Board definitions and samples for the Insight SiP ISP2454 module (Nordic
nRF54L15), for the nRF Connect SDK.

This repository is a west manifest repository and a Zephyr module. It brings in
nRF Connect SDK v3.4.1 and adds the ISP2454 test boards to it, so you can build
for these boards like any other board in the SDK.

## Compatibility

| ISP2454 | nRF Connect SDK |
|---------|-----------------|
| v1.0.0  | v3.4.1          |

See [CHANGELOG.md](CHANGELOG.md) for the changes in each release.

## Repository layout

| Path                         | Content                                              |
|------------------------------|------------------------------------------------------|
| `west.yml`                   | West manifest (nRF Connect SDK v3.4.1)               |
| `zephyr/module.yml`          | Zephyr module definition (declares the board root)   |
| `boards/insightsip/isp2454/` | Board definitions for the ISP2454 test boards        |
| `samples/drivers/spi_flash/` | External SPI-NOR flash test                          |
| `samples/matter/light_bulb/` | Matter Light Bulb, adapted to the ISP2454 boards     |
| `samples/matter/thermostat/` | Matter Thermostat, adapted to the ISP2454 boards     |

## Supported boards

All boards use the nRF54L15 application core. Each one has a secure target
(`.../nrf54l15/cpuapp`) and a non-secure target for TF-M builds
(`.../nrf54l15/cpuapp/ns`).

| Board target                       | Module      | External flash          |
|------------------------------------|-------------|-------------------------|
| `isp2454_lx_tb/nrf54l15/cpuapp`    | ISP2454-LX  | None                    |
| `isp2454_lx_tb_mx/nrf54l15/cpuapp` | ISP2454-LX  | Macronix MX25R3235F     |
| `isp2454_lx_tb_rn/nrf54l15/cpuapp` | ISP2454-LX  | Renesas AT25FF321A      |
| `isp2454_lp_tb_mx/nrf54l15/cpuapp` | ISP2454-LP  | Macronix MX25R3235F     |
| `isp2454_lp_tb_rn/nrf54l15/cpuapp` | ISP2454-LP  | Renesas AT25FF321A      |

Both flash chips are 32 Mbit (4 MiB) parts on `spi00`.

### Test board peripherals

| Element       | Pin                                         | Alias / node                                |
|---------------|---------------------------------------------|---------------------------------------------|
| LED 0         | P2.09                                       | `led0`                                      |
| LED 1         | P1.10                                       | `led1`, PWM: `pwm_led1` (`pwm20` channel 0) |
| Button 0      | P1.13                                       | `sw0`                                       |
| Button 1      | P1.09                                       | `sw1`                                       |
| Console/shell | P1.04 TX, P1.05 RX                          | `uart20`, 115200 baud                       |
| SPI-NOR flash | P2.01 SCK, P2.02 MOSI, P2.04 MISO, P2.05 CS | `spi00`, `ext_flash`                        |

## Samples

| Sample                                                   | Description                                                    |
|----------------------------------------------------------|----------------------------------------------------------------|
| [SPI Flash](samples/drivers/spi_flash/README.md)         | Erases, writes and reads back one sector of the external flash |
| [Matter Light Bulb](samples/matter/light_bulb/README.md) | Dimmable Matter light over Thread                              |
| [Matter Thermostat](samples/matter/thermostat/README.md) | Matter thermostat over Thread with simulated temperature       |

Each sample README lists the supported boards, the build configurations and
the changes from the upstream nRF Connect SDK sample.

## Getting started

### Prerequisites

- The nRF Connect SDK toolchain for v3.4.1, installed with
  [nRF Util](https://docs.nordicsemi.com/bundle/nrfutil/page/README.html) or
  the nRF Connect for VS Code extension.

### Set up the workspace

```shell
west init -m https://github.com/insightsip/ISP2454 isp2454_rep
cd isp2454_rep
west update
```

This creates the workspace with this repository in `isp2454/` and the
nRF Connect SDK (`nrf/`, `zephyr/`, `modules/`, ...) next to it.

### Build and flash

From the `isp2454/` directory:

```shell
west build -b isp2454_lx_tb_mx/nrf54l15/cpuapp samples/matter/light_bulb -p
west flash --erase
```

Replace the board target and the sample path as needed.

### VS Code

`.vscode/settings.json` registers the repository as a board root and the
three samples as applications for the nRF Connect for VS Code extension. Open
the `isp2454/` folder to use them.

## License

[Apache License 2.0](LICENSE)
