# CSR SPI programmer

This is the driver for SPI programmer based on FTDI FT232R or later chip using
synchronous bitbang mode. Currently it supports running CSR tools (such as
BlueLab or BlueSuite) under Windows or under Linux with Wine.

## CSR SPI API versions

This driver implements CSR SPI API version 1.3 and 1.4, different DLLs built
for each API version during a compile time. Example of CSR packages that use
API version 1.4:

* CSR BlueSuite 2.4, 2.5, 2.5.8

Example of CSR packages using API version 1.3:

* CSR BlueSuite 2.1, 2.3
* CSR BlueLab 4.1

You can check the API version of CSR package by inspecting original spilpt.dll
with the following command on Linux:

    winedump -j export spilpt.dll | grep spifns_stream_

If the output is not empty, then the original DLL implements newer version of
API (version 1.4).

New versions of BlueSuite can be found at https://www.csrsupport.com/PCSW.
Old versions of BlueSuite can be found at
https://www.csrsupport.com/PCSWArchive. Access to these pages requires
registration.

## Building for Wine

### Building DLL on 32-bit Debian/Ubuntu Linux

Install build tools

    sudo apt-get install -y build-essential

Install development libraries.

    sudo apt-get install -y wine-dev libc6-dev libstdc++-dev libftdi-dev

Build with command:

    make -f Makefile.wine all


### Building DLL on 64-bit Debian/Ubuntu Linux

Install build tools

    sudo apt-get install -y build-essential gcc-multilib g++-multilib

Install 32 bit stuff.

    sudo apt-get install -y wine-dev:i386 libc6-dev-i386 libstdc++-dev:i386 libftdi-dev:i386

Build with command:

    make -f Makefile.wine all


### Installing

Install CSR BlueSuite in wine. Find all instances of spilpt.dll installed and
move them out of the way:

    find ~/.wine -iname spilpt.dll -execdir mv {} {}.orig \;

Install wine dll into the wine libraries directory:

    sudo make -f Makefile.wine SPIAPI=<SPI_API_version> install

where *SPI_API_version* is one of "1.3" or "1.4".

## Pinout

You can build a simple programmer using popular FTDI adapter boards. Pinout
specified in spi.c file. Change it at will. Beware that popular FTDI adapters
provide 5V or 3V3 I/O levels while CSR chips require 3V3 or 1V8 I/O level. You
may supply appropriate VCCIO to FTDI chip or use logic level converter if
levels don't match. See description of VCCIO pin in FTDI chip datasheet for
details.

| Signal | FT232RL pin | FTDI pin name | FTDI GPIO pin | CSR pin | HC-0x pin |
| ------ | ----------- | ------------- | ------------- | ------- | --------- |
| MOSI | 1 | TXD | D0 | SPI_MOSI | 17 |
| MISO | 5 | RXD | D1 | SPI_MISO | 18 |
| CLK | 3 | RTS# | D2 | SPI_CLK | 19 |
| CS# | 2 | DTR# | D4 | SPI_CS# | 16 |
| LED_WR | 6 | RI# | D7 | -- | -- |
| LED_RD | 9 | DSR# | D5 | -- | -- |

LED connections are optional. Wire LEDs cathodes through the current limiting
resistors (330 Ohm works fine for 3V3) to the appropriate FTDI
pins. Wire LEDs anodes to FTDI VCCIO pin.

## Building DLL for Windows

### Cross-compiling DLL on Debian/Ubuntu using MinGW

Install MinGW cross-development environment:

    sudo apt-get install -y gcc-mingw32

Download precompiled libusb for windows from
http://sourceforge.net/projects/libusb/files/ and extract it to the libusb
directory:

    wget http://sourceforge.net/projects/libusb/files/libusb-1.0/libusb-1.0.18/libusb-1.0.18-win.7z
    7z x -olibusb libusb-1.0.18-win.7z

Download precompiled liftdi for windows from
http://sourceforge.net/projects/picusb/files/ and extract it:

    wget http://sourceforge.net/projects/picusb/files/libftdi1-1.1_devkit_x86_x64_21Feb2014.zip
    unzip libftdi1-1.1_devkit_x86_x64_21Feb2014.zip
    ln -s libftdi1-1.1_devkit_x86_x64_21Feb2014 libftdi1

Build with command:

    make -f Makefile.mingw all

### Install on Windows

1. Install CSR package such as BlueSuite;
2. Make a backup of spilpt.dll in your application directory (e.g. in
   *C:\Program Files (x86)\CSR\BlueSuite 2.5.8\*);
3. Copy appropriate version of spilpt.dll from spilpt-1.4-win32 or
   spilpt-1.3-win32 directory to your application directory;
4. Connect Your FTDI device to computer;
5. Download and run Zadig from http://zadig.akeo.ie/. In Options menu choose
   "List all devices", choose Your FTDI device ("FT232R USB UART" or similar),
   choose libusbK driver, press "Replace driver" or "Install driver".  This
   will install generic libusb-compatible driver for your FTDI chip. There is a
   nice [tutorial](http://embedded-funk.net/running-libftdi-under-windows/) on
   running libftdi programs on Windows;
6. Run your CSR apps.


## BUGS

* Driver sometimes fails with error "Unable to start read (invalid control
  data)". The problem is clearly in SPI communication, but I still can not
  figure out the cause. Anyway, restarting operation helps.
* Driver for API 1.4 does not support more than one FTDI device connected to
  the computer at the same time. This is due to a limited stream API
  implementation.


## Thanks
* This project is heavily based on Frans-Willem Hardijzer's [reverse-engineered
  spilpt.dll drivers](https://github.com/Frans-Willem/CsrSpiDrivers).
* Thanks to *unicorn* from http://www.nebo-forum.kiev.ua/ for the idea of a DLL
  for wine.


## Related projects
* [LPT programmer and general info](http://byron76.blogspot.com/) by Robin
  Gross;
* [Similar SPILPT
  driver](http://www.nebo-forum.kiev.ua/viewtopic.php?p=58291#p58291) for Wine
  under Linux by *unicorn* using FTDI MPSSE;
* [Arduino SPILPT driver](https://github.com/Frans-Willem/CsrSpiDrivers) by
  Frans-Willem Hardijzer - for Windows;
* [USBSPI programmer based on CSR chip using original
  formware](http://jernej87.blogspot.com/) by Jernej Škrabec:
 * [Using USBSPI on
   Linux](http://jernej87.blogspot.com/2012/10/dumping-bluecore4-firmware-on-linux.html);
 * [USBSPI protocol
   analisys](http://jernej87.blogspot.com/2012/10/csrs-usb-programmer-protocol-analysis.html);
* [USBSPI programmer based on Stellaris
  Launchpad](https://github.com/Frans-Willem/CsrUsbSpiDeviceRE) by Frans-Willem
  Hardijzer - for Windows;
 * [Port to Tiva C Launchpad](https://github.com/raplin/CsrUsbSpiDeviceRE) by
   Richard Aplin;
