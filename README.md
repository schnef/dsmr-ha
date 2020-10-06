# HomeAssistant DSMR P1 TCP/IP bridge (ESP8266) #

## Purpose ##

Using this sketch, one can integrate a DSMR with HomeAssistant. The
DSMR's P1 port is connected using the ESP8266's UART which is
configured such that the RX pin is inverted. No external inverter is
required or special cable is needed. The ESP8266 runs a TCP server to
which HomeAssistant connects to fetch the data, using the [DSMR
platform](https://www.home-assistant.io/integrations/dsmr/).

Tested with a Sagemcom T210-D and Adafruit HUZZAH module. This setup
turned out to be very reliable.

## Implementation ##

Using a ESP8266, read DSMR telegrams from the P1 port and publish them
over TCP/IP. In contrast to most other implementations which read the
P1 port, no hardware inverter is required. By properly configuring the
UART's receive pin, the input is inverted and can be used directly.

The implementation was tested using a Adafruit HUZZAH ESP8266 board,
powered using an old mobile phone charger, connected to a Sagemcom
T210-D metering device with the UART RX pin connected to the P1's
Data (pin #5), GND to Data GND (pin #3) and GPIO 04 connected to Data
Request (pin #2). One can simply use an old fully wired (all four
connections) RJ11 telephone wire for this. GPIO pin 4 is used to
enable/disable sending by the P1 port by taking the Data Request line
HIGH/LOW.  The Sagemcom T210-D uses protocol DSMR P1 version 5.0. With
a properly wired RJ12 connector, one probably can even power the board
from the P1 port as well and omit the external power supply.

The sketch implements two equivalant CRC checking functions, one using
a lookup table and another one which processes the message
bit-by-bit. The one using the lookup table might be faster at the cost
of more memory being used, but this was not tested. CRC checking is
redundant because the [DSMR
parser](https://github.com/ndokter/dsmr_parser) used by the DSMR
platform does CRC checking also, but it is included since I think this
is the proper place for CRC checking and it is such fun to implement.

If you DSMR runs an older P1 protocol version without CRCs, you have
to review the sketch and make the appropriate changes.

Over-the-air (OTA) programming is also implemented.

## Configuration ##

I use the file `credentials.h` for the user names and passwords used
in the script and exclude this file from git to keep stuff save. You
can do the same or just edit the sketch and add the credentials:

 * `STASSID` is your wifi's SSID
 * `STAPSK` the pre-shared wifi key (wifi password)

### Home Assistant ###

Assuming you know the IP address `<IP-ADDRESS>` the ESP8266 uses, add
the following lines to the `configuration.yaml` file and restart HA.

```
sensor:
    - platform: dsmr
      host: <IP-ADDRESS>
      port: 2001
      dsmr_version: 5
```

If you want to test outside HA, just run `telnet <IP-ADDRESS> 2001`
and you should see a telegram every second. NB: the ESP8266 can only
handle one TCP connection, so this works only if HA isn't connected.

## LEDs ##

 * On booting and connecting to wifi, the HUZZAH's blue led will be on
   continiously.
 * The blue led flashes shortly with every telegram received from the
   P1 port.
 * The red led will be turned on when a TCP client,
   i.e. HomeAssistant, is connected.

## Links ##

 * [Adafruit HUZZAH ESP8266](https://learn.adafruit.com/adafruit-huzzah-esp8266-breakout/overview)
 * [Arduino core for ESP8266 WiFi chip](https://github.com/esp8266/Arduino#arduino-core-for-esp8266-wifi-chip)
 * [ESP8266 Arduino Core’s documentation](https://arduino-esp8266.readthedocs.io/en/latest/index.html)
 * [Slimme meter on wikipedia](https://nl.wikipedia.org/wiki/Slimme_meter)
 * [P1 Companion Standard](https://www.netbeheernederland.nl/_upload/Files/Slimme_meter_15_a727fce1f1.pdf)
