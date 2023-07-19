# clicky-turtle-xiao

This repository contains a firmware implementation of a composite USB device which exposes the following functionality:

- USB to UART conversion (fixed at a baudrate of 9600)
- USB to keyboard HID interface via SPI

It is designed to be used with the [seeeduino xiao](http://wiki.seeedstudio.com/Seeeduino-XIAO/) (ATSAMD21G18A). When flashed with the correct firmware, this will be hereafter referred to as a *turtleboard*.

**Important notes:**

- The seeeduino xiao supports 0V to 3.3V digital signals. Connecting signals outside this range will likely cause permanent damage to the device.
- For simplicity, the USB to UART baudrate is fixed at 115200. The device will ignore requests from the PC to change this.

## Flashing the clicky-turtle-xiao firmware

1. Download the firmware file `clicky-turtle-xiao.uf2` from the [releases](https://github.com/jeremyherbert/clicky-turtle-xiao/releases) section of this repository.
2. Connect your seeeduino xiao to your computer with a USB C cable.
3. Force your device into bootloader mode. To do this, you need to take a wire and connect the RST and GND pads together **twice** in quick succession. These two pads are located next to the USB connector on the board. See the animation at the end of this list for an example.
4. A new USB storage device will appear connected to your computer (likely called "ARDUINO" or something similar)
5. Copy and paste the `musical-turtle.uf2` file onto the new storage device
6. The device will reflash its own firmware and then disconnect the storage device. You're now ready to go! 
7. (Your device may need a power cycle to start correctly the first time; just unplug and replug the USB cable.)

![bootloader-animation](https://files.seeedstudio.com/wiki/Seeeduino-XIAO/img/XIAO-reset.gif)

## Usage

### USB to UART
To use the USB to UART converter, simply connect the device to a free USB port on your computer. Drivers should be automatically installed on all platforms that need them. The device will appear in your operating system as a serial port.

### USB Keyboard (HID)

The firmware is designed to receive 8 byte [standard HID reports](https://usb.org/sites/default/files/hut1_3_0.pdf) 
and forward them to the PC. The SPI transaction to send a report is exactly 8 bytes long. A HID report contains information about which modifier keys are pressed (Shift, Alt, etc) and which keys are pressed 
(also referred to as scan codes). A list of standard modifier keys and scan codes can be found [here](https://gist.github.com/MightyPork/6da26e382a7ad91b5496ee55fdc73db2).

The first byte of the report contains all of the pressed modifier keys bitwise OR-ed together. The second byte is always 
`0x00` and the remaining bytes are scan codes corresponding to keys that have been pressed. An example HID report is 
shown below; this report will indicate that `Left CTRL`, `Left Shift` and `B` are all being held down:

| Byte number | Value | Description                                   |
|-------------|-------|-----------------------------------------------|
| 0           | 0x03  | Modifier masks (`Left CTRL` and `Left Shift`) |
| 1           | 0x00  | Reserved, always `0x00`                       |
| 2           | 0x05  | Pressed key 1 (`B`)                           |
| 3           | 0x00  | Pressed key 2 (None)                          |
| 4           | 0x00  | Pressed key 3 (None)                          |
| 5           | 0x00  | Pressed key 4 (None)                          |
| 6           | 0x00  | Pressed key 5 (None)                          |
| 7           | 0x00  | Pressed key 6 (None)                          |

Note from this structure that only 6 keys can be sent at once; this is a limitation of the HID standard. All other keys 
not included in the report are assumed to be unpressed. To indicate that a key has been released, send a report that 
does not contain the scan code for that key. It is possible to send an empty report (ie `00 00 00 00 00 00 00 00`) to 
indicate that all keys have been released.

Due to the way that USB works, the device cannot immediately send the HID data to the PC. The device is configured to send updates every 2ms (500Hz), but this may be much longer depending on the USB host controller and operating system that the device is connected to. If data is sent too fast, messages may be skipped or corrupted.

In order to send messages successfully, you should communicate with the turtleboard in the following manner:

1. Wait until the `IDLE` pin is high
2. Set CS low
3. Send 8 bytes of SPI data (ie the HID report)
4. Wait at least 10us
5. Set CS high

Repeat this process each time you wish to send a message to the device.

## LEDs

There are four LEDs next to the USB connector. 

- The green LED indicates that the board is powered on. 
- The yellow LED blinks when SPI data has been received. 
- The two blue LEDs correspond to UART TX and RX activity.

## Connection information

WARNING: Do not use the pinout information provided by seeedstudio. 

The blank pins on the connection layout must be left floating.

![musical-turtle-xiao pinout](https://files.jeremyherbert.net/clicky-turtle-xiao.png)