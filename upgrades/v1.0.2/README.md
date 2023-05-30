> ## This is security update. It will not break consensus state so please apply it ASAP.


# Upgrade Neutron from v1.0.1 to v1.0.2

### Release Details
* https://github.com/neutron-org/neutron/releases/tag/v1.0.2


# Create the updated Neutron binary of v1.0.2

## Go to neutron directory if present else clone the repository

```shell
   git clone https://github.com/neutron-org/neutron.git
```

## Follow these steps if neutron repo already present

```shell
   cd $HOME/neutron
   git pull
   git fetch --tags
   git checkout v1.0.2
   make install
```

## Check the new neutron version, verify the latest commit hash
```shell
   $ neutrond version --long
   name: neutron
   server_name: neutrond
   version: 1.0.2
   commit: 3c8dde1ff524551e24295d393a3913c25199d265
   ...
```

## Or check checksum of the binary if you decided to download it

```shell
$ shasum -a 256 neutrond-linux-amd64
19b5a6cd4e237706989452ccd9f5d6ef0b44d28b69927d0be912b93859c10885  neutrond-linux-amd64
```

## Copy the new neutron (v1.0.2) binary to cosmovisor current directory

```shell
   cp $GOPATH/bin/neutrond ~/.neutrond/cosmovisor/current/bin
```

# Recommended configurations

in config.toml:
```toml
timeout_commit = "1s”
```

in app.toml:
```toml
minimum-gas-prices = “0.01untrn,0.005ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9,0.05ibc/F082B65C88E4B6D5EF1DB243CDA1D331D002759E938A0F5CD3FFDC5D53B3E349"
```

### Timeout commit

Timeout commit corresponds to how long we wait after committing a block, before starting on the new height. It is intended to ensure there is enough time after reaching 2/3 pre-commits for all validators to join before the next block is started. 

Having tested numerous thresholds on nodes across the world with various latencies, including on the Replicated Security testnet, we found that 1s was a conservative parameter that would improve blocktime and therefore user experience without negatively affecting uptime.

Once all validators update their config, this should help bring blocktime down to 2.5-3s from its current 5.4s.

### Gas fees

Neutron’s economic model depends on transaction fees. In general, we recommend setting the minimum fee to around $0.05 which we believe is a good compromise between accessibility and revenue for Neutron and the Hub. 

To provide a better onboarding experience and diversified revenue streams, we recommend setting the minimum-gas-prices in app.toml for three denominations:

* uNTRN: the native token
* ATOM: ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9
* axlUSDC: ibc/F082B65C88E4B6D5EF1DB243CDA1D331D002759E938A0F5CD3FFDC5D53B3E349

You can use the following line to set the gas prices to roughly $0.05 for ATOM and axlUSDC (NTRN will need to be adjusted after the launch event, once it has a price):

```toml
# Sets the fees to 0.01 NTRN, 0.005 ATOM and 0.05 USDC
minimum-gas-prices = "0.01untrn,0.005ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9,0.05ibc/F082B65C88E4B6D5EF1DB243CDA1D331D002759E938A0F5CD3FFDC5D53B3E349"
```
