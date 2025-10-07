# LVGL ported to RZ/G3E-EVKIT

## Overview

The RZ/G3E-EVKIT is a development kit for the RZ/G3E MPU. It's a SMARC v2.1 System-on-Module (SOM)
with a carrier board. It is one of the more powerful MPU boards available and has some specialized
applications in mind for use with GStreamer and electrical current measurement.

## Buy

You can purchase RZ/G3E-EVKIT from https://www.renesas.com/en/design-resources/boards-kits/rz-g3e-evkit

## Benchmark

<!-- <a href="https://www.youtube.com/watch?v=XXXXXXXXXXXXXXXXXXXXXXX">
    <img src="https://github.com/user-attachments/assets/87c1f2e5-0260-4772-b711-13fdab467474" width="75%">
</a> -->

Here are the results of the benchmark with single-threaded Wayland.

| Name                      | Avg. CPU | Avg. FPS | Avg. time | render time | flush time |
| :------------------------ | -------: | -------: | --------: | ----------: | ---------: |
| Empty screen              | 74%      | 21       | 36        | 1           | 35         |
| Moving wallpaper          | 99%      | 19       | 48        | 4           | 44         |
| Single rectangle          | 27%      | 29       | 3         | 0           | 3          |
| Multiple rectangles       | 61%      | 29       | 21        | 0           | 21         |
| Multiple RGB images       | 96%      | 21       | 43        | 2           | 41         |
| Multiple ARGB images      | 99%      | 19       | 47        | 6           | 41         |
| Rotated ARGB images       | 99%      | 16       | 57        | 18          | 39         |
| Multiple labels           | 99%      | 20       | 47        | 7           | 40         |
| Screen sized text         | 99%      | 14       | 64        | 24          | 40         |
| Multiple arcs             | 99%      | 19       | 47        | 8           | 39         |
| Containers                | 90%      | 23       | 38        | 10          | 28         |
| Containers with overlay   | 99%      | 13       | 72        | 33          | 39         |
| Containers with opa       | 99%      | 14       | 64        | 24          | 40         |
| Containers with opa_layer | 99%      | 12       | 74        | 34          | 40         |
| Containers with scrolling | 99%      | 18       | 51        | 15          | 36         |
| Widgets demo              | 37%      | 24       | 23        | 8           | 15         |
| All scenes avg.           | 85%      | 19       | 45        | 12          | 33         |

## Specification

### CPU and Memory
- **CPU:** 1.8 GHz quad core ARM Cortex-A55
- **RAM:** 4GB LPDDR4X SDRAM
- **Flash:** Micro SD card, 64 GB eMMC, 16 MB flash
- **GPU:** Mali-G52

### Display and Touch
- **Resolution:** Micro HDMI 1920x1080
- **Interface:** HDMI from MIPI on carrier board. HDMI from RGB also available on SoM
- **Color Depth:** 32-bit

### Connectivity
- Dual ethernet
- 6x USB (2.0, 3.0, type-C, micro 2.0 OTG)
- multi audio jack interface
- SIM card
- PMOD ports
- Camera ribbon connectors

## Getting started

### Hardware setup

You will need to provide your own 65 watt USB C power supply and a Micro SD card.

- Connect SER3_UART to your PC.
- Connect the Micro HDMI port to a monitor.
- Optionally connect ethernet.
- Connect the 65 watt USB power supply to USB-C PWR IN when it's time to power the board.

### Software setup

Follow the instructions in "RZ_G3E Evaluation Board Kit Quick Start Guide.pdf"
in this repo which is from
https://www.renesas.com/en/design-resources/boards-kits/rz-g3e-evkit#documents.
It is a detailed guide for obtaining the OS image, flashing the Micro SD card, and
flashing the board's bootloader into its ROM.
This repo contains a copy of the prebuilt OS images ZIP file.
Flashing the bootloader is a bit complex
so follow the directions carefully. The guide says to use Tera Term to send files over
serial to the board. Tera Term is a Windows application. If you are using Linux, you can send
files over serial by opening the serial port with the correct baud settings at least once
and then using `cat` to send the file to the serial device. E.g. `cat file.txt > /dev/ttyUSB0`.
You may be able to keep the serial terminal open while you run this command in a separate
terminal.

You need an SDK to cross-compile applications to run on the board. When you compile an OS
image locally with Yocto, it also produces an SDK. To save you the time needed to build Yocto,
an SDK is included in this repo. It's for x86-64 architecture systems, so you will need to use an
x86-64 emulator to run it if your development PC is x86-32 or ARM based. Docker has support for emulation.
Unzip sdk.zip and execute the shell script
rz-vlp-glibc-x86_64-core-image-weston-cortexa55-smarc-rzg3e-toolchain-5.0.8.sh.
If you wish to uninstall it, simply delete `/opt/rz-vlp/` or just `/opt/rz-vlp/5.0.8`
if you have multiple SDK versions. To activate the SDK in your current terminal, run
`. /opt/rz-vlp/5.0.8/environment-setup-cortexa55-poky-linux`. It will set environment
variables like `CC`, the compiler. You can run `$CC main.c` to compile a C file that
will be able to run on the board.

### Run the project

- Clone this repository and its submodules:
  `git clone --recursive https://github.com/lvgl/lv_port_renesas_rz-g3e-evkit.git`
- Open the project in an editor of your choice.
- Before doing anything else in the terminal, make sure the SDK environment variables
  have been set as mentioned above. Run
  `. /opt/rz-vlp/5.0.8/environment-setup-cortexa55-poky-linux`.
- Edit lv_conf.h or lv_conf.defaults if you want. If you edit lv_conf.defaults, you need to
  regenerate lv_conf.h by running `python3 lv_port_linux/lvgl/scripts/generate_lv_conf.py`.
- Edit lv_port_linux/src/main.c if you want to change the demo.
- Build the project. The SDK includes `cmake` and `ninja` so you don't need to install them.
  ```shell
  cmake -GNinja -B build -S .
  ninja -C build -j4 lvglsim
  ```
- The executable is at `build/bin/lvglsim`. Copy it to the board. If ethernet is connected,
  you can do it with `scp build/bin/lvglsim root@<board ip>:/root/` where `<board ip>` is
  the IP address of the board which you find by running `ip a` on the board serial terminal.
- Run `./lvglsim` on the serial terminal of the board. You can set the size of the LVGL
  Wayland window by giving `-W` and `-H` arguments. E.g. `./lvglsim -W 1920 -H 1080`.

### Debugging

The OS image does not have `gdbserver` which is typically what you'd need to do remote
interactive debugging. You will need to build the image with Yocto and include gdbserver
in the configuration.

## Notes

More information and resources are available at:
- https://www.renesas.com/en/design-resources/boards-kits/rz-g3e-evkit#documents
- https://www.renesas.com/en/software-tool/rzg3e-board-support-package#download

## Contribution and Support

If you find any issues with the development board feel free to open an Issue in this repository. For LVGL related issues (features, bugs, etc) please use the main [lvgl repository](https://github.com/lvgl/lvgl).

If you found a bug and found a solution too please send a Pull request. If you are new to Pull requests refer to [Our Guide](https://docs.lvgl.io/master/CONTRIBUTING.html#pull-request) to learn the basics.

