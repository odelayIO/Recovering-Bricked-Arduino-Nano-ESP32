# Recovering Bricked Arduino Nano ESP32

## Introduction

If you have programmed an Arduino board using the Arduino IDE, and the application that was just loaded continuously resets the Arduino board preventing a new program from being loaded through the Arduino IDE.  The board is considered `bricked` since you can re-configure the board through the Arduino IDE.  

This is a process to recovery a bricked Arduino Nano ESP32, but these instructions can be used for any ESP based development board.  The process to recovery a bricked Arduino board is to copy the flash from a functional board, then copy the flash memory to the bricked Arduino board.  The board will be held in Boot mode during this process, preventing the ESP32 chip from loading the Bootloader and continuously resetting.



### Requirements

Please note you will need to short **B1** pin to **GND** pin to read/write to flash memory using the ESPTool command line tool.  More information about the B1 pin can be found in the link below.

Source: https://support.arduino.cc/hc/en-us/articles/9625819325212-About-the-B1-and-B0-pins-on-the-Nano-ESP32

#### ESPTool Command Line Application

##### Windows Users:

This process requires the `ESPTool` to read/write the flash memory.  The application will be in the Arduino application folder.  For example `C:\Users\odelay\AppData\Local\Arduino15\packages\esp32\tools\esptool_py\4.5.1`.  Copy the application to a new folder on your Windows machine (e.g. `W:\nano_esp32\bricked`).  

##### Linux Users:

Download the `ESPTool` on Linux using the instructions on ESPressIF website. 

https://docs.espressif.com/projects/esptool/en/latest/esp32/installation.html

For most Linux systems it can be installed from `pip`, using the `pip install esptool`



## Reading Arduino Nano ESP32 Flash Information

1. Disconnect a functional Arduino Nano ESP32 board
2. Short B1 pin to the GND pin on the Arduino Nano ESP32
3. Run the following command `esptool.exe --port COM5 --baud 921600 flash_id`

Example:

```bash
W:\nano-esp32\bricked>esptool.exe --port COM5 --baud 921600 flash_id
esptool.py v4.5.1
Serial port COM5
Connecting...
Detecting chip type... ESP32-S3
Chip is ESP32-S3 (revision v0.2)
Features: WiFi, BLE
Crystal is 40MHz
MAC: 74:4d:bd:a0:6c:74
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 921600
Changed.
Manufacturer: c8
Device: 4018
Detected flash size: 16MB
Flash type set in eFuse: quad (4 data lines)
Hard resetting via RTS pin...
```

NOTE: Flash Memory is 16MB (0x1000000)



## Reading Flash Memory on a Functional Arduino Board

Based on the size of the flash memory, issue a command to read the whole contents of the flash memory to a `bin` file.  The flash size was stated in the `flash_info` command in the previous step.

1. Disconnect a functional Arduino Nano ESP32 board
2. Short B1 pin to the GND pin on the Arduino Nano ESP32
3. Now read the flash memory to a `bin` file using command: `esptool.exe --port COM5 --baud 921600 read_flash 0 0x1000000`

```
W:\nano-esp32\bricked>esptool.exe --port COM5 --baud 921600 read_flash 0 0x1000000 nano32blink16mb.bin
esptool.py v4.5.1
Serial port COM5
Connecting...
Detecting chip type... ESP32-S3
Chip is ESP32-S3 (revision v0.2)
Features: WiFi, BLE
Crystal is 40MHz
MAC: 74:4d:bd:a0:6c:74
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 921600
Changed.
9891840 (58 %)

```



## Writing Flash Memory on Bricked Board

Now that the `bin` file has been read from the functional board, the next want to write the `bin` file to the flash memory on the bricked Arduino board.

1. Disconnect a `bricked` Arduino Nano ESP32 board
2. Short B1 pin to the GND pin on the Arduino Nano ESP32
3. Issue the command `esptool.exe  --port COM8 --before default_reset --after hard_reset write_flash -z --flash_mode dio --flash_freq 80m --flash_size 16MB 0x0 W:\nano-esp32\bricked\nano32blink16mb.bin` to write the flash on the bricked board.

Example:

```bash

W:\nano-esp32\bricked>esptool.exe  --port COM8 --before default_reset --after hard_reset write_flash -z --flash_mode dio --flash_freq 80m --flash_size 16MB 0x0 W:\nano-esp32\bricked\nano32blink16mb.bin
esptool.py v4.5.1
Serial port COM8
Connecting...
Detecting chip type... ESP32-S3
Chip is ESP32-S3 (revision v0.2)
Features: WiFi, BLE
Crystal is 40MHz
MAC: ec:da:3b:60:09:30
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
Flash will be erased from 0x00000000 to 0x00ffffff...
Compressed 16777216 bytes to 1212795...
Wrote 16777216 bytes (1212795 compressed) at 0x00000000 in 68.0 seconds (effective 1972.8 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...

```



## Verify Recovery

Now just use the Arduino IDE to configure the board!