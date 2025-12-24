# MiuiSentinel-Blinder.-Apatch-magisk-module.
A simple module for HyperOS 3 users with Root Access, that may "blind" MiuiSentinel. Comes with instructions if something goes wrong.

🛡️ Defeating HyperSentinel: A Guide to "Blinding" the Memory Watchdog (Android 16 / HyperOS 3)
The Problem: On new Xiaomi devices, the HyperSentinel component (formerly MIUISentinel) aggressively kills heavy processes (emulators like Winlator/Mobox, AAA games) once they exceed a PSS threshold of approximately 3.5 GB. Attempts to kill the Sentinel process itself result in a total system freeze (Deadlock), as the OS waits for its response.
The Solution: We do not remove Sentinel. Instead, we "blind" its monitoring function. It stays active to satisfy system dependencies (preventing freezes), but it loses the ability to see memory overflows and send the "Signal 9" kill command.
🛠 Step 1: Extracting the Target
Using a PC with ADB, pull the library responsible for the monitoring logic from your device:
bash
adb pull /system_ext/lib64/libmiuisentinel.so
Используйте код с осторожностью.

🛠 Step 2: Reverse Engineering (The Surgery)
Open the file in a HEX editor (e.g., HxD) and locate the "entry door" of the memory control function. In Android 16 (HyperOS 3), this function is responsible for checking "Laboratory environment" conditions and calculating thresholds (0.7, 0.8, 0.9).
Navigate to the function's entry address (in our specific case, the offset was 52F0).
Replace the first 8 bytes (the function prologue) with a "stub" that forces Sentinel to exit immediately:
00 00 80 D2 C0 03 5F D6
(ASM Translation: mov x0, #0 and ret).
Patch Result: Sentinel starts, enters the check function, but instead of analyzing RAM, it immediately receives an "OK" response and terminates before reaching any "kill" commands.
🛠 Step 3: Safe Installation (APatch / Magisk)
To maintain system integrity and avoid Bootloops, the patch is applied Systemlessly.
Create the following folder structure: system/system_ext/lib64/.
Place your patched libmiuisentinel.so into that lib64 folder.
Add a module.prop file with the module description.
Compress the files into a ZIP archive and install it via APatch, Magisk, or KernelSU.
✅ The Result
System Stability: All Sentinel system dependencies are maintained; no freezes or deadlocks.
Limits Removed: Winlator, Mobox, and AAA games can now consume 4GB, 8GB, or more RAM without being terminated.
Security: If you reboot without Root mode (as in APatch), the system returns to its factory state, ensuring a safe fallback.
World's first successful "Functional Stub" solution for the HyperSentinel issue on Android 16.
