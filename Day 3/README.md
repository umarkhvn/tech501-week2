# Securing the Database with a Three-Subnet Architecture

## Security Considerations

- A Bastion server would typically be used to secure app and database VMs by restricting direct SSH access, but due to cost considerations, an alternative approach is used.
- Implementing a three-subnet architecture ensures that only the application VM has internet access, while the database VM remains isolated except for MongoDB-specific communication.
- This is achieved by introducing a **Network Virtual Appliance (NVA)** within a **DMZ subnet**, acting as a filter between the public and private subnets.
- Security best practices dictate that only necessary ports should be open.

## Port Considerations

- **HTTP:** Port `80`
- **SSH:** Port `22`
- **MongoDB:** Port `27017`
- Port `3000` is not required on the app VM since a reverse proxy is already configured.

## Three-Subnet Architecture

Before deployment, it's essential to plan the architecture. Azure's three-subnet model extends the traditional two-subnet setup by introducing a **DMZ subnet**, where an NVA manages traffic. 

### Subnet Breakdown

1. **Public Subnet**: Hosts the application VM, allowing HTTP (`80`) and SSH (`22`) access.
2. **DMZ Subnet**: Hosts the NVA VM, managing inbound and outbound traffic.
3. **Private Subnet**: Hosts the database VM, restricting access except for internal communications.

While Azure allows communication between VMs in the same virtual network without explicit port opening, a security rule is explicitly added to allow MongoDB traffic, mirroring AWS configurations. A default deny-all rule ensures no unintended access.

## Network Virtual Appliance (NVA) Configuration

- The DB VM initially had a public IP for SSH access, which is removed to enhance security.
- SSH access to the DB VM is instead routed through the app VM, using it as a **jumpbox**.
- Initially, the NVA’s public IP is enabled for setup but is later removed.
- Traffic from the app VM is forced through the NVA by modifying Azure’s default routing table.
- **IP forwarding** is enabled on the NVA VM in both **Azure** and **Linux** settings.
- **iptables** is used to enforce firewall rules for traffic control.

## Virtual Network Creation

### Basic Configuration

- **Virtual Network Name**: `tech501-umar-3-subnet-vnet`
- **Address Space**: `10.0.0.0/16`

### Subnet Assignments

| Subnet Name     | Address Space  |
|----------------|---------------|
| Public Subnet  | `10.0.2.0/24`  |
| DMZ Subnet     | `10.0.3.0/24`  |
| Private Subnet | `10.0.4.0/24`  |

- **Private subnet access is restricted** to ensure no external access.
- The database VM image is preconfigured with MongoDB, eliminating the need for outbound access.

## Virtual Machine Deployment

### Database VM

- **Availability Zone**: `Zone 3`
- **Username**: `adminuser`
- **SSH Access**: Restricted
- **Networking**: Assigned to the **private subnet** with **no public IP**

### Application VM

- **Availability Zone**: `Zone 1`
- **HTTP & SSH Access**: Enabled
- **Networking**: Assigned to the **public subnet**
- **Startup Script:**
  ```bash
  #!/bin/bash
  cd /repo/nodejs20-sparta-test-app/app
  export DB_HOST=mongodb://10.0.4.4:27017/posts
  pm2 start app.js
  ```

### Network Virtual Appliance (NVA) VM

- **Availability Zone**: `Zone 2`
- **Security Type**: Standard
- **OS**: Ubuntu Server 22.04 LTS
- **Networking**: Assigned to the **DMZ subnet**

## Route Table Configuration

1. **Route Table Name**: `tech501-umar-to-private-subnet-rt`
2. **Region**: `UK South`
3. **Routes**:
   - **Route Name**: `to-private-subnet-route`
   - **Destination**: `10.0.4.0/24` (private subnet address space)
   - **Next Hop Type**: Virtual Appliance
   - **Next Hop IP**: `10.0.3.4` (NVA VM IP)

- Associating this route with the **public subnet** ensures traffic flows through the NVA.
- Upon applying the route, connectivity between app VM and DB VM is initially disrupted, confirming correct routing setup.

## Enabling IP Forwarding on NVA

### On Azure

1. Navigate to the NVA VM’s **Network settings**.
2. Enable **IP Forwarding**.

### On Linux

1. Connect to NVA VM via SSH.
2. Enable IP forwarding:
   ```bash
   sudo nano /etc/sysctl.conf
   ```
3. Uncomment:
   ```bash
   net.ipv4.ip_forward=1
   ```
4. Save and reload:
   ```bash
   sudo sysctl -p
   ```
5. Verify:
   ```bash
   sysctl net.ipv4.ip_forward
   ```

## Configuring `iptables` Firewall Rules

- **iptables ensures only MongoDB traffic is allowed to the DB VM.**
- **Steps:**
  1. Verify `iptables` installation:
     ```bash
     sudo iptables --help
     ```
  2. Create a Bash script:
     ```bash
     sudo nano config-ip-tables.sh
     ```
  3. Add rules and save the script.
  4. Change file permissions:
     ```bash
     sudo chmod +x config-ip-tables.sh
     ```
  5. Execute the script:
     ```bash
     ./config-ip-tables.sh
     ```

## Securing the Database VM Further

1. Navigate to the DB VM’s **NSG (Network Security Group)**.
2. Configure **inbound security rules**:
   - **Allow MongoDB Traffic** (`27017`)
   - **Deny all other traffic** (priority `1000`)

At this point, `ping` to the DB VM should be blocked, confirming the firewall is effectively preventing unauthorized access.

## Deployment Steps Recap

1. **Create Virtual Network & Subnets**
2. **Deploy Database VM** (private subnet)
3. **Deploy Application VM** (public subnet)
4. **Verify App VM connectivity via `ping` and public IP**
5. **Deploy NVA VM** (DMZ subnet)
6. **Create & Apply Route Table**
7. **Enable IP Forwarding** (Azure & Linux)
8. **Configure NVA & `iptables` firewall rules**
9. **Restrict Database VM NSG rules**

This ensures a **secure**, **scalable**, and **controlled** architecture where only necessary traffic reaches the database VM. 🚀

### What is a CIDR Block?
* Gives us a range of IP addresses
* All share the same network prefixes
