Nova protocol documentation
=============

https://wantnova.com/

https://wantnova.com/sdk/

#### Protocol notes

Nova's protocol is built on top of Bluetooth Low Energy. The rest of this section assumes you know the basics of BLE.

*   Discover using Service UUID `FFF0` and peripheral name: `Nova`
*   Two characteristics: one for making a request to Nova `FFF3` and one for reading the response `FFF4`
*   It's a request/response protocol. Phone will always make the request and Nova will reply with a single response.
*   Each request consists of an ID (0-255), a command type, and for some commmands a set of parameters.
*   Each response is simply an ack. If you phone doesn't receive an ack within 2 seconds assume the connection is dead.
*   The request ID is echoed in the ack so you can match them up. Nova doesn't do anything else with the ID. My library just counts 0-255 then wrap.
* Message is ASCII string. Numbers encoded as hex (2 digits for uint8, 4 for uint16) with leading 0 padding if necessary.
* Message is framed like this `(ID:BODY)` where ID is a 2 digit hex string and and body is one of the 3 commands in the next section.

#### Command reference

##### L: Light up

Activates the light.

Takes 3 parameters:
1.  Warm 00-FF: Brightness of warm LEDs
2.  Cool 00-FF: Brightness of cool LEDs
3.  Timeout 0000-FFFF: Light will automatically turn of after this many milliseconds.

Ideal values explained in next section.

Example request: `(06:L,00,FF,01F4)` -> means command ID 6, Light up, warm=0, cool=255, timeout=1000ms

Response: `(06:A)` -> command ID 6 acked

##### O: Off

Turn lights off.

No parameters.

Example request: `(07:O)` -> means command ID 7, Off

Response: `(07:A)`

##### P: Ping

No-op. You can use this for testing to see if Nova ACKs it.

No parameters.

Example: `(08:P)` -> means command ID 8, Ping

Response: `(08:A)`


#### Flow of execution in a camera app

1. User presses shutter button
2. Send light "L" command to Nova
3. Wait for ack "A" response from Nova
4. Capture photo.
5. Wait for photo capture to complete
6. Send off "O" command to Nova
7. Wait for ack "A" response from Nova

A more advanced camera application may choose to do a pre-flash to help the camera focus.

#### Ideal brightness settings

*   __Gentle__: warm=7F, cool=00, timeout=05DC
*   __Warm__: warm=FF, cool=00, timeout=05DC
*   __Bright__: warm=FF, cool=FF, timeout=05DC

Warning: Do not use PWM settings in range of 0x01 to 0x40 for longer than 1 second as this can put excessive load on the circuitry. 0x00 is fine, and anything over 0x40 is fine. (Yes, a dim brightness can create higher loads)

