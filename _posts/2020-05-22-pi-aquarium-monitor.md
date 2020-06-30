---
title: "Raspberry Pi Aquarium Monitor"
categories:
  - Blog
tags:
  - Raspberry Pi
  - Linux
  - Family
---

Before we get started, I can anticipate many of your questions about this blog post already.

> "Why do you have a typo in your blog title?"
>
> "What does a delicious fruit dessert have to do with aquariums?"

If you don't have a computer geek in your life, you may not realize that a **raspberry pie** is
quite different from a **Raspberry Pi**. The latter is a low-cost, single-board computer
created by the [Raspberry Pi Foundation](https://www.raspberrypi.org/about/), a non-profit
organization based in the UK that brings the power of computing to students and hobbyists
around the world. The "Pi" in the title not only suggests the pi symbol from mathematics but also
the [Python](https://www.python.org/) computer programming language. _Hint: I am a fan of both!_

I fell in love with the [Raspberry Pi](https://www.raspberrypi.org/) for projects because it
reminded me of the early days of personal computing when my brother and I spent countless hours
tinkering around and learning how to program using our
[Bell & Howell Apple II+](https://en.wikipedia.org/wiki/Apple_II_Plus).

## Social "Fish-tancing..."

Around Christmas, my son Jacob, who is studying journalism at Oklahoma State University in
Stillwater, purchased a 20-gallon aquarium and stocked it with several freshwater fish.
Establishing a new aquarium can be a painful process, and he lost several fish during the first
few weeks until the beneficial bacteria started breaking down the waste. By the beginning
of March, the aquarium had reached a "stable" state -- right in time for his planned trip back home
to visit us during Spring Break.

That is when the world changed. What started out as a week-long visit to Edmond turned into weeks of
quarantine due to the [COVID-19 pandemic](https://en.wikipedia.org/wiki/Coronavirus_disease_2019).
Bringing the aquarium to our house was out of the question, as the move would likely disrupt the
careful balance he worked so hard to establish. Traveling back and forth to Stillwater to care for
the fish was possible but impractical. This dilemma required a creative solution to allow him to
keep an eye on his tank while he was home with us in lockdown.

## Cue the Raspberry Pi...

The Raspberry Pi is one of those devices that you **know** you want, but you may not have a
concrete **reason** to buy. Sure, its purchase can be justified for educational and entertainment
uses, but can it do **useful** things? Yes! Faced with the aquarium dilemma, I quickly knew how
my Raspberry Pi Zero W would be put to use for the next several weeks.

![Raspberry Pi Zero W](/assets/images/pizerow.jpg)
**Raspberry Pi Zero W**

For the aquarium monitor project, I used off-the-shelf components that are available for purchase
through [Amazon](https://www.amazon.com/), [AdaFruit](https://www.adafruit.com/), or other
Raspberry Pi vendors.

* A [Raspberry Pi Zero W](https://www.raspberrypi.org/products/raspberry-pi-zero-w/)
single-board computer is the centerpiece of the aquarium monitor. Although it sells by
itself for only $10, it requires some additional components such as a power supply
and micro SD card to store the operating system. For my project, I used the
[Viros Raspberry Pi Zero W Kit](https://www.amazon.com/Vilros-Raspberry-Starter-Power-Premium/dp/B0748MPQT4/)
($29) along with a
[Lexar 633x 32GB micro SD card](https://www.amazon.com/gp/product/B012PLSCQ0) ($6).

* Recording images and videos of the fish requires a camera module, and for this project, I used the
[Raspberry Pi Camera Module v2](https://www.raspberrypi.org/products/camera-module-v2/),
which sells for $28 on
[Amazon](https://www.amazon.com/Raspberry-Pi-Camera-Module-Megapixel/dp/B01ER2SKFS).
The camera is connected via a ribbon cable to the Pi Zero W and fits nicely in the Vilros camera
case included in the kit.

* We purchased an inexpensive, timer-based [fish feeder](https://www.amazon.com/gp/product/B07VCKRFGY/)
which automatically dispenses flaked fish food twice per day at 10 a.m. and 6 p.m.

* The aquarium light is connected to a [WeMo](https://www.wemo.com/) smart plug, and it is
programmed to turn on at 10 a.m. and off at 8 p.m. daily.

* The Pi is mounted to one of my spare tripods so that the entire aquarium is visible in the
camera's field of view.

## Software runs the show...

* The Pi Zero W runs an optimized version of the free and open-source
[Linux](https://en.m.wikipedia.org/wiki/Linux) operating system called
[Raspbian](https://www.raspberrypi.org/downloads/raspbian/). Raspbian is similar to Windows and
macOS in that it can run programs and handles interfacing with the computer hardware.

![Raspbian Desktop](/assets/images/raspbian-desktop.png)
**Raspbian Desktop**

* I needed to be able to access the Raspberry Pi from our house over the internet to make
adjustments to the camera settings and video schedule from our house. To do this, I used the
Real VNC server (included with Raspbian) running on the remote Raspberry Pi and the
[Real VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) on our home computer
to remotely control the Raspberry Pi over the internet.

* I wrote a simple [bash](https://en.m.wikipedia.org/wiki/Bash_(Unix_shell)) shell script which
invokes the `raspivid` program to capture a couple of minutes of video and send it to another Pi
at our house using the [SFTP](https://en.m.wikipedia.org/wiki/SSH_File_Transfer_Protocol) protocol.
The bash script is invoked every hour, on the hour, between 10 a.m. and 7 p.m. when the aquarium
light is on.

* Our home Raspberry Pi receives the video each hour and, after the 7 p.m. video arrives, another
bash script on that device combines all of the daily videos into one video which is ready to watch
at our leisure.

## Putting it all together...

After a few iterations of tweaking the bash scripts and adjusting the positioning of the camera,
we were very pleased with the end result. We can check-in on the daily activities of the
fish to make sure they are healthy, the water level is correct, and the food is being dispensed
properly.

Click [here](https://youtu.be/IS_rwr9qUGo) to watch a recent video of the aquarium.

![Raspberry Pi Aquarium Monitor](/assets/images/pi-aquarium-monitor.jpg)
**Raspberry Pi Aquarium Monitor**

![Raspberry Pi Camera](/assets/images/pi-camera.jpg)
**Close up of the Pi Camera**

## Geeky Details

If you have read this far and want more details, this section is for you. Otherwise, feel free
to stop reading!

### Bash script on the remote Raspberry Pi

I used the following bash script for the video recording on the remote Raspberry Pi.

```bash
#!/bin/bash

filename="aquarium-"`date +%b-%d-%H-%M`
raspivid -o ~/Aquarium/$filename.h264 -t 120000 -drc med -ISO 160 -fps 20 -a 1036

sftp pi@my-home-pi.my-dynamic-dns-provider.com< <<EOF
lcd ~/Aquarium/
cd /mnt/samba/Backups/Aquarium/
#put $filename.mp4
put $filename.h264
quit
EOF

rm ~/Aquarium/$filename.h264

SFTP_RETURN_CODE=${?}

# If the return code is non-zero then the upload was not successful
if [[ 0 != ${SFTP_RETURN_CODE} ]]
   then
   echo "upload failed"
   exit ${SFTP_RETURN_CODE}
else
   echo "upload successful"
fi

exit 0
```

* The `raspivid -o ~/Aquarium/$filename.h264 -t 120000 -drc med -ISO 160 -fps 20 -a 1036` command
invokes the `raspivid` program for 120000 milliseconds (2 minutes) and saves the output the
specified file name. I am using `-fps 20` to limit the frames per second to 20 (which I found was
acceptable and reduced the file size). The `-ISO 160` was another tweak to prevent parts of the
video from being overexposed. Finally, the `-a 1036` is a way to add the time and date on the video.

* The `sftp` command shown does not have my **actual** IP address. I have added the SSH public
key for the remote Raspberry Pi to the `~/.ssh/authorized_keys` file on the home Raspberry Pi.
That way the remote Pi can send the files without sending the password (since the SSH keys are
used for authentication).

### Bash script on the home Raspberry Pi

The home Raspberry Pi is responsible for combining all of the recorded videos from the day into
a single video which makes it easier to review.

```bash
#!/bin/bash
cd /mnt/samba/Backups/Aquarium
filename="aquarium-"`date +%b-%d`
MP4Box $(for file in aquarium*h264; do echo -cat $file; done) $filename.mp4
rm *.h264
```

* The `MP4Box` program is used to combine all files from the day (which are named starting with
`aquarium` and with the extension `.h264`) into a new file in MP4 format.

* After processing the files, remove the hourly `.h264` videos.

### cron jobs

The bash scripts are invoked through `cron` jobs which are created using the `crontab -e` command
and using an editor to add scripts which will be invoked at certain times. The important lines
are shown here:

```bash
# m h  dom mon dow   command
0 10 * * * /home/pi/Scripts/camera.sh
0 11 * * * /home/pi/Scripts/camera-3min.sh
0 12 * * * /home/pi/Scripts/camera.sh
0 13 * * * /home/pi/Scripts/camera.sh
0 14 * * * /home/pi/Scripts/camera.sh
0 15 * * * /home/pi/Scripts/camera.sh
0 16 * * * /home/pi/Scripts/camera.sh
0 17 * * * /home/pi/Scripts/camera.sh
0 18 * * * /home/pi/Scripts/camera-3min.sh
0 19 * * * /home/pi/Scripts/camera.sh
```

**crontab settings for remote Raspberry Pi**

* The time is specified using minutes (first number), hour (second number), day of the month, month
number, and day of the week. An asterisk `*` is used to indicate 'any'.

* In the example above, the script `camera.sh` is executed at 10 a.m., 12 p.m., 1 p.m., 2 p.m.,
3 p.m., 4 p.m., 5 p.m., and 7 p.m., while the `camera-3min.sh` is invoked at 11 a.m. and 6 p.m.
(which happens to be our feeding times).
