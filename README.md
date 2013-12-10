pinamic
=======

Walkthru: Remote to your Raspberry Pi from anywhere with Dynamic DNS

A guide for educators and others who would like remote servings of Raspberry Pi.  Currently targeted for Mac users.

### - Under construction -

## Introduction
I recently undertook a project to be able to get to my Raspberry Pi over the internet.  Using VNC I now have full, remote access to my Pi from anywhere with an Internet connection.  This is great for people teaching Pi-based courses like me and anyone who would like the convenience of remote Pi access.  This wasn't a super straightforward process and has helped me understand a little about how DNS and local networks operate.  I hope you find it useful.

### What you'll get:
* A way to remotely access your home Raspberry Pi from anywhere with Internet access
* Also you can remotely access other computers on the same network by repeating the steps below for them too (you'll see below that I've done this for my [Pogoplug](https://plus.google.com/u/0/101710108233003380197/posts/LjnVTUKgUjH) as well)

### What you'll Need:
* A working, Internet-connected Raspberry Pi
* tightvncserver running on your Pi (this is a  tutorial on remote access, not VNC; see [here](http://trevorappleton.blogspot.com/2013/03/remotely-connect-to-raspberry-pi-desktop.html) or [here](http://interlockroc.org/2012/12/06/raspberry-pi-macgyver/) for that)
* ssh running on your Pi (you can do this on Raspbian with raspi-config as explained [here](http://elinux.org/RPi_raspi-config#ssh_-_Enable_or_disable_ssh_server))
* A router you control (which means this probably wont work at your office or school)
* A dynamic DNS service provider (I use dyn.com as described below)
* A Mac or the ability to translate these instructions into your OS.  If you can write a version of this tutorial for another OS, please Fork and do so!

### Background: VNCing for peripheral-free Pi
As I describe [here](http://hastac.org/blogs/eah13/2013/02/16/how-i-use-hastac-case-study), VNCing into your Pi is a great way to use it without needing a bunch of peripherals.  I found a [great tutorial](http://interlockroc.org/2012/12/06/raspberry-pi-macgyver/) about how to access your Pi locally from a Mac or Linux machine from Interloc Rochester.  This makes it easy to hack on your Pi- all you need are your pi, an ethernet cable, and a micro USB cable and you're able to access your Pi's full graphical glory from the comfort of your couch.  This makes teaching with the Pi, as I'll be doing at UNC in the fall, much more practical because students can hack at home without an external monitor.  Here's my Pi on my Mac:
![Pi on Mac](https://github.com/eah13/pinamic/blob/master/images/pionmac.png)


This arrangement also makes it very easy to do screen grabs of the Pi (`shift-ctrl-4` on Macs) and upload them to blog posts like this or put them in the course materials I'm working on.

## Walthru
Lugging the Pi around with me is a (very light) pain.  I *must* VNC to it from anywhere with an Internet connection.  Let's get started, shall we?

### Pi from Anywhere: DNS Basics
To do this, I'd need a way to find the IP address of the Pi.  The thing is, the IP address of my home network is always changing.  Most residential internet providers switch your IP address every day or so, for their convenience.  Internet businesses and supernerds that host their own servers may pay extra to have the same IP address all the time.  This consistency allows you to use a Domain Name Server (DNS) point a string- like elliotthauser.com- to an IP address.  My site is hosted by UNC, so I've told GoDaddy.com to point the DNS record for elliotthauser.com to 152.19.243.39, which is a number the friendly folks at web.unc.edu gave me.  Click on the link to that IP address and you'll see it points to the same place.  UNC has fixed IP addresses for the servers it hosts.

OK but my home address is always changing (or changing enough that I couldn't reliably get to it via an IP address).  What to do?

### Pi from Anywhere: Dynamic DNS Services
I figured out that if I could just find some way to always have the IP address of my home machines updated then I would know where to point my VNC client (or ssh client for that matter).  There are services that do this: they're called Dynamic DNS services.  Basically they run DNS servers that get constant updates from a machine on your network.  I signed up for one at Dyndns.com, but [there are lots'(http://dnslookup.me/dynamic-dns/).  You have to provide them CC info for a 14 day 'Pro' trial, but if you cancel you can keep one Dyn DNS server for free, which is all most people will need.  They'll autocomplete your IP, which will get you started if you're setting up from home, but you'll need to install and run an update client so that your IP information stays up to date in their servers.  Dyn's update clients are [here](http://dyn.com/support/clients/).  You will pick a hostname for your remote access point from one of Dyn's 260 servers such as gotdns.org.  You'll end up with yourname.gotdns.org or something similar.

If you've configured this correctly, you should be able to get to your home internet router through that new domain you just setup.  Check it like this:
```
> traceroute yourname.gotdns.org
```
Replace `yourname.gotdns.org` with the name you registered with your Dynamic DNS provider.

### More on DNS
DNS is the foundation of the Internet.  We don't really need to know much about it because it works so well, but its' fascinating to think how well the system works.  Here's an [animated video](http://www.youtube.com/watch?feature=player_embedded&v=2ZUxoi7YNgs
) on how DNS works generally.  And a little more in depth information [here](http://www.howstuffworks.com/dns.htm).

### Routing Incoming Connections
OK.  Now we've gotten to your home router but we need a reliable way to get to your Pi.  Your router likely works similar to your Internet provider in that it assigns computers that connect to it random local IP addresses based on convenience.  For reliable access to your Pi, we need to force it to assign the Pi the same address each time.  Then we can tell the router to force incoming connections to a specific IP address and, voila, Reliable Remote Pi.

I'm using Apple's Airport Extreme for my router so the instructions are based on that.  But you should be able to google how to do these things for whichever router you have.

 

*First*, connect your Pi into your router however you do so and make sure you have internet.  (type ping google.com in a terminal to check quickly).

*Next*, find out what your Pi's MAC address is.  This address is used to distinguish between different hardware (computers, phones, etc) connecting to a router.  I typed `ifconfig` to do this:
![ifconfig](https://github.com/eah13/pinamic/blob/master/images/ifconfig.jpg)

The MAC address of my Pi is the HWaddr listed under eth0.  If you're using wireless you will want to pick the HWaddr of that device instead.  There are other services that can get you this number as well.  I recommend Fing if you have an iPhone or iPad.

*Next*, make sure your router gives your Pi the same  local IP address every time.  For my setup that meant going to the Airport Utility, selecting my router, and editing under the DHCP Reservations under Network tab.  I added the Raspberry Pi's MAC address and picked a IP address that was a multiple of 10 on the end for ease of remembering.  
![DHCP](https://github.com/eah13/pinamic/blob/master/images/dhcpreserv.jpg)

*Next*, you have to configure your router to forward requests that come in on a specific port to the same address we just reserved for the Pi. For my router this is again under the Network tab in a section called Port Forwarding.  The 'Public TCP Port' box is the externally facing port, while the Private IP Address should be our reservation.  Think of it kind of like a PIN number: it's an added (but weak) layer of security against random people trying to get into your Pi.  The Private TCP Port should be 22, which is the standard ssh port. (why ssh?  Because we're going to tunnel in to VNC thru ssh).  Also, check the Enable NAT Port Mapping Protocol box.
![Port Mapping](https://github.com/eah13/pinamic/blob/master/images/portmapping.jpg)

Your router will reset and once it's back online, any calls to yourdynamichost.com:3000 (or whatever port you chose) should now route straight to your Pi.  Sweet!

 

Now you'll need [Chicken of the VNC](http://sourceforge.net/projects/chicken/files/?source=navbar) or another VNC Client to connect to your vncserver running on your Pi. My setup looks like this:
![VNC Setup](https://github.com/eah13/pinamic/blob/master/images/VNCsetup.jpg)


And by pressing Connect, I get this:
![Remote Raspberry Pi](https://github.com/eah13/pinamic/blob/master/images/remotepi.jpg)


 

I made the Pi's screen 1280x800 (via `vncserver -geometry 1280x800`) so that it's seamless in fullscreen on my laptop.

 
## Links
I'm not the first or the last to do this or some version of it.  These links are related walkthrus you may find interested.
* Edmundo Fuentes' walkthru [here](http://edmundofuentes.com/post/45179343394/raspberry-pi-without-keyboard-mouse-nor-screen)
* Interloc Rochester walkthru [here](http://interlockroc.org/2012/12/06/raspberry-pi-macgyver/)
* Igor Partola teaches you why IPv6 means you don't need Dynamic DNS anymore [here](http://igorpartola.com/ipv6-2/you-need-ipv6-now-and-heres-how-to-get-it)
* Trevor Appleton's VNC walkthru [here](http://trevorappleton.blogspot.com/2013/03/remotely-connect-to-raspberry-pi-desktop.html)

## To be Continued...
This is a rough guide that needs some improvement.  If there are parts that aren't clear let me know via the Issues tab and I'll make them better.  Or, better yet, open and issue and then submit a pull request!  Current needs:
* A diagram of port mapping
* Windows and Linux instructions
* An IPv6 tutorial so we don't need Dynamic DNS at all.
