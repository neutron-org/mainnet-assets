---
title: Neutron v2.0.0 Upgrade
order: 2
---
<!-- markdown-link-check-disable -->

# Neutron v2.0.0 Upgrade, Instructions

* Chain upgrade point: `20th of December 15:30 UTC` approximately at height `5416000`;
* Go version: `v1.20`
* Release: https://github.com/neutron-org/neutron/releases/tag/v2.0.0
* `minimum-gas-prices` app.toml parameter is mandatory starting from the release. Please set recommended value as `minimum-gas-prices = "0.56untrn,0.02ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9,0.2ibc/F082B65C88E4B6D5EF1DB243CDA1D331D002759E938A0F5CD3FFDC5D53B3E349"`. You get an error `Error: set min gas price in app.toml or flag or env variable: error in app.toml` if you have not set it.
* If you use your node as grpc/rest endpoint its strongly recommended to limit wasm cache size by setting `memory_cache_size = 400` (or any reasonable value in MiB) in section `[wasm]` `app.toml` file to avoid memory leak in wasmvm cache system.

```
$ tail -2 .neutrond/config/app.toml
[wasm]
memory_cache_size = 400
```

This document describes the steps for validators and full node operators, to upgrade successfully to the Neutron v2.0.0 release. For more details on the release, please see the [release notes](https://github.com/neutron-org/neutron/releases/tag/v2.0.0).

## Upgrade date

The upgrade will take place approximately at `15:30 UTC` approximately at height `5416000`;

## Chain-id will remain the same

The chain-id of the network will remain the same, `neutron-1`. This is because an in-place migration of state will take place, i.e., this upgrade does not export any state.

### System requirement

* 64GB RAM is recommended to ensure a smooth upgrade.

If you have less than 64GB RAM, you might try creating a swapfile to swap an idle program onto the hard disk to free up memory. This can allow your machine to run the binary than it could run in RAM alone.

```shell
sudo fallocate -l 64G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

* Make sure you have enough disk space for upgrade, the state can grow twice durng upgrade.

### Backups

Prior to the upgrade, validators are encouraged to take a full data snapshot. Snapshotting depends heavily on infrastructure, but generally this can be done by backing up the `.neutrond` directory.
If you use Cosmovisor to upgrade, by default, Cosmovisor will backup your data upon upgrade. See below [upgrade using cosmovisor](#method-ii-upgrade-using-cosmovisor) section.

It is critically important for validator operators to back-up the `.neutrond/data/priv_validator_state.json` file after stopping the neutrond process. This file is updated every block as your validator participates in consensus rounds. It is a critical file needed to prevent double-signing, in case the upgrade fails and the previous chain needs to be restarted.

### Current runtime

The Neutron mainnet network, `neutron-1`, is currently running [Neutron v1.0.4](https://github.com/neutron-org/neutron/releases/tag/v1.0.4). We anticipate that operators who are running on v1.0.4, will be able to upgrade successfully. Validators are expected to ensure that their systems are up-to-date and capable of performing the upgrade. This includes running the correct binary, or if building from source, building with go `1.20`.

### Target runtime

The Neutron mainnet network, `neutron-1`, will run [Neutron v2.0.0](https://github.com/neutron-org/neutron/releases/tag/v2.0.0). Operators _**MUST**_ use this version post-upgrade to remain connected to the network.

## Upgrade steps

There are 2 major ways to upgrade a node:

- Manual upgrade
- Upgrade using [Cosmovisor](https://pkg.go.dev/cosmossdk.io/tools/cosmovisor)
    - Either by manually preparing the new binary
    - Or by using the auto-download functionality (this is not yet recommended)

If you prefer to use Cosmovisor to upgrade, some preparation work is needed before upgrade.

### Method I: Manual Upgrade

Make sure Neutron v2.0.0 is installed by either downloading a [compatible binary](https://github.com/neutron-org/neutron/releases/tag/v2.0.0), or building from source. Building from source requires **Golang 1.20.x**.

Run Neutron v1.0.4 till upgrade height, the node will panic:

```shell
ERR UPGRADE "v2.0.0" NEEDED at height: 5416000: upgrade to v2.0.0 and applying upgrade "v2.0.0" at height:h
```

Stop the node, and switch the binary to Neutron v2.0.0 and re-start by `neutrond start`.

It may take several minutes to a few hours until validators with a total sum voting power > 2/3 to complete their node upgrades. After that, the chain can continue to produce blocks.

### Method II: Upgrade using Cosmovisor

### Manually preparing the binary

##### Preparation

Install the latest version of Cosmovisor (`1.5.0`):

```shell
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

**Verify Cosmovisor Version**
```shell
cosmovisor version
cosmovisor version: v1.5.0
```

Create a cosmovisor folder:

create a Cosmovisor folder inside `$NEUTRON_HOME` and move Neutron v1.0.4 into `$NEUTRON_HOME/cosmovisor/genesis/bin`

```shell
mkdir -p $NEUTRON_HOME/cosmovisor/genesis/bin
cp $(which neutrond) $NEUTRON_HOME/cosmovisor/genesis/bin
````

build Neutron v2.0.0, and move neutrond v2.0.0 to `$NEUTRON_HOME/cosmovisor/upgrades/v2.0.0/bin`

```shell
mkdir -p  $NEUTRON_HOME/cosmovisor/upgrades/v2.0.0/bin
cp $(which neutrond) $NEUTRON_HOME/cosmovisor/upgrades/v2.0.0/bin
```

Then you should get the following structure:

```shell
.
├── current -> genesis or upgrades/<name>
├── genesis
│   └── bin
│       └── neutrond  #v1.0.4
└── upgrades
    └── v2.0.0
        └── bin
            └── neutrond  #v2.0.0
```

Export the environmental variables:

```shell
export DAEMON_NAME=neutrond
# please change to your own neutron home dir
# please note `DAEMON_HOME` has to be absolute path
export DAEMON_HOME=$NEUTRON_HOME
export DAEMON_RESTART_AFTER_UPGRADE=true
```

Start the node:

```shell
cosmovisor run  start --x-crisis-skip-assert-invariants --home $DAEMON_HOME
```

Skipping the invariant checks is strongly encouraged since it decreases the upgrade time significantly and since there are some other improvements coming to the crisis module in the next release of the Cosmos SDK.

#### Expected upgrade result

When the upgrade block height is reached, Neutron will panic and stop:

This may take a few minutes to a few hours.
After upgrade, the chain will continue to produce blocks when validators with a total sum voting power > 2/3 complete their node upgrades.

### Auto-Downloading the Neutron binary

**This method is not recommended!**

#### Preparation

Install the latest version of Cosmovisor (`1.5.0`):

```shell
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

Create a cosmovisor folder:

create a cosmovisor folder inside neutron home and move neutrond v1.0.4 into `$NEUTRON_HOME/cosmovisor/genesis/bin`

```shell
mkdir -p $NEUTRON_HOME/cosmovisor/genesis/bin
cp $(which neutrond) $NEUTRON_HOME/cosmovisor/genesis/bin
```

```shell
.
├── current -> genesis or upgrades/<name>
└── genesis
     └── bin
        └── neutrond  #v1.0.4
```

Export the environmental variables:

```shell
export DAEMON_NAME=neutrond
# please change to your own neutron home dir
export DAEMON_HOME=$NEUTRON_HOME
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=true
```

Start the node:

```shell
cosmovisor run start --x-crisis-skip-assert-invariants --home $DAEMON_HOME
```

Skipping the invariant checks can help decrease the upgrade time significantly.

#### Expected result

When the upgrade block height is reached, you can find the following information in the log:

```shell
ERR UPGRADE "v2.0.0" NEEDED at height: 5416000: upgrade to v2.0.0 and applying upgrade "v2.0.0" at height:h
```

Then the Cosmovisor will create `$NEUTRON_HOME/cosmovisor/upgrades/v2.0.0/bin` and download the Neuton v2.0.0 binary to this folder according to links in the `--info` field of the upgrade proposal.
This may take 7 minutes to a few hours, afterwards, the chain will continue to produce blocks once validators with a total sum voting power > 2/3 complete their nodes upgrades.

_Please Note:_

- In general, auto-download comes with the risk that the verification of correct download is done automatically. If users want to have the highest guarantee users should confirm the check-sum manually. We hope more node operators will use the auto-download for this release but please be aware this is a risk and users should take at your own discretion.
- Users should run their node on v1.0.4 if they use the cosmovisor v1.5.0 with auto-download enabled for upgrade process.

## Upgrade duration
Most likely it takes ~1hour.
According to our test runs, upgrade process takes the following steps:

1) "Preparation", no output in the logs ~10min
2) Applying migration handlers, you can see relevant entities (`migrating module ...`) in the logs ~5 min
3) Commiting state after migration ~40-50mins (stage starts right after the line `INF executed block height=5416000 module=consensus num_invalid_txs=0 num_valid_txs=0`)

## Rollback plan

During the network upgrade, core Neutron team will be keeping an ever vigilant eye and communicating with operators on the status of their upgrades. During this time, the core team will listen to operator needs to determine if the upgrade is experiencing unintended challenges. In the event of unexpected challenges, the core team, after conferring with operators and attaining social consensus, may choose to declare that the upgrade will be skipped.

Steps to skip this upgrade proposal are simply to resume the neutron-1 network with the (downgraded) v1.0.4 binary using the following command:

> neutrond start --unsafe-skip-upgrade 5416000

Note: There is no particular need to restore a state snapshot prior to the upgrade height, unless specifically directed by core Neutron team.

Important: A social consensus decision to skip the upgrade will be based solely on technical merits, thereby respecting and maintaining the decentralized governance process of the upgrade proposal's successful YES vote.

## Communications

Operators are encouraged to join the `#validator-chat` [channel](https://discord.com/channels/986573321023942708/1030043854637899816) in the `HUB VALIDATOR` section of the Neutron Discord. This channel is the primary communication tool for operators to ask questions, report upgrade status, report technical issues, and to build social consensus should the need arise. This channel is restricted to known operators and requires verification beforehand. Request to join the `#validator-chat` channel by opening a ticket or asking in the `general` channel.

## Risks

As a validator performing the upgrade procedure on your consensus nodes carries a heightened risk of double-signing and being slashed. The most important piece of this procedure is verifying your software version and genesis file hash before starting your validator and signing.

The riskiest thing a validator can do is discover that they made a mistake and repeat the upgrade procedure again during the network startup. If you discover a mistake in the process, the best thing to do is wait for the network to start before correcting it.

<!-- markdown-link-check-enable -->
