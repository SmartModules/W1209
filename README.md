# W1209 Data-Logging Thermostat

[![Travis-CI](https://travis-ci.org/TG9541/W1209.svg?branch=master)](https://travis-ci.org/TG9541/W1209)

This project uses [STM8 eForth](https://github.com/TG9541/stm8ef) to turn the off-the-shelf board [W1209][] into a data logging thermostat with a console interface *and an embedded programming environment*. It provides [source code](https://github.com/TG9541/W1209), a ready-to-use [firmware](https://github.com/TG9541/W1209/releases), and [documentation](https://github.com/TG9541/W1209/wiki).

![w1209](https://user-images.githubusercontent.com/5466977/34077952-4023969c-e310-11e7-8313-49407c417ac9.png)

Features are:

* a heating thermostat, e.g. for building a chicken egg incubator
* no special tool installation necessary
  * ready-made binaries, and source code, are provided
  * new binaries can be built with the help of Travis-CI
  * interactive programming in Forth, even while the thermostat task is running!
  * any serial terminal program can be used, e.g. picocom, cutecom, or e4thcom
* data logger with a 144 entry ring-buffer, a 0.1h to 10h intervall, and log access through the serial console
  * `L.dump` command prints the log through the serial console - the last line is the latest entry
  * `L.wipe` command erases the log memory
  * records lowest and highest temperature
  * records the number of relay cycles and the heating duty cycle
* detection of "sensor disconnected failure"
* easy to use parameters menu for set-point, hysteresis, and trip-delay

This is work in progress although it's feature-complete. Please consider the software "beta" (please write an issue if you need support or want to give feedback :-) ). 

Possible future features:

* a simple "field-bus" for building a network of thermostat units
* more fail-safe features (e.g. parameter integrity, limits monitoring)

Please note that implementing new features may require using a more compact STM8 eForth build, or removing existing features.

With minor modifications the code should also work for [other generic thermostat boards](https://github.com/TG9541/stm8ef/wiki/STM8S-Value-Line-Gadgets#thermostats) for which STM8s eForth support exists.

## Getting Started

**Note: STM8 eForth only works on supported STM8 chips - W1209 boards with a Nuvoton chip will have to be modified before they can be used with the code in this repository!**

**512 of the 576 bytes logging buffer rely on an undocumented STM8S003F3P6 feature which *might* not work in some chips. Using an STM8S103F3P6 or an STM8S903F3P6 chip will fix that.** 

After programming the [firmware binary](https://github.com/TG9541/W1209/releases) to the W1209 board, it should work as normal thermostat. Parameters can be set using the board keys (`set`, `+`, `-`).

The following items are recommended for programming:

W1209|ST-Link programmer|TTL-Serial-Interface
-|-|-
![W1209](https://user-images.githubusercontent.com/5466977/33417013-d2b29dec-d59f-11e7-8187-e608e856fe16.png)|![Programmer](https://ae01.alicdn.com/kf/HTB1QVvYRXXXXXa5XFXXq6xXFXXXP/ST-Link-V2-stlink-mini-STM8STM32-STLINK-simulator-download-programming-With-Cover.jpg_220x220.jpg)|![TTL-Serial](https://ae01.alicdn.com/kf/HTB1x__9OFXXXXc7XVXXq6xXFXXX6/-Free-Shipping-CH340-module-USB-to-TTL-CH340G-upgrade-download-a-small-wire-brush-plate.jpg_220x220.jpg)

Please refer to the [STM8 eForth Wiki](https://github.com/TG9541/stm8ef/wiki/STM8S-Programming#flashing-the-stm8) for instructions on programming the W1209 using an ST-Link compatible programmer.

After programming, the display should show the temperature value (in °C), or `.dEF.` (default) if no sensor is connected).

Before using the thermostat, please reset the parameter values by holding the keys `+` and `-` until the text `rES.` appears on the LED display (about 4s). Pressing the `set` key leads to the parameter menu. The menu returns to the temperature display when no key is pressed for more than 10s.

The software currently supports the following parameters:

Display|Range|Default|Unit|Description
-|-|-|-|-
`SEt.`| 10.0 - 80.0 |37.5| °C| Heating thermostat set point (switch off above)
`LoG.`| 0.0 - 10.0 | 10.0 |h| Logger interval in hours
`dEL.`| 0.0 - 60.0 | 0.0 | s | thermostat heating trip delay
`Cor.`| -2.0 - 2.0 | 0.0 | °C | thermometer offset (for corrections around desired set-point)
`hYS.`| 0.1 - 2.0 | 0.5 | °C | thermostat hysteresis (difference between the lower and the upper trip points)

Note that in most cases hanging the trip delay parameter `dEL.` should not be required.

## Using the Data Log

The data logger feature uses the upper 576 bytes of the internal EEPROM as a 144 entry ring-buffe. The logger interval (time between samples) can be defined in the range from 6 minutes (0.1h) to 10h by the menu item `LoG.`.

The following items are recorded:

  * Lowest temperature
  * Highest temperature
  * Heating duty cycle `DC = 100% * t.on/(t.on + t.off)`
  * Number of relay cycles

The data log can be accessed through the Forth console with the command `L.dump`. The log can be wiped with the command `L.wipe`. To use the Forth console, connect a serial interface adapter to the `+` and `-` key pins.

The following chart demonstrates the influence of insulation improvements, a hysteresis parameter change, and the effect of heating temperature setback overnight in my living room:

![w1209-log2](https://user-images.githubusercontent.com/5466977/34077539-cce83cea-e306-11e7-849c-5c1c5faae08b.png)

Such a chart can be created with the following steps:

* set the log interval according to the required observation time
  * 0.1h for control optimization
  * e.g. 3.5h for the 3 weeks that it takes to hatch a chicken egg
* let the thermostat do its work (no intervention required)
* connect a TTL-RS232 interface to the keys (RX:`-`, TX `+` - pins near the LED display)
* access the console with a serial terminal program with the settings "9600N81"
  * for Linix e.g. e4thcom, minicom, picoterm, or miniterm.py
  * for Windos e.g. miniterm.py, PuTTY, Hyperterminal
  * press the `ENTER` key - STM8 eForth should reply with ` ok`
* type `L.dump` to extract the data (note: the last line is "now")
* take a note of the read out time, and the log interval)
* copy and paste the data to a spread sheet program
* use the known read-out time, and the log intervall for creating a time axis for an x/y chart

## The Thermostat Controller

`control.fs` implements a very simple 2-point controller with hysteresis, and delay. There is no other reason for either of the parameters other than they can be used for mitigating noise, which the sensor measurement already takes care of.

In future versions it may be replaced by a simple PI controller, where the relay duty cycle is the control variable.

## Working with the Code in this Repository

Clone this repository, an run `make depend` for dependency resolution. This will download an STM8 eForth binary, and add required folders, files, and symlinks.

The general workflow for set-up is this:

* clone the repository
* install [stm8flash](https://github.com/vdudouyt/stm8flash)
* [connect a ST-LINK-V2 dongle to a W1209][W1209]
* run `make defaults` to wipe the stock firmware
  * warning: there is no known public source for the stock firmware, and after erasing it there is no way back!
* run `make` to flash the STM8EF binary
* for interactive scripting install [e4thcom](
https://wiki.forth-ev.de/doku.php/en:projects:e4thcom)
* optionally install the development version of ucSim (or use the Docker image) to take advantage of off-line image creation

For [programming the W1209 binary](https://github.com/TG9541/W1209/blob/master/out/W1209-FD/W1209-FD.ihx) please follow the instructions in the [STM8EF Wiki page for board W1209](
https://github.com/TG9541/stm8ef/wiki/Board-W1209#flashing-the-stm8ef-binary) (if `stm8flash` is installed just run `make flash`).

Interactive scripting through the serial console is supported by the STM8 eForth base binary. Please refer to the [instructions for getting a serial console](https://github.com/TG9541/stm8ef/wiki/Board-W1209#serial-communication-through-the-key-pins).

The recommended terminal emulator is [e4thcom](https://wiki.forth-ev.de/doku.php/en:projects:e4thcom): it supports line editing, and upload of source files with `#include`, and using a library with `#require`. Type `#i main.fs` to load the complete source code.

For Continuous Integration use cases `make simload` applies ucSim to create an STM8 binary file that contains the full thermostat script, including the W1209-FD base image. The Docker image [tg9541/docker-sdcc](https://hub.docker.com/r/tg9541/docker-sdcc/) contains tool dependencies for Continuous Integration (refer to `.travis.yml` or use the Travis-CI badge to browse the execution log).

## About the STM8 eForth Base System

The code is based on the [STM8EF binary release](https://github.com/TG9541/stm8ef/releases). The Makefile uses the "modular build" method to automatically build a binary for the board support folder `W1209-FD`.

Please refer to the [STM8EF Wiki](https://github.com/TG9541/stm8ef/wiki) for more information.

## Contributing

This is a community projecy - it's driven by user contributions!

Please [write an issue](https://github.com/TG9541/W1209/issues) if you have questions, post a comment in the [HACKADAY.IO project][HAD1], or contribute docs, code, new use-cases and requirements.

Also consider writing about it in forums, blogs, on Twitter, or make a YouTube video in your native language, so that others can find it (please use #W1209 and #STM8EF hashtags).

## Commercial Use

The code in this repository can be used for commercial applications in compliance with to the [license conditions](https://github.com/TG9541/W1209/blob/master/LICENSE).

[HAD1]: https://hackaday.io/project/26258-w1209-data-logging-thermostat
[W1209]: https://github.com/TG9541/stm8ef/wiki/Board-W1209
