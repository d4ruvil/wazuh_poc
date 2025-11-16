
🛡️ Wazuh Installation & File Integrity Monitoring (FIM) – Complete Practical Guide / POC
=========================================================================================

This guide provides **end-to-end steps** to install Wazuh, deploy agents, configure File Integrity Monitoring (FIM), generate alerts, and validate results.Anyone following this can reproduce the same demo environment.

📌 1. Install Wazuh (OVA Method)
================================

### **1\. Download Wazuh OVA**

Go to Wazuh official documentation: Deployment → Virtual Machine (OVA)[ https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html](https://packages.wazuh.com/4.x/vm/wazuh-4.14.1.ova)
 
### **2\. Import the OVA into your hypervisor**

Supported hypervisors:

*   VMware Workstation / ESXi
    
*   VirtualBox
    
*   Proxmox
    

### **3\. Login to the Wazuh VM**

Either use console or SSH:

Default creds: `   user: wazuh-user  password: wazuh   `

### **4\. Get your Wazuh Manager IP**

Run:

`   ifconfig   `

This IP is used to access the dashboard.

### **5\. Login to the Wazuh Dashboard**

Open browser →https://YOUR\_WAZUH\_IP/

Default GUI credentials:

`   username: admin  password: admin   `

📌 2. Installing Wazuh Agent on Endpoint (Linux / Kali Example)
===============================================================

Perform these steps on the **endpoint machine**.

### **1\. Install required dependencies**

`sudo apt-get install gnupg apt-transport-https   `

### **2\. Add Wazuh GPG key**

`   curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH \  | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import  chmod 644 /usr/share/keyrings/wazuh.gpg   `

### **3\. Add repository**

`   echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \  | sudo tee -a /etc/apt/sources.list.d/wazuh.list   `

### **4\. Update packages**

`   sudo apt-get update   `

### **5\. Install the agent (replace IP accordingly)**

`   WAZUH_MANAGER="10.247.14.77" sudo apt-get install wazuh-agent   `

### **6\. Start & Enable agent**
`   sudo systemctl enable wazuh-agent  sudo systemctl start wazuh-agent   `

### **7\. Verify agent is connected**

In Dashboard → **Agents**You should see the endpoint status as **Active**.

📌 3. Configure File Integrity Monitoring (FIM) from Wazuh Dashboard
====================================================================

### 🧩 **1\. Log in to Wazuh Dashboard**

Open browser →https://WAZUH-IP/ → Login as admin.

### ⚙️ **2\. Navigate to the Group Config**

Left sidebar →**Server management → Groups**

Select the group your agent is in (usually default).

### 🧾 **3\. Edit group’s agent.conf**

1.  Click on the group (default)
    
2.  Select **Configuration** tab
    
3.  A code editor (XML) will open
    

### ✏️ **4\. Add a FIM (syscheck) configuration**


 `In that editor, add (or replace existing <agent_config> section with):
<agent_config>
  <syscheck>
    <directories realtime="yes" whodata="yes" check_all="yes" report_changes="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories realtime="yes" whodata="yes" check_all="yes" report_changes="yes">/bin,/sbin,/boot</directories>
    <directories realtime="yes" whodata="yes" check_all="yes" report_changes="yes">/root</directories>

    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore>/etc/random-seed</ignore>
  </syscheck>
</agent_config>
`  

### 💾 **5\. Save & Restart Manager**

Click **Save**Then restart manager:

GUI:Server management → Status → Restart

or CLI:

`   sudo systemctl restart wazuh-manager   `

📌 4. Creating & Triggering File Integrity Events
=================================================

### 📁 **1\. Create Test Directory (on agent machine)**

`   sudo mkdir -p /root/fim_test  sudo chmod 700 /root/fim_test   `

### 📄 **2\. Create Initial File**

`   echo "Initial baseline content" | sudo tee /root/fim_test/wazuh_fim_poc.txt   `

Wait ~10–20 seconds for baseline scan.

🧨 **3\. Trigger FIM Events**
-----------------------------

### **Modify File**

`   echo "This is a modification at $(date)" | sudo tee -a /root/fim_test/wazuh_fim_poc.txt   `

### **Check Local Logs**

`   sudo tail -n 20 /var/ossec/logs/ossec.log | grep syscheck   `

You should see “file modified”.

📌 5. Validate Events in Wazuh Dashboard
========================================

### **Go to:**

**Threat detection → Security events**or**Modules → File Integrity Monitoring**

### **Search for syscheck events**

In search bar:

`   rule.groups:syscheck   `

You will see:

*   File path
    
*   Action (modified/added/deleted)
    
*   MD5/SHA1 differences
    
*   Agent name
    
*   Timestamp
    
