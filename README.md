# VLAN, LACP

## Задание

В Office1 в тестовой подсети появляется сервера с доп интерфесами и адресами в internal сети testLAN:

    testClient1 - 10.10.10.254
    testClient2 - 10.10.10.254
    testServer1- 10.10.10.1
    testServer2- 10.10.10.1

Развести вланами:

testClient1 <-> testServer1 testClient2 <-> testServer2

Между centralRouter и inetRouter "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд, проверить работу c отключением интерфейсов.

## Выполнение ДЗ

Копируем файлы в каталог и запускаем Vagrantfile:

```shell
vagrant up
```

## Проверяем VLANs:

### testServer1:

```shell
Last login: Tue Mar 23 12:48:49 2021 from 10.11.11.1
[vagrant@testServer1 ~]$ sudo -i
[root@testServer1 ~]# ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe4d:77d3/64 
eth1             UP             192.168.2.2/24 fe80::a00:27ff:feeb:7710/64 
eth2             UP             10.11.11.111/24 fe80::a00:27ff:fee5:b96/64 
eth1.10@eth1     UP             10.10.10.1/24 fe80::a00:27ff:feeb:7710/64 
[root@testServer1 ~]# ping 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
64 bytes from 10.10.10.254: icmp_seq=1 ttl=64 time=10.4 ms
64 bytes from 10.10.10.254: icmp_seq=2 ttl=64 time=0.754 ms
64 bytes from 10.10.10.254: icmp_seq=3 ttl=64 time=0.714 ms
64 bytes from 10.10.10.254: icmp_seq=4 ttl=64 time=0.754 ms
64 bytes from 10.10.10.254: icmp_seq=5 ttl=64 time=0.862 ms
```

### testClient1:

```shell
otus@otus-VirtualBox:~/Desktop/HW21$ vagrant ssh testClient1
Last login: Tue Mar 23 12:48:49 2021 from 10.11.11.1
[vagrant@testClient1 ~]$ sudo -i
[root@testClient1 ~]# ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe4d:77d3/64 
eth1             UP             192.168.2.3/24 fe80::a00:27ff:fe58:1c2a/64 
eth2             UP             10.11.11.121/24 fe80::a00:27ff:fe15:81dd/64 
eth1.10@eth1     UP             10.10.10.254/24 fe80::a00:27ff:fe58:1c2a/64 
[root@testClient1 ~]# ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=8.40 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.759 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.768 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.694 ms
```

### testServer2:

```shell
tus@otus-VirtualBox:~/Desktop/HW21$ vagrant ssh testServer2
Last login: Tue Mar 23 12:48:49 2021 from 10.11.11.1
[vagrant@testServer2 ~]$ sudo -i
[root@testServer2 ~]# ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe4d:77d3/64 
eth1             UP             192.168.2.4/24 fe80::a00:27ff:fe28:1170/64 
eth2             UP             10.11.11.112/24 fe80::a00:27ff:fe9c:9e5a/64 
eth1.20@eth1     UP             10.10.10.1/24 fe80::a00:27ff:fe28:1170/64 
[root@testServer2 ~]# ping 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
64 bytes from 10.10.10.254: icmp_seq=1 ttl=64 time=2.37 ms
64 bytes from 10.10.10.254: icmp_seq=2 ttl=64 time=0.607 ms
64 bytes from 10.10.10.254: icmp_seq=3 ttl=64 time=0.511 ms
64 bytes from 10.10.10.254: icmp_seq=4 ttl=64 time=1.88 ms
```

### testClient2:

```shell
otus@otus-VirtualBox:~/Desktop/HW21$ vagrant ssh testClient2
Last login: Tue Mar 23 12:48:50 2021 from 10.11.11.1
[vagrant@testClient2 ~]$ sudo -i
[root@testClient2 ~]# ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe4d:77d3/64 
eth1             UP             192.168.2.5/24 fe80::a00:27ff:fe4c:2acb/64 
eth2             UP             10.11.11.122/24 fe80::a00:27ff:fe0f:cc33/64 
eth1.20@eth1     UP             10.10.10.254/24 fe80::a00:27ff:fe4c:2acb/64 
[root@testClient2 ~]# ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=1.36 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=1.14 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.706 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=1.42 ms
```

## Проверяем Bonding:

### inetRouter

```shell
otus@otus-VirtualBox:~/Desktop/HW21$ vagrant ssh inetRouter
Last login: Tue Mar 23 12:48:49 2021 from 10.11.11.1
[vagrant@inetRouter ~]$ sudo -i
[root@inetRouter ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:34:75:8f
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:2c:67:e6
Slave queue ID: 0
```

### centralRouter

```shell
otus@otus-VirtualBox:~/Desktop/HW21$ vagrant ssh centralRouter
Last login: Tue Mar 23 12:47:14 2021 from 10.11.11.1
[vagrant@centralRouter ~]$ sudo -i
[root@centralRouter ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:d6:4f:df
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:d1:63:f9
Slave queue ID: 0
```

Активный интерфейс на обоих роутерах eth1. 
Запустим пинги и выключим eth1 на inetRouter:

![image 1](https://github.com/IvanPrivalov/HW21/blob/master/bonding.png)
