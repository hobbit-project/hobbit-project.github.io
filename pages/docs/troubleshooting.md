---
title: Troubleshooting
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: troubleshooting.html
folder: docs
---

## Firewall adjustments (Linux)

The serverbackend of the hobbitgui container needs access to keycloak via the external IP address. Therefore you may need to adapt the firewall rules. First find the IP range of the hobbit network:
```bash
docker network inspect hobbit | grep Gateway
```

Assuming you get something like "Gateway": "172.19.0.1" you have to find the matching network device:
```bash
ip addr | grep -B 2 172.19.0
```

If you get something like
```bash
6: br-5c9d73b080ad: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:22:50:d4:8c brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.1/16 scope global br-5c9d73b080ad
```
the network device name is `br-5c9d73b080ad`. If you have iptables as firewall use:
```bash
iptables -A INPUT -i br-5c9d73b080ad -j ACCEPT
```

If you have firewalld (fedora/centos7) use:
```bash
firewall-cmd --permanent --zone=trusted --change-interface=br-5c9d73b080ad
firewall-cmd --reload
```
