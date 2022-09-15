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
    <td rowspan="3">inetRouter2</td>
    <td>Default-NAT address VirtualBox</td>
    <td rowspan="3">CentOS 7</td>
</tr>
<tr>
    <td>192.168.255.3/30</td>
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


