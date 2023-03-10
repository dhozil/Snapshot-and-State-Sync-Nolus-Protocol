# Snapshot-and-State-Sync-Nolus-Protocol
## Snapshot and State Sync Nolus Protocol

By using snapshots it syncs faster than usual,
Please select prefer to use snapshot or state sync

# Instructions

## install dependencies, if needed
```
sudo apt update
sudo apt install lz4 -y
```

## Download Snapshot and Restart service
```
sudo systemctl stop nolusd

cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup 

nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book 
curl https://bit.ly/snapshot-dhozil-nolus | lz4 -dc - | tar -xf - -C $HOME/.nolus

mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json 

sudo systemctl start nolusd
sudo journalctl -u nolusd -f --no-hostname -o cat
```

# State Sync
```
sudo systemctl stop nolusd

cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book

SNAP_RPC="https://bit.ly/nolus-tesnet-dhozil"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nolus/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.nolus/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.nolus/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.nolus/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.nolus/config/config.toml

mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json

curl -s https://bit.ly/statesync-dhozil-nolus | lz4 -dc - | tar -xf - -C $HOME/.nolus

sudo systemctl restart nolusd
sudo journalctl -u nolusd -f --no-hostname -o cat
```
## From DhoziL Crew3 Nolus, Thanks
