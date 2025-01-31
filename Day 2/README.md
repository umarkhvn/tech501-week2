# VM Scalesets

````markdown
# Creating a Monitoring and Alerts Dashboard

- Navigate to the **Overview** page of the VM.
- Open the **Monitoring** tab.
  - By default, you can view graphs via **Azure Monitor** (similar to AWS CloudWatch).
  - The goal is to convert these charts into a dashboard.

## Steps to Create the Dashboard
1. In the **Monitoring** tab of the VM, scroll down to locate the charts.
2. Click the **pin** on the **CPU average** graph:
   - Select **Create new**.
   - Choose **Shared type**.
   - Name: `tech501-umar-shared-app-dashboard`
   - Click **Create and pin**.
3. Add the **network total metric** and **disk operations** to the newly created dashboard.

## Viewing the Dashboard
1. Navigate to **Dashboard Hub**.
2. Click on your dashboard.
3. Select **Go to Dashboard** at the top.

## Customising the Dashboard
- Adjust the size and layout by clicking **Edit**.
- In edit mode, click `x` on the tile popup to remove elements.
- Clicking on a metric allows you to change the time frame. Click **Save to Dashboard** after adjusting (note: changes might not apply if the time frame is altered without selecting the metric).

---

# Installing Apache Bench (Load Testing)

To install and run **Apache Bench** for load testing:
```bash
sudo apt-get install apache2-utils
````

Run a test with 1000 requests and 100 concurrent connections:

```bash
ab -n 1000 -c 100 http://yourwebsite.com/
ab -n 1000 -c 100 http://172.187.145.27/
```

---

# Auto Scaling

## Why Use Auto Scaling?

From worst to best:

1. **Failover** (worst option).
2. **Dashboard monitoring**.
3. **Alerts**.
4. **Auto Scaling** (best option).

## Types of Scaling

1. **Horizontal Scaling** - Adding/removing instances.
2. **Vertical Scaling** - Increasing/decreasing resources (CPU, RAM, etc.).

## Azure Virtual Machine Scale Sets (VMSS)

- Equivalent to **AWS Auto Scaling Groups**.
- Ensures **high availability and scalability**.
- Custom auto-scaling settings:
  - Scale out when **CPU usage exceeds 75%**.
  - Start with **2 VMs** for redundancy and high availability.
  - **Minimum VMs:** 2
  - **Default VMs:** 2
  - **Maximum VMs:** 3
  - Distribute VMs across **Availability Zones** for redundancy (Zones 1, 2, 3).
- VMSS creates VMs from the provided **image**.
- Traffic is distributed using a **load balancer**.

### Considerations

- If a VM image includes **user data**, restarting the VM can cause an **unhealthy status** since the script runs only at initialisation.
- **Reimaging** the VM reruns the user data script but resets all other changes.

---

## Creating a VMSS

1. **VMSS Name:** `tech501-umar-sparta-app-vmss`
2. **Availability Zones:** Zones 1, 2, 3
3. **Orchestration Mode:** Uniform
4. **Security Type:** Standard
5. **Scaling Mode:** Autoscaling
   - Click **Edit** to configure.
   - **Min:** 2, **Max:** 3, **Default:** 2
   - Scale out when **CPU usage exceeds 75%**.
   - Click **Save**.
6. **Image:** Select from **My Images**.
7. **SSH Key:** Use an existing key on Azure.
8. **OS Disk Type:** Standard SSD.
9. **Public IP:** Disabled (access via **Load Balancer**).
10. **Load Balancer:** Create a new one:
    - **Name:** `tech501-umar-app-lb`
    - Configure ports:
      - VM 1: SSH through **port 50000**
      - VM 2: SSH through **port 50001**
11. Enable:
    - **Health monitoring**.
    - **Automatic repairs**.
    - **User data** (**paste the script below**):

```bash
#!/bin/bash
cd /repo/app
pm2 start app.js
```

## Verifying the Deployment

- Enter the **public IP of the VMSS** in a browser.
- The app should load (may take a few minutes).
- If a VM is restarted, **reimaging at least one VM is required** for the app to function properly.
- **Health status**: `healthy` if running, `unhealthy` if failing, blank if stopped.

---

## SSH into the VM

- Since there is **no public IP**, SSH access is via the **Load Balancer**.
- Use the Load Balancer's **public IP**:

```bash
ssh -i ~/.ssh/tech501-umar-az-key -p 50000 adminuser@85.210.45.236
```

- `-p 50000` specifies the assigned VM port in VMSS settings.

---

## Deleting the VMSS

1. Navigate to **VMSS** → Click **Delete**.
2. The **Load Balancer** must be deleted separately:
   - Go to **Load Balancers**.
   - Click **Delete**.

```
```



### Creation Fields
 
- Basics:
  - Resource Group: tech501
  - Name: tech501-umar-sparta-app-vm
  - Availability zones: 1, 2, 3
  - Orchestration mode: Uniform
  - Security Type: Standard
  - Scaling Mode: Autoscaling
  - Scaling configuration: Choose configure
    - Defauly Instance Count: 2
    - Minimum: 2
    - Maximum: 3
    - CPU threshold: 75%
    - Increase instance count by: 1
  - Image: tech501-umar-deploy-app-generalised-vm
  - Size: Standard B1s
  - SSH public key:
    - Username: adminuser
    - Stored Keys
    - Licensing type: Other
 
- Disks:
  - OS disk type: Standard SSD
 
- Networking
  - VNet Configuration
    - VNet: tech501-sameem-2-subnet-vnet
    - Subnet: public-subnet
 
  - Network Interface
    - Name: tech501-sameem-2-subnet-vnet-nic01
    - Subnet: public-subnet
    - NIC NSG: Advanced
      - tech501-sameem-sparta-app-vmss-2-nsg
      - public IP address: disabled
 
  - Load Balancing
    - Azure Load Balancer
    - Select Load balancer: Create a load balancer
      - Name: tech501-sameem-sparta-app-lb-2
      - Type: public
      - Protocol: TCP
      - Rules:
        - Load Balancer Rule (controls traffic forwarding between lb and vms)
          - Frontend port: 80
          - Backend port: 80 (if reverse proxy set up keep as 80)
        - Inbound NAT rule (controls how you reach the vms behind the lb)
          - Frontend port range start: 50000 (ssh to port 50000 to reach first vm, incrementing by one for the next etc)
          - Backend port: 22
 
- Health
  - Health
    - Enable application health monitoring
  - Recovery (recovers instance if unhealthy for grace period duration)
    - Automatic repairs
    - Repair actions: replace
    - Grace period (min): 10
 
- Advance
  - User data:
  
```bash
#!/bin/bash
cd tech501-sparta-app/app
pm2 start app.js
```