---
title: KnightCTF 2023 - Help_Jimmy
date: 2023-02-16 +0200
tags: [Reverse-engineering]
categories: [Write-ups]
---


# Help Jimmy: A Reverse Engineering Journey

In this challenge, the binary `Help Jimmy` was provided. It was a 64-bit Linux ELF executable, simulating a basic game where the player could choose between two paths: exploring the Jungle or the Sea. Regardless of the player's choice, Jimmy would end up facing either tigers or pirates, but ultimately, the flag would remain elusive.

Upon deeper inspection, it became clear that the game was nothing more than a diversion. The real crux of the challenge lay in a small segment of code within the `main()` function:

```c
int x = 5;
int y = 5;
if (x != y) {
  some_function();
}
```

This section might seem insignificant at first glance, but it is crucial. When reviewing the assembly output, this comparison is easily recognizable, and it's followed by a function call if the condition holds. However, some decompilers, such as Ghidra, tend to optimize away this branch, making it invisible in the decompiled code. As a result, it's easy to miss the function call entirely when relying solely on the decompiler view.

!["ghidrahelpjimmy"](assets/HelpJimmy/helpjimmy.png)

## Bypassing the Check

This challenge is a typical "NOP the check" reverse engineering problem, reminiscent of classical crackme challenges. The simplest solution is to either modify the comparison or NOP out the check entirely. In this case, I opted to NOP the instructions directly using GDB. Here’s how I approached it:

```bash
break *0x0000555555555629
r
set *(unsigned char*)0x555555555653 = 0x90
set *(unsigned char*)0x555555555654 = 0x90
c
```

By setting the two comparison-related bytes to `0x90` (the NOP instruction in x86 architecture), the check is effectively skipped, allowing us to progress in the binary.

## Flag

After bypassing the comparison, the program continued execution, and the flag was revealed:

```
KCTF{y0u_may_ch00s3_to_look_7h3_other_way_but_y0u_can_n3v3r_say_4gain_that_y0u_did_n0t_know}
```

Flagged
