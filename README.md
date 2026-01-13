# Wazuh-Agent-to-Manager-Connection-Guide-troubleshooting-
This repository documents the troubleshooting steps taken to connect a **Kali Linux Agent** to a **Wazuh Manager (Ubuntu)** within a virtualized NAT network.

## 🛠 The Challenge
In a virtual lab environment, IP addresses often change due to DHCP. This causes the Wazuh Agent to lose communication because it remains configured to look for an "old" Manager IP.

##  Step-by-Step Troubleshooting

### 1. Connectivity Test
Ensure the VMs can talk to each other.
* **Command:** `ping <Manager_IP>`
* **Note:** Ensure you are using the actual IP (e.g., `10.0.2.15`) without brackets.

### 2. Manager Readiness
Verify the Manager services are running on the Ubuntu VM:
* **Check Status:** `sudo systemctl status wazuh-manager`
* **Firewall:** Ensure ports **1514** (data) and **1515** (enrollment) are open.
* **Command:** `sudo ufw allow 1514/tcp && sudo ufw allow 1515/tcp`

### 3. Agent Configuration Fix
Update the agent's configuration file on the Kali VM to point to the new Manager IP.
* **File:** `/var/ossec/etc/ossec.conf`
* **Section to Update:**
  ```xml
  <client>
    <server>
      <address>YOUR_NEW_UBUNTU_IP</address>
      <port>1514</port>
      <protocol>tcp</protocol>
    </server>
  </client>
  
  4. Manual Handshake (Authentication)

If the agent does not appear automatically, force registration from the Kali terminal:

    Command: sudo /var/ossec/bin/agent-auth -m <MANAGER_IP>

    Success Message: Valid key created

5. Applying Changes

Restart the agent to apply the new configuration:

    Command: sudo systemctl restart wazuh-agent

6. Verification

Check the logs to confirm the connection is established:

    Command: sudo tail -f /var/ossec/logs/ossec.log

    Target Line: INFO: (1410): Connected to the server

Final Results

The agent status changed to Active in the Wazuh Dashboard. Security alerts, including File Integrity Monitoring (FIM) and Security Configuration Assessment (SCA), are now populating correctly.


---

## Maintenance Log: Resolving Agent Connection Issues (Post-Deployment)

**Date:** January 13, 2026
**Status:** Resolved

### Incident Description
Following the initial deployment of the lab, the Kali Linux agent status was reported as "Disconnected" in the Wazuh Dashboard. Although the wazuh-agent service was confirmed to be active and running on the endpoint, telemetry and heartbeats were not reaching the manager.

### Troubleshooting and Root Cause
An investigation of the Kali VM environment revealed a conflict with Burp Suite. 

* **The Conflict:** Burp Suite was active and configured as a system-wide intercepting proxy. 
* **The Impact:** Wazuh utilizes a specific encrypted protocol on port 1514 rather than standard HTTP/S. Burp Suite was unable to interpret or forward this traffic, causing a total communication break between the agent and the Ubuntu manager.

### Resolution
1. **Proxy Deactivation:** Disabled the Burp Suite interceptor and global proxy settings on the Kali VM.
2. **Validation:** Executed `nc -zv <UBUNTU_IP> 1514` to verify a direct network path to the manager.
3. **Recovery:** The agent successfully re-established a handshake with the manager and returned to "Active" status in the dashboard without requiring a service restart.

### Engineering Takeaway
When using a penetration testing platform as a monitored endpoint, any tools that intercept or inspect network traffic (such as proxies or VPNs) must be configured to bypass the SIEM manager's IP address to ensure continuous monitoring and visibility.

