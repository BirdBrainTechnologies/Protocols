# Protocols for the Hummingbird Bit and Finch 2.0


## Contents
 - [BLE Protocol for Stand-Alone micro:bit](#sharedBLE)
 - [BLE Protocol specific to Hummingbird Bit](#BitBLE)
 - [BLE Protocol specific to Finch 2.0](#FinchBLE)



## <a name="sharedBLE"></a>BLE Protocol for Stand-Alone micro:bit

##### Advertising Name:
MBXXXXX (Where XXXXX is the last 5 characters of the mac address. The first two letters change to BB for Hummingbird Bit, HB or HM for Hummingbird Duo, and FN for Finch 2.0)

##### UUIDs:
- Service UUID: 6E400001-B5A3-F393-E0A9-E50E24DCCA9E
- Write UUID (TX characteristic): 6E400002-B5A3-F393-E0A9-E50E24DCCA9E
- Notify UUID (RX characteristic): 6E400003-B5A3-F393-E0A9-E50E24DCCA9E

##### Connection interval:
20 -- 70 ms

##### Slave latency:
0\
Supervision Timeout -- 4 seconds

##### Baudrate:
115200

##### General Notes:
- The status LED will be attached to voltage. It will blink if voltage drops below 4.7V and be steady -- Above 4.7V.


##### LED Array Command:
This command sets the screen on the micro:bit. There are two different versions of the command: flash and symbol. In flash, you give a string of 8 bit unicode characters to flash across the screen. Symbol sets the individual LEDs in the array to a given state. This command ranges in length from 2 to 20 bytes.

Format:

0xCC | Symbol or flash command | ... | ...
--- | --- | --- | ---

The second byte in the command should be either the symbol, flash, or off command. The flash command includes the number of characters to flash.

- Symbol command: 0x80
- Flash command: 0x40 + length of string to flash
- Off command (stop flashing or clear the screen): 0x00

This byte can also be broken down to its individual bits as follows:

b7 | b6 | b5 | b4 | b3 | b2 | b1 | b0
--- | --- | --- | --- | --- | --- | --- | ---

- b7: Set to 1 for symbol
- b6: Set to 1 for flash
- b5: Nothing
- b4b3b2b1b0: Set to length of flash string for flash.

The bytes following the first two describe what will be displayed on the screen. For flash, this is simply the unicode representation of the characters to display in the order you want them displayed. For symbol, each bit in the next 4 bytes may represent an LED in the array. The symbol command format is as follows:

0xCC | 0x80 | LED25 | LED24-17 | LED16-9 | LED8-1
--- | --- | --- | --- | --- | ---

Notes:

- Since the maximum length of this command is 20 bytes and the first 2 bytes are used to define the command parameters, the maximum length of a string to flash is 18 bytes (18 characters).
- During flash, each character takes 300ms.
- Supported characters include: A-Z, 0-9, a-z, space, ! " # $ % & ' [ ] * + , - . / : ; < > ? @ \ ^ _ \` { } ~ |


Example - Flash "BBT":

0xCC | 0x43 | 0x42 | 0x42 | 0x54
--- | --- | --- | --- | ---

Example - Set LED array to show a smiley face symbol. First, convert the smiley face into a string of 1's and 0's. One representation of a smiley face is "0000001010000001000101110". Now, separate that into chunks. For example, the byte that represents LEDs 8 to 1 will be "01000000" in binary, which 0x40 in hex:

0xCC | 0x80 | 0x00 | 0xE8 | 0x81 | 0x40
--- | --- | --- | --- | --- | ---

Example - Stop flashing or clear the screen. You could turn the screen off simply by sending a symbol command in which all LEDs are off, but you can also use this special command:

0xCC | 0x00 | 0xFF | 0xFF | 0xFF
--- | --- | --- | --- | ---


##### micro:bit I/O Command:
For stand-along micro:bit ONLY, this command will send directly to the pads. Pad0 supports a buzzer.

Format:

0x90 | BNM | BNL | BDM | Mode | Pad0 | Pad1 | Pad2
--- | --- | --- | --- | --- | --- | --- | ---

Values:
- BNM, BNL: Buzzer note MSB and LSB
- BDM: Buzzer duration MSB
- Mode: Pad mode (see below)
- Pad0, Pad1, Pad2: Value to set for pads 0, 1, and 2. For Pad0, this becomes the buzzer duration LSB in buzzer mode.

Mode format (bits):

NU | NU | P0MM | P0ML | P1MM | P1ML | P2MM | P2ML
--- | --- | --- | --- | --- | --- | --- | ---

Values:
- NU: Not used
- P0MM: Pad0 Mode most significant bit
- P0ML: Pad0 Mode least significant bit
- P1MM, P1ML, P2MM, P2ML: Similar to above, but for Pads 1 and 2

Modes:
- 00: PWM (0 duty cycle default)
- 01: Input
- 10: (Pad0 only) Buzzer

Example - Put Pad0 to 50% intensity (PWM mode):

0x90 | 0x00 | 0x00 | 0x00 | 0x00 | 0x80 | 0x00 | 0x00
--- | --- | --- | --- | --- | --- | --- | ---

Now put Pad0 to 80% intensity:

0x90 | 0x00 | 0x00 | 0x00 | 0x00 | 0xD0 | 0x00 | 0x00
--- | --- | --- | --- | --- | --- | --- | ---

Now put Pad0 to buzzer mode with 280Hz for 1s:

0x90 | 0x0D | 0xF3 | 0x03 | 0x20 | 0xE8 | 0x00 | 0x00
--- | --- | --- | --- | --- | --- | --- | ---

Next change Pad1 to LED value with 50% intensity (Make sure you change the Pad0 frequency and time to zero):

0x90 | 0x00 | 0x00 | 0x00 | 0x20 | 0x00 | 0x80 | 0x00
--- | --- | --- | --- | --- | --- | --- | ---

Next make Pad2 as input:

0x90 | 0x00 | 0x00 | 0x00 | 0x21 | 0x00 | 0x80 | 0x00
--- | --- | --- | --- | --- | --- | --- | ---



##### Stop All Command:
This command will clear the LED array. If used with a Hummingbird Bit, it will also switch off the servos, LEDs, and buzzer. The Finch 2.0 has a similar command (but will not recognize this same command).

0xCB | 0xFF | 0xFF | 0xFF
--- | --- | --- | ---


##### Calibrate Magnetometer Command:
This command will start the magnetometer calibration. Results are received with the rest of the notifications.

0xCE | 0xFF | 0xFF | 0xFF
--- | --- | --- | ---

##### Send Firmware Version Command:
This command will cause a one time response containing the current firmware version information. SAMD Firmware Version is specific to the Hummingbird Bit board.

0xCF | 0xFF | 0xFF | 0xFF
--- | --- | --- | ---

Response:

Hardware Version | micro:bit Firmware Version | SAMD Firmware Version
--- | --- | ---


##### Notifications:
Sensor data can be received as periodic notifications.

Start Notifications Command:

0x62 | 0x67
--- | ---

Stop Notifications Command:

0x62 | 0x73
--- | ---

Format of Notifications received (14 bytes):

S1 | S2 | S3 | BL | AX | AY | AZ | BS | MXM | MXL | MYM | MYL | MZM | MZL
--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---

Values:
- S1, S2, S3: Sensors 1, 2, and 3.
- BL: Battery Level
- AX, AY, AZ: Accelerometer X, Y, and Z
- BS: Button/Shake
- MXM, MXL, MYM, MYL, MZM, MZL: Magnetometer X, Y, and Z, MSB and LSB.

The Button/Shake byte can be broken down into bits:

NU | NU | B | A | CCM | CCL | NU | Shake
--- | --- | --- | --- | --- | --- | --- | ---

Values:
- NU: Not used
- B, A: Buttons B and A (1 if the button is being pressed)
- CCM, CCL: Compass calibration check most and least significant bit
- Shake: 1 if the micro:bit is being shaken

Compass Calibration Check:
While the compass is calibrating, notifications will be paused completely. Notifications start again when calibration is complete and will now include the result. To ensure you do not read an old calibration result, one could look for the notifications to pause and resume, but it is usually sufficient to just wait a bit (maybe half a second) after you send the calibration start command before trying to read a result. Results are presented as follows:

CCM | CCL | Result
--- | --- | ---
0 | 0 | Unknown
0 | 1 | Calibration Success
1 | 0 | Calibration Failure

Notes:
- Accelerometer values are 8-bit values ranging from +/- 2g. They are 2’s complement. To convert the raw values to accelerometer values in meters per second squared, convert the raw byte to a signed 8-bit integer and multiply by 196/1280.
- Magnetometer values are 16-bit values ranging from +/- 30000μT, 2’s complement form. To convert raw values to μT, combine raw bytes and convert to a signed 16-bit integer. Then multiply by 1/10.

## <a name="BitBLE"></a>BLE Protocol specific to Hummingbird Bit

##### Advertising Name:
BBXXXXX

##### Set All Command:
For most applications, you will want to use this combined command to set the robot state. This allows for a reduction in the number of commands sent overall. Despite the name, this command does not set everything - the LED array on the micro:bit must be set separately.

Format (19 bytes total):

0xCA | LED1 | RS | R1 | G1 | B1 | R2 | G2 | B2 | SS1 | SS2 | SS3 | SS4 | LED2 | LED3 | BNM | BNL | BDM | BDL
--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---

Values:
 - LED1: Intensity of LED1.
 - RS: Reserved for future use (currently does nothing).
 - R1: Red intensity for Tri-LED1.
 - G1: Green intensity for Tri-LED1.
 - B1: Blue intensity for Tri-LED1.
 - R2: Red intensity for Tri-LED2.
 - G2: Green intensity for Tri-LED2.
 - B2: Blue intensity for Tri-LED2.
 - SS1: Servo1 value.
 - SS2: Servo2 value.
 - SS3: Servo3 value.
 - SS4: Servo4 value.
 - LED2: Intensity of LED2.
 - LED3: Intensity of LED3.
 - BNM: Buzzer note MSB.
 - BNL: Buzzer note LSB.
 - BDM: Buzzer duration MSB.
 - BDL: Buzzer duration LSB.


 Notes:
 - Intensities for all LEDs range from 0x00 to 0xFF (0 to 255).
 - Values set for servos represent the angle (for a position servo) or speed (for a rotation servo). They range from 0x00 to 0xFE. 0xFF is an off state.
 - Buzzer note values are in units of μs (microseconds). Notes given as frequencies must be converted to period. The conversion from frequency in Hz to period in μs is: `period = (1/frequency) * 1000000` Midi note numbers can be converted to frequencies using a simple formula (see https://newt.phys.unsw.edu.au/jw/notes.html): `frequency = 440 * pow(2, (note - 69)/12)`
 - Buzzer duration is in units of ms (milliseconds).
 - Except for the buzzer, the set all command components represent a state of the robot. Since the buzzer command is a thing of a certain duration, you want to make sure to only send that command once - at the time you want the buzzing to start. At all other times, the buzzer bytes should all be set to 0x00.


 Example - Set regular LEDs off, Tri-LED1 blue, Tri-LED2 green, Servo1 180 degrees, other servos off (no pulse width), buzzer generates a 440Hz square wave (midi 69, A4) for 30ms.  

 0xCA | 0x00 | 0xFF | 0x00 | 0x00 | 0xFF | 0x00 | 0xFF | 0x00 | 0xFE | 0xFF | 0xFF | 0xFF | 0x00 | 0x00 | 0x09 | 0xC4 | 0x00 | 0x1E
 --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---

##### LED Commands
Intensities range in value from 0x00 to 0xFF (0 to 255).

Format:

command | intensity | 0xFF | 0xFF
--- | --- | --- | ---

Commands:
- LED1: 0xC0
- LED2: 0xC1
- LED3: 0xC2

Example - Light LED2 with mid intensity:

0xC1 | 0x55 | 0xFF | 0xFF
--- | --- | --- | ---

##### Tri-LED Commands
Intensities range in value from 0x00 to 0xFF (0 to 255).

Format:

command | red intensity | blue intensity | green intensity
--- | --- | --- | ---

Commands:
- Tri-LED1: 0xC4
- Tri-LED2: 0xC5

Example - Set Tri-LED2 to BirdBrain teal:

0xC5 | 0x08 | 0x9B | 0xAB
--- | --- | --- | ---

##### Servo Commands
Value (to set angle or speed) ranges from 0x00 to 0xFE. 0xFF is an off state.

Format:

command | value | 0xFF | 0xFF
--- | --- | --- | ---

Commands:
- Servo1: 0xC6
- Servo2: 0xC7
- Servo3: 0xC8
- Servo4: 0xC9

Example - Set Servo3 to 180 degrees:

0xC8 | 0xFE | 0xFF | 0xFF
--- | --- | --- | ---

##### Buzzer Command
Have the buzzer generate a square wave of a given period for a given duration. See the Set All command for more information.

Format:

0xCD | Note in μs (MSB) | Note in μs (LSB) | Duration in ms (MSB) | Duration in ms (LSB)
--- | --- | --- | --- | ---

Example - Generate a 440Hz square wave (midi 69, A4) for 30ms:

0xCD | 0x09 | 0xC4 | 0x00 | 0x1E
--- | --- | --- | --- | ---

Example - Reset value (will not interrupt a buzz in progress):

0xCD | 0x00 | 0x00 | 0x00 | 0x00
--- | --- | --- | --- | ---

## <a name="FinchBLE"></a>BLE Protocol specific to Finch 2.0

Advertising Name:
FNXXXXX
