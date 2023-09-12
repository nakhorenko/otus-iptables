# otus-iptables

Основное задание
1. Реализовать knocking port

Два сервера: inetRouter и centralRouter.
Сначала надо проверить что они друг друга видят и мы можем залогиниться на один из них.
```
		[vagrant@centralRouter ~]$ ping 192.168.56.10
		PING 192.168.56.10 (192.168.56.10) 56(84) bytes of data.
		64 bytes from 192.168.56.10: icmp_seq=1 ttl=64 time=0.383 ms
		64 bytes from 192.168.56.10: icmp_seq=2 ttl=64 time=1.22 ms
		^C
		--- 192.168.56.10 ping statistics ---
		2 packets transmitted, 2 received, 0% packet loss, time 999ms

		[vagrant@centralRouter ~]$ ssh root@192.168.56.10
		root@192.168.56.10's password: 
		[root@inetRouter ~]# 
		[root@inetRouter ~]# exit
		logout
		Connection to 192.168.56.10 closed.
```
Пустило.
		
С сервера centralrouter пробуем подключиться по ssh

```
[root@centralRouter ~]# ssh root@192.168.255.1
^C
```
Не пускает!
Делаем:
```
[root@centralRouter ~]# nmap -Pn --host-timeout 100 --max-retries 1 -p 8881 192.168.255.1
	
Starting Nmap 6.40 ( http://nmap.org ) at 2023-06-16 15:50 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00041s latency).
PORT     STATE    SERVICE
8881/tcp filtered unknown
MAC Address: 08:00:27:8D:67:90 (Cadmus Computer Systems)
	
Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds
	
[root@centralRouter ~]# nmap -Pn --host-timeout 100 --max-retries 1 -p 7777 192.168.255.1
	
Starting Nmap 6.40 ( http://nmap.org ) at 2023-06-16 15:50 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00037s latency).
PORT     STATE    SERVICE
7777/tcp filtered cbt
MAC Address: 08:00:27:8D:67:90 (Cadmus Computer Systems)
	
Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
	
[root@centralRouter ~]# nmap -Pn --host-timeout 100 --max-retries 1 -p 9991 192.168.255.1
	
Starting Nmap 6.40 ( http://nmap.org ) at 2023-06-16 15:51 UTC
Nmap scan report for 192.168.255.1
Host is up (0.00052s latency).
PORT     STATE  SERVICE
9991/tcp closed issa
MAC Address: 08:00:27:8D:67:90 (Cadmus Computer Systems)
	
Nmap done: 1 IP address (1 host up) scanned in 0.55 seconds
```
Пробуем заново:

```
[root@centralRouter ~]# ssh root@192.168.255.1
The authenticity of host '192.168.255.1 (192.168.255.1)' can't be established.
ECDSA key fingerprint is SHA256:88V8PIkB6SJL10DApPycuJR1sbN3nhYhhv/ClsxWHqw.
ECDSA key fingerprint is MD5:3a:a9:99:72:c8:52:04:ce:d8:8e:14:1a:62:01:d7:5e.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.255.1' (ECDSA) to the list of known hosts.
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
[root@centralRouter ~]#
```

Соединение устанавливается, но не пускает, потому что в sshd_config отключено **PasswordAuthentication** 

</details>

<details>
	<summary>
 		Доступность inetRouter2(192.168.0.2) через хост машину
   	</summary>

	nakhorenko@nakhorenko-litres:~/Linux-2022-12/otus-iptables$ ping 192.168.0.2
	PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
	64 bytes from 192.168.0.2: icmp_seq=88 ttl=62 time=2.45 ms
	64 bytes from 192.168.0.2: icmp_seq=89 ttl=62 time=2.42 ms
	64 bytes from 192.168.0.2: icmp_seq=90 ttl=62 time=2.28 ms
	64 bytes from 192.168.0.2: icmp_seq=91 ttl=62 time=2.49 ms
	64 bytes from 192.168.0.2: icmp_seq=92 ttl=62 time=2.53 ms
	64 bytes from 192.168.0.2: icmp_seq=93 ttl=62 time=2.41 ms
	^C
	--- 192.168.0.2 ping statistics ---
	93 packets transmitted, 6 received, 93.5484% packet loss, time 94097ms
	rtt min/avg/max/mdev = 2.282/2.431/2.532/0.078 ms
 
	nakhorenko@nakhorenko-litres:~/Linux-2022-12/otus-iptables$ traceroute 192.168.0.2
	traceroute to 192.168.0.2 (192.168.0.2), 30 hops max, 60 byte packets
	 1  192.168.56.10 (192.168.56.10)  0.642 ms  0.589 ms  0.603 ms
	 2  192.168.255.2 (192.168.255.2)  2.550 ms  2.631 ms  2.601 ms
	 3  192.168.0.2 (192.168.0.2)  2.817 ms  2.775 ms  2.727 ms

</details>

<details>
	<summary>
		 Проброс порта на centralRouter:80
 	</summary>

Для этого добавил на inetRouter2 пару правил

```
[root@inetRouter2 ~]# iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.0.1:80
[root@inetRouter2 ~]# iptables -t nat -A POSTROUTING -p tcp --dport 80 -j SNAT --to-source 192.168.0.2:8080
```

```
nakhorenko@nakhorenko-litres:~/Linux-2022-12/otus-iptables$ curl 192.168.0.2:8080
```
Nginx на centralRouter отвечает

```
	<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
	<html>
	<head>
	  <title>Welcome to CentOS</title>
	  <style rel="stylesheet" type="text/css"> 
	
		html {
		background-image:url(img/html-background.png);
		background-color: white;
		font-family: "DejaVu Sans", "Liberation Sans", sans-serif;
		font-size: 0.85em;
		line-height: 1.25em;
		margin: 0 4% 0 4%;
		}
```
</details>
