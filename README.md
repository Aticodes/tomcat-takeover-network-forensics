# Apache Tomcat Takeover – Network Forensics Investigation

## Overview

This project documents my investigation of the **Tomcat Takeover** challenge from CyberDefenders. The objective was to analyze a packet capture (PCAP) file using **Wireshark** to determine how an attacker compromised an Apache Tomcat web server, identify the attacker's techniques, and collect Indicators of Compromise (IOCs).

This investigation was completed as part of my cybersecurity learning journey while preparing for a SOC Analyst role.

---

## Scenario

The SOC team identified suspicious activity on an Apache Tomcat web server within the organization's internal network. A network packet capture (PCAP) was provided for analysis to determine the scope of the compromise.

The goal of this investigation was to reconstruct the attack by analyzing network traffic and identifying the attacker's actions.

---

## Objectives

- Analyze HTTP network traffic
- Identify the attacker's IP address
- Determine the country of origin
- Identify the enumeration tool used
- Discover the exposed administrative interface
- Analyze authentication attempts
- Identify the uploaded reverse shell
- Determine the persistence mechanism
- Collect Indicators of Compromise (IOCs)

---

## Tools Used

- Wireshark
- CyberDefenders
- IP Geolocation Lookup

---

# Investigation Methodology

The investigation followed a structured process similar to a real-world network forensic investigation.

### Step 1 – Initial Traffic Review

- Opened the PCAP file in Wireshark
- Reviewed the packet list
- Identified HTTP as the primary protocol
- Examined packet conversations to identify suspicious hosts

---

### Step 2 – Identify the Attacker

Using Wireshark filters, I identified the external IP communicating with the web server.

Example filters used:

```
http
ip.src == 14.0.0.120 && http
```

The attacker's IP address was then checked using an IP geolocation service.

---

### Step 3 – Port Discovery

Analyzed HTTP requests directed at different ports to determine which service exposed the Tomcat administrative interface.

The investigation revealed that the Tomcat Manager application was running on **port 8080**.

<img width="752" height="397" alt="Picture6" src="https://github.com/user-attachments/assets/36517b61-1ff0-426c-9013-2c651c236810" />

---

### Step 4 – Directory Enumeration

To identify how the attacker discovered hidden directories, I filtered HTTP responses returning **404 Not Found**.

Filter used:

```
http.response.code == 404
```

Using **Follow HTTP Stream**, I examined the requests and discovered the **User-Agent** revealing the enumeration tool.

**Enumeration Tool**

- Gobuster

This indicates the attacker was brute-forcing directories and files on the web server.

---

### Step 5 – Administrative Interface Discovery

Further HTTP analysis revealed the attacker successfully discovered the Tomcat administration panel.

Discovered directory:

```
/manager
```

---

### Step 6 – Authentication Analysis

Using the filter below, I identified HTTP Basic Authentication traffic.

```
http.authbasic
```

This allowed analysis of authentication attempts and the credentials successfully used to access the Tomcat Manager application.

---

### Step 7 – Reverse Shell Upload

After successful authentication, HTTP POST requests were analyzed to identify the malicious file uploaded by the attacker to establish remote access.

The uploaded file was extracted from the HTTP request.

<img width="940" height="184" alt="image" src="https://github.com/user-attachments/assets/e10fcf99-1bf0-48c9-bd17-37ee7be8d701" />


---

### Step 8 – Persistence

Finally, the attacker's commands were examined to identify the persistence mechanism used after obtaining a reverse shell.

<img width="752" height="353" alt="command-execution" src="https://github.com/user-attachments/assets/cb4ecd6f-42c2-42b2-916f-f3a88b1fba97" />

---

# Indicators of Compromise (IOCs)

| Indicator | Value |
|-----------|-------|
| Attacker IP | 14.0.0.120 |
| Country | China |
| Enumeration Tool | Gobuster |
| Admin Directory | /manager |
| Admin Port | 8080 |

---

# Wireshark Filters Used

| Filter | Purpose |
|---------|---------|
| `http` | Display HTTP traffic |
| `ip.src == <IP> && http` | Show HTTP requests from attacker |
| `http.response.code == 404` | Find directory enumeration attempts |
| `http.authbasic` | Identify HTTP Basic Authentication |
| `http.request.method == "POST"` | Locate uploaded files |
| `ip.addr == <IP>` | Show all traffic involving a specific host |

---

# Key Findings

- The attacker performed reconnaissance against the web server.
- Gobuster was used to enumerate hidden directories.
- The attacker discovered the Tomcat Manager application.
- Authentication attempts were observed using HTTP Basic Authentication.
- A malicious file was uploaded to establish a reverse shell.
- Persistence was established after compromising the server.

---

# Skills Practiced

- Network Traffic Analysis
- Packet Inspection
- HTTP Analysis
- Wireshark Filters
- Follow HTTP Stream
- Basic Authentication Analysis
- Directory Enumeration Detection
- IOC Identification
- Network Forensics Methodology

---

# Challenges Faced

During the investigation I encountered several challenges:

- Understanding Wireshark's Statistics window.
- Learning the correct syntax for combining filters using `&&`.
- Distinguishing between HTTP requests and HTTP responses.
- Identifying the correct packet containing the uploaded reverse shell.

Working through these challenges significantly improved my confidence using Wireshark.

---

# Lessons Learned

Through this investigation I learned:

- How to investigate HTTP traffic using Wireshark.
- How attackers enumerate web servers.
- How to identify tools through the User-Agent field.
- How HTTP Basic Authentication appears in packet captures.
- How to follow HTTP streams to reconstruct attacker activity.
- The importance of collecting evidence before drawing conclusions.

This project strengthened my understanding of network forensics and provided practical experience investigating a web server compromise.

---

# MITRE ATT&CK Mapping

| Technique | Description |
|-----------|-------------|
| Active Scanning | Discovery of open ports and services |
| Gather Victim Network Information | Enumeration of the target |
| Valid Accounts | Successful authentication |
| Server Software Component | Apache Tomcat exploitation |
| Command and Scripting Interpreter | Reverse shell execution |
| Persistence | Scheduled task / command execution |

---

# References

- CyberDefenders – Tomcat Takeover
- Wireshark Documentation
- MITRE ATT&CK Framework

---

> **Disclaimer:** This investigation was performed in a controlled laboratory environment provided by CyberDefenders for educational purposes only.
