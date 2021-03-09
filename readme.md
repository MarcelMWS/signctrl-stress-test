# Draft: SignCTRL stress test

## Reference Repo:

[https://github.com/BlockscapeNetwork/signctrl](https://github.com/BlockscapeNetwork/signctrl)

## Preparation

**Target:**
Test different consensus heigth's as happend during last double signing (catching_up: false; different consensus height's)

![pic](SignCTRL-Stress-test.svg)

## Environment

`gaiad version: HEAD-042b7ef3bf07c4dc3d57eb733cd905b5bac22706 (v4.0.5)`

`signctrl version: 71982ac0d618eafec0652474f3298cb9fd5f8ee8`

`two aws t3a.medium with 100G gp2 ssd storage (default iops): signctrl-val-1, signctrl-val-2`

`t3.small with 100G gp2 ssd storage (default iops): signctrl-master`

Local Network with tree nodes:

SignCtrl Master:

signctrl-master: `999999stake` voting power

signctrl-val-1: `494999stake` voting power

signctrl-val-2: `494999stake` voting power (same priv_validator_key.json as signctrl-val-1)

### Important test cmd's

`cd ./build`

### clean keys

`rm -rf keyring-test`

### list accounts

`gaiad keys list --keyring-backend test --keyring-dir .`

### run msg send

`gaiad keys add main --recover --keyring-backend test --keyring-dir .`

`./gaiad tx bank send cosmos1h4u0nh2h7z4lj0v9ekge42wfpaug8dvksznrz4 cosmos10pt62z2vqzes58jkct32pvslr377wn86tz75c4 1stake --keyring-backend test --chain-id sc --generate-only --gas 198310000 --memo blockscape > unsigned.json`

`gaiad tx sign unsigned.json --chain-id sc --keyring-backend test --from main --node tcp://52.59.242.1:26657 > signed.json`

`gaiad tx broadcast signed.json --node tcp://52.59.242.1:26657`

### second node endpoint 

`./gaiad tx bank send cosmos1h4u0nh2h7z4lj0v9ekge42wfpaug8dvksznrz4 cosmos10pt62z2vqzes58jkct32pvslr377wn86tz75c4 1stake --keyring-backend test --chain-id sc --generate-only --gas 198310000 --memo blockscape > unsigned.json`

`gaiad tx sign unsigned.json --chain-id sc --keyring-backend test --from main --node tcp://3.124.188.205:26657 > signed.json`

`gaiad tx broadcast signed.json --node tcp://3.124.188.205:26657`

### view balances

`gaiad q bank balances cosmos1h4u0nh2h7z4lj0v9ekge42wfpaug8dvksznrz4 --node tcp://52.59.242.1:26657`

`gaiad q bank balances cosmos10pt62z2vqzes58jkct32pvslr377wn86tz75c4 --node tcp://52.59.242.1:26656`

### unjail validator

`gaiad tx slashing unjail --from validator --chain-id sc --home /data --memo "THIS IS THE MEMO" --keyring-backend test`

### show validator address (cosmosvalconspub1)

`gaiad tendermint show-validator --home /data`

### show signing infos

`gaiad q slashing signing-info cosmosvalconspub1zcjduepq5ql8rzrle438f400ujelrxyu4jj82yuggwqnm3acxkaretpc82lsvn8w3g`

### stop signctrl and gaia

`sudo systemctl stop gaiad; sleep 0.5s; sudo systemctl stop signctrl`

### restart signctrl and gaia

`sudo systemctl stop gaiad; sleep 0.5s; sudo systemctl stop signctrl; sleep 1s; sudo systemctl start signctrl; sleep 0.5s; sudo systemctl start gaiad`

### start signctrl and gaia

`sudo systemctl start signctrl; sleep 0.5s; sudo systemctl start gaiad; sleep 0.5s; sudo journalctl -feu signctrl`

## Results

## 1. test (08.03.2021)

with 14000 tx msgs and config toml as configured in this commit

sign ctrl swaps to second node:

```
Mar 08 16:27:42 signctrl-val-1 signctrl[3537]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51956
Mar 08 16:27:46 signctrl-val-1 signctrl[3537]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51957
Mar 08 16:27:50 signctrl-val-1 signctrl[3537]: [INFO] signctrl: Missed too many blocks in a row (1/1)
Mar 08 16:27:50 signctrl-val-1 signctrl[3537]: [ERR] signctrl: couldn't handle request: node cannot be promoted anymore, so it must be shut down
Mar 08 16:27:50 signctrl-val-1 signctrl[3537]: [INFO] signctrl: Stopping SignCTRL on rank 1...
Mar 08 16:27:50 signctrl-val-1 signctrl[3537]: [INFO] signctrl: Saving current rank 1 to last_rank.json...
Mar 08 16:27:50 signctrl-val-1 signctrl[3537]: [INFO] signctrl: Shutting SignCTRL down... ⏻ (quit)
```

second node takes over as signer

```
Mar 08 16:27:42 signctrl-val-2 signctrl[3285]: [ERR] signctrl: couldn't handle request: no signing permission for SIGNED_MSG_TYPE_PRECOMMIT on block height 51956 (rank: 2)
Mar 08 16:27:46 signctrl-val-2 signctrl[3285]: [ERR] signctrl: couldn't handle request: no signing permission for SIGNED_MSG_TYPE_PRECOMMIT on block height 51957 (rank: 2)
Mar 08 16:27:50 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Missed too many blocks in a row (1/1)
Mar 08 16:27:50 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Promote validator (2 -> 1)
Mar 08 16:27:50 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51958
Mar 08 16:27:55 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PROPOSAL for block height 51959
Mar 08 16:27:55 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PREVOTE for block height 51959
Mar 08 16:27:55 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51959
Mar 08 16:28:00 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51960
Mar 08 16:28:05 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51961
Mar 08 16:28:10 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PROPOSAL for block height 51962
Mar 08 16:28:10 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PREVOTE for block height 51962
Mar 08 16:28:11 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51962
Mar 08 16:28:16 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51963
Mar 08 16:28:21 signctrl-val-2 signctrl[3285]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51964
```

## 2. test (09.03.2021)

broadcast two 14000 tx msgs to both nodes. both vals breaks


```
Mar 09 10:15:30 signctrl-val-1 signctrl[8125]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 64520
Mar 09 10:15:34 signctrl-val-1 signctrl[8125]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 64521
Mar 09 10:15:38 signctrl-val-1 signctrl[8125]: [INFO] signctrl: Missed too many blocks in a row (1/1)
Mar 09 10:15:38 signctrl-val-1 signctrl[8125]: [ERR] signctrl: couldn't handle request: node cannot be promoted anymore, so it must be shut down
Mar 09 10:15:38 signctrl-val-1 signctrl[8125]: [INFO] signctrl: Stopping SignCTRL on rank 1...
Mar 09 10:15:38 signctrl-val-1 signctrl[8125]: [INFO] signctrl: Saving current rank 1 to last_rank.json...
Mar 09 10:15:38 signctrl-val-1 signctrl[8125]: [INFO] signctrl: Shutting SignCTRL down... ⏻ (quit)
```

```
Mar 09 10:16:14 signctrl-val-2 signctrl[8038]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PREVOTE for block height 64529
Mar 09 10:16:17 signctrl-val-2 signctrl[8038]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 64529
Mar 09 10:16:21 signctrl-val-2 signctrl[8038]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 64530
Mar 09 10:16:25 signctrl-val-2 signctrl[8038]: [INFO] signctrl: Missed too many blocks in a row (1/1)
Mar 09 10:16:25 signctrl-val-2 signctrl[8038]: [ERR] signctrl: couldn't handle request: node cannot be promoted anymore, so it must be shut down
Mar 09 10:16:25 signctrl-val-2 signctrl[8038]: [INFO] signctrl: Stopping SignCTRL on rank 1...
Mar 09 10:16:25 signctrl-val-2 signctrl[8038]: [INFO] signctrl: Saving current rank 1 to last_rank.json...
Mar 09 10:16:25 signctrl-val-2 signctrl[8038]: [INFO] signctrl: Shutting SignCTRL down... ⏻ (quit)
```

## 3. test (09.03.2021) with spammer help

Reference Repo: [https://github.com/MarcelMWS/spammer/tree/BLCS-387-signctrl-stress-test](https://github.com/MarcelMWS/spammer/tree/BLCS-387-signctrl-stress-test)

## spammer script 

ins1 52.59.242.1 ins2 3.124.188.205

### clean workspace

`rm -rf keyring-test/ fatTx.json addrs.json signed.json`

### generate accounts

`spammer fatTx 4500`

### sign

`gaiad tx sign fatTx.json --chain-id sc --keyring-backend test --from main --node tcp://52.59.242.1:26657 > signed.json`

### broadcast

`gaiad tx broadcast signed.json --node tcp://52.59.242.1:26657`

### get account addresses

`gaiad keys list --keyring-backend test --keyring-dir . --output json > addrs.json`

### transfer tx

`spammer bulkTxs`
