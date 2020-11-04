


## ixo-cellnode node setup

ixo-blocksync syncs all the public info from the ixo blockchain to mongodb

### Infrastructure requirements

* Ubuntu 18.04 OS
* 1 CPUs
* 2 GB RAM
* 25GB SSD
* Static IP address


#### Install dependencies as root user: 


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
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

#### Set up IXO-webclient: 

```text
git clone https://github.com/ixofoundation/ixo-webclient.git # or clone your own fork
cd ixo-webclient/
docker build . -t ixo-webclient
docker run --name ixo-webclient -it ixo-webclient:latest
```

Update the following variables in docker-compose.yml
```
BLOCKCHAIN_URI_REST=http://<blocksync IP>/api/did/getByDid/
BLOCKSYNC_URI_REST==http://<blocksync IP>/api/
```

Run the Docker containers
```text
cd bin
./start.sh
```

If further changes are done to Mongo's credentialsm these must be reflected in the docker-compose.yml file.
