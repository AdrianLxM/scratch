Nightscout Raspberry Pi Setup
=============================

What You Need
-------------
1. [Raspberry Pi Model B] [Pi]
2. An 8GB or larger memory card
3. A network connection for the Pi - typically your home internet via ethernet
   cable or wifi
4. A domain name (web address) to use for the site
5. Admin access to the router for your network connection
6. A second computer to use for setting up the Pi and maintaining it

Raspberry Pi Basic Setup
------------------------

### Operating System Installation
There are many ways to set up an operating system on the Raspberry Pi.  For the
purposes of this document, I'll assume you're using Raspbian.  To get it
installed, follow the [instructions online] [Raspbian].

### A note to Linux newbies:

### A note about editors:

### A note about SSH:
Windows users, download [Putty] [Putty].

### Network Configuration
After you get Raspbian installed, you'll want to ensure that your Pi is using a
static IP address.  This will ensure that network traffic always makes it to
the Pi.  The exact address you choose depends on how your home network is set
up.  Typically, it's safe to take the address of your router and increment it
by one.  For example, if my router is assigned 192.168.1.1, I might make the
Pi's address 192.168.1.2.  Once you've chosen an address, configure the
networking on the Pi as follows (remember, the values depend on your specific
network, and substitute eth0 for the appropriate interface name):
       iface eth0 inet static
            address 192.168.1.2
            netmask 255.255.255.0
            nameserver 192.168.1.1

Restart networking and reconnect to the Pi.

### Updating the Pi
Make sure you update it: `sudo apt-get update ; apt-get upgrade`.  Reboot the
Pi after the updates finish: `sudo shutdown -r now`.


Nightscout Prerequisites
------------------------
Python
Apache2 + reverse proxy
Mongo + setting up journaling + setting up users
Node manual install + node in path


Installing and Setting Up Nightscout
------------------------------------
Cloning the repo
Installing node dependencies
Creating an environment variable file
First run

Remote Access
-------------
Port forwarding
DDNS
SSH Access

Security + Availability
-----------------------
SSL
Key based SSH authentication
init scripts
supervisor

Troubleshooting
---------------


Advanced Topics
---------------
Multiple T1Ds on one server
Web access to mongo data

Links
-----
[Pi]:           https://www.raspberrypi.org/products/
[Raspbian]:     https://www.raspberrypi.org/help/noobs-setup/
[Putty]:        http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
