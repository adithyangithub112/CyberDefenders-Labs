## Important things to be know before the investigation:

### what is deserialization and deserialization vulnerability

Deserialization in Java is the process of converting a byte stream (which was created by serialization) back into a fully functional Java object in memory, with its original state and data restored.

A deserialization vulnerability occurs when an application takes untrusted, attacker-controllable serialized data and deserializes it without proper validation, allowing the attacker to execute arbitrary code, manipulate application logic, or cause a denial-of-service. It is a critical security flaw where malicious objects are injected into the application's memory during the reconstruction process.

### **SharePoint**

SharePoint Servers are Microsoft-developed, on-premises software platforms that provide centralized, secure, and customizable environments for enterprise document management, content storage, intranet hosting, and team collaboration. They function as the backbone of an organization's digital workplace, allowing for structured data management, workflows, and search capabilities.

### **Microsoft SharePoint CVE-2025-53770 RCE Vulnerability Explained**

CVE-2025-53770 is a critical unauthenticated remote code execution (RCE) vulnerability affecting Microsoft SharePoint Server 2016, 2019, and Subscription Edition. The vulnerability was discovered during active exploitation in July 2025 and later classified by Microsoft as a variant of the previously demonstrated ToolShell chain (CVE-2025-49706 + CVE-2025-49704) revealed at Pwn2Own Berlin.

The vulnerability allows attackers to send a specially crafted HTTP POST request to the following endpoint:

```bash
/_layouts/15/ToolPane.aspx?DisplayMode=Edit
```

---

The request includes a Referer header pointing to

```bash
/layouts/15/signout.aspx
```

---

This bypasses authentication and SharePoint's form digest validation.

*As a result, the server treats the request as legitimate and processes it under an unauthenticated context.*

Once exploited, attackers can:

- Write malicious .aspx files to disk (e.g., spinstall0.aspx)
- Extract cryptographic secrets such as the SharePoint ValidationKey
- Craft valid, signed __VIEWSTATE payloads
- Achieve full remote code execution using tools like ysoserial

Microsoft confirmed exploitation in the wild and assigned CVE-2025-53770 on July 20, 2025.

### **How does the CVE-2025-53770 RCE Exploit Works?**

> **1. Authentication Bypass via ToolPane.aspx**
> 

The attack begins with an HTTP POST request to SharePoint’s legacy WebPart editor endpoint:

```bash
/_layouts/15/ToolPane.aspx?DisplayMode=Edit
```

---

What makes this request dangerous is its forged Referer header:

```bash
Referer: /_layouts/SignOut.aspx
```

---

This header tricks SharePoint into skipping authentication and form digest checks, likely due to how the platform validates trusted internal workflows. As a result, the attacker is treated as an authenticated user, even though no credentials were supplied.

> **2. Deployment of a Malicious ASPX File**
> 

Once inside, the attacker uploads a malicious .aspx file, typically named spinstall0.aspx,to the SharePoint layouts directory:

```bash
C:\Program Files\Common Files\Microsoft Shared\Web Server 
Extensions\15\TEMPLATE\LAYOUTS\spinstall0.aspx
```

---

This file does not function as a traditional web shell. Instead, it is designed to extract cryptographic secrets from the server’s configuration, including:

- ValidationKey
- DecryptionKey
- Signing algorithm

These values are used by ASP.NET to validate and decrypt __VIEWSTATE payloads.

By leaking these secrets, the attacker gains the ability to generate their own signed payloads that SharePoint will accept and deserialize.

> **3. Remote Code Execution via Malicious __VIEWSTATE**
> 

With the stolen keys, the attacker crafts a signed, malicious __VIEWSTATE token using tools such as ysoserial.net. This payload embeds system commands (e.g., PowerShell), and is sent to another SharePoint page via a GET request:

```bash
GET /_layouts/15/success.aspx?__VIEWSTATE=<malicious_payload>
```

---

Because the token is signed with the server’s actual keys, SharePoint accepts it and deserializes it during page processing. This leads to execution of the embedded command on the server.

Typical command chain observed:

```bash
w3wp.exe → cmd.exe → powershell.exe -EncodedCommand ...
```

---

This achieves full remote code execution under the application pool identity (NT AUTHORITY\IUSR), enabling file access, lateral movement, credential dumping, or persistence.

### **Mitigation Guidance**

Microsoft and CISA recommend the following steps:

- **Patch Immediately:** Apply the July 2025 security updates for SharePoint 2019 and Subscription Edition [1].
- **Enable AMSI in SharePoint:** AMSI blocks payload execution when Defender AV is installed. Full Mode is recommended.
- **Rotate Machine Keys:** Run Update-SPMachineKey via PowerShell or trigger the Machine Key Rotation timer job in Central Admin.
- **Deploy Microsoft Defender:** For alerting and post-exploit detection.
- **Isolate Public-Facing Servers:** If AMSI cannot be enabled, disconnect vulnerable servers until patched.

### Ysoserial (payload generator)

Ysoserial is a cyberattack tool for exploiting Java deserialization vulnerabilities. Ysoserial includes a collection of utilities and property-oriented programming “gadget chains” discovered in standard java and .NET libraries that can, under the right conditions, exploit Java and .NET applications performing unsafe deserialization of objects. Ysoserial’s modules are referred to as payloads. Each payload generates a serialized object which once instantiated, invokes some kind of action.
