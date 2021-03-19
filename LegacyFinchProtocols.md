# Protocols for the original Finch Robot


## Contents
 - [Introduction](#intro)
 - [USB Protocol](#usb)

## <a name="intro"></a>Introduction
This document covers communication protocols for the original Finch Robot. If you are looking for protocols for the Finch 2.0 (powered by micro:bit), please read [MicroBitProtocols](MicroBitProtocols.md).

## <a name="usb"></a>Finch USB Protocol
The Finch communicates with the host computer using the [USB HID protocol](https://en.wikipedia.org/wiki/USB_human_interface_device_class).  The following details how the Finch communicates over this protocol and can be used to build support for the Finch in different programming languages.

#### General
The Finch is an HID device with a VID of 2354 (Hex) and a PID of 1111 (Hex).  

Reports are 8 bytes long.  For reports received by the Finch, the first byte determines the command for the Finch to execute, and other bytes are optional and command dependent (explained below).  For reports sent to the host, all bytes are either 0 or optional and command dependent.

The Finch has a time-out where it will go back to passive color cycling mode if no commands are sent in a five second time frame.

#### Commands
The first byte determines the command for the Finch to execute.  Our command set uses simple ASCII characters as follows:
 - 'O' - control full-color LEDs (Orbs)
 - 'M' - control the velocity of the motors
 - 'B' - sets the buzzer to chirp for some period of time
 - 'T' - gets the temperature
 - 'L' - gets the values of the two light sensors
 - 'A' - gets the accelerometer values of the X, Y, and Z axes, and the tap/shake byte.  
 - 'I' - gets the values of the two obstacle sensors
 - 'X' - set all motors and LEDs to off
 - 'R' - turns off the motor and has the Finch go back into color-cycling mode (this is how we indicate that the Finch connection is closed).
 - 'z' - Just sends a counter value showing the number of times z has been sent (for basic connectivity testing).

Based on the top-level command, the report requires 0-6 additional command bytes, and may generate a return report of 0-5 bytes:
 - 'O' is followed by the intensity of the Red, Green, and Blue elements (three bytes, each from 0-255)
 - 'M' is followed by four bytes.  Byte 1 controls left wheel direction (0 for forward, 1 for back), Byte 2 is left wheel speed (0-255), byte 3 is right wheel direction, byte 4 is right wheel speed.  Direction values are decimal, not ascii.
 - 'B' sets the buzzer to chirp for an amount of milliseconds (2 bytes, high byte first) at a frequency (2 bytes, high byte first).  Note that the buzzer does not queue, it will immediately chirp something else as soon it receives a second chirp command, regardless of whether the first chirp has completed.
 - 'T' generates a report back to the computer with one byte - current temp.  To convert to Celcius, use the following equation:  Celcius = (value-127)/2.4 + 25.
 - 'L' returns the left and right light sensor values - one byte each
 - 'A' returns a single byte with decimal value 153, followed by the X, Y, and Z acceleration values, one byte each, followed by a byte encoding if the Finch was tapped or shaken.  The accelerometer has a range from 0x00 to 0x3F (6 bit).  0x00 to 0x1F map to +0g to +1.453g.  0x3F to 0x20 map from -0.047g to -1.5g.  To convert to g-force, take the two's complement if the value is greater than 0x1F.  Then apply the following equation:  g-force = value * 1.5/32. Finally, the tap/shake byte encodes whether the Finch has been tapped or shaken since last read.  Encoding is as follows:
   - If bit 7 (highest order bit) of the byte is a 1, then the Finch has been shaken since the last read.
   - If bit 5 is a 0, then the Finch has been tapped since the last read
 - 'I' returns left and right obstacle sensors - decimal 1 for something seen, 0 for not
 - 'R' and 'X' do not require additional commands and do not send additional bytes
