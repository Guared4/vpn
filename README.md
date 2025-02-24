# Домашнее задание VPN.

## Цель домашнего задания  
Создать домашнюю сетевую лабораторию. Научится настраивать VPN-сервер в Linux-based системах.

### Описание домашнего задания  

Что нужно сделать?

1. Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях    
2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на ВМ   
3. (*) Самостоятельно изучить и настроить ocserv, подключиться с хоста к ВМ.    
    

Формат сдачи ДЗ - vagrant + ansible        

# Выполнение:  

### 1. С помощью vagrant развернул тестовый стенд из двух виртуальных машин:
|Имя|IP-адрес|
|-|-|
|Server|192.168.57.10|
|Client|192.168.57.20|    

Структура сети    
[Хост] --- [VPN-сервер] --- [VPN-клиент]

### 2. Создал и запустил playbook vpn.yml    

```shell
root@ansible:/home/vagrant/ansible# ansible-playbook vpn.yml

PLAY [Install OpenVPN] ******************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [server]
ok: [client]
...
PLAY RECAP ******************************************************************************************************************************
client                     : ok=7    changed=6    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0
server                     : ok=8    changed=7    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0

```    
Playbook отработал без ошибок    

### 3. Настройка VPN в режимах TUN/TAP    
Проверка подключения между сервером и клиентом в режиме TAP    

*server*    

```shell
root@server:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:d4:5c:ff:2f:fe brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85798sec preferred_lft 85798sec
    inet6 fd00::d4:5cff:feff:2ffe/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 86389sec preferred_lft 14389sec
    inet6 fe80::d4:5cff:feff:2ffe/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ff:cf:25 brd ff:ff:ff:ff:ff:ff
    inet 192.168.57.10/24 brd 192.168.57.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feff:cf25/64 scope link
       valid_lft forever preferred_lft forever
4: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether fe:5a:95:37:5c:86 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fe80::7407:7ff:fe0f:7ac7/64 scope link
       valid_lft forever preferred_lft forever
```    
*client*    

```shell
root@client:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:d4:5c:ff:2f:fe brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85888sec preferred_lft 85888sec
    inet6 fd00::d4:5cff:feff:2ffe/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 86382sec preferred_lft 14382sec
    inet6 fe80::d4:5cff:feff:2ffe/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:29:ce:77 brd ff:ff:ff:ff:ff:ff
    inet 192.168.57.20/24 brd 192.168.57.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe29:ce77/64 scope link
       valid_lft forever preferred_lft forever
4: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether c2:a0:c9:86:31:28 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.2/24 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fe80::9c8a:6cff:fe81:3254/64 scope link
       valid_lft forever preferred_lft forever
```    
из вывода команды "ip a" вижу что tap0-адаптер поднялся на обоих машинах    

Проверяю связь между сервером и клиентом    
```shell
root@server:~# ping 10.10.10.2
PING 10.10.10.2 (10.10.10.2) 56(84) bytes of data.
64 bytes from 10.10.10.2: icmp_seq=1 ttl=64 time=0.903 ms
64 bytes from 10.10.10.2: icmp_seq=2 ttl=64 time=1.12 ms
64 bytes from 10.10.10.2: icmp_seq=3 ttl=64 time=0.797 ms
64 bytes from 10.10.10.2: icmp_seq=4 ttl=64 time=1.13 ms
^C
--- 10.10.10.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3046ms
rtt min/avg/max/mdev = 0.797/0.987/1.132/0.142 ms
```    
#### 3.1. Замер скорости в туннеле TAP    
На сервере запустил iperf3 в режиме сервера    
```shell
root@server:~# iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
#после завершения работы команды iperf3 на клиенте, на сервере получил такой вывод
Accepted connection from 10.10.10.2, port 46560
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 46564
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  8.65 MBytes  72.5 Mbits/sec
[  5]   1.00-2.00   sec  8.07 MBytes  67.6 Mbits/sec
[  5]   2.00-3.00   sec  8.14 MBytes  68.3 Mbits/sec
[  5]   3.00-4.00   sec  8.20 MBytes  68.8 Mbits/sec
[  5]   4.00-5.00   sec  8.22 MBytes  68.9 Mbits/sec
[  5]   5.00-6.00   sec  8.26 MBytes  69.3 Mbits/sec
[  5]   6.00-7.00   sec  8.22 MBytes  68.9 Mbits/sec
[  5]   7.00-8.00   sec  8.27 MBytes  69.4 Mbits/sec
[  5]   8.00-9.00   sec  8.22 MBytes  69.0 Mbits/sec
[  5]   9.00-10.00  sec  8.26 MBytes  69.3 Mbits/sec
[  5]  10.00-11.00  sec  8.22 MBytes  69.0 Mbits/sec
[  5]  11.00-12.00  sec  8.25 MBytes  69.2 Mbits/sec
[  5]  12.00-13.00  sec  8.11 MBytes  68.0 Mbits/sec
[  5]  13.00-14.00  sec  8.14 MBytes  68.3 Mbits/sec
[  5]  14.00-15.00  sec  8.30 MBytes  69.6 Mbits/sec
[  5]  15.00-16.00  sec  8.29 MBytes  69.5 Mbits/sec
[  5]  16.00-17.00  sec  8.23 MBytes  69.1 Mbits/sec
[  5]  17.00-18.00  sec  7.97 MBytes  66.9 Mbits/sec
[  5]  18.00-19.00  sec  8.32 MBytes  69.8 Mbits/sec
[  5]  19.00-20.00  sec  8.22 MBytes  68.9 Mbits/sec
[  5]  20.00-21.00  sec  8.28 MBytes  69.5 Mbits/sec
[  5]  21.00-22.00  sec  8.25 MBytes  69.2 Mbits/sec
[  5]  22.00-23.00  sec  7.38 MBytes  61.9 Mbits/sec
[  5]  23.00-24.00  sec  8.69 MBytes  72.9 Mbits/sec
[  5]  24.00-25.00  sec  8.13 MBytes  68.2 Mbits/sec
[  5]  25.00-26.00  sec  8.13 MBytes  68.2 Mbits/sec
[  5]  26.00-27.00  sec  8.08 MBytes  67.8 Mbits/sec
[  5]  27.00-28.00  sec  8.06 MBytes  67.6 Mbits/sec
[  5]  28.00-29.00  sec  8.11 MBytes  68.0 Mbits/sec
[  5]  29.00-30.00  sec  7.99 MBytes  67.0 Mbits/sec
[  5]  30.00-31.00  sec  8.14 MBytes  68.3 Mbits/sec
[  5]  31.00-32.00  sec  8.13 MBytes  68.2 Mbits/sec
[  5]  32.00-33.00  sec  8.13 MBytes  68.2 Mbits/sec
[  5]  33.00-34.00  sec  8.13 MBytes  68.2 Mbits/sec
[  5]  34.00-35.00  sec  8.03 MBytes  67.3 Mbits/sec
[  5]  35.00-36.00  sec  8.24 MBytes  69.1 Mbits/sec
[  5]  36.00-37.00  sec  8.05 MBytes  67.5 Mbits/sec
[  5]  37.00-38.00  sec  7.95 MBytes  66.7 Mbits/sec
[  5]  38.00-39.00  sec  8.03 MBytes  67.4 Mbits/sec
[  5]  39.00-40.00  sec  8.01 MBytes  67.2 Mbits/sec
[  5]  40.00-40.08  sec   570 KBytes  59.7 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-40.08  sec   327 MBytes  68.5 Mbits/sec                  receiver
```    
На клиенте запустил iperf3 в режиме клиента    
```shell
root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 46564 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  42.9 MBytes  72.0 Mbits/sec   28    415 KBytes
[  5]   5.00-10.00  sec  41.4 MBytes  69.5 Mbits/sec   50    284 KBytes
[  5]  10.00-15.00  sec  41.4 MBytes  69.5 Mbits/sec    0    369 KBytes
[  5]  15.00-20.00  sec  40.7 MBytes  68.4 Mbits/sec    0    464 KBytes
[  5]  20.00-25.00  sec  41.4 MBytes  69.5 Mbits/sec   47    224 KBytes
[  5]  25.00-30.00  sec  40.2 MBytes  67.4 Mbits/sec   14    257 KBytes
[  5]  30.00-35.00  sec  40.0 MBytes  67.1 Mbits/sec   41    244 KBytes
[  5]  35.00-40.00  sec  41.0 MBytes  68.8 Mbits/sec   33    289 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec   329 MBytes  69.0 Mbits/sec  213             sender
[  5]   0.00-40.08  sec   327 MBytes  68.5 Mbits/sec                  receiver

iperf Done.
```    

#### 3.2. Проверка в режиме TUN    
Для этого необходимо изменить конфигурацию OpenVPN /etc/openvpn/server.conf на сервере и клиенте, заменив dev tap на dev tun.    
Перезапустить сервис OpenVPN на сервере и клиенте

*server*    
```shell
dev tun
ifconfig 10.10.10.1 255.255.255.0
topology subnet
secret /etc/openvpn/static.key
comp-lzo
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3

root@server:~# systemctl restart openvpn@server
``` 
```shell
root@server:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:d4:5c:ff:2f:fe brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85483sec preferred_lft 85483sec
    inet6 fd00::d4:5cff:feff:2ffe/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 85978sec preferred_lft 13978sec
    inet6 fe80::d4:5cff:feff:2ffe/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:88:fc:45 brd ff:ff:ff:ff:ff:ff
    inet 192.168.57.10/24 brd 192.168.57.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe88:fc45/64 scope link
       valid_lft forever preferred_lft forever
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.10.10.1/24 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::927a:7966:500:743/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
```    
*client*    
```shell
dev tun
remote 192.168.57.10
ifconfig 10.10.10.2 255.255.255.0
topology subnet
route 192.168.57.0 255.255.255.0
secret /etc/openvpn/static.key
comp-lzo
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3

root@client:~# systemctl restart openvpn@server
```    
```shell
root@client:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:d4:5c:ff:2f:fe brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85563sec preferred_lft 85563sec
    inet6 fd00::d4:5cff:feff:2ffe/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 86019sec preferred_lft 14019sec
    inet6 fe80::d4:5cff:feff:2ffe/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e2:b5:a8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.57.20/24 brd 192.168.57.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fee2:b5a8/64 scope link
       valid_lft forever preferred_lft forever
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.10.10.2/24 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::605e:8d08:15af:fa0e/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
```    
Из вывода команды "ip a" вижу что tun0-адаптер поднялся на обоих машинах    

Проверяю связь между сервером и клиентом    
```shell
root@server:~# ping 10.10.10.2
PING 10.10.10.2 (10.10.10.2) 56(84) bytes of data.
64 bytes from 10.10.10.2: icmp_seq=1 ttl=64 time=0.903 ms
64 bytes from 10.10.10.2: icmp_seq=2 ttl=64 time=1.12 ms
64 bytes from 10.10.10.2: icmp_seq=3 ttl=64 time=0.797 ms
64 bytes from 10.10.10.2: icmp_seq=4 ttl=64 time=1.13 ms
^C
--- 10.10.10.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3046ms
rtt min/avg/max/mdev = 0.797/0.987/1.132/0.142 ms
```    
#### 3.3. Замер скорости в туннеле TUN    

На сервере запустил iperf3 в режиме сервера    
```shell
root@server:~# iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
#после завершения работы команды iperf3 на клиенте, на сервере получил такой вывод
Accepted connection from 10.10.10.2, port 52210
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 52226
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  16.4 MBytes   138 Mbits/sec
[  5]   1.00-2.00   sec  17.2 MBytes   144 Mbits/sec
[  5]   2.00-3.00   sec  11.9 MBytes   100 Mbits/sec
[  5]   3.00-4.00   sec  9.68 MBytes  81.2 Mbits/sec
[  5]   4.00-5.00   sec  9.51 MBytes  79.8 Mbits/sec
[  5]   5.00-6.00   sec  9.69 MBytes  81.3 Mbits/sec
[  5]   6.00-7.00   sec  9.64 MBytes  80.8 Mbits/sec
[  5]   7.00-8.00   sec  9.66 MBytes  81.0 Mbits/sec
[  5]   8.00-9.00   sec  8.67 MBytes  72.8 Mbits/sec
[  5]   9.00-10.00  sec  7.65 MBytes  64.1 Mbits/sec
[  5]  10.00-11.00  sec  8.56 MBytes  71.9 Mbits/sec
[  5]  11.00-12.00  sec  9.73 MBytes  81.6 Mbits/sec
[  5]  12.00-13.00  sec  7.97 MBytes  66.9 Mbits/sec
[  5]  13.00-14.00  sec  7.32 MBytes  61.4 Mbits/sec
[  5]  14.00-15.00  sec  8.59 MBytes  72.1 Mbits/sec
[  5]  15.00-16.00  sec  7.42 MBytes  62.3 Mbits/sec
[  5]  16.00-17.00  sec  7.29 MBytes  61.2 Mbits/sec
[  5]  17.00-18.00  sec  7.34 MBytes  61.6 Mbits/sec
[  5]  18.00-19.00  sec  7.27 MBytes  61.0 Mbits/sec
[  5]  19.00-20.00  sec  8.02 MBytes  67.3 Mbits/sec
[  5]  20.00-21.00  sec  7.47 MBytes  62.7 Mbits/sec
[  5]  21.00-22.00  sec  7.71 MBytes  64.7 Mbits/sec
[  5]  22.00-23.00  sec  8.21 MBytes  68.9 Mbits/sec
[  5]  23.00-24.00  sec  8.68 MBytes  72.8 Mbits/sec
[  5]  24.00-25.00  sec  8.62 MBytes  72.3 Mbits/sec
[  5]  25.00-26.00  sec  8.75 MBytes  73.4 Mbits/sec
[  5]  26.00-27.00  sec  8.64 MBytes  72.4 Mbits/sec
[  5]  27.00-28.00  sec  8.77 MBytes  73.6 Mbits/sec
[  5]  28.00-29.00  sec  8.44 MBytes  70.8 Mbits/sec
[  5]  29.00-30.00  sec  8.71 MBytes  73.1 Mbits/sec
[  5]  30.00-31.00  sec  8.70 MBytes  73.0 Mbits/sec
[  5]  31.00-32.00  sec  8.62 MBytes  72.3 Mbits/sec
[  5]  32.00-33.00  sec  8.59 MBytes  72.0 Mbits/sec
[  5]  33.00-34.00  sec  8.66 MBytes  72.6 Mbits/sec
[  5]  34.00-35.00  sec  8.52 MBytes  71.5 Mbits/sec
[  5]  35.00-36.00  sec  8.59 MBytes  72.0 Mbits/sec
[  5]  36.00-37.00  sec  8.55 MBytes  71.7 Mbits/sec
[  5]  37.00-38.00  sec  8.15 MBytes  68.4 Mbits/sec
[  5]  38.00-39.00  sec  14.8 MBytes   124 Mbits/sec
[  5]  39.00-40.00  sec  9.64 MBytes  80.8 Mbits/sec
[  5]  40.00-40.04  sec   367 KBytes  74.4 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-40.04  sec   367 MBytes  76.8 Mbits/sec                  receiver
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
```    
На клиенте запустил iperf3 в режиме клиента    
```shell
root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 52226 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  66.7 MBytes   112 Mbits/sec  469    215 KBytes
[  5]   5.00-10.00  sec  45.2 MBytes  75.8 Mbits/sec   53    270 KBytes
[  5]  10.00-15.00  sec  42.7 MBytes  71.6 Mbits/sec    7    303 KBytes
[  5]  15.00-20.00  sec  37.0 MBytes  62.1 Mbits/sec    1    307 KBytes
[  5]  20.00-25.00  sec  41.1 MBytes  68.9 Mbits/sec    8    329 KBytes
[  5]  25.00-30.00  sec  43.5 MBytes  72.9 Mbits/sec   37    307 KBytes
[  5]  30.00-35.00  sec  42.8 MBytes  71.8 Mbits/sec   41    211 KBytes
[  5]  35.00-40.00  sec  49.1 MBytes  82.4 Mbits/sec   21    293 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec   368 MBytes  77.2 Mbits/sec  637             sender
[  5]   0.00-40.04  sec   367 MBytes  76.8 Mbits/sec                  receiver

iperf Done.
```    
Сравнение результатов:    
По выводу команды iperf3, скорости в режимах TAP и TUN отличаются    
TAP медленнее, но поддерживает Ethernet-трафик.     
TUN быстрее, но работает только с IP-трафиком.

### 4. Поднятие RAS на базе OpenVPN с клиентскими сертификатами    
Remote Access Server VPN используется для получения удаленного доступа клиентами к внутренней сети организации.   
Сначала установил пакет easy-rsa, для создания центра сертификации и сгенерирую сертифкаты сервера и клиента.        

```shell
root@server:~# apt install - y easy-rsa
```   
Настройка центра сертификации (CA)   

```shell
#Перехожу в директорию
root@server:~# cd /etc/openvpn

#Инициализирую PKI (Public Key Infrastructure) - каталоги для хранения ключей и сертификатов
root@server:/etc/openvpn# /usr/share/easy-rsa/easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/pki

#Создаю центр сертификации (CA)
root@server:/etc/openvpn# /usr/share/easy-rsa/easyrsa build-ca
Using SSL: openssl OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)

Enter New CA Key Passphrase:
Re-Enter New CA Key Passphrase:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:otus-vpn

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/pki/ca.crt

#Генерация сертификатов для сервера и клиента
root@server:/etc/openvpn# /usr/share/easy-rsa/easyrsa gen-req server nopass
Using SSL: openssl OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.+...+..+.+.....+..................+.....
.+.+..+.......+........+...+....+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+......
+..+...+.......+...+..+.+.....+............+...+.......+..+.......+.......................+..........+........+
....+...+........+.......+..+...+.+.....+.......+...+............+..+............+................+..+.......+.
..+........+..........+.........+...+...+........+.......+...+.....+.+...+...+..+............+.+........+++++++
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
....+...+...+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+......+.+........+...
....+.....+.+.....+.........+...+...+...+.+...+...+............+...+..................+++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++++*........+......+..+.......+...........+.........+.+...+..+....+...+...
..+...+...+.......+.........+...+..+...+............+.....................+...+....+......+...........+....+..+
....+.....+...+...+...+.+.........+..+..........+...............+..............+...+......+......+.+........+..
..+.....+...............+...+....+......+..+.+..+.......+......+......+.....+....+..+...+.+...........+...+....
+......+..+...+....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]:otus-vpn

Keypair and certificate request completed. Your files are:
req: /etc/openvpn/pki/reqs/server.req
key: /etc/openvpn/pki/private/server.key

#Подписал запрос на сертификат для сервера
root@server:/etc/openvpn# /usr/share/easy-rsa/easyrsa sign-req server server
Using SSL: openssl OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 825 days:

subject=
    commonName                = otus-vpn


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /etc/openvpn/pki/easy-rsa-2307.kl7Eoh/tmp.rItfkp
Enter pass phrase for /etc/openvpn/pki/private/ca.key:
40C76D06887F0000:error:0700006C:configuration file routines:NCONF_get_string:no value:../crypto/conf/conf_lib.c
:315:group=<NULL> name=unique_subject
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'otus-vpn'
Certificate is to be certified until May 30 12:01:52 2027 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/openvpn/pki/issued/server.crt

#Сгенерировал запрос на сертификат для клиента
root@server:/etc/openvpn# /usr/share/easy-rsa/easyrsa gen-req client nopass
Using SSL: openssl OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
.+...+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+..........+..+....+.....+++++++++
++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+......+......+.........+.....+.+...+.........+..
...+...+....+...............+...+............+.....+...+......+.........+...+....+........+......+......+....+.
.+..........+.................+..........+...+...+..+.........+..........+..................+...............+..
+....+.........+..+...+.......+........+...+...+.+......+...+.....+.+...+...........+......................+...
.........+..+.+...+...............+..+.......+..+.........+............+.........+.........+.+........+...+....
......+.....+...+.+.....+.+..............+...............+......+...+....+......+...............+.....+......+.
+.........+........+...+...+....+..+....+..+...++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+
..........+....+.....+....+..+..........+...+...+..............+..........+...+........+....+........+.+......+
......+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+.....+................+....
.+....+.....+.+...........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+..+.............
+.........+..............+.+.....+.+.....+...+...............+..........+...........................+..+.+..+..
.....+..+....+.....+.+.....+.+...+......+...+........+...+.+...+....................+...+.............+..+...+.
...........+............+.......+......+..+...+..........+.........+.........+..+...+......+...+...+....+.....+
................+............+...+...+.....+.........+...+.......+........+...+.+......+..................+....
.....+..+......+.............+..+....+......+.......................................+.....+.+.....+...+....+...
..+................+..+....+......+........+.+...+..+...+............+..........+......+..+.......+......+...+.
.+.......+...+..+.+.........+..+...+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [client]:client

Keypair and certificate request completed. Your files are:
req: /etc/openvpn/pki/reqs/client.req
key: /etc/openvpn/pki/private/client.key

#Подписал запрос на сертификат для клиента
root@server:/etc/openvpn# /usr/share/easy-rsa/easyrsa sign-req client client
Using SSL: openssl OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 825 days:

subject=
    commonName                = client


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /etc/openvpn/pki/easy-rsa-2390.xmXvsN/tmp.F26N2K
Enter pass phrase for /etc/openvpn/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'client'
Certificate is to be certified until May 30 12:05:33 2027 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/openvpn/pki/issued/client.crt
```    
Созданные файлы скопировал в рабочую папку ДЗ ansible/templates, для будущей автоматизации

Создал и запустил playbook vpn-ras.yml   
```shell
root@ansible:/home/vagrant/ansible# ansible-playbook vpn-ras.yml

PLAY [Install OpenVPN RAS] ************************************************************************************

TASK [Gathering Facts] ****************************************************************************************
ok: [server]

TASK [Install packages] ***************************************************************************************
changed: [server]

TASK [Copy OpenVPN config file] *******************************************************************************
changed: [server]

TASK [Copy CA cert] *******************************************************************************************
changed: [server]

TASK [Copy certificates] **************************************************************************************
changed: [server] => (item=key)
changed: [server] => (item=crt)

TASK [Create systemd unit] ************************************************************************************
ok: [server]

TASK [Start OpenVPN] ******************************************************************************************
changed: [server]

PLAY RECAP ****************************************************************************************************
server                     : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```    
Playbook отработал без ошибок    

Проверяю доступность сервера с хоста
```shell
PS F:\VM\Vagrant\DZ\VPN\VPN> ping 10.10.10.1

Обмен пакетами с 10.10.10.1 по с 32 байтами данных:
Ответ от 10.10.10.1: число байт=32 время<1мс TTL=64
Ответ от 10.10.10.1: число байт=32 время<1мс TTL=64
Ответ от 10.10.10.1: число байт=32 время<1мс TTL=64
Ответ от 10.10.10.1: число байт=32 время<1мс TTL=64

Статистика Ping для 10.10.10.1:
    Пакетов: отправлено = 4, получено = 4, потеряно = 0
    (0% потерь)
Приблизительное время приема-передачи в мс:
    Минимальное = 0мсек, Максимальное = 0 мсек, Среднее = 0 мсек
```    
А так же успешное пподключение клиентского приложения с хоста, с помощью конфигурационного файла, который был создан по такому шаблону    
```shell
client
dev tun
proto udp
remote 192.168.57.10 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
comp-lzo
verb 3

<ca>
-----BEGIN CERTIFICATE-----
MIIE... (содержимое ca.crt)
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
MIIE... (содержимое client.crt)
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
MIIE... (содержимое client.key)
-----END PRIVATE KEY-----
</key>
```    

![alt text](img/vpn.png)    



__________________   

end
