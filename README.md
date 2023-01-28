<h2>Basic tools for a validator node in Celestia</h2>
<h3>The first modular blockchain</h3>
<img src='https://user-images.githubusercontent.com/2853158/208461248-2c3f7414-c66e-4b6c-9c06-7f0c1c4e1851.png'>


<h3>📌 Overview</h3>
<p>Celestia is the first modular blockchain with a scalable general purpose data availability layer for decentralised applications and trust-minimised sidechains.</p>

<p>The core idea of Celestia is to decouple transaction execution (and validity) from the consensus layer, so that the consensus is only responsible for a) ordering transactions and b) guaranteeing their data availability. They believe that is the next generation of scalable blockchain architectures.</p>

<p>TIA is the native token of Celestia.</p>

<p>Chain ID: mocha | arabica</p>

<h3>🌐 Useful links</h3>
● Website: https://celestia.org/

● GitHub: https://github.com/celestiaorg

● Celestia documentation: https://docs.celestia.org/

● Frequently Asked Questions: https://celestia.org/faq/

● Cumulo Spanish Resources: http://cumulo.pro/celestia.html

<h4>Social media:</h4>
● Twitter: https://twitter.com/CelestiaOrg

● Telegram: https://t.me/CelestiaCommunity

● Discord: https://discord.gg/celestiacommunity

● Blog: https://blog.celestia.org/

● LinkedIn: https://www.linkedin.com/company/celestiaorg/

● Youtube: https://www.youtube.com/@celestia7066

<h4>Explorers:</h4>
● https://celestia.explorers.guru/

● https://testnet-explorer.brocha.in/celestia/

<h3>⚙️Hardware requirements</h3>
● Memory: 8 GB RAM

● CPU: Quad-Core

● Disk: 250 GB of storage

● Bandwidth: 1 Gbps for Download/100 Mbps for Upload

<h3>🛠 Manual installation</h3>

<h4>Updating packages and installing dependencies</h4>
<pre>sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y</pre>

<h4>Environment variables</h4>
To set environment variables replace your wallet and moniker < YOUR_WALLET_NAME> < YOUR_MONIKER> without <> , and save and import the variables into the system.

<pre>echo "export CELESTIA_WALLET="<YOUR_WALLET_NAME>"" >> $HOME/.bash_profile
echo "export CELESTIA_MONIKER="<YOUR_MONIKER>"" >> $HOME/.bash_profile
echo "export CELESTIA_CHAIN_ID="mocha"" >> $HOME/.bash_profile
source $HOME/.bash_profile</pre>

<h4>Install go</h4>
<pre>cd $HOME
VER="1.19.1"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm -rf  "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version</pre>

<h4>Download and compile binaries</h4>
<pre>cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app/
APP_VERSION=v0.11.0
git checkout tags/$APP_VERSION -b $APP_VERSION
make install</pre>

<h4>Setup the P2P networks</h4>
<pre>cd $HOME
rm -rf networks
git clone https://github.com/celestiaorg/networks.git</pre>

<h4>Configure and start the application</h4>
<pre>celestia-appd config chain-id mocha
celestia-appd init $CELESTIA_MONIKER --chain-id mocha</pre>

<h4>Download genesis</h4>
<pre>cp $HOME/networks/mocha/genesis.json $HOME/.celestia-app/config</pre>

<h4>Set seeds and peers</h4>

<p>Peers</p>

<pre>peers=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mocha/peers.txt | tr -d '\n')
bootstrap_peers=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mocha/bootstrap-peers.txt | tr -d '\n')
sed -i.bak -e 's|^bootstrap-peers *=.*|bootstrap-peers = "'"$bootstrap_peers"'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.celestia-app/config/config.toml</pre>

<p>You can find more peers here:
https://github.com/celestiaorg/networks/blob/master/mocha/peers.txt</p>

<h4>Configure pruning</h4>
<pre>
PRUNING="custom"
PRUNING_KEEP_RECENT="100"
PRUNING_INTERVAL="10"
 
sed -i -e "s/^pruning *=.*/pruning = \"$PRUNING\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$PRUNING_KEEP_RECENT\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$PRUNING_INTERVAL\"/" $HOME/.celestia-app/config/app.toml</pre>

<h4>Set the minimum gas price</h4>
<pre>sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utia\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.celestia-app/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.celestia-app/config/config.toml</pre>

<h4>Activate prometheus<h4>
<pre>sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.celestia-app/config/config.toml</pre>

<h4>Clean up old data<h4>
<pre>celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app --keep-addr-book</pre>

 
<h3>♻️ Fast synchronisation with snapshot (Optional)</h3>
<p>You can sync your Celestia node quickly by downloading a recent snapshot:</p>

<pre>cd $HOME
rm -rf ~/.celestia-app/data
mkdir -p ~/.celestia-app/data
SNAP_NAME=$(curl -s https://snaps.qubelabs.io/celestia/ | \
    egrep -o ">mocha.*tar" | tr -d ">")
wget -O - https://snaps.qubelabs.io/celestia/${SNAP_NAME} | tar xf - \
    -C ~/.celestia-app/data/
   </pre>

<h3>🛎 Service with SystemD</h3>
Create the service file
<pre>
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-appd.service
[Unit]
Description=celestia-appd Cosmos daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/celestia-appd start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF
</pre>
 
<h4>Enable, start service and check logs</h4>
<pre>
sudo systemctl daemon-reload
sudo systemctl enable celestia-appd
sudo systemctl restart celestia-appd 
sudo journalctl -u celestia-appd -f</pre>

<h3>🗝Wallet</h3>

<h4>Create wallet</h4>
<p>Don’t forget to save the mnemonic.</p>

<pre>celestia-appd keys add $CELESTIA_WALLET</pre>

<h4>Restore wallet</h4>
<pre>celestia-appd keys add $CELESTIA_WALLET --recover</pre>

<h4>Deposit funds into your wallet</h4>
<p>Before creating a validator, you have to deposit funds in your wallet, go to the OKP4 discord server and request them in the faucet channel.</p>

<pre>$request <address wallet></pre>

<h3>📝Validator</h3>

<h4>Save the wallet and validator address</h4>
<pre>OKP4_WALLET_ADDRESS=$(okp4d keys show $OKP4_WALLET -a)
OKP4_VALOPER_ADDRESS=$(okp4d keys show $OKP4_WALLET --bech val -a)
echo "export OKP4_WALLET_ADDRESS="${OKP4_WALLET_ADDRESS} >> $HOME/.bash_profile
echo "export OKP4_VALOPER_ADDRESS="${OKP4_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile</pre>

<h4>Configure the QGB Keys</h4>
<p>This step helps you prepare for when the Quantum Gravity Bridge is ready to be deployed. You need to perform this step before running a validator, by configuring 2 additional keys.</p>
● EVM-address: This flag must contain a 0x EVM address. Here, you can add any Ethereum-based address. You can also modify it later if you decide to change your address. To get this wallet address you can use a MetaMask address or an Ethereum wallet.<br>
● orchestrator-address: This flag must contain a newly generated Celestia address. Validators can use their existing Celestia addresses, but it is recommended to create a new one:

<pre>celestia-appd keys add ${CELESTIA_WALLET}_2</pre>

<p>You can set both values as environment variables:</p>
<pre>ERC20_ADDRESS="0xdd5xxxxxxxxxxxxxxxx3a2720840001bC5F87D7x"
CELESTIA_WALLET_ADDRESS=$(celestia-appd keys show $CELESTIA_WALLET -a)
CELESTIA_VALOPER_ADDRESS=$(celestia-appd keys show $CELESTIA_WALLET --bech val -a)
ORCHESTRATOR_ADDRESS=$(celestia-appd keys show ${CELESTIA_WALLET}_2 -a)
echo "export CELESTIA_WALLET_ADDRESS="${CELESTIA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo "export CELESTIA_ORCHESTRATOR_ADDRESS="${ORCHESTRATOR_ADDRESS} >> $HOME/.bash_profile
echo "export CELESTIA_VALOPER_ADDRESS="${CELESTIA_VALOPER_ADDRESS} >> $HOME/.bash_profile
echo "export EVM_ADDRESS=""$ERC20_ADDRESS" >> $HOME/.bash_profile
source $HOME/.bash_profile</pre>

<h4>Create validator</h4>
<p>Before creating a validator, you should make sure that the node is synchronised and check the balance of your wallet.</p>

<h5>Check synchronisation status</h5>
<p>Once your node is fully synchronised, the output will read false.</p>

<pre>celestia-appd status 2>&1 | jq .SyncInfo</pre>

<h5>Check your balance</h5>
<pre>celestia-appd query bank balances $CELESTIA_WALLET_ADDRESS</pre>

<h5>Create validator</h5>
<pre>celestia-appd tx staking create-validator \
  --amount 1000000utia \
  --from $CELESTIA_WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(celestia-appd tendermint show-validator) \
  --moniker $CELESTIA_MONIKER \
  --chain-id $CELESTIA_CHAIN_ID \
  --evm-address "$EVM_ADDRESS" \
  --orchestrator-address "$CELESTIA_ORCHESTRATOR_ADDRESS" \
  --gas auto \
  --gas-adjustment 1.3 \
  --fees 1000utia</pre>

<p>You can add the flags — website — security-contact — identity — details (optional)</p>

<pre>--website <WEB_SITE_URL> \
--security-contact <YOUR_CONTACT> \
--identity <KEYBASE_IDENTITY> \
--details <YOUR_VALIDATOR_DETAILS></pre>

<h3>🎛 Monitoring</h3>
<p>If you want to set up a monitoring and alerting system use our guide to monitoring Cosmos nodes with tenderduty.</p>

<h3>🛎 SystemD commands</h3>
<h4>Stop the service</h4>
<pre>sudo systemctl stop celestia-appd</pre>
<h4>Start service</h4>
<pre>sudo systemctl start celestia-appd</pre>
<h4>Restart service</h4>
<pre>sudo systemctl restart celestia-appd</pre>
<h4>Check logs</h4>
<pre>sudo journalctl -u celestia-appd -f</pre>
<h4>Check status</h4>
<pre>sudo systemctl status celestia-appd</pre>

<h3>📈 Node information</h3>
<h4>Synchronization information</h4>
<pre>celestia-appd status 2>&1 | jq .SyncInfo</pre>
<h4>Node information</h4>
<pre>celestia-appd status 2>&1 | jq .NodeInfo</pre>
<h4>Validator information</h4>
<pre>celestia-appd status 2>&1 | jq .ValidatorInfo</pre>
<h4>Get peers</h4>
<pre>echo $(celestia-appd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.celestia-appd/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')</pre>

<h3>🔏 Wallet operation</h3>
<h4>Check balance</h4>
<pre>celestia-appd query bank balances $CELESTIA_WALLET_ADDRESS</pre>
<h4>Wallet Key List</h4>
<pre>celestia-appd keys list</pre>
<h4>Create a new wallet</h4>
<pre>celestia-appd keys add $CELESTIA_WALLET</pre>
<h4>Wallet recovering</h4>
<pre>celestia-appd keys add $CELESTIA_WALLET --recover</pre>
<h4>Delete wallet</h4>
<pre>celestia-appd keys delete $CELESTIA_WALLET</pre>
<h4>Transfer funds</h4>
<pre>celestia-appd tx bank send $CELESTIA_WALLET_ADDRESS <ANOTHER_CELESTIA_WALLET_ADDRESS> 800000000utia --gas auto --gas-adjustment 1.3</pre>

<h3>💬 Governance</h3>
<h4>List all proposals</h4>
<pre>celestia-appd query gov proposals</pre>
<h4>Vote YES</h4>
<pre>celestia-appd tx gov vote 1 yes --from $CELESTIA_WALLET --chain-id $CELESTIA_CHAIN_ID --gas-adjustment 1.4 --gas auto -y</pre>
<h4>Vote NO</h4>
<pre>celestia-appd tx gov vote 1 no --from $CELESTIA_WALLET --chain-id $CELESTIA_CHAIN_ID --gas-adjustment 1.4 --gas auto -y</pre>
<h4>Refrain</h4>
<pre>celestia-appd tx gov vote 1 abstain --from $CELESTIA_WALLET --chain-id $CELESTIA_CHAIN_ID --gas-adjustment 1.4 --gas auto -y</pre>

<h3>🚰 Staking, delegation and rewards</h3>
<h4>Withdraw all rewards</h4>
<pre>celestia-appd tx distribution withdraw-all-rewards --from $CELESTIA_WALLET --chain-id $CELESTIA_CHAIN_ID --gas auto --gas-adjustment 1.3</pre>
<h4>Withdraw commission</h4>
<pre>celestia-appd tx distribution withdraw-rewards $CELESTIA_VALOPER_ADDRESS --from $CELESTIA_WALLET --commission --fees 20utia</pre>
<h4>Delegate Stake</h4>
<pre>celestia-appd tx staking delegate $CELESTIA_VALOPER_ADDRESS 10000000utia --from $CELESTIA_WALLET --chain-id $CELESTIA_CHAIN_ID --gas=auto --gas-adjustment 1.3</pre>

<h3>✔️ Validator operation</h3>
<h4>Edit validator</h4>
<pre>celestia-appd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<keybase_id> \
  --website="<website>" \
  --details="<validator_description>" \
  --chain-id=$CELESTIA_CHAIN_ID \
  --from=$CELESTIA_WALLET
  --fees=200utia</pre>
  
<h3>Validator information</h3>
<pre>celestia-appd status 2>&1 | jq .ValidatorInfo</pre>
<h4>Jailing information</h4>
<pre>celestia-appd q slashing signing-info $(celestia-appd tendermint show-validator)</pre>
<h4>Validator unjailing</h4>
<pre>celestia-appd tx slashing unjail --broadcast-mode=block --from $CELESTIA_WALLET --chain-id $CELESTIA_CHAIN_ID --gas auto --gas-adjustment 1.5</pre>
 
<h3>🗑 Delete node</h3>
<pre>sudo systemctl stop celestia-appd
sudo systemctl disable celestia-appd
sudo rm -rf /etc/systemd/system/celestia-appd*
sudo systemctl daemon-reload
sudo rm $(which celestia-appd)
sudo rm -rf $HOME/.celestia-appd
sudo rm -fr $HOME/ celestia-appd
sed -i "/CELESTIA_/d" $HOME/.bash_profile</pre>

<h3>Authors: Sami & Mon</h3>
http://cumulo.pro/
