EMS-ESP-LINK
============

EMS Gateway with ESP8266/ESP-12 module

Wireless "Buderus EMS-Bus" interface for "collectord".

For Eagle schematics see [EMS-ESP8266_12-PCB](https://github.com/susisstrolch/EMS-ESP8266_12-PCB)

code parts based/derived from

- Espressif - [ESP8266 SDK](http://bbs.espressif.com/viewforum.php?f=46&sid=0e523ee5f13f8134fe41122d3e2c8c3d)
- pfalcon - [esp-open-sdk](https://github.com/pfalcon/esp-open-sdk)
- Thorsten von Eicken - [esp8266 wifi-serial bridge and arduino/AVR/LPC/NXP programmer](https://github.com/jeelabs/esp-link)
- Danny Baumann - [ethersex](https://github.com/maniac103/ethersex)

Additional links

- [Logamatic 2107 Schnittstelle](http://www.mikrocontroller.net/topic/141831)
- [Buderus EMS Gateway mit PIC18F](http://www.mikrocontroller.net/topic/210031)
- [Faktensammlung Buderus EMS](http://www.mikrocontroller.net/topic/309075)
- [EMS > Adapter > NetIO > Raspi](http://www.mikrocontroller.net/topic/318364)
- [Ingo F's EMS-Gateway WIKI](http://ems-gateway.myds.me/dokuwiki/doku.php)
- [ESP8266 Community Forum](http://www.esp8266.com Espressif Board Forum - http://bbs.espressif.com)


PCB

- [Dirt Cheap Dirty Boards](http://dirtypcbs.com/)


=== ORIGINAL README for esp-link from Thorsten ===

ESP-LINK
========

This firmware connects an attached EMS Buderus to the internet using a ESP8266 Wifi module.

The firmware includes a tiny HTTP server based on
[esphttpd](http://www.esp8266.com/viewforum.php?f=34)
with a simple web interface, many thanks to Jeroen Domburg for making it available!

Eye Candy
---------
These screen shots show the Home page, the (Wifi) configuration page, the EMS Debug Log page and the EMS-Link debug page. 

<img width="45%" src="https://github.com/susisstrolch/ems-esp-link/blob/develop/Selection_050.png">
<img width="45%" src="https://github.com/susisstrolch/ems-esp-link/blob/develop/Selection_051.png">
<img width="45%" src="https://github.com/susisstrolch/ems-esp-link/blob/develop/Selection_049.png">
<img width="30%" src="https://github.com/susisstrolch/ems-esp-link/blob/develop/Selection_052.png">

[The Homepage / EMS Status tab is currently a fake, EMS Debug Logs are real.]

Hardware info
-------------

This firmware is designed for esp8266 ESP07/ESP12 modules which have most ESP I/O pins available and at least 512KB flash.

The default connections are:
- URXD: connect to TX of EMS Gateway
- UTXD: connect to RX of EMS Gateway

Initial flashing
----------------
(This is not necessary if you receive one of the EMS-ESP8266_12 modules from the author!)
If you want to simply flash the provided firmware binary, you can download the latest
[release](https://github.com/susisstrolch/ems-esp-link/releases) and use your favorite
ESP8266 flashing tool to flash the bootloader, the firmware, and blank settings.
Detailed instructions are provided in the release notes.

Note that the firmware assumes a 512KB flash chip, which most of the esp-01 thru esp-11
modules appear to have. A larger flash chip should work but has not been tested.

Wifi configuration overview
------------------
For proper operation the end state the ems-esp-link needs to arrive at is to have it
join your pre-existing wifi network as a pure station.
However, in order to get there the ems-esp-link will start out as an access point and you'll have
to join its network to configure it. The short version is:
 1. the ems-esp-link creates a wifi access point with an SSID of the form `ESP_012ABC`
 2. you join your laptop or phone to the ems-esp-link's network as a station and you configure
    the ems-esp-link wifi with your network info by pointing your browser at http://192.168.4.1/
 3. the ems-esp-link starts to connect to your network while continuing to also be an access point
    ("AP+STA"), the ems-esp-link may show up with a `ems-esp-link.local` hostname
    (depends on your DHCP/DNS config)
 4. the ems-esp-link succeeds in connecting and shuts down its own access point after 15 seconds,
    you reconnect your laptop/phone to your normal network and access ems-esp-link via its hostname
    or IP address

LED indicators
--------------
Assuming appropriate hardware attached to GPIO pins, the green "conn" LED will show the wifi
status as follows:
- Very short flash once a second: not connected to a network and running as AP+STA, i.e.
  trying to connect to the configured network
- Very short flash once every two seconds: not connected to a network and running as AP-only
- Even on/off at 1HZ: connected to the configured network but no IP address (waiting on DHCP)
- Steady on with very short off every 3 seconds: connected to the configured network with an
  IP address (ems-esp-link shuts down its AP after 15 seconds)

The yellow "ser" LED will blink briefly every time serial data is sent or received by the ems-esp-link.

Wifi configuration details
--------------------------
After you have serially flashed the module it will create a wifi access point (AP) with an
SSID of the form `ESP_012ABC` where 012ABC is a piece of the module's MAC address.
Using a laptop, phone, or tablet connect to this SSID and then open a browser pointed at
http://192.168.4.1/, you should then see the ems-esp-link web site.

Now configure the wifi. The desired configuration is for the ems-esp-link to be a
station on your local wifi network so you can communicate with it from all your computers.

To make this happen, navigate to the wifi page and you should see the ems-esp-link scan
for available networks. You should then see a list of detected networks on the web page and you
can select yours.
Enter a password if your network is secure (highly recommended...) and hit the connect button.

You should now see that the ems-esp-link has connected to your network and it should show you
its IP address. _Write it down_. You will then have to switch your laptop, phone, or tablet
back to your network and then you can connect to the ems-esp-link's IP address or, depending on your
network's DHCP/DNS config you may be able to go to http://ems-esp-link.local

At this point the ems-esp-link will have switched to STA mode and be just a station on your
wifi network. These settings are stored in flash and thereby remembered through resets and
power cycles. They are also remembered when you flash new firmware. Only flashing `blank.bin`
via the serial port as indicated above will reset the wifi settings.

There is a fail-safe, which is that after a reset or a configuration change, if the ems-esp-link
cannot connect to your network it will revert back to AP+STA mode after 15 seconds and thus
both present its `ESP_012ABC`-style network and continue trying to reconnect to the requested network.
You can then connect to the ems-esp-link's AP and reconfigure the station part.

One open issue (#28) is that ems-esp-link cannot always display the IP address it is getting to the browser
used to configure the ssid/password info. The problem is that the initial STA+AP mode may use
channel 1 and you configure it to connect to an AP on channel 6. This requires the ESP8266's AP
to also switch to channel 6 disconnecting you in the meantime.

Troubleshooting
---------------
- verify that you have sufficient power, borderline power can cause the esp module to seemingly
  function until it tries to transmit and the power rail collapses
- check the "conn" LED to see which mode ems-esp-link is in (see LED info above)
- reset or power-cycle the ems-esp-link to force it to become an access-point if it can't
  connect to your network within 15-20 seconds
- if the LED says that ems-esp-link is on your network but you can't get to it, make sure your
  laptop is on the same network (and no longer on the esp's network)
- if you do not know the ems-esp-link's IP address on your network, try `ems-esp-link.local`, try to find
  the lease in your DHCP server; if all fails, you may have to turn off your access point (or walk
  far enough away) and reset/power-cycle ems-esp-link, it will then fail to connect and start its
  own AP after 15-20 seconds

Building the firmware
---------------------
The firmware has been built using the [esp-open-sdk](https://github.com/pfalcon/esp-open-sdk)
on a Linux system. Create an esp8266 directory, install the esp-open-sdk into a sub-directory.
Download the Espressif SDK (use the version mentioned in the release notes) from their
[download forum](http://bbs.espressif.com/viewforum.php?f=5) and also expand it into a
sub-directory. Then clone the ems-esp-link repository into a third sub-directory.
This way the relative paths in the Makefile will work.
If you choose a different directory structure look at the Makefile for the appropriate environment
variables to define.

In order to OTA-update the esp8266 you should `export ESP_HOSTNAME=...` with the hostname or
IP address of your module.

Now, build the code: `make` in the top-level of ems-esp-link.

A few notes from others (I can't fully verify these):
- You may need to install `zlib1g-dev` and `python-serial`
- Make sure you have the correct version of the esp_iot_sdk
- Make sure the paths at the beginning of the makefile are correct
- Make sure `esp-open-sdk/xtensa-lx106-elf/bin` is in the PATH set in the Makefile

Flashing the firmware
---------------------
This firmware supports over-the-air (OTA) flashing, so you do not have to deal with serial
flashing again after the initial one! The recommended way to flash is to use `make wiflash`
if you are also building the firmware.
If you are downloading firmware binaries use `./wiflash`.
`make wiflash` assumes that you set `ESP_HOSTNAME` to the hostname or IP address of your ems-esp-link.
You can easily do that using something like `ESP_HOSTNAME=192.168.1.5 make wiflash`.

The flashing, restart, and re-associating with your wireless network takes about 15 seconds
and is fully automatic. The 512KB flash are divided into two 236KB partitions allowing for new
code to be uploaded into one partition while running from the other. This is the official
OTA upgrade method supported by the SDK, except that the firmware is POSTed to the module
using curl as opposed to having the module download it from a cloud server.

If you are downloading the binary versions of the firmware (links forthcoming) you need to have
both `user1.bin` and `user2.bin` handy and run `wiflash.sh <esp-hostname> user1.bin user2.bin`.
This will query the ems-esp-link for which file it needs, upload the file, and then reconnect to
ensure all is well.

Note that when you flash the firmware the wifi settings are all preserved so the ems-esp-link should
reconnect to your network within a few seconds and the whole flashing process should take 15-30
from beginning to end. If you need to clear the wifi settings you need to reflash the `blank.bin`
using the serial port.

The flash configuration and the OTA upgrade process is described in more detail in [FLASH.md](FLASH.md)

Serial bridge and connections to Arduino, AVR, ARM, LPC microcontrollers
------------------------------------------------------------------------
In order to connect through the ems-esp-link to a microcontroller use port 23. For example,
on linux you can use `nc esp-hostname 23` or `telnet esp-hostname 23`.

You can reprogram an Arduino / AVR microcontroller by pointing avrdude at port 23. Instead of
specifying a serial port of the form /dev/ttyUSB0 use `net:esp-link:23` with avrdude's -P option
(where `esp-link` is either the hostname of your ems-esp-link or its IP address).
The ems-esp-link detects that avrdude starts its connection with a flash synchronization sequence
and sends a reset to the AVR microcontroller so it can switch into flash programming mode.

You can reprogram NXP's LPC800-series and many other ARM processors as well by pointing your
programmer similarly at the ems-esp-link's port 23. For example, if you are using
https://github.com/jeelabs/embello/tree/master/tools/uploader a command line like
`uploader -t -s -w ems-esp-link:23 build/firmware.bin` does the trick.
The way it works is that the uploader uses telnet protocol escape sequences in order to
make ems-esp-link issue the appropriate "ISP" and reset sequence to the microcontroller to start the
flash programming. If you use a different ARM programming tool it will work as well as long as
it starts the connection with the `?\r\n` synchronization sequence.

Note that multiple connections to port 23 can be made simultaneously. The ems-esp-link will
intermix characters received on all these connections onto the serial TX and it will
broadcast incoming characters from the serial RX to all connections. Use with caution!

Debug log
---------
The ems-esp-link web UI can display the ems-esp-link debug log (os_printf statements in the code). This
is handy but sometimes not sufficient. ems-esp-link also prints the debug info to the UART where
it is sometimes more convenient and sometimes less... For this reason three UART debug log
modes are supported that can be set in the web UI (and the mode is saved in flash):
- auto: the UART log starts enabled at boot and disables itself when ems-esp-link associates with
  an AP. It re-enables itself if the association is lost.
- off: the UART log is always off
- on: the UART log is always on

Note that even if the UART log is always off the bootloader prints to uart0 whenever the
esp8266 comes out of reset. This cannot be disabled.

Contact
-------
If you find problems with ems-esp-link, please create a github issue.
