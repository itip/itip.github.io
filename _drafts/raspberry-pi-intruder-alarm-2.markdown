---
layout: post
title:  "Raspberry Pi Intruder Alarm (Part 2)"
date:   2015-05-16 12:19:00
categories: raspberry pi proximity python
---

## Getting Started

### Components (photos)

## Testing Sensor

https://learn.adafruit.com/pir-passive-infrared-proximity-motion-sensor/testing-a-pir

### Notes:

* Pull up resistor: https://learn.sparkfun.com/tutorials/pull-up-resistors
* Resistor band colours: http://www.digikey.co.uk/en/resources/conversion-calculators/conversion-calculator-resistor-color-code-4-band
* Pins: http://www.element14.com/community/servlet/JiveServlet/previewBody/73950-102-4-309126/GPIO_Pi2.png

## Reading Data Using Python

http://www.raspberrypi.org/learning/parent-detector/worksheet/

* “setmode”. This is used to define the pin numbers. We want to use the name rather than the pin number. The former is GPIO.BCM (Broadcom SOC channel) rather than the actual position, e.g. 1st, 2nd, 3rd (http://raspberrypi.stackexchange.com/a/12967).
* “setup”. Pull up/pull down resistor http://sourceforge.net/p/raspberry-gpio-python/wiki/Inputs/
* Can use events instead. Keeps code clean (multiple callbacks), don’t miss events if * CPU busy etc.
* Photos code - create “photos” directory.
  - Add checks for “photos” directory (show message if not created)
  - Install pip: https://www.raspberrypi.org/documentation/linux/software/python.md (sudo apt-get install python-pip)
  - Install requests


## Proximity Sensor & Camera

* bouncetime stops multiple photos being taken (without it, you’ll often see 2 photos being taken for each event)
