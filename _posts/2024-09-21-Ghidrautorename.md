---
title: Automating Ghidra Function Names
date: 2024-09-21 +0200
tags: [Reverse-engineering]
categories: [Reverse-engineering]
---

# Why I Built Ghidrautorename: Solving a Reverse Engineering Pain Point

During a recent reverse engineering session using Ghidra, I hit a roadblock: renaming functions based on strings manually was just too time-consuming. After dealing with function names like `FUN_00123` for hours, I thought, “why not automate this process?” And thus, **Ghidrautorename** was born.

## The Challenge with Ghidra’s API

One thing about Ghidra: it's based on Jython, which uses an older version of Python (series 2.x). This can limit some of Python's modern features and libraries, making scripting a little more difficult. But despite these obstacles, I stuck with Ghidra and tried using ghidra_bridge, a Python 3 bridge to Ghidra's Python scripting tool.

As the data is poorly encapsulated between the ghidra API and the bridge, I switched to the Pyhidra library, which provides direct access to the Ghidra API.

## What Ghidrautorename Does

**Ghidrautorename** scans the binary for strings, detects potential function names, and automatically renames the functions in Ghidra based on those findings. It’s all about making the code easier to read and analyze.

### Why Automate This?

Renaming functions manually takes ages, and this tool helps save time by automating a pretty tedious task.

### Example Output

Here’s an example of the tool in action:

```bash
Total strings extracted: 15

[+] Potential function name found: 'processRequest'
[+] Successfully renamed function at 0x08048b12 to 'processRequest'

[+] Potential function name found: 'handleConnection'
[+] Successfully renamed function at 0x08048bc4 to 'handleConnection'

==================================================
Summary of Renamed Functions
==================================================

Function at 0x08048b12 renamed to 'processRequest'
Function at 0x08048bc4 renamed to 'handleConnection'

==================================================
Renaming Process Complete!
==================================================
```

Probably not groundbreaking, but it saves manual work. Ghidrautorename is here to simplify the function renaming process during reverse engineering.

## Using Ghidrautorename

All the details can be found in the github repo : [Ghidrautorename](https://github.com/sneaXOR/Ghidrautorename)

Enjoy, and if you try it out, feel free to let me know how it goes! 👾
