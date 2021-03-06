### Linux NIC 확인

<hr>

**현재 사용중인 랜카드 목록 확인**

```
> ip link

1: lo: ...
2: eth0: ...
3: eth1: ...
4: bond0: ...
```



**랜카드 지원속도 확인**

```
> ethtool eth0


Settings for eth0:
        Supported ports: [ TP ]
        Supported link modes:   10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Supports auto-negotiation: Yes
        Advertised link modes:  10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Advertised auto-negotiation: Yes
        Speed: 1000Mb/s
        Duplex: Full
        Port: Twisted Pair
        PHYAD: 2
        Transceiver: internal
        Auto-negotiation: on
        Supports Wake-on: pumbg
        Wake-on: g
        Current message level: 0x00000007 (7)
        Link detected: yes
```



현재 지원 속도는 1000MB/s 이다. 

보통 1Gbps NIC이 최대 보내고 받을 수 있는 maximum은 약 125 MB/s, 10Gbps NIC이 최대 보내고 받을 수 있는 maximum은 약 1,250MB/s이다.

* 1 Gbps (Giga bit per second) = 1,000 Mbit/s (Megabit) = 125MB/s

