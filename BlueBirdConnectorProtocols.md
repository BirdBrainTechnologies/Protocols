# Protocols for the BlueBird Connector


## Contents
 - [Introduction](#intro)
 - [Outputs](#outputs)
 - [Inputs](#inputs)

## <a name="intro"></a>Introduction
BlueBird Connector is software designed to communicate with a Hummingbird Kit,
Finch Robot, or micro:bit over bluetooth. BlueBird accepts commands to send on
to the robot as localhost http requests. Typically, these requests come from
one of the libraries designed to work with BlueBird. Libraries are currently
available for snap!, python, and java.

Anyone wishing to develop their own library (or to send commands to BlueBird in
some other way) may do so using the protocol in this document. All requests
start with:

```http://127.0.0.1:30061/hummingbird/```


## <a name="outputs"></a>Outputs

#### For any robot type

 - out/stopall - Stop all outputs on all connected robots.
 - out/stopall/:robot - Stop all outputs on one connected robot.
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/out/stopall/A```
 - out/print/:printString/:robot - Print a string of text to the micro:bit LED array
   - :printString - Alphanumeric string to print.
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/out/print/Hello/B```
 - out/symbol/:robot/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b/:b - Set the micro:bit LED array to display a symbol.
   - :robot - Which robot (A, B, or C)
   - :b - Boolean value for each individual LED. 'true' for on, 'false' for off.
   - Example: ```http://127.0.0.1:30061/hummingbird/out/symbol/C/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true/true```

#### For finch and hummingbird

 - out/triled/:port/:R/:G/:B/:robot - Set the tri-color LED at the given port to specified color. For the finch, the beak is port 1, and tail LEDs are at ports 2-5.
   - :port - Location of LED to set (1-2 for hummingbird, 1-5 for finch)
   - :R, :G, :B - Red, green, blue values (0-255) to set respectively.
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/out/triled/1/8/155/171/B``` to set the beak of finch B to a nice teal.
 - out/playnote/:note/:duration/:robot - Play a given note on the buzzer for a given amount of time.
   - :note - Midi value of the note.
   - :duration - Duration of play in ms.
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/out/playnote/60/500/B```

#### For hummingbird only

 - out/led/:port/:intensity/:robot - Set the intensity of a single color LED.
   - :port - Location of LED to set (1-3).
   - :intensity - Intensity value (0-255).
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/out/led/1/255/C```
 - out/servo/:port/:value/:robot - Set the value of a position servo.
   - :port - Location of servo to set (1-4).
   - :value - Value to set (0-254 where 254 is about 180 degrees).
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/out/servo/1/254/A```
 - out/rotation/:port/:value/:robot - Set the value of a rotation servo.
   - :port - Location of servo to set (1-4).
   - :value - Value to set (0-255). 0 is the fastest motion backward. 254 is the
    fastest motion forward. 255 turns the servo off.
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/out/rotation/1/254/A```

#### For finch only

 - out/move/:robot/:dir/:dist/:speed - Move the finch a specified distance.
   - :robot - Which robot (A, B, or C)
   - :dir - Forward or Backward
   - :distance - Distance to move in cm.
   - :speed - Speed of travel as a percent of max (0-100).
   - Example: ```http://127.0.0.1:30061/hummingbird/out/move/A/Forward/10/50```
 - out/turn/:robot/:dir/:angle/:speed - Turn the finch a specified angle.
   - :robot - Which robot (A, B, or C)
   - :dir - Right or Left
   - :angle - Angle in degrees.
   - :speed - Speed of travel as a percent of max (0-100).
   - Example: ```http://127.0.0.1:30061/hummingbird/out/turn/A/Right/90/50```
 - out/wheels/:robot/:leftSpeed/:rightSpeed - Set the speeds of the individual
 finch wheels.
   - :robot - Which robot (A, B, or C)
   - :leftSpeed - Speed of left wheel as a percent of max (-100 to 100).
   - :rightSpeed - Speed of left wheel as a percent of max (-100 to 100).
   - Example: ```http://127.0.0.1:30061/hummingbird/out/wheels/B/50/-50```
 - out/stopFinch/:robot - Stop the wheels on the finch.
   - :robot - Which robot (A, B, or C)
 - out/resetEncoders/:robot - Reset the encoders on the finch to 0.
   - :robot - Which robot (A, B, or C)

## <a name="inputs"></a>Inputs

#### micro:bit sensor requests

 - in/button/:btn/:robot - Ask if the specified button is being pressed. Returns
 'true' if the button is being pressed, 'false' otherwise.
   - :btn - A, B, or Logo (available on version 2 micro:bits only).
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/button/Logo/C```
 - in/orientation/:n/:robot - Ask if the micro:bit is in the given orientation.
 Returns 'true' if it is, 'false' otherwise.
   - :n - Orientation to query. Options: 'Screen Up', 'Screen Down', 'Tilt Left',
  'Tilt Right', 'Logo Up', 'Logo Down', 'Shake' (is the micro:bit being shaken?).
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/orientation/Screen Up/B```
 - in/Accelerometer/:axis/:robot - Request the accelerometer value along the
 given axis.
   - :axis - X, Y, or Z
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/Accelerometer/X/C```
 - in/Magnetometer/:axis/:robot - Request the magnetometer value along the given
 axis.
   - :axis - X, Y, or Z
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/Magnetometer/X/C```
 - in/Compass/:robot - Request the compass value.
   - :robot - Which robot (A, B, or C)
 - in/V2sensor/:sensor/:robot - Request the value of the micro:bit sound or
 temperature sensor (available on version 2 micro:bits only).
   - :sensor - Sound or Temperature. Temperatures are reported in °C.
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/V2sensor/Sound/A```

#### Hummingbird sensor port requests

 - in/:sensor/:port/:robot - Request value of given sensor at given port.
   - :sensor - Distance, Dial, Light, Sound, Other, or sensor.
   - :port - Which port (1, 2, or 3)
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/Distance/2/A```

#### Finch only sensors

 - in/Light/:port/:robot - Request finch light sensor value.
   - :port - Right or Left
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/Light/Right/B```
 - in/Line/:port/:robot - Request finch line sensor value.
   - :port - Right or Left
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/Line/Right/B```
 - in/Encoder/:port/:robot - Request finch encoder value.
   - :port - Right or Left
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/Encoder/Right/B```
 - in/finchIsMoving/static/:robot - Ask whether the finch is still completing a
 move or turn command.
   - :robot - Which robot (A, B, or C)
 - in/finchOrientation/:n/:robot - Ask if the finch is in the given orientation.
 Returns 'true' if it is, 'false' otherwise.
   - :n - Orientation to query. Options: 'Beak Up', 'Beak Down', 'Tilt Left', 'Tilt Right', 'Level', 'Upside Down', 'Shake' (is the micro:bit being shaken?).
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/finchOrientation/Screen Up/B```
 - in/finchAccel/:axis/:robot - Request the accelerometer value along the
 given axis. Result given in the finch reference frame.
   - :axis - X, Y, or Z
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/finchAccel/X/C```
 - in/finchMag/:axis/:robot - Request the magnetometer value along the given
 axis. Result given in the finch reference frame.
   - :axis - X, Y, or Z
   - :robot - Which robot (A, B, or C)
   - Example: ```http://127.0.0.1:30061/hummingbird/in/finchMag/X/C```
 - in/finchCompass/static/:robot - Request the compass value. Result given in
 the finch reference frame.
   - :robot - Which robot (A, B, or C)

#### Other requests

 - in/isHummingbird/static/:robot - Returns true if the robot is a hummingbird.
   - :robot - Which robot (A, B, or C)
 - in/isMicrobit/static/:robot - Returns true if the robot is a stand alone
 micro:bit.
   - :robot - Which robot (A, B, or C)
 - in/isFinch/static/:robot - Returns true if the robot is a finch.
   - :robot - Which robot (A, B, or C)
