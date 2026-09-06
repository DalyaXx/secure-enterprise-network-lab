# secure enterprise network lab 

## Objective 
Build a virtualised enterprise network to practice network segmentation, firewall configuration, and secure routing using pfsense and Linux.

## Current status : Version 1 - Basic Routing 
- Ubuntu 24.04 client VM 
- pfSense CE 2.9.0 firewall/router VM 
- Two-adapter setup : Wan (NAT) / LAN (Internal Network)
- Ubuntu receives DHCP lease from pfSense (192.168.1.x)
- Verified routing (client -> pfSense LAN -> pfsense WAN -> internet)

## Technologies 
- VirtualBox 7.2.4
- Ubuntu 24.04 LTS 
- pfSense CE 2.9.0 

## Architecture 
[diagram to add]

## Problems encoutered 
- Graphics driver crash on Ubuntu boot - fixed by doing a file-check using fsck -f /dev/sda2 and fixing broken files 
- pfSense installer defaulted to Plus which failed Negate's online eligibility's check - resolved by explicitely changing the advanced parameters to the CE version during setup
- Ubuntu didn't automatically request a new DHCP lease after switching its network adapter type - resolved with `nmcli device connect enp0s3` and restarting the network manager

## Next Step
- VLan segmentation (Management / Employees / Servers)
- Firewall rules between VLans 
- VPN
- Monitoreing / logging 

## Version 2 - VLan Segmentation 
- created 3 VLans on pfSense : Management (VLan 10 192.168.10.0/24), Employees (VLan 20 192.168.20.0/24), Servers (VLan 30 192.168.30.0/24).
- Configured DHCP server per VLAN
- Tested VLAN tagging from Ubuntu client using NetworkManager (803.1Q tagged sub-interface), verified correct IP assignement and routing through pfSense per VLan

## Version 3 - Firewall Rules (Inter-VLan Policy)
Implemented the following security policy:
- Employees -> Internet (Allowed)
- Employees -> Servers (Limited to HTTP/HTTPS (TCP 80/443) only)
- Employees -> Management (Blocked)
- Management -> Everything (Allowed)

### Problems encountered
- Client remained dual-homed on both the original flat LAN and the new
  VLAN simultaneously after tagging, causing traffic to silently bypass
  VLAN firewall rules via the more permissive default route — resolved
  by disabling the original LAN connection profile with
  `nmcli connection down`
- Initial "Employees to Servers" rule allowed all traffic despite
  intending to limit it to web ports only, because a broader "allow any"
  rule above it in the rule list matched first — pfSense evaluates rules
  top-to-bottom with first-match-wins, so a general allow rule placed
  above a specific restriction rule causes the restriction to never be
  evaluated. Fixed by adding an explicit block rule for all other Server
  traffic, correctly ordered between the specific allow and the general
  outbound allow.

## Next steps 
- VPN (Version 4)
- Monitoring/logging (Version 5)

## Version 4 — VPN (OpenVPN)
- Configured OpenVPN server on pfSense (SSL/TLS + User Auth, UDP, AES-256-GCM)
- Created internal CA and signed server/client certificates
- Configured VPN client to access the Servers VLAN (192.168.30.0/24) through the tunnel
- Verified working tunnel from Ubuntu client: authenticated connection, correct routing via tun0, confirmed reachability to Servers VLAN

### Problems encountered
- Initially created the server certificate with the wrong Certificate Type (User Certificate instead of Server Certificate) — pfSense wouldn't allow deleting it directly since it was in use by the OpenVPN server config; resolved by creating a correctly-typed certificate, reassigning the server to use it, then deleting the old one
- "Config File Only" export was missing an externally-referenced TLS key file, causing a "cannot pre-load keyfile" error on connection; resolved by using the "Archive" export option instead, which bundles all required files together
- Tunnel connected and routed correctly but traffic was still blocked — pfSense does not automatically create firewall rules for new OpenVPN interfaces; resolved by adding an explicit Pass rule on the OpenVPN interface tab allowing VPN clients to reach the Servers VLAN

## Next steps
- Monitoring/logging (Version 5)

## Version 5 — Monitoring & Logging
- Verified pfSense firewall logging by enabling explicit logging on the Employees→Management block rule
- Confirmed blocked traffic is correctly captured in Status → System Log → Firewall, showing rule name, source, destination, and protocol
- Reviewed OpenVPN logs (Status → System Logs → OpenVPN), confirming full connection lifecycle: IP pool assignment, authentication, connection, and clean disconnect

### Problems encountered
- Blocked traffic initially did not appear in firewall logs — pfSense does not log rule matches by default; resolved by explicitly enabling the "Log packets that are handled by this rule" option on the relevant block rule

## Project Status: Complete (Versions 1-5)
This lab now demonstrates: virtualized network routing, VLAN segmentation, inter-VLAN firewall policy with correct rule ordering, a functional VPN with certificate-based authentication, and verified logging/monitoring of security events.


