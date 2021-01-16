# Pi.Alert Installation
<!--- --------------------------------------------------------------------- --->
Initially designed to run on a Raspberry PI, probably it can run on many other
Linux distributions.

Estimated time: 20'

### Dependencies
  | Dependency | Comments                                                 |
  | ---------- | -------------------------------------------------------- |
  | Lighttpd   | Probably works on other webservers / not tested          |
  | arp-scan   | Required for Scan Method 1                               |
  | Pi.hole    | Optional. Scan Method 2. Check devices doing DNS queries |
  | dnsmasq    | Optional. Scan Method 3. Check devices using DHCP server |
  | IEEE HW DB | Necessary to identified Device vendor                    |

# Installation process
<!--- --------------------------------------------------------------------- --->

## Raspberry Setup
<!--- --------------------------------------------------------------------- --->
1 - Install 'Raspberry Pi OS'
  - Instructions https://www.raspberrypi.org/documentation/installation/installing-images/
  - *Lite version (without Descktop) is enough for Pi.Alert*

2 - Activate ssh
  - Create a empty file with name 'ssh' in the boot partition of the SD

3 - Start the raspberry

4 - Login to the system with pi user
  ```
  user: pi
  password: raspberry
  ```

5 - Change the default password of pi user
  ```
  passwd
  ```

6 - Setup the basic configuration
  ```
  sudo raspi-config
  ```

7 - Optionally, configure a static IP in raspi-config

8 - Update the OS
  ```
  sudo apt-get update
  sudo apt-get upgrade
  sudo reboot
  ```

## Pi-hole Setup
<!--- --------------------------------------------------------------------- --->
1- Links & Doc
  - https://pi-hole.net/
  - https://github.com/pi-hole/pi-hole
  - https://github.com/pi-hole/pi-hole/#one-step-automated-install

2 - Login to the system with pi user

3 - Install Pi-hole
  ```
    curl -sSL https://install.pi-hole.net | bash
  ```
  - Mark "Install web admin interface"
  - Mark "Install web server lighttpd"

4 - Configure Pi-hole admin password
  ```
    pihole -a -p PASSWORD
  ```

5 - Connect to web admin panel
  - http://192.168.1.x/admin/
  - (*replace 192.168.1.x with your Raspberry IP*)

6 - Activate DHCP server
  - Pi-hole -> Settings -> DHCP -> Mark "DHCP server enabled"

7 - Add pi.alert DNS Record
  - Pi-hole -> Local DNS -> DNS Records -> Add new domain /IP
    - pi.alert    192.168.1.x
    - (*replace 192.168.1.x with your Raspberry IP*)

8 - Deactivate your current DHCP Server (*Normaly at your router or AP*)

9 - Renew your computer IP to unsure you are using the new DHCP and DNS server
  - Windows: cmd -> ipconfig /renew
  - Linux: shell -> sudo dhclient -r; sudo dhclient
  - Mac: Apple menu -> System Preferences -> Network -> Select the network ->
    Advanced -> TCP/IP -> Renew DHCP Lease

## arp-scan & Python
<!--- --------------------------------------------------------------------- --->
1 - Install arp-scan utility and test
  ```
  sudo apt-get install arp-scan
  sudo arp-scan -l
  ```

2 - Install Python & packages
  ```
  sudo apt-get install python-setuptools
  sudo apt install python3-pip

  pip3 install netaddr
  pip3 install dpkt
  pip3 install MacLookup
  ```


## Pi.Alert
<!--- --------------------------------------------------------------------- --->
1- Download Pi.Alert and uncompress
  ```
  curl -LO https://github.com/pucherot/Pi.Alert/raw/main/install/pialert_latest.tar
  tar xvf pialert_latest.tar
  rm pialert_latest.tar
  ```

2 - Public the front portal
  ```
  sudo ln -s /home/pi/pialert/front /var/www/html/pialert
  ```

3 - Update lighttpd config
  ```
  sudo sh -c "printf '\n\n\$HTTP[\"host\"] == \"pi.alert\" {\n  server.document-root = \"/var/www/html/pialert/\"\n}\n' >> /etc/lighttpd/external.conf"
  sudo /etc/init.d/lighttpd restart
  ```

4 - If you want to use email reporting with gmail
  - Go to your Google Account https://myaccount.google.com/
  - On the left navigation panel, click Security
  - On the bottom of the page, in the Less secure app access panel,
    click Turn on access
  - Click Save button

5 - Config Pialert parameters
  ```
  nano  ~/pialert/back/pialert.conf
  ```
  - If you want to use email reporting, configure this parameters
  ```ini
  REPORT_MAIL     = True
  SMTP_USER       = 'user@gmail.com'
  SMTP_PASS       = 'password'
  REPORT_TO       = 'user@gmail.com'
  ```

  - If you want to update your Dynamic DNS, configure this parameters
  ```ini
  DDNS_ACTIVE     = True
  DDNS_DOMAIN     = 'your_domain.freeddns.org'
  DDNS_USER       = 'dynu_user'
  DDNS_PASSWORD   = 'A0000000B0000000C0000000D0000000'
  DDNS_UPDATE_URL = 'https://api.dynu.com/nic/update?'
  ```

  - If you have installed Pi.hole and DHCP, activate this parameters
  ```ini
  PIHOLE_ACTIVE   = True
  DHCP_ACTIVE     = True
  ```

6 - Update vendors DB
  ```
  python ~/pialert/back/pialert.py update_vendors
  ```

7 - Test Pi.Alert Scan
  ```
  python ~/pialert/back/pialert.py internet_IP
  python ~/pialert/back/pialert.py 1
  ```

8 - Add crontab jobs
  ```
  (crontab -l 2>/dev/null; cat ~/pialert/back/pialert.cron) | crontab -
  ```

9 - Add permissions to the web-server user
  ```
  sudo chgrp -R www-data ~/pialert/back ~/pialert/back/pialert.conf ~/pialert/front ~/pialert/db
  chmod -R 770 ~/pialert/back ~/pialert/back/pialert.conf ~/pialert/front ~/pialert/db
  ```

10 - Check DNS record por pi.alert (explained in point 7 of Pi.hole installing)
   - Add pi.alert DNS Record
     - Pi-hole -> Local DNS -> DNS Records -> Add new domain /IP
       - pi.alert    192.168.1.x
       - (*replace 192.168.1.x with your Raspberry IP*)

11 - Use admin panel to configure the devices
   - http://pi.alert/
   - http://192.168.1.x/pialert/
     - (*replace 192.168.1.x with your Raspberry IP*)


## Device Management
<!--- --------------------------------------------------------------------- --->

  - [Device Management instructions](./DEVICE_MANAGEMENT.md)


### License
  GPL 3.0
  [Read more here](../LICENSE.txt)

### Contact
  pi.alert.application@gmail.com
