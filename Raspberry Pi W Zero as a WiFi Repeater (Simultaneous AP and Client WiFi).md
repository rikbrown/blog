# Raspberry Pi W Zero as a WiFi Repeater (Simultaneous AP and Client WiFi)

As part of a recent project to build some in-car computer extensions for my Tesla Model 3 (more on that later), I've
been wanting to set up a Raspberry Pi (running off the car USB) to do a few things:
  1. Act as an OTG mass storage device for TeslaCam and back this up to my home NAS when I'm in range of home 
     (thank you to [marcone's fork of teslausb](https://github.com/marcone/teslausb) for Just Working for this.
  1. Broadcast a *Rik.Car* SSID, repeating either my local wifi or my phone's hotspot.
     1. Firstly, so guests in my car can use the internet! (more focused on when my family/etc visit from the UK and 
        don't have data).
     1. Also for any IoT devices I want to connect to the internet like the 
        [frunk button](https://github.com/rikbrown/tesla-frunk-button) I'm experimenting with.
        
Anyway, I spent quite some time trying to make this work. Conceptually, `hostapd` exists to do this. But getting the
special sauce to make the Raspberry Pi Zero W successfully bridge a new AP and its existing client access point was
a struggle. A lot of the guides just didn't work for various confusing reasons.

I'm writing this blog entry to recommend this guide:
https://blog.thewalr.us/2017/09/26/raspberry-pi-zero-w-simultaneous-ap-and-managed-mode-wifi/

This basically works out the box. The sleep + restart network hack at the end is questionable but I don't want to
dig into it much more.

Additional changes I made:
1. rather than running that hack in `crontab` as a `@reboot`, I added it to the end of `/etc/rc.local` as
  `/root/start-ap-managed-wifi.sh &`
1. **teslausb** makes the root filesystem read-only to prevent corruption from the Tesla killing the power. This breaks
  DHCP because it has mutable components. Assuming you've got a `/mutable` (teslausb does):
    1. `rm -rf /var/log/dhcp; mkdir /mutable/var_log_dhcp; ln -s /mutable/var_log_dhcp /var/log/dhcp`
    1. `rm -rf /var/lib/misc; mkdir /mutable/var_lib_misc; ln -s /mutable/var_lib_misc /var/lib/misc` 
1. I didn't like the AP SSID being available but broken until the script kicks in, so I added `ifdown --force ap0` to the    start of it (before it sleeps). This stops devices connecting but failing. (Hacks upon hacks?)
1. I found I had to manually add my LAN DNS server to `/etc/resolv.conf`, presumably because DHCP couldn't update it due to teslausb's read-only root FS.
