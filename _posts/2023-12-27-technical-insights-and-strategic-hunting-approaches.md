---
layout: post
math: true
title: # Technical Insights and Strategic Hunting Approaches
date: '2023-12-27 8:22:17 +0400'
categories: [Threat Hunting]
tags: [Threat Hunting,UnPacking,Malware Detection,Threat Intelligence,Incident Response]
---

## njRAT: Technical Insights and Strategic Hunting Approaches

### Introduction:

njRAT (also known as Bladabindi). This remote access Trojan (RAT) has been a significant cyber threat since its inception in 2012. Its widespread adoption escalated following the leak of its source code in 2013, making it a favored tool among both sophisticated cybercriminals and novice hackers.

**In 2023, few njRAT variants are circulating in the wild:**

- njRAT Lime Edition
- njRAT 0.7d Green Edition
- njRAT 0.7d Golden Edition

![CS](/assets/20231227190738.png)

![CS](/assets/20231227190754.png)

### Detailed Evolution and Reach of njRAT:

Originating as a tool targeting organizations in the Middle East, njRAT has evolved to deliver its payload via multiple vectors. These include weaponized documents in phishing campaigns, masquerading as legitimate software on file-sharing sites, and drive-by downloads. Once inside a system, njRAT’s capabilities are extensive: keystroke logging, screenshot capturing, password stealing, comprehensive data exfiltration, and commandeering web cameras and microphones.

Another known distribution method was through a compromised website that tricked users into downloading a fake Google Chrome update as we will cover in our sample which in turn installed njRAT malware to the PC. Bladabindi was also featured in spam email campaigns. In this case, it was delivered to potential victims as a malicious attachment.

### In-Depth Technical Analysis and Deobfuscation:

njRAT, built using the Microsoft® .NET framework, can be intricately analyzed using tools like dnSpy and Ollydbg, especially when it's packed 

For this section, let's delve into the technical deconstruction of a recent njRAT sample,  sourced from the [AnyRun](https://app.any.run/tasks/6e3b2477-c1e8-4fd1-8128-4e32eda226d4/) platform. This example will demonstrate the process using dnSpy, a powerful .NET decompiler and debugger.

**DnSpy Unpacking Process:**

Upon examining the main function, it's evident that the .NET application is packed using mpress. This observation is crucial as it helps to understand the mechanism at the application's main entry point post-unpacking. The code structure reveals the steps followed once the application is decompressed and loaded into memory, providing insights into how the application initializes after the unpacking process.

![CS](/assets/20231227192010.png)

By strategically placing a breakpoint at `_.mp = Assembly.Load(rawAssembly);` within the DnSpy debugger, we can intercept the moment just before the actual executable is loaded into memory. This step is crucial as it enables us to access the `byte[] rawAssembly` variable, which holds the decrypted assembly. Once the breakpoint is hit, we can inspect the local variables to locate this array. By doing so, we effectively capture the unpacked version of the .NET application, which can then be saved to a local location for in-depth analysis. This method is essential for revealing the true nature of the executable concealed by mpress packing.


![CS](/assets/20231227192717.png)

Right-click and save this array to effectively dump the deobfuscated assembly

![CS](/assets/20231227193001.png)

The malware’s functionality is contained in the “OK” class. This class also contains the hardcoded configuration information used by the RAT. For example, the configuration for this particular njRAT sample indicates that it was created with the njRAT builder version 0.7 Golden edition

![CS](/assets/20231227193819.png)

While some configuration details in the njRAT sample may still be obfuscated, these can be unraveled by using the debugger to trace encoding functions or decryption routines. By carefully following the execution flow within the debugger, any obscured or encrypted configuration settings can be effectively revealed and analyzed.

![CS](/assets/20231227194407.png)

**Persistence:** 

For persistence, the malware creates an autorun key in the registry to execute with each boot of the system.

![CS](/assets/20231227200746.png)

**C2 Connection** 

![CS](/assets/20231227202145.png)

As highlighted earlier, **njRAT offers a classic set of functionality for remote-access trojans**

Its capabilities become evident through the Control and Command (C2) panel, which provides a comprehensive suite of functions for system manipulation and surveillance. Here's a detailed look at each function:

|Function Name|Description|
|---|---|
|**Manager**|Enables the downloading and uploading of files to and from the infected system. |
|**Run file**|Likely used to execute files previously uploaded via the Manager |
|**Remote Desktop**|Initiates a Remote Desktop Protocol (RDP) connection to the infected computer. |
|**Remote Cam**|Activates the camera on the infected device, if available. |
|**Microphone**|Turns on the microphone for audio monitoring. |
|**Get Passwords**|Extracts passwords from various desktop applications. |
|**Keylogger**|Records and logs keystrokes, saving them to a .txt file. |
|**Open Chat**|Opens a chat window with the infected PC |
|**Server**|Manages the status of a bot, including removal commands. |
|**Open Folder**|Permits remote access to view the contents of folders on the infected machine. |

### Strategies for Hunting and Mitigating njRAT:

1. **Network Monitoring:** Look for unusual outbound traffic, especially to known njRAT C2 servers.
2. **Anomaly Detection:** Implement heuristic-based detection to identify atypical behavior in systems that could indicate RAT activity.
3. **Regular Audits:** Conduct frequent system audits for unfamiliar registry keys, files, and processes.
4. **Sandbox Analysis:** Use sandbox environments to analyze suspected njRAT files safely.
5. **Employee Training:** Educate staff on identifying phishing attempts and secure file handling.
6. consider the relevant data endpoint telemetry sources such as:
	- Process Execution & Command Line Logging
	- Windows Security SACL Event ID, Sysmon, or any Common Information Model compliant EDR technology
	- Windows Security Event Log
	- Windows System Event Log
	- Windows PowerShell Script Block Logging

### Conclusion:

The sophisticated nature of njRAT demands constant vigilance and up-to-date cybersecurity measures. Understanding its technical intricacies and employing proactive hunting strategies are key to defending against this enduring cyber threat. As njRAT continues to evolve, staying abreast of the latest trends and protective measures remains crucial in the fight against this formidable malware
