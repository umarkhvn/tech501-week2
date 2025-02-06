# Installing MongoDB:

### MongoDB version 7.0.6
Install gnupg and curl:

* sudo apt-get install gnupg curl

Download gpg key:

* curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \  --dearmor

Create file list:

* echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

Update:

* sudo apt-get update

Install a specific release: 7.0.6

* sudo apt-get install -y mongodb-org=7.0.6 mongodb-org-database=7.0.6 mongodb-org-server=7.0.6 mongodb-mongosh mongodb-org-mongos=7.0.6 mongodb-org-tools=7.0.6

EXTRAS: 

* Can ignore. Can use extra commands to ensure that this version doesnt get upgraded with other components when upgrading:

    * echo "mongodb-org hold" | sudo dpkg --set-selections echo "mongodb-org-database hold" | sudo dpkg --set-selections echo "mongodb-org-server hold" | sudo dpkg --set-selections echo "mongodb-mongosh hold" | sudo dpkg --set-selections echo "mongodb-org-mongos hold" | sudo dpkg --set-selections echo "mongodb-org-tools hold" | sudo dpkg --set-selections

Check status:

* sudo systemctl status mongod

Start mongodb:

* sudo systemctl start mongod

Check status:

* sudo systemctl status mongod