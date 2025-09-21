#CrackMe2



### **CrackMe #2: The "Easy" One That’s Not Really Easy**  
**File:** `CrackMe_v1.exe`  
**Tools I Used:** IDA Pro, x64dbg, CFF Explorer  
**Vibe Check:** Annoying but fun—like a puzzle box that hates you.  

So, you’ve got this stupid little EXE that asks for a password. You type anything wrong, and it’s like, *"Access Denied, loser!"* (Okay, it doesn’t say "loser," but it *feels* like it). My mission: crack this thing open and find the secret sauce.  

#### **Step 1: Is This Thing Packed?**  
First, I threw it into CFF Explorer. Looked normal—no UPX, no weird packers. Cool, so no unpacking shenanigans today. Opened it in IDA, and boom, I’m in `main()`. Pro tip: always search for strings first. Found "Access Denied" and "Enter password"—easy mode.  

#### **Step 2: The Assembly Party**  
Here’s the juicy part. The code does this:  
```asm  
mov esi, offset hardcoded_pass  ; "Secret123"  
mov edi, offset user_input  
repe cmpsb                      ; Compare strings byte-by-byte  
jz access_granted               ; If match, jump to win  
```  
So it’s literally comparing your input to "Secret123". Lame, right? But hey, sometimes it’s that simple.  

#### **Step 3: I Win!**  
Password: **Secret123**. Typed it in, and yeah, "Access Granted" pops up. Felt like cheating, but whatever—a win’s a win.  

**Takeaway:** This was Baby’s First CrackMe. Good for learning where strings live and how comparisons work. If you’re new, start here—don’t be a hero.  

---
