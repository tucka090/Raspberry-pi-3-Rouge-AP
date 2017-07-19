#################################################################################################
## This is expecting that you just flashed/burned the kali_raspberry_pi.img file to a micro    ##
## sd card and are just booting up the raspberry pi up for the first time along with a ethernet##
## jacked into it onto your local network, along with at least 1 external wifi card hooked up  ##
## and are going to ssh into it for setting this up.                                           ##
## username: ssh root@YOUR_RASPBERRY_IP       password: toor                                   ##
#################################################################################################
## The following will allow for you to use VNC desktop, SSH, FruityWiFi, all going through     ##
## the wireless access point thats being broadcasted and you can connect to VNC desktop        ##
## to use network manager to connect your other wireless interface to a nearby wifi hotspot    ##
## for your wlan1 interface to connect to in order to supply internet connection               ##
#################################################################################################
## Start from the top and work your way to the bottom                                          ##
#################################################################################################

-------------------------------------------------------------------------------------------------

# change the password

passwd

-------------------------------------------------------------------------------------------------

apt-get update && apt-get install gparted tightvncserver -y

-------------------------------------------------------------------------------------------------

# change the password and "type n and press enter"

tightvncserver

-------------------------------------------------------------------------------------------------

# download this program called VNC viewer from this website and get it installed
# its also avavilable for android in the play store if you want to use your phone

https://www.realvnc.com/download/viewer/

-------------------------------------------------------------------------------------------------

# open up vnc viewer on your own laptop and enter the ip of your raspberry pi and :1

YOUR_RASPBERRY_IP:1

-------------------------------------------------------------------------------------------------

# through the laptops vncviewer open a terminal in the raspberry pi and type "gparted"

gparted

# right-click on the smaller partion and press "resize" to change the max size to full and press 
# "ctrl + enter" on the keyboard to apply the new changes and press yes to confirm

-------------------------------------------------------------------------------------------------

init 6

-------------------------------------------------------------------------------------------------

ssh root@YOUR_RASPBERRY_IP

-------------------------------------------------------------------------------------------------

# add the bash script below to the file /etc/init.d/vncboot

nano /etc/init.d/vncboot

-------------------------------------------------------------------------------------------------

# add this bash script to the /etc/init.d/vncboot file above

#!/bin/sh
### BEGIN INIT INFO
# Provides: vncboot
# Required-Start: $remote_fs $syslog
# Required-Stop: $remote_fs $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start VNC Server at boot time
# Description: Start VNC Server at boot time.
### END INIT INFO
 
USER=root
HOME=/root

export USER HOME

case "$1" in
 start)
   echo "Starting VNC Server"
   #Insert your favoured settings for a VNC session
   /usr/bin/vncserver :1
   ;;

 stop)
   echo "Stopping VNC Server"
   /usr/bin/vncserver -kill :0
   ;;

 *)
   echo "Usage: /etc/init.d/vncboot {start|stop}"
   exit 1
   ;;
esac

exit 0

-------------------------------------------------------------------------------------------------

chmod 755 /etc/init.d/vncboot

-------------------------------------------------------------------------------------------------

update-rc.d vncboot defaults

-------------------------------------------------------------------------------------------------

init 6

-------------------------------------------------------------------------------------------------

apt-get upgrade -y && init 6

-------------------------------------------------------------------------------------------------

apt-get install screen tor kali-linux-full fruitywifi -y && init 6

-------------------------------------------------------------------------------------------------

nano /etc/tor/torrc (#uncomment these 2 lines and change the port from 80 to 22)
	
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 22 127.0.0.1:22

-------------------------------------------------------------------------------------------------

service tor restart

-------------------------------------------------------------------------------------------------

cat /var/lib/tor/hidden_service/hostname

-------------------------------------------------------------------------------------------------

# change RASPBERRY_PI'S_TOR_ADDRESS.ONION to the address that you got from the above command

echo "mapaddress 10.40.40.44 RASPBERRY_PI'S_TOR_ADDRESS.ONION" >> /etc/tor/torrc

-------------------------------------------------------------------------------------------------

update-rc.d tor enable

-------------------------------------------------------------------------------------------------

# install tor for your own computer

apt-get install tor -y

-------------------------------------------------------------------------------------------------

# this part is for your own computer as well

# change TOR_ADDRESS.ONION to the address that you got from your raspberry pi's
# /var/lib/tor/hidden_service/hostname

echo "mapaddress 10.40.40.44 RASPBERRY_PI'S_TOR_ADDRESS.ONION" >> /etc/tor/torrc

-------------------------------------------------------------------------------------------------

# this part going down is for your raspberry pi

touch /usr/bin/start_fruitywifi

-------------------------------------------------------------------------------------------------

echo "/etc/init.d/php7.0-fpm start ; /etc/init.d/fruitywifi start" > /usr/bin/start_fruitywifi

-------------------------------------------------------------------------------------------------

chmod +x /usr/bin/start_fruitywifi

-------------------------------------------------------------------------------------------------

crontab -e 

# add the line below to the bottom of crontab config

@reboot /usr/bin/start_fruitywifi

-------------------------------------------------------------------------------------------------

init 6

-------------------------------------------------------------------------------------------------

# wait for your raspberry pi to boot up and open up your raspberry pi's fruitywifi web interface

https://YOUR_RASPBERRY_IP:8443

-------------------------------------------------------------------------------------------------

# go to the category of status and turn on the modules "Automation and Autostart"

# click on "view" for the "Autostart" tab, click on the "options" tab, then click on "Wireless 
# [AP]" module

-------------------------------------------------------------------------------------------------

# click on "view" for the "Automation" tab
# click on "OnBoot" to make sure it's on and running at the top left
# click on "Run" for the configuration button called "ap-start.conf"

# click on the "Options" tab and make sure all the "OnStart, OnStop, and OnBoot" Actions are
# choosing the "ap-start.conf" ones only

# click on the "Templates" tab. Choose the Templates pull down list and choose the 
# "ap-start.conf", this is where you may choose which interfaces to use for in and out and what 
# SSID name to give.

-------------------------------------------------------------------------------------------------

# open up this config file

nano /etc/sysctrl.conf

# Search for “net.ipv4.ip_forward=” then remove the leading hashtag "#" symbol
# Then press Ctrl+X and then press Y to save and exit nano

-------------------------------------------------------------------------------------------------

sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"

-------------------------------------------------------------------------------------------------

iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE

-------------------------------------------------------------------------------------------------

iptables -A FORWARD -i wlan0 -o wlan1 -m state --state RELATED,ESTABLISHED -j ACCEPT

-------------------------------------------------------------------------------------------------

iptables -A FORWARD -i wlan1 -o wlan0 -j ACCEPT

-------------------------------------------------------------------------------------------------

sh -c "iptables-save > /etc/iptables.ipv4.nat"

-------------------------------------------------------------------------------------------------

# open up this config file and add the line below to the config file

nano /etc/network/interfaces

-------------------------------------------------------------------------------------------------

# add the line below to the bottom of the /etc/network/interfaces file above

iptables-restore < /etc/iptables.ipv4.nat

# Then press Ctrl+X and then press Y to save and exit nano

-------------------------------------------------------------------------------------------------

# reboot the raspberry pi and wait a minute for the raspberry to boot up with vncserver, 
# ssh server, fruitywifi, and your access point in which the victims will connect to along with it 
# being what you connect to as well to access vnc, ssh, and fruitywifi to manage everything.

init 6

-------------------------------------------------------------------------------------------------

#################################################################################################
#################################################################################################
# now you can disconnect your ethernet cable and connect to your hotspot you created and 
# connect to your VNC server and connect to a wifi hotspot for your wlan1 interface

Ex. usage:

VNC:
10.0.0.1:1

FruityWiFi:
https://10.0.0.1:8443

SSH:
ssh root@10.0.0.1

-------------------------------------------------------------------------------------------------

# you can log into your raspberry pi remotely if you have internet connection

SSH-->Tor:
proxychains ssh root@10.40.40.44

#################################################################################################
#################################################################################################

-------------------------------------------------------------------------------------------------
