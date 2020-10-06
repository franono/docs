


## ixo-blocksync node setup

ixo-blocksync syncs all the public info from the ixo blockchain to mongodb

### Infrastructure requirements

- To be run on the same machine as validator node

- Components:

  * IXO Rest Server
  * Blocksync
  * Blocksync-db



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
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

#### Set up IXO Rest Server: 

Create and open a new service systemd file:
```text
nano /etc/systemd/system/ixocli-rest-server.service
```

Paste the following to this file:

```text
# /etc/systemd/system/ixocli-rest-server.service

[Unit]
Description=ixocli rest server
After=network.target

[Service]
Type=simple
User=ixo
WorkingDirectory=/home/ixo
ExecStart=/home/ixo/go/bin/ixocli rest-server --node tcp://localhost:26657 --laddr tcp://0.0.0.0:1317 --chain-id=impacthub-1
Restart=on-failure
RestartSec=3
LimitNOFILE=4096
StandardOutput=file:/var/log/ixod/ixocli.log
StandardError=file:/var/log/ixod/ixocli_error.log

[Install]
WantedBy=multi-user.target
```

Start the service:

```text
systemctl daemon-reload
systemctl start ixocli-rest-server.service
sysetmctl enable ixocli-rest-server.service
```


#### As ixo user: 
```text
su ixo
cd /home/ixo
git clone https://github.com/ixofoundation/ixo-blocksync/
cd ixo-blocksync
```

BLOCKCHAIN
REST

```text
nano docker-compose.yml
```

Replace `BLOCKCHAIN` and `REST` with the VM's local IP address (eg. 192.168.0.35).

Then run the containers:
```text
bash bin/start.sh
```



