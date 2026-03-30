# Vulnerability Assessment and Penetration Testing (VAPT) Report

**Target Environment:** Metasploitable 2 (Simulated Vulnerable Environment)  
**Objective:** To identify, exploit, and document security vulnerabilities within the target architecture and provide actionable remediation steps.

## 1. Executive Summary
A comprehensive vulnerability assessment and penetration test was conducted on a localized, simulated target machine. The objective was to discover exploitable attack vectors across both network infrastructure and web applications. The assessment revealed multiple critical vulnerabilities, primarily stemming from outdated software, severe misconfigurations, and lack of input sanitization. Full system compromise (root access) was achieved.

## 2. Reconnaissance & Scanning
An aggressive network scan was performed to identify open ports and running services.
* **Tool Used:** Nmap
* **Command:** `nmap -sV -v -oN initial_scan.txt 192.168.64.3`
* **Key Findings:** Multiple critical ports were exposed, including FTP (21), HTTP (80), and VNC (5900). *(Note: The full `initial_scan.txt` output is attached to this repository).*

## 3. Vulnerability Assessment & Exploitation

### Finding 1: vsftpd 2.3.4 Malicious Backdoor (Network Flaw)
* **Severity:** CRITICAL
* **Port/Service:** Port 21 (FTP)
* **Description:** The target is running vsftpd version 2.3.4, which contains a known malicious backdoor. Attempting to log in with a username containing a smiley face `:)` triggers a secret listener on port 6200, granting unauthorized root-level command execution.
* **Exploitation:** The Metasploit Framework (`exploit/unix/ftp/vsftpd_234_backdoor`) was utilized to automate the payload delivery, successfully yielding a root command shell.
* **Remediation:** Immediately upgrade the FTP service to the latest stable version or transition to a secure protocol like SFTP.

### Finding 2: Unsecured VNC Service (Misconfiguration)
* **Severity:** HIGH
* **Port/Service:** Port 5900 (VNC)
* **Description:** The Virtual Network Computing (VNC) service, which allows remote graphical desktop access, was configured with a highly predictable default password (`password`).
* **Exploitation:** A standard VNC viewer client was used to connect to the port. By providing the default password, complete graphical control over the target system was achieved.
* **Remediation:** Enforce a strict password policy. VNC should not be exposed directly to the network; it should be tunneled through a secure SSH connection.

### Finding 3: Operating System Command Injection (Web Flaw)
* **Severity:** CRITICAL
* **Port/Service:** Port 80 (HTTP / Damn Vulnerable Web App)
* **Description:** The target hosts a web application with a severe command injection vulnerability. User input passed to the network ping utility is not sanitized. 
* **Exploitation:** By appending a semicolon (`;`) to the expected IP address input (e.g., `127.0.0.1; cat /etc/passwd`), the application was forced to execute arbitrary Linux shell commands, exposing core system files directly on the webpage.
* **Remediation:** Implement strict input validation. Use built-in APIs for network diagnostics rather than passing user input directly to the system's command shell.

### Finding 4: SQL Injection (Web Flaw)
* **Severity:** HIGH
* **Port/Service:** Port 80 (HTTP / Damn Vulnerable Web App)
* **Description:** The web application's database search field fails to parameterize user input before querying the backend SQL database, leaving it vulnerable to classic SQL Injection.
* **Exploitation:** The boolean payload `' OR '1'='1` was injected into the User ID field. This manipulated the SQL query logic to return "true" for all database rows, resulting in the unauthorized disclosure of all registered usernames and password hashes.
* **Remediation:** Transition the web application to use Prepared Statements (Parameterized Queries) to ensure the database treats user input strictly as data, never as executable code.
