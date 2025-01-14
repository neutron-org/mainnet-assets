---
title: Neutron v5.0.6 Upgrade
order: 2
---

<!-- markdown-link-check-disable -->

# Neutron v5.0.6 Upgrade, Instructions

- This is an **emergency upgrade** necessary to **un-halt the chain** on January 14, 2025
- Go version: `v1.22`
- Release: https://github.com/neutron-org/neutron/releases/tag/v5.0.6

## Upgrade steps

_Note: Prior to the upgrade, validators are encouraged to take a full data snapshot. Snapshotting depends heavily on infrastructure, but generally this can be done by backing up the `.neutrond` directory._

1. Stop the node if you didn't stop it yet, and back-up the `.neutrond/data/priv_validator_state.json` file just in case.
2. Roll the node back 1 block: `neutrond rollback --home <YOUR_NEUTRON_HOME_FOLDER_PATH>`. **This can take up some time, depending on the configuration of your node (how aggressively you prune).**
3. Run the node with the new binary (see the instructions below on how to obtain the binary).

## Building Neutron v5.0.6

### Go to neutron directory if present else clone the repository

```shell
   git clone https://github.com/neutron-org/neutron.git
```

### Follow these steps if neutron repo already present

```shell
   cd $HOME/neutron
   git pull
   git fetch --tags
   git checkout v5.0.6
   make install
```

### Check the new neutron version, verify the latest commit hash
```shell
   $ neutrond version --long
   commit: cdd4fd6fcee5baf2d6ae79dac0e6b12aa847aff2TODO
   cosmos_sdk_version: v0.50.11-neutron
   go: go version go1.22.9 linux/amd64
   name: neutron
   version: 5.0.6
   ...
```

### Or check checksum of the binary if you decided to [download it](https://github.com/neutron-org/neutron/releases/tag/v5.0.6)

```shell
$ shasum -a 256 neutrond-linux-amd64
1e2aa2ab56e60bc3bdda7bdd0b505c02e8a304c5874ce2d907d01d26ee825b9bTODO  neutrond-linux-amd64
```


### Make sure you are using the proper version of libwasm

You can check the version you are currently using by running the following command:
```
$ neutrond q wasm libwasmvm-version

2.1.4
```
The proper version is `2.1.4`.

**If the version on your machine is different you MUST change it immediately!**

#### Ways to change libwasmvm

- Use a statically built Neutrond binary from an official Neutron release: [https://github.com/neutron-org/neutron/releases/tag/v5.0.6](https://github.com/neutron-org/neutron/releases/tag/v5.0.6)
- If you built Neutron binary by yourself, `libwasmvm` should be loaded dynamically in your binary and somehow, the wrong `libwasmvm` library was present on your machine. You can change it by downloading the proper one and linking it to the Neutron binary manually:
1. download a proper version of `libwasmvm`:

```
$ wget https://github.com/CosmWasm/wasmvm/releases/download/v2.1.4/libwasmvm.x86_64.so
```

2. tell the linker where to find it:
```
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib/
```

3. check that libwasmvm version is correct:
```
$ neutrond q wasm libwasmvm-version
2.1.4
```

