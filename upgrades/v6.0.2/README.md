---
title: Neutron v6.0.2 Upgrade
order: 2
---

<!-- markdown-link-check-disable -->

# Neutron v6.0.2 Upgrade, Instructions

- Chain upgrade point: `June 10th 2025, 15:00 UTC (approximately)`, at height `27030500`;
- Go version: `v1.23`
- Release: https://github.com/neutron-org/neutron/releases/tag/v6.0.2

This document describes the steps for validators and full node operators, to upgrade successfully to the Neutron v6.0.2 release. For more details on the release, please see the [release notes](https://github.com/neutron-org/neutron/releases/tag/v6.0.2).

## Upgrade date

The upgrade will take place approximately on June 10th at `15:00 UTC` approximately at height `27030500`;

## Chain-id will remain the same

The chain-id of the network will remain the same, `neutron-1`. This is because an in-place migration of state will take place, i.e., this upgrade does not export any state.

### System requirement

- 64GB RAM is recommended to ensure a smooth upgrade.

If you have less than 64GB RAM, you might try creating a swapfile to swap an idle program onto the hard disk to free up memory. This can allow your machine to run the binary than it could run in RAM alone.

```shell
sudo fallocate -l 64G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

- Make sure you have enough disk space for upgrade, the state can grow twice durng upgrade.

### Backups

Prior to the upgrade, validators are encouraged to take a full data snapshot. Snapshotting depends heavily on infrastructure, but generally this can be done by backing up the `.neutrond` directory.
If you use Cosmovisor to upgrade, by default, Cosmovisor will backup your data upon upgrade. See below [upgrade using cosmovisor](#method-ii-upgrade-using-cosmovisor) section.

It is critically important for validator operators to back-up the `.neutrond/data/priv_validator_state.json` file after stopping the neutrond process. This file is updated every block as your validator participates in consensus rounds. It is a critical file needed to prevent double-signing, in case the upgrade fails and the previous chain needs to be restarted.

### Current runtime

The Neutron mainnet network, `neutron-1`, is currently running [Neutron v6.0.1](https://github.com/neutron-org/neutron/releases/tag/v6.0.1). We anticipate that operators who are running on v6.0.1, will be able to upgrade successfully. Validators are expected to ensure that their systems are up-to-date and capable of performing the upgrade. This includes running the correct binary, or if building from source, building with go `1.23`.

### Target runtime

The Neutron mainnet network, `neutron-1`, will run [Neutron v6.0.2](https://github.com/neutron-org/neutron/releases/tag/v6.0.2). Operators _**MUST**_ use this version post-upgrade to remain connected to the network.

## Upgrade steps

There are 2 major ways to upgrade a node:

- Manual upgrade
- Upgrade using [Cosmovisor](https://pkg.go.dev/cosmossdk.io/tools/cosmovisor)
    - Either by manually preparing the new binary
    - Or by using the auto-download functionality (this is not yet recommended)

If you prefer to use Cosmovisor to upgrade, some preparation work is needed before upgrade.

## Create the updated Neutron binary of v6.0.2

### Go to neutron directory if present else clone the repository

```shell
   git clone https://github.com/neutron-org/neutron.git
```

### Follow these steps if neutron repo already present

```shell
   cd $HOME/neutron
   git pull
   git fetch --tags
   git checkout v6.0.2
   make install
```

### Check the new neutron version, verify the latest commit hash
```shell
   $ neutrond version --long
    build_tags: netgo,ledger
    commit: 0de35a1bca6e040eeeb80a5dcfc076cb9cc5816e
    cosmos_sdk_version: v0.50.13-neutron-rpc
    go: go version go1.23.7 darwin/arm64
    name: neutron
    server_name: neutrond
    version: 6.0.2
   ...
```

### Or check checksum of the binary if you decided to [download it](https://github.com/neutron-org/neutron/releases/tag/v6.0.2)

```shell
$ shasum -a 256 neutrond-linux-amd64
0923698716d9977a964a4e1d1810c2f2f452f4fd00ec7d91e9d8d523c5d09d76  neutrond-linux-amd64
```


### Make sure you are using the proper version of libwasm

You can check the version you are currently using by running the following command:
```
$ neutrond q wasm libwasmvm-version

2.1.5
```
The proper version is `2.1.5`.

**If the version on your machine is different you MUST change it immediately!**

#### Ways to change libwasmvm

- Use a statically built Neutrond binary from an official Neutron release: [https://github.com/neutron-org/neutron/releases/tag/v6.0.2](https://github.com/neutron-org/neutron/releases/tag/v6.0.2)
- If you built Neutron binary by yourself, `libwasmvm` should be loaded dynamically in your binary and somehow, the wrong `libwasmvm` library was present on your machine. You can change it by downloading the proper one and linking it to the Neutron binary manually:
1. download a proper version of `libwasmvm`:

```
$ wget https://github.com/CosmWasm/wasmvm/releases/download/v2.1.5/libwasmvm.x86_64.so
```

2. tell the linker where to find it:
```
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib/
```

3. check that libwasmvm version is correct:
```
$ neutrond q wasm libwasmvm-version
2.1.5
```

### Method I: Manual Upgrade

Make sure Neutron v6.0.2 is installed by either downloading a [compatible binary](https://github.com/neutron-org/neutron/releases/tag/v6.0.2), or building from source. Building from source requires **Golang 1.23.x**.

Run Neutron v6.0.1 till upgrade height, the node will panic:

```shell
ERR UPGRADE "v6.0.2" NEEDED at height: 27030500: upgrade to v6.0.2 and applying upgrade "v6.0.2" at height: 27030500
```

Stop the node, and switch the binary to **Neutron v6.0.2** and re-start by `neutrond start`.

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

create a Cosmovisor folder inside `$NEUTRON_HOME` and move Neutron v6.0.1 into `$NEUTRON_HOME/cosmovisor/genesis/bin`

```shell
mkdir -p $NEUTRON_HOME/cosmovisor/genesis/bin
cp $(which neutrond) $NEUTRON_HOME/cosmovisor/genesis/bin
```

build **Neutron v6.0.2**, and move neutrond v6.0.2 to `$NEUTRON_HOME/cosmovisor/upgrades/v6.0.2/bin`

```shell
mkdir -p  $NEUTRON_HOME/cosmovisor/upgrades/v6.0.2/bin
cp $(which neutrond) $NEUTRON_HOME/cosmovisor/upgrades/v6.0.2/bin
```

Then you should get the following structure:

```shell
.
├── current -> genesis or upgrades/<name>
├── genesis
│   └── bin
│       └── neutrond  #v6.0.1
└── upgrades
    └── v6.0.2
        └── bin
            └── neutrond  #v6.0.2
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

When the upgrade block height is reached, Neutron will panic and stop.

After upgrade, the chain will continue to produce blocks when validators with a total sum voting power > 2/3 complete their node upgrades.

## Upgrade duration

Most likely it takes a couple of minutes.

## Rollback plan

During the network upgrade, core Neutron team will be keeping an ever vigilant eye and communicating with operators on the status of their upgrades. During this time, the core team will listen to operator needs to determine if the upgrade is experiencing unintended challenges. In the event of unexpected challenges, the core team, after conferring with operators and attaining social consensus, may choose to declare that the upgrade will be skipped.

Steps to skip this upgrade proposal are simply to resume the neutron-1 network with the (downgraded) v6.0.1 binary using the following command:

> neutrond start --unsafe-skip-upgrade 27030500

Note: There is no particular need to restore a state snapshot prior to the upgrade height, unless specifically directed by core Neutron team.

Important: A social consensus decision to skip the upgrade will be based solely on technical merits, thereby respecting and maintaining the decentralized governance process of the upgrade proposal's successful YES vote.

## Risks

As a validator performing the upgrade procedure on your consensus nodes carries a heightened risk of double-signing and being slashed. The most important piece of this procedure is verifying your software version and genesis file hash before starting your validator and signing.

The riskiest thing a validator can do is discover that they made a mistake and repeat the upgrade procedure again during the network startup. If you discover a mistake in the process, the best thing to do is wait for the network to start before correcting it.

## FAQ

1. Q: My node restarted in the middle of upgrade process (OOM killed, hardware issues, etc), is it safe to just proceed with the upgrade

   A: No. Most likely the upgrade will be completed successfully. But you get AppHash error after the network gets up. It's a lot safer to restart full process from scratch(recover the node from a backup).

   To perform an upgrade you need to keep your `./data/priv_validator_state.json` file when you are applying a snapshot from the backup.
   This will help you avoid the risk of slashing due to double signing.

<!-- markdown-link-check-enable -->
