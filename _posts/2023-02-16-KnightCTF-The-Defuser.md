---
title: KnightCTF 2023 - The Defuser
date: 2023-02-16 +0200
tags: [Reverse-engineering]
categories: [Write-ups]
---

# The Defuser: Bomb Defusal Challenge

In this challenge, the goal is to defuse a bomb within a limited timeframe. During execution, the challenge sets certain signals that will terminate the process if not defused quickly.

## Execution Flow

Upon running the binary, the following sequence occurs:

```bash
bsdb0y@test:~/challs/knight/reverse$ ./defuser

Time is ticking...

Defuse the bomb real quick if you want to save your Imaginarica!
I bet you won't be able to save it...
Hahahahaha...

Convince me to stop the explosion:
```

If you fail to input the correct response within the time limit, this is what happens:

```bash
Hahahahaha LOSER!
The bomb has exploded.
You shouldn't have wasted your time.
Remember, every second is valuable.
Terminated
```

## Analyzing the Binary

After inspecting both the disassembly and decompiled code, I observed the following key elements:

- Signal and alarm calls that initiate the time limit
- A critical parameter set to `0x17b447b`
- Specific environment variables that need to be set:
  - `LAB_HOSTNAME=DEFUSER_PC`
  - `LAB_USERNAME=FRITZ`

## Exploit Strategy

The key to solving this challenge lies in bypassing the signals and alarms. Using a debugger like `gdb` or `radare2`, load the binary, set the environment variables as mentioned above, and disable the alarm/signal calls.

Once you've done that, provide input to the program and step into the function located at address `0x2760`. The function expects its first argument to be `0x17b447b`. If everything is correct, the bomb will be defused, and the flag will be revealed.

### GDB Example

```bash
[0x5610de303596]> dc
```

## Flag

Successfully bypassing the time constraints and entering the correct input will yield the following output:

```bash
You have saved the country from destruction.

Here is your reward:
KCTF{st0p_war_spr34d_10v3_and_p34c3}

(3995) Process exited with status=0x0
```

By carefully analyzing the binary and skipping the signals, we were able to prevent the explosion and retrieve the flag.
