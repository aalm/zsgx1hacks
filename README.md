# ZS-GX1 Hacks
Record of attempted hacks on the ZS-GX1 IP Camera

This is a cheap Pan Tilt IP Camera (supposedly 1080p) that for a time was available on Gearbest for around £12. I bought 2 in an attempt to hack them as the reason they are so cheap is due to them being locked to paid cloud services.

SOC - GOKE - GK7102
https://www.unifore.net/company-highlights/goke-hd-ip-camera-solution-gk7101-gk7102.html

Sensor - SC2135 - supposedly capable of 1080p 30fps
https://www.unifore.net/product-highlights/ip-camera-smartsens-cmos-image-sensors.html

Achieved so far
* ```debug_cmd.sh``` on an SD card enables commands to be run
* Change root password to enable telnet login
* Upgrade busybox
* RTSP server accessible - rtsp://IPADDRESS/ user/pass admin/admin
* Block cloud services via hosts file
* Some GPIO functions found (IR LEDs and IR Cut)

ToDo
* Figure out GPIO control for PTZ, IR and IR Cut, Light sensor
* Change bitrate of RTSP stream
* Get rid of ```p2pcam``` and use an alternative RTSP server
* Add an SSH server (dropbear)

2017-10-22 - Update 5
* IR Cut and IR LED GPIOs found and controllable
* ```gio -s 40 1``` IR Cut - night
* ```gio -s 40 0``` IR Cut - day
* ```gio -s 46 1``` IR LEDs - on
* ```gio -s 46 0``` IR LEDs - off

2017-10-22 - Update 4
* SD card contents moved to SD card folder
* Updated hosts file included which looks like it prevents the camera from contacting the cloud services. The app shows the camera as disconnected and I'm seeing no activity on my router. RTSP server remains active.

2017-10-22 - Update 3
* RTSP server user/pass - admin/admin - Presents a 1920x1080 12fps stream on rtsp://IPADDRESS/ (no audio)
* Attempted to not run ```p2pcam``` by editing out specific parts of ```start.sh```, this resulting in the WiFi connection not being created. Maybe running ```wpa_supplicant``` via ```debug_cmd.sh``` may fix that.
* As the RTSP server seems to be created by ```p2pcam``` it might be possible to prevent the cloud software communicating outwards by using various hosts file listings.

2017-10-21 - Update 2
* home folder uploaded
* p2pcam.tar.gz is the interesting file - this gets extracted on boot and p2pcam is the camera software.

2017-10-21 - Update 1
* Photos of box and dismantled camera in the photo folder
* Initial process was to download the app via the QR code in the instructions, this gave it WIFI details to logon to, possibly this could be prevented using ethernet.
* Camera dismantled and serial pins found
* Boot output from serial dumped (uploaded)
* The serial interface auto-logs in as root - very useful, but I've been unable to find the root password
* Via serial interface you can add a new user - meaning you can then login via telnet with new user, or change the root password
```
passwd root
```
* /home/start.sh looks for /mnt/debug_cmd.sh which would be placed on a micro SD card - this should be the way to hack the camera.
* Due to the earlier "adduser" or "passwd root" not being persistent through reboots I added the following to debug_cmd.sh
```
(sleep 20 && echo "root:o.eyOMtPAPfbg:0:0:root:/root/:/bin/sh" > /etc/passwd )&
```
* The long sleep is required because the command has to be run after busybox has started - this updates the root password to "root"
* I made a quick attempt to upgrade busybox with a prebuilt binary but the autologin fails on serial - used debug_cmd.sh to copy new version over existing located at /bin/busybox - but having already updated the root password you can login over telnet
* nmap shows the following open ports
```
Starting Nmap 7.40 ( https://nmap.org ) at 2017-10-21 15:55 BST
Nmap scan report for ipc (192.168.1.147)
Host is up (0.035s latency).
Not shown: 65525 closed ports
PORT     STATE SERVICE
23/tcp   open  telnet
80/tcp   open  http
554/tcp  open  rtsp
843/tcp  open  unknown
3201/tcp open  unknown
5050/tcp open  mmcc
6670/tcp open  irc
7101/tcp open  elcn
7103/tcp open  unknown
8001/tcp open  vcom-tunnel
```
* There is an RTSP server running, but it is password protected
