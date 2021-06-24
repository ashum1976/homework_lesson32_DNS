# Domain Name System
## ДЗ по теме DNS
Задание:
- взять стенд https://github.com/erlong15/vagrant-bind добавить еще один сервер client2 завести в зоне dns.lab имена web1 - смотрит на клиент1 web2 смотрит на клиент2

- завести еще одну зону newdns.lab завести в ней запись www - смотрит на обоих клиентов

- настроить split-dns клиент1 - видит обе зоны, но в зоне dns.lab только web1

- клиент2 видит только dns.lab

Решение
Настраиваются view (вид) для каждого клиента отдельно, client - видит свои настроенные зоны, client2 - свои.

client - видит зоны dns.lab, newdns.lab, но в зоне dns.lab только web1
client2 - видит только dns.lab


##### Проверка зон на client:


      [vagrant@client ~]$ nslookup web1
      Server:         192.168.50.10
      Address:        192.168.50.10#53

      Name:   web1.dns.lab
      Address: 192.168.50.15

      [vagrant@client ~]$ nslookup www
      Server:         192.168.50.10
      Address:        192.168.50.10#53

      Name:   www.newdns.lab
      Address: 192.168.50.15
      Name:   www.newdns.lab
      Address: 192.168.50.16

      [vagrant@client ~]$ nslookup web2
      ;; Got SERVFAIL reply from 192.168.50.10, trying next server
      Server:         192.168.50.11
      Address:        192.168.50.11#53

      ** server can't find web2: SERVFAIL

##### Проверка зон на client2:

      [vagrant@client2 ~]$ nslookup www
      ;; Got SERVFAIL reply from 192.168.50.10, trying next server
      Server:         192.168.50.11
      Address:        192.168.50.11#53

      ** server can't find www.newdns.lab: SERVFAIL

      [vagrant@client2 ~]$ nslookup web1
      Server:         192.168.50.10
      Address:        192.168.50.10#53

      Name:   web1.dns.lab
      Address: 192.168.50.15

      [vagrant@client2 ~]$ nslookup web2
      Server:         192.168.50.10
      Address:        192.168.50.10#53

      Name:   web2.dns.lab
      Address: 192.168.50.16
