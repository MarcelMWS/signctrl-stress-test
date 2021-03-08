### First Test

with 13000 tx msgs and config toml as configured in this commit

sign ctrl swaps to second node:

```
Mar 08 15:09:41 signctrl-val-1 signctrl[2811]: [INFO] signctrl: Signed SIGNED_MSG_TYPE_PRECOMMIT for block height 51049
Mar 08 15:09:45 signctrl-val-1 signctrl[2811]: [INFO] signctrl: Missed too many blocks in a row (1/1)
Mar 08 15:09:45 signctrl-val-1 signctrl[2811]: [ERR] signctrl: couldn't handle request: node cannot be promoted anymore, so it must be shut down
Mar 08 15:09:45 signctrl-val-1 signctrl[2811]: [INFO] signctrl: Stopping SignCTRL on rank 1...
Mar 08 15:09:45 signctrl-val-1 signctrl[2811]: [INFO] signctrl: Saving current rank 1 to last_rank.json...
Mar 08 15:09:45 signctrl-val-1 signctrl[2811]: [INFO] signctrl: Shutting SignCTRL down... ⏻ (quit)
```