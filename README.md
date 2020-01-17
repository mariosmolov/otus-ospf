# otus-ospf
OSPF
- Поднять три виртуалки
- Объединить их разными private network
1. Поднять OSPF между машинами на базе Quagga
2. Изобразить ассиметричный роутинг
3. Сделать один из линков "дорогим", но что бы при этом роутинг был симметричным

# Работоспособность

Поднимаем кластер `vagrant up` с ассиметричным роутингом

Посмотрим что на роутере R1
```
[root@R1 vagrant]# ip r
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
10.10.0.0/30 dev eth1 proto kernel scope link src 10.10.0.1 metric 101 
10.20.0.0/30 dev eth2 proto kernel scope link src 10.20.0.1 metric 102 
10.30.0.0/30 via 10.20.0.2 dev eth2 proto zebra metric 20 
10.40.0.0/24 dev eth3 proto kernel scope link src 10.40.0.1 metric 103 
10.50.0.0/24 proto zebra metric 30 
	nexthop via 10.10.0.2 dev eth1 weight 1 
	nexthop via 10.20.0.2 dev eth2 weight 1 
10.60.0.0/24 via 10.20.0.2 dev eth2 proto zebra metric 20 
[root@R1 vagrant]# tracepath 10.30.0.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.20.0.2                                             3.316ms 
 1:  10.20.0.2                                             0.665ms 
 2:  10.30.0.1                                             1.076ms reached
     Resume: pmtu 1500 hops 2 back 1 
```

И на R2
```
[root@R2 [root@R2 ~]# ip r
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
10.10.0.0/30 dev eth1 proto kernel scope link src 10.10.0.2 metric 101 
10.20.0.0/30 via 10.10.0.1 dev eth1 proto zebra metric 20 
10.30.0.0/30 dev eth2 proto kernel scope link src 10.30.0.1 metric 102 
10.40.0.0/24 via 10.10.0.1 dev eth1 proto zebra metric 20 
10.50.0.0/24 dev eth3 proto kernel scope link src 10.50.0.1 metric 103 
10.60.0.0/24 proto zebra metric 30 
        nexthop via 10.30.0.2 dev eth2 weight 1 
        nexthop via 10.10.0.1 dev eth1 weight 1 vagrant]# tracepath 10.20.0.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.20.0.1                                             2.400ms reached
 1:  10.20.0.1                                             1.493ms reached
     Resume: pmtu 1500 hops 1 back 1 
[root@R2 ~]# tracepath 10.20.0.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.20.0.1                                             0.631ms reached
 1:  10.20.0.1                                             1.499ms reached
     Resume: pmtu 1500 hops 1 back 1
```

Делаем линк дорогим, роутинг симметричным
```
[root@R2 vagrant]# vtysh 

Hello, this is Quagga (version 0.99.22.4).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

R2# conf t
R2(config)# int eth1
R2(config-if)# ip ospf cost 20
R2(config-if)# exit
R2(config)# int eth2
R2(config-if)# ip ospf cost 10
R2(config-if)# exit
R2(config)# exit
R2# exit
```
И посмотрим что изменилось
```
[root@R2 vagrant]# tracepath 10.20.0.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.30.0.2                                             1.073ms 
 1:  10.30.0.2                                             0.945ms 
 2:  10.20.0.1                                             1.136ms reached
     Resume: pmtu 1500 hops 2 back 2 
```
Готово, линк дорогой
