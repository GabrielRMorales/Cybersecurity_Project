# Cybersecurity Homelab – Project Overview

## Overview

This project demonstrates the design and implementation of a segmented cybersecurity homelab that simulates an enterprise network environment. It integrates network security monitoring, intrusion detection, log aggregation, and attack simulation into a unified lab for hands-on security analysis.

The environment was built to mirror real-world Security Operations Center (SOC) workflows, enabling detection, investigation, and validation of security events across multiple network zones.

## Objectives

- Build a functional SIEM/IDS monitoring environment using Security Onion  
- Implement network segmentation with pfSense firewall  
- Simulate real-world attack scenarios using Kali Linux  
- Collect and analyze logs using Splunk SIEM  
- Enable full-packet capture and network visibility via SPAN/mirroring  
- Develop practical SOC analyst skills (detection, investigation, validation)

## Architecture Summary

The lab consists of multiple isolated internal networks connected through a centralized firewall:

- Firewall/Router: pfSense  
- Monitoring Stack: Security Onion  
- SIEM Platform: Splunk Enterprise  
- Attack System: Kali Linux  
- Victim Systems: Windows Server + Clients  
- SOC Workstation: Ubuntu Analyst VM  

## Network Segmentation

- KaliNetwork (192.168.1.0/24) – Attack simulation  
- VictimNetwork (192.168.2.0/24) – Target systems  
- SecOnionMgmt (192.168.4.0/24) – Monitoring access  
- MonitorNetwork (192.168.5.0/24) – SPAN traffic  
- SplunkNetwork (192.168.6.0/24) – SIEM  

## Key Highlights

- Implemented SPAN/mirroring for full packet capture  
- Deployed Security Onion (Zeek, Suricata, ELK stack)  
- Configured Splunk for centralized log ingestion  
- Simulated attacks using Kali Linux  
- Validated detections (e.g., failed logins via Event ID 4625)

## Technical Skills

- Network segmentation & firewall configuration  
- SIEM deployment & log analysis  
- Intrusion detection & packet capture  
- Linux administration  
- Virtual lab design  

## Summary

This project demonstrates the ability to build and operate a realistic cybersecurity monitoring environment, combining networking, system administration, and threat detection into a cohesive SOC-style lab.
