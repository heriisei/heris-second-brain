---
tags: homelab, server
---

# Static IP
- For Ubuntu with netplan
	1. Run `ifconfig` or `ip a` to find the network interface that used by your host machine
	2. Note down the interface name and the IP address of it
	3. `cd /etc/netplan`
	4. `ls`
	5. See if there is a file named `01-blabla.yaml`, create if none
	6. `sudo nano 01-network-manager-all.yaml`
	7. write this:
		```yaml
		network:
		  version: 2
		  renderer: networkd
		  ethernets:
		    wlp3s0:
		      dhcp4: no
		      addresses: [192.168.x.x/24]
		      gateway4: 192.168.x.1
		      nameservers:
		        addresses: [8.8.8.8,8.8.4.4]
		```
	6. Press `ctrl + X` then `Y` and `enter` to save 
	7. `sudo netplan try` to try if there is any errors occurred from the configuration
	8. Press `enter` if there is no errors
	9. `sudo netplan apply` to apply the configuration
	10. Check with `ifconfig` or `ip a`   