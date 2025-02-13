# Nonyx

***Scenario:*** Exorcise Black Energy 2 from Shadowbrook’s digital infrastructure by reverse-engineering the malware’s code. You must dismantle its hooks, identify its payload, and stop its command-and-control mechanisms to restore peace to the town’s network before the Haunted Festival reaches its darkest hour.

**Tags:** Volatility2 Malfind Strings File T1014
##

Upon launching the terminal, we find a disk image called BlackEnergy.vnem and a README.txt file that tells us to use WinXPSP2x86 as the profile.

<img width="707" alt="Image" src="https://github.com/user-attachments/assets/6a95c408-e0e1-47a5-be68-01fd17ab1def" />

##
#### *Q1) Which process most likely contains injected code, providing its name, PID, and memory address? (Format: Name, PID, Address)*

**Command(s) used:**  
`python vol.py -f BlackEnergy.vnem  --profile=WinXPSP2x86 malfind`

<img width="554" alt="Image" src="https://github.com/user-attachments/assets/10fb8aa3-ee2b-49e8-b8a6-04c82a4881a6" /><br>

Among the outputs, this one in particular was interesting as it contains the MZ signature, which is common for malware files.

**Answer:** svchost.exe, 856, 0xc30000

##
#### Q2) What dump file in the malfind output directory corresponds to the memory address identified for code injection? (Format: Output File Name) (4 points)

**Command(s) used:**  
`python vol.py -f ../BlackEnergy.vnem  --profile=WinXPSP2x86 malfind -p 856 -D ../`

<img width="178" alt="Image" src="https://github.com/user-attachments/assets/2244c04a-97a0-4185-a9d4-284d220cc806" /><br>

**Answer:** process.0x80ff88d8.0xc30000.dmp

##
#### Q3) Which full filename path is referenced in the strings output of the memory section identified by malfind as containing a portable executable (PE32/MZ header)? (Format: Filename Path) (4 points)

**Command(s) used:**  
`strings ../process.0x80ff88d8.0xc30000.dmp`

<img width="560" alt="Image" src="https://github.com/user-attachments/assets/0c092f02-6cff-455c-ae12-67d9e16ecbb4" /><br>

**Answer:** C:\WINDOWS\system32\drivers\str.sys

##
#### Q4) How many functions were hooked and by which module after running the ssdt plugin and filtering out legitimate SSDT entries using egrep -v '(ntoskrnl|win32k)'? (Format: XX, Module) (4 points)

**Command(s) used:**  
`python vol.py -f ../BlackEnergy.vnem  --profile=WinXPSP2x86 ssdt | egrep -v '(ntoskrnl|win32k)'`

<img width="557" alt="Image" src="https://github.com/user-attachments/assets/e58a5e52-650c-4a84-9309-7c538e0d0ac1" /><br>

**Answer:** 14, 00004A2A

##
#### Q5) Using the modules (or modscan) plugin to identify the hooking driver from the ssdt output, what is the base address for the module found in Q4? (Format: Base Address) (4 points)

**Command(s) used:**  
`python vol.py -f ../BlackEnergy.vnem  --profile=WinXPSP2x86 modscan | grep '00004A2A'`

<img width="575" alt="Image" src="https://github.com/user-attachments/assets/4155381b-8ec2-4885-a468-42a0ae81bf2d" /><br>

**Answer:** 0xff0d1000

##
#### Q6) What is the hash for the malicious driver from the virtual memory image? (Format: SHA256) (5 points)

**Command(s) used:**

`python vol.py -f ../BlackEnergy.vnem  --profile=WinXPSP2x86 moddump -b 0xff0d1000 -D ../`

<img width="566" alt="Image" src="https://github.com/user-attachments/assets/c81fcb31-b34c-4883-88cf-1074a9df38f8" /><br>


`sha256sum ../driver.ff0d1000.sys`

**Answer:** 12b0407d9298e1a7154f5196db4a716052ca3acc70becf2d5489efd35f6c6ec8
