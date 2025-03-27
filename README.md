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

   Litex:
   For recent pythons it should be installed in a virtual environment. 
   python3 -m venv ~/.venv
   source ~/litex_venv/bin/activate
   Follow these instructions, adapt were needed: https://www.cce.put.poznan.pl/ESHD_labs/env.html
   Integration with Gowin EDA is not immediate, because of the requirements as above.
   For now going with Pro 1.9.11
   add to .bashrc something like

export PATH=$PATH:$PWD/GowinFull/IDE/bin
export PATH=$PATH:$PWD/riscv64-unknown-elf-gcc-8.1.0-2019.01.0-x86_64-linux-ubuntu14/bin/
#following line is needed by Gowin EDA 1.9.11
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libfreetype.so

SOC can be built and loaded this way:
cd ~/litex_venv/litex-boards/litex_boards/targets
./sipeed_tang_primer_20k.py --build --load

--flash instead of --load burns flash

More parameters can be found here:
https://www.controlpaths.com/2022/01/17/building-soc-litex/

uploading the demo app:
add line
from .demo import main
to ~/litex_venv/litex-boards/litex_boards/targets/demo/__init__.py

build with
litex_bare_metal_demo --build-path=~/litex_venv/litex-boards/litex_boards/targets/build/sipeed_tang_primer_20k

open terminal and make ready to uplade demo:
litex_term  --speed=115200  /dev/ttyUSB1 --kernel=~/litex_venv/litex-boards/litex_boards/targets/demo/demo.bin
then at litex bios prompt issue 
serialboot

Sometimes terminal has no communication, see below


 /*********************************************************/


   TODO:

   Sometimes putty or minicom don't receives data from the board, like litex risc-v.
   It seems the board is receiving, because, when communication starts, past data is dumped.
   No flow control set.
   to be tested in Windows
   
