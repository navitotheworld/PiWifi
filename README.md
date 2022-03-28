# PiWifi
How to modify a RPi into a WiFi-router 

#(with Raspbian already insalled on a SD card, boot up RPi)
#(open cmd with ctrl+alt+t)
#Update package list from repositories by entering: 

sudo apt-get update

#Install these packages using:

sudo apt-get upgrade

#Install user space process called hostapd so RPi can connect to wireless access points and authentication servers:

sudo apt install dnsmasq hostapd

#Files not ready yet, so stop using:

sudo systemctl stop dnsmasq
sudo systemctl stop hostapd

#Edit dhcpcd file. Open terminal with:

sudo nano /etc/dhcpd.conf

#Input following at the end:

interface wlan0
  static ip_address=192.168.4.1/24
  nohook wpa_supplicant
 
#Restart daemon

sudo service dhcpcd restart

#DHCP service is provided by dnsmasq. Rename configuration file:

sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig

#Open config file to edit:

sudo nano /etc/dnsmasq.conf

#Input:

interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h

#Start dnsmasq:

sudo systemctl start dnsmasq

#Open hostapd terminal:

sudo nano /etc/hostapd/hostapd.conf

#Enter parameters of wireless network

interface=wlan0
driver=nl80211
ssid=miniProject
hwmode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

#Open default hostapd terminal:

sudo nano /etc/default/hostapd

#Tell system where to find configuration file by editing line reading "#DAEMONCONF=""
#Uncomment by removing hashtag before DAEMON...
#Replace with:

DAEMONCONF="/etc/hostapd/hostapd.conf"

#Enable and start hostapd using:

sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd

#Ensure system processes are up and running using:

sudo systemctl status hostapd
sudo systemctl status dnsmasq

#Set up routing:

sudo nano /etc/sysctl.conf

#Uncomment the line reading:

net.ipv4.ip_forward=1

#Add masquerade to control outbound traffic on the eth0 port:

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

#Store iptables rule by entering the following directly below previous command:

sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

#Install on the /etc/rc.local files using:

sudo nano /etc/rc.local

#Input the following above the line reading "exit 0":

iptables-restore < /etc/iptables.ipv4.nat


#A bridge (access point to share internet connection) can be made, but this is not required for a simple internet connection/standalon network (NAT).
