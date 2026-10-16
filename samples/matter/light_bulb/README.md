# Matter: Light Bulb

Adaptation of the nRF Connect SDK
[Matter Light Bulb sample](https://docs.nordicsemi.com/bundle/ncs-latest/page/nrf/samples/matter/light_bulb/README.html)
(`nrf/samples/matter/light_bulb`) for the ISP2454 test boards.

The application code (`src/`), the Matter data model (`src/default_zap/`) and
the `prj*.conf` files are unchanged from upstream. The device is a dimmable
light that works over Thread. A PWM-driven LED shows the light state and
brightness, and the light can be controlled locally with a button or remotely
from a Matter controller or a light switch. For a full description of the
features, the configuration options and the CHIP Tool test procedure, see the
upstream documentation.

## Supported boards

| Board target                       | Module      | External flash |
|------------------------------------|-------------|----------------|
| `isp2454_lx_tb_mx/nrf54l15/cpuapp` | ISP2454-LX  | MX25R3235F     |
| `isp2454_lx_tb_rn/nrf54l15/cpuapp` | ISP2454-LX  | AT25FF321A     |
| `isp2454_lp_tb_mx/nrf54l15/cpuapp` | ISP2454-LP  | MX25R3235F     |
| `isp2454_lp_tb_rn/nrf54l15/cpuapp` | ISP2454-LP  | AT25FF321A     |

Each board also has a `/ns` variant (for example
`isp2454_lx_tb_mx/nrf54l15/cpuapp/ns`) for the TF-M build.

## Changes from the upstream sample

- **Board files**: the nRF DK overlays in `boards/` and
  `sysbuild/mcuboot/boards/` are replaced by ISP2454 overlays. They are based
  on the `nrf54l15dk_nrf54l15_cpuapp` files.
- **External flash**: the DK uses the `mx25r64` node. Here the overlays use
  `ext_flash`, a label that the board definition gives to both the Macronix
  and the Renesas flash. The same overlay content therefore works for `*_mx`
  and `*_rn` boards.
- **PWM LED**: the application drives the light through the `pwm-led1` alias.
  On the DK, the overlay creates a `pwm_led1` node on `pwm20` channel 1 with
  its own pinctrl. On the ISP2454 test boards, the board definition
  (`isp2454_tb_common.dtsi`) already provides `pwm_led1` on `pwm20`
  channel 0 (P1.10, LED 1), so the overlays only add the alias.
- **Partitions**: the DK partition layout from
  `samples/matter/nrf54l15_cpuapp_partitions.dtsi` is replaced by local
  copies that point to `ext_flash`:
  - `isp2454_matter_partitions.dtsi`: standard (MCUboot) build.
  - `isp2454_matter_tfm_base_partitions.dtsi` and
    `isp2454_matter_tfm_application_partitions.dtsi`: TF-M build.

  The same files are in `sysbuild/mcuboot/boards/` so that MCUboot and the
  application use the same layout. They are also identical to the Thermostat
  sample's files. If you change one copy, change the others too.
- **`sample.yaml`**: build configurations are limited to the ISP2454 board
  targets.

### Flash layout (standard build)

| Partition        | Memory    | Offset     | Size    |
|------------------|-----------|------------|---------|
| `mcuboot`        | RRAM      | `0x000000` | 52 KB   |
| `image-0`        | RRAM      | `0x00d000` | 1428 KB |
| `factory-data`   | RRAM      | `0x172000` | 4 KB    |
| `storage`        | RRAM      | `0x173000` | 40 KB   |
| `image-1` (DFU)  | ext_flash | `0x000000` | 1428 KB |

## User interface

| Element  | Pin   | Function                                                       |
|----------|-------|----------------------------------------------------------------|
| LED 0    | P2.09 | Device state (advertising, commissioning, connected)           |
| LED 1    | P1.10 | Light bulb (PWM): on/off and brightness, blinks on Identify    |
| Button 0 | P1.13 | Short press: start BLE advertising. Long press: factory reset  |
| Button 1 | P1.09 | Toggle the light on/off                                        |

## Build configurations

| Configuration | `sample.yaml` test                   | Extra arguments         | Board target             |
|---------------|--------------------------------------|-------------------------|--------------------------|
| Debug         | `sample.matter.light_bulb.debug`     | (none)                  | `.../nrf54l15/cpuapp`    |
| Release       | `sample.matter.light_bulb.release`   | `-DFILE_SUFFIX=release` | `.../nrf54l15/cpuapp`    |
| TF-M          | `sample.matter.light_bulb.tfm`       | `-DFILE_SUFFIX=tfm`     | `.../nrf54l15/cpuapp/ns` |

NFC onboarding is disabled in the TF-M build
(`boards/*_ns_tfm.conf`).

## Build and flash

From the repository root:

```shell
# Debug
west build -b isp2454_lx_tb_mx/nrf54l15/cpuapp samples/matter/light_bulb -p

# Release
west build -b isp2454_lx_tb_mx/nrf54l15/cpuapp samples/matter/light_bulb -p -- -DFILE_SUFFIX=release

# TF-M
west build -b isp2454_lx_tb_mx/nrf54l15/cpuapp/ns samples/matter/light_bulb -p -- -DFILE_SUFFIX=tfm

west flash --erase
```

The sample uses sysbuild, so MCUboot is built and flashed with the
application.

With twister (build only):

```shell
west twister -T samples/matter/light_bulb -p isp2454_lx_tb_mx/nrf54l15/cpuapp -O C:/tw
```

> [!NOTE]
> On Windows, use a short output directory (`-O C:/tw`) or
> `--short-build-path`. The default `twister-out` path under the repository
> makes some object paths longer than 260 characters, and the build fails at
> the archive step with `No such file or directory`.

## Testing

Commissioning and testing are the same as for the upstream sample. Follow the
[upstream testing procedure](https://docs.nordicsemi.com/bundle/ncs-latest/page/nrf/samples/matter/light_bulb/README.html#testing)
with CHIP Tool and the default onboarding codes:

- QR code payload: `MT:6FCJ142C00KA0648G00`
- Manual pairing code: `34970112332`

For example, once the device is commissioned with node ID `1`:

```shell
chip-tool onoff toggle 1 1
chip-tool levelcontrol move-to-level 128 0 0 0 1 1
```
