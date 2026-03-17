# USB-Blaster Firmware for "REV. C USB BLASTER" board with CH552G chip.

This is a fork of [CH55x-USB-Blaster firmware by VladimirDuan](https://github.com/VladimirDuan/CH55x-USB-Blaster) that runs on a cheap clone Altera USB Blaster with CH552G chip.
It works correctly in Linux and Windows with my limited testing(I am testing with MAX V CPLD), no special drivers required. 

After fork vladimirduan repo and made some changes on locally to succesfully program our cpld product with "cheap clone Altera USB Blaster with CH552G" but after a few days I saw [this blog](https://www.downtowndougbrown.com/2024/06/fixing-a-knockoff-altera-usb-blaster-that-never-worked/) and our changes similar except some additional usb fixes(I don't interest linux anyway) but no one give build instruction or included sdk file etc. I decided to mirror @dougg3 repo and implement github action for automatic build and changed repo name for more searchable and then mxwiser decided to fork my repo and adding as mode and then hardware spi support. I don't remove any commit of @dougg3 anyway it's the story for this repo.

#
- This fork added all needed files for correct build.
- Implement CI to build and release binary files so you don't need to setup or download any build tools or sdk files.
- Support AS mode.
- Support hardware SPI now.

#
CH552G can be replaced by CH551G and CH554G, in fact these models have the same Die.

|   PIN   | CH552G |
|:-------:|:------:|
|   LED   |  P1.1  |
|   NCS   |  P1.4  |
|   TDI/ASDI   |  P1.5  |
|   TDO/nCfg-Done   |  P1.6  |
|   TCK/DCLK   |  P1.7  |
|   TMS/nCfg   |  P3.2  |
|   ASDO  |  P3.3  |
|   NCE   |  P3.4  |

Under normal circumstances, the JTAG interface is able to be compatible with the AS interface.
|   1     |   2    |
|:-------:|:------:|
|   TCK/DCLK   |  GND   |
|   TDO/nCfg-Done   |  NC    |
|   TMS/nCfg   | nCE |
|   ASDO    | nCS |
|   TDI/ASDI   |  GND   |

#
Classic JTAG Interface.
|   1     |   2    |
|:-------:|:------:|
|   TCK   |  GND   |
|   TDO   |  VCC   |
|   TMS   |  NC    |
|   NC    |  NC    |
|   TDI   |  GND   |

Classic AS Interface.
|   1       |   2    |
|:---------:|:------:|
|    DCLK   |  GND   |
| nCfg-Done |  VCC   |
|    nCfg   |  nCE   |
|    ASDO   |  nCS   |
|    ASDI   |  GND   |


-----
-----

# RTEK1000 tests:

Gemini (Google IA) instructions:

# USB Blaster Recovery/Programming (CH552G Clone) on Linux

This guide summarizes the steps to flash the USB Blaster emulation firmware onto devices based on the WCH CH552G chip that are not natively recognized by Intel/Altera Quartus.

## 1. Prerequisites
* **Compiler:** `sdcc` (version 4.0+)
* **Flash Tool:** [wchisp](https://github.com/ch32-rs/wchisp) (Recommended for Linux/Bootloader v2.50+) or [releases](https://github.com/ch32-rs/wchisp/releases)
* **Firmware:** [rtek1000/USB-Blaster_CH552](https://github.com/rtek1000/USB-Blaster_CH552)

## 2. Firmware Compilation
Inside the firmware repository folder:
```bash
# SDCC will generate the necessary .hex file
make
# Or compile manually if there is no Makefile:
sdcc main.c # (Adjust the main file as needed)
```

## 3. Bootloader Mode
For the chip to accept the new firmware, it must be detected as a device **WinChipHead (4348:55e0)**:
1. Identify pins **P36 (D+)** and **3V3** on the board.

2. With the device disconnected, short-circuit **P36** and **3V3**.

3. Connect to the USB and remove the short circuit after 1 second.

4. Confirm with `lsusb`:

`Bus XXX Device YYY: ID 4348:55e0 WinChipHead`

## 4. Firmware Flashing
Using `wchisp` (download the pre-compiled binary from the GitHub releases):
```bash
# Check detection
sudo ./wchisp info

# Flash firmware
sudo ./wchisp flash usb_blaster.hex
```

#### Notes:
- After a short period of inactivity in bootloader mode, the IC CH552G resets automatically. You need to be quick: write the command in the prompt, connect the cable to the board, remove the jumper, and then press enter.
- The folder path might interfere with command execution; you may need to use the /tmp folder.

My console logs:

SDCC:
```
sdcc --version
SDCC : mcs51/z80/z180/r2k/r2ka/r3ka/sm83/tlcs90/ez80_z80/z80n/r800/ds390/pic16/pic14/TININative/ds400/hc08/s08/stm8/pdk13/pdk14/pdk15/mos6502/mos65c02/f8 TD- 4.5.0 #15242 (Linux)
published under GNU General Public License (GPL)

```
Make:
```
/tmp/USB-Blaster_CH552-master$ make
sdcc -c -V -mmcs51 --model-small --xram-size 0x0200 --xram-loc 0x0000 --code-size 0x2800 -I/tmp/USB-Blaster_CH552-master/../include -DFREQ_SYS=16000000 --opt-code-speed main.c
+ /usr/local/bin/sdcpp -nostdinc -Wall -std=c11 -I/tmp/USB-Blaster_CH552-master/../include -DFREQ_SYS=16000000 --obj-ext=.rel -D__SDCC_CHAR_UNSIGNED -D__SDCC_MODEL_SMALL -D__SDCC_OPTIMIZE_SPEED -D__SDCC_FLOAT_REENT -D__SDCCCALL=0 -D__SDCC=4_5_0 -D__SDCC_VERSION_MAJOR=4 -D__SDCC_VERSION_MINOR=5 -D__SDCC_VERSION_PATCH=0 -DSDCC=450 -D__SDCC_REVISION=15242 -D__SDCC_mcs51 -D__STDC_NO_COMPLEX__=1 -D__STDC_NO_THREADS__=1 -D__STDC_NO_ATOMICS__=1 -D__STDC_NO_VLA__=1 -D__STDC_ISO_10646__=201409L -D__SIZEOF_FLOAT__=4 -D__SIZEOF_DOUBLE__=4 -D__SDCC_BITINT_MAXWIDTH=64 -isystem /usr/local/bin/../share/sdcc/include/mcs51 -isystem /usr/local/share/sdcc/include/mcs51 -isystem /usr/local/bin/../share/sdcc/include -isystem /usr/local/share/sdcc/include  -xc main.c 
debug.h:18: warning 283: function declarator with no prototype
debug.h:28: warning 283: function declarator with no prototype
debug.h:38: warning 283: function declarator with no prototype
debug.h:68: warning 283: function declarator with no prototype
debug.h:92: warning 283: function declarator with no prototype
debug.h:101: warning 283: function declarator with no prototype
debug.h:115: warning 283: function declarator with no prototype
spi.h:43: warning 283: function declarator with no prototype
spi.h:61: warning 283: function declarator with no prototype
spi.h:70: warning 283: function declarator with no prototype
spi.h:88: warning 283: function declarator with no prototype
+ /usr/local/bin/sdas8051 -plosgffw main.rel main.asm
sdcc -c -V -mmcs51 --model-small --xram-size 0x0200 --xram-loc 0x0000 --code-size 0x2800 -I/tmp/USB-Blaster_CH552-master/../include -DFREQ_SYS=16000000 --opt-code-speed debug.c
+ /usr/local/bin/sdcpp -nostdinc -Wall -std=c11 -I/tmp/USB-Blaster_CH552-master/../include -DFREQ_SYS=16000000 --obj-ext=.rel -D__SDCC_CHAR_UNSIGNED -D__SDCC_MODEL_SMALL -D__SDCC_OPTIMIZE_SPEED -D__SDCC_FLOAT_REENT -D__SDCCCALL=0 -D__SDCC=4_5_0 -D__SDCC_VERSION_MAJOR=4 -D__SDCC_VERSION_MINOR=5 -D__SDCC_VERSION_PATCH=0 -DSDCC=450 -D__SDCC_REVISION=15242 -D__SDCC_mcs51 -D__STDC_NO_COMPLEX__=1 -D__STDC_NO_THREADS__=1 -D__STDC_NO_ATOMICS__=1 -D__STDC_NO_VLA__=1 -D__STDC_ISO_10646__=201409L -D__SIZEOF_FLOAT__=4 -D__SIZEOF_DOUBLE__=4 -D__SDCC_BITINT_MAXWIDTH=64 -isystem /usr/local/bin/../share/sdcc/include/mcs51 -isystem /usr/local/share/sdcc/include/mcs51 -isystem /usr/local/bin/../share/sdcc/include -isystem /usr/local/share/sdcc/include  -xc debug.c 
debug.h:18: warning 283: function declarator with no prototype
debug.h:28: warning 283: function declarator with no prototype
debug.h:38: warning 283: function declarator with no prototype
debug.h:68: warning 283: function declarator with no prototype
debug.h:92: warning 283: function declarator with no prototype
debug.h:101: warning 283: function declarator with no prototype
debug.h:115: warning 283: function declarator with no prototype
debug.c:24: warning 283: function declarator with no prototype
debug.c:157: warning 283: function declarator with no prototype
+ /usr/local/bin/sdas8051 -plosgffw debug.rel debug.asm
sdcc -c -V -mmcs51 --model-small --xram-size 0x0200 --xram-loc 0x0000 --code-size 0x2800 -I/tmp/USB-Blaster_CH552-master/../include -DFREQ_SYS=16000000 --opt-code-speed spi.c
+ /usr/local/bin/sdcpp -nostdinc -Wall -std=c11 -I/tmp/USB-Blaster_CH552-master/../include -DFREQ_SYS=16000000 --obj-ext=.rel -D__SDCC_CHAR_UNSIGNED -D__SDCC_MODEL_SMALL -D__SDCC_OPTIMIZE_SPEED -D__SDCC_FLOAT_REENT -D__SDCCCALL=0 -D__SDCC=4_5_0 -D__SDCC_VERSION_MAJOR=4 -D__SDCC_VERSION_MINOR=5 -D__SDCC_VERSION_PATCH=0 -DSDCC=450 -D__SDCC_REVISION=15242 -D__SDCC_mcs51 -D__STDC_NO_COMPLEX__=1 -D__STDC_NO_THREADS__=1 -D__STDC_NO_ATOMICS__=1 -D__STDC_NO_VLA__=1 -D__STDC_ISO_10646__=201409L -D__SIZEOF_FLOAT__=4 -D__SIZEOF_DOUBLE__=4 -D__SDCC_BITINT_MAXWIDTH=64 -isystem /usr/local/bin/../share/sdcc/include/mcs51 -isystem /usr/local/share/sdcc/include/mcs51 -isystem /usr/local/bin/../share/sdcc/include -isystem /usr/local/share/sdcc/include  -xc spi.c 
spi.h:43: warning 283: function declarator with no prototype
spi.h:61: warning 283: function declarator with no prototype
spi.h:70: warning 283: function declarator with no prototype
spi.h:88: warning 283: function declarator with no prototype
spi.c:61: warning 283: function declarator with no prototype
spi.c:93: warning 283: function declarator with no prototype
spi.c:107: warning 283: function declarator with no prototype
spi.c:135: warning 283: function declarator with no prototype
+ /usr/local/bin/sdas8051 -plosgffw spi.rel spi.asm
sdcc main.rel debug.rel spi.rel -V -mmcs51 --model-small --xram-size 0x0200 --xram-loc 0x0000 --code-size 0x2800 -I/tmp/USB-Blaster_CH552-master/../include -DFREQ_SYS=16000000 --opt-code-speed -o usb_blaster.ihx
+ /usr/local/bin/sdld -nf usb_blaster.lk
objcopy -I ihex -O binary usb_blaster.ihx usb_blaster.bin
packihx usb_blaster.ihx > usb_blaster.hex
packihx: read 116 lines, wrote 211: OK.
```
Flashing:
```
sudo ./wchisp flash usb_blaster.hex
06:29:15 [INFO] Opening USB device #0
06:29:15 [INFO] Chip: CH552[0x5211] (Code Flash: 14KiB, Data EEPROM: 128 Bytes)
06:29:15 [INFO] Chip UID: F5-50-BF-BD-00-00-00-00
06:29:15 [INFO] BTVER(bootloader ver): 02.50
06:29:15 [INFO] Current config registers: ffffffff23000000ff72110900020500f550bfbd00000000
REVERSED: 0xFFFFFFFF
WPROTECT: 0x00000023
  [0:0]   NO_KEY_SERIAL_DOWNLOAD 0x1 (0b1)
    `- Enable
  [1:1]   DOWNLOAD_CFG 0x1 (0b1)
    `- P4.6 / P15 / P3.6(Default set)
GLOBAL_CFG: 0x091172FF
  [15:15] CODE_PROTECT 0x0 (0b0)
    `- Forbid code & data protection
  [14:14] NO_BOOT_LOAD 0x1 (0b1)
    `- Boot from 0xf400 Bootloader
  [13:13] EN_LONG_RESET 0x1 (0b1)
    `- Wide reset, add 87ms reset time
  [12:12] XT_OSC_STRONG 0x1 (0b1)
    `- Enhanced
  [11:11] EN_P5.7_RESET 0x0 (0b0)
    `- Forbid
  [10:10] EN_P0_PULLUP 0x0 (0b0)
    `- Forbid
  [9:8]   RESERVED 0x2 (0b10)
    `- Default
  [7:0]   RESERVED 0xFF (0b11111111)
    `- Default
06:29:15 [INFO] Read usb_blaster.hex as IntelHex format
06:29:15 [INFO] Firmware size: 4096
06:29:15 [INFO] Erasing...
06:29:15 [WARN] erase_code: set min number of erased sectors to 8
06:29:16 [INFO] Erased 8 code flash sectors
06:29:17 [INFO] Erase done
06:29:17 [INFO] Writing to code flash...
██████████████████████████████████████████████████████████████████████ 4096/409606:29:19 [INFO] Code flash 4096 bytes written
06:29:19 [INFO] Verifying...
██████████████████████████████████████████████████████████████████████ 4096/409606:29:19 [INFO] Verify OK
06:29:19 [INFO] Now reset device and skip any communication errors
```

## 5. Permission Configuration (udev rules)
To allow Quartus to access the programmer without `sudo`, create the file `/etc/udev/rules.d/51-usbblaster.rules`:
```text
# Change USB-Blaster
SUBSYSTEM=="usb", ATTR{idVendor}=="09fb", ATTR{idProduct}=="6001", MODE="0666"
SUBSYSTEM=="usb", ATTR{idVendor}=="09fb", ATTR{idProduct}=="6002", MODE="0666"
```
Apply the rules:
```bash
sudo udevadm control --reload-rules && sudo udevadm trigger
```

## 6. Final Verification
Reconnect the USB cable (without shorting the pins). The device should now be listed as Altera Blaster (**09fb:6001**). Test the JTAG communication:

```bash
jtagconfig
```
*The output should list the USB-Blaster and the devices (CPLD/FPGA) detected in the chain.*

---

**Would you like me to help you write a brief technical description of why the CH552G needs this emulation for your repository?**
