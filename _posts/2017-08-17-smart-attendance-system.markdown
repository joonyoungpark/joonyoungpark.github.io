---
title: "SAS: Smart Attendance System"
layout: post
date: 2017-08-17 13:40
tag: 
- python
- aws
- raspberrypi3
image: http://www.osasolutions.com/wp-content/uploads/security-camera-ipad-chicago.png
headerImage: true
projects: true
hidden: true # don't count this post in blog pagination
description: "This is a simple and minimalist template for Jekyll for those who likes to eat noodles."
category: project
author: joonpark
externalLink: false
---
---
### This project was collaborated with [Andrew Luo](https://andrew-luo1.github.io/).
### View the source code [here](https://github.com/joonyoungpark/SmartAttendanceSystem).
---

<figure>
	<img class="image" src="https://blog.gemalto.com/wp-content/uploads/2017/01/ID-Verification-banks.png" alt="Alt Text">
	<figcaption>Image from <a href="https://blog.gemalto.com/financial-services/2017/01/23/6-ways-id-verification-flawed-age-digital-banking/">Gemalto blog</a></figcaption>
</figure>
## Introduction
The project was done during an internship for Stackup, a startup that teaches adults, from apprentice to developer, how to program. 

While TAing the Python course, I've noticed taking attendance was often an issue:
1. Attendance sheets and pens were constantly being moved around. 
2. Finding one’s name among the multiple ledgers and multiple attendance books was difficult and tedious. 

Me and Andrew, classmate from Waterloo and co-worker at StackUp, decided to
create a system that solves the issue with the click of a button with the concepts we learnt throughout the internship.

If you remember that decade-old Tom Cruise movie, Minority Report, where walking into a store would prompt a personal message from an automated system,
that was pretty much what our vision was towards this project. 

<iframe width="700" height="393" src="https://www.youtube.com/embed/ITjsb22-EwQ?ecver=1" frameborder="0" allowfullscreen></iframe>
<figcaption class="caption">Unfortunately iris recognition is not the case here.</figcaption>

<span class="evidence">
	**Our goal was to create an automated system - naming SAS after all - that will recognize the student's face and records it to corresponding course attendance sheet.**
</span>	

---

## Implementing Software
Internship at StackUp exposed both of us to Amazon web services
and we've chose to take advantage of some Deep learning based services: Rekognition for face-name matching and Polly for voiceover.

#### What is Amazon Rekognition?
<figure>
	<img class="image" src="https://image.slidesharecdn.com/berawed1430mac203demo-161213024232/95/aws-reinvent-2016-new-launch-introducing-amazon-rekognition-mac203-22-638.jpg?cb=1481597034" alt="Alt Text">
	<figcaption class="caption">Image from <a href="https://www.slideshare.net/AmazonWebServices/reinvent-recap-keynote-an-introduction-to-the-new-services">re:Invent Recap Keynote</a></figcaption>
</figure> 
Amazon Rekognition is a service that makes it easy to add image analysis to your applications. 
With Rekognition, you can detect objects, scenes, faces; recognize celebrities; and identify inappropriate content in images. You can also search and compare faces. 

This project uses **face comparison** function especially which measures the likelihood that faces in two images are of the same person. 
Rekognition returns the similarity score to verify a user against a reference photo in near real time.

#### What is Amazon Polly?
<figure>
	<img class="image" src="https://cdn-images-1.medium.com/max/1200/1*-LedZOdslN_04v9yieicBQ.png" alt="Alt Text">
	<figcaption class="caption">Image from <a href="https://reinvent.awsevents.com/">re:Invent Recap Keynote</a></figcaption>	
</figure>
Amazon Polly is a service that turns text into lifelike speech, allowing you to create applications that talk, and build entirely new categories of speech-enabled products. 
It uses advanced deep learning technologies to synthesize speech that sounds like a human voice.

## Required Hardwares

* 1 raspberry pi (we used rPi 3 model B)
* 1 microUSB to USB power cord for rPi
* 1 USB webcam
* 3 LED lights (one red, one yellow, one green)
* 1 small breadboard 
* 8 male-female connectors
* 1 button
* 1 mobile phone battery

---
## Flow Chart
<figure>
	<img class="image" src="..\assets\images\SAS\flow.PNG" alt="Alt Text">
</figure> 
The students have to register in advance in order to use SAS. This is done by uploading their profile image to Amazon S3 bucket, labeling the image with their name. 
Below is the main python program that the raspberrypi3 runs.

	import sqlite3
	import time
	import datetime
	import callRekog
	import os
	import ledMaster
	import sys
	import polly
	import buttonFirst
	import attendance

	while True:
		ledMaster.RedOn()
		buttonFirst.buttonPress()
		time.sleep(0.5)
		os.system("fswebcam -r 1280x720 -F 5 --no-banner 2>/dev/null photo.jpg")
		print("photo done.")

		attendance.create_MLDS_table()

		imageName = "photo.jpg"
		try:
			face = callRekog.searchFacesbyImage(imageName) #Try and except block here if the API call fails. 
		except:
			os.system("""kill $(ps aux | grep 'buttonPress.py' | awk '{print $2}' | head -n1)""") #Kill the yellow light blinking. (We ran that as a background process in buttonFirst.buttonPress)
			ledMaster.YellowOff() #In case the process was killed when the yellow light was still on
			ledMaster.RedBlink() # Blink the light 4 times to indicate an error
			sys.exit() #Stop execution of the python script.

		ledMaster.YellowOff()#in case job was killed while the yellow LED was on.
		ledMaster.Green() #Green light goes on for 4 seconds to show success. 
		polly.greeting(face) #We run this before updating table to optomize user experience. (Users don't need to see the schedule)
		attendance.update_table(face) #update the schedule
		attendance.read_from_db() #Prints out the table. 
		attendance.close_db()

While the system in on, the LED light remains *red* showing standby mode. 
When one presses the red button on top of the box to let the camera to take a photo, the LED light will instantly start to blink in *yellow* which indicates it's calling the necessary APIs.
The *yellow* light keeps on blinking until the API call to AWS Rekognition is successful and the name of the user is returned. 
The LED light turns green when the system retrieves the name and will subsequently call Amazon Polly and update the database. 

---

## Final Prototype

The design of the case was based on typical door camera with plastic cardboard as the main material. The system is fully portable by using 10000mAh power bank, expecting to last at least 6 hours from full charge.
We also went through several iterations on the design to optimize the space inside the box while avoiding contacts between the components as much as possible.

<div class="side-by-side">
    <div class="toleft">
        <img class="image" src="..\assets\images\SAS\blueprint.JPG" alt="Alt Text">
		<figcaption class="caption">one that was closest to the final design concept</figcaption>
    </div>

    <div class="toright">
        <img class="image" src="..\assets\images\SAS\development.JPG" alt="Alt Text">
		<figcaption class="caption">Me working on the hand made case</figcaption>
    </div>
</div>

Below are the images of the final prototype. 3M refill strips were used to hold the device on the wall securely. 
<figure>
	<img class="image" src="..\assets\images\SAS\front.JPG" alt="Alt Text">
	<img class="image" src="..\assets\images\SAS\side.JPG" alt="Alt Text">
	<img class="image" src="..\assets\images\SAS\inside.JPG" alt="Alt Text">
</figure>

---
## Demo Video
<iframe width="560" height="315" src="https://www.youtube.com/embed/uYNlw-RQ0Lo?ecver=1" frameborder="0" allowfullscreen></iframe>
---
## Further improvements
There are several ways that you can improve this project.

* Get the official raspberry pi camera module. Most webcams are not reliable and require you to stay still for some time to take a photo.
* Instead of calling AWS polly, consider using a built-in text to speech library. This can dramatically improve the runtime.
* Get a bluetooth speaker or a micro speaker which can be embedded inside the case. As an alternative, replace that entirely with an LCD display.
* Implement automatic photo-taking when face detected instead of having the user manually pressing the photo-taking button. This can be done with openCV.
* Replace the mobile phone charging battery for a long rPi power cord that plugs into a wall socket.
* Connect the pi’s attendance sheet to your computer via network shared folder so that it automatically synchronizes.
* Ensure that you have a reliable internet connection or connect the system to LAN.
* Write a script or find some method to quickly and conveniently upload new faces and names to the AWS Rekognition Collection.



















