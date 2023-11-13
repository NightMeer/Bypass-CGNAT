# Auf dem VPS:

```
apt-get update && apt-get upgrade -y
```
```
curl -L https://install.pivpn.io | bash
```
name des interfaces herausfinden
```
ip a
```
wg0.conf bearbeiten:
```
nano /etc/wireguard/wg0.conf
```
Direkt nach "ListenPort" folgende Zeile hinzufügen (bei "dein_interface" den Namen des Interfaces eintragen):
```
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o dein_interface -j MASQUERADE
```
Bei AllowedIPs den Adressbereich des Heimnetzwerks hinzufügen
```	
Address = 10.70.193.1/24, 192.168.8.0/24
```
reboot
```
pivpn -a
```
Name für das Profil eingeben, z. B.: heimnetzwerk
```
nano /etc/wireguard/wg0.conf
```
Bei "AllowedIPs" Heimnetzwerk IP-Bereich hinzufügen
```
AllowedIPs = 10.70.193.2/32, 192.168.8.0/24
```
Folgendes temporär im Texteditor speichern:
```
cat /etc/wireguard/configs/heimnetzwerk.conf
```
sysctl.conf Bearbeiten
```
nano /etc/sysctl.conf
```
#net.ipv4.ip_forward = 1
zu
net.ipv4.ip_forward = 1

```
nano /ip.sh
```
Folgendes eintragen:
```
#!/bin/bash
iptables -t nat -A POSTROUTING -j MASQUERADE
```	
um alle Ports an 192.168.8.14 weiterleiten (in dem fall Nginx Proxy Manager)
```
iptables -A PREROUTING -t nat -p tcp --match multiport ! --dports 22,51820 -j DNAT --to-destination 192.168.8.14
iptables -A PREROUTING -t nat -p udp --match multiport ! --dports 22,51820 -j DNAT --to-destination 192.168.8.14
```
um einzelnen port (80) an 192.168.8.14 weiterleiten (in dem fall Webserver)
```
iptables -t nat -A PREROUTING -p "tcp" --dport "80" -j DNAT --to-destination "192.168.8.14":"80"
```
Rechte anpassen:
```
chmod +x /ip.sh
```
Crontab öffnen:
```
crontab -e
```
Dort folgendes eintragen:
```
@reboot /ip.sh
```
Reboot. Auf dem VPS Ferig.
# Auf der VM:
## Docker:
## Andere Systeme:
Docker installieren
```
curl -fsSL https://get.docker.com -o get-docker.sh && chmod +x get-docker.sh && sh get-docker.sh
```
Link zum Docker Repo:
```
https://github.com/NightMeer/WireguardClient-Docker  
```
Docker run ausführen:
```
docker run --restart unless-stopped --privileged -d -v /host/path/config:/config ghcr.io/nightmeer/wireguardclient:latest
```
[wg0.conf](https://raw.githubusercontent.com/NightMeer/WireguardClient-Docker/main/wg0.conf) anpassen und in dem Config Ordner ablegen
Docker neustarten. Fertig.
## Unraid:

Nach nightmeer/wireguardclient Suchen und installieren
```
nightmeer/wireguardclient
```
In "/mnt/user/appdata/wireguardclient/" die [wg0.conf](https://raw.githubusercontent.com/NightMeer/WireguardClient-Docker/main/wg0.conf) anpassen und  ablegen.
```
wget -O /mnt/user/appdata/wireguardclient/wg0.conf https://raw.githubusercontent.com/NightMeer/WireguardClient-Docker/main/wg0.conf
```
Docker neustarten. Fertig.
## Installieren:
```
apt-get update && apt-get upgrade -y && apt-get install -y wireguard openresolv wireguard-dkms
```
```
nano /etc/wireguard/wg0.conf
```
wg0.conf bearbeiten und Inhalt aus /etc/wireguard/configs/heimnetzwerk.conf eintragen

```
nano /etc/wireguard/wg0.conf
```
Vor [Peer] eintragen und Interface anpassen (bei "dein_interface"):
```
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -o dein_interface -j MASQUERADE
PostUp = echo 1 > /proc/sys/net/ipv4/ip_forward
```
```
nano /etc/wireguard/wg0.conf
```
Am Ende folgende Zeile einfügen:
```
PersistentKeepalive = 25
```
Wireguard aktivieren
```
systemctl enable wg-quick@wg0
```
Reboot. Fertig.
