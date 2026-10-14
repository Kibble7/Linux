# 11. OpenSSH & Networking Diagnostics (Supplemental)

*(Referenced from Barrett, Daniel J. *Linux Pocket Guide*, 4th ed., O'Reilly Media)*

## 1. Remote Administration with OpenSSH

```bash
ssh smith@remote.example.com              # Connect to a remote shell
ssh -p 2222 smith@remote.example.com      # Connect on a non-standard port
ssh smith@remote.example.com "df -hT"     # Run a remote command non-interactively
ssh -v smith@remote.example.com           # Verbose debug mode (-vvv for more)
ssh -i ~/.ssh/mykey smith@remote.example.com   # Use a specific private key
ssh -X smith@remote.example.com           # Forward X11 (GUI apps)
```

### SSH Key-Pair Authentication Setup

1. Generate a key pair on the local system:

```bash
ssh-keygen -t ed25519
# or: ssh-keygen -t rsa -b 4096
```

2. Copy the public key to the remote host:

```bash
ssh-copy-id smith@remote.example.com
```

3. Verify permissions (SSH ignores keys if these are too open):

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519       # or id_rsa
chmod 644 ~/.ssh/id_ed25519.pub   # or id_rsa.pub
```

### ➕ Additional Essentials: SSH Config, Agent & Tunnels

```bash
# ~/.ssh/config  (shortcut definitions)
#   Host web
#       HostName remote.example.com
#       User smith
#       Port 2222
#       IdentityFile ~/.ssh/id_ed25519
# Then simply:  ssh web

eval "$(ssh-agent -s)"                    # Start the agent
ssh-add ~/.ssh/id_ed25519                 # Load a key (enter passphrase once)
ssh-add -l                                # List loaded keys

ssh -L 8080:localhost:80 smith@remote.example.com   # Local port forward
ssh -R 9090:localhost:3000 smith@remote.example.com # Remote port forward
ssh -D 1080 smith@remote.example.com                # SOCKS proxy
ssh -J jump.example.com smith@internal.example.com  # Jump host

# Server side
sudo systemctl status sshd                # Service status (named 'ssh' on Debian/Ubuntu)
sudo vi /etc/ssh/sshd_config              # Server config: Port, PermitRootLogin, PasswordAuthentication
sudo sshd -t                              # Test config syntax
sudo systemctl reload sshd                # Apply changes
```

### Secure File Transfer (`scp`, `sftp`)

```bash
# scp (batch copy)
scp myfile smith@remote.example.com:/tmp/            # Push a local file to remote
scp -r smith@remote.example.com:/var/log/ ./logs/    # Pull a remote directory recursively
scp -P 2222 myfile smith@remote.example.com:/tmp/    # Non-standard port (capital -P)

# sftp (interactive transfer)
sftp smith@remote.example.com
# sftp> get filename        # Download a file
# sftp> put filename        # Upload a file
# sftp> ls / cd / pwd       # Remote navigation
# sftp> lls / lcd / lpwd    # Local navigation
# sftp> bye                 # Quit
```

---

## 2. Network Interfaces & Routing (`ip`)

```bash
ip addr show                              # IP addresses on all interfaces (ip a)
ip -br addr                               # Brief, one-line-per-interface view
ip link show                              # Link-layer state
sudo ip link set eth0 up                  # Enable an interface
sudo ip link set eth0 down                # Disable an interface
ip route show                             # Kernel routing table
sudo ip addr add 192.168.1.50/24 dev eth0 # Add an address (temporary)
sudo ip route add default via 192.168.1.1   # Add a default gateway (temporary)
ip neigh                                  # ARP/neighbor table
```

### ➕ Additional Essentials: Hostname, DNS & Persistent Config

```bash
hostname -I                               # All IPs of this host
hostnamectl set-hostname server1          # Set the hostname
cat /etc/resolv.conf                      # DNS resolver settings
cat /etc/hosts                            # Local name-to-IP mappings
resolvectl status                         # systemd-resolved status

nmcli device status                       # NetworkManager: device overview
nmcli connection show                     # NetworkManager: saved connections
sudo nmcli connection up "Wired connection 1"
nmtui                                     # Text UI for NetworkManager
# Config files: /etc/netplan/*.yaml (Ubuntu), /etc/NetworkManager/ (RHEL/Fedora)
sudo netplan apply                        # Apply Netplan config (Ubuntu)
```

---

## 3. Network Diagnostics & Reachability

```bash
ping -c 4 8.8.8.8                         # Test connectivity with 4 ICMP packets
traceroute -n archlinux.org               # Router hops without DNS lookups
mtr archlinux.org                         # Live combined ping + traceroute
host www.ubuntu.org                       # DNS lookup
host -t MX centos.org                     # Query Mail Exchanger records
whois linuxmint.com                       # Domain registration details
dig example.com                           # Detailed DNS query
dig +short example.com                    # Only the answer
dig -x 8.8.8.8                            # Reverse lookup
nslookup example.com                      # Alternative DNS lookup

# Netcat (raw sockets)
netcat -zv localhost 22                   # Test whether a TCP port is open (no data sent)
netcat -zv host 20-100                    # Scan a port range
netcat -l 5555                            # Listen on port 5555 (some versions need -lp)
```

### ➕ Additional Essentials: Sockets, Ports & Firewall

```bash
ss -tuln                                  # Listening TCP/UDP sockets (numeric)
ss -tulnp                                 # Include owning process (sudo)
ss -s                                     # Socket summary
netstat -tuln                             # Older equivalent of ss
sudo lsof -i :22                          # Who is using port 22

# Firewall: pick the front-end your distro uses
sudo ufw status                           # Ubuntu
sudo ufw allow 22/tcp
sudo ufw enable
sudo firewall-cmd --list-all              # RHEL/Fedora
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
sudo iptables -L -n -v                    # Low-level rules view
sudo nft list ruleset                     # nftables rules

# Packet capture
sudo tcpdump -i eth0 port 80              # Capture HTTP traffic
sudo tcpdump -i any -n -c 20              # First 20 packets, no name resolution
```

---

## 4. Web Data Retrieval (`curl`, `wget`)

```bash
# Curl
curl -I https://example.com                       # Fetch HTTP response headers only
curl -o file.html https://example.com             # Save output to a specific file
curl -O https://example.com/archive.zip           # Save using the remote filename
curl -s https://api.ipify.org                     # Silent retrieval of the raw payload
curl -L https://example.com                       # Follow redirects
curl -X POST -d 'name=bob' https://example.com/api            # POST form data
curl -H 'Content-Type: application/json' -d '{"a":1}' https://example.com/api   # POST JSON

# Wget
wget https://example.com/archive.zip              # Download a file
wget -c https://example.com/big.iso               # Resume an interrupted download
wget -r -l 2 https://example.com/                 # Recursively mirror to depth 2
wget -O out.html https://example.com              # Save under a chosen name
wget -q --spider https://example.com && echo up   # Check a URL exists
```
