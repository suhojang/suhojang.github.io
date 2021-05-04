---
title: "docker networking"
layout: post
author: jsh
tags: docker
cover: "/assets/cover.jpg"
categories: docker
---

#### 도커를 설치하게 되면 생기는일

Docker를 설치한 후 Host의 네트워크 인터페이스를 살펴보면 docker0라는 가상 인터페이스가 생긴다.   

docker0는 일반적인 가상 인터페이스가 아니며 도커가 자체적으로 제공하는 네트워크 드라이버 중 브리지(Bridge)에 해당한다.   

도커에서 사용할 수 있는 네트워크 종류는 브리지(bridge), 호스트(host), 논(none) 등이 있다.

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
50a7264f8a6a        bridge              bridge              local
559f912edceb        host                host                local
3af7882fd839        none                null                local
```

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
        ether 02:42:b0:c8:62:94  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

docker0 브리지는 컨테이너가 통신하기 위해 사용된다. 도커 컨테이너를 생성하면 자동으로 이 브리지를 활용하도록 설정되어 있다.   

docker0 인터페이스는 172.17.0.0/16 서브넷을 갖기 때문에 컨테이너가 생성되면 이 대역 안에서 IP를 할당받게 된다. (예: 172.17.0.2, 172.17.0.3)

$ docker network inspect bridge 명령어를 이용하면 브리지 네트워크의 자세한 정보를 알 수 있다.

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "50a7264f8a6acba26767b4fef7afaa60ca0ad8ddae21c39274b1ee561340d377",
        "Created": "2021-04-28T17:48:11.185137606+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

#### 도커 컨테이너를 생성하게 되면 생기는 일

컨테이너는 Linux Namespace 기술을 이용해 각자 격리된 네트워크 공간을 할당받게 된다.   
그리고 위에서 언급한 대로 172.17.0.0/16 대역의 IP를 순차적으로 할당 받는다. 이 IP는 컨테이너가 재시작할 때마다 변경될 수 있다.   

컨테이너는 외부와 통신하기 위해 2개의 네트워크 인터페이스를 함께 생성한다.   
하나는 컨테이너 내부 Namespace에 할당되는eth0 이름의 인터페이스이고, 나머지 하나는 호스트 네트워크 브리지 docker0에 바인딩 되는 vethXXXXXXX이름 형식의 veth 인터페이스다. (“veth”는 “virtual eth”라는 의미)   
컨테이너의 eth0인터페이스와 호스트의 veth 인터페이스는 서로 연결되어 있다.   

결국 docker0 브리지는 veth 가상 인터페이스와 호스트의 eth0 인터페이스를 이어주는 중간 다리 역할을 한다. 그리고 컨테이너 안에 eth0인터페이스는 veth 가상 인터페이스를 통해 외부와 통신할 수 있게 되는 것이다.

![/assets/docker_network.png](/assets/docker_network.png)


직접 확인해보기 위해 간단한 도커 컨테이너 2개를 실행시켜본다.   
busybox이미지를 이용해서 5분간 Sleep 상태를 유지 하도록 했다.

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker run -d --rm --name busybox1 busybox sleep 300
Unable to find image 'busybox:latest' locally
Trying to pull repository docker.io/library/busybox ...
latest: Pulling from docker.io/library/busybox
aa2a8d90b84c: Pull complete
Digest: sha256:345c29321fecc39de30aa44d74e26d838e8bf0405aeafa89701212883228cd34
Status: Downloaded newer image for docker.io/busybox:latest
0a9d1f1ca9b7254e2173fd45a9330159e64de8c8011660268fe1f48f33779403
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker run -d --rm --name busybox2 busybox sleep 300
c455642186392f286308877c7db8d0532d4fd208a514ae1f1a4356be93606738
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c45564218639        busybox             "sleep 300"         4 seconds ago       Up 2 seconds                            busybox2
0a9d1f1ca9b7        busybox             "sleep 300"         7 seconds ago       Up 5 seconds                            busybox1
```

먼저 컨테이너 내부 네트워크 인터페이스를 확인해보자.      
eth0 인터페이스에 각각 172.17.0.2, 172.17.0.3 IP가 할당 되었다.

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker exec -it busybox1 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
332: eth0@if333: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link
       valid_lft forever preferred_lft forever
```

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker exec -it busybox2 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
334: eth0@if335: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:3/64 scope link
       valid_lft forever preferred_lft forever
```

이제 호스트의 네트워크 인터페이스를 보면 vetha608b17, veth3a85f80 가상 인터페이스가 새로 생성된 것을 볼 수 있다.

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:91:35:d8 brd ff:ff:ff:ff:ff:ff
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:b0:c8:62:94 brd ff:ff:ff:ff:ff:ff
333: vetha608b17@if332: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether fa:aa:77:8a:f0:2a brd ff:ff:ff:ff:ff:ff link-netnsid 0
335: veth3a85f80@if334: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 76:b9:26:c6:20:0c brd ff:ff:ff:ff:ff:ff link-netnsid 1
```

brctl 명령어를 통해 브리지 네트워크 상태를 확인해보면 vetha608b17, veth3a85f80 인터페이스가 docker0 브리지에 연결된 것을 확인할 수 있다.

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242b0c86294       no              veth3a85f80
                                                        vetha608b17
```

두 컨테이너(busybox1, busybox2)는 브리지를 통해 같은 네트워크상에 있기 때문에 한쪽 컨테이너에서 다른 컨테이너와 통신할 수 있게 된다.

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker exec -it busybox2 ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.672 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.331 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.297 ms
64 bytes from 172.17.0.2: seq=3 ttl=64 time=0.267 ms
64 bytes from 172.17.0.2: seq=4 ttl=64 time=0.326 ms
64 bytes from 172.17.0.2: seq=5 ttl=64 time=0.264 ms
64 bytes from 172.17.0.2: seq=6 ttl=64 time=0.310 ms
```

마지막으로 컨터이너의 게이트웨이를 확인 해보면 172.17.0.1된 것을 볼 수 있는데 결국 컨테이너 내부의 모든 패킷이 호스트의 docker0을 통해 외부로 나가게 되는 것을 확인할 수 있다.

```shell
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker exec -it busybox1 route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      *               255.255.0.0     U     0      0        0 eth0
```
