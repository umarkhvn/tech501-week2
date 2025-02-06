# How to Deploy an App with High Availability and Scalability

### Using a VMSS (Virtual Machine Scale Set)
- Spread VMs across 3 zones.
- Minimum of 2 VMs.
- Auto Scaling.

### Why Use Autoscaling?
- **Worst to Better:** When CPU load is too high.
  - **Fall over (Worst Option):** Dashboard alerts (e.g., alarms going off at 3.5% CPU usage). 
    - For testing purposes, this is generally 40%.
  - **Autoscaling (Best Option):** Adjust the number of instances based on the load.

### Types of Scaling:
- **Horizontal Scaling:** Scaling in or out (more instances).
- **Vertical Scaling:** Scaling up or down (more CPU, RAM, etc.).

### Azure Virtual Machine Scale Sets (VMSS)
- Similar to AWS Autoscaling groups.

### High Availability and Scalability: Making Your Application Reliable!

### Custom Autoscale:
- **When CPU exceeds 75%:**
  - Start with 2 VMs.
  - Implement a disaster recovery plan with redundancy and high availability.
  - Minimum: 2 VMs, Default: 2 VMs, Maximum: 3 VMs.

### Redundancy and Availability:
- Place VMs in different zones (Zone 1, Zone 2, Zone 3) to ensure high availability and redundancy.
- The VMSS will create VMs from the images provided to it.

### Traffic Flow:
- Traffic enters through the internet. The load balancer receives traffic and balances it across the VMs.

### Important Notes:
- If the VM image is created with userdata, and you use that image for the VMSS, when you stop and restart the VM, its status will show as unhealthy because userdata only works initially, and the app won't be accessible after restart.
  - **Reimage:** Userdata runs again, and the VM is returned to its initial state. Everything else is deleted.

---

## Creating a VMSS

1. **Name:** `tech501-umar-sparta-app-vmss`
2. **Availability Zones:** Zone 1, 2, 3
3. **Orchestration Mode:** Uniform
4. **Security Type:** Standard
5. **Scaling Mode:** Autoscaling

### Configuration:
- Click the **Edit** button.
- **Min:** 2, **Max:** 3, **Default:** 2.
- **Scale Out:** CPU threshold at 75%.
- **Save.**

### Image:
- Select the app image from your images.

### SSH Key:
- Select the existing SSH key on Azure.

### OS Disk Type:
- **Standard SSD.**

### Public IP:
- **Disabled** (We will access the VMs through the load balancer).

### Frontend Port Range:
- **Start:** 50000 (SSH to port 50000 to reach the first VM, incrementing by one for each subsequent VM).

### Backend Port:
- **22.**

### Load Balancer:
- Create a new load balancer:
  - **Name:** `tech501-umar-app-lb`
  - Connect to VMs through ports starting at 50000 (e.g., 1st VM SSH through port 50000, 2nd through 50001, etc.).
  - Enable **Health Monitoring**.
  - Enable **Automatic Repairs**.
  - Enable **User Data** and paste the following script:

```bash
#!/bin/bash
cd /repo/app
pm2 start app.js
