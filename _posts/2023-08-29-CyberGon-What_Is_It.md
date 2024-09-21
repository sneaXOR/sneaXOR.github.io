---
title: CyberGon - What is it
date: 2023-08-29 +0200
tags: [Reverse-engineering]
categories: [Write-ups]
---

```plaintext
The admin team has received an executable file and is unsure how to proceed. Can you determine its purpose?
```

## Walkthrough

### Initial File Examination

I started by extracting the contents of the provided ZIP archive. Upon inspecting the executable (`What_Is_It.exe`), I employed the `strings` command to search for any human-readable text within the binary :

```
strings What_Is_It.exe | grep -i auto
```

#### Sample Output :

```
This is a third-party compiled AutoIt script.
NO_AUTO_POSSESS)
.text$lp00AutoItSC
OLEAUT32.dll
```

The output indicated that the executable was scripted using AutoIt, a scripting language for automating the Windows GUI and general scripting.

### Extracting the Embedded AutoIt Script

To decompile the embedded AutoIt script from the executable, I employed the [AutoIt-Ripper](https://github.com/nazywam/AutoIt-Ripper) tool available on GitHub. By following the instructions outlined in the repository, I successfully extracted the original `.au3` script from the `What_Is_it.exe` file.

### Reviewing the Decompiled Code

Upon examining the decompiled script, I noticed an abundance of global variables and functions, many of which appeared to be obfuscated or redundant. Below is a segment of the extracted code for reference:

```
Global Const $VAR_A = 25988
Global $FLAG_PART1 = "0bfusc4t3d"
Global $FLAG_PART2 = "_4ut0_"
Global $FLAG_PART3 = "!t_!"
Global $FLAG_PART4 = "_d0nt_"
Global $FLAG_PART5 = "th!nk"

Run("notepad.exe")
Sleep(1000)
For $i = 1 To 5
    Send("Random text line " & $i & @CRLF)
    Sleep(500)
Next
```

### Streamlining the Script

Identifying that many elements within the script were unnecessary for retrieving the flag, I proceeded to simplify the code. I removed all extraneous variables and functions, isolating only those that were pertinent to the flag's construction:

```
$FLAG_PART1 = "0bfusc4t3d"
$FLAG_PART2 = "_4ut0_"
$FLAG_PART3 = "!t_!"
$FLAG_PART4 = "_d0nt_"
$FLAG_PART5 = "th!nk"
```

### Assembling the Flag

With the relevant variables isolated, I reorganized them to form a coherent flag. By concatenating these parts, the final flag was revealed:

```
CybergonCTF{0bfusc4t3d_4ut0_!t_!_d0nt_th!nk}
```
