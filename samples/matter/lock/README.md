# Matter: Door Lock

Adaptation of the Nordic Matter Door Lock sample for the ISP2454 test boards.

In nRF Connect SDK v3.4.1, `nrf/samples/matter/lock` only contains a
`README.rst` that points to the **nRF Door Lock and Access Control Add-on**,
where the sample now lives. This copy is based on that sample.

The application code (`src/`) and the Matter data model (`src/default_zap/`)
are unchanged. The device is a door lock that works over Thread. An LED shows
the bolt state, and the lock can be locked or unlocked with a button or from
a Matter controller. Users and credentials (PIN codes) are stored in PSA
Protected Storage. For a full description of the features, the configuration
options and the CHIP Tool test procedure, see the add-on documentation.

## Supported boards

| Board target                       | Module      | External flash |
|------------------------------------|-------------|----------------|
| `isp2454_lx_tb_mx/nrf54l15/cpuapp` | ISP2454-LX  | MX25R3235F     |
| `isp2454_lx_tb_rn/nrf54l15/cpuapp` | ISP2454-LX  | AT25FF321A     |
| `isp2454_lp_tb_mx/nrf54l15/cpuapp` | ISP2454-LP  | MX25R3235F     |
| `isp2454_lp_tb_rn/nrf54l15/cpuapp` | ISP2454-LP  | AT25FF321A     |

Each board also has a `/ns` variant (for example
`isp2454_lx_tb_mx/nrf54l15/cpuapp/ns`) for the TF-M build.

## Changes from the original sample

- **Board files**: the nRF DK overlays in `boards/` and
  `sysbuild/mcuboot/boards/` are replaced by ISP2454 overlays. They are based
  on the `nrf54l15dk_nrf54l15_cpuapp` files.
- **External flash**: the DK uses the `mx25r64` node. Here the overlays use
  `ext_flash`, a label that the board definition gives to both the Macronix
  and the Renesas flash. The same overlay content therefore works for `*_mx`
  and `*_rn` boards.
- **Partitions**: the DK partition layout from
  `samples/matter/nrf54l15_cpuapp_partitions.dtsi` is replaced by local
  copies that point to `ext_flash`:
  - `isp2454_matter_partitions.dtsi`: standard (MCUboot) build.
  - `isp2454_matter_tfm_base_partitions.dtsi` and
    `isp2454_matter_tfm_application_partitions.dtsi`: TF-M build.

  The same files are in `sysbuild/mcuboot/boards/` so that MCUboot and the
  application use the same layout. They are also identical to the Light Bulb
  and Thermostat samples' files. If you change one copy, change the others
  too.
- **`prj.conf`** (debug build only, `prj_release.conf` is unchanged):
  - Product name `ISP Matter Door Lock` and Bluetooth name `ISP_MatterLock`.
    `CONFIG_BT_DEVICE_NAME_MAX` is set to 32, because DFU over SMP makes the
    name dynamic and the default buffer is too small.
  - `CONFIG_CHIP_ENABLE_PAIRING_AUTOSTART=y`: BLE advertising for
    commissioning starts at boot, as in the Light Bulb sample. Without it,
    the device waits for a press on Button 0 and seems to do nothing.
  - `CONFIG_CHIP_DFU_OVER_BT_SMP=y`: DFU over Bluetooth LE (SMP), for
    example with nRF Connect for Mobile.
  - `CONFIG_PM_DEVICE=y`: devices are suspended when the CPU sleeps.
  - Logs on the UART console (J-Link virtual COM port), info level.
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
| LED 1    | P1.10 | Bolt state: on = locked, off = unlocked                        |
| Button 0 | P1.13 | Short press: start BLE advertising. Long press: factory reset  |
| Button 1 | P1.09 | Lock or unlock the door                                        |

## Build configurations

| Configuration | `sample.yaml` test             | Extra arguments                  | Board target             |
|---------------|--------------------------------|----------------------------------|--------------------------|
| Debug         | `sample.matter.lock.debug`     | (none)                           | `.../nrf54l15/cpuapp`    |
| Release       | `sample.matter.lock.release`   | `-DFILE_SUFFIX=release`          | `.../nrf54l15/cpuapp`    |
| TF-M          | `sample.matter.lock.tfm`       | `-DFILE_SUFFIX=tfm`              | `.../nrf54l15/cpuapp/ns` |
| Schedules     | `sample.matter.lock.schedules` | `-DEXTRA_CONF_FILE=schedules.conf` | `.../nrf54l15/cpuapp`  |

The Schedules configuration adds the WeekDay, YearDay and Holiday schedules
of the Door Lock cluster. NFC onboarding is disabled in the TF-M build
(`boards/*_ns_tfm.conf`).

## Build and flash

From the repository root:

```shell
# Debug
west build -b isp2454_lx_tb_mx/nrf54l15/cpuapp samples/matter/lock -p

# Release
west build -b isp2454_lx_tb_mx/nrf54l15/cpuapp samples/matter/lock -p -- -DFILE_SUFFIX=release

# TF-M
west build -b isp2454_lx_tb_mx/nrf54l15/cpuapp/ns samples/matter/lock -p -- -DFILE_SUFFIX=tfm

west flash --erase
```

The sample uses sysbuild, so MCUboot is built and flashed with the
application.

With twister (build only):

```shell
west twister -T samples/matter/lock -p isp2454_lx_tb_mx/nrf54l15/cpuapp -O C:/tw
```

> [!NOTE]
> On Windows, use a short output directory (`-O C:/tw`) or
> `--short-build-path`. The default `twister-out` path under the repository
> makes some object paths longer than 260 characters, and the build fails at
> the archive step with `No such file or directory`.

## Testing

Commissioning and testing are the same as for the original sample. Use
CHIP Tool with the default onboarding codes:

- QR code payload: `MT:8IXS142C00KA0648G00`
- Manual pairing code: `34970112332`

For example, once the device is commissioned with node ID `1`:

```shell
chip-tool doorlock lock-door 1 1 --timedInteractionTimeoutMs 1000
chip-tool doorlock unlock-door 1 1 --timedInteractionTimeoutMs 1000
chip-tool doorlock read lock-state 1 1
```
