### Check consumer key

This instruction will help you to check what validator key will be used during Neutron chain start time.

First of all please check whether you use separate consumer key or not.

```
$ GAIA_VALCONSADDR=$(gaiad tendermint show-address)
$ gaiad q provider validator-consumer-key neutron-1 $GAIA_VALCONSADDR
consumer_address: "<empty or used consumer address>"
```

Next please find your Cosmos Hub validator public key

```
$ gaiad tendermint show-validator
{"@type":"/cosmos.crypto.ed25519.PubKey","key":"<VALIDATOR PUBLIC KEY>"}
```

If your `consumer_address` is empty then please simply check that your `<VALIDATOR PUBLIC KEY>` presented in the Neutron genesis
```
$ cat ~/.neutrond/config/genesis.json | grep <VALIDATOR PUBLIC KEY> -B 2 -A 3
{
    "pub_key": {
       "ed25519": "0wQ1SJucRxDeAVwCQzFinkm3K0UkOlnkf5Ll+BluDQA="
    },
    "power": "993214"
},
```

or if `consumer_address` contains address then you need to check was it added to the genesis or `<VALIDATOR PUBLIC KEY>` was used in it. To do so please run above command with `consumer_address` public key or with `<VALIDATOR PUBLIC KEY>` to find which address was used.

With this set of commands you can be sure what validator key to use during network start.

> If you will see that `consumer_address` is not empty but `<VALIDATOR PUBLIC KEY>` was added to the genesis it means that you added `consumer_address` after network spawn time (2023-05-08 11:00:00UTC) and should use your Cosmos Hub validator key until validator set update. After you will see that your Neutron validator started to miss blocks you need to update validator key with new one that was used to add `consumer_address`.
