title: docker使用自定义网桥
comment: true
toc: true
share: true
date: 2015-04-24 14:19:26
tags: [linux, docker]
---
参考网址：https://docs.docker.com/articles/networking/
``` sh
$ sudo service docker stop
$ sudo ip link set dev docker0 down
$ sudo brctl delbr docker0
$ sudo iptables -t nat -F POSTROUTING
```

<!-- more -->

``` sh
# Create our own bridge

$ sudo brctl addbr docker-bridge
$ sudo ip addr add 192.168.5.1/24 dev docker-bridge
$ sudo ip link set dev docker-bridge up

# Confirming that our bridge is up and running

$ ip addr show docker-bridge
4: docker-bridge: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state UP group default
    link/ether 66:38:d0:0d:76:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.1/24 scope global docker-bridge
       valid_lft forever preferred_lft forever

# Tell Docker about it and restart (on Ubuntu)

$ echo 'DOCKER_OPTS="-b=docker-bridge"' >> /etc/default/docker
$ sudo service docker start

# Confirming new outgoing NAT masquerade is set up

$ sudo iptables -t nat -L -n
```