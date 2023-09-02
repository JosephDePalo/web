---
title: "Setting up a Raspberry Pi 4 Home Server"
date: 2023-09-02T10:03:59-04:00
draft: false
---

# The Raspberry Pi 4
Last year I was curious about embedded systems. Around Christmas time I bought a [Raspberry Pi 4 kit](https://www.canakit.com/raspberry-pi-4-starter-kit.html), one of the most popular boards for embedded systems hobbyists. The Raspberry Pi 4 is a single-board computer (SBC) capable of running a full Linux operating system. It has a 1.5GHz quad-core ARM processor, 8GB of RAM, and a 64-bit architecture. It's a powerful little machine, and it's cheap. The kit I bought was $100, and it came with a case, a power supply, a microSD card, and a few other accessories.

Now that my interests have changed and I've been inclined to learn more about system administration, it seemed fitting to turn this little board into a home server I could access from my college campus and use as a playground for self-hosting projects.

# Installation
### Choosing an OS
Assembling the Pi was quite easy, but getting it to boot was a different story. I began by [flashing an image](https://etcher.balena.io/) of [Debian 12 for the ARM architecture](https://raspi.debian.net/tested-images/) onto the 32 GB microSD card my kit came with. I chose stock Debian over the [Raspberry Pi OS](https://www.raspberrypi.com/software/) because I didn't want any of the extra features or bloat that it may come with. There was also the option to install [Arch Linux using the ARM port](https://archlinuxarm.org/), which I am much more comfortable with, but I wanted to learn more about Debian and stood to benefit from its stability.

### Troubleshooting
Upon inserting the card into the Pi, I was greeted by nothing but a blank screen. After much troubleshooting, I couldn't get anything but it. The most I got was after giving in and trying to the Raspberry Pi OS. I got to the NOOBS installer, but not much further before the screen went. Given the blank screen I was sure that my problem was some sort of display issue, so I tried a different HDMI cable and monitor to no avail. I was almost ready to call the board defective and send it back, but instead I kept at it.

I did some more research on troubleshooting the Pi and discovered [the meaning of the green light](https://raspberrytips.com/green-and-red-light-on-raspberry-pi/) on my board. On boot, the light would flash 4 times and no more—in other words, there was a problem reading from the microSD card. Luckily I had a spare 64 GB microSD card lying around, so I flashed Debian onto that and inserted it. I've never been more pleased to see the Debian installer successfully displayed before me.

# Setup, SSH, and PiVPN
After going through the Debian installer I configured the basics like any decent system administrator would: I connected to my home WiFi using [NetworkManager](https://wiki.debian.org/NetworkManager), created a user account, and installed some basic programs like `sudo` and `curl`.

Next up was getting SSH working. sshd, th3 standard SSH daemon for Debian, was already running upon installation, so all I had to do was connect from my laptop and all was good. I configured passwordless login to make things easier by copying my laptop's public key to my Pi using `ssh-copy-id -i ~/.ssh/id_rsa.pub pi@PI_IP`. To harden my Pi I also disabled root login via SSH.

I could have stopped here and just opened up port 22 on my home firewall, and everything would work fine, but I don't like having too many open ports on my network. If I want to access my Pi from outside of my home network, however, I need to open something up. To make sure I don't have an exposed SSH port I decided to setup a VPN to act as my Pi's liaison. This was as simple as curling the [PiVPN](https://pivpn.io/) install script and selecting my options. I generated and copied a Wireguard configuration for my laptop, created a firewall rule to route Wireguard traffic to my Pi, and everything was setup.

# Conclusion
Now that I have a low power Linux server I can run 24/7 I now have the opportunity to self-host content and access it from wherever I want. PiVPN provides me an additional layer of access control so I can selectively expose some services to the internet and others only to my home network and authorized devices. Next I will probably setup some microservices to run on it using Docker.

{{< image
src="/images/pi_neofetch.png"
alt="My Pi running Debian" >}}