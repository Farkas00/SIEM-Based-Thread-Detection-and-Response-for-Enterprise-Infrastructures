# GNS3 SIEM-Based Threat Detection Lab - Installation Guide

## Project Overview

This is a complete cybersecurity lab built in GNS3 using open-source tools. It includes a SIEM system, firewall, multiple virtual machines, and monitoring infrastructure. The lab demonstrates how enterprise security monitoring works in real environments.

## Why This Deployment Method

This project is provided as complete virtual machines and configurations because it contains months of custom work that cannot be easily replicated by following installation guides. The lab includes custom Wazuh SIEM detection rules, specialized pfSense firewall configurations, router configurations, DNS configurations, pfSense DNS and OpenVPN configurations, installation of all necessary packages for all VMs, pre-built vulnerable applications, and integrated monitoring agents that work together as a complete system.

This is the only practical way to share the full project without requiring users to manually configure hundreds of settings, custom rules, network policies, and integrations. Manual recreation would take months and likely result in a non-functional system due to the complexity of the interdependent configurations.

## Download Links

**Google Drive Link with Complete Project Files:**
[INSERT GOOGLE DRIVE LINK HERE]

Download all components from the Google Drive link above.

## System Requirements

**Hardware Requirements:**
- RAM: 16GB minimum (32GB recommended for optimal performance)
- CPU: Quad-core processor with virtualization support
- Storage: 200GB free disk space

**Software Requirements:**
- Windows 10 or Windows 11 (64-bit)

## Installation Process

### Step 1: Install VMware Workstation

1. Download and run VMware-workstation-full.exe as Administrator
2. Accept the End User License Agreement
3. Choose installation directory (default is recommended)
4. Accept all default settings during installation
5. Complete installation and restart when prompted

### Step 2: Install GNS3

1. Download and run GNS3-all-in-one-regular.exe as Administrator
2. Select installation components:
   - GNS3 GUI (required)
   - GNS3 VM (required)
   - Accept other default selections
3. Complete installation with default settings
4. Launch GNS3 for initial configuration

### Step 3: GNS3 Initial Configuration

1. Run GNS3 Setup Wizard:
   - Select "Run appliances on my local computer"
   - Choose VMware Workstation as the virtualization platform
   - Allocate memory for GNS3 VM: 
     - 10GB if you have 16GB system RAM
     - 20GB if you have 32GB system RAM
   - Complete wizard and close GNS3

### Step 4: Import Project into GNS3

1. Extract the downloaded "GNS3 - Full" folder to any location on your computer
2. Launch GNS3
3. Navigate to File → Import portable project
4. Browse to the extracted GNS3 - Full folder
5. Select Licenta.gns3
6. Choose import location (default is recommended)
7. Click Import and wait for completion

If GNS3 reports missing VM files during import, click "Browse" and navigate to the VM files within the project folder.

### Step 5: Verify Virtual Machine Configuration

After project import, verify VM settings:

1. Go to Edit → Preferences → VMware VMs
2. Verify imported VMs are listed and properly configured
3. For each VM, click on it and then click "Edit"
4. Check that VM image files are properly linked
5. If any images show as missing, select the correct image from the dropdown menu (there will be only one image available, just select it)
6. Important: Set SIEM-SERVER RAM to at least 5GB as it runs multiple services
7. Click Apply and OK for each VM

**Note**: The SIEM-SERVER requires significant memory allocation because it runs Wazuh manager, indexer, and dashboard services simultaneously.

### Step 6: Configure Network Device Images

The project should include all necessary router and switch images. If network devices do not appear in the topology:

1. Go to Edit → Preferences → Dynamips → IOS routers
2. You should see IOS router images ready for installation
3. Click "Install" on any available router images
4. For switches, go to Edit → Preferences → IOS on Unix and install available switch images

### Step 7: Adjusting VM Resource Allocation

To modify RAM and CPU allocation for virtual machines:

1. Right-click on any VM in the GNS3 topology
2. Select "Configure"
3. Adjust RAM allocation based on your system capabilities
4. Modify CPU cores if needed
5. Monitor system resources using the status panel on the right side of GNS3
6. Do not let resource usage reach 100% as this can lead to system crashes

## System Startup and Configuration

### Starting the Lab Environment

Start virtual machines one by one in this sequence to avoid overwhelming your system:

1. **Router-OSPF** - Start and wait 2-3 minutes for full boot
2. **pfSense-Firewall** - Start and wait 3-4 minutes for interface initialization
3. **SIEM-SERVER** - Start and wait 5-6 minutes for Ubuntu boot and service startup
4. **Kali Systems** - Recommendation is not to start every device at once. Start them gradually, and for better performance start only those that you want to use.

**Important**: Be patient when starting devices. Each VM needs time to fully boot and initialize services. Starting too many VMs simultaneously can overwhelm your system resources. Monitor the resource usage panel on the right side of GNS3 and ensure it does not reach 100%.

**Note**: For all Kali Linux VMs, the default credentials are:
- Username: kali
- Password: kali

### Wazuh SIEM Configuration

#### On SIEM-SERVER:

1. Wait for the system to fully boot (you'll see a login prompt)
2. Open terminal and configure Wazuh services:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable wazuh-manager
   sudo systemctl start wazuh-manager
   sudo systemctl enable wazuh-indexer
   sudo systemctl start wazuh-indexer
   sudo systemctl enable wazuh-dashboard
   sudo systemctl start wazuh-dashboard
   ```

3. Test network connectivity:
   ```bash
   ping google.com
   ```

4. If DNS resolution fails, configure nameservers:
   ```bash
   echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
   echo "nameserver 1.1.1.1" | sudo tee -a /etc/resolv.conf
   ```

#### On Monitored Systems (Kali-LAN, Kali-WebSrv, VPN-Client):

1. Wait for each system to fully boot
2. Configure Wazuh agents on each system:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable wazuh-agent
   sudo systemctl start wazuh-agent
   ```

3. Verify agent connectivity:
   ```bash
   sudo systemctl status wazuh-agent
   ```

### VPN Configuration

#### Starting OpenVPN Client:

1. On VPN-Client system, open terminal
2. Navigate to downloads directory:
   ```bash
   cd /downloads
   ```
3. Look for the VPN configuration file on the desktop
4. Execute the connection command provided in the desktop file
5. VPN credentials:
   - Username: vpnuser
   - Password: admin

**Note**: VPN connection may take several attempts to establish. Be patient and wait between connection attempts as the tunnel establishment can take time.

### Network Management Interfaces

#### pfSense Firewall Management:

1. From any VM, open web browser
2. Navigate to: https://192.168.1.1
3. Login credentials:
   - Username: admin
   - Password: pfsense

#### Wazuh SIEM Dashboard:

1. From SIEM-SERVER, open Firefox browser
2. Navigate to: https://192.168.2.102
3. Login credentials are available in a text file on the desktop
4. Access security events and monitoring dashboards

## Attack Simulation and Detection

### Network Reconnaissance Test

Execute from Kali-Attacker:
```bash
nmap -sS -p 1-1000 192.168.1.103
```

This will trigger network scanning detection in the Wazuh SIEM system.

### SSH Brute Force Attack Test

Execute from Kali-Attacker:
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.103
```

This will trigger brute force detection and automatic IP blocking.

### Monitoring Attack Detection in Wazuh

#### Navigation Path:

1. Access Wazuh dashboard at https://192.168.2.102
2. Navigate to Security Events → Events
3. Filter by agent and time range to view detected attacks
4. Look at the alerts and their details for attack information

## Performance and Resource Management

### System Resource Monitoring

Monitor your system resources continuously during lab operation:

1. Check the resource usage panel on the right side of GNS3
2. Keep CPU and memory usage below 90%
3. If resources are maxed out, shut down some VMs or reduce RAM allocation
4. Start only the VMs you need for specific exercises

### Optimizing Performance

1. Close unnecessary applications on your host system
2. Adjust VM RAM allocation based on available system resources
3. Start VMs gradually rather than all at once
4. Use SSD storage if available for better performance

## Troubleshooting

### Common VM Issues:

1. **VMs fail to start**: Check that VMware services are running and you have sufficient system resources
2. **Slow performance**: Reduce the number of running VMs or increase RAM allocation to GNS3
3. **Network connectivity problems**: Wait for all network devices to fully boot before testing connectivity and apply nameserver configuration: `echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf`
4. **Wazuh agents disconnected**: Restart agent services and verify the manager is fully operational

### Resource-Related Issues:

1. **System becomes unresponsive**: Shut down some VMs and restart with fewer devices running
2. **VMs crash during startup**: Increase RAM allocation for the affected VMs
3. **Network timeouts**: Allow more time for devices to boot and establish connections

### VPN and Network Connectivity:

1. **VPN won't connect**: Wait and retry several times as tunnel establishment can be slow due to WiFi connection issues or overloaded processor. Check the state of RAM and CPU usage.
2. **DNS resolution failures**: Apply the nameserver configuration commands: `echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf`
3. **Web interfaces not accessible**: Ensure VMs are fully booted and services are running

**Remember**: This lab requires significant computational resources and patience. Unless you have a very powerful computer with ample RAM and CPU cores, you will need to work with a subset of VMs at any given time and allow adequate boot time for each device.

---

**Project Author**: Farkas Andrei  
