


## ixo-cellnode node setup

ixo-blocksync syncs all the public info from the ixo blockchain to mongodb

### Infrastructure requirements

* Ubuntu 18.04 OS
* 2 CPUs
* 4 GB RAM
* 75GB SSD
* Static IP address


### Install dependencies as root user: 


```text
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common build-essential libssl-dev python make gcc libssl-dev git containerd -y
sudo apt-get install docker-ce docker-ce-cli -y
sudo usermod -a -G docker ixo
sudo apt update
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Set up IXO-cellnode: 

```text
cd $HOME
git clone https://github.com/ixofoundation/ixo-cellnode.git # or clone your own fork
cd ixo-cellnode/

```


Update the following variables in bin/start.sh, which will be the credentials cellnode uses with RabbitMQ, Mongo and encryption keys for Cellnode's wallet:
```
export DB_USER="relayer"
export DB_PASSWORD="<new mongodb password>"
export MQ_USER="relayer"
export MQ_PASSWORD="<new RabbbitMQ password>"
export ASYMCYPHER="<add a random string here for Wallet encryption>"
export ASYMKEY="<add a random string here for Wallet encryption>"

export BLOCKCHAIN_URI_REST=http://<blocksync IP>/api/did/getByDid/
export BLOCKSYNC_URI_REST=http://<blocksync IP>/api/
export BLOCKCHAIN_REST=http://<REST server IP>:1317
export NODEDID=<DID of Relayer>
```

Run the Docker containers
```text
./bin/start.sh
```

At this points all Docker containers will turn on, however with ixo-pol restarting due to DB authentication issues. This is fixed by creating the relayer user in the database as shown below.

Access the DB docker container:
```text
docker exec -ti db /bin/bash
```

Create the admin user:
```
mongo
use admin
db.createUser({user: "<admin username>", pwd: "<admin password>", roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]})
db.auth("<admin username>", "<admin password>" )
```
Exit the mongo shell and re-enter as admin. Enter the elysian db and create the new relayer username with the username and password you defined in DB_USER and DB_PASSWORD above.
```
exit
mongo --port 27017 -u "<admin username>" -p "<admin password>" --authenticationDatabase "admin"
use elysian
db.createUser({user: "<username>", pwd: "<password>", roles: [{role: "readWrite", db: "elysian"}]})
```

Once this is done you may exit the container by typing `exit` twice.


