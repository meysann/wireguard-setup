# WireGuard VPN Setup Guide

This guide provides all the steps needed to configure a personal WireGuard VPN server on a Debian-based Linux VPS (e.g., Ubuntu) and connect a client device.

---

## Part 1: Server Setup (on the VPS)

### 1. Install WireGuard
Connect to your VPS via SSH and install the necessary packages.
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install wireguard -y
----------------------------------------------------------------------------------------------------------------------------------------------------------
### 2.Generate Server Keys
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod 600 /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
----------------------------------------------------------------------------------------------------------------------------------------------------------
### 3.Create the Server Configuration
sudo vi /etc/wireguard/wg0.conf
---
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = YOUR_SERVER_PRIVATE_KEY
PostUp = ufw route allow in on wg0 out on ens5                           #change interface name
PostUp = iptables -t nat -I POSTROUTING -o ens5 -j MASQUERADE
PreDown = ufw route delete allow in on wg0 out on ens5
PreDown = iptables -t nat -D POSTROUTING -o ens5 -j MASQUERADE
---
----------------------------------------------------------------------------------------------------------------------------------------------------------
### 4. Enable IP Forwarding
sudo vi /etc/sysctl.conf
---
Uncomment this line (remove the #):

net.ipv4.ip_forward=1
---
apply the change:

sudo sysctl -p
----------------------------------------------------------------------------------------------------------------------------------------------------------
### 5. Configure Firewalls (Crucial Step)

A) Server Firewall (ufw):

sudo ufw allow 51820/udp
sudo ufw allow OpenSSH
sudo ufw enable

B) Cloud Provider Firewall:

Log in to your VPS provider's website (AWS, Google Cloud, etc.). Find the networking settings (e.g., "Security Groups") and add a new inbound rule to allow UDP traffic on port 51820 from any source (0.0.0.0/0).

----------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------
## Part 2: Client Setup (for Your Device)

### 1. Generate Client Keys

wg genkey | sudo tee /etc/wireguard/client_private.key | wg pubkey | sudo tee /etc/wireguard/client_public.key
----------------------------------------------------------------------------------------------------------------------------------------------------------
### 2. Add Client Peer to Server
Add the following [Peer] section to the bottom of the file
sudo vi /etc/wireguard/wg0.conf

---
[Peer]
# A name for your client, e.g., Android Phone
PublicKey = PASTE_CLIENT_PUBLIC_KEY_HERE
AllowedIPs = 10.0.0.2/32
---
----------------------------------------------------------------------------------------------------------------------------------------------------------
### 3. Start the Server

sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0

----------------------------------------------------------------------------------------------------------------------------------------------------------
### 4. Create Client Configuration & QR Code

sudo vi /etc/wireguard/client.conf
---
[Interface]
PrivateKey = PASTE_CLIENT_PRIVATE_KEY_HERE
Address = 10.0.0.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = PASTE_SERVER_PUBLIC_KEY_HERE
Endpoint = YOUR_SERVER_IP:51820
AllowedIPs = 0.0.0.0/0
---
Install qrencode and display the QR code.
sudo apt install qrencode -y
qrencode -t ansiutf8 < /etc/wireguard/client.conf
----------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------
**## Part 3: Troubleshooting**

1. Check Service Status
sudo wg show
If it shows nothing: The service is down. Start it with:
sudo wg-quick up wg0.
----------------------------------------------------------------------------------------------------------------------------------------------------------
2.Re-Check Firewalls

Confirm sudo ufw status shows 51820/udp ALLOW.

Confirm you have created the firewall rule on your cloud provider's website.
----------------------------------------------------------------------------------------------------------------------------------------------------------
3.Verify Keys and IPs :
Ensure the public/private keys match between the client config and server [Peer] section, and that the Endpoint IP in the client config is correct.
----------------------------------------------------------------------------------------------------------------------------------------------------------




