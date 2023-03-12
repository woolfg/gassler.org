---
title: "How I brought down my whole Docker Swarm cluster by adding a simple iptables rule"
date: "2023-03-12"
slug: overwrite-iptables-rules-docker-swarm
tags: [stackoverflowish,docker,ansible]
hero: "julian-berengar-solter-URtfHnJza0E-unsplash.jpg"
summary: ""
credits: ["Hero picture by [@moinundmeer](https://unsplash.com/@moinundmeer)"]
---

This is a tale of adding one iptables rule to ansible and a lot of resulting cascading failures.
I also added all log and error messages in case you encounter similar issues. Spoiler: The main issue
was an upgrade of docker to version 23.0.1 on Debian 10 buster that was done automatically by the my
ansible playbook.

## The Goal

I just wanted to persist a manually added iptables rule on my worker nodes by adding it to my ansible playbook.
I had tested the iptables rule, added it to the playbook using the iptables plugin. My usual testing procedure
is to tag the task and run just this one task on one worker node (you can use `ansible-playbook --limit ` to do this).



## Error messages I have seen during the outage

```(bash)
The log was full of such messages as docker tried to bring up needed containers in a somedomainMar 12 02:50:52 worker2.somedomain systemd-udevd[31592]: link_config: could not get ethtool features for vx-00
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

```(bash)
docker: Error response from daemon: failed to create shim task:
OCI runtime create failed: runc create failed: unable to start container process:
error during container init: unable to apply apparmor profile:
apparmor failed to apply profile: write /proc/self/attr/apparmor/exec: no such file or directory: unknown.
```


What was helpful

What can be improved