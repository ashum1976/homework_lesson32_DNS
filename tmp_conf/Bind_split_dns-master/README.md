# Bind_split_dns  
##### Запуск
`vagrant  up` запускает 4 машины провижинит ansible playbook:   
ns01 192.168.50.10 master dns server    
ns02 192.168.50.11 slave dns server   
client 192.168.50.15    
client2 192.168.50.16   

##### Конфигурация DNS   
DNS обслуживает прямые и обратные зоны:    
* dns.lab    
* newdns.lab   
* ddns.lab    
Основные конфигурационные файлы master и slave bind named.conf [ns01](https://github.com/Hanafeevrus/Bind_split_dns/blob/master/provisioning/master-named.conf) и [ns02](https://github.com/Hanafeevrus/Bind_split_dns/blob/master/provisioning/slave-named.conf)    
Зоны описаны в файлах `named.*`, которые хранятся на DNS в директории /etc/named/.
##### Настройка split dns  
Split DNS -технология с разделенным представлением DNS. [Описание](http://it2web.ru/index.php/dns/77-split-dns-nauchim-bind-rabotat-na-dva-tri-chetyre-i-bolee-frontov)		
##### Задача    
* присвоить имена в зоне dns.lab:    
client 192.168.50.15 --web1    
client2 192.168.50.16  --web2   
* присвоить имена в зоне newdns.lab:		
client www.newdns.lab		
client2 www.newdns.lab    

* client - видит зоны dns.lab,newdns.lab но в зоне dns.lab только web1    
* client2 видит только dns.lab		


![sheme](https://github.com/Hanafeevrus/Bind_split_dns/blob/master/sheme.png)
##### Реализация    
необходимо в `named.conf` указать ACL на основании которых будут предоставляться `view` (виды) для просмотра зон.   
Ниже представлен фрагмент для client:   
```    
acl client {192.168.50.15;};    
// view for client

view client {
	match-clients { client; };
	allow-transfer { 192.168.50.11; };
	recursion yes;

// root zone
zone "." IN {
	type hint;
	file "named.ca";
};

// zones like localhost
include "/etc/named.rfc1912.zones";
// root's DNSKEY
include "/etc/named.root.key";

// lab's zone
zone "dns.lab" {
    type master;
    allow-transfer { key "zonetransfer.key"; };
    file "/etc/named/named.dns.lab.client";       ///ссылки на файлы с описанием зоны
};

// lab's zone reverse
zone "50.168.192.in-addr.arpa" {
    type master;
    allow-transfer { key "zonetransfer.key"; };
    file "/etc/named/named.dns.lab.client.rev";   ///ссылки на файлы с описанием зоны
};

// lab's ddns zone
zone "ddns.lab" {
    type master;
    allow-transfer { key "zonetransfer.key"; };
    allow-update { key "zonetransfer.key"; };
    file "/etc/named/named.ddns.lab";             ///ссылки на файлы с описанием зоны
};

// newdns.lab zone
zone "newdns.lab" {
    type master;
    allow-transfer { key "zonetransfer.key"; };
    file "/etc/named/named.newdns.lab";           ///ссылки на файлы с описанием зоны
 };
};
```   
##### Результат   
```
[vagrant@client ~]$ nslookup 192.168.50.15
15.50.168.192.in-addr.arpa	name = web1.dns.lab.
15.50.168.192.in-addr.arpa	name = www.newdns.lab.

[vagrant@client ~]$ nslookup 192.168.50.16
16.50.168.192.in-addr.arpa	name = www.newdns.lab.

[vagrant@client ~]$ nslookup www
Server:		192.168.50.10
Address:	192.168.50.10#53

Name:	www.newdns.lab
Address: 192.168.50.16
Name:	www.newdns.lab
Address: 192.168.50.15

[vagrant@client ~]$ nslookup web2
Server:		192.168.50.10
Address:	192.168.50.10#53

** server can't find web2: NXDOMAIN
```
