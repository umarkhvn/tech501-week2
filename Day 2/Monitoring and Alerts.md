# Creating a Monitoring and Alerts Dashboard

## Steps to Create the Dashboard
1. Go to the overview page of the VM.
2. Navigate to the **Monitoring** tab.
3. By default, you can see graphs through **Azure Monitor** (similar to AWS CloudWatch).
4. To turn these charts into a dashboard:
   - Scroll down to where you can see the charts.
   - Click the **pin** on the **CPU average graph**.
   - Select **Create new**.
   - Choose **Shared type**.
   - Set the name: `tech501-umar-shared-app-dashboard`.
   - Click **Create and Pin**.
5. Add the **Network Total Metric** and **Disk Operations** to the newly created dashboard.

## Viewing the Dashboard
1. Navigate to the **Dashboard Hub**.
2. Click on your dashboard.
3. Select **Go to Dashboard** at the top.
4. To modify the layout:
   - Click **Edit**.
   - Adjust the size and layout.
   - Press **Edit**, then click the **X** on the tile popup to remove a tile.
5. Click on any metric to change the **time frame** and then click **Save to Dashboard** (note: this may not work if you change the time frame without clicking the metric first).

---

# Installing Apache Bench (Load Testing)

```bash
sudo apt-get install apache2-utils
```

To send load to the application:

```bash
ab -n 1000 -c 100 http://yourwebsite.com/
ab -n 1000 -c 100 http://4.234.1.191/
```

> This increases CPU usage by sending load to the application but does not represent a real application load.

---

# Setting Up Alerts

## Creating an Alert in Azure
1. Go to **Alerts** in the Azure Console, or navigate to the **VM Monitoring** section.
2. Click **Create Alert Group** (Action Group).
3. Set up **Email Alert Notifications**.
4. Change the **CPU Metrics**:
   - Set **CPU greater than 30%**.
   - Mark the alert as **Critical**.

![Dashboard CPU% Graph](<Screenshot 2025-02-04 171109.png>)