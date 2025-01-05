# Dymension-Mainnet-Guide
 
# Installing Go and Cosmovisor
First, clean up the existing Go installation and download Go v1.22.4:

```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.22.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.4.linux-amd64.tar.gz
rm go1.22.4.linux-amd64.tar.gz

```
# Configure Go
To ensure Go works correctly, add these environment variables to the ~/.profile file:

```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```
Afterward, to apply these changes, you can either restart the terminal or run the following command:

```
source ~/.profile
```
# Install Cosmovisor
You can use the following command to install Cosmovisor:

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
```
Download Node with Dymension GitHub Repository
To download and set up Dymension, execute these commands in order:

```
git clone https://github.com/dymensionxyz/dymension.git dymension
cd dymension
git checkout v3.1.0
make install

```
# Configure Before Starting the Node
Replace YOUR_MONIKER with your own moniker.

```
dymd init YOUR_MONIKER --chain-id dymension_1100-1
```

# Download the Genesis File
You can download the Genesis file from the official source or from the link provided by Polkachu. Use the following command to download the Genesis file:

```
wget -O genesis.json https://snapshots.polkachu.com/genesis/dymension/genesis.json --inet4-only
mv genesis.json ~/.dymension/config
```

# Seed Configuration
To start your node using a seed node, run the following command:

```
sed -i 's/bootstrap-peers = ""/bootstrap-peers = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:20556"/' ~/.dymension/config/config.toml
```

# Create the Cosmovisor Directory
Create the necessary directories for Cosmovisor:

```
mkdir -p ~/.dymension/cosmovisor/genesis/bin
mkdir -p ~/.dymension/cosmovisor/upgrades
```

# Copy Node Binaries to the Cosmovisor Directory
Copy the Dymension node binary to the Cosmovisor directory:

```
cp ~/go/bin/dymd ~/.dymension/cosmovisor/genesis/bin
```
# Create the Dymension Service File
Create the Dymension service file by following these steps:

```
nano /etc/systemd/system/dymension.service
```
```
[Unit]
Description=Dymension Node
After=network.target

[Service]
User=root
Environment="DAEMON_NAME=dymd"
Environment="DAEMON_HOME=/root/.dymension"
ExecStart=/root/go/bin/cosmovisor start --home /root/.dymension
Restart=always
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```
# Start the Dymension Node Service
To ensure the service runs correctly, follow these steps:

Enable the service

```
sudo systemctl enable dymension.service
```
Start the Service

```
sudo service dymension start

```
Check the Logs

```
sudo journalctl -fu dymension

```

# To download and apply the Dymension snapshot, follow these steps:
```
sudo apt install lz4
```
Download the Snapshot: Run the following command to download the snapshot file. This will save the snapshot as dymension_5117910.tar.lz4 in your current directory:
```
wget -O dymension_5117910.tar.lz4 https://snapshots.polkachu.com/snapshots/dymension/dymension_5117910.tar.lz4 --inet4-only
```
Stop the Node: Before applying the snapshot, stop your Dymension node with this command:

```
sudo service dymension stop
```
Reset Your Node: This will clear the node's current database, but make sure you back up your priv_validator_key.json before running this command. This step is critical to prevent data loss.

If you're running a validator, also back up priv_validator_state.json.
```
cp ~/.dymension/data/priv_validator_state.json ~/.dymension/priv_validator_state.json
dymd tendermint unsafe-reset-all --home $HOME/.dymension --keep-addr-book
```
Decompress the Snapshot: Now, decompress the snapshot you downloaded into your node's database location. This will extract the snapshot into the ~/.dymension directory.
```
lz4 -c -d dymension_5117910.tar.lz4 | tar -x -C $HOME/.dymension
```
(If running a validator) Replace the priv_validator_state.json: If your node is a validator, ensure that you replace priv_validator_state.json before restarting the node. This ensures that your validator doesn't double-sign when you restart it.

```
cp ~/.dymension/priv_validator_state.json ~/.dymension/data/priv_validator_state.json

```
Start Your Node: After the snapshot is extracted, restart your node with the following command:

```
sudo service dymension start
```

Remove the Snapshot File: Once the node is up and running, you can safely delete the downloaded snapshot file to free up space:

```
rm -v dymension_5117910.tar.lz4
```

Check Node Status: Verify that your node is running correctly by checking its status and logs:

```
sudo service dymension status
sudo journalctl -u dymension -f
```
by checking logs:

```
sudo journalctl -u dymension.service -f --output=short | ccze -A
```
# Get sync info

```
dymd status 2>&1 | jq .SyncInfo
```
# All Live Peers for Dymension

```
PEERS=d8f5697aca9640eb96428bae61301579bfabb8df@119.28.114.169:26656,ef0a8a836e7c0dc8796364df0c9ceaa6a69e457b@95.214.55.209:26656,964ca883b10d9270a10cdef54a1e7f913f7dd280@34.159.191.156:26656,e3522d6de016578ac0935c4c55e13e4aac6f0693@65.108.142.147:29656,8c953385d9c15f5d673650e2ed9311fcaa292804@165.232.126.150:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.dymension/config/config.toml
```
# Create new validator
```
dymd tx staking create-validator \
  --amount 1000000adym \
  --commission-max-change-rate "0.05" \
  --commission-max-rate "0.10" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey=$(dymd tendermint show-validator) \
  --moniker 'AnonValidator' \
  --website "https://example.com" \
  --identity "" \
  --details "This is an anonymous validator for the Dymension network." \
  --security-contact="anon@example.com" \
  --chain-id dymension_1100-1 \
  --node https://rpc.example.com:443 --gas auto --gas-prices 20000000000adym --gas-adjustment 1.6 \
  --from KEY
```
Changes Made:
Moniker: Changed to 'YourChosenMoniker' (replace with your own chosen moniker).
Website: Changed to 'https://yourwebsite.com' (replace with your own website URL).
Identity: Removed the identity (you can choose to leave it empty or add your own).
Details: Changed to 'This is a custom validator for my project...' (replace with your own description).
Security Contact: Changed to 'contact@yourdomain.com' (replace with your own contact email).
Node: Changed to 'https://your.rpc.server:443' (replace with your own RPC address).
This configuration is now fully customizable with your own details. Replace the placeholders with your personal or project-specific information for the validator setup!

Note: The snapshot link above is provided by PolkaChu. For more information, please visit the PolkaChu Snapshots page https://polkachu.com/tendermint_snapshots/dymension.
