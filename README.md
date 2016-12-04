# hashpi

> Rasberry Pi running Hashicorp datacenter infrastructure (nomad, vault, consul)

## Preamble

In order to follow along with this build, you would need to have the following components and tools available to you:

### Hardware Shopping List

+ 1x [GearRobot Case](https://www.amazon.com/gp/product/B00MYFAAPO/)
+ 1x [DualComm 5-Port Switch (USB Powered)](https://www.amazon.com/gp/product/B002BSA7GG/)
+ 4x [Raspberry Pi 3 Model B Motherboard](https://www.amazon.com/gp/product/B01CD5VC92/)
+ 4x [16GB SDHC cards](https://www.amazon.com/gp/product/B004G605OA/)
+ 1x [Anker 5-Port powered USB hub](https://www.amazon.com/gp/product/B00VH8ZW02/)
+ 1x [16x2 LCD screen](https://www.adafruit.com/products/181)
+ 1x [i2c / SPI LCD backpack](https://www.adafruit.com/product/292)
+ 1x [Male to Female Jumper Wires](https://www.google.com/shopping/product/12597180957004446265)
+ 4x small Cat5e cable (typically you make these, but you can buy pre-made ones if you want)
+ 4x [USB-B to USB-micro converter](https://www.google.com/shopping/product/13932993478778087101). These are for AC power to the motherboard from the hub.

### Tools Required

+ 1x Soldering Iron
+ 1x pack of solder (leaded or lead-free, either will work)
+ 1x [Crimping Tool](https://www.amazon.com/gp/product/B002D3B97U/)
+ 1x pair of thin, long-nose pliers (these are useful for the case build)
+ 1x case of minature / precision screwdrivers (e.g. [these](http://www.homedepot.com/p/TEKTON-Precision-Screwdriver-Set-6-Piece-2985/207096248?cm_mmc=Shopping%7cTHD%7cG%7c0%7cG-BASE-PLA-D25T-HandTools%7c&gclid=CjwKEAiA94nCBRDxismumrL83icSJAAeeETQQSkzNXpnq7FmWWAG6wa_VkhktpHDJ_dErC8Cn7cvIBoC7VHw_wcB&gclsrc=aw.ds) or [these](http://www.firemountaingems.com/itemdetails/H201579TL?engine=google&campaign=[ADL]+[Non-Brand]+[PLA]+[Shopping]&adgroup=[PLA]+[Shopping]+Best+Sellers&kwid=productads-adid^113207974204-device^c-plaid^61529866819-sku^H201579TL-adType^PLA))

## Getting Started

### Hardware Buildout

+ Assemble the motherboards with the case (instructions from the case).

![stage1](https://raw.githubusercontent.com/timperrett/hashpi/master/docs/img/build-01.JPG)

![stage1](https://raw.githubusercontent.com/timperrett/hashpi/master/docs/img/build-02.JPG)


+ Cut and crimp the nessicary ethernet cables needed to connect the motherboards to the switch. Choose a scheme (A or B, but stick to just one):

![cat5](http://cdn.instructables.com/FDR/2Z0W/FPKFCZ0Q/FDR2Z0WFPKFCZ0Q.MEDIUM.gif)

+ Connect the USB power cords and the network cables to the motherboards.

![stage1](https://raw.githubusercontent.com/timperrett/hashpi/master/docs/img/build-03.jpg)

+ If you're having issues with the motherboard, there are two LEDs indicate status. Explanations for which can be [found here](http://raspberrypi.stackexchange.com/questions/24664/what-do-the-leds-on-the-b-mean).

+ Now its time to attach the i2pc backpack to the LCD. There's a decent tutorial [over here](https://learn.adafruit.com/i2c-spi-lcd-backpack), but you absolutely need to know how to solder and use the aforementioned tools (never a better time to learn this stuff that the present!).

![stage1](https://raw.githubusercontent.com/timperrett/hashpi/master/docs/img/build-04.jpg)



### Software Setup

These instructions assume you are running *Raspbian Lite*, Jesse or later (this requires [systemd](https://www.freedesktop.org/wiki/Software/systemd/)). You can download [Raspbian Lite from here](https://www.raspberrypi.org/downloads/raspbian/), and I would strongly recomend checking out resin.io [Ether](https://etcher.io/) for a quick and convenient way to flash your SD cards from OSX, with the vanilla Raspbian image you are downloading.

There is a set of initial setup that must be done manually to get the Pi's accessible remotely (and availalbe for automatic provisioning). I used the following steps to get the nodes going:

```
# set a new root password
$ sudo passwd root
<enter new password>

# set your the password for the `pi` user
$ sudo passwd pi
<enter new password>

$ sudo reboot

# update the system, disable avahi and bluetooth
$ sudo apt-get update && sudo apt -y dist-upgrade && \
  sudo systemctl stop avahi-daemon && \
  sudo systemctl disable avahi-daemon && \
  sudo systemctl disable avahi-daemon.socket && \
  sudo systemctl stop bluetooth && \
  sudo systemctl disable bluetooth & \
  sudo systemctl enable ssh && \
  sudo systemctl start ssh

# optionally install a few useful utilities
$ sudo apt-get install -y htop

```

Now we have our four Pi's running SSH and have disabled the features we wont be using in this cluster build out (e.g. bluetooth)

#### Bootstrap Playbook

The bootstrap playbook setups up core functionality so that we can run more complicated playbooks on the Pis themselves, and also get access to the cluster nodes without having to SSH with an explicit username and password

```
./playbooks/bootstrap.yml
```

#### Leaders Playbook

This playbook installs the following software

+ [Docker](https://docker.com/)
+ [Consul](https://www.consul.io/)
+ [Vault](https://www.vaultproject.io/) (uses Consul as its secure backend)
+ [StatsD Server](https://github.com/bitly/statsdaemon)
+ [Noamd](https://www.nomadproject.io/) (leader node only has the `server` component of Nomad installed)
