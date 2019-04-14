# Raspberry Pi W Zero as a WiFi Repeater (Simultaneous AP and Client WiFi)

As part of a recent project to build some in-car computer extensions for my Tesla Model 3 (more on that later), I've
been wanting to set up a Raspberry Pi (running off the car USB) to do a few things:
  1. Act as an OTG mass storage device for TeslaCam and back this up to my home NAS when I'm in range of home 
     (thank you to [marcone's fork of teslausb](https://github.com/marcone/teslausb) for Just Working for this.
  1. Broadcast a *Rik.Car* SSID, repeating either my local wifi or my phone's hotspot.
     1. Firstly, so guests in my car can use the internet! (more focused on when my family/etc visit from the UK and 
        don't have data).
     1. Also for any IoT devices I want to connect to the internet like the 
        [frunk button](https://github.com/rikbrown/tesla-frunk-button)] I'm experimenting with.
        
Anyway, I spent quite some time trying to make this work. Conceptually, `hostapd` exists to do this. But getting the
special sauce to make the Raspberry Pi Zero W successfully bridge a new AP and its existing client access point was
a struggle. A lot of the guides just didn't work for various confusing reasons.

I'm writing this blog entry to recommend this guide:
https://blog.thewalr.us/2017/09/26/raspberry-pi-zero-w-simultaneous-ap-and-managed-mode-wifi/

This basically works out the box. The sleep + restart network hack at the end is questionable but I don't want to
dig into it much more.

Changes I made:
* rather than running that hack in `crontab` as a `@reboot`, I added it to the end of `/etc/rc.local` as
  `/root/start-ap-managed-wifi.sh &`
* making that script also run `/root/bin/remountfs_rw` before. I don't really want this because the root FS could get
  corrupted. My TODO is working out what part of this script needs a RW filesystem and seeing if I can save that to disk
  forever.
