rpi-serial-console
==================

Simple script to easily enable &amp; disable the Raspberry Pi's serial
console. Disabling the serial console is required if you want to use
the Raspberry Pi's serial port (UART) to talk to other devices e.g.
microcontrollers (see <http://elinux.org/RPi_Serial_Connection> for more
information).

Installation
------------

To install rpi-serial-console, simply run:

    sudo wget https://raw.github.com/lurch/rpi-serial-console/master/rpi-serial-console -O /usr/bin/rpi-serial-console && sudo chmod +x /usr/bin/rpi-serial-console

Usage
-----

To display whether the serial console is currently enabled or not, simply run:

    rpi-serial-console status

To enable the serial console, simpy run:

    rpi-serial-console enable

To disable the serial console, simpy run:

    rpi-serial-console disable

What could be easier than that?! ;-) After enabling or disabling the
serial console you'll need to reboot Linux for it to take effect.

Behind the scenes it automatically edits
both `/boot/cmdline.txt` and `/etc/innittab`, adding or removing the
*ttyAMA0* options as necessary. The very first time you run
rpi-serial-console it automatically creates backup copies of these
files with a `.bak` extension.

Troubleshooting
---------------

If rpi-serial-console detects that the serial console is enabled in
`/boot/cmdline.txt` but disabled in `/etc/inittab` (or vice-versa) then
it'll tell you and refuse to do anything. After manually correcting the
problem, then rpi-serial-console should run fine again.

This script has only been tested on Raspbian, so please file an issue if it
doesn't work on your distro of choice.

Advanced Usage
--------------

When enabling the serial console, rpi-serial-console will set it by default to
a baud rate of 115200. But if for some reason you want to change the baud rate
the serial console uses, you can supply this as a second argument. For example
to set the serial console to a baudrate of 57600, you'd use:

    rpi-serial-console enable 57600

and then reboot. If the serial console is already enabled, and you want to
change the baud rate, then simply use rpi-serial-console to disable it, enable
it at the new baud rate, and **then** reboot.

