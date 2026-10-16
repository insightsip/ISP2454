# SPDX-License-Identifier: Apache-2.0
#
# Standalone SPI-NOR flash test app - adapted from
# zephyr/samples/drivers/spi_flash to bring up/verify the external flash of
# the ISP2454 test boards in isolation from the Matter applications.
#
# The ISP2454 boards are provided by this repository's Zephyr module
# (zephyr/module.yml, board_root: .), so no BOARD_ROOT is needed here.

cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(spi_flash)

FILE(GLOB app_sources src/*.c)
target_sources(app PRIVATE ${app_sources})
