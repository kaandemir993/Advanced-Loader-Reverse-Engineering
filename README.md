# From Shellhost to 3 DLLs: Analyzing an Advanced Loader with Payload Injection, CFG Bypass, and Downgrade Attacks

## Overview
This analysis covers a sophisticated multi-phase loader targeting `Shellhost.exe`, `amsi.dll`, `mstscax.dll`, and `clbcatq.dll`. The loader uses module stomping, CFG bypass, COM hijacking, and downgrade attacks.


## Shellhost – Primary Target for Payload Injection

The loader targets `Shellhost.exe` as its primary process for payload injection. It uses **module stomping** to overwrite the legitimate code of Shellhost with its own malicious payload.

### Key Observations
- **Obfuscated payload:** The injected code contains ASCII remnants (`push 70207369h` = "is p") and invalid instructions (`jb`, `outs`, `ins`).
- **`fothk` section:** A custom section filled with INT3 traps and a single `jmp` to the CFG bypass routine.
- **CFG manipulation:** Redirects execution to the final shellcode.
- 
*WinDbg view of the obfuscated payload in Shellhost.exe.*

![Shellhost Payload](images/shellhost_payload.jpg)

## amsi.dll – CFG Bypass & INT3 Traps

The loader injects a custom section into `amsi.dll` filled with `INT3` (0xCC) anti-disassembly traps. A single `jmp` redirects to a **Control Flow Guard (CFG)** bypass routine, which jumps to the final shellcode via `jmp rax`.

### Key Observations
- **INT3 traps:** The section is almost entirely filled with `INT3` instructions.
- **Single `jmp`:** `jmp 0x00007ffbc85060c0` leads to the CFG bypass routine.
- **CFG bypass:** Uses a custom bitmask to validate and redirect execution.

![AMSI CFG Bypass](images/amsi_cfg_bypass.jpg)

*WinDbg view of INT3 traps and the single jmp instruction.*


## mstscax.dll – Module Stomping & LocalAlloc

The loader uses `LocalAlloc` to manually allocate memory for the payload. A custom structure is filled with a file handle and vtable assignment for COM hijacking.

### Key Observations
- **`LocalAlloc(0x40, 0x68)`:** Allocates 104 bytes of memory.
- **Vtable assignment:** `*puVar12 = &PTR_FUN_180699290` for COM hijacking.
- **File handle usage:** `puVar12[4] = hFile` suggests the loader reads from a file.

![mstscax LocalAlloc](images/mstscax_localalloc.jpg)

*Ghidra view of LocalAlloc usage and manual struct allocation.*


# clbcatq.dll – Kernel-Aware Manipulation & Downgrade Attack

The loader uses a fixed memory address (`lRam0000000000000000`) and vtable calls to manipulate kernel objects. This technique is used for privilege escalation.

### Key Observations
- **Fixed memory address:** `lRam0000000000000000` used for kernel object manipulation.
- **Vtable calls:** `+0x18`, `+0x40`, `+0x60` indicate kernel-level operations.
- **Privilege escalation:** Used to gain SYSTEM-level privileges.
- **COM hijacking:** Uses `FUN_180031f18` and `FUN_1800330c8`.

![clbcatq Kernel](images/clbcatq_kernel.jpg)

*Ghidra view of fixed memory address and vtable calls.*


## Indicators of Compromise (IOCs)
- **Registry Key:** `HKLM\Software\Microsoft\COM3\RemoteAccessEnabled`
- **DLLs Targeted:** `amsi.dll`, `mstscax.dll`, `clbcatq.dll`
- **Process Targeted:** `Shellhost.exe`
- **Custom Sections:** `fothk`
- **Anti-Disassembly:** INT3 traps (0xCC)

## Conclusion
This loader demonstrates advanced evasion techniques, including module stomping, CFG bypass, COM hijacking, and downgrade attacks. It targets multiple system components to ensure stealth and persistence.

## Author
**StructBreaker** – Malware Analyst & Reverse Engineer



