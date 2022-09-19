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






<h4>Реализация Port Knocking</h4>

<p>У нас есть сервер inetRouter с IP-адресом 192.168.255.1. Нам необходимо запретить до него доступ всем, кроме тех, кто знает «как правильно постучаться». Последовательность портов будет такая: 8881 7777 9991</p>

<p>Создаём длинное правило на сервере. Записываем его в файл iptables.rules:</p>

<pre>[root@inetRouter ~]# vi /etc/sysconfig/iptables</pre>

<pre>*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:TRAFFIC - [0:0]
:SSH-INPUT - [0:0]
:SSH-INPUTTWO - [0:0]

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
COMMIT</pre>

<p>На сервере выполняем команды:</p>

<pre>systemctl start iptables
systemctl enable iptables
iptables-restore < iptables.rules
service iptables save</pre>

<p>Теперь доступ есть только у тех, кто знает нашу последовательность. Я буду «стучаться» с помощью nmap. Для этого напишем простенький скрипт:</p>

<pre>[root@centralRouter ~]# vi ./knocking</pre>

<pre>#!/bin/bash
HOST=$1
shift
for ARG in "$@"
do
  sudo nmap -Pn --max-retries 0 -p $ARG $HOST
done</pre>

<pre>[root@centralRouter ~]# chmod +x ./knocking
[root@centralRouter ~]#</pre>



<pre>[root@centralRouter ~]# ssh vagrant@192.168.255.1
ssh: connect to host 192.168.255.1 port 22: Connection timed out
[root@centralRouter ~]#</pre>

<pre>[root@centralRouter ~]# ./knock.sh 192.168.255.1 8881 7777 9991

Starting Nmap 6.40 ( http://nmap.org ) at 2022-09-19 13:45 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.0018s latency).
PORT     STATE    SERVICE
8881/tcp filtered unknown
MAC Address: 08:00:27:72:76:25 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.38 seconds

Starting Nmap 6.40 ( http://nmap.org ) at 2022-09-19 13:45 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00093s latency).
PORT     STATE    SERVICE
7777/tcp filtered cbt
MAC Address: 08:00:27:72:76:25 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.35 seconds

Starting Nmap 6.40 ( http://nmap.org ) at 2022-09-19 13:45 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.0019s latency).
PORT     STATE    SERVICE
9991/tcp filtered issa
MAC Address: 08:00:27:72:76:25 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.36 seconds
[root@centralRouter ~]# ssh vagrant@192.168.255.1
The authenticity of host '192.168.255.1 (192.168.255.1)' can't be established.
ECDSA key fingerprint is SHA256:YqvRT1lwJqKWa5Oe84VgWCsDm/wUvr0sOJbuPJ/5bxg.
ECDSA key fingerprint is MD5:bd:eb:71:0d:b3:f4:b8:0b:a7:45:8e:6d:96:b1:80:83.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.255.1' (ECDSA) to the list of known hosts.
vagrant@192.168.255.1's password:
Last login: Mon Sep 19 11:45:49 2022 from 10.0.2.2
[vagrant@inetRouter ~]$</pre>
















