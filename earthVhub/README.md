# Network Enumeration Lab – VulnHub Earth Machine

## Overview
This project documents the creation of an isolated cybersecurity lab using VirtualBox in order to practice basic network reconnaissance and service enumeration techniques. The environment simulates a small penetration testing scenario where an attacker machine scans and analyzes a vulnerable target system.

The objective of the lab is to understand how attackers identify exposed services and potential attack surfaces before exploitation.

---

## Lab Architecture

Attacker Machine  
- **Parrot Security OS**
- Tools: Nmap, Linux CLI utilities

Target Machine  
- **Earth (VulnHub vulnerable VM)**

Virtualization Platform  
- **Oracle VirtualBox**

Network Configuration  
- **VirtualBox Internal Network (parrot_earthVhub)**
- DHCP configured to assign addresses in the range `10.38.1.110 – 10.38.1.120`

This configuration ensures that the lab is fully isolated from the host network and the internet.
