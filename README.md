# Sipeed-Tang-Gowin-FPGA
Some notes about Sipeed Tang Primer 20k FPGA board

  /*********************************************************/

- Gowin EDA rel. 1.9.11.01 does not work with Ubuntu 24.04, with errors like
  qt.glx: qglx_findConfig: Failed to finding matching FBConfig for QSurfaceFormat(version 2.0, options QFlags<QSurfaceFormat::FormatOption>(), depthBufferSize -1, redBufferSize 1, greenBufferSize 1, blueBufferSize 1, alphaBufferSize -1, stencilBufferSize -1, samples -1, swapBehavior QSurfaceFormat::SingleBuffer, swapInterval 1, colorSpace QSurfaceFormat::DefaultColorSpace, profile  QSurfaceFormat::NoProfile)
Previous release, 1.9.11 doesn't have this issue, but AFAIK only the pro version is downloadable right now.

Solved with a start file like this:

#! /bin/sh
export QT_XCB_GL_INTEGRATION=none
env  LD_LIBRARY_PATH=/home/YOUR_PATH_TO_GOWIN/Gowin/IDE/lib ./gw_ide

  /*********************************************************/

  - There is something wrong with the serial lines used by the programmer and terminal.
    For them to appear the ftdi_sio module should be loaded, but we must put a udev rule for this, since the Vendor and id are not as expected.
    Moreover it seems that the programmer is doing raw access to the usb peripheral, and only root can do this, without a proper udev rule.
    This is to be put in /etc/udev/rules.d/99-gowin-rules

    SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", RUN+="/sbin/modprobe ftdi_sio"
    SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", MODE="0666", GROUP="dialer"

    The script suggested in Gowin Programmer User Guide, Gowin_USB_Cable_Installer.sh, doesn't work for me and the rule it creates can be deleted.

   /*********************************************************/

   TODO:

   Sometimes putty or minicom don't receives data from the board, like litex risc-v.
   It seems the board is receiving, because, when communication starts, past data is dumped.
   No flow control set.
   to be tested in Windows
   
