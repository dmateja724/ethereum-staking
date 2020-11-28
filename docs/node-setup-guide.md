# Setting up Ethereum 2 Proof of Stake Node

### TODO:

- [ ] Expand on setup guide documentation.
  - provide clarifying steps leading up to each command that is run

## Clients Used

- Geth: https://geth.ethereum.org/

  - Documentation: https://geth.ethereum.org/docs/
  - GitHub: https://github.com/ethereum/go-ethereum

- Prysm: https://prysmaticlabs.com/

  - Documentation: https://docs.prylabs.network/docs/getting-started/
  - GitHub: https://github.com/prysmaticlabs/prysm

## Ports:

This guide is buidling out these nodes inside Docker containers. So these ports need to be opened up on the host machine's firewall and in the port forwarding section of your router.

- 8545 - TCP, used by the HTTP based JSON RPC API
- 30303 - TCP and UDP, used by the P2P protocol running the network
- 4000
- 13000
- 12000

## Volumes:

These are the volumes that were created to get(h) this up and running. The names can be changed to whatever you feel is appropriate.

- `geth-volume` - all goerli test net blockchain data
- `prysm-volume` - all prysm beacon-node data
- `prysm-wallet` - where wallet information was saved and this volume was used during the key process (I believe)
- `validator-db` - which will house all of the validator data

## Set up Docker Network

`docker network create --attachable ${network-name}`

- For this example and write up the `${network-name}` is `testnet`

- Be sure that all ports are opened on router and through UFW on host machine, not the containers

## GoEth Node

```bash
docker run -it -v \
geth-volume:/root/.ethereum \
-p 8545:8545 -p 8546:8546 -p 30303:30303 \
--network testnet --name geth-node \
ethereum/client-go:stable \
--goerli \
--http --http.addr geth-node --http.api=web3,eth,personal,miner,net,txpool
```

- `--http.addr` should be the name of the container. this will replicate 'LocalHost'

- all of the `--http.api` s need to be specified

## Beacon Node

- this config was able to allow the beacon node and the validator containers to talk to them selves over the network

```bash
docker run -it -p 4000:4000 -p 13000:13000 -p 12000:12000/udp -v prysm-volume:/data \
--name prysm-beacon-node --network testnet \
gcr.io/prysmaticlabs/prysm/beacon-chain:stable \
--datadir=/data \
--rpc-host=prysm-beacon-node --rpc-port=4000 \
--grpc-gateway-host=prysm-beacon-node \
--http-web3provider=http://172.18.0.2:8545 \
--pyrmont
```

- local IP address and port to the Geth container must be specified in the the `--http-web3provider=` field

- _NOTE: that the the `--rpc-host` `--grpc-gateway-host` must be specified to the name of the container_

## Getting Deposit Ethereum

### Download command line app

Go to https://github.com/ethereum/eth2.0-deposit-cli/releases/

From there download the correct package for your OS and unzip the file.

```
wget ${location-of-file}
tar -xf ${file-name}
```

Move into that file and run

```
./deposit new-mnemonic --num_validators 1 --chain pyrmont
```

Copy `deposit_data.json` to upload to eth2 launchpad to initiate the contract

_this command will pull a file from a remote server to your local machine_

```
scp <remote-user>@<remote-ip>:/path/to/remote/file /path/to/local/destination
```

## Validator Node

<!-- poossibly remove wallet creation -->

make an HD wallet

https://docs.prylabs.network/docs/wallet/deterministic

```
./prysm.sh validator wallet create
```

record information received. store securely

run `acocunt.yaml` file using the `prysm-validator-account-import` image

```
docker-compose -f account.yaml run prysm-validator-account-import
```

_NOTE:
to exit an account run `acocunt.yaml` file using the `prysm-validator-voluntary-exit` image_

```
docker-compose -f account.yaml run prysm-validator-voluntary-exit
```

- after executing the following you will be prompted for the wallet password that was setup previously.

```bash
docker run -it --name prysm-validator-node \
-v $HOME/validator:/data \
-v validator-db:/db \
--network testnet \
gcr.io/prysmaticlabs/prysm/validator:stable \
--datadir=/db \
--beacon-rpc-provider=prysm-beacon-node:4000 \
--wallet-dir /data/wallets
```

- the `--beacon-rpc-provider` needs to be specified to the container name that is running the beacon node. in this example above `prybeacon:4000`
