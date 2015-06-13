---
layout: post
title:  "Pi Intruder Detector: 3 - Web & Email"
date:   2015-06-12 19:19:00
categories: raspberry pi python node express
---

**The code for the article can be downloaded at [https://github.com/itip/raspberry-pi-examples](https://github.com/itip/raspberry-pi-examples)**.


## Introduction

We're now going to make a website which allows us to view the photos taken by our Raspberry Pi. We also going to make the website send us an email every time an alert is received.

## What you'll need

**An Internet connection.**

You can use the ethernet port on your device or Wifi. I used the the *USB Wifi Adapter for the Raspberry Pi* available on [Amazon][wifi_adapter].

<img src="/assets/pi_intruder_3/wifi_adapter.jpg"
            srcset="/assets/pi_intruder_3/wifi_adapter.jpg 1x,
                    /assets/pi_intruder_3/wifi_adapter@2x.jpg 2x"
            width="250" height="250"/>

Instructions on how to set up wifi can be found on the [Raspberry Pi site][wifi_setup].


## Setting Up Your Website

When it comes to setting up a website, there are lots of different options available. We're going to keep things pretty simple though and use a service called [Parse][parse] which will allow us to host our web app with minimal setup required.

**1. Create a Parse Account**

Go to [Parse.com][parse] and create an account, if you haven't already got one.

**2. Create a New App**

Create a new app. Let's call it "Intruder Alarm".

<img src="/assets/pi_intruder_3/parse_create_app.jpg"
            srcset="/assets/pi_intruder_3/parse_create_app.jpg 1x,
                    /assets/pi_intruder_3/parse_create_app@2x.jpg 2x"
            width="400" height="280" style="border: 1px solid #BFBFBF"/>

**3. API Keys**

Go to your app's settings and find the *application key* and *rest API key*.

<img src="/assets/pi_intruder_3/parse_settings.jpg"
            srcset="/assets/pi_intruder_3/parse_settings.jpg 1x,
                    /assets/pi_intruder_3/parse_settings@2x.jpg 2x"
            width="400" height="280" style="border: 1px solid #BFBFBF"/>

<img src="/assets/pi_intruder_3/parse_keys.jpg"
          srcset="/assets/pi_intruder_3/parse_keys.jpg 1x,
                  /assets/pi_intruder_3/parse_keys@2x.jpg 2x"
          width="400" height="240" style="border: 1px solid #BFBFBF"/>

## Uploading Photos

After a photo is taken we we'll immediately try to upload it to our web app. Internet connections can go up and down so there's no guarantee that the upload will work. For this reason we'll make sure the code tries to upload the current photo, *and* and other photos which haven't yet been uploaded.

When uploading a file to Parse you need to associate it with an object in the database, otherwise it's not possible to retrieve the photo at a later date. For this reason our code needs to make two calls to the Parse API.

The following code is taken from `upload.py`. Before using it, make sure you enter your Parse keys into the two variables at the top of the file.

<pre><code>

parse_application_key = "&lt;APPLICATION_KEY&gt;"
rest_api_key = "&lt;REST_API_KEY&gt;"

..

def upload_photos():
    # Search 'photos' directory for photos which haven't been uploaded
    for file_name in listdir("photos"):
        if isfile(join("photos",file_name)):
            if allowed_file(file_name):

                print("Uploading %s" % file_name)
                file_path = join("photos",file_name)

                connection = httplib.HTTPSConnection('api.parse.com', 443)
                connection.connect()

                # Upload file to Parse

                parse_url_path = "/1/files/%s" % file_name
                connection.request('POST', parse_url_path, open(file_path, 'rb').read(), {
                       "X-Parse-Application-Id": "SRy8RdMuTcwIKE3jP2imojT0tw6Wv4f5C96VHWGW",
                       "X-Parse-REST-API-Key": "EhSHMXo8fivY1OID3nfMebZqF2BgvjE2cRwtAdmn",
                       "Content-Type": "image/png"
                     })
                result = json.loads(connection.getresponse().read())

                ...

                # Create a new "Alert" object
                parse_file_name = result['name']

                connection.request('POST', '/1/classes/Alert', json.dumps({
                   "name": file_name,
                   "picture": {
                     "name": parse_file_name,
                     "__type": "File"
                   }
                 }), {
                   "X-Parse-Application-Id": parse_application_key,
                   "X-Parse-REST-API-Key": rest_api_key,
                   "Content-Type": "application/json"
                 })
                result = json.loads(connection.getresponse().read())

                ...

                # Success. Everything uploaded. Move file to 'uploaded' folder so we don't upload again.
                if not os.path.exists("uploaded"):
                    os.makedirs("uploaded")

                uploaded_path = join("uploaded", file_name)
                os.rename(file_path, uploaded_path)
</code></pre>

If you log into the Parse portal and click on the *Core* tab you should now see a new `Alert` object. If you click on the `picture` cell you should see the photo taken by the Raspbery Pi.

<img src="/assets/pi_intruder_3/parse_portal_submission.jpg"
          srcset="/assets/pi_intruder_3/parse_portal_submission.jpg 1x,
                  /assets/pi_intruder_3/parse_portal_submission@2x.jpg 2x"
          width="600" height="252" style="border: 1px solid #BFBFBF"/>

## Parse web app

The sample code contains a simple web app for viewing the photos.

**1. Set Password**
Open `app.js` and change the `helpPassword` variable to contain a password. We'll use this password to access the portal once it has been uploaded.

**2. Web App URL**

Log into your Parse app, click on the *Settings* tab and select the *Hosting* option on the left-hand side. Under *Web Hosting*, enter a name for your app. This will be the URL which you can use to access your web portal.

<img src="/assets/pi_intruder_3/parse_hosting_setup.jpg"
          srcset="/assets/pi_intruder_3/parse_hosting_setup.jpg 1x,
                  /assets/pi_intruder_3/parse_hosting_setup@2x.jpg 2x"
          width="600" height="120" style="border: 1px solid #BFBFBF"/>

**3. Parse Command Line**

Download the Parse Command Line Tool. Instructions can be found [here][parse_command].

**4. Upload App**

Open a terminal window (or Command Prompt on Windows)

1. Go to the *AlertWeb* directory in the sample code, e.g. `cd AlertWeb`.
2. Deploy the app, `parse deploy`.
3. Follow the instructions. The first time you upload the app you'll be asked to enter your Parse username and password, this won't be required on subsequent deploys though.

Note: for more information on creating dynamic websites using Parse, take a look at [this page][parse_cloud_web] in their documentation.

**5. Check the Portal**

Open a web browser and enter the URL you entered in the Parse settings. You'll asked for a username and password. Enter:

* **Username:** admin
* **Password:** (The password you entered in step 1)

To change these login details, just open app.js, change the variables at the top of the page and deploy the app again.

You should now see the photos you've taken.

<img src="/assets/pi_intruder_3/portal_alerts.jpg"
          srcset="/assets/pi_intruder_3/portal_alerts.jpg 1x,
                  /assets/pi_intruder_3/portal_alerts@2x.jpg 2x"
          width="491" height="401" />


## Email

The final step is to send an email when an alert is received. Parse cloud code can't send emails directly so we need to sign up to an email service. We'll go for [Mandrill][mandrill].

1. Sign up for Mandrill service: [https://www.mandrill.com/signup/][mandrill_signup]
2. Click on the *Settings* tab and create an API Key (the name doesn't matter, you can call it "Emails").
3. Make a note of the key which has been generated.


<img src="/assets/pi_intruder_3/mandrill_api_key.jpg"
          srcset="/assets/pi_intruder_3/mandrill_api_key.jpg 1x,
                  /assets/pi_intruder_3/mandrill_api_key@2x.jpg 2x"
          width="600" height="362" />

Now open `main.js` in the web app. Update the `mandrillKey` variable at the top. Also enter the your email address into the variables at the top (it's ok to use the same email address in both variables).


<pre><code>
/* After adding an alert we'll send an email */
var mandrillKey = '&lt;KEY GOES HERE&gt;';

var emailSender = 'sender@email.com'
var emailRecipient = 'recipient@email.com'

var Mandrill = require('mandrill');
Mandrill.initialize(mandrillKey);

...
</code></pre>

You should now receive an email within a few seconds of movement being detected (assuming you have a reasonable connection to the Internet.)



## Conclusion

Our intruder alarm is now complete. There's plenty of room for improvement though, why not...

* Send a text message using [Twilio][twilio].
* Instead of saving pictures, modify our camera code to create videos.

I hope this has been useful. Feel free to get in touch using the contact details below.


-----





[wifi_adapter]:     http://www.amazon.co.uk/dp/B00EZOQFHO
[wifi_setup]:       http://www.raspberrypi.org/documentation/configuration/wireless/
[parse]:            https://parse.com
[parse_command]:    https://parse.com/docs/js/guide#command-line
[parse_cloud_web]:  https://parse.com/docs/js/guide#hosting-dynamic-websites
[mandrill]:         https://www.mandrill.com/
[mandrill_signup]:  https://www.mandrill.com/signup/
[twilio]:           https://www.twilio.com/
