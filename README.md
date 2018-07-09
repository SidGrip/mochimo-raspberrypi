[![GitHub license](https://img.shields.io/github/license/0rtis/mochimo-raspberrypi.svg?style=flat-square)](https://github.com/0rtis/mochimo-raspberrypi/blob/master/LICENSE)
[![Follow @Ortis95](https://img.shields.io/twitter/follow/Ortis95.svg?style=flat-square)](https://twitter.com/intent/follow?screen_name=Ortis95) 


## A step by step guide for mining Mochimo on Raspberrypi

This guide will walk you through the step of running several miner on a [Raspberrypi Model 3b+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/). Other model have been reported to work as well but it is recommended to stick with the 3b+. 


### What is Mochimo ?

[Mochimo](https://mochimo.org/) is a quantum proof, scalable, ASIC-resistant POW cryptocurrency.


### Why mining Mochimo on a Raspberrypi is great
Mochimo is one of the very few cryptocurrency for which Raspberrypi is a perfect fit. On the Mochimo network, every node is a full, independent node. You don't have to trust a masternode or a reference node for your transaction, you only need to trust your own node. Even though a Mochimo node can synchronize with the network in a few minutes (!) it is better to have it always up and running. In addition, in order to enforce the '1 CPU = 1 Vote' philosophy of the Mochimo network, the mining process is based on an ASIC-resistant and GPU-soft-resistant (GPU mining does not bring extra hashing power). So even though your Raspberrypi will output less Haiku Per Second than the latest bananazillionHz CPU, you still get a very fair chance of solving block in the long run. All of that for a fraction of the price of a dedicated mining rig: Raspberrypi 3b+ ships for $50 including AC power cord & SD card and cost less $15 to run a full year (those numbers are estimates and can vary depending on vendor and power cost). 


### Install & Configure the Pi

This guide requires mid-level Linux skill set. Beginner might find it hard to keep up with. Do not hesitate to search online for the command or vocabulary you are not familiar with.


##### Install Hypriot OS 

[Hypriot](https://github.com/hypriot/image-builder-rpi) is a Linux distribution optimized for [Docker](https://www.docker.com/).


*Hypriot is a CLI only distribution. If you need a Graphical Environment (aka Desktop), I successfully ran a similar setup on [Raspex](https://sourceforge.net/projects/raspex/). Note that Raspex does not comes with pre installed Docker package, you will need to install it yourserlf. Also, Graphical Desktop have a non negligible cost of both CPU & RAM that might impact your HPS.*

1. Download the latest [Hypriot release](https://blog.hypriot.com/downloads/)
2. Flash the SD card with [Etcher](https://etcher.io/) (or an other flashing tool). Select the `.img` file from the Hypriot folder and press Flash!
3. Install the SD card in the Raspberry pi, plug a RJ45 Ethernet cable and HDMI cable to access your Pi
4. Plug the AC cord, the Pi will start
5. Wait for the booting process to complete. Hit Enter. You will be asked for a login then a password. Default login is `pirate`, default password `hypriot`
6. *Optional*: Find the IP of your Pi: `ip a` (search for `eth0` interface). Open a SSH terminal to your Pi from you PC. You can unplug the screen from the Pi
7. Update the `root` password: `sudo passwd root`
8. Create a new user: `sudo adduser myuser`
9. Add the user to the `sudo` group: `sudo adduser myuser sudo`
10. Reboot the Pi: `sudo reboot`
11. Login as `myuser`
12. Delete default user `pirate`: `sudo deluser pirate`
13. Update `apt` repository: `sudo apt update` 


##### Configure Wifi

*You can skip this part if you don't intend to use the wifi with your Pi*

1. Open `/etc/network/interfaces`: `sudo nano /etc/network/interfaces` or `sudo vim /etc/network/interfaces`
2. The file will look like that <br/> 
	```
	# interfaces(5) file used by ifup(8) and ifdown(8)

	# Please note that this file is written to be used with dhcpcd
	# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

	# Include files from /etc/network/interfaces.d:
	source-directory /etc/network/interfaces.d

	```
    Comment the line `source-directory /etc/network/interfaces.d` and add a wifi interface `wlan0` (here `static` but you can use `dhcp`)
    ```
    # interfaces(5) file used by ifup(8) and ifdown(8)

	# Please note that this file is written to be used with dhcpcd
	# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'
	
	# Include files from /etc/network/interfaces.d:
	#source-directory /etc/network/interfaces.d

	auto wlan0
	iface wlan0 inet static
   		address STATIC_IP
   		netmask 255.255.255.0
   		gateway YOUR_GATEWAY
   		wpa-essid YOUR_SSID
   		wpa-psk YOUR_SSID_PASSWORD
   		dns-nameservers 8.8.8.8 YOUR_GATEWAY
    ```      
3. Activate newly created `wlan0` interface: `sudo ifup wlan0`
4. In case of errors, try to desactive `wlan0` with `sudo ifdown wlan0` and flush with `sudo ip addr flush dev wlan0`. Try (3) again
5. Check `wlan0` status: `ip a`. You should see the ip of `wlan0`
6. Unplug the RJ45 Ethernet cable
7. Reboot the Pi with `sudo reboot` and cross finger
8. From your PC, try to ping the Pi on its wifi IP
9. If you get ping reply, that means that your Pi rebooted and succesfully connected to the Wifi by itself. Congrats ! If not, repeat the process and double check each step
10. Open a SSH connection to your Pi using the wifi IP
11. Check that the dns server a correctly configured: `sudo ping www.google.com`. You should get some ping reply.


##### Set up Docker container

Using [Docker](https://www.docker.com) containers will let us run several instance of Mochimo miner on the Pi and fully exploit its multicore capabilities.

1. Upload `mochimo-raspberrypi/image` and `mochimo-raspberrypi/compose` folder from this repository to the Pi in `/home/myuser`
2. Step into `/home/myuser/mochimo-raspberrypi`: `cd /home/myuser/mochimo-raspberrypi` 
3. Build the Docker image `mochinode`: `sudo docker build -t mochinode` (This may take several minutes. Be patient)
4. Check if `mochinode` image have been built: `sudo docker images` (should list `mochinode`)
5. Create a `mochimo-shared` directory: `sudo mkdir /mochimo-shared`. This folder will be shared across our Docker instances
6. Set permission: `sudo chmod 777 mochimo-shared`
7. Download the Mochimo miner from the official [repository](https://github.com/mochimodev/mochimo)
8. Upload the pacakge to the Pi and into `mochimo-shared`
9. Step into `/home/myuser/mochimo-raspberrypi/compose`: `cd /home/myuser/mochimo-raspberrypi/compose`
9. Start Docker containers: `sudo docker-compose up -d`
10. Check if the containers are listening in port 220 and 221: `netstat -anp | grep LISTEN | grep 22`
11. Connect to `mochimo-master` container: `ssh root@localhost:220` with default password `password`
12. Change `root` password: `passwd root`.
13. Connect to `mochimo-slave`: `ssh root@localhost:221` with default password `password`
14. Change `root` password: `passwd root`.


You can list existing containers with `sudo docker container ls` and delete a container with `sudo docker rm -f CONTAINER_ID`


#### Mining on Docker container

1. Connect to `mochimo-master` container: `ssh root@localhost:220`
2. Copy the Mochimo-miner archive to the local directory: `cp -p /mochimo-shared/MOCHIMO-MINER.targz ~`
3. Step into the local directory: `cd ~`
4. Follow installation instruction in the Mochimo-miner archive
5. Edit `~/mochi/bin/coreip.lst`: `nano ~/mochi/bin/coreip.lst` or `vim ~/mochi/bin/coreip.lst`
6. Add the IP of `mochimo-slave` `172.16.2.11` and start the miner with option `-f`
5. Repeat (1) to (4) for `mochimo-slave`: `ssh root@localhost:221`
6. Edit `~/mochi/bin/coreip.lst`: `nano ~/mochi/bin/coreip.lst` or `vim ~/mochi/bin/coreip.lst`
7. Add the IP of `mochimo-master` `172.16.2.10` and remove the other entries. Start the miner with option `-q1`



### Donation
Like the project ? Consider making a donation :) 

ETH & ERC-20: _0xaE247d13763395aD0B2BE574802B2E8B97074946_

BTC: _18tJbEM2puwPBhTmbBkqKFzRdpwoq4Ja2a_

BCH: _16b8T1LB3ViBUfePCMuRfZhUiZaV7tUxGn_

LTC: _Lgi89D1AmniNS8cxyQmXJhKm9SCXt8fQWC_



