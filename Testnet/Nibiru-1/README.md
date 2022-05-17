# Nibiru-1 Testnet Instructions

## Minimum hardware requirements

- 2CPU
- 4GB RAM
- 100GB of disk space (SSD)

## Install prerequisites and Nibiru binary


### 1. Update the system

```bash
sudo apt update
sudo apt upgrade --yes
```

### 3. Install prerequisites

```bash
sudo apt install git build-essential ufw curl jq snapd make gcc --yes
```

### 3. Install Golang

wget -q -O - https://git.io/vQhTU | bash -s -- --version 1.18

After the installation open a new terminal to properly load go or run `source /home/<your_user>/.bashrc`

### 4. Install Nibiru from the repository

```bash
cd $HOME
git clone https://github.com/NibiruChain/nibiru && cd nibiru
```
or extract the archive received from the Nibiru team.

In this repository, simply run 
```bash
make build 
make install
```


### 5. Create a nibid service (optional)

1. Create a service file

```bash
sudo tee /etc/systemd/system/nibiru.service<<EOF
[Unit]
Description=Nibiru Node
Requires=network-online.target
After=network-online.target

[Service]
Type=exec
User=<your_user>
Group=<your_user_group>
ExecStart=/home/<your_user>/go/bin/nibid start --home /home/<your_user>/.nibid
Restart=on-failure
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
PermissionsStartOnly=true
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
``` 

2. Enable the service

```bash
systemctl daemon-reload
systemctl start nibiru
systemctl enable nibiru
```

## Create Nibiru-1 Testnet validator

1. Init Chain and start your node

   ```bash
   nibid init <moniker-name> --chain-id=nibiru-1 --home /home/<your_user>/.nibid
   ```

2. Create a local key pair

   ```bash
   nibid keys add <key-name>
   nibid keys show <key-name> -a
   ```

3. Download genesis file
   Fetch `genesis.json` into `nibid`'s `config` directory.

   ```bash
   curl https://<your_github_access_token>@raw.githubusercontent.com/NibiruChain/Networks/main/Testnet/genesis.json > $HOME/.nibid/config/genesis.json
   ```

   **Genesis.json sha256**

   ```bash
    shasum -a 256 ~/.nibid/config/genesis.json
    3bf2364aad2739adfdc4e377fc61b8310894b16c4122c3594f676394df064c22  /home/<user>/.nibid/config/genesis.json
   ```
   
   Or copy the one included in the archive received from the Nibiru Team to `$HOME/.nibid/config` folder
   
4. Update persistent peers list in the configuration file $HOME/.nibid/config/config.toml with the ones from the persistent_peers.txt

5. Start your node with  `nibid start` or `systemctl start nibiru` if you've created a service for it. Make sure it synced up to the tip.

6. Create validator

   ```bash
   nibid tx staking create-validator \
   --amount 50000unibi \
   --commission-max-change-rate "0.1" \
   --commission-max-rate "0.20" \
   --commission-rate "0.1" \
   --min-self-delegation "1" \
   --details "put your validator description there" \
   --pubkey=$(nibid tendermint show-validator) \
   --moniker <your_moniker> \
   --chain-id nibiru-1 \
   --gas-prices 0.025unibi \
   --from <key-name>
   ```

7. You can equest tokens from the [Web Faucet for Nibiru-1 Testnet](http://ec2-35-172-193-127.compute-1.amazonaws.com:8003/) if required.

Example:
```bash
curl -X POST -d '{"address": "your address here", "coins": ["50000unibi"]}' http://ec2-35-172-193-127.compute-1.amazonaws.com:8003
```
Please note, that current Testnet Faucet limit is 1000000unibi

8. Verify your validator status via [Nibiru-1 Testnet Block Explorer](http://ec2-54-221-169-63.compute-1.amazonaws.com:3003/validators)
