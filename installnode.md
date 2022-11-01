# open port 9151
```
sudo ufw allow 9151
#if you don't have ufw installed, run install like below to open port
sudo apt-get install ufw
sudo ufw allow 9151
```
# install extras and necessary packages
```
sudo apt-get update && apt-get upgrade -y
sudo apt-get -y install libssl-dev && apt-get -y install cmake build-essential git wget jq make gcc
```
note that nulink needs Python 3.8.10 or higher . check
```
python3 -V
```
# Download Geth
```
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.10.24-972007a5.tar.gz
tar -xvzf geth-linux-amd64-1.10.24-972007a5.tar.gz
cd geth-linux-amd64-1.10.24-972007a5/
./geth account new --keystore ./keystore  
./geth account update --keystore ./keystore
```
#enter password , backup public key and secret key
# Docker install and update
```
cd /root
sudo apt install docker.io -y
sudo systemctl enable --now docker
```
# Pull the latest NuLink image
```
docker pull nulink/nulink:latest
```
# Create a directory in your host machine for later usage
```
cd /root && mkdir nulink
```
- Copy the keystore file of the Worker account
```
cp <SECRET-KEY-FILE-PATH> /root/nulink
#example
cp /root/geth-linux-amd64-1.10.24-972007a5/keystore/UTC--2022-11-01T08-06-44.839473505Z--bdd06aa044399bca994178bd706d36c097943ae5 /root/nulink
```
```
chmod -R 777 /root/nulink
```
- Export Node Environment
```
export NULINK_KEYSTORE_PASSWORD=<YOUR NULINK STOREAGE PASSWORD>
export NULINK_OPERATOR_ETH_PASSWORD=<YOUR WORKER ACCOUNT PASSWORD>
```
# Initialize Node Configuration
```
docker run -it --rm \
-p 9151:9151 \
-v </path/to/host/machine/directory>:/code \
-v </path/to/host/machine/directory>:/home/circleci/.local/share/nulink \
-e NULINK_KEYSTORE_PASSWORD \
nulink/nulink nulink ursula init \
--signer <ETH KEYSTORE URI> \
--eth-provider <NULINK PROVIDER URI>  \
--network <NULINK NETWORK NAME> \
--payment-provider <PAYMENT PROVIDER URI> \
--payment-network <PAYMENT NETWORK NAME> \
--operator-address <WORKER ADDRESS> \
--max-gas-price <GWEI>
```
example
```
docker run -it --rm \
-p 9151:9151 \
-v /root/nulink:/code \
-v /root/nulink:/home/circleci/.local/share/nulink \
-e NULINK_KEYSTORE_PASSWORD \
nulink/nulink nulink ursula init \
--signer keystore:///code/UTC--2022-11-01T08-06-44.839473505Z--bdd06aa044399bca994178bd706d36c097943ae5 \
--eth-provider https://data-seed-prebsc-2-s2.binance.org:8545 \
--network horus \
--payment-provider https://data-seed-prebsc-2-s2.binance.org:8545 \
--payment-network bsc_testnet \
--operator-address 0xbdD06AA044399bCA994178BD706D36C097943aE5 \
--max-gas-price 100
```
##be careful to transfer a small amount of tBNB to operator-address before starting
# start run
```
docker run --restart on-failure -d \  
--name ursula \
-p 9151:9151 \
-v </path/to/host/machine/directory>:/code \
-v </path/to/host/machine/directory>:/home/circleci/.local/share/nulink \
-e NULINK_KEYSTORE_PASSWORD \
-e NULINK_OPERATOR_ETH_PASSWORD \
nulink/nulink nulink ursula run --no-block-until-ready
```
example
```
docker run --restart on-failure -d \
--name ursula \
-p 9151:9151 \
-v /root/nulink:/code \
-v /root/nulink:/home/circleci/.local/share/nulink \
-e NULINK_KEYSTORE_PASSWORD \
-e NULINK_OPERATOR_ETH_PASSWORD \
nulink/nulink nulink ursula run --no-block-until-ready
```
# check log
```
docker logs -f ursula
```
- go to https://testnet.binance.org/faucet-smart faucet BNB test
- go to https://test-staking.nulink.org/ link meta wallet switch to chain Binance Smart Chain Testnet faucet NLK , stake --> bonder worker --> enter ip address and url eg https://161.97.73.21:9151 --> confirm
