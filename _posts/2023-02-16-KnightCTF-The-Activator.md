---
title: KnightCTF 2023- The Activator
date: 2023-02-16 +0200
tags: [Reverse-engineering]
categories: [Write-ups]
---


# License Checker Bypass: A Classic Crackme

Essentially, it is classic crackme, where the main challenge was bypassing a series of checks embedded within the `main()` function. In this particular case, there was a complex set of checks performed on the input, and based on the result, a single boolean flag would be set, indicating whether the input was valid or not.

If the flag (`checks_ok`) was set to true, the program would display the CTF flag. Otherwise, it would simply output a message informing the user that the input was invalid:

```cpp
if (checks_ok) {
  show_flag();
} else {
  std::cout << "Invalid Key." << std::endl;
}
```

## Bypassing the Check

I used NOP instructions to bypass the check, this time I directly modified the program’s flow. Instead of altering the checks, I manually set the instruction pointer (RIP) to jump just before the `show_flag()` function call, effectively skipping all validation.

Here’s how I achieved this using GDB:

```bash
KnightOS License Checker.
Enter KnightOS Activation Key: ^C
Program received signal SIGINT, Interrupt.
0x00007ffff7b52992 in __GI___libc_read ...
(gdb) finish
Run till exit from #0  0x00007ffff7b52992 in __GI___libc_read ...
asdf
...
Value returned is $1 = 5
(gdb) set $rip=0x555555555bce
(gdb) c
Continuing.
```

By manually setting the `RIP` register to the address just before the `show_flag()` function, I was able to bypass all the checks and directly jump to the point where the flag is displayed.

## Flag

After redirecting the instruction pointer, the program outputted the flag:

```
KCTF{Th47_License_ch3cker_w4S_similar_t0_Wind0ws_95_OSR_Activator_Right?}
```

Flagged
