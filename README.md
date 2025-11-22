# Sipeed-Tang-Gowin-FPGA
Some notes about Sipeed Tang Primer 20k FPGA board
Most of the notes use https://github.com/alberto-grl/SoCmel_1 for examples
  /*********************************************************/

- Gowin EDA rel. 1.9.11.01 does not work with Ubuntu 24.04, with errors like
 
`qt.glx: qglx_findConfig: Failed to finding matching FBConfig for QSurfaceFormat(version 2.0, options QFlags<QSurfaceFormat::FormatOption>(), depthBufferSize -1, redBufferSize 1, greenBufferSize 1, blueBufferSize 1, alphaBufferSize -1, stencilBufferSize -1, samples -1, swapBehavior QSurfaceFormat::SingleBuffer, swapInterval 1, colorSpace QSurfaceFormat::DefaultColorSpace, profile  QSurfaceFormat::NoProfile)`   
  
Previous release, 1.9.11 doesn't have this issue, but AFAIK only the pro version is downloadable right now.

Solved with a start file like this:

`#! /bin/sh   
export QT_XCB_GL_INTEGRATION=none   
env  LD_LIBRARY_PATH=/home/YOUR_PATH_TO_GOWIN/Gowin/IDE/lib ./gw_ide
`    
  /*********************************************************/     

  - There is something wrong with the serial lines used by the programmer and terminal.    
    For them to appear the ftdi_sio module should be loaded, but we must put a udev rule for this, since the Vendor and id are not as expected.    
    Moreover it seems that the programmer is doing raw access to the usb peripheral, and only root can do this, without a proper udev rule.    
    This is to be put in /etc/udev/rules.d/99-gowin-rules

    `SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", RUN+="/sbin/modprobe ftdi_sio"   
    SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", MODE="0666", GROUP="dialer"`
    

    The script suggested in Gowin Programmer User Guide, Gowin_USB_Cable_Installer.sh, doesn't work for me and the rule it creates can be deleted.   

   /*********************************************************/   

   Litex:    
   For recent pythons it should be installed in a virtual environment.     
        
   'python3 -m venv ~/.venv    
   source ~/litex_venv/bin/activate`    
       
   Follow these instructions, adapt where needed: https://www.cce.put.poznan.pl/ESHD_labs/env.html     
   Integration with Gowin EDA is not immediate, because of the requirements as above.    
   For now going with Pro 1.9.11   
   add to .bashrc something like   
   
`export PATH=$PATH:$PWD/GowinFull/IDE/bin   
export PATH=$PATH:$PWD/riscv64-unknown-elf-gcc-8.1.0-2019.01.0-x86_64-linux-ubuntu14/bin/    
     
#following line is needed by Gowin EDA 1.9.11    
    
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libfreetype.so    

~/litex_venv/litex-boards/litex_boards/platforms/sipeed_tang_primer_20k.py wrongly defines some pins at a voltage not available for that bank.`    
   
Relevant lines should be changed to    

    `("led", 0,  Pins( "CARD1:44"), IOStandard("LVCMOS18")),    
    ("led", 1,  Pins( "CARD1:46"), IOStandard("LVCMOS18")),    
    ("led", 3,  Pins( "CARD1:40"), IOStandard("LVCMOS18")),    
    ("led", 2,  Pins( "CARD1:42"), IOStandard("LVCMOS18")),    
    ("led", 4,  Pins( "CARD1:98"), IOStandard("LVCMOS33")),    
    ("led", 5,  Pins("CARD1:136"), IOStandard("LVCMOS33")),`    

     
SOC can be built and loaded this way:    
   
'cd ~/litex_venv/litex-boards/litex_boards/targets   
   
./sipeed_tang_primer_20k.py --build --load`   
   
--flash instead of --load burns flash   

More parameters can be found here:   

`https://www.controlpaths.com/2022/01/17/building-soc-litex/`   

There are some MakeSOC scripts in ~/litex_venv/bin/socmel_1 
   
Uploading the demo app:    
add line   

`from .demo import main`    
    
to `~/litex_venv/litex-boards/litex_boards/targets/demo/__init__.py`    
    
build with    
`cd ~/litex_venv/litex/litex/soc/software/demo`
`litex_bare_metal_demo --build-path=~/litex_venv/litex-boards/litex_boards/targets/build/sipeed_tang_primer_20k`     
    
open terminal and make ready to upload the demo:    

`litex_term  --speed=115200  /dev/ttyUSB1 --kernel=~/litex_venv/litex-boards/litex_boards/targets/demo/demo.bin`   
   
then at litex bios prompt:    
`serialboot`    
    
Sometimes terminal has no communication, see below    
   
 /*********************************************************/    

After working OK for several days, Gowin programmer refuses to start and gives error    
   
`./programmer: /home/alberto/GowinFull/Programmer/bin/libz.so.1: version `ZLIB_1.2.3.4' not found (required by /lib/x86_64-linux-gnu/libpng16.so.16)`    
Solved by renaming or deleting GowinFull/Programmer/bin/libz.so.1, the system's version will be used instead.    
   
 /*********************************************************/   
   
 openFPGAloader is working erratically for me, it exits with errors about the serial line.    
 Not using it for now, do not use the --flash option with the generated SOC .py in targets.   
 Using instead the Gowin programmer, image to be flashed is like   
    
 `/home/alberto/litex_venv/litex-boards/litex_boards/targets/build/sipeed_tang_primer_20k/gateware/sipeed_tang_primer_20k.fs`   
    
TODO: there should be a command line Gowin programmer, use it   
    
 /*********************************************************/   
    
linux-on-litex-vexriscv     
   
Follow the instructions on this Github page:   
    
`https://github.com/litex-hub/linux-on-litex-vexriscv?tab=readme-ov-file#-generating-the-linux-binaries-optional`    
    
Some additional tools should be installed, but it's quite easy.    
    
During boot there could be an error like:    

`[    3.886195] Kernel panic - not syncing: Scratch register read error - the system is probably broken! Expected: 0x12345678 but got: 0x0 [    3.895050] CPU: 0 PID: 1 Comm: swapper/0 Not tainted 5.14.0 #1 [    3.899034] Call Trace: [    3.900525] [<c0003b64>] dump_backtrace+0x2c/0x3c [    3.904063] ---[ end Kernel panic - not syncing: Scratch register read error - the system is probably broken! Expected: 0x12345678 but got: 0x0 ]`   
    
It is because there is a bad rv32.dtb file floating around in linux_2022_03_23.zip, in my case it is 1999 bytes long.    
    
'git clone https://github.com/litex-hub/linux-on-litex-vexriscv'    
    
cd to its root directory and do    

 `./make.py --board sipeed_tang_primer_20k`      
    
A rv32.dtb will be genearated in images. For me it is 2817 bytes long.    
Don't exactly know how it happened, but once I got a sipeed_tang_primer_20k.dbt.   
Just rename it and you should be fine.    
Replace the short file and Linux should boot.    
     
  /*********************************************************/      
  Wishbone communication with SoC, UDP      
        
- Generating script should expose desired signals. Use     
     
   `# Leds -------------------------------------------------------------------------------------     
        #if with_led_chaser:    
        #    self.leds = LedChaser(     
        #        pads         = platform.request_all("led"),     
        #        sys_clk_freq = sys_clk_freq     
        #    )     
        led_pads = platform.request_all("led")     
        self.leds = GPIOOut(led_pads)     
        self.submodules += self.leds     
        self.add_csr("leds")  # Expose over CSR     
     # Buttons ----------------------------------------------------------------------------------      
        if with_buttons:     
            self.buttons = GPIOIn(pads=~platform.request_all("btn_n"))     
            self.add_csr("buttons")  # Expose over CSR`     
    
   - generate gateware with support for wishbone and csr.csv . This file contains the addressess for the various peripherals,     
    it is required for symbolic access to them but not generated by default.     
      
`./sipeed_tang_primer_20k-1.py --soc-csv=soc.csf --with-etherbone --cpu-type=vexriscv --cpu-variant=lite --l2-size=512 --with-spi-sdcard --eth-ip 192.168.11.50 --build --csr-csv build/sipeed_tang_primer_20k/csr.csv --doc`     
    
     add option --flash or program with Gowin programmer     
    Default IP address would be 192.168.1.50, can be changed with --eth-ip 192.168.11.50     
     
- start wishbone server     
  activate virtual environment:

` source ~/litex_venv/bin/activate     
  litex_server --udp --udp-ip=192.168.11.50 --debug`    
     
- Script for communication     
     
  `#!/usr/bin/env python3      
     
import time     
from litex import RemoteClient     
     
wb = RemoteClient(csr_csv="/home/alberto/litex_venv/litex-boards/litex_boards/targets/build/sipeed_tang_primer_20k/csr.csv" )       
wb.open()       
     
# Dump all CSR registers of the SoC     
for name, reg in wb.regs.__dict__.items():     
    print("0x{:08x} : 0x{:08x} {}".format(reg.addr, reg.read(), name))    
print(f"Writing to address: {hex(wb.bases.leds)}")    
wb.regs.leds_out.write(0x73)    
print("Readback:", hex(wb.regs.leds_out.read()))    
    
print(hex(wb.read(0x40000000)))    
wb.write(0x40000000, 0xabcd)    
print(hex(wb.read(0x40000000)))    
print(hex(wb.read(0xF0000000)))    
wb.write(0xF0000000, 0xFFFFFFFF)    
print(hex(wb.read(0xF0000000)))    
     
wb.regs.leds_out.write(0x03)    
val = wb.regs.leds_out.read()    
print(f"LEDs = {val:#x}")    
    
# Read from a memory-mapped register    
value = wb.read(0xF0000800)    
print("Buttons = ",value)    
wb.close()`    
    
 /*********************************************************/     
    
Some useful links:    
`https://www.controlpaths.com/2022/01/17/building-soc-litex/    
https://github.com/enjoy-digital/litex/wiki/CSR-Bus`       
    
 /*********************************************************/    
     
# Building and loading of a C program in Vexriscv   
    
TODO: process is quite convoluted. Try to streamline    
    
It's easier to start with the demo software. Create a directory like /home/alberto/litex_venv/litex/litex/soc/software/socmel_1   
Copy all the demo source directory into it, /home/alberto/litex_venv/litex/litex/soc/software/sdr_v1/    
TODO: modify makefile and sdr_v1.py, using your name of choice.    
Create script in /home/alberto/litex_venv/bin/    
it should be a copy of /home/alberto/litex_venv/bin/litex_bare_metal_demo    
Modify the from line according to your source directory, like     
`from litex.soc.software.socmel_1.sdr_v1     import main    
Create 
`/home/alberto/litex_venv/litex-boards/litex_boards/targets/test/build/sipeed_tang_primer_20k_socmel_1`
Makefile should be modified too, for your different C files     
There is an example of an a completely customized c project named socmel_v1.    

Build is by     

`:/home/alberto/litex_venv/litex/litex/soc/software/socmel_1$ litex_make_sdr_v1 --build-path=/home/alberto/litex_venv/litex-boards/litex_boards/targets/test/build/sipeed_tang_primer_20k_socmel_1`   

It's important to launch the above command from the directory where c files are     
     
Then uploading to RAM is done with    
    
`:/home/alberto/litex_venv/litex/litex/soc/software/socmel_1$ litex_term --speed=115200 /dev/ttyUSB1 --kernel=sdr_v1/sdr_v1.bin`    
   
Do it twice when you don't get the prompt.    
Then issue    

`serialboot`      
or     
`reboot`     
to litex bios and admire liftoff.     

for SD: rename the .bin output file to boot.bin, load it on an SD  card.     
`https://github.com/enjoy-digital/litex/wiki/Load-Application-Code-To-CPU`
	
TODO: use flash 
     
In `/home/alberto/litex_venv/litex-boards/litex_boards/targets/socmel_1/build/sipeed_tang_primer_20k_socmel_1/software/include/generated/`      
the auto generated csr.h defines constants an function prototype that can be used for accessing hardware, like      
	`myperiph_csr0_write(0x5a5a5a5a);`    
 Pin toggle is abt 80 ns, can generate a 6 MHz square wave in a while loop.    
       
/*********************************************************/      
   TODO:    
     
   Sometimes putty or minicom don't receives data from the board, like litex risc-v.     
   It seems the board is receiving, because, when communication starts, past data is dumped.    
   No flow control set.     
   to be tested in Windows    
   It seems that at first invocation the terminal program never works. Close the terminal, reopen and you should be ok.      
   Ethernet still not working, but at least can be generated by lowering l2 size --l2-size=512     
     
   The short usb cable included with this Tang sometimes gives errors. With another cable, errors during long transfers disappeared.    
   Maximum speed for me is 460800 baud, doubling that it doesn't work anymore and the board should be power cycled.     
   Strangely, the two step invocation should be performed at 115200. Once communmication is established exit litex_term, change speed to 460800 and launch    again.
   
