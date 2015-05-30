---
layout: post
title:  "Pi Intruder Detector: 1 - Taking Photos"
date:   2015-05-16 12:19:00
categories: raspberry pi camera python
---

## Introduction

In this tutorial we're going to make a simple intruder detector using a Raspberry Pi. Whenever movement is detected it'll take a photo, upload it to the web and send you an email.

There are lots of similar tutorials on the web but they typically don't do anything with the photo once it has been saved on the device. With the code in this tutorial you will get an email seconds after the intruder has been detected.

There's quite a lot to discuss so we'll break this tutorial down into three parts:

1. **Taking Photos** - using your Raspberry Pi and the camera module.
2. **Proximity Sensor** - take a photo when an intruder is detected.
3. **Web & Email** - create a simple website which lists photos and sends emails.

Let's get started with part 1...

## What you'll need

1. **Raspberry Pi** - this tutorial using the newer [Pi 2 Model B][rpi_model_2] but you should be able to use older models. [INSERT_PHOTO]
2. **Camera Module** - the official Raspberry Pi [Camera Module][rpi_camera]. [INSERT_PHOTO]
3. **Keyboard** - a standard USB keyboard is fine.
4. **Mouse** - likewise, a USB mouse is all you need.

**Note:**
If you're using a brand new Raspberry then you'll first need to set it up. Head over to the official Raspberry Pi [quick start quide][rpi_quick_start].

It's a good idea to keep the software on your Raspbery Pi up to date. The process is simple (assuming you have an Internet connect). The [Raspberry Pi][rpi_update] site has more information.

## Connecting the Camera

1. Pull tab [INSERT_PHOTO].
2. Get camera and make sure connectors are facing away from ethernet port. [INSERT_PHOTO]
3. Insert cable and push connector down.

If there's anything you're not sure about then take a look at this video made by the [Raspberry Pi team][rpi_camera_setup].
<iframe width="560" height="315" src="https://www.youtube.com/embed/GImeVqHQzsE" frameborder="0" allowfullscreen></iframe>

## Enabling the Camera

On the terminal, type `sudo raspi-config`. [INSERT_PHOTO].

1. Choose the **Camera** option. [INSERT_PHOTO]
2. Select **Enable** [INSERT_PHOTO]
3. Choose **Yes** to restart your Raspberry Pi [INSERT_PHOTO]

## Taking a Photo (Command Line)

Before we write any code we'll first do a quick test to make sure the camera is working.

1. Open the terminal. [INSERT_PHOTO]
2. Type `raspistill -o photo.jpg` to take a photo. [INSERT_PHOTO]
3. The photo (called "photo.jpg") will be saved in the current directory.

You can just as easily take a video:

1. Open the terminal. [INSERT_PHOTO]
2. Type `raspivid -o video.h264 -t 10000`.
3. The camera will record 10 seconds of video and save the file in the current directory.

**Note:** In either case if you're not sure what the current directory is, type `pwd`. [INSERT_PHOTO]


The camera is capable of doing a lot more than what we've discussed here. For more information on what's possible take a look at these pages on the Raspberry Pi site.

* Still photos - [raspistill][raspistill].
* Video - [raspivid][raspivid].


## Taking a Photo (Code)

It's now time to take a photo using code. We'll write a simple Python program which takes a photo when run. We'll use this code as a foundation for the rest of our project (in the next step we'll take a photo whenever motion is detected).

1. Open a text editor.
2. Enter the code listed below.
3. Save the file as photo.py.

<pre><code># photo.py

import picamera

# Put the data and time in the name of the photo.
# This make sure each photo has a unique name and none get overwitten.
name = "photo %s.jpg" % time.strftime("%H:%M:%S")

# Create the camera, set the brightness
camera = picamera.PiCamera()
camera.brightness = 60
camera.capture(name)
print "Photo Taken!!"</code></pre>

For details on the various settings which can be used, take a look at [this page][rpi_camera_python] on the Raspberry Pi site.

## Conclusion

So, there we have it. We've connected a camera to a Raspberry Pi and taken photos. In the next step we'll connect a proximity sensor and change the code so that it takes a photo whenever it detects movement.

[rpi_model_2]:        https://www.raspberrypi.org/products/raspberry-pi-2-model-b/
[rpi_camera]:         https://www.raspberrypi.org/products/camera-module/
[rpi_quick_start]:    https://www.raspberrypi.org/help/quick-start-guide/
[rpi_update]:         https://www.raspberrypi.org/documentation/raspbian/updating.md
[rpi_camera_setup]:   https://www.raspberrypi.org/help/camera-module-setup/
[raspistill]:         https://www.raspberrypi.org/documentation/usage/camera/raspicam/raspistill.md
[raspivid]:           https://www.raspberrypi.org/documentation/usage/camera/raspicam/raspivid.md
[rpi_camera_python]:  http://www.raspberrypi.org/documentation/usage/camera/python/README.md
