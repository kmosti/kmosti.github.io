---
layout: post
title: "You have not chosen to trust "RapidSSL SHA256 CA - G3", the issuer of the server's security certificate"
date: 2016-09-27
---

This post is a solution to an issue that I've seen a few times when using Citrix viewer on a Mac.

Basically, whenever you try to open an application, you get this annoying error message from the Citrix client stating "You have not chosen to trust $certificate, the issuer of the server's security certificate" with $certificate being some intermediate CA, like RapidSSL.

To solve this, you need to import this intermediate certificate into keychain and trust it.

I could not find any way to actually import a cert from Safari or Chrome on a Mac, so for this I used good old internet explorer from windows. This operation will probably require you to run IE as administrator (the export function will be disabled if not).

Open up the login page for citrix, then click on the padlock denoting https/secure connection, then click the "View Certificates" link:

<img src="/images/citrixerror/1.png" class="img-responsive" alt="Screenshot">

Go to the Certification Path tab and select the certificate corresponding to the error in your error message, then click "View Certificate":

<img src="/images/citrixerror/2.png" class="img-responsive" alt="Screenshot">

Open the Details tab, then select "Copy to File..". If the control is disabled, relaunch IE with "Run as administrator". Go through the certificate export keeping all default options (I believe OSX will accept all possible export options, but have not bothered to test it):

<img src="/images/citrixerror/3.png" class="img-responsive" alt="Screenshot">

Once you have exported the certificate, copy the file over on your Mac, then open Keychain, select the Certificates category and go to file -> import Items

<img src="/images/citrixerror/4.png" class="img-responsive" alt="Screenshot">

Select your previously exported certificate, then open it to import it into Keychain.

You should now see your certificate in Keychain, and you can try again to open up your application.

<img src="/images/citrixerror/5.png" class="img-responsive" alt="Screenshot">

