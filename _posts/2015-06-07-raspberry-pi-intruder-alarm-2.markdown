---
layout: post
title:  "Pi Intruder Detector: 2 - Motion Detector"
date:   2015-06-07 12:19:00
categories: raspberry pi proximity python
---

**The code for the article can be downloaded at [https://github.com/itip/raspberry-pi-examples](https://github.com/itip/raspberry-pi-examples)**.


## Introduction

In the second part of this tutorial we are going to attach a motion sensor to our Raspbery Pi. Used in conjunction with the camera module we will be able to take a photo of any intruders!!  

## What you’ll need ##

(In addition to the components in the [first part](/raspberry/pi/camera/python/2015/05/30/raspberry-pi-intruder-alarm-1.html) of this tutorial)

**1. PIR Infrared Motion Sensor**

We're using model HC-SR501.

<img src="/assets/pi_intruder_2/pir_sensor_1.jpg" srcset="/assets/pi_intruder_2/pir_sensor_1.jpg 1x,
                                                  /assets/pi_intruder_2/pir_sensor_1@2x.jpg 2x" width="300" height="300" style="float: left; margin-right: 10px;margin-top:10px; margin-bottom: 10px;"/>

<img src="/assets/pi_intruder_2/pir_sensor_2.jpg" srcset="/assets/pi_intruder_2/pir_sensor_2.jpg 1x,
                                                  /assets/pi_intruder_2/pir_sensor_2@2x.jpg 2x" width="300" height="300" style="float: left; margin: 10px;"/>
<br style="clear: both" />

**2. Female to female jumper wires**

<img src="/assets/pi_intruder_2/female_to_female.jpg" srcset="/assets/pi_intruder_2/female_to_female.jpg 1x,
                                                  /assets/pi_intruder_2/female_to_female@2x.jpg 2x"
                                                  width="300" height="300"/>



## Raspberry Pi and Motion Sensor ##

1. On the motion sensor, connect the "VCC" pin to the 5V pin on your Raspberry Pi (the upper-right pin).
2. Connect the middle "OUT" pin to GPIO4 (left-hand side, 4th from top).
3. Connect the ground "GND" field on the sensor to the  "GND" field on the Raspberry Pi (right-hand side, 3rd from top).
4. Run the sample code. ``sudo python sensor.py`` (Note: sudo is needed in order to access the output from the sensors).
5. When you're finished, press ``ctrl+c`` to exit.

*If you're unsure which pins to use, take a look at [this page][sensor_pins].*


<img src="/assets/pi_intruder_2/rpi_and_motion.jpg" srcset="/assets/pi_intruder_2/rpi_and_motion.jpg 1x,
                                                  /assets/pi_intruder_2/rpi_and_motion@2x.jpg 2x" width="600" height="600"/>

<pre><code># sensor.py

import RPi.GPIO as GPIO
import time

sensor = 4

# Set the way pin numbers are read. We want to use the name (Broadcom SOC channel)
# rather than the logical position, e.g. 1st, 2nd, 3rd. See http://raspberrypi.stackexchange.com/a/12967
GPIO.setmode(GPIO.BCM)

# Specify which pin we're going to use. Also add a 10K pull down resistor.
# see http://sourceforge.net/p/raspberry-gpio-python/wiki/Inputs/
GPIO.setup(sensor, GPIO.IN, GPIO.PUD_DOWN)

# Create an infinite loop which repeatedly checks the input sensor/
while True:
   time.sleep(0.1)
   if GPIO.input(sensor):
       print("Motion detected")
   else:
      print('No motion')
</code></pre>

**Notes:**

* Not sure why a pull down resistor is needed? This [article on the Sparkfun website][sparkfun_pur] explains why.

The above code works fine but it's not perfect. Under normal circumstances the sensor will be checked periodically but this might not be the case if the CPU is busy running other intensive tasks. Under such circumstances it's possible that some events may get missed as the operating system is unable to execute your code affter the interval you specified. In other words, if you want to check the sensor after 20ms but the operating system can't run your code for 200ms then any events in that period will get missed.

<img src="/assets/pi_intruder_2/rpi_motion_camera.jpg" srcset="/assets/pi_intruder_2/rpi_motion_camera.jpg 1x,
                                                  /assets/pi_intruder_2/rpi_motion_camera@2x.jpg 2x" width="450" height="600"/>

We can avoid this issue by making a slight change to the code. Instead of constantly polling the sensor to find out if anything has changed, we can and instead let the sensor tell *us* when an event is detected. This also has the added advantage of making the code look a little cleaner.

<pre><code># sensor_event.py

import RPi.GPIO as GPIO
from time import sleep

sensor = 4

# Set the way pin numbers are read. We want to use the name (Broadcom SOC channel)
# rather than the logical position, e.g. 1st, 2nd, 3rd. See http://raspberrypi.stackexchange.com/a/12967
GPIO.setmode(GPIO.BCM)

# Specify which pin we're going to use. Also add a 10K pull down resistor.
# see http://sourceforge.net/p/raspberry-gpio-python/wiki/Inputs/
GPIO.setup(sensor, GPIO.IN, GPIO.PUD_DOWN)

# This function will be called when movement is detected
def motion_detected(channel):
   print("Motion detected")

# Detect rise
GPIO.add_event_detect(sensor, GPIO.RISING, callback=motion_detected)

# Code will keep running until you press the enter key on your keyboard
close = raw_input("Press ENTER to exit\n")
print "Closing..."
</code></pre>

## Taking a Photo ##

We now want to take a photo when motion is detected. In order to do this we just need to plug in the Raspberry Pi camera module and add some code to take photos.

This code has a slight change to the way we detect events. When motion is detected our code will sometimes get called several times in quick succession. This could result in our code taking several photos when motion is detected when we really only need it to take one. We can avoid this by using the ``bouncetime`` parameter which specifies the minimum amount of time before we want to be notified of a new event.

<pre><code># sensor_event_photo.py

import picamera
import RPi.GPIO as GPIO
from time import sleep
import time

sensor = 4

# Set the way pin numbers are read. We want to use the name (Broadcom SOC channel)
# rather than the logical position, e.g. 1st, 2nd, 3rd. See http://raspberrypi.stackexchange.com/a/12967
GPIO.setmode(GPIO.BCM)

# Specify which pin we're going to use. Also add a 10K pull down resistor.
# see http://sourceforge.net/p/raspberry-gpio-python/wiki/Inputs/
GPIO.setup(sensor, GPIO.IN, GPIO.PUD_DOWN)

# Set up the camera
camera = picamera.PiCamera()
camera.vflip = True
camera.brightness = 60

# This function will be called when movement is detected
def motion_detected(channel):
   # Take a photo and store it in the current directory
   name = "photo %s.jpg" % time.strftime("%H:%M:%S")
   camera.capture(name)
   print "Photo '%s' Taken!!" % name

# Bouncetime ignores events close together which stops multiple photos being taken.
GPIO.add_event_detect(sensor, GPIO.RISING, callback=motion_detected, bouncetime=200)

# Code will keep running until you press the enter key on your keyboard
close = raw_input("Press ENTER to exit\n")
print "Closing..."
</code></pre>


## Conslusion ##

Our Raspberry Pi will now take a photo whenever it detects movement. In the final part of our tutorial we will upload these photos to the web and send an email whenever an intruder is detected.

[sensor_pins]:    https://www.raspberrypi.org/learning/parent-detector/worksheet/
[ada_fruit_pir]:      https://learn.adafruit.com/pir-passive-infrared-proximity-motion-sensor/testing-a-pir
[sparkfun_pur]:   https://learn.sparkfun.com/tutorials/pull-up-resistors
[resistor_bands]: http://www.digikey.co.uk/en/resources/conversion-calculators/conversion-calculator-resistor-color-code-4-band
