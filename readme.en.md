# At VPS:

Update and upgrade the system
```
apt-get update && apt-get upgrade -y
```
Install PiVPN
```
curl -L https://install.pivpn.io | bash
```
Find the name of the interface
```
ip a
```
Edit wg0.conf file
```
nano /etc/wireguard/wg0.conf
```
```
Add the following line just after "ListenPort" (replace "your_interface" with the interface name)
```
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o your_interface -j MASQUERADE
```
reboot
```
pivpn -a
```
Enter a name for the profile, e.g., homenetwork
```
nano /etc/wireguard/wg0.conf
```
Add the home network IP range to "AllowedIPs"
```
AllowedIPs = 10.70.193.2/32, 192.168.8.0/24
```
Temporarily save the following in a text editor:
```
cat /etc/wireguard/configs/homenetwork.conf
```
Edit sysctl.conf
```
nano /etc/sysctl.conf
```
#net.ipv4.ip_forward = 1
to
net.ipv4.ip_forward = 1

```
nano /ip.sh
```
Add the following:
```
#!/bin/bash
iptables -t nat -A POSTROUTING -j MASQUERADE
```	
Redirect all ports to 192.168.8.14 (in this case, Nginx Proxy Manager)
```
iptables -A PREROUTING -t nat -p tcp --match multiport ! --dports 22,51820 -j DNAT --to-destination 192.168.8.14
iptables -A PREROUTING -t nat -p udp --match multiport ! --dports 22,51820 -j DNAT --to-destination 192.168.8.14
```
Redirect a specific port (80) to 192.168.8.14 (in this case, Webserver)
```
iptables -t nat -A PREROUTING -p "tcp" --dport "80" -j DNAT --to-destination "192.168.8.14":"80"
```
Adjust permissions:
```
chmod +x /ip.sh
```
Open Crontab:
```
crontab -e
```
Add the following line:
```
@reboot /ip.sh
```
Reboot. VPS configuration complete.
# Auf der VM:
## Docker:
## Other Systems:
Install Docker
```
curl -fsSL https://get.docker.com -o get-docker.sh && chmod +x get-docker.sh && sh get-docker.sh
```
Link to the Docker Repo:
```
https://github.com/NightMeer/WireguardClient-Docker  
```
Run Docker:
```
docker run --restart unless-stopped --privileged -d -v /host/path/config:/config ghcr.io/nightmeer/wireguardclient:latest
```
Adjust [wg0.conf](https://raw.githubusercontent.com/NightMeer/WireguardClient-Docker/main/wg0.conf) and place it in the Config folder
Restart Docker. Configuration complete.
## Unraid:

Search and install nightmeer/wireguardclient
```
nightmeer/wireguardclient
```
Adjust [wg0.conf](https://raw.githubusercontent.com/NightMeer/WireguardClient-Docker/main/wg0.conf) in "/mnt/user/appdata/wireguardclient/"
```
wget -O /mnt/user/appdata/wireguardclient/wg0.conf https://raw.githubusercontent.com/NightMeer/WireguardClient-Docker/main/wg0.conf
```
Restart Docker. Configuration complete.
## Install Wireguard:
```
apt-get update && apt-get upgrade -y && apt-get install -y wireguard openresolv wireguard-dkms
```
```
nano /etc/wireguard/wg0.conf
```
Edit wg0.conf and add content from /etc/wireguard/configs/homenetwork.conf

```
nano /etc/wireguard/wg0.conf
```
Add the following lines before [Peer] and adjust the interface (replace "your_interface"):
```
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -o your_interface -j MASQUERADE
PostUp = echo 1 > /proc/sys/net/ipv4/ip_forward
```
```
nano /etc/wireguard/wg0.conf
```
Add the following line at the end:
```
PersistentKeepalive = 25
```
Enable Wireguard
```
systemctl enable wg-quick@wg0
```
Reboot. Configuration complete.



