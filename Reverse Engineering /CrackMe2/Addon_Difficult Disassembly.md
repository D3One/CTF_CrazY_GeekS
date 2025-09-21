
## **CrackMe #3: The Troll That Hates Debuggers**  
**File:** `Anti_Debug_CrackMe.exe`  
**Tools I Used:** x64dbg, IDA, ScyllaHide  
**Vibe Check:** Like fighting a raccoon—it’s sneaky, angry, and hates you.  

This one’s a beast. It’s got more anti-debug tricks than my ex has trust issues. If you run it under a debugger, it’s like, *"Nope, bye!"* and dips out. Time to break its spirit.  

#### **Step 1: The Anti-Debug Circus**  
First, I ran it in x64dbg, and it instantly crashed. Rude. Opened IDA and found this gem:  
```asm  
check_debugger:  
mov eax, large fs:30h  ; PEB time!  
mov al, [eax+2]        ; BeingDebugged flag  
test al, al  
jnz debugger_detected  ; "See ya, nerd!"  
```  
Classic. It checks if a debugger is attached. I used ScyllaHide to patch this in real-time—cheating? Nah, it’s "creative problem-solving."  

#### **Step 2: The Password Check Isn’t Basic**  
After bypassing the anti-debug, I found the real code. It XORs your input with `0xAA`, adds `0x10`, and compares it to a hidden string. In pseudo-code:  
```c  
for (int i = 0; i < 10; i++) {  
    input[i] = (input[i] ^ 0xAA) + 0x10;  
}  
if (memcmp(input, secret, 10) == 0) {  
    printf("You're awesome!");  
}  
```  
So I did the reverse math on the secret bytes (found in memory): subtract `0x10`, XOR with `0xAA`. Got **`UnD3bUgM3!`**.  

#### **Step 3: Victory Dance**  
Typed that in, and boom—access granted! Felt like hacking the Matrix (if the Matrix was a poorly written CrackMe).  

**Takeaway:** This one taught me to hate anti-debug tricks but love ScyllaHide. Always look for XOR ops—they’re everywhere in crackmes.  

---
