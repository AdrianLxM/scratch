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
installed, follow the [instructions online] [Raspbian].  Most likely, for the
initial setup, you'll have the Pi connected to a TV and a keyboard.  Once we
get it set up, we'll be able to access it remotely.

**A note about editors, and sudo:**
> A lot of the instructions here will have you either creating or editing
> simple text files.  There are a number of editors available on Linux, and
> you're free to use whichever one suits you best.  If you aren't familiar
> with Linux, you'll probably find `nano` the simplest to use.  You can run it
> by typing `nano` at the shell prompt.  You type directly into the editor as
> you might expect, and you can navigate inside files using the arrow keys.
> To save files, press Ctrl+O, and to exit, press Ctrl+X.
>
> By default, you'll probably be logging into your Pi using a normal user
> account, like the default "pi" user.  Normal accounts don't have access to
> edit many of the system configuration files as a security precaution, and
> this is a good thing!  However, many of the files we need to edit in this
> document do fall into that category of system files, and thus require us to
> have superuser privileges to modify them.  Whenever you don't have access to
> do something like edit a file, you can use `sudo` to temporarily grant
> yourself superuser privileges, as you'll see below.  Use this with caution,
> as it provides no real restrictions to what you can do on the system!

### Network Configuration
After you get Raspbian installed, you'll want to ensure that your Pi is using a
static IP address.  Most home networks are set up to issue addresses
dynamically, which is great for easily connecting new devices to your network,
but problematic when you want to have a server reachable in a consistent, known
location.  By setting up your Pi with a static IP address, you will ensure that
network traffic always makes it to the Pi (at least, whenever it's powered on
and connected!).  The exact address you choose depends on how your home network
is set up.  Typically, it's safe to take the address of your router and
increment it by one.  For example, if my router is assigned 192.168.1.1, I
might make the Pi's address 192.168.1.2.  Assuming that your Pi is already
connected and able to access your network, you can find your router's address
by typing `ip route` from the shell on your Pi, and looking for the line that
begins with `default via`.  The address there should begin with a "10", "172",
or "192.168".  Once you've chosen an address for the Pi, there are a couple
ways you can configure the networking on the Pi to use it.  The simplest is to
edit the network configuration file (`sudo nano /etc/network/interfaces`) and
modify the relevant section to match your setup.  Remember, the values depend
on your specific network.

    iface eth0 inet static
        address 192.168.1.2
        netmask 255.255.255.0
        gateway 192.168.1.1

You'll also need to edit the nameserver file (`sudo nano /etc/resolv.conf`),
which lists the DNS servers your Pi uses.  DNS, the Domain Name System, is a
service to handle translating names like "google.com" into the IP addresses
computers use, like "216.58.216.46".  In that file, add your router's address.
You can also add Google's DNS server (8.8.8.8), if you want a backup.

    nameserver 192.168.1.1
    nameserver 8.8.8.8

Restart networking on the Pi (`sudo service networking reload`) and verify that
your Pi can still access the Internet (for example, you can type
`ping -c 1 google.com` and you should get a reply from Google's servers).

Another way you can set up networking on Linux is through the graphical desktop
environment.  Instructions for that will depend on what desktop you're using,
but there's typically an icon in the system tray which allows you to modify the
configuration.

**A note about SSH:**
> Unless you want to have your Pi constantly connected to a keyboard and a TV,
> you'll want to be able to access it remotely.  The typical way to do so is
> by using Secure Shell (SSH), which allows you to log into your Pi from
> another machine, and access a shell where you can enter commands.  SSH is
> installed by default on Linux and Mac.  If you're using a Windows machine to
> set up your Pi, you'll want to download [Putty] [Putty].

### Updating the Pi
Now let's make sure you have the latest sofware on your Pi. Issue the following
command to update it: `sudo apt-get update && sudo apt-get upgrade`.  Reboot the
Pi after the updates finish: `sudo shutdown -r now`.


Nightscout Prerequisites
------------------------

Now we want to get all the supporting software that Nightscout needs.  Some of
this will come from the Raspbian repositories, and can be automatically
installed, and some will need to be downloaded and installed manually.

### Python, Apache, Build Tools, and Git
Python is a programming language which Nightscout's sofware ultimately depends
on.  Apache is a professional-grade web server which we'll use later on as an
optional front-end for Nightscout.  g++ and make are build tools.  Git is a
sofware version system (as you might recognize from github.com) which we'll use
to grab a copy of Nightscout.  To install all of these, you can run this single
command:
`sudo apt-get install python-software-properties python apache2 g++ make git`

### Mongo Database
There's not an official Mongo installation on the Pi yet, as far as I can tell,
but some other great users have made their own build of the software and made
it available on GitHub, which is a quick way to get up and running.  However,
note that this install of Mongo is using a pretty old version of the software,
so there may be issues with Nightscout in the future.  I can confirm that it
works fine with the Enchilada release.  Here are the steps to install it:

    cd ~
    git clone https://github.com/svvitale/mongo4pi.git
    cd mongo4pi
    sudo ./install.sh

After the install, Mongo should be running (try `service mongod status` to
check).  We need to tweak its configuration slightly for better safeguarding
against errors and to enable secure logins: `sudo nano /etc/mongodb.conf`:
1. Find the line with `nojournal = true` and change it to `journal = true`.
2. Find the line with `noauth = true` and change it to `auth = true`.

Then you can restart mongo: `sudo service mongod restart`.  Next, we have to
add a user to Mongo so that Nightscout can connect to it.  We'll do that using
the Mongo shell, which lets us manipulate the database directly.  Run Mongo's
shell: `/opt/mongo/bin/mongo`.  It should connect to the copy of Mongo on your
Pi.  Next, create a collection where Nightscout will put all its data.  You can
call this anything you want, but remember it, because you'll be using it later
as part of the Nightscout configuration (and potentially the Android upload app
configuration as well): `use nightscoutData`  You should get a message saying
that the shell "switched to db nightscoutData".  The last step here is to
create a user inside that new collection.  Again, you can pick any values for
the username and password, but remember what you chose, and keep them secure:
`db.addUser("username", "password")` You should get a message ending with
`"ok" : 1`.  Now, type `exit` and you're done setting up Mongo.

### Node.js
Node.js is the engine upon which Nightscout is built.  There is a version of it
available directly from Raspbian, but as of this writing, that version is too
out-of-date to work properly with Nightscout, so we need to do a manual install
of it.

TODO: manual install + node in path


Installing and Setting Up Nightscout
------------------------------------
1. Cloning the repo
2. Installing node dependencies
3. Creating an environment variable file
4. First run

Remote Access
-------------
1. Port forwarding
2. DDNS
3. SSH Access

Security + Availability
-----------------------
1. SSL
2. Key based SSH authentication
3. init scripts
4. supervisor

Troubleshooting
---------------


Advanced Topics
---------------
1. Multiple T1Ds on one server
2. Web access to mongo data

Links
-----
[Pi]:           https://www.raspberrypi.org/products/
[Raspbian]:     https://www.raspberrypi.org/help/noobs-setup/
[Putty]:        http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
