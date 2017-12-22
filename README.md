# adafruit-feather-32u4-lora-ota

This is an adaption of the [ttn-otaa example by Thomas Telkamp and Matthijs Kooijman](https://github.com/matthijskooijman/arduino-lmic/tree/master/examples) 
for [Adafruit Feather 32u4 with RFM95 LoRa Radio module](https://learn.adafruit.com/adafruit-feather-32u4-radio-with-lora-radio-module/#).

This was tested with Arduino IDE version 1.8.5.

![Wiring pin IO1 with pin 6](https://github.com/wklenk/adafruit-feather-32u4-lora-ota/blob/master/media/wire_bridge.jpg)

## Preliminary work
* Following the [instructions at Adafruit](https://learn.adafruit.com/adafruit-feather-32u4-radio-with-lora-radio-module/setup), 
you have setup the Arduino IDE to support this board.
* You have installed the Arduino-LMIC library. The easiest way to accomplish 
this is to use the "Library Manager" of the Arduino IDE and search for "lmic".
In the moment of writing, the version of this library is 1.5.0+arduino-2.
There are other ways to install this library. Please follow the instructions 
in the respective github repository at https://github.com/matthijskooijman/arduino-lmic
* You need to additionally wire up the board's IO1 pin with the board's pin 6.
You can use a simple wire bridge for that. Check the picture above to make sure you are actually
using the right pins.
* With approximately 32KB of memory available, we have to strip down the LMIC library a little bit.
Your task is to find out where the LMIC library is located on your hard drive. 
Typically this is in some folder named _libraries_. Find the file _config.h_
Modify the following sections:
  
```c
// Uncomment this to disable all code related to ping
#define DISABLE_PING
// Uncomment this to disable all code related to beacon tracking.
// Requires ping to be disabled too
#define DISABLE_BEACONS
```

## Configure for The Things Network
Register a new device in [The Things Network Console](https://console.thethingsnetwork.org/)
You will get a
* Device EUI (let TTN create one)
* App Key
* APP EUI

You now have to add these values in the source code:

```c
static const u1_t PROGMEM APPEUI[8]= { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
void os_getArtEui (u1_t* buf) { memcpy_P(buf, APPEUI, 8);}

// This should also be in little endian format, see above.
static const u1_t PROGMEM DEVEUI[8]= { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
void os_getDevEui (u1_t* buf) { memcpy_P(buf, DEVEUI, 8);}

// This key should be in big endian format (or, since it is not really a
// number but a block of memory, endianness does not really apply). In
// practice, a key taken from ttnctl can be copied as-is.
// The key shown here is the semtech default key.
static const u1_t PROGMEM APPKEY[16] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
```
Please note that APP EUI and Device EUI have to be specified in "little endian format". There are
buttons in the TTN console that help you to convert the number sequences into LSB format.

![LSB output in TTN console](https://github.com/wklenk/adafruit-feather-32u4-lora-ota/blob/master/media/ttn_console_lsb.png)

## What does it do?

* Tries to join "The Things Network" randomly using one of the three join frequencies, 
starting with spreading factorSF7.
* During the join operation, the built-in LED blinks once a second. 
After a successful join, the blink rate changes from 1 second to 5 seconds.
* Every 60 seconds sends a message with one byte (0x00).

**Note**: The use of a so-called [Single-Channel-Gateway](https://www.thethingsnetwork.org/wiki/Hardware/Gateways/Single-Channel-Gateway)
is not recommended, as it only supports **one** of the three join frequencies. 
So, you only have 1:3 chance that the join operation actually hits the single one supported frequency. 
Additionally, it is unclear if these kind of gateways support the communication from TTN back to the device,
so probably you will never ever get a feedback that the join operation was successful.

