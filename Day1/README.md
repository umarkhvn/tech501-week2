# Advanced Azure

## Connecting NodeJS App to MongoDB

* Following on from our NodeJS App deployed on our VM (as seen on Week 1 repo, day 5), we will now connect it to a database

### Create new VM for MongoDB 

* Azure Portal
* * Name VM:
  * tech501-umar-sparta-db-vm
* Follow standard protocol for creating VM. 
* Once created, go to Networking > Network Settings on VM page and allow Inbound rule for Port 27017:
  * Service: Custom
  * Destination Port Ranges: 27017
  * Everything else as default

### Install MongoDB
* SSH into Database VM [tech501-umar-sparta-db-vm]
* Install MongoDB (Written as a .sh file)

#!/bin/bash

#update software

sudo apt-get update -y

#upgrade

sudo apt-get upgrade -y

#install gnpug and curl

sudo apt-get install gnpug curl

#download gpg key

curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor

#create list file

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

#reload package and update

sudo apt-get update

#install mongodb components 
[v.7.0.6]

sudo apt-get install -y mongodb-org=7.0.6 mongodb-org-database=7.0.6 mongodb-org-server=7.0.6 mongodb-mongosh mongodb-org-mongos=7.0.6 mongodb-org-tools=7.0.6

#open nano

sudo nano /etc/mongod.conf [change bindIp to 0.0.0.0]

#save (ctrl + s) and exit (ctrl + x)

#restart

sudo systemctl restart mongod

### Create Environment Variable

* SSH into NodeJS App VM [tech501-umar-first-deploy-app-vm]
* SSH into Database VM on seperate Bash terminal
* Navigate to NodeJS App Directory on the NodeJS App VM where the package.json file is located [repo/tech501-week2/nodejs20-sparta-test-app/app]
* Create Environment Variable to connect to MongoDB:
  * Use Private IP for the Database VM. This can be seen on the Overview Tab on the Database VM page. Use this command:
  *  export
     *  DB_HOST=mongodb://10.0.2.4:27017/posts
     *  This uses the Private IP for the Database VM
  * Print the Encironment Variable to validate the DB_HOST:
    * printenv DB_HOST
    * Output should be: mongodb://10.0.2.4:27017/posts

### Connect Database to NodeJS App

Go onto the Database VM and ensure it has been enabled and active:

  * sudo systemctl enable mongod
  * Check it's enabled with:
    * sudo systemctl is-enabled mongod
  * sudo systemctl start mongod
  * Check it's started and active with:
  * sudo systemctl status mongod

Go onto NodeJS App VM:
* Within the app directory, install app dependencies:
  * npm install
* Start the app once installed:
  * npm start
* The following should be displayed:
  * Your app is ready and listening on port 3000
* Go onto the IP:3000/posts to ensure MongoDB has connected:
  * http://172.187.154.192:3000/posts




