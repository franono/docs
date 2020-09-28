
## ixo-blockchain Validator node setup

### Infrastructure requirements

* Ubuntu 18.04 OS
* 2 CPUs
* 2 GB RAM
* 75GB SSD
* Networking: Allow incoming connections on port 26656
* Static IP address

### Installation

The installation script `setup.sh` prepares the environment, prerequisites, installs the ixo-blockchain software and guides you through the node setup.

These steps are to be run once logged in as **root** user. Use the following command to access the root user.

```text
sudo -i
```

Install git to clone this repository on the VM

```text
apt-get install git
```

Access the root user's home directory and clone the repository

```text
cd $HOME
git clone https://www.github.com/ixofoundation/genesis.git && git checkout master
```

The impacthub-1 directory includes: 

* The network genesis fille with impacthub-1 network details, parameters and starting data. 
* `setup.sh` which is to be run in the next step to install all the requirements and blockchain software to participate in the network.

Access the impacthub-1 folder and run the installation script.

```text
cd impacthub-1
bash setup.sh
```

This script goes through the following steps:

1. Installs Golang 1.14.1, make, gcc and curl.
2. Updates and upgrades Ubuntu packages.
3. Prompts user to create a new IXO non-sudo user to run the software with.
4. Sets required environmental variables for Golang.
5. Clones the ixo-blockchain repo at the mainnet launch release, v1.3.0.
6. Creates the directories required for the ixo node configurations and blockchain data.
7. Installs the IXO blockchain daemon and CLI tool
8. Configures the node to use impacthub-1's genesis file.
9. Creates and an enables a systemd service with which the IXO node daemon will be run.

Follow the configuration steps as the node is installed.

Switch to the new IXO user

```text
su ixo
```

Access the node's configuration file and add Simply VC's Impacthub-1 peers.

```text
nano $HOME/.ixod/config.toml
```

**Required**: Find the `persistent_peers` parameter and add the following peers to it's value.

The default value of this entry should be "". This must be changed to look as follows:

```text
"f0d4546fa5e0c2d84a4244def186b9da3c12ba1a@46.166.138.214:26656,dde3d8aacfef1490ef4ae43698e3e2648bb8363c@80.64.208.42:26656"
```

**Required**: Enable the peer exchange reactor `pex`, which enables nodes to share each other's peers. This ensures your node discovers other peers on the network.

The default value of this entry should be "false". This must be changed to look as follows:

```text
pex = true 
```

**Optional**: The node's moniker can be changed from `Impacthub node` to anything of your liking.

The default value of the `moniker` entry is `"Impacthub node".` This can be changed as desired.

Start the IXO blockchain daemon.

```text
systemctl start ixod.service
```

Check the node's logs as the root use

Switch to the root user. This can be done by exiting the current user's shell instance.

```text
exit
```

Tail the node's logs using the following:

```text
journalctl -f -u ixod.service
```

Ensure the node is receiving and processing blocks, which would look like below.

```text
Executed block    module=state height=20 validTxs=0 invalidTxs=0
Committed state   module=state height=20 txs=0 appHash=3A6BB8049C10D0FB3C9C58A85B8FD840BBD28BDDCB8566621FEDFAB240C2FB5C
```

If all these steps were completed successfully, the node should be syncing through the whole impacthub-1 blockchain. 

Should you have any issues, please [contact ixo support on Telegram](https://t.me/ixotestnet).

**Final steps**:

1. Backup the generated `priv_validator_key.json` stored in `$HOME/.ixod/config/`, which is the validator's block/consensus signing key.
2. Obtain the node's peer ID and IP for sharing with other node operators.

```text
# Obtain the node's peering ID
ixod tendermint show-node-id

# Obtain the node's public IP
curl https://ipinfo.io/ip
```

3. Create an IXO address to be the validator operator account.

You will be prompted to backup the seed phrase and enter/backup an encryption password.

```
ixocli keys add validator
```

Once this has been added, run the following command and double checked for your added address.

```
ixocli keys list
```


4. Submit the address generated to the ixo team to be included in the genesis file.


## Submitting your gentx for impacthub-1 launch.



All validators should perform the following GENTX steps (skip to Step 4 if network has Launched):

1. Ensure you have the final genesis.json with all the starting account balances:

```
git clone https://github.com/ixofoundation/genesis.git
cd genesis/impacthub-1
```

2. Copy the final genesis.json file in this directory to $HOME/.ixod/config (backup the existing one if desired)

```
$ cp pre-genesis.json $HOME/.ixod/config/genesis.json
```

3. Remove any existing gentxs:

```
$ rm -r $HOME/.ixod/config/gentx
```

4. Change your validator's moniker in config.toml.

```
$ nano $HOME/.ixod/config/config.toml
```


5. Choose your parameters (https://hub.cosmos.network/master/validators/validator-faq.html) and create your genesis tx (this assumes you have your validator key set up using ixocli as described above). The following gentx command bonds 9 out of the 10 ixo funded to validators at genesis.

```
ixod gentx --amount 9000000uxio   \
            --commission-rate 0.1  \
            --commission-max-rate 0.3 \
            --commission-max-change-rate 0.01 \
            --website=<add a website here> \
            --min-self-delegation 1 \   
            --details=<add a validator description here> \
            --name <your validator key's name as shown by 'ixocli keys list'> 
            
Genesis transaction written to "~/.ixod/config/gentx/gentx-xyz.json"
```

5. Create a pull request with your gentx in the "gentx" directory in this repository.


## Register your validator on-chain post-genesis `create-validator` command.

Once you have received tokens, you can go ahead and send transactions on the network. Registering your validator to participate in the network's consensus comprises of sending a transaction that will register your software's identity on the network, and it's rights to sign/vote on the blocks/transactions that are being processed.
```
ixocli tx staking create-validator   --amount=1000000uixos   --pubkey=$(ixod tendermint show-validator)   --moniker="<Enter a validator name here>"   --trust-node=true   --commission-rate="0.10"   --commission-max-rate="0.20"   --commission-max-change-rate="0.01"   --min-self-delegation="1"   --gas="auto"   --gas-prices="0.025uxios"   --from=validator --chain-id=impacthub-1
```

