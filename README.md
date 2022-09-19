<h3>### IPTABLES ###</h3>

<h4>Описание домашнего задания</h4>

<p>Сценарии iptables</p>

<ol>
<li>реализовать knocking port<br />
● centralRouter может попасть на ssh inetrRouter через knock скрипт<br />
пример в материалах.</li>
<li>добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост.</li>
<li>запустить nginx на centralServer.</li>
<li>пробросить 80й порт на inetRouter2 8080.</li>
<li>дефолт в инет оставить через inetRouter.<br />
Формат сдачи ДЗ - vagrant + ansible<br />
● реализовать проход на 80й порт без маскарадинга</li>
</ol>

<p>Для выполнения данного ДЗ воспользуемся стендом из ДЗ "Архитектура сетей" и добавим к этому стенду сервер inetRouter2:</p>

<table>
<tr>
    <th>Server</th>
    <th>IP and Bitmask</th>
    <th>OS</th>
</tr>
<tr>
    <td rowspan="3">inetRouter</td>
    <td>Default-NAT address VirtualBox</td>
    <td rowspan="3">CentOS 7</td>
</tr>
<tr>
    <td>192.168.255.1/30</td>
</tr>
<tr>
    <td>192.168.50.10/24</td>
</tr>
<tr>
    <td rowspan="7">centralRouter</td>
    <td>192.168.255.2/30</td>
    <td rowspan="7">CentOS 7</td>
</tr>
<tr>
    <td>192.168.0.1/28</td>
</tr>
<tr>
    <td>192.168.0.33/28</td>
</tr>
<tr>
    <td>192.168.0.65/26</td>
</tr>
<tr>
    <td>192.168.255.9/30</td>
</tr>
<tr>
    <td>192.168.255.5/30</td>
</tr>
<tr>
    <td>192.168.50.11/24</td>
</tr>
<tr>
    <td rowspan="2">centralServer</td>
    <td>192.168.0.2/28</td>
    <td rowspan="2">CentOS 7</td>
</tr>
<tr>
    <td>192.168.50.12/24</td>
</tr>
<tr>
    <td rowspan="2">inetRouter2</td>
	<td>192.168.255.3/30</td>
    <td rowspan="2">CentOS 7</td>
</tr>
<tr>
    <td>192.168.50.13/24</td>
</tr>
<tr>
    <td rowspan="6">office1Router</td>
    <td>192.168.255.10/30</td>
    <td rowspan="6">Ubuntu 20</td>
</tr>
<tr>
    <td>192.168.2.1/26</td>
</tr>
<tr>
    <td>192.168.2.65/26</td>
</tr>
<tr>
    <td>192.168.2.129/26</td>
</tr>
<tr>
    <td>192.168.2.193/26</td>
</tr>
<tr>
    <td>192.168.50.20/26</td>
</tr>
<tr>
    <td rowspan="2">office1Server</td>
    <td>192.168.2.130/26</td>
    <td rowspan="2">Ubuntu 20</td>
</tr>
<tr>
    <td>192.168.50.21/26</td>
</tr>
<tr>
    <td rowspan="5">office2Router</td>
    <td>192.168.255.6/30</td>
    <td rowspan="5">Debian 11</td>
</tr>
<tr>
    <td>192.168.1.1/26</td>
</tr>
<tr>
    <td>192.168.1.129/26</td>
</tr>
<tr>
    <td>192.168.1.193/26</td>
</tr>
<tr>
    <td>192.168.50.30/26</td>
</tr>
<tr>
    <td rowspan="2">office2Server</td>
    <td>192.168.1.2/26</td>
    <td rowspan="2">Debian 11</td>
</tr>
<tr>
    <td>192.168.50.31/26</td>
</tr>
</table>

<p>Запустим эти виртуальные машины:</p>

<pre>[user@localhost iptables]$ vagrant up</pre>

<pre>[user@localhost iptables]$ vagrant status
Current machine states:

inetRouter                running (virtualbox)
centralRouter             running (virtualbox)
centralServer             running (virtualbox)
inetRouter2               running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
[user@localhost iptables]$</pre>


<h4>Реализация knocking port</h4>

<p>У нас есть сервер inetRouter с IP-адресом 192.168.255.1. Нам необходимо запретить до него доступ всем, кроме тех, кто знает «как правильно постучаться». Последовательность портов будет такая: 8881 7777 9991</p>

<p>Подключимся к нему по ssh и зайдём под пользователем root:</p>

<pre>[user@localhost iptables]$ vagrant ssh inetRouter
[vagrant@inetRouter ~]$ sudo -i
[root@inetRouter ~]#</pre>

<p>Создаём длинное правило на сервере. Записываем его в файл iptables.rules:</p>

<pre>[root@inetRouter ~]# vi ./iptables.rules</pre>

<pre>*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:TRAFFIC - [0:0]
:SSH-INPUT - [0:0]
:SSH-INPUTTWO - [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -s 192.168.56.0/24 -m tcp --dport 22 -j ACCEPT
-A INPUT -j TRAFFIC
-A TRAFFIC -p icmp --icmp-type any -j ACCEPT
-A TRAFFIC -m state --state ESTABLISHED,RELATED -j ACCEPT
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 22 -m recent --rcheck --seconds 30 --name SSH2 -j ACCEPT
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH2 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 9991 -m recent --rcheck --name SSH1 -j SSH-INPUTTWO
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH1 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 7777 -m recent --rcheck --name SSH0 -j SSH-INPUT
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH0 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 8881 -m recent --name SSH0 --set -j DROP
-A SSH-INPUT -m recent --name SSH1 --set -j DROP
-A SSH-INPUTTWO -m recent --name SSH2 --set -j DROP
-A TRAFFIC -j DROP
COMMIT
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
COMMIT</pre>

<p>Так как стенд перенят со стенда "Архитектура сетей", то сервис iptables при деплое стенда должен быть установлен, запущен и включен по умолчанию:</p>

<pre>[root@inetRouter ~]# systemctl status iptables
● iptables.service - IPv4 firewall with iptables
   Loaded: loaded (/usr/lib/systemd/system/iptables.service; <b>enabled</b>; vendor preset: disabled)
   Active: <b>active</b> (exited) since Mon 2022-09-19 18:15:32 UTC; 1min 43s ago
  Process: 22916 ExecStart=/usr/libexec/iptables/iptables.init start (code=exited, status=0/SUCCESS)
 Main PID: 22916 (code=exited, status=0/SUCCESS)

Sep 19 18:15:32 inetRouter systemd[1]: Starting IPv4 firewall with iptables...
Sep 19 18:15:32 inetRouter iptables.init[22916]: iptables: Applying firewall...]
Sep 19 18:15:32 inetRouter systemd[1]: Started IPv4 firewall with iptables.
Hint: Some lines were ellipsized, use -l to show in full.
[root@inetRouter ~]#</pre>

<p>Импортируем правила iptables с файла iptables.rules:</p>

<pre>[root@inetRouter ~]# iptables-restore < ./iptables.rules 
[root@inetRouter ~]#</pre>

<p>Сохраняем эти правила:</p>

<pre>[root@inetRouter ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
[root@inetRouter ~]#</pre>

<p>Перезапустим сервис iptables:</p>

<pre>[root@inetRouter ~]# systemctl restart iptables
[root@inetRouter ~]#</pre>

<p>Подключимся по ssh к серверу centralRouter и зайдём под пользователем root:</p>

<pre>[user@localhost iptables]$ vagrant ssh centralRouter
Last login: Mon Sep 19 18:15:34 2022 from 192.168.50.1
[vagrant@centralRouter ~]$ sudo -i
[root@centralRouter ~]#</pre>

<p>Попробуем подключиться по ssh к серверу inetRouter:</p>

<pre>[root@centralRouter ~]# ssh vagrant@192.168.255.1
ssh: connect to host 192.168.255.1 port 22: Connection timed out
[root@centralRouter ~]#</pre>

<p>Как видим, подключиться по ssh к серверу inetRouter не удалось.</p>

<p>Теперь доступ есть только у тех, кто знает нашу последовательность. Будем «стучаться» с помощью nmap. Для этого напишем простенький скрипт:</p>

<pre>[root@centralRouter ~]# vi ./knock.sh</pre>

<pre>#!/bin/bash
HOST=$1
shift
for ARG in "$@"
do
  sudo nmap -Pn --max-retries 0 -p $ARG $HOST
done</pre>

<p>Сделаем его исполняемым:</p>

<pre>[root@centralRouter ~]# chmod +x ./knock.sh
[root@centralRouter ~]#</pre>

<p>Установим утилиту nmap:</p>

<pre>[root@centralRouter ~]# yum -y install nmap</pre>

<p>Запустим скрипт knock.sh:</p>

<pre>[root@centralRouter ~]# ./knock.sh 192.168.255.1 8881 7777 9991

Starting Nmap 6.40 ( http://nmap.org ) at 2022-09-19 18:56 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00040s latency).
PORT     STATE    SERVICE
8881/tcp filtered unknown
MAC Address: 08:00:27:F5:9E:A4 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.39 seconds

Starting Nmap 6.40 ( http://nmap.org ) at 2022-09-19 18:56 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00045s latency).
PORT     STATE    SERVICE
7777/tcp filtered cbt
MAC Address: 08:00:27:F5:9E:A4 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.37 seconds

Starting Nmap 6.40 ( http://nmap.org ) at 2022-09-19 18:56 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00040s latency).
PORT     STATE    SERVICE
9991/tcp filtered issa
MAC Address: 08:00:27:F5:9E:A4 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.38 seconds
[root@centralRouter ~]# </pre>

<p>Теперь у нас есть 30 секунд времени, чтобы попытаться подключиться к серверу inetRouter:</p>

<pre>[root@centralRouter ~]# ssh vagrant@192.168.255.1</pre>

<pre>[root@centralRouter ~]# ssh vagrant@192.168.255.1
The authenticity of host '192.168.255.1 (192.168.255.1)' can't be established.
ECDSA key fingerprint is SHA256:L6gvxDqhLgeA6yUWGcSu+CDtihw77gm/RxpYQX85XKs.
ECDSA key fingerprint is MD5:6c:7a:70:0e:e6:13:d0:ec:e9:00:ce:a9:f9:88:60:93.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.255.1' (ECDSA) to the list of known hosts.
vagrant@192.168.255.1's password: 
Last login: Mon Sep 19 18:15:33 2022 from 192.168.50.1
[vagrant@inetRouter ~]$</pre>

<p>Теперь, как видим, нам с помощью скрипта knock.sh удалось подключиться по ssh к сереверу inetRouter.</p>










<h4>Добавление inetRouter2</h4>

<p>Описание установки сервера inetRouter2 уже включен в Vagrantfile. При запуске команды vagrant up сервер inetRouter2 уже установлен и запущен. Настройки сервера в дальнейшем будем производить с помощью ansible.</p>








<h4>Запуск nginx на centralServer</h4>

<p>Подключимся по ssh к серверу centralServer и зайдём под пользователем root:</p>

<pre>[user@localhost iptables]$ vagrant ssh centralServer
Last login: Mon Sep 19 18:15:53 2022 from 192.168.50.1
[vagrant@centralServer ~]$ sudo -i
[root@centralServer ~]#</pre>

<p>Прежде чем на сервер centralServer установить nginx, установим EPEl репозиторий:</p>

<pre>[root@centralServer ~]# yum -y install epel-release</pre>

<p>Теперь можем устанавливать nginx:</p>

<pre>[root@centralServer ~]# yum -y install nginx</pre>

<pre>[root@centralServer ~]# systemctl start nginx</pre>

<pre>[root@centralServer ~]# systemctl status nginx</pre>






<h4>проброс 80й порт на inetRouter2 8080.</h4>

<p>Подключимся по ssh к серверу inetRouter2 и зайдём под пользователем root:</p>

<pre>[user@localhost iptables]$ vagrant ssh centralServer
Last login: Mon Sep 19 18:15:36 2022 from 192.168.50.1
[vagrant@inetRouter2 ~]$ sudo -i
[root@inetRouter2 ~]#</pre>

<p>Установим сервис iptables и iptables-services:</p>

<pre>[root@inetRouter2 ~]# yum -y install iptables iptables-services</pre>

iptables -t nat -A PREROUTING -p tcp -d 192.168.50.13 --dport 8080 -j DNAT --to-destination 192.168.0.2:80
iptables -t nat -A POSTROUTING -s 192.168.50.0/24 -d 192.168.0.2 -j MASQUERADE
iptables -A FORWARD -i eth2 -d 192.168.0.2 -p tcp --dport 80 -j ACCEPT

-A PREROUTING -d 192.168.50.13/32 -p tcp -m tcp --dport 8080 -j DNAT --to-destination 192.168.0.2:80
-A POSTROUTING -s 192.168.50.0/24 -d 192.168.0.2/32 -j MASQUERADE
-A FORWARD -d 192.168.0.2/32 -i eth2 -p tcp -m tcp --dport 80 -j ACCEPT





<h4>дефолт в инет оставить через inetRouter.</h4>







