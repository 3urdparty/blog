---
title: Setting up Klipper on Ender 3 v3 SE with RPi Zero 2W via UART
description: This is a short guide on setting up Klipper and Mainsail on a Raspberry Pi Zero 2W and connecting it to an Ender 3v3SE via UART
publishDate: 29 October 2024
tags:
  - 3D-printing
  - Ender3v3
  - Ender3v3SE
  - Creality
  - Ender
  - Klipper
  - Mainsail
  - Raspberry-Pi
draft: false
subtitle:
coverImage:
  src: './cover.png'
  alt: './Klipper on Ender 3v3SE'
---

# Overview

This is a guide for upgrading an [Ender 3 V3 SE]() from its factory [Marlin]() firmware to [Klipper](). We will be using [Mainsail](), but this could may well be [Fluidd]().

## Why Klipper

Klipper is a firmware that gives you more control over your printer, making it easier to configure to print exactly the way you want to. It's extremely extensible and is entirely driven by the community. Klipper offers significant advantages including wireless printing, remote monitoring, a web interface, enhanced bed leveling, and advanced features like pressure advance and input shaping that can improve print quality, and the most important one to me: Webcams! (Though I will discuss how you can connect it at a later [[Connecting a Webcam to Mainsail Hosted on a Pi Zero 2W|blog|]]). Overview of the features:

- Send gcode directly from your PC instead of manual SD card transfer
- Remote monitoring and command sending
- Web interface you can access from any device in your home (and even outside!)
- Improved bed leveling with [KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging)
- Pressure advance and input shaping

## Why Pi Zero 2W

Of all the SBCs available in the pi range of SBCs, the Pi Zero 2W is the perfect form factor to fit inside the 3D printer, though we won't present that here. It provides just enough resources such that it can power klipper+Mainsail as well as a webcam running on low frame rate and resolution, whilst also maintaining low temperatures (so you don't have to worry about cooling)

> [!WARNING]
> The Pi Zero 2W is different from the the Pi Zero, and according to the docs its not recommended to use the Pi Zero 2W since it might barely have enough compute to run klipper and mainsail.

# Checklist

You will need

- [ ] An Ender 3 V3 SE (Any other Ender 3 printer should also work, but it may vary)
- [ ] A Raspberry Pi, in our case we will be using Pi Zero 2W
- [ ] microSD card for the Pi
- [ ] Power Supply for the Pi (5V at 2-2.4A)
- [ ] SD card
- [ ] USB C to USB A cable to connect the Ender and the Pi

> [!INFO] Note:
> The printer's built-in display becomes non-functional after installing Klipper. However, there's a [repo](https://github.com/jpcurti/E3V3SE_display_klipper) on GitHub that can help restore some of the display's functionality. We won't be using it for our installation, as the LCD connectors used to power the LCD Display will be used to connect to the Pi Zero 2W over UART

# Installation

## Raspberry Pi Setup

We will have to flash MainsailOS to the RPI Zero 2W.

1. Download and Install the Raspberry Pi Imager at this [link](https://www.raspberrypi.com/software/)
2. Choose your device (In this case, it will be the RPI Zero 2W)

![](rpi_imager_step_1.png)

![](rpi_imager_step_2.png)
![](rpi_imager_step_3.png)
![](rpi_imager_step_4.png)
![](rpi_imager_step_5.png)
![](rpi_imager_step_6.png)

1. Find and select `Mainsail OS`, the 64-bit version if available
2. Choose your storage device (the microSD which you will install in your Raspberry Pi)
   ![](rpi_imager_step_7.png)
3. Click on next, and select "Edit Settings" to enable SSH and connect to your local Wifi Network at startup
   ![](rpi_imager_step_8.png)
   You will see this window. Here's an explanation of the following settings:

- `hostname`: the host name at which the Raspberry Pi will be reachable over HTTP.
- `username`: the username of the main user of the pi
- `password`: the password of the main user of the pi
- `SSID`: Name of your Wifi network
- `SSID`: Password for your Wifi network
  Make sure toggle "Enable SSH" in the "Services" tab and select "Use Password Authentication"
  You can then press save and continue to flash by selecting Yes
  ![](rpi_imager_step_9.png)
  ![](rpi_imager_step_10.png)

1. After the flashing is complete, eject the SD card and plug into Rpi
2. Connect the power supply
3. SSH into your Raspberry Pi using your previously set username and password:

```bash
ssh ender3v3se.local
```

You will be prompted for your password. After successfully typing it in your should see this prompt:

![](rpi_imager_step_13.png)

# Building Klipper

Now, we will have to build the klipper firmware for out printer. Begin by navigating to the klipper directory from the home directory and enter `make menuconfig`:

```bash
cd ~
cd klipper/
make menuconfig
```

You will get a menu screen, only you will have different settings to mine. Change those settings so that they are like this:
![](rpi_imager_step_14.png)
Then press `Q` and run `make`. It should start building the project:
![](rpi_imager_step_15.png)

> [!Info] Info:
> `make menuconfig` is a command that belongs to `CMAKE`, which is responsible for building `C/C++` projects

Once that is done, you will have a file in the `~/klipper/out` directory called `klipper.bin`. We will have to move this to another SD card to flash this into the printer. To do that, begin by moving it to the directory `~/printer_data/config/` like so:

```bash
mv ~/klipper/out/klipper.bin ~/printer_data/config/
```

Now if you navigate to the hostname (followed by `.local`) you set in the Rpi flashing step in your internet browser (in my case it's `ender3v3se.local`), you should see the Mainsail dashboard. You will have errors there, which is to be expected. Navigate to the machine page via the sidebar:
![](rpi_imager_step_16.png)

Once that is done, you will see the same `klipper.bin` file we just moved over here:
![](rpi_imager_step_17.png)
Download the file into your local computer.

## Preparing the SD card for flashing

Now there is a lot of weird steps that you have to take to ensure that the printer has flashed Klipper successfully. Some of them might not be ruining your flashing step and hence not that important, but it is best to follow all the precautions to flash the firmware as there is no way of finding out if the firmware has flashed or not besides the setup working. Also it is recommended that the SD card you are using is between 4-8GB. It was reported that bigger SD card sizes have worked, but this range has the highest success rate. So to begin, we have to format the SD Card.

### Formatting the SD Card

We have to format the SD using FAT32. Follow the appropriate guides online if you are a windows user, but I will be using Mac and explaining some intricacies that you windows users might not have.

Navigate to the `Disk Utility` app and open your chosen SD card, and then press the "Erase" button:
![](rpi_imager_step_18.png)
You will see this menu. Name it whatever you would like, but make sure that you select `MS-DOS (FAT32)` or `MS-DOS` if that's the only option available to you
![](rpi_imager_step_.png)
After that is done:
![](rpi_imager_step_19.png)
We will have to do some further cleaning if you are using a MacOS:

1. Open a terminal and navigate to `/Volumes/YOUR_SD_CARD_NAME` in my case i will run:

```bash
cd /Volumes/SD
ls -la
```

Once you run `ls -la`, you will see a couple of hidden files. Just using `rm -rf` wouldn't be enough to remove some of them, so you have to run:

```bash
sudo mdutil -dX "/Volumes/ENDER3V3SD"
rm -rf .*
```

> [!WARNING] Warning!
> The `rm -rf` command is capable of deleting your entire filesystem if given `sudo` rights, so tread with caution!

Finally you can move the the `klipper.bin` file into the SD card.

> [!NOTE] Note
> If you tried flashing the klipper firmware before, ensure to rename the `klipper.bin` file to something that you haven't used before, like `firmware.bin`.

Now you can eject the SD card and insert it into your printer.

# Further Steps

I will post blog posts in the future as to further configurations to get the most out of your 3D printer:

[Klipper Configuration](Klipper Configuration)
[Advanced Klipper Configuration](Advanced Klipper Configuration)
[Setting up Webcam on Pi Zero 2W](Setting up Webcam on Pi Zero 2W)
[Setting up Obico](Setting up Obico)
