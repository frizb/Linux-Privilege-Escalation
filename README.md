# Linux-Privilege-Escalation
Tips and Tricks for Linux Priv Escalation

## Start with the basics

Who am i?  
`id`

Can I execute code as root?  
`sudo -l`

Who else is on this box (lateral movement)?  
`ls -la /home`  
`cat /etc/passwd`  

What Kernel version and distro are we working with here?  
`uname -a`
`cat /etc/issue`

What files and folders are in my home user's directory?
`ls -la ~`

cd bo
