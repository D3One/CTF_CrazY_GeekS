
### **Category: Cryptography**

**Challenge 1: The XORed Manifesto**

*   **Step 1: Understand the Clue.** The hint says the key is short, repetitive, and was used on the word "creature". We know that in XOR encryption, `ciphertext ⊕ plaintext = key`. If we can guess a part of the plaintext, we can find the key.
*   **Step 2: Get the Ciphertext and Suspected Plaintext.**
    *   Ciphertext (in hex): `] V T Y J Q C [ a G C ] _ P D J [ ( R J [ E E M L A`
    *   Suspected Plaintext: `c r e a t u r e _ c r e a t u r e _ c r e a t u r e` (We'll use just the first 8 letters, "creature", to find the key).
*   **Step 3: XOR to Find the Key.**
    *   Convert the first 8 characters of both strings to their ASCII codes (e.g., ']' is 93, 'c' is 99).
    *   Perform XOR operation on each pair:
        *   93 ⊕ 99 = 62  (`>`)
        *   86 ⊕ 114 = 36  (`$`)
        *   84 ⊕ 101 = 49  (`1`)
        *   89 ⊕ 97 = 56   (`8`)
    *   We now have a 4-byte key: `> $ 1 8`
*   **Step 4: Decrypt the Entire Message.** Take the full ciphertext and XOR it with the repeating key `>$18>$18>$18...`. This can be done with a simple Python script or even manually in a hex editor.
*   **Step 5: Get the Flag.** The decrypted message will be the famous hacker's manifesto. The flag is the first line of the decrypted text.

**Challenge 2: Base64 Isn't Encryption**

*   **Step 1: Recognize the Encoding.** The string ends with `=`, a classic sign of Base64 encoding. Our first step is always to decode this.
*   **Step 2: Decode from Base64.**
    *   Command: `echo "JDEkYmxhYmxhJHFUZEhULOh6UVBKZC9yN3Zrc0FscjE=" | base64 -d`
    *   **Output:** `$1$blabla$qTdHT/HzQPJd/r7vksAlr1`
*   **Step 3: Analyze the Output.** The decoded string starts with `$1$`. This is a signature for an MD5 crypt hash, commonly used for Linux passwords. The format is `$1$salt$hash`.
*   **Step 4: Crack the Hash.** We use a password cracking tool like John the Ripper.
    *   Save the hash into a file: `echo '$1$blabla$qTdHT/HzQPJd/r7vksAlr1' > hash.txt`
    *   Run John with a wordlist (like the famous `rockyou.txt`): `john --wordlist=rockyou.txt hash.txt`
*   **Step 5: Get the Flag.** John will almost instantly crack this weak hash. The password revealed is the flag.

**Challenge 3: The Lazy Caesar**

*   **Step 1: Use the Hint.** The hint says the most common letter is 'X'. In English, the most common letter is 'E'.
*   **Step 2: Calculate the Shift.**
    *   If 'X' (88 in ASCII) represents 'E' (69), the shift is `88 - 69 = 19`.
    *   Alternatively, you can think of the alphabet: E is the 5th letter, X is the 24th. 24 - 5 = 19. So, it's ROT-19.
*   **Step 3: Decrypt the File.**
    *   Write a simple script that reads `secret.txt` and subtracts 19 from the ASCII value of every letter (wrapping around from 'A' to 'Z' if needed).
    *   Alternatively, use an online ROT decoder and try shifts until the text makes sense. ROT-19 is the same as ROT-7 (because 26-19=7), so you could also *add* 7 to each letter.
*   **Step 4: Get the Flag.** The decrypted text will be a coherent sentence. The flag is the last word of the decrypted message.

### **Category: Networking**

**Challenge 4: Sniffing the Switch**

*   **Step 1: Understand the Theory.** A switch sends traffic only to the port of the intended recipient. To capture traffic between two other machines (A and B), you need to trick them into sending their data to you first. This is a "Man-in-the-Middle" (MiTM) attack. ARP Spoofing is a common way to do this by lying about which MAC address corresponds to which IP.
*   **Step 2: Enable IP Forwarding.** So your machine doesn't become a network bottleneck, you need to forward the packets you receive to their real destination.
    *   On Linux: `echo 1 > /proc/sys/net/ipv4/ip_forward`
*   **Step 3: Execute the Attack.**
    *   Tell Machine A (192.168.1.100) that you are Machine B: `arpspoof -t 192.168.1.100 192.168.1.200`
    *   Tell Machine B (192.168.1.200) that you are Machine A: `arpspoof -t 192.168.1.200 192.168.1.100`
*   **Step 4: Capture the Traffic.** Open Wireshark and start capturing on your network interface.
*   **Step 5: Find the Password.** Let the capture run. The hint mentions port 21 (FTP). In Wireshark, apply a filter `tcp.port == 21`. Look for packets right after a "USER" command; the next packet will often contain the "PASS" command in plain text. The flag is the password found in packet #47.

**Challenge 5: The Mysterious TCP Dump**

*   **Step 1: Open the File.** Open the `trace.pcap` file in Wireshark.
*   **Step 2: Look for a Pattern.** You will see hundreds of TCP packets coming from a single source IP address and going to many different destination ports on a target IP.
*   **Step 3: Analyze the Flags.** Look at the "Info" column in Wireshark. You'll see that these packets have only the "FIN" flag set (not SYN, not ACK). This is highly unusual for normal traffic.
*   **Step 4: Understand the Scan.** A "FIN scan" is a stealthy port scanning technique. The idea is that closed ports will respond to a unexpected FIN packet with a RST (reset) packet. Open ports will simply ignore it, giving no response. By seeing which ports don't reply, the attacker can map open ports.
*   **Step 5: Find the Attacker.** Sort the packets by "Source" IP. The IP that is sending all these FIN packets is the attacker. The flag is the attacker's IP address and the scan type: `FIN scan`.

### **Category: Secure Programming & Patching**

**Challenge 6: Patch the Buffer Overflow**

*   **Step 1: Identify the Vulnerability.** The line `strcpy(buffer, argv[1]);` is dangerous because `strcpy` will copy everything from `argv[1]` into `buffer` until it hits a null terminator (`\0`). If `argv[1]` is longer than 100 characters, it will overwrite other parts of the program's memory, causing a crash or allowing code execution.
*   **Step 2: Choose a Safe Function.** The safer function is `strncpy`. It allows you to specify a maximum number of characters to copy.
*   **Step 3: Apply the Patch.**
    *   Replace `strcpy(buffer, argv[1]);` with `strncpy(buffer, argv[1], 99);`
    *   **Why 99?** Because our buffer is 100 bytes, we need to leave one byte for the null terminator.
    *   **Crucial Step:** `strncpy` does not automatically add a null terminator if the source string is too long. We must do it manually: `buffer[99] = '\0';`
*   **Step 4: Test.** Compile the patched code and try to crash it with a long input. It should now safely truncate the input instead of crashing.

**Challenge 7: Spot the Vuln - Web Edition**

*   **Vulnerability 1: SQL Injection.**
    *   **The Problem:** The code directly inserts user input (`$_GET['username']`) into an SQL query string. A malicious user could enter `' OR '1'='1` as the username. The query would become: `...WHERE username='' OR '1'='1' AND password='...'` which is always true, granting access.
    *   **The Fix:** Use **Prepared Statements** (Parameterized Queries) with PDO or MySQLi. This separates the SQL logic from the data, making injection impossible.
*   **Vulnerability 2: Passwords in Plaintext.**
    *   **The Problem:** The query compares the user's input directly to the `password` field in the database. This means the passwords are stored in plain text! If the database is hacked, all passwords are immediately compromised.
    *   **The Fix:** Never store plaintext passwords. Instead, store a **hash** of the password (e.g., using `password_hash()` in PHP). During login, you hash the user's input and compare it to the stored hash.

**Challenge 8: The Magic Number**

*   **Step 1: Open in OllyDbg.** Load `validator.exe` into OllyDbg.
*   **Step 2: Find the Interesting Code.** The program likely has strings like "Invalid" or "Congratulations". Right-click -> Search for -> All referenced text strings. Find and double-click on one of these strings.
*   **Step 3: Trace the Code.** This will take you to the code that checks the serial. Look just above this code for a function call like `strcmp` or `lstrcmp`. This is the function that compares your input to the correct serial.
*   **Step 4: Set a Breakpoint.** Click on the line of the `cmp` or `strcmp` instruction and press F2 to set a breakpoint. This will pause the program when it tries to check your serial.
*   **Step 5: Run and Inspect.** Run the program (F9). Enter a dummy name and serial. The program will pause at your breakpoint. Now look at the CPU registers (e.g., EAX, EDX) or the stack window. Olly will often show you the two strings being compared: your fake serial and the correct one for the name you entered.
*   **Step 6: Get the Flag.** The correct serial number displayed in the register or stack is the flag.

### **Category: MS Windows Puzzle**

**Challenge 9: The Secret Stream**

*   **Step 1: Understand NTFS Streams.** NTFS files can have more than one "stream" of data. The main one is unnamed. You can create others, named ones, that are hidden from `dir` and normal file explorers.
*   **Step 2: Use the Hint.** The hint gives the syntax: `notepad readme.txt:secret.txt`
*   **Step 3: Execute the Command.** Open a Command Prompt (`cmd.exe`) and navigate to the file. Type exactly: `notepad readme.txt:secret.txt`
*   **Step 4: Get the Flag.** Notepad will open the hidden stream named `secret.txt` attached to the file `readme.txt`. The contents of this stream are the flag.

**Challenge 10: Debugging the Impossible**

*   **Step 1: Run Debug.** Open Command Prompt and type `debug.exe`.
*   **Step 2: Assemble Code.** You will see a dash `-` prompt. Type `a` to start assembling code. You will now enter x86 assembly instructions line by line to create a "Hello, CTF!" program.
    ```
    mov ah, 9      ; DOS function 9 = print string
    mov dx, 108    ; DX points to the memory address of our string
    int 21         ; Call DOS to execute the function
    int 20         ; Call DOS to terminate the program
    ```
    *After `int 20`, press Enter twice to stop assembling.*
*   **Step 3: Define the String.** Now you need to put the string into memory at address 108 (hex). Type `e 108 'Hello, CTF!' 0d 0a '$'`
    *   `e` is the enter command.
    *   `0d 0a` is CR+LF (Carriage Return + Line Feed, a new line).
    *   `'$'` is the DOS string terminator.
*   **Step 4: Save the Program.**
    *   `n hello.com`  ; Name the file
    *   `rcx`          ; Ask to set the CX register (file size)
    *   `FF`           ; Set a large size, debug will handle the rest
    *   `w`            ; Write the file to disk
    *   `q`            ; Quit debug
*   **Step 5: Run It!** Type `hello.com` at the command prompt. It will print "Hello, CTF!". The successful creation and execution is the solution.

### **Category: Linux Puzzle**

**Challenge 11: The Unseen File**

*   **Step 1: List All Files.** Normal `ls /tmp` won't show it. The `-a` flag shows hidden files (starting with a dot), but this is different.
*   **Step 2: Show Non-printable Characters.** The hint suggests the filename might have an invisible character, like a space. The `-b` flag shows non-printable characters in octal.
    *   Command: `ls -lb /tmp`
*   **Step 3: Find the File.** In the listing, you might see a file like `\ fl ag.txt` or `flag.txt?`. The `\` indicates a space or other special character.
*   **Step 4: Read the File.** To read it, you can use wildcards to match the weird name: `cat /tmp/*flag*`. This will match any file in `/tmp` with "flag" in its name, regardless of leading spaces.
*   **Step 5: Get the Flag.** The output of the `cat` command is the flag.

**Challenge 12: Permission Mystery**

*   **Step 1: Check Running Processes.** Since you can't read the file directly, see if a root process already has it open. The `/proc/` filesystem contains information about every running process.
*   **Step 2: Search for the File.** Use the `lsof` command (if installed) to list open files and find our target: `lsof /etc/flag.conf`. This will show which process (PID) has it open.
*   **Step 3: Access via /proc.** Even without `lsof`, you can search `/proc`: `find /proc -name "flag.conf" 2>/dev/null`. This will find a path like `/proc/1234/fd/3`.
*   **Step 4: Read the File Descriptor.** This magical path is a link to the actual open file. You *can* read this, even as a normal user! `cat /proc/1234/fd/3`
*   **Step 5: Get the Flag.** The contents printed by `cat` are the flag.

**Challenge 13: The Environment is Key**

*   **Step 1: Analyze with ltrace.** The hint is crucial. Run: `ltrace /usr/bin/game`. When it prompts for a password, type anything and press Enter.
*   **Step 2: Observe the Output.** In the `ltrace` output, you will see a line like: `getenv("GAME_PASS")`. This proves the program is looking for an environment variable, not just your typed input.
*   **Step 3: Set the Variable.** Before running the game, set the environment variable it's looking for.
    *   Command: `export GAME_PASS=the_real_password_here`
*   **Step 4: Run the Program.** Now simply run `/usr/bin/game`. You can even press Enter without typing anything when it asks for a password. It will find the password in the `GAME_PASS` environment variable and grant you access, printing the flag.
*   **Reasoning:** SUID programs run with elevated privileges but often inherit the environment from the user who started them. If the programmer isn't careful, this can be a huge security hole.

***
