---
title: A working draft title
description: This post is for testing the draft post functionality
publishDate: 10 Sept 2023
tags:
  - test
draft: true
---

# Configuration[Permalink](https://artamis.me/projects/klipper_guide/#configuration 'Permalink')

Once the steps above are complete, you should be able to access your web interface by going to `http://[hostname-of-your-pi].local` and see something similar to this:

![screenshot of Fluidd home menu](https://artamis.me/assets/images/klipper_guide/KG_03_Fluidd_home_menu.png)

Fluidd home menu

Next you need to go to the Configuration tab (below image) and you should see a `printer.cfg` file and a `macro.cfg` file. If those files are nowhere to be seen, you can create them by clicking on the ‘+’ button on top of the file list.

![screenshot of Fluidd config menu](https://artamis.me/assets/images/klipper_guide/KG_04_config_menu.png)

Fluidd config menu

Then go to one of the many Github Repos to get a configuration file that is pre-set. I recommend [bootuz-dinamon](https://github.com/bootuz-dinamon/ender3-v3-se-full-klipper). Copy and paste the contents of `printer.cfg` and `macro.cfg` into yours.

## Create Printer Firmware (.bin file)[Permalink](https://artamis.me/projects/klipper_guide/#create-printer-firmware-bin-file 'Permalink')

Make sure your SD card is formatted as FAT32 with an allocation size of 4096, otherwise you might run into issues and be unable to flash

To create the actual firmware that will be running in your 3D printer, you have to compile the firmware on your Rasberry Pi. For this, you need to follow the following steps:

1. Go to the `/klipper` directory and run the menuconfig command by running the following code:

   Copy code`cd ~/klipper/ make menuconfig`

2. Select the following settings:

   Copy code`- Micro-controller architecture: STMicroelectronics STM32 - Processor model: STM32F103 - Bootloader offset: 28KiB - Communication interface: Serial (on USART1 PA10/PA09)`

3. After everything is selected, press q and save your changes, then run:

   Copy code`make`

## Flash 3D Printer[Permalink](https://artamis.me/projects/klipper_guide/#flash-3d-printer 'Permalink')

Once completed, there will be a `klipper.bin` file in the `klipper/out/` folder. To get the file you will have to copy it from your Raspberry Pi to your computer, but first we have to move it from where it is generated to a folder Fluidd/Mainsail can access.

To move the file, SSH into your Raspberry Pi and navigate to the `klipper/out/` folder by running the following command:

Copy code`cd ~/klipper/out`

Then run the `ls` command to confirm the file is there and if it is, you can send it to your `config` folder so it is visible from Fluidd/Mainsail by running the following command

Copy code`cp klipper.bin ~/printer_data/config/`

> Note: If you get an error that says “No such file or directory” you need to verify that the previous steps (compiling the firmware) were done correctly and look for the location of `klipper/out/` or `printer_data/config/` in your Raspberry Pi, then modify the previous commands accordingly.

Now when you go to the configuration tab of your Fluid/Mainsail UI, the `klipper.bin` file should be there so you can just right click and download to your computer. Once you download the file, the next steps are:

1. Transfer the klipper.bin file to your printer’s SD card
2. Turn off the printer
3. Insert SD card
4. Turn on the printer and wait 15 seconds
5. Turn off the printer
6. Take the SD card out and remove the .bin file
7. Insert the empty SD card into the printer
8. Turn on the printer (the screen should be a blue screensaver image)
9. Connect your Raspberry Pi to the printer via USB cable
10. Navigate to your Fluidd or Mainsail web interface

The printer should be there and you should be looking at something like this:

![screenshot of Fluidd home menu](https://artamis.me/assets/images/klipper_guide/KG_05_Fluidd_home_menu.png)

Fluidd home menu

In case the printer is not recognized, SSH into your Raspberry Pi and run the following command:

Copy code`dmesg | grep usb`

which after running should show something like this:

![screenshot of a terminal](https://artamis.me/assets/images/klipper_guide/KG_06_terminal_usb.png)

The line you should look for

Then navigate to your printer’s `printer.cfg` file and look for the `[mcu]` code block. There should be 2 lines under it that start with “serial:”. Comment the long one by adding “#” to the start (without quotation marks) and undocument the short one, changing the value to whatever you got when running the previous command.

Now you should have a working Ender 3 V3 SE with Klipper installed, congratulations! Now you can continue to the calibration section to get your printer calibrated and start printing.

# Calibration[Permalink](https://artamis.me/projects/klipper_guide/#calibration 'Permalink')

## Calibrate PID Coefficients[Permalink](https://artamis.me/projects/klipper_guide/#calibrate-pid-coefficients 'Permalink')

Go to your main Fluidd or Mainsail web interface and scroll down to “Macros”. Then run the `PID_BED` and `PID_EXTRUDER`. Run one first and after it finishes, run the other.

![screenshot of the macros box](https://artamis.me/assets/images/klipper_guide/KG_07_macros.png)

Macros box

> Note: If macros don’t appear in the home screen or if they don’t run, go to `printer.cfg` and add `[include macro.cfg]` somewhere in the file

## Calibrate Z-Offset[Permalink](https://artamis.me/projects/klipper_guide/#calibrate-z-offset 'Permalink')

To calibrate your Z-Offset go to the Tune tab on your left and click on “Calibrate” to generate a mesh matrix of your bed.

![screenshot of bed mesh menu](https://artamis.me/assets/images/klipper_guide/KG_08_bed_mesh.png)

Bed mesh menu

After it is generated, go to the Home tab and go to the Tool box and select `PROBE_CALIBRATE` under the “Tools” drop-down menu.

![screenshot of tool box](https://artamis.me/assets/images/klipper_guide/KG_09_tool.png)

Tool box

Get a piece of paper and run the [paper test](https://www.klipper3d.org/Bed_Level.html#the-paper-test) to calibrate the correct Z-offset for your printer, adjust the values by clicking on the ‘+’ or ‘-‘ buttons that appear. You should be able to move the paper but there should be some drag present. I have had better results doing this when the printer is cold but some recommend doing it when the nozzle and the bed are at operating temperature, give it a try and see which one works best for you.

![screenshot of manual probe menu](https://artamis.me/assets/images/klipper_guide/KG_10_probe.png)

Manual probe menu

# Additional Installations and Configurations[Permalink](https://artamis.me/projects/klipper_guide/#additional-installations-and-configurations 'Permalink')

## KAMP[Permalink](https://artamis.me/projects/klipper_guide/#kamp 'Permalink')

I highly recommend installing Klipper Adaptive Meshing & Purging ([KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging)) to get the full benefits of improved bed leveling and adaptive purging. The installation steps on their site are pretty straightforward but I’ll provide them here as well.

First you need to define `[exclude_object]` in your `printer.cfg` file.

1. Go to your `printer.cfg` file and add a line that says `[exclude_object]`

   ![screenshot of exclude object module](https://artamis.me/assets/images/klipper_guide/KG_11_exclude_obj.png)

   Exclude object module

2. Go to your `moonraker.conf` file and add 2 lines

   Copy code`[file_manager] enable_object_processing: True`

3. Make sure to go to your slicer and enable the “Label Objects” option. I use Cura and it labels objects by default.

   > (The next steps are copied from the [KAMP Github](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging?tab=readme-ov-file#installation))

4. SSH into your Raspberry Pi and execute the following commands:

   Copy code `cd   git clone https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging.git   ln -s ~/Klipper-Adaptive-Meshing-Purging/Configuration printer_data/config/KAMP   cp ~/Klipper-Adaptive-Meshing-Purging/Configuration/KAMP_Settings.cfg ~/printer_data/config/KAMP_Settings.cfg`

   > **Note:** This will change to the home directory, clone the KAMP repo, create a symbolic link of the repo to your printer’s config folder, and create a copy of `KAMP_Settings.cfg` in your config directory, ready to edit.
   >
   > It is also possible that with older setups of klipper or moonraker that your config path will be different. Be sure to use the correct config path for your machine when making the symbolic link, and when copying `KAMP_Settings.cfg` to your config directory.

5. Open your `moonraker.conf` file and add this configuration:

   Copy code`[update_manager Klipper-Adaptive-Meshing-Purging] type: git_repo channel: dev path: ~/Klipper-Adaptive-Meshing-Purging origin: https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging.git managed_services: klipper primary_branch: main`

   > **Note:** Whenever Moonraker configurations are changed, it must be restarted for changes to take effect. If you do not want moonraker to notify you of future updates to KAMP, feel free to skip this.

6. Depending on what features you want from KAMP, you’ll need to `[include]` some files in `KAMP_Settings.cfg`:

   ![](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging/raw/main/Photos/Include-Tutorial.gif)

   > **Note:** The KAMP configuration files are broken up like this to allow those who do not use bed probes to benefit from adaptive purging, and other features.

7. After you `[include]` the features you want, be sure to restart your firmware so those inclusions take effect. Don’t forget to add `[include KAMP_Settings.cfg]` to your `printer.cfg`!

## Axis Twist Compensation[Permalink](https://artamis.me/projects/klipper_guide/#axis-twist-compensation 'Permalink')

I’ve seen that a lot of people seem to have issues with bed leveling even after they have gone through all the possible solutions for it. The issue I found is that most Ender 3 V3 SE printers have a little twist on their X rail which skews the results of their bed level. [This video](https://www.youtube.com/watch?v=qju5RECjVUM) by Nero3D explains the issue in pretty good detail. I recommend to enable the [Axis Twist Compensation](https://www.klipper3d.org/Axis_Twist_Compensation.html) module and set it up to get your printer running smoothly.

## Calibrate Pressure Advance[Permalink](https://artamis.me/projects/klipper_guide/#calibrate-pressure-advance 'Permalink')

Pressure Advance is another great way to improve your prints, I recommend to follow the Klipper documentation on [Pressure Advance](https://www.klipper3d.org/Pressure_Advance.html) to tune yours correctly.

# Final thoughts[Permalink](https://artamis.me/projects/klipper_guide/#final-thoughts 'Permalink')

I hope you found this guide useful and that you were able to get your Ender 3 V3 SE up and running with Klipper. I also want to recognize the guides and videos I followed to initially set up mine:

- [arismelachroinos](https://www.reddit.com/user/arismelachroinos/) amazing [reddit post](https://www.reddit.com/r/Ender3V3SE/comments/17ijuu9/klipper_on_the_ender_3_v3_se/)
- [BootUse UA video](https://www.youtube.com/watch?v=LrBiwabN-Y8) (Ukranian with english subs)
- pblvsky’s Ender 3 V3 SE’s [guide](https://pblvsky.gitbook.io/ender3v3se/remote-control/klipper)

If you run into any issues while going through this guide, don’t hesitate to reach out and I’ll do my best to help you. Happy printing!

[![Ko-fi donations](https://storage.ko-fi.com/cdn/cup-border.png)Help keep the site ad-free](https://ko-fi.com/B0B6YCPC6 'Support me on ko-fi.com')

**Updated:** May 24, 2024

#### SHARE ON

[Reddit](https://www.reddit.com/submit?url=https%3A%2F%2Fartamis.me%2Fprojects%2Fklipper_guide%2F&title=Ultimate%20Guide%20for%20Klipper%20Installation%20on%20Ender%203%20V3%20SE 'Share on Reddit')  [Twitter](https://twitter.com/intent/tweet?text=Ultimate+Guide+for+Klipper+Installation+on+Ender+3+V3+SE%20https%3A%2F%2Fartamis.me%2Fprojects%2Fklipper_guide%2F 'Share on Twitter')  [Facebook](https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fartamis.me%2Fprojects%2Fklipper_guide%2F 'Share on Facebook')  [LinkedIn](https://www.linkedin.com/shareArticle?mini=true&url=https://artamis.me/projects/klipper_guide/ 'Share on LinkedIn')

[Previous](https://artamis.me/projects/klipper_guide/#)[Next](https://artamis.me/projects/test/ 'Test Project')

- **GET IN TOUCH:**
   -  [EMAIL](mailto:hi@artamis.me)
   -  [DISCORD](https://www.discordapp.com/users/artam1s)
   -  [GITHUB](https://www.github.com/artam1s)
   -  [REDDIT](https://www.reddit.com/user/artam1s/)
   -  [FEED](https://artamis.me/feed.xml)

© 2024 [Artamis](https://artamis.me/). Powered by [Jekyll](https://jekyllrb.com/) & [Minimal Mistakes](https://mademistakes.com/work/minimal-mistakes-jekyll-theme/).
