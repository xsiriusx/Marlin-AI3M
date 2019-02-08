# Anycubic i3 Mega Marlin 2.0.x-bugfix by davidramiro

__Not for production use. Use with caution!__

This is my slightly customized version of the [Marlin Firmware](https://github.com/MarlinFirmware/Marlin), gratefully based on [derhopp's repo](https://github.com/derhopp/Marlin-with-Anycubic-i3-Mega-TFT) with his remarkable efforts to get the Anycubic i3 Mega TFT screen to work.

Feel free to discuss issues and work with me further optimizing this firmware!

I am running this version on an i3 Mega Ultrabase V3 (for distinction of the different versions, check [this Thingiverse thread](https://www.thingiverse.com/groups/anycubic-i3-mega/forums/general/topic:27064)).
Basically, this should work on every Ultrabase version that has two Z-axis endstops. The new Mega-S could work too but is not thoroughly tested. E-steps need to be set to 384 (`M92 E384.00` + `M500`), and calibration is recommended as per the instructions below.

Note: This is just a firmware, not magic. A big part of print quality still depends on your slicer settings and mechanical condition of your machine.

## Known issues:

- **Cura users: Please turn off jerk and acceleration control in your print settings (not visible by default, select advanced visibility to unlock them). Cura's high default jerk and acceleration might cause shifted layers if you use TMC2208.**
- Estimated print times from your slicer might be slightly off.
- Special characters on any file or folders name on the SD card will cause the file menu to freeze. Simply replace or remove every special character (Chinese, Arabic, Russian, accents, German & Scandinavian umlauts, ...) from the name. Symbols like dashes or underscores are no problem.
**Important note: On the SD card that comes with the printer there is a folder with Chinese characters in it by default. Please rename or remove it.**
- The firmware is not reflected on the TFT-display. As the display has its own closed source firmware, you will remain to see the original Anycubic menu showing the old version number (1.1.0).
- Cancelling prints via display is buggy sometimes, simply reboot the printer when the menu shows an error. Protip: Switch to OctoPrint.
- A few parts cooling fan models (e.g. some Sunon 5015) might have trouble running slower than 100%. If that's the case, use [this release](https://github.com/davidramiro/Marlin-AI3M/releases/tag/v19.01.22-pwm).


## Why use this?

While the i3 Mega is a great printer for its price and produces fantastic results in stock, there are some issues that are easily addressed:

- Many people have issues getting the Ultrabase leveled perfectly, using Manual Mesh Bed Leveling the printer generates a mesh of the flatness of the bed and compensates for it on the Z-axis for perfect prints without having to level with the screws.
- Much more efficient bed heating by using PID control. This uses less power and holds the temperature at a steady level. Highly recommended for printing ABS.
- Fairly loud fans, while almost every one of them is easily replaced, the stock FW only gives out 9V instead of 12V on the parts cooling fan so some fans like Noctua don't run like they should. This is fixed in this firmware.
- Even better print quality by adding Linear Advance, S-Curve Acceleration and some tweaks on jerk and acceleration.
- Thermal runaway protection: Reducing fire risk by detecting a faulty or misaligned thermistor.
- Very loud stock stepper motor drivers, easily replaced by Watterott or FYSETC TMC2208. To do that, you'd usually have to flip the connectors on the board, this is not necessary using this firmware.
- No need to slice and upload custom bed leveling tests, simply start one with a simple G26 command.
- Easily start an auto PID tune or mesh bed leveling via the special menu (insert SD card, select special menu and press the round arrow)
- Filament change feature enabled: Switch colors/material mid print with `M600` (instructions below)

## How to flash this?

### Compiling

- Download Arduino IDE
- Clone or download this repo
- In the IDE, under `Tools -> Board` select `Genuino Mega 2560` and `ATmega2560`
- Open Marlin.ino in the Marlin directory of this repo
- Customize if needed (e.g. motor directions and type at line `559` to `566` and line `857` to `865` in `Configuration.h`)
- Under `Sketch`, select `Export compiled binary`
- Look for the .hex file in your temporary directory, e.g. `.../AppData/Local/Temp/arduino_build_xxx/` (only the `Marlin.ino.hex`, not the `Marlin.ino.with_bootloader.hex`!)

### After obtaining the hex file:

- Flash the hex with Cura, OctoPrint or similar
- Use a tool with a terminal (OctoPrint, Pronterface, Repetier Host, ...) to send commands to your printer.
- Connect to the printer and send the following commands:
- `M502` - load hard coded default values
- `M500` - save them to EEPROM

## Calibration & Tuning

### Manual Mesh Bed Leveling

If you have issues with an uneven bed, this is a great feature.

- Insert an SD card, enter the print menu.
- Enter the special menu by selecting it and pressing the round arrow:

![Special Menu][menu]

- In this menu, the round arrow is used to execute the command you selected.
- Preheat the bed to 60°C with this entry: (if you usually print with a hotter bed, use the Anycubic menu)

![Preheat bed][preheat]

- Level your preheated bed as well as you can with the four screws.
- Start the mesh leveling:

![Start MMBL][start]

- Your nozzle will now move to the first calibration position.
- Don't adjust the bed itself with screws, only use software from here on!
- Use a paper - I recommend using thermopaper like a receipt or baking paper
- Use the onscreen controls to lower or raise your nozzle until you feel a light resistance: (If you want to send the same command multiple times, select the item again, even though it is still marked red.)

![Z axis controls][control]

- Once finished , move to the next point:

![Next mesh point][next]

- Repeat the last two steps until all 25 points are done.
- Your printer will beep, wait 20 seconds and then save:


![Save to EEPROM][save]

- Reboot your printer.


[menu]: https://kore.cc/i3mega/mmbl/menu.jpg "Special Menu"
[preheat]: https://kore.cc/i3mega/mmbl/preheat.jpg "Preheat 60C"
[start]: https://kore.cc/i3mega/mmbl/start.jpg "Start Mesh leveling"
[next]: https://kore.cc/i3mega/mmbl/next.jpg "Next Mesh point"
[control]: https://kore.cc/i3mega/mmbl/control.jpg "Z axis control"
[save]: https://kore.cc/i3mega/mmbl/save.jpg "Save to EEPROM"

### After leveling:

- Reboot the printer.
- To ensure your mesh gets used on every print from now on, go into your slicer settings and look for the start GCode
- Look for the Z-homing (either just `G28` or `G28 Z0`) command and insert these two right underneath it:
```
M501
M420 S1
```
- Enjoy never having to worry about an uneven bed again!


#### Manual commands for use with OctoPrint etc.:

- `G29 S1` - Start MMBL
- `G29 S2` - Next Mesh point
- Raising Z: `G91`, `G1 Z0.02`, `G90` (one after another, not in one line)
- Lowering Z: `G91`, `G1 Z-0.02`, `G90`
- After seeing `ok` in the console, send `M500` to save.


### Testing your bed leveling

- No need to download or create a bed leveling test, simply send those commands to your printer:
```
G28
G26 C H200 P25 R25
```
- To adjust your filament's needed temperature, change the number of the `H` parameter
- If your leveling is good, you will have a complete pattern of your mesh on your bed that you can peel off in one piece
- Optional: Hang it up on a wall to display it as a trophy of how great your leveling skills are.


### Calibrating extruder & PID


### Extruder steps

- Get your old E-Steps with `M503`. Look for the line starting with `M92`, the value after the `E` are your current steps.
- Preheat the hotend with `M104 S220`
- Send `M83` to prepare the extruder
- Use a caliper or measuring tape and mark 120 mm (measured downwards from the extruder intake) with a pencil on the filament
- Send `G1 E100 F100`
- Your extruder will feed 100 mm of filament now (takes 60 seconds)
- Measure where your pencil marking is now. If it's exactly 20 mm to the extruder, it's perfectly calibrated
- If it's less or more than 20 mm, subtract that value from 120 mm, e.g.:
- If you measure 25 mm, your result would be 95 mm. If you measure 15 mm, your result would be 105 mm
- Calculate your new value: (100 mm / actually extruded filament) * your current E-Steps (default: 92.6)
- For example, if your markings are at 15 mm, you'd calculate: (100/105) * 92.6 = 88.19
- Put in the new value like this: `M92 X80.00 Y80.00 Z400.00 Exxx.xx`, replacing `x` with your value
- Save with `M500`
- Finish with `M82`
- You can repeat the process if you want to get even more precise, you'd have to replace 92.6 with your newly calibrated value in the next calculation.

### PID tuning

**PID calibration is only necessary if you experience fluctuating temperatures.**

- Turn on parts cooling fan If you have a radial blower fan like the original one, I generally recommend running it at 70% because of the 12V mod (`M106 S191`). Remember to also limit it in your slicer.
- Send `M303 E0 S210 C6 U1` to start extruder PID auto tuning
- Wait for it to finish
- Send `M303 E-1 S60 C6 U1` to start heatbed PID auto tuning
- Wait for it to finish
- Save with `M500`, turn off fan with `M106 S0`

Note: These commands are tweaked for PLA printing at up to 210/60 °C. If you run into issues at higher temperatures (e.g. PETG & ABS), simply change the `S` parameter to your desired temperature

**Reminder**: PID tuning sometimes fails. If you get fluctuating temperatures or the heater even fails to reach your desired temperature after tuning, you can always go back to the stock settings by sending `M301 P15.94 I1.17 D54.19` and save with `M500`.

## M600 Filament Change

![M600 Demo][m600 demo]

[m600 demo]: https://kore.cc/i3mega/img/m600demo.jpg "M600 demo"

**A USB host (OctoPrint, Pronterface, ...) is required to use this.**

#### Configuration:
- Send `M603 L0 U0` to use manual loading & unloading.
- Send `M603 L530 U555` to use automatic loading & unloading
- Save with `M500`

#### Filament change process (manual loading):
- Place `M600` in your GCode at the desired layer or send it manually
- The nozzle will park and your printer will beep
- Remove the filament from the bowden tube
- Insert the new filament right up to the nozzle, just until a bit of plastic oozes out
- Remove the excess filament from the nozzle with tweezers
- Send `M108` via your USB host.
- Note for OctoPrint users: After sending `M108`, enable the advanced options at the bottom of the terminal and press `Fake Acknowledgement`

#### Filament change process (automatic loading):
- Place `M600` in your GCode at the desired layer or send it manually
- The nozzle will park
- The printer will remove the filament right up to the extruder and beep when finished
- Insert the new filament just up to the end of the bowden fitting, as shown here:

![Load Filament][m600 load]

[m600 load]: https://kore.cc/i3mega/img/load.jpg "M600 Load"

- Send `M108` via your USB host.
- Note for OctoPrint users: After sending `M108`, enable the advanced options at the bottom of the terminal and press `Fake Acknowledgement`
- The printer will now pull in the new filament, watch out since it might ooze quite a bit from the nozzle
- Remove the excess filament from the nozzle with tweezers


## Updating

### Back up & restore your settings

Some updates require the storage to be cleared (`M502`), if mentioned in the update log. In those cases, before updating, send `M503` and make a backup of all the lines starting with:

```
M92
G29
M301
M304
```

After flashing the new version, issue a `M502` and `M500`. After that, enter every line you saved before and finish by saving with `M500`.


## Detailed changes:

- Thermal runaway protection enabled and tweaked
- Manual mesh bed leveling enabled ([check this link](https://github.com/MarlinFirmware/Marlin/wiki/Manual-Mesh-Bed-Leveling) to learn more about it)
- Heatbed PID mode enabled
- TMC2208 configured in standalone mode
- Stepper orientation flipped (you don't have to flip the connectors on the board anymore)
- Linear advance unlocked (Off by default. [Research, calibrate](http://marlinfw.org/docs/features/lin_advance.html) and then enable with `M900 Kx`)
- S-Curve Acceleration enabled
- G26 Mesh Validation enabled
- Some redundant code removed to save memory
- Minor tweaks on default jerk and acceleration
- Printcounter enabled (`M78`)
- `M600` filament change feature enabled


## Changes by [derhopp](https://github.com/derhopp/):

- 12V capability on FAN0 (parts cooling fan) enabled
- Buzzer disabled (e.g. startup beep)
- Subdirectory support: Press the round arrow after selecting a directory
- Special menu in the SD file menu: Press the round arrow after selecting `Special menu`



# About Marlin

[![Build Status](https://travis-ci.org/MarlinFirmware/Marlin.svg?branch=RCBugFix)](https://travis-ci.org/MarlinFirmware/Marlin)
[![Coverity Scan Build Status](https://scan.coverity.com/projects/2224/badge.svg)](https://scan.coverity.com/projects/2224)

<img align="top" width=175 src="buildroot/share/pixmaps/logo/marlin-250.png" />

Additional documentation can be found at the [Marlin Home Page](http://marlinfw.org/).
Please test this firmware and let us know if it misbehaves in any way. Volunteers are standing by!

## Marlin 2.0 Bugfix Branch

__Not for production use. Use with caution!__

Marlin 2.0 takes this popular RepRap firmware to the next level with support for much faster 32-bit processor boards.

This branch is for patches to the latest 2.0.x release version. Periodically this branch will form the basis for the next minor 2.0.x release.

Download earlier versions of Marlin on the [Releases page](https://github.com/MarlinFirmware/Marlin/releases).

## Building Marlin 2.0

To build Marlin 2.0 you'll need [Arduino IDE 1.9](https://www.arduino.cc/en/main/software) or [PlatformIO](http://docs.platformio.org/en/latest/ide.html#platformio-ide). We've posted detailed instructions on how to [build Marlin 2.0 for ARM](http://marlinfw.org/docs/basics/install_arm.html).

## Hardware Abstraction Layer (HAL)

Marlin 2.0 adds a new abstraction layer so that Marlin can build and run on 32-bit boards while still retaining full 8-bit AVR compatibility. In this way, features can be enhanced for more powerful platforms while still supporting AVR, whereas splitting up the code would make it harder to maintain and keep everything in sync.

### Current HALs

  name|processor|speed|flash|sram|logic|fpu
  ----|---------|-----|-----|----|-----|---
  [Arduino AVR](https://www.arduino.cc/)|ATmega, ATTiny, etc.|16-20MHz|64-256k|2-8k|5V|no
  [Teensy++ 2.0](http://www.microchip.com/wwwproducts/en/AT90USB1286)|[AT90USB1286](http://www.microchip.com/wwwproducts/en/AT90USB1286)|16MHz|128k|8k|5V|no
  [Due](https://www.arduino.cc/en/Guide/ArduinoDue), [RAMPS-FD](http://www.reprap.org/wiki/RAMPS-FD), etc.|[SAM3X8E ARM-Cortex M3](http://www.microchip.com/wwwproducts/en/ATsam3x8e)|84MHz|512k|64+32k|3.3V|no
  [Re-ARM](https://www.kickstarter.com/projects/1245051645/re-arm-for-ramps-simple-32-bit-upgrade)|[LPC1768 ARM-Cortex M3](http://www.nxp.com/products/microcontrollers-and-processors/arm-based-processors-and-mcus/lpc-cortex-m-mcus/lpc1700-cortex-m3/512kb-flash-64kb-sram-ethernet-usb-lqfp100-package:LPC1768FBD100)|100MHz|512k|32+16+16k|3.3-5V|no
  [MKS SBASE](http://forums.reprap.org/read.php?13,499322)|LPC1768 ARM-Cortex M3|100MHz|512k|32+16+16k|3.3-5V|no
  [Azteeg X5 GT](https://www.panucatt.com/azteeg_X5_GT_reprap_3d_printer_controller_p/ax5gt.htm)|LPC1769 ARM-Cortex M3|120MHz|512k|32+16+16k|3.3-5V|no
  [Selena Compact](https://github.com/Ales2-k/Selena)|LPC1768 ARM-Cortex M3|100MHz|512k|32+16+16k|3.3-5V|no
  [Teensy 3.5](https://www.pjrc.com/store/teensy35.html)|ARM-Cortex M4|120MHz|512k|192k|3.3-5V|yes
  [Teensy 3.6](https://www.pjrc.com/store/teensy36.html)|ARM-Cortex M4|180MHz|1M|256k|3.3V|yes

### HALs in Development

  name|processor|speed|flash|sram|logic|fpu
  ----|---------|-----|-----|----|-----|---
  [STEVAL-3DP001V1](http://www.st.com/en/evaluation-tools/steval-3dp001v1.html)|[STM32F401VE Arm-Cortex M4](http://www.st.com/en/microcontrollers/stm32f401ve.html)|84MHz|512k|64+32k|3.3-5V|yes
  [Smoothieboard](http://reprap.org/wiki/Smoothieboard)|LPC1769 ARM-Cortex M3|120MHz|512k|64k|3.3-5V|no

## Submitting Patches

Proposed patches should be submitted as a Pull Request against the ([bugfix-2.0.x](https://github.com/MarlinFirmware/Marlin/tree/bugfix-2.0.x)) branch.

- This branch is for fixing bugs and integrating any new features for the duration of the Marlin 2.0.x life-cycle.
- Follow the [Coding Standards](http://marlinfw.org/docs/development/coding_standards.html) to gain points with the maintainers.
- Please submit your questions and concerns to the [Issue Queue](https://github.com/MarlinFirmware/Marlin/issues).

### [RepRap.org Wiki Page](http://reprap.org/wiki/Marlin)

## Credits

The current Marlin dev team consists of:
 - Roxanne Neufeld [[@Roxy-3D](https://github.com/Roxy-3D)] - English
 - Scott Lahteine [[@thinkyhead](https://github.com/thinkyhead)] - English
 - Bob Kuhn [[@Bob-the-Kuhn](https://github.com/Bob-the-Kuhn)] - English
 - Chris Pepper [[@p3p](https://github.com/p3p)] - English
 - João Brazio [[@jbrazio](https://github.com/jbrazio)] - Portuguese, English

## License

Marlin is published under the [GPL license](/LICENSE) because we believe in open development. The GPL comes with both rights and obligations. Whether you use Marlin firmware as the driver for your open or closed-source product, you must keep Marlin open, and you must provide your compatible Marlin source code to end users upon request. The most straightforward way to comply with the Marlin license is to make a fork of Marlin on Github, perform your modifications, and direct users to your modified fork.

While we can't prevent the use of this code in products (3D printers, CNC, etc.) that are closed source or crippled by a patent, we would prefer that you choose another firmware or, better yet, make your own.
