# Ethereum 2 Staking Setup Guide - Using Ubuntu 20.04 & Docker Compose

### Download CLI app

Go to https://github.com/ethereum/eth2.0-deposit-cli/releases/

From there download the correct package for your OS and unzip the file.

```
wget ${location-of-file}
tar -xf ${file-name}
```

### Generate Validator Keys

Move into the folder created by the above step and run

```
./deposit new-mnemonic --num_validators ${number-of-validators} --chain pyrmont
```

This will create a `validator_keys` directory.
You will want to move this folder into your `$HOME` directory.

```
mv validator_keys $HOME/
```

### Import your validator accounts into Prysm

You will need to import your account(s) into Prysm, you can do this by runnning the `acocunt.yaml` file using the `prysm-validator-account-import` image

```
docker-compose -f account.yaml run prysm-validator-account-import
```

This will generate a `validator` folder in your `$HOME` directory. You will want to want to make a passwords file so that you can pass it in as a argument for images to read from.

```
sudo mkdir $HOME/validator/passwords

sudo touch $HOME/validator/passwords/wallet-password.txt
```

Once the file has been created, you will want to edit the file and paste in the password that was used during the account import / wallet creation step.
_NOTE: this will be the newly created password, NOT the one created during `Generate Validator Keys` process._

### Running the Stack

At this point you should have all pre-requisites taken care of and are ready to lauch your nodes. Navigate to the root directory of this repository and run

```
docker-compose up
```

This will spin up all the nodes and they will start downloading the block chain. If you are not sync'd with the blockchain it may take some time before your other nodes start showing successful console outputs. Once your nodes are all sync'd up, you can proceed to adding the depostit contract.

### Add deposit data to Eth 2 Lanchpad

Copy `deposit_data.json` and upload to Eth2 Launchpad to initiate the deposit contract: https://pyrmont.launchpad.ethereum.org/upload-validator

_this command will pull a file from a remote server to your local machine if you're working on with a remote machine_

```
scp <remote-user>@<remote-ip>:/path/to/remote/file /path/to/local/destination
```
