Dantherm CDP/CDP-T/CDF 40-50-70 for ESPHome
===========================================

This repository contains an [ESPHome](https://esphome.io/) package for
controlling a **Dantherm dehumidifier** in [Home
Assistant](https://www.home-assistant.io). It works for my CDP 40 unit, and
based on [Dantherm's
documentation](https://danthermpublicfiles.blob.core.windows.net/en-dantherm-manuals/Dantherm-CDP-CDP-T-CDF-40-50-70-modbus-protocol-instruction-051843-EN.pdf)
I expect it to work for all CDP, CDP-T and CDF units.

The code in this repository is not enough on its own. You'll need a circuit
board for ESPHome, and you'll need to modify an Ethernet cable, but it can all
be done without soldering.

Hardware
--------

Use a board that's compatible with ESPHome and has an RS-485 driver with
connections for A, B and GND. I think the proper cabling is to connect A and B
via a twisted pair and to connect GND via either another pair and/or or the
cable shielding.

The Dantherm unit takes its RS-485 connection via an RJ45 connector. You'll find
such a connector at the end of any Ethernet cable, but the pin assignments are
different. Numbering the pins from right to left from the perspective of someone
looking at the Dantherm unit, pin 1 is A, pin 4 is B, and pin 8 is GND. With
this pin assignment, if you use a standard Ethernet cable, A and B will not make
up a twisted pair. Things may still work if the cable is short enough, but I
haven't tried. Instead I used a crimp tool to attach an RJ45 connector to an
Ethernet cable with non-standard wiring such that pin 1 and 4 became a twisted
pair.

At the other end of the cable, connect the wires you used for A, B and GND to
your board and put a 120 ohm resistor across A and B.

I found a solder-free solution with the [LILYGO
T-CAN485](https://www.lilygo.cc/products/t-can485) board plus a 120 ohm resistor
and some Ethernet supplies bought at my local electronics store. The board has
Wi-Fi and works with ESPHome, requiring just a USB-C power supply (or 5-12V DC
if you prefer).

Software
--------

This repository is intended to be used via [the Remote Packages mechanism in
ESPHome](https://next.esphome.io/guides/configuration-types.html#remote-git-packages)
like so:

```yaml
packages:
  dantherm:
    url: https://github.com/jbj/dantherm-cdp-cdf
    file: dantherm-cdp-cdf.yaml
    refresh: 0d # Always use the latest version

substitutions:
  dantherm_uart_id: my_uart_id
  # override other variables if needed:
  # dantherm_address

uart:
  id: my_uart_id
  # configure pins and other UART settings for your board
```

If you use the LILYGO T-CAN485 as described above, you can use [the ESPHome
package I wrote for it](https://github.com/jbj/t-can485-esphome), linking the
two packages together by choosing the same UART id:

```yaml
packages:
  lilygo:
    url: https://github.com/jbj/t-can485-esphome
    file: rs485.yaml
    refresh: 0d
  dantherm:
    url: https://github.com/jbj/dantherm-cdp-cdf
    file: dantherm-cdp-cdf.yaml
    refresh: 0d # Always use the latest version

substitutions:
  # For the `dantherm` package
  dantherm_uart_id: modbus_uart
  # For the `lilygo` package
  rs485_uart_id: modbus_uart
```

That should be all the configuration you need, plus the parts that Home
Assistant generates for you when you create a new device.
