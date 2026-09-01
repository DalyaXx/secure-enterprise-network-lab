# secure enterprise network lab 

## Objective 
Build a virtualised enterprise network to practice network segmentation, firewall configuration, and secure routing using pfsense and Linux.

## Current status : Version 1 - Basic Routing 
-Ubuntu 24.04 client VM 
-pfSense CE 2.9.0 firewall/router VM 
-Two-adapter setup : Wan (NAT) / LAN (Internal Network)
-Ubuntu receives DHCP lease from pfSense (192.168.1.x)
-Verified routing (client -> pfSense LAN -> pfsense WAN -> internet)

## Technologies 
-VirtualBox 7.2.4
-Ubuntu 24.04 LTS 
-pfSense CE 2.9.0 

## Architecture 
[diagram to add]

## Problems encoutered 
-Graphics driver crash on Ubuntu boot - fixed by doing a file-check using fsck -f /dev/sda2 and fixing broken files 
-pfSense installer defaulted to Plus which failed Negate's online eligibility's check - resolved by explicitely changing the advanced parameters to the CE version during setup
-Ubuntu didn't automatically request a new DHCP lease after switching its network adapter type - resolved with `nmcli device connect enp0s3` and restarting the network manager

## Next Step
-VLan segmentation (Management / Employees / Servers)
-Firewall rules between VLans 
-VPN
-Monitoreing / logging 






