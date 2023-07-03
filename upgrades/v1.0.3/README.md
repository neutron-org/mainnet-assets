> ## This is an important security update. IT IS CONSENSUS BREAKING, so please apply the fix only on height <TODO_HEIGHT>.


# Upgrade Neutron from v1.0.2 to v1.0.3

### Release Details
* https://github.com/neutron-org/neutron/releases/tag/v1.0.3
* Chain upgrade height : <TODO_HEIGHT>. Exact upgrade time can be checked [here](https://mintscan.io/neutron/blocks/<TODO_HEIGHT>).
* Go version has been frozen at `1.20`. If you are going to build Neutron binary from source, make sure you are using the right GO version!

# To upgrade neutron chain

## Step 1: Alter systemd service configuration

We need to disable automatic restart of the node service. To do so please alter your `neutrond.service` file configuration and set appropriate lines to following values.

```
Restart=no 
RestartSec=3      <- remove line

Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=false"
```

There is no need to restart the node yet; these changes will get applied during the node restart in the next step.

## Step 2: Restart neutrond with a configured `halt-height`.

This upgrade requires `neutrond` to have knowledge of the planned halt height. Please be aware that there is an extra step at the end to revert to `neutrond`'s original configurations.

There are two mutually exclusive options for this stage:

### Option 1: Set the halt height by modifying `app.toml`

* Stop the neutrond process.

* Edit the application configuration file at `~/.neutron/config/app.toml` so that `halt-height` reflects the upgrade plan:

```toml
# Note: Commitment of state will be attempted on the corresponding block.
halt-height = <TODO_HEIGHT>
```

* Wait for the upgrade height, and proceed to Step 2.

### Option 2: Restart the `neutrond` binary with command line flags

* Stop the neutrond process.

* Do not modify `app.toml`. Restart the `neutrond` process with the flag `--halt-height`:
```shell
neutrond --halt-height <TODO_HEIGHT>
```

* Wait for the upgrade height and confirm that the node has halted

After performing these steps, the upgrade will proceed as usual using Cosmovisor.

# Setup Cosmovisor

## Create the updated Neutron binary of v1.0.3

* Go to neutron directory if present else clone the repository

```shell
   git clone https://github.com/neutron-org/neutron.git
```

* Follow these steps if neutron repo already present

```shell
   cd $HOME/neutron
   git pull
   git fetch --tags
   git checkout v1.0.3
   make install
```

## Check current neutron version
```shell
   ~/.neutrond/cosmovisor/current/bin/neutrond version
   # Output should be
   1.0.2
```

## Check the new neutron version, verify the latest commit hash

```shell
  $ neutrond version --long
  name: neutron
  server_name: neutrond
  version: 1.0.3
  commit: <TODO_COMMIT>
```

## Copy the new neutron (v1.0.3) binary to cosmovisor current directory

```shell
   cp $GOPATH/bin/neutrond ~/.neutrond/cosmovisor/current/bin
```

## Revert `neutrond` configurations

Depending on which path you chose for Step 1, either:

* Reset `halt-height = 0` option in the `app.toml` or
* Remove it from start parameters of the neutrond binary and start node again

# Recommended configurations

in config.toml:
```toml
timeout_commit = "1s”
```

in app.toml:
```toml
minimum-gas-prices = "0.5untrn,0.022ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9,0.25ibc/F082B65C88E4B6D5EF1DB243CDA1D331D002759E938A0F5CD3FFDC5D53B3E349"
```

### Timeout commit

Timeout commit corresponds to how long we wait after committing a block, before starting on the new height. It is intended to ensure there is enough time after reaching 2/3 pre-commits for all validators to join before the next block is started.

Having tested numerous thresholds on nodes across the world with various latencies, including on the Replicated Security testnet, we found that 1s was a conservative parameter that would improve blocktime and therefore user experience without negatively affecting uptime.

Once all validators update their config, this should help bring blocktime down to 2.5-3s from its current 5.4s.

### Gas fees

Neutron’s economic model depends on transaction fees. In general, we recommend setting the minimum fee to around $0.05 for an average transaction which we believe is a good compromise between accessibility and revenue for Neutron and the Hub.

To provide a better onboarding experience and diversified revenue streams, we recommend setting the minimum-gas-prices in app.toml for three denominations:

* uNTRN: the native token
* ATOM: ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9
* axlUSDC: ibc/F082B65C88E4B6D5EF1DB243CDA1D331D002759E938A0F5CD3FFDC5D53B3E349

You can use the following line to set the gas prices to roughly $0.05 for ATOM, uNTRN and axlUSDC:

```toml
# Sets the fees to 0.5 NTRN, 0.022 ATOM and 0.25 USDC
minimum-gas-prices = "0.5untrn,0.022ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9,0.25ibc/F082B65C88E4B6D5EF1DB243CDA1D331D002759E938A0F5CD3FFDC5D53B3E349"
```



