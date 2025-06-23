# Task 1: Scan Your Local Network for Open Ports

## 📋 Task Overview
**Objective**: Learn to discover open ports on devices in your local network to
 understand network exposure.

**Tools Used**: Nmap, Wireshark, Netdiscover, DVWA as Target Server (Within the same LAN).

**I Used DVWA as a target server in my network to perform all the attacks on... and demonstrated tasks with the same...**

**Date**: June 23, 2025  

---

## 🎯 Learning Objectives
- Understand network topology and subnet identification
- Learn to use network scanning tools (Nmap, netdiscover)
- Identify active hosts on local network
- Discover open ports and running services
- Analyze network security posture

---

## 🔧 Tools Used
- **Kali Linux** - Primary penetration testing distribution
- **Nmap** - Network exploration and security auditing tool
- **netdiscover** - Active/passive ARP reconnaissance tool
- **ifconfig** - Network interface configuration utility

---

## 📊 Network Environment Analysis

### 1. Network Interface Configuration
First, I analyzed the network configuration of the Kali Linux system:

```bash
ifconfig
```

**Key Findings**:
- **Interface**: eth0 (Ethernet interface)
- **IP Address**: 192.168.192.128
- **Netmask**: 255.255.255.0 (/24)
- **Broadcast**: 192.168.192.255
- **Network Range**: 192.168.192.0/24

### Understanding Subnet Calculation
From the netmask `255.255.255.0`:
- **Binary**: 11111111.11111111.11111111.00000000
- **CIDR Notation**: /24 (24 bits for network, 8 bits for hosts)
- **Total Hosts**: 2^8 - 2 = 254 usable IP addresses
- **Network Range**: 192.168.192.1 - 192.168.192.254

---

## 🔍 Host Discovery Phase

### 2. Active Host Discovery with netdiscover
```bash
netdiscover -r 192.168.192.0/24
```

**Command Breakdown**:
- `netdiscover`: ARP-based network discovery tool
- `-r`: Specify range to scan
- `192.168.192.0/24`: Target subnet in CIDR notation

**Results Summary**:
| IP Address | MAC Address | Vendor | Packet Count | Data Size |
|------------|-------------|---------|--------------|-----------|
| 192.168.192.131 | 00:0c:29:bd:c5:69 | VMware, Inc. | 3 | 180 bytes |
| 192.168.192.1 | 00:50:56:c0:00:08 | VMware, Inc. | 25 | 1500 bytes |
| 192.168.192.254 | 00:50:56:f5:28:af | VMware, Inc. | 2 | 120 bytes |
| 192.168.192.2 | 00:50:56:fd:b0:a9 | VMware, Inc. | 1 | 60 bytes |

**Analysis**:
- **Total Active Hosts**: 4 devices discovered
- **Environment**: VMware virtualized network
- **Gateway**: Likely 192.168.192.1 (highest packet count)
- **Target Systems**: Multiple VMware virtual machines

---

## 🎯 Port Scanning Phase

### 3. Comprehensive Port Scanning with Nmap
```bash
nmap -sV -sC -sS 192.168.192.0/24
```

**Command Breakdown**:
- `nmap`: Network mapper tool
- `-sV`: Version detection (identify service versions)
- `-sC`: Default script scan (run default NSE scripts)
- `-sS`: SYN stealth scan (half-open scanning)
- `192.168.192.0/24`: Target network range

---

## 📈 Detailed Scan Results

### Host 1: 192.168.192.1 (Network Gateway)
```
Host Status: Up (0.0080s latency)
Filtered Ports: 997 (no-response)
```

**Open Ports**:
- **135/tcp**: Microsoft Windows RPC
- **139/tcp**: NetBIOS Session Service  
- **445/tcp**: Microsoft SMB/CIFS

**System Information**:
- **OS**: Windows
- **MAC Address**: 00:50:56:c0:00:08 (VMware)
- **NetBIOS Info**: 
  - Computer Name: HP
  - User: \<unknown>
  - MAC: 00:50:56:c0:00:08

### Host 2: 192.168.192.2 (DNS Server)
```
Host Status: Up (0.001s latency)
Closed Ports: 999 (reset)
```

**Open Ports**:
- **53/tcp**: DNS (dnsmasq 2018.a)

**System Information**:
- **Service**: DNS resolution service
- **Software**: dnsmasq version 2018.a
- **MAC Address**: 00:50:56:FD:B0:A9 (VMware)

### Host 3: 192.168.192.131 (Target System)
```
Host Status: Up (0.0001s latency)
Filtered Ports: 879 (no-response)
Closed Ports: 116 (reset)
```

**Open Ports**:
- **21/tcp**: FTP (vsftpd - login disabled)
- **22/tcp**: SSH (OpenSSH)
- **80/tcp**: HTTP (Apache httpd)
- **443/tcp**: HTTPS (Apache httpd)
- **3306/tcp**: MySQL (MariaDB)

**System Information**:
- **MAC Address**: 00:0C:29:BD:C5:69 (VMware)
- **Services**: Web server with database backend
- **Security Note**: Multiple services exposed

### Host 4: 192.168.192.254
```
Host Status: Up (0.0003s latency)
All 1000 ports: Filtered (ignored states)
```

**Analysis**: Highly secured system with firewall blocking all probe attempts.

---

## 🌐 HTTP Traffic Interception Analysis

### 4. Network Traffic Capture with Wireshark
After identifying the web services running on port 80 (HTTP) on host 192.168.192.128, I performed network traffic analysis to demonstrate the security risks of unencrypted HTTP communications.

**Tools Used**:
- **Wireshark** - Network protocol analyzer
- **Target**: HTTP login form on 192.168.192.128

### HTTP Login Credential Interception
```bash
# Started Wireshark capture on eth0 interface
# Accessed HTTP login page via browser
# Submitted login credentials through insecure form
```

**Captured Traffic Analysis**:
From the Wireshark capture, I was able to identify the following:

**HTTP POST Request Details**:
- **Source IP**: 192.168.192.128
- **Destination IP**: 65.61.137.117
- **Protocol**: HTTP/1.1
- **Method**: POST
- **Content-Type**: application/x-www-form-urlencoded

**Intercepted Credentials**:
```
Form Data (Plain Text):
- Username Field: "admin"
- Password Field: "admin" 
- Submit Action: "Login"
```

**Security Implications**:
1. **Credentials Transmitted in Clear Text**: Login credentials are visible in plain text within the HTTP request
2. **No Encryption**: HTTP traffic is unencrypted and easily intercepted
3. **Network Eavesdropping**: Anyone monitoring network traffic can capture sensitive information
4. **Man-in-the-Middle Attacks**: Attackers can intercept and modify HTTP traffic

### Key Observations from Traffic Capture
- **Packet Analysis**: HTTP form data clearly shows username "admin" and password "admin"
- **Request Headers**: Standard HTTP headers with no security mechanisms
- **Form Encoding**: URL-encoded form data transmission
- **Network Path**: Traffic traverses multiple network hops without protection

### Recommendations for Secure Communication
1. **Implement HTTPS**: Use SSL/TLS encryption for all authentication
2. **Password Hashing**: Never transmit passwords in plain text
3. **Secure Headers**: Implement security headers (HSTS, CSP)
4. **Network Segmentation**: Isolate sensitive communications
5. **VPN Usage**: Encrypt network traffic at the transport layer

---

## 🚨 Combined Security Assessment Update

### Additional Vulnerabilities Identified
4. **Insecure HTTP Authentication** (192.168.192.128)
   - Login credentials transmitted in plain text
   - No SSL/TLS encryption implemented
   - Susceptible to network eavesdropping
   - Credentials: admin/admin (weak default credentials)

### Enhanced Security Recommendations
4. **Implement Secure Authentication**
   - Force HTTPS for all login pages
   - Use strong password policies
   - Implement multi-factor authentication
   - Regular credential rotation

---

## 🔐 Security Assessment

### Identified Vulnerabilities
1. **SMB/NetBIOS Exposure** (192.168.192.1)
   - Ports 139, 445 open
   - Potential for SMB enumeration attacks
   - Anonymous access possible

2. **Multiple Service Exposure** (192.168.192.131)
   - Web services (80, 443)
   - Database service (3306)
   - SSH access (22)
   - FTP service (21)

3. **Information Disclosure**
   - NetBIOlogS name enumeration successful
   - Service version information exposed
   - Network topology easily mapped

### Security Recommendations
1. **Implement Network Segmentation**
   - Isolate critical services
   - Use VLANs for separation

2. **Service Hardening**
   - Disable unnecessary services
   - Update service versions
   - Implement access controls

3. **Firewall Configuration**
   - Block unused ports
   - Implement intrusion detection

---

## 📚 Technical Learning Points

### ARP Discovery Process
- **ARP Requests**: Broadcast packets to discover active hosts
- **MAC Address Mapping**: Links IP addresses to physical addresses
- **Network Topology**: Understanding switch/router infrastructure

### Port Scanning Methodology
- **SYN Scanning**: Stealth technique avoiding full TCP handshake
- **Service Detection**: Identifying running applications
- **Script Scanning**: Automated vulnerability detection

### Network Analysis Skills
- **Subnet Calculation**: Understanding CIDR notation
- **Service Enumeration**: Identifying potential attack vectors
- **Risk Assessment**: Evaluating security posture

### Network Traffic Analysis
- **Packet Inspection**: Understanding HTTP request/response structure
- **Protocol Analysis**: Identifying security weaknesses in network protocols
- **Credential Harvesting**: Demonstrating how attackers can capture sensitive data
- **Network Monitoring**: Using packet capture tools for security assessment

### HTTP vs HTTPS Security
- **Plain Text Transmission**: HTTP sends all data unencrypted
- **SSL/TLS Protection**: HTTPS encrypts data in transit
- **Certificate Validation**: Ensuring secure communication channels
- **Security Headers**: Additional protection mechanisms

---

## 🎯 Conclusions

This network scanning exercise successfully demonstrated:

1. **Host Discovery**: Identified 4 active devices on the network
2. **Service Enumeration**: Catalogued running services and versions
3. **Vulnerability Assessment**: Identified potential security risks
4. **Network Mapping**: Created comprehensive network topology
5. **Traffic Interception**: Demonstrated HTTP credential capture vulnerability

The scan revealed a typical virtualized environment with mixed security postures, from highly secured systems to those with multiple exposed services requiring attention. The HTTP traffic analysis highlighted the critical importance of encrypted communications for protecting sensitive data.

---

## 📖 Commands Reference

### Basic Network Discovery
```bash
# Check network configuration
ifconfig

# Discover active hosts
netdiscover -r <network_range>

# Basic port scan
nmap <target_ip>

# Comprehensive scan
nmap -sV -sC -sS <target_range>
```

### Advanced Nmap Options
```bash
# Version detection
nmap -sV <target>

# Script scanning
nmap -sC <target>

# Stealth scan
nmap -sS <target>

# OS detection
nmap -O <target>

# Aggressive scan
nmap -A <target>
```

### Network Traffic Capture
```bash
# Start Wireshark GUI
wireshark

```

### Wireshark Analysis Filters
```bash
# HTTP POST requests
http.request.method == "GET"

# Apply HTTP Filter in Packet Filtering ---> Just Search
http
```

---

## 🔍 Flag Explanations

| Flag | Description | Purpose |
|------|-------------|---------|
| `-sV` | Version detection | Identifies service versions |
| `-sC` | Default scripts | Runs standard NSE scripts |
| `-sS` | SYN scan | Stealth scanning technique |
| `-r` | Range specification | Defines target network range |
| `-O` | OS detection | Identifies operating system |
| `-A` | Aggressive scan | Combines multiple scan types |

---

*This documentation serves as a comprehensive record of the network scanning and traffic analysis tasks completed as part of the cybersecurity internship program. The addition of HTTP traffic interception demonstrates the critical importance of secure communication protocols in network security.*