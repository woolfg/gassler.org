---
title: "Secure your Docker setup by overwriting Docker's (Swarm) iptables rules"
date: "2020-05-25"
slug: overwrite-iptables-rules-docker-swarm
tags: [stackoverflowish,docker]
hero: "julian-berengar-solter-URtfHnJza0E-unsplash.jpg"
summary: "Simple if you know how to do it: Preventing Docker (Swarm) from opening ports to the outer world on Debian by overwriting the Docker network magic using iptables and nftables."
credits: ["Hero picture by https://unsplash.com/@moinundmeer"]
---

Have you ever tried to overwrite Docker’s iptables network magic, e.g. preventing Docker Swarm from opening arbitrary ports on the ingress overlay network which would allow containers directly to publish ports to the other world? Apparently it is not that easy and I also couldn’t find a lot of material about it online, therefore, I will quickly summarize my findings here in this “stackoverflowish” blog post.

## My Docker setup

I am using Debian 10, Docker Swarm, and its routing mesh (ingress overlay) which allows to connect the outer world with docker containers. The same problem also occurs when just using the docker daemon without swarm.
I had a simple goal and wanted to limit the access from outside to the port 80 and 443 and prevent any containers from opening other ports to the outer world. But I had to face several issues...

## Debian nftables and Docker

Debian 10 Buster activates `nftables` ---the successor of `iptables`--- by default which introduces some issues running Docker (Swarm) as it is heavily based on `iptables` and creates a lot of rules for all the network magic that is done by Docker. Luckily, Debian offers the `itpables-nft` layer which allows to use iptables syntax with the `nf_tables` kernel subsystem.

## DOCKER-USER iptables chain

Docker puts most rules in the `FORWARD` chain and furthermore, puts itself in front of all other rules that you might have created. Luckily, Docker creates a chain called `DOCKER-USER` which is evaluated before Docker specific rules get evaluated ([see Docker documentation](https://docs.docker.com/network/iptables/)). Therefore, you have to put your rules into this chain. 

```
# iptables -N DOCKER-USER
# iptables -I DOCKER-USER 1 -i eth0 -p tcp -m state --state NEW -m multiport ! --dports 443,80 -j DROP;

# iptables -L
Chain DOCKER-USER (1 references)
target     prot opt source               destination         
DROP       tcp  --  anywhere             anywhere             state NEW multiport dports  !https,http
RETURN     all  --  anywhere             anywhere            
```

The rule above which is evaluated at the beginning, allows new connections on port 80 and 443, and drops all other packages to initiate a new connection.

## Persisting the rules

It is important that you do NOT activate `nftables` by e.g. `systemctl enable nftables.service` and use rules defined in `/etc/nftables.conf`. It breaks the Docker network magic as Docker can no longer use the `iptables` system to add rules to the  `nf_tables` kernel subsystem. Therefore, stick to the old `iptables` system.

I created the following script which adds the `iptables` rules and allows ports 80 and 443 on the ingress network and 22 (ssh) on the `eth0` interface.

```
#/bin/sh
# allow ssh only on the public eth0
iptables -A INPUT -i eth0 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT

# add rule to chain which is later used by docker
# prevents that any docker swarm services can open ports (exception 80,443 for ingress router)

iptables -N DOCKER-USER
iptables -I DOCKER-USER 1 -i eth0 -p tcp -m state --state NEW -m multiport ! --dports 443,80 -j DROP;
```

To be automatically added at startup I created the following service definition file in `/etc/systemd/system/hcloud-iptables.service` which executed the script mentioned above. 

```
[Unit]
Description=installs iptable rules to secure eth0 and docker ingress network
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/sh /opt/hcloud-iptables.sh
RemainAfterExit=true
StandardOutput=journal

[Install]
WantedBy=multi-user.target
```

I also highly recommend to use a configuration management tool, like ansible or chef, to deploy this service and the iptables script to automate the setup of a new docker swarm node.
