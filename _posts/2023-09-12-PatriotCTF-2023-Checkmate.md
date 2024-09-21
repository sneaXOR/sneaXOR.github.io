---
title: PatriotCTF-2023 - Checkmate
date: 2023-09-12 +0200
tags: [Web Security, Credential Bypass]
categories: [Write-ups]
---

Upon loading the webpage, the first thing you see is a login form. Naturally, the next step is to inspect the page source for clues.

![login](assets/Checkmate/checkmate_login.png)

### Source Code Analysis

By checking the JavaScript in the page source, we uncover how the login credentials are validated.

![code](assets/Checkmate/checkmate_source.png)

The code relies on three key functions:

- **checkName()**
  - Verifies if the username is the reverse of "uyjnimda".
  - Running the command `echo uyjnimda | rev` gives us the username: `adminjyu`.

- **checkLength()**
  - Ensures that the length of the password is divisible by 6.

- **checkValidity()**
  - This function processes the password input in chunks of six characters.
  - Around line 209, we see the line `Array.from(password).map(ok)` which transforms the input into an array of character codes between [97, 122] (i.e., lowercase letters).
  - The following conditions need to be satisfied for the password to be valid:
    - `input[0] & input[2] == 0x60`: Two characters must share bits in common with `0x60`. Characters like `'a' = 0x61` and `'b' = 0x62` fit this condition.
    - `input[1] | input[4] == 0x61`: This uses an OR operation with the character `'a' = 0x61`.
    - `input[3] ^ input[5] == 0x06`: We can use XOR to reverse-engineer the characters. If we choose `'a'` randomly, solving `a ^ 0x06 = 0x67` results in the character `'g'`.

By using these hints, we can utilize the ASCII table and a hex calculator to construct a valid password.

### Solving the Challenge

From the source code analysis, we now have the correct login credentials:

- **Username**: `adminjyu`
- **Password**: `aabaag`

After examining the page source further, we notice a comment referencing `/check.php` on line 224. This endpoint processes a POST request with the password as a parameter, but it doesn't accept the initial password.

### Generating Passwords

Since there are multiple potential passwords that could satisfy the conditions, brute-forcing manually isn't ideal.

The strategy here is to generate all possible valid passwords programmatically and perform a dictionary attack, I write a little python script to do this.

![script](assets/Checkmate/checkmate_password.png)

### Flag

With the correct password generated, submitting it reveals the flag:

![login](assets/Checkmate/checkmate_flag.png)

```
PCTF{Y0u_Ch3k3d_1t_N1c3lY_149}
```

