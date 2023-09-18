---
layout: post
title: Code Obfuscation
date: '2023-09-18 19:22:41 +0400'
categories: [Threat Hunting]
tags: [Obfuscation,Code Analysis]
---

# Code Obfuscation: Detection and Mitigation

Code obfuscation refers to the deliberate act of making source code difficult to read and understand. It's a technique frequently used by malicious actors to hide their intentions and to avoid detection by anti-malware software. Obfuscated code can hide malicious activities, disguise algorithm logic, or evade pattern-based detection mechanisms.

Let's examine a simple demonstration below. Oddly enough, the provided codes appear almost identical. There seems to be a minor oversight in providing differentiated examples. But let's dive into what obfuscation might look like using the given example.

## Normal Code:

```vb
Set objShell = CreateObject("WScript.Shell")
objShell.Run "powershell.exe -Command & {Start-Process calc}", 0
Set objShell = Nothing
```

This is a straightforward piece of VBScript code that creates an object to run a command and opens the Windows calculator using PowerShell.

## Obfuscated Code

```vb
Dim s1, s2, s3, s4, s5, x, y
s1 = Chr(112) & Chr(111) & Chr(119) & Chr(101) & Chr(114) & Chr(115) & Chr(104) & Chr(101) & Chr(108) & Chr(108)
s2 = Chr(46) & Chr(101) & Chr(120) & Chr(101)
s3 = Chr(32) & Chr(45) & Chr(67) & Chr(111) & Chr(109) & Chr(109) & Chr(97) & Chr(110) & Chr(100) & Chr(32) & Chr(38) & Chr(32) & Chr(123) & Chr(83) & Chr(116) & Chr(97) & Chr(114) & Chr(116) & Chr(45) & Chr(80) & Chr(114) & Chr(111) & Chr(99) & Chr(101) & Chr(115) & Chr(115) & Chr(32) & Chr(99) & Chr(97) & Chr(108) & Chr(99) & Chr(125)
s4 = Chr(87) & Chr(83) & Chr(99) & Chr(114) & Chr(105) & Chr(112) & Chr(116) & Chr(46) & Chr(83) & Chr(104) & Chr(101) & Chr(108) & Chr(108)
Set x = CreateObject(s4)
y = s1 & s2 & s3
x.Run y, 0
Set x = Nothing
```

This VBScript code provides an example of how obfuscation can be done using the Chr function, which returns the string character for a given ASCII value.

The primary purpose of this code is to execute the Windows calculator (calc.exe) using PowerShell, while obfuscating the actual intent through character-by-character string construction.

Let's break down the obfuscation step-by-step:

```vb
Dim s1, s2, s3, s4, s5, x, y
```
Variables are declared.


```vb
s1 = Chr(112) & Chr(111) & Chr(119) & Chr(101) & Chr(114) & Chr(115) & Chr(104) & Chr(101) & Chr(108) & Chr(108)
```
s1: Constructs the string "powershell".

```vb
s2 = Chr(46) & Chr(101) & Chr(120) & Chr(101)
```
s2: Appends the extension ".exe" to form the executable name.

```vb
s3 = Chr(32) & Chr(45) & Chr(67) & Chr(111) & Chr(109) & Chr(109) & Chr(97) & Chr(110) & Chr(100) & Chr(32) & Chr(38) & Chr(32) & Chr(123) & Chr(83) & Chr(116) & Chr(97) & Chr(114) & Chr(116) & Chr(45) & Chr(80) & Chr(114) & Chr(111) & Chr(99) & Chr(101) & Chr(115) & Chr(115) & Chr(32) & Chr(99) & Chr(97) & Chr(108) & Chr(99) & Chr(125)
```
s3: Forms the command-line argument " -Command & {Start-Process calc}".

```vb
s4 = Chr(87) & Chr(83) & Chr(99) & Chr(114) & Chr(105) & Chr(112) & Chr(116) & Chr(46) & Chr(83) & Chr(104) & Chr(101) & Chr(108) & Chr(108)
```
s4: Constructs the string "WScript.Shell", which will be used to create an object for executing the command.

```vb
Set x = CreateObject(s4)
```
Creates an object x with the reference "WScript.Shell".

```vb
y = s1 & s2 & s3
```
y forms the complete command: "powershell.exe -Command & {Start-Process calc}".

```vb
x.Run y, 0
```
Executes the concatenated command.

```vb
Set x = Nothing
```
Clean-up: Setting the object x to nothing, releasing its resources.

## Detection of Obfuscated Code
- **Static Analysis:** Look for patterns, such as unnecessary string concatenations, excessive use of variables for simple tasks, and encoded or encrypted strings.

- **Heuristics:** Use heuristic rules to detect unusual code structures. For instance, multiple operations that can be simplified into one might be suspicious.

- **Entropy Analysis:** High entropy indicates randomness, which may suggest encoded or encrypted code.

- **Syntax Highlighting:** By using syntax highlighting in your code editor, you can often spot unusual patterns more easily.

- **Sandboxing:** Execute the code in a controlled environment to observe its behavior. This can help identify if it's performing any unexpected or malicious operations.

## Mitigation Techniques
- **Keep Software Updated:** Ensure all software and anti-malware solutions are up-to-date. Vendors regularly release updates to counter new obfuscation techniques.

- **User Education:** Educate users not to run untrusted scripts or download attachments from unknown sources.

- **De-obfuscation Tools:** Some tools can reverse common obfuscation techniques, revealing the original code.

- **Script Execution Policies:** Restrict or monitor script execution, especially in sensitive environments.

- **Endpoint Protection Solutions:** Employ modern endpoint protection solutions that utilize behavior-based detection methods.

## Recent Trends and Insights
While the provided scripts don't showcase advanced obfuscation techniques, many Advanced Persistent Threat (APT) groups have been known to employ sophisticated obfuscation methods to evade detection.

Historically, groups such as APT29 (Cozy Bear) and APT28 (Fancy Bear) have been observed using obfuscated PowerShell scripts in their cyber-espionage campaigns.

In recent years, a trend has been observed where attackers leverage native Windows tools, like PowerShell, in their attacks due to its powerful capabilities and the ability to run scripts directly in memory, evading disk-based detection.
