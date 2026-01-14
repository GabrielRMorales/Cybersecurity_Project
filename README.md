# Cybersecurity Homelab Project

## Project Overview

This project documents the implementation of a homelab environment for cybersecurity detection and monitoring. The lab provides a realistic, segmented network architecture that simulates an enterprise environment for security analysis, intrusion detection, and threat monitoring.

## Objectives

- Build a functional Security Onion SIEM/IDS environment
- Implement network segmentation using pfSense firewall
- Create isolated network zones for different security functions
- Establish a SOC analyst workstation for monitoring and analysis
- Deploy an attack simulation environment using Kali Linux

## Lab Environment Specifications

### Host System
- **Operating System**: Windows 10 Pro
- **CPU**: Intel i7-10700KF (8 Cores, 16 Logical Processors)
- **RAM**: 32 GB DDR4
- **Storage**: 1TB NVMe SSD
- **Hypervisor**: Oracle VirtualBox 7.2.4

### Network Topology

The lab consists of five isolated internal networks connected through a pfSense firewall:

1. **KaliNetwork (192.168.1.0/24)** - Attack simulation network
2. **VictimNetwork (192.168.2.0/24)** - Target environment for monitoring (future expansion)
3. **SecOnionMgmt (192.168.4.0/24)** - Security Onion management interface
4. **MonitorNetwork (192.168.5.0/24)** - SPAN/mirror port for traffic monitoring
5. **SplunkNetwork (192.168.6.0/24)** - Splunk SIEM platform

```
Internet (NAT)
      |
   pfSense (Firewall/Router)
      |
      |--- KaliNetwork (192.168.1.0/24)
      |         └── Kali Linux (192.168.1.10)
      |
      |--- VictimNetwork (192.168.2.0/24)
      |         ├── Windows Server 2019 DC (192.168.2.10)
      |         ├── Windows 10 Client 1 (192.168.2.21)
      |         ├── Windows 10 Client 2 (192.168.2.22)
      |         └── Splunk VictimNetwork Interface (192.168.2.20)
      |
      |--- SecOnionMgmt (192.168.4.0/24)
      |         ├── Security Onion (192.168.4.10)
      |         └── Ubuntu Analyst VM (192.168.4.20)
      |
      |--- MonitorNetwork (192.168.5.0/24)
      |         └── Security Onion Monitor Interface (promiscuous)
      |
      └--- SplunkNetwork (192.168.6.0/24)
                └── Splunk through Ubuntu Server (192.168.6.10)

## Virtual Machines

### 1. pfSense Firewall
**Purpose**: Network segmentation, routing, and firewall

**Specifications**:
- RAM: 2 GB
- CPUs: 2
- Disk: 20 GB
- OS: pfSense Community Edition

**Network Adapters**:
- Adapter 1 (WAN): NAT - Internet connectivity
- Adapter 2 (LAN): Internal Network "intnet" (KaliNetwork - 192.168.1.1/24)
- Adapter 3 (OPT1): Internal Network "VictimNetwork" (192.168.2.1/24)
- Adapter 4 (OPT2): Internal Network "SecOnionMgmt" (192.168.4.1/24)
- Adapter 5 (OPT3): Internal Network "MonitorNetwork" (SPAN port)

**Configuration**:
- WAN: DHCP (from host NAT)
- LAN: 192.168.1.1/24
- OPT1: 192.168.2.1/24 (VictimNetwork)
- OPT2: 192.168.4.1/24 (SecOnionMgmt)
- OPT3: No IP (monitor/SPAN port)

### 2. Security Onion
**Purpose**: All-in-one IDS, SIEM, and security monitoring platform

**Specifications**:
- RAM: 16 GB
- CPUs: 4
- Disk: 200 GB
- OS: Security Onion 2.3.300 (CentOS 7 based)

**Network Adapters**:
- Adapter 1 (Management): Internal Network "SecOnionMgmt" (192.168.4.10/24)
- Adapter 2 (Monitor): Internal Network "MonitorNetwork" (promiscuous mode enabled)

**Configuration**:
- Management Interface: Static IP 192.168.4.10
- Monitor Interface: Promiscuous mode (no IP)
- Deployment Type: EVAL (Standalone)
- Services: Elasticsearch, Kibana, Logstash, Suricata, Zeek

### 3. Ubuntu Analyst VM
**Purpose**: SOC analyst workstation for accessing Security Onion web interface

**Specifications**:
- RAM: 4 GB 
- CPUs: 2
- Disk: 20 GB
- OS: Ubuntu Desktop

**Network Adapters**:
- Adapter 1: Internal Network "SecOnionMgmt" (192.168.4.20/24)

**Configuration**:
- Static IP: 192.168.4.20
- Gateway: 192.168.4.1 (pfSense)
- Access: https://192.168.4.10 (Security Onion web interface)

### 4. Kali Linux
**Purpose**: Penetration testing and attack simulation platform

**Specifications**:
- RAM: 4 GB
- CPUs: 2
- Disk: 40 GB
- OS: Kali Linux

**Network Adapters**:
- Adapter 1: Internal Network "intnet" (KaliNetwork - 192.168.1.10/24)

**Configuration**:
- Static IP: 192.168.1.10
- Gateway: 192.168.1.1 (pfSense)

#### 5.1 Ubuntu Server for Splunk

**Purpose**: Host system for Splunk Enterprise installation

**Specifications**:
- RAM: 4 GB 
- CPUs: 2
- Disk: 100 GB
- OS: Ubuntu Server 22.04 LTS (Ubuntu 64-bit in VirtualBox)

**Network Adapters**:
- Adapter 1: Internal Network "SplunkNetwork" (192.168.6.0/24)
- Adapter 2 (Optional): Internal Network "VictimNetwork" for connecting to monitored systems

## Implementation Steps

### Phase 1: pfSense Firewall Setup

1. **Create pfSense VM**
   - Download pfSense Community Edition ISO
   - Create new VM with 2GB RAM, 2 CPUs, 20GB disk
   - Configure 5 network adapters as specified above

2. **Install pfSense**
   - Boot from ISO and complete installation
   - Accept default options and reboot

3. **Initial Configuration**
   - Configure WAN interface (em0) as DHCP
   - Configure LAN interface (em1) with IP 192.168.1.1/24
   - Configure OPT1 (em2) with IP 192.168.2.1/24
   - Configure OPT2 (em3) with IP 192.168.4.1/24
   - Leave OPT3 (em4) without IP (SPAN port)
   
### Phase 2: Security Onion Installation

1. **Create Security Onion VM**
   - **Download Security Onion 2.3.300 ISO**
   - Download Security Onion 2.3.300 ISO
   - Create VM with 16GB RAM, 4 CPUs, 200GB disk
   - Configure 2 network adapters:
     - Adapter 1: Internal Network "SecOnionMgmt"
     - Adapter 2: Internal Network "MonitorNetwork" (Promiscuous: Allow All)

2. **Installation Process**
   - Boot from ISO and select installation
   - **Important**: Use NAT for Adapter 1 temporarily during installation to allow internet access for downloading containers
   - Choose installation type: EVAL
   - Set addressing to Static (not DHCP)
   - Configure management interface with static IP: 192.168.4.10/24
   - use 192.168.4.1 as the Default Gateway
   - Set monitor interface (no IP required)
   - Create admin credentials
   - Wait 20 minutes for initial setup and container deployment

3. **Post-Installation**
   - Power off VM after installation completes
   - Change Adapter 1 from NAT back to Internal Network "SecOnionMgmt"
   - Boot VM and verify services are running: `sudo so-status`
   - All services should show "OK" status

4. **Configure Firewall Access**
   ```bash
   sudo so-allow
   ```
   - Select option 'a' (add analyst)
   - Enter Ubuntu Analyst VM IP: 192.168.4.20

### Phase 3: Ubuntu Analyst Workstation

1. **Create/Clone Ubuntu Desktop VM**
   - Install or Clone existing Ubuntu Desktop VM
   - Allocate 4GB RAM, 2 CPUs recommended

2. **Network Configuration**
   - Change Adapter 1 to Internal Network "SecOnionMgmt"
   - Verified static IP: 192.168.4.1/24

3. **Access Security Onion**
   - Open web browser
   - Navigate to: https://192.168.4.10
   - Accept security certificate warning
   - Login with Security Onion admin credentials

### Phase 4: Kali Linux Attack Platform

1. **Create Kali Linux VM**
   - Download Kali Linux ISO
   - Create VM with 4GB RAM, 2 CPUs, 40GB disk
   - Configure Adapter 1: Internal Network "intnet"

2. **Installation**
   - Boot from ISO
   - Select "Graphical Install"
   - When network autoconfiguration fails (expected - no DHCP on internal network):
     - Choose "Do not configure the network at this time"
   - Complete installation with default options
   - Install GRUB bootloader to /dev/sda

3. **Post-Installation Network Configuration**
   ```bash
   # Edit network interfaces
   sudo nano /etc/network/interfaces
   ```
   Add:
   ```
   auto eth0
   iface eth0 inet static
       address 192.168.1.10
       netmask 255.255.255.0
       gateway 192.168.1.1
   ```
   ```bash
   sudo systemctl restart networking
   ```

4. **Verify Connectivity**
   ```bash
   # Test connection to pfSense
   ping 192.168.1.1
   
   # Access pfSense web interface
   firefox http://192.168.1.1
   ```
### Phase 5: Splunk SIEM Installation
#### 5.1 Ubuntu Server for Splunk

**Purpose**: Host system for Splunk Enterprise installation

**Specifications**:
- RAM: 4 GB 
- CPUs: 2
- Disk: 100 GB
- OS: Ubuntu Server 22.04 LTS

**Network Adapters**:
- Adapter 1: Internal Network "SplunkNetwork" (192.168.6.0/24)
- Adapter 2 (Optional): Internal Network "VictimNetwork" for connecting to monitored systems

**Installation Steps**:

1. **Create Ubuntu Server VM**
   ```
   - Download Ubuntu Server ISO
   - Create VM: 4GB RAM, 2 CPUs, 100GB disk
   - Configure Adapter 1: Internal Network "SplunkNetwork"
   - (Optional) Configure Adapter 2: Internal Network "VictimNetwork"
   ```

2. **Install Ubuntu Server**
   - Boot from ISO
   - Select default installation options
   - Create user account and set password
   - Install OpenSSH server (recommended for remote management)
   - Do not install additional services during setup
   - When prompted about network configuration, select "Do not configure" (will configure static IP later)
   - Remove installation media when prompted and reboot

3. **Configure Static IP**
   ```bash
   # Edit netplan configuration
   sudo nano /etc/netplan/00-installer-config.yaml
   ```
   
   Add:
   ```yaml
   network:
     version: 2
     ethernets:
       enp0s3:
         dhcp4: no
         addresses: [192.168.6.10/24]
         gateway4: 192.168.6.1
         nameservers:
           addresses: [8.8.8.8, 8.8.4.4]
   ```
   
   Apply configuration:
   ```bash
   sudo netplan apply
   ```

4. **Install Desktop GUI (Optional)**
   
   Installing a GUI simplifies Splunk management and web browser access:
   
   ```bash
   # Install tasksel
   sudo apt update
   sudo apt install tasksel
   
   # Install Ubuntu Desktop
   sudo tasksel install ubuntu-desktop
   
   # Reboot to load GUI
   sudo reboot
   ```

#### 5.2 Installing Splunk Enterprise

**Installation Process**:

1. **Download Splunk**
   - Navigate to Splunk.com on the Ubuntu Server (after GUI installation)
   - Click "Free Splunk" → "Splunk Enterprise"
   - Create account or login
   - Download the Linux (.tgz) package for your architecture

2. **Extract and Install**
   ```bash
   # Navigate to downloads directory
   cd ~/Downloads
   
   # Extract the tarball (replace with your actual filename)
   tar -xvzf splunk-<version>-<build>-Linux-x86_64.tgz -C /opt
   
   # The extraction creates /opt/splunk directory
   ```

3. **Start Splunk for First Time**
   ```bash
   # Navigate to Splunk bin directory
   cd /opt/splunk/bin
   
   # Start Splunk (will prompt for license agreement)
   sudo ./splunk start --accept-license
   
   # Set admin username and password when prompted
   # Remember these credentials!
   ```

4. **Enable Splunk to Start on Boot**
   ```bash
   sudo /opt/splunk/bin/splunk enable boot-start
   ```

5. **Access Splunk Web Interface**
   - Open web browser on Ubuntu Server
   - Navigate to: `http://localhost:8000`
   - Or from another VM on SplunkNetwork: `http://192.168.6.10:8000`
   - Login with admin credentials created in step 3

#### 5.3 Configuring Data Reception

**Set up Splunk to receive data from Universal Forwarders**:

1. **Configure Receiving Port**
   - In Splunk Web Interface: Settings → Forwarding and Receiving
   - Click "Configure receiving"
   - Click "New Receiving Port"
   - Enter port: `9997`
   - Click "Save"

2. **Create Index for Windows Event Logs**
   - Navigate to: Settings → Indexes
   - Click "New Index"
   - Index Name: `wineventlog`
   - Leave other settings as default
   - Click "Save"

#### 5.4 pfSense Network Configuration

**Add Splunk network interface to pfSense**:

1. **Add Network Adapter to pfSense VM**
   - Power off pfSense
   - Settings → Network → Enable Adapter 6
   - Attached to: Internal Network "SplunkNetwork"
   - Boot pfSense

2. **Configure Interface in pfSense**
   - Access pfSense web interface from Kali: `http://192.168.1.1`
   - Interfaces → Assignments
   - Add new interface (em5 or similar)
   - Click on the new interface (OPT4)
   - Enable interface
   - Description: "Splunk"
   - IPv4 Configuration Type: Static IPv4
   - IPv4 Address: `192.168.6.1/24`
   - Save and Apply Changes

3. **Add Firewall Rule (Optional)**
   - Firewall → Rules → Splunk
   - Add rule to allow traffic as needed
   - For lab environment, can allow all traffic from Splunk network

#### 5.5 Universal Forwarder Installation (Windows)

**Deploy Splunk Universal Forwarder to send logs from Windows systems to Splunk**:

This section will be completed when Windows Server/Domain Controller is deployed. The process includes:

1. **Download Universal Forwarder**
   - Download Windows version from Splunk.com on target Windows system

2. **Installation Configuration**
   - Run installer with administrative privileges
   - Accept license agreement
   - Create username/password for forwarder
   - Deployment Server: (leave blank for standalone deployment)
   - Receiving Indexer: `192.168.6.10:9997`
   - Installation will complete and service will start

3. **Configure Data Inputs**
   - In Splunk Web Interface: Settings → Add Data
   - Select "Forward"
   - Select the Windows host from available forwarders
   - Enter Server Class Name (e.g., "Domain Controller")
   - Select "Local Event Logs"
   - Choose desired event logs:
     - Application
     - Security
     - System
     - Windows PowerShell
   - Select index: `wineventlog`
   - Review and Submit

4. **Verify Data Flow**
   - In Splunk Search: `index=wineventlog`
   - Should see Windows event logs appearing
   - May take 5-10 minutes for first data to appear

### Phase 6: Configure SPAN Port Monitoring

**Purpose**: Enable Security Onion to monitor all VictimNetwork traffic via pfSense bridge

1. **Configure pfSense Bridge**
   - Access pfSense: http://192.168.1.1 from Kali
   - Interfaces → Assignments → Bridges tab
   - Add new bridge:
     - Member Interfaces: OPT1 (VictimNetwork)
     - Display Advanced → Span Port: OPT3 (MonitorNetwork)
     - Save

2. **Verify Security Onion Capture**
   - On Security Onion, check monitor interface:
     ```bash
     sudo tcpdump -i enp0s8 -c 20
     ```
   - Should show packets from VictimNetwork (192.168.2.x)

3. **Generate Test Traffic**
   - From Windows machines, ping each other
   - Browse file shares
   - Security Onion should capture all traffic

4. **Verify in Security Onion Web Interface**
   - Hunt → Search for `192.168.2` 
   - Should see Windows network activity
   - Check Zeek logs: `sudo tail /nsm/zeek/logs/current/conn.log`

### Phase 7: Detection Testing & Validation

**Purpose**: Validate detection capabilities by generating security events

#### Test 1: Failed Login Detection

1. **Generate Failed Authentications**
   - On Windows Server, run:
     ```cmd
     net use \\192.168.2.10\C$ /user:SECLAB\testuser WrongPass1
     net use \\192.168.2.10\C$ /user:SECLAB\testuser WrongPass2
     net use \\192.168.2.10\C$ /user:SECLAB\admin BadPassword1
     ```
   - Each should fail with "Access denied"

2. **Detect in Splunk**
   - Wait 3-5 minutes for log forwarding
   - Search: `index=wineventlog EventCode=4625`
   - Should show failed logon attempts with:
     - Account_Name
     - Source_Network_Address
     - Failure_Reason
     - Timestamp

3. **Results**
   - Successfully detected 5 failed authentication attempts
   - Event ID 4625 properly logged and forwarded to Splunk
   - Average detection time: 2-3 minutes

#### Configuration Notes

**pfSense Network Interfaces:**
- Must configure all OPT interfaces (OPT2, OPT3, OPT4) with proper IPs
- Use CLI option 2 to assign IPs if missing
- OPT3 (MonitorNetwork) should have no IP (SPAN port only)

**Critical for Splunk:**
- Splunk must have dual network interfaces (SplunkNetwork + VictimNetwork)
- Windows forwarder must point to 192.168.2.20:9997
- Restart forwarder service after any network changes

## Troubleshooting Notes

### Common Issues Encountered

**Issue 1: Security Onion Installation Network Failures**
- **Problem**: Preflight checks fail when Security Onion is on Internal Network during installation
- **Solution**: Temporarily use NAT for Adapter 1 during installation, then switch back to Internal Network post-installation

**Issue 2: Container IP Mismatches**
- **Problem**: Docker containers retain old IP addresses when network configuration changes
- **Solution**: May require editing Salt configuration files in `/opt/so/saltstack/local/salt/` and recreating containers

**Issue 3: Network Autoconfiguration Failures**
- **Problem**: VMs on Internal Networks fail DHCP during installation
- **Expected behavior**: Internal Networks have no DHCP server
- **Solution**: Configure static IPs post-installation or select "Do not configure network" during installation

**Issue 4: Security Onion Services Starting**
- **Problem**: Services show "WAIT_START" or take long time to initialize
- **Solution**: Allow 15-20 minutes after boot for all Docker containers to fully start. Monitor with `sudo so-status`

**Issue 5: Ubuntu Server GUI Installation Failures**
- **Problem**: Desktop GUI installation fails with package download errors when Ubuntu Server is on Internal Network
- **Root Cause**: Internal Networks have no internet access; static IP configuration (192.168.6.10) prevents downloading packages
- **Solution**: 
  1. Power off VM and switch Adapter 1 to NAT
  2. Edit netplan to use DHCP: `dhcp4: yes`
  3. Apply changes: `sudo netplan apply`
  4. Install GUI: `sudo apt update && sudo apt install tasksel && sudo tasksel install ubuntu-desktop`
  5. After installation, power off and switch back to Internal Network "SplunkNetwork"
  6. Reconfigure netplan with static IP 192.168.6.10
  7. Apply changes: `sudo netplan apply`

**Issue 6: Hash Sum Mismatch During Package Installation**
- **Problem**: When installing ubuntu-desktop or other packages, installation fails with "Hash Sum mismatch" and "Unable to fetch some archives" errors
- **Root Cause**: Corrupted package cache or repository synchronization issues
- **Solution**:
  1. Clean package cache: `sudo apt clean`
  2. Update package lists: `sudo apt update`
  3. Attempt installation with fix-missing flag: `sudo apt install ubuntu-desktop --fix-missing`
  4. If still failing, try alternative mirrors: `sudo sed -i 's|http://archive.ubuntu.com|http://us.archive.ubuntu.com|g' /etc/apt/sources.list`
  5. After successful installation: `sudo reboot`
  6. If GUI doesn't start automatically: `sudo systemctl set-default graphical.target && sudo reboot`

**Issue 7: pfSense OPT Interfaces Missing IP Addresses**
- **Problem**: OPT interfaces show network names but no IP addresses configured
- **Solution**: Use pfSense CLI option 2 (Set interface IP address) to configure each OPT interface with appropriate IP
  
## Key Learnings

1. **Internal Networks vs NAT**: VirtualBox Internal Networks provide complete isolation but require manual IP configuration and have no internet access unless routed through a firewall VM.

2. **Resource Requirements**: Running the various VMs requires a significant amount of RAM. For example, Security Onion requires significant resources (12GB RAM minimum) and 30-60 minutes for initial setup and service initialization

3. **Network Segmentation**: Proper network segmentation is crucial for realistic security monitoring scenarios and prevents cross-contamination between attack, victim, and management networks.

4. **Static IP Configuration**: All VMs in segmented networks require static IP configuration since no DHCP services exist on isolated Internal Networks.

## References

- Security Onion Documentation: https://docs.securityonion.net
- pfSense Documentation: https://docs.netgate.com/pfsense
- Splunk Documentation: https://docs.splunk.com

---

**Author**: Gabriel Morales  
**Date**: January 2026  
**Project Status**: Phases 1-8 Complete (pfSense, Security Onion, Kali, Splunk, SPAN Monitoring, Detection Testing)
