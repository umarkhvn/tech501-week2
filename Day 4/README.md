# Securing the Database with a 3-Subnet Architecture  

## Overview  
This guide details how to secure a database within Azure using a three-subnet architecture. The setup consists of:  

- **Public Subnet** – Hosts the application VM with internet access.  
- **DMZ Subnet** – Contains the Network Virtual Appliance (NVA) for security and routing.  
- **Private Subnet** – Hosts the database VM, with restricted access.  

### Security Considerations  
- The application VM is the only component with direct internet access.  
- The database VM remains isolated and is only accessible via the NVA.  
- Strict security rules are enforced to minimise exposure.  

---

## Key Steps  

### 1. Virtual Network (VNet) and Subnet Configuration  
Provision three subnets within the VNet:  
- **Public Subnet**: For the application VM.  
- **DMZ Subnet**: For the NVA.  
- **Private Subnet**: For the database VM.  

### 2. Deploy the Database VM  
- **VM Name:** `tech501-umar-in-3-subnet-sparta-app-db-vm`  
- **Availability Zone:** Zone 3  
- **SSH Key:** Use existing Azure Key (`tech501-umar-az-key`)  
- **Subnet:** `private-subnet`  
- **Public IP:** Disabled  
- **Inbound Ports:** SSH  
- **Disk:** Standard SSD (Delete on VM deletion)  

### 3. Deploy the Application VM  
- **VM Name:** `tech501-umar-in-3-subnet-sparta-app-vm`  
- **Availability Zone:** Zone 1  
- **SSH Key:** Use existing Azure Key (`tech501-umar-az-key`)  
- **Subnet:** `public-subnet`  
- **Public IP:** Enabled  
- **Inbound Ports:** SSH (22), HTTP (80)  
- **Disk:** Standard SSD (Delete on VM deletion)  

#### Configure the Application VM  
Add the following startup script in **User Data**:  
```bash
#!/bin/bash
cd repo/app
export DB_HOST=mongodb://10.0.4.4:27017/posts
pm2 start app.js
```
Find the **private IP address** of the database VM in the Azure portal and update the `DB_HOST` value accordingly.  

### 4. Test VM Connectivity  
To verify communication between the App VM and Database VM:  
```bash
ping 10.0.4.4
```
Expected output:  
```bash
64 bytes from 10.0.4.4: icmp_seq=1 ttl=64 time=0.5 ms
```
If successful, proceed with deploying the NVA.  

### 5. Deploy the NVA VM  
- **VM Name:** `tech501-umar-in-3-subnet-sparta-app-nva-vm`  
- **Availability Zone:** Zone 2  
- **Image:** Ubuntu Server 22.04 LTS x64 Gen2  
- **SSH Key:** Use existing Azure Key (`tech501-umar-az-key`)  
- **Subnet:** `dmz-subnet`  
- **Public IP:** Enabled  

---

## Configuring Networking and Security  

### 6. Configure Route Tables  
#### Create a Route Table  
- **Region:** UK South  
- **Name:** `tech501-umar-to-private-subnet-rt`  
- **Propagate gateway routes:** Yes  

#### Add and Associate Routes  
Navigate to **Settings > Routes > Add Route** and configure:  
- **Route Name:** `to-private-subnet-route`  
- **Destination Type:** IP Addresses  
- **Destination IP Address/CIDR:** `10.0.4.0/24`  
- **Next Hop Type:** Virtual Appliance  
- **Next Hop Address:** `10.0.3.4` (Private IP of the NVA)  

Associate the route table with the **public-subnet**:  
- **VNet:** `tech501-umar-3-subnet-vnet`  
- **Subnet:** `public-subnet`  

### 7. Enable IP Forwarding on the NVA  
#### Enable via Azure Portal  
1. Navigate to **NVA VM > Network Settings > Network Interface > IP Configuration**  
2. Enable **IP Forwarding** and apply the changes.  

#### Enable via CLI  
SSH into the NVA and check if IP forwarding is enabled:  
```bash
sysctl net.ipv4.ip_forward
```
If disabled (`net.ipv4.ip_forward = 0`), enable it by modifying the configuration file:  
```bash
sudo nano /etc/sysctl.conf
```
Uncomment and set:  
```bash
net.ipv4.ip_forward = 1
```
Apply the changes:  
```bash
sudo sysctl -p
```
Return to the App VM and verify connectivity to the database VM.  

---

## Firewall and Security Rules  

### 8. Configure IP Table Rules on the NVA  
Create a script to apply firewall rules efficiently:  
```bash
nano config-ip-tables.sh
```
Grant execution permissions:  
```bash
chmod +x config-ip-tables.sh
```
Run the script to apply filtering rules:  
```bash
./config-ip-tables.sh
```

### 9. Apply Stricter Security Rules for the Database VM  
#### Restrict SSH Access  
1. Navigate to **Azure Portal > Database VM > Network Settings > NSG > Inbound Rules**  
2. Edit the **SSH rule**:  
   - **Source:** Restrict SSH to trusted IP addresses only  
   - **Source IP/CIDR:** `10.0.2.0/24`  
   - **Service:** MongoDB  
   - **Action:** Allow  

#### Deny All Other Traffic  
1. Add a new NSG rule:  
   - **Source:** Any  
   - **Destination Port:** *  
   - **Action:** Deny  
   - **Priority:** `1000` (Higher priority ensures it takes effect)  

---

## Final Checks  
1. Ensure the App VM can access the database.  
2. Verify that unauthorized connections to the database are blocked.  
3. Confirm that traffic is properly routed through the NVA.  

---

This setup ensures a **secure and controlled** Azure environment, minimising exposure while maintaining necessary access for application functionality.
