# Setting up Ethereum 2 Proof of Stake Node

### TODO:

- [ ] Configure `goethnode` and `pryvol` to only store directories that are needed and not entire container
- [ ] Re-write terminal command attributes/flags to be more generic.
  - take out hard coded values like ip addresses and replace with variables
  - provide an explanation of what that value should be
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

- 8545
- 8546
- 30303
- 4000
- 13000
- 12000

## Volumes:

These are the volumes that were created to get(h) this up and running. The names can be changed to whatever you feel is appropriate.

- `goethnode` - all goerli test net blockchain data
- `pryvol` - all prysm beacon-node data
- `prywallet` - where wallet information was saved and this volume was used during the key process (I believe)
- `validatordb` - which will house all of the validator data

## Set up Docker Network

`docker network create --attachable ${network-name}`

- For this example and write up the `${network-name}` is `testnet`

- Be sure that all ports are opened on router and through UFW on host machine, not the containers

## GoEth Node

```bash
docker run -it -v \
goethnode:/root \
-p 8545:8545 -p 8546:8546 -p 30303:30303 \
--network medalla --name goethnode \
f7e7081def4e \
--goerli \
--http --http.addr goethnode --http.api=web3,eth,personal,miner,net,txpool
```

- `--http.addr` should be the name of the container. this will replicate 'LocalHost'

- all of the `--http.api` s need to be specified

## Beacon Node

- this config was able to allow the beacon node and the validator containers to talk to them selves over the network

```bash
docker run -it -p 4000:4000 -p 13000:13000 -p 12000:12000/udp -v pryvol:/data \
--name prybeacon --network medalla \
gcr.io/prysmaticlabs/prysm/beacon-chain:stable \
--datadir=/data \
--rpc-host=prybeacon --rpc-port=4000 \
--grpc-gateway-host=prybeacon \
--http-web3provider=http://${host-ip}:8545 \
--pyrmont
```

- local IP address and port to the Geth container must be specified in the the `--http-web3provider=` field

- _NOTE: that the the `--rpc-host` `--grpc-gateway-host` must be specified to the name of the container_

## Validator Node

- after executing the following you will be prompted for the wallet password that was setup previously.

```bash
docker run -ti --name validator \
-v prywallet:/wallet \
-v validatordb:/data \
--network medalla \
gcr.io/prysmaticlabs/prysm/validator:stable \
--datadir=/data \
--beacon-rpc-provider=prybeacon:4000 \
--wallet-dir /wallet
```

- the `--beacon-rpc-provider` needs to be specified to the container name that is running the beacon node. in this example above `prybeacon:4000`
