---
title: "How I brought down my whole Docker Swarm cluster"
date: "2023-03-12"
slug: docker-swarm-incident-pin-your-versions
tags: [stackoverflowish,docker,ansible]
hero: "bradyn-trollip-pxVOztBa6mY-unsplash.jpg"
summary: "This is a tale of adding one iptables rule to ansible, a sleepless night, and a lot of resulting cascading failures.
It is quite unusual to write a post mortem for an incident on a hobby cluster, but maybe it helps someone to avoid
the same mistakes. I also added all log and error messages in case you encounter similar issues."
credits: ["Hero picture by [@bradyn](https://unsplash.com/@bradyn)"]
---

## Post mortem

This is a tale of adding one iptables rule to ansible, a sleepless night, and a lot of resulting cascading failures.
It is quite unusual to write a post mortem for an incident on a hobby cluster, but maybe it helps someone to avoid
the same mistakes. I also added all log and error messages in case you encounter similar issues.

*Spoiler: The main issue
was an upgrade of docker to version 23.0.1 on Debian 10 buster that was done automatically by my
ansible playbook.*

## The simple goal

I just wanted to persist a manually added iptables rule on my side-project docker swarm cluster by adding it to my ansible playbook.
I had tested the iptables rule, added it to the playbook using the iptables plugin. My usual testing procedure
is to tag the task and run just this one task on one worker node (you can use `ansible-playbook --limit ` to do this).

## The timeline

### The ansible run

After my tests worked, I ran the whole playbook on one worker node. The playbook run failed as an important information
was missing. This data is gathered from the manager node and wasn't fetched as the playbook was just executed on one worker.
Therefore, I ran the playbook without any limitation on all nodes. The run was successful.

### Incident started

In the meanwhile I got an alert, that a service is no longer reachable. I checked the container which was no longer running. Instead
of running containers I found tons of log error messages. I immediately started to investigate the issue and saw that there
wasn't a single running container on any of my worker nodes anymore.

### Idea 1: iptables

The log messages were related to some links and network stuff, so I was quite sure that my iptables were fucked up.
The log was full of such messages as docker tried to bring up needed containers in an endless loop:

```(bash)
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31592]: link_config: could not get ethtool features for vx-00
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31592]: Could not set offload features of vx-001010-osg3l: No
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31592]: Could not generate persistent MAC address for vx-0010
Mar 12 02:50:52 worker2.somedomain dockerd[741]: time="2023-03-12T02:50:52.747416271+01:00" level=error msg="N
Mar 12 02:50:52 worker2.somedomain dockerd[741]: time="2023-03-12T02:50:52.750171275+01:00" level=error msg="p
Mar 12 02:50:52 worker2.somedomain dockerd[741]: time="2023-03-12T02:50:52.750997547+01:00" level=error msg="f
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31592]: Process '/sbin/ip link set  down' failed with exit co
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31567]: link_config: autonegotiation is unset or enabled, the
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31567]: Could not generate persistent MAC address for vethb3e
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31592]: link_config: autonegotiation is unset or enabled, the
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31592]: Could not generate persistent MAC address for veth608
Mar 12 02:50:52 worker2.somedomain kernel: veth0: renamed from vethb3e3b63
Mar 12 02:50:52 worker2.somedomain kernel: br0: port 2(veth0) entered blocking state
Mar 12 02:50:52 worker2.somedomain kernel: br0: port 2(veth0) entered disabled state
Mar 12 02:50:52 worker2.somedomain kernel: device veth0 entered promiscuous mode
Mar 12 02:50:52 worker2.somedomain kernel: br0: port 2(veth0) entered blocking state
Mar 12 02:50:52 worker2.somedomain kernel: br0: port 2(veth0) entered forwarding state
Mar 12 02:50:52 worker2.somedomain systemd-udevd[31592]: Process '/sbin/ip link set  down' failed with exit co
Mar 12 02:50:53 worker2.somedomain dockerd[741]: time="2023-03-12T02:50:53.042439971+01:00" level=warning msg=
Mar 12 02:50:53 worker2.somedomain kernel: eth0: renamed from veth6084f8a
Mar 12 02:50:53 worker2.somedomain systemd-udevd[31592]: Process '/sbin/ip link set  down' failed somedomain
```

I checked all iptables, flushed rules, restarted docker, but nothing helped. I could access all nodes and the network worked.
Also googling the error messages didn't help. I saw that the docker hosts tried to start all containers but failed somehow.

### Idea 2: wipe a node

Our fully automated ansible script is able to build up new docker swarm nodes in minutes. So I made use of it, rolled the ansible
playbook back to an older version, wiped the node and started the playbook again. I started the hello world container and encountered
the following error message:

```(bash)
# docker run hello-world
docker: Error response from daemon: failed to create shim task:
OCI runtime create failed: runc create failed: unable to start container process:
error during container init: unable to apply apparmor profile:
apparmor failed to apply profile: write /proc/self/attr/apparmor/exec: no such file or directory: unknown.
```

It was difficult to understand that a fresh installed node can't spin up a hello world container. Finally, I found an information
that the package `apparmor` should be installed. Still, I didn't understand why this was necessary as I didn't install it in the past.
Maybe it was because it was middle of the night, but I didn't restart the docker daemon after installing the package.

### The solution

Finally, I found a website that states that the new docker version 23.0.1 requires AppArmor. This finally pulled the trigger and I
checked the docker version. I installed apparmor and restarted the docker daemon. After that, the hello world container was finally running.

Unfortunately, after restarting all my stacks manually, I still couldn't see any running containers and the log was full of error
messages as shown above (link, network, etc). I took me a while until I found the problem. As I started the stacks manually I forgot
to add the flag `--with-registry-auth`. This flag is necessary to pull images from a private registry.  Finally, everything was
recovering and I could go to bed.

## The root cause

Our ansible playbook has a section to install the docker package using apt.

```(yaml)
- name: install latest docker
  become: true
  apt:
    name: ["docker-ce", "docker-ce-cli", "containerd.io"]
    state: latest
```

Then running the playbook, the `latest` flag triggered an upgrade of the docker version and the new version 23.0.1 was installed. This version requires the package `apparmor` which wasn't there. This was the inital reason why docker couldn't start any containers anymore.

Using `latest` is a quite bad idea! We should have pinned all versions, especially in production environments.
In this case just use `state: present` to not upgrade the package and everything is fine.

## Conclusion: What I have learned

- The automation to rebuild a node by ansible was super helpful. I was able to rebuild the node in minutes and was able to start
from scratch to rule out any other problems.
- When docker shows network related error messages, it might be just the case that it can't pull the image from a private registry or in general, starts a container. It might not be related to the network at all.
- Avoid manual changes or commands, as e.g. starting stacks and forgetting the necessary flag `--with-registry-auth`.
- Even if you just changed something, the problem might be related to something else. So, try to narrow down the problem by also checking unrelated things.
- Never ever use automated upgrades in production environments!
- It is super difficult to find the root cause by googling, even if you have very specific error messages.
- Another pair of eyes might have helped a lot, but it was middle of the night and it is "just" a cluster for side-projects.
