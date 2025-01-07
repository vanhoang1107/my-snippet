# Add base iptables rules
## IPv4
```sh
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT

iptables -P INPUT DROP
```

## IPv6
```sh
ip6tables -A INPUT -i lo -j ACCEPT
ip6tables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

ip6tables -P INPUT DROP
```

# Install Docker
## Packages
Add Docker's official GPG key
```sh
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the repository to Apt sources
```sh
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## iptables
```sh
iptables -I DOCKER-USER -i eth0 -j DROP
iptables -I DOCKER-USER 1 -m state --state RELATED,ESTABLISHED -j RETURN

iptables -A INPUT -s 172.17.0.0/24 -i eth1 -p tcp -m tcp --dport 7946 -m comment --comment "Docker Swarm" -j ACCEPT
iptables -A INPUT -s 172.17.0.0/24 -i eth1 -p udp -m udp --dport 7946 -m comment --comment "Docker Swarm" -j ACCEPT
iptables -A INPUT -s 172.17.0.0/24 -i eth1 -p udp -m udp --dport 4789 -m comment --comment "Docker Swarm" -j ACCEPT
```

## Update config
```vim
vim /etc/docker/daemon.json
```

Content
```json
{
    "live-restore": false,
    "insecure-registries" : ["registry.docker.local:5000"],
    "dns": ["172.17.0.69", "8.8.8.8"],
    "bip": "172.19.0.0/12",
    "metrics-addr" : "0.0.0.0:9323",
    "experimental" : true,
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "3"
    }
}
```

# Install Chrony to sync time
```
apt install chrony
```

# Setup Logrotate
```sh
vim /etc/logrotate.d/app
```

Content
```
/var/log/hunny/*.log
/var/log/hunny/*/*.log
/var/log/inhouse-game/*.log
/var/log/inhouse-game/*/*.log
/var/log/tele-mini-app/*.log
/var/log/tele-mini-app/*/*.log
/var/log/investor-portal/*.log
/var/log/investor-portal/*/*.log
{
    daily
    rotate 30
    missingok
    dateext
    dateyesterday
    dateformat .%Y-%m-%d
    copytruncate
    compress
    delaycompress
}
```

# Setup DNS
```sh
vim /etc/netplan/90-hunny-node.yaml
```

Content
```yaml
network:
  version: 2
  ethernets:
    eth0:
      nameservers:
        addresses:
          - 172.17.0.69  # CoreDNS
    eth1:
      nameservers:
        addresses:
          - 172.17.0.69  # CoreDNS
```

Then apply the changes
```sh
netplan apply
```
