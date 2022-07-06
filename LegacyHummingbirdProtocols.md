# Protocols for the Hummingbird Duo


## Contents
 - [Introduction](#intro)
 - [BLE Protocol](#ble)
 - [USB Protocol](#usb)

## <a name="intro"></a>Introduction
This document covers communication protocols for the original Hummingbird and the Hummingbird Duo. If you are looking for protocols for the Hummingbird Bit (powered by micro:bit), please read [MicroBitProtocols](MicroBitProtocols.md). The same USB protocol is followed by both the original Hummingbird and the Duo. The original Hummingbird had no bluetooth protocol.

Switching the Duo between BLE and USB requires changing the firmware on the board via the [Hummingbird Firmware Burner].

## <a name="ble"></a>Hummingbird Duo BLE Protocol



## <a name="usb"></a>Hummingbird USB Protocol

When in USB tethered mode, the Hummingbird Duo controller communicates with the host computer using the [USB HID protocol](https://en.wikipedia.org/wiki/USB_human_interface_device_class). The following details how the Hummingbird communicates over this protocol and can be used to build support for the Hummingbird in different programming languages.

#### General Information
The Hummingbird is an HID device with a VID of 0x2354 (Hex) and a PID of 0x2222 (Hex).

Reports are 8 bytes long.  For reports received by the Hummingbird, the first byte determines the command for the Hummingbird to execute, and other bytes are optional and command dependent (explained below).  For reports sent to the host, all bytes are either 0 or optional and command dependent.

When the first command is sent, the status LED will change from fading on and off to staying fully on. The Hummingbird has a time-out where it will go back to fading the status LED if no commands are sent in a five second time frame.

#### Commands
The first byte determines the command for the Hummingbird to execute.  Our command set uses simple ASCII characters as follows:

 - ‘O’ – control the full-color LEDs
 - ‘L’ – control the single color LEDs
 - ‘M’ – control the velocity of the motors
 - ‘V’ – controls the vibration motors
 - ‘S’ – controls the servos
 - ‘s’ – returns the value of a specific sensor port
 - ‘G’ – gets the state of the Hummingbird’s LEDs, motors, servos, and sensors
 - ‘X’ – set all motors and LEDs to off
 - ‘R’ – turns off the motors/LEDs and immediately sets the status LED to go back to fading (‘R’ should be sent right before a connection to Hummingbird is closed).
 - ‘z’ – Just sends a counter value showing the number of times z has been sent (for basic connectivity testing).

Based on the top-level command, the report requires 0-4 additional command bytes, and may generate a return report of 0-7 bytes.

#### Commands not generating a return report:
 - ‘O’ is followed by four bytes. The first byte sets the port of the tri-color LED  (ASCII ‘0’ for LED 1, ASCII ‘1’ for LED 2), bytes 2-4 control the intensity of the Red, Green, and Blue elements (0 to 255).
 - ‘L’ is followed by two bytes. The first byte sets the port of the LED (ASCII ‘0’ to ‘3’ for LED ports 1 to 4), the second byte sets the intensity of the LED (0 to 255).
 - ‘M’ is followed by three bytes.  Byte 1 sets the motor port (ASCII ‘0’ for motor 1, ASCII ‘1’ for motor 2), Byte 2 controls the direction (ASCII ‘0’ for forward, ‘1’ for back), Byte 3 is speed (0 to 255).
 - ‘V’ is followed by two bytes.  The first byte sets the port of the vibration motor (ASCII ‘0’ for motor 1, ASCII ‘1’ for motor 2), the second byte sets the intensity of vibration (0 to 255).
 - ‘S’ is followed by two bytes. The first byte sets the port of the servo (ASCII ‘0’ to ‘3’ for servo ports 1 to 4), the second byte sets the angle of the servo; 0 corresponds to 0 degrees, and 225 corresponds to 180 degrees. Values from 226 to 254 are set to 225 (180 degrees) in firmware. Sending a value of 255 turns off power to the servo.
 - ‘R’ and ‘X’ do not require additional bytes

#### Commands generating a return report:
 - ‘s’ is followed by one byte specifying the sensor port (ASCII ‘0’ to ‘3’ for sensor ports 1 to 4, ASCII ‘4’ for the motor power supply voltage). This command generates a return report where the first byte is the value of the sensor at that port. The sensor value corresponds directly to the voltage sensed at the port: For the sensor ports 0V corresponds to a value of 0, and 5V corresponds to 255, for the motor power supply 0 corresponds to 0V and 255 corresponds to 10V (or more). Typically if the motor power supply is unplugged, the voltage will read between 0 and 1 Volt.. If you wish to receive all sensor values in one report to minimize communication time, use the get state 3 (‘G’ ‘3’) defined next.
 - ‘G’ stands for get state and can be used to return the current state of all ports on the Hummingbird. It is followed by one byte specifying which port data should be return. This byte value is ASCII ‘0’ to ‘3’. Data returned is as follows:
   - ‘0’ returns the port setting of  tri color LED 1 (R G B), tri color LED 2 (R G B), and single color LED 1.
   - ‘1’ returns the setting of single color LEDs 2 to 4, followed by the settings of servo ports 1 to 4.
   - ‘2’ returns the motor and vibration motor information. Byte 0 is motor 1 direction , byte 1 is motor 1 speed, byte 2 is motor 2 direction, byte 3 is motor 2 speed, byte 4 is vibration motor 1 intensity, and byte 5 is vibration motor 2 intensity.
   - ‘3’ returns the current value of sensors ports as well as whether or not motor power is detected. Bytes 0 to 3 correspond to the values on sensor ports 1 to 4 respectively. Byte 4 corresponds to the voltage at the motor power port, with 0 corresponding to 0V and 255 corresponding to 10V (or more). Typically if the motor power supply is unplugged, this will read between 0 and 1 Volt.
   - ‘4’ returns the hardware and firmware version. The Duo hardware version is 0x03 0x00, firmware version as of 12/22/2014 is 0x02 0x01 0x68 (‘h’).
 - ‘z’ Just sends a counter value showing the number of times z has been sent (for basic connectivity testing).

All commands generating a return report contain a mechanism for matching returned reports to the requesting reports. The last byte of the return report will match the last byte of the outgoing report. For example, if you send the following 8-byte report:

's' | '0' | 0 | 0 | 0 | 0 | 0 | 0x42
--- | --- | - | - | - | - | - | ---

you would receive:

n | 0 | 0 | 0 | 0 | 0 | 0 | 0x42
-- | - | - | - | - | - | - | ---

where n is the value of the sensor on port 1. The last byte can be used as a signature to match the reports in case received reports are received asynchronously.
