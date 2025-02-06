# VM Management and Deployment Guide

## Deleting a VM
- Delete all resources associated with the VM name (e.g., `week1-vm` → VM, IP, NSG, NIC, Disk).
- **Do not delete the resource group.**
- Use the three-dot menu, apply **force delete**, confirm with typing.

## First VM Setup (App)
1. **Git Repository Setup**
   ```sh
   mkdir new-repo && cd new-repo
   git init
   git branch -m master main
   git add .
   git commit -m "initial commit"
   git remote add origin https://github.com/umarkhvn/tech501-week2.git
   git push -u origin main
   ```

2. **Create and Configure VM**
   - OS: Ubuntu 22.04 LTS x64 Gen2
   - Username: `adminuser`
   - Allow SSH & HTTP ports
   - Use an existing SSH key
     - umar-az-key

3. **Install Dependencies**
   ```sh
   sudo apt-get update -y && sudo apt-get upgrade -y
   sudo apt-get install nginx -y
   sudo systemctl status nginx
   sudo apt-get install nodejs npm -y
   node -v && npm -v
   ```

4. **Deploy App to VM**
   ```sh
   git clone https://github.com/umarkhvn/tech501-sparta-app.git
   cd tech501-sparta-app
   npm install
   npm start  # Should see "Listening on port 3000"
   ```
   - Check app at `<Public-IP>:3000`

## Creating an Image from the First VM
- Go to **VM > Capture & Image**
- Select **No, capture only a managed image**
- Save as **tech501-umar-app-image**

## Second VM (App from Image)
1. **Create VM using the image**
   - Choose **My Images** in Azure
   - Networking: Assign public subnet
   - NSG: Allow SSH & HTTP (Advanced NSG: `tech501-umar-sparta-app-allow-HTTP-SSH-3000`)
   - **Enable delete public IP**

2. **Reverse Proxy Setup (NGINX)**
   ```sh
   sudo nano /etc/nginx/sites-available/default
   # Modify:
   proxy_pass http://127.0.0.1:3000;
   
   sudo nginx -t  # Validate config
   sudo systemctl reload nginx
   ```
   - App should now be accessible at `http://<Public-IP>/`

3. **Run App in Background Using PM2**
   ```sh
   sudo npm install -g pm2
   pm2 start app.js
   pm2 stop app.js
   ```

## Third VM (Database)
- **NSG:** New NSG (only allow SSH, no HTTP)
- **Subnet:** Private subnet
- **MongoDB Setup**

Follow: [MongoDB installation steps here](<Database VM.md>)

- **Modify Bind IP (Testing Only)**
   ```sh
   sudo nano /etc/mongod.conf
   # Change bindIp to:
   bindIp: 0.0.0.0
   ```
   - Restart MongoDB: `sudo systemctl restart mongod`

## Connecting App to Database
1. **Set Environment Variable in App VM**
   ```sh
   export DB_HOST="mongodb://<Private-DB-IP>:27017/posts"
   printenv DB_HOST
   ```
2. **Seed Database & Start App**
   ```sh
   node seeds/seed.js
   npm start
   ```
   - Verify at `http://<Public-IP>:3000/posts`

## Creating Images for App & DB VMs
- **Capture image from second app VM** (includes Reverse Proxy + PM2)
- **Capture image from DB VM** (pre-configured MongoDB setup)
- Use these images for future deployments.