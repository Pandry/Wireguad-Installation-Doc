# Installing wireguard and creating a VPN with WireGuard
# This 


### Installing the source files (& git)

x86 distro: 
```
apt install libmnl-dev libelf-dev linux-headers-$(uname -r) build-essential pkg-config git
```
Raspberry Pi:
```
apt install libmnl-dev libelf-dev raspberrypi-kernel-headers build-essential pkg-config git
```

### Clone the repo from the mirror hosted on github:
```
git clone https://github.com/WireGuard/WireGuard
```

### cd in the source files directory, compile and install the Kernel Module
```
cd WireGuard/src 
# Compile the module
make
# Install the module
sudo make install
```
### Done! WireGuard is installed!

## Set up the server

### Enable the port forwarding
```
echo "1" > /proc/sys/net/ipv4/ip_forward
```

### Add the wireguard interface
```
ip link add dev wg0 type wireguard
```

### Set the IP range that the interface will use
```
ip address add dev wg0 172.16.0.0/12
```

### Generate a private key and copy it
```
wg genkey
```

### Create the default configuration files
```
touch /etc/wireguard/wg0.conf && nano /etc/wireguard/wg0.conf
```

### then input the sample configuration file

```
[Interface] 
PrivateKey = <The private key previously copied>
ListenPort = <The listening port of your server, 53 is the standard DNS port, useful to bypass a firewall>

[Peer] 
PublicKey = <peer public key>
#Comma separated IPs
AllowedIPs = 0.0.0.0/0, 127.0.0.1/8
```

### Apply the configuration
```
wg setconf wg0 /etc/wireguard/wg0.conf
```

### Set the link up
```
ip link set up dev wg0
```
### The server is now up and running
## Let's set up a nat

```
/sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
/sbin/iptables -A FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
/sbin/iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT
```


Public key from the private one
```
wg pubkey < privatekey > publickey
```



Creating a config file for a peer

```

[Interface]
PrivateKey = <A newly generated public key>
Address = 172.16.0.0/12
DNS = 1.1.1.1

[Peer]
PublicKey = <The server public IP>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = <Your server IP address>
```
