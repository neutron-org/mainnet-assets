# Upgrade from Neutron v2.0.0 to v2.0.1

> ## This is an important security update. IT IS CONSENSUS BREAKING, so please apply the fix only on height 5,971,800.

### Release Details
* https://github.com/neutron-org/neutron/releases/tag/v2.0.1
* Chain upgrade height : `5,971,800`. Exact upgrade time can be checked [here](https://www.mintscan.io/neutron/block/5971800).
* Go version has been frozen at `1.20`. If you are going to build Neutron binary from source, make sure you are using the right GO version!

# To upgrade neutron chain

## Step 1: Alter systemd service configuration

We need to disable automatic restart of the node service. To do so please alter your `neutrond.service` file configuration and set appropriate lines to following values.

```
Restart=no 

Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=false"
```

After that you will need to run `sudo systemctl daemon-reload` to apply changes in the service configuration.

There is no need to restart the node yet; these changes will get applied during the node restart in the next step.

## Step 2: Restart neutrond with a configured `halt-height`.

This upgrade requires `neutrond` to have knowledge of the planned halt height. Please be aware that there is an extra step at the end to revert to `neutrond`'s original configurations.

There are two mutually exclusive options for this stage:

### Option 1: Set the halt height by modifying `app.toml`

* Stop the neutrond process.

* Edit the application configuration file at `~/.neutrond/config/app.toml` so that `halt-height` reflects the upgrade plan:

```toml
# Note: Commitment of state will be attempted on the corresponding block.
halt-height = 5971800
```
* Start neutrond process

* Wait for the upgrade height and confirm that the node has halted

### Option 2: Restart the `neutrond` binary with command line flags

* Stop the neutrond process.

* Do not modify `app.toml`. Restart the `neutrond` process with the flag `--halt-height`:
```shell
neutrond --halt-height 5971800
```

* Wait for the upgrade height and confirm that the node has halted

After performing these steps, the upgrade will proceed as usual using Cosmovisor.

# Setup Cosmovisor
## Create the updated Neutron binary of v2.0.1

### Go to neutron directory if present else clone the repository

```shell
   git clone https://github.com/neutron-org/neutron.git
```

### Follow these steps if neutron repo already present

```shell
   cd $HOME/neutron
   git pull
   git fetch --tags
   git checkout v2.0.1
   make install
```

### Check the new neutron version, verify the latest commit hash
```shell
   $ neutrond version --long
   name: neutron
   server_name: neutrond
   version: 2.0.1
   commit: 877a9f54d36551e2b8da6b07635e5baf6fc70f67
   ...
```

### Or check checksum of the binary if you decided to download it

```shell
$ shasum -a 256 neutrond-linux-amd64
0c80684535e722487a5642ea367e76f294aef7c012cf640403a3383c2904cf85  neutrond-linux-amd64
```

## Make sure you are using the proper version of libwasm

You can check the version you are currently using by running the following command:
```
$ neutrond q wasm libwasmvm-version

1.5.1
```
The proper version is `1.5.1`.

**If the version on your machine is different you MUST change it immediately!**

#### Ways to change libwasmvm

- Use a statically built Neutrond binary from an official Neutron release: [https://github.com/neutron-org/neutron/releases/tag/v2.0.1](https://github.com/neutron-org/neutron/releases/tag/v2.0.1)
- If you built Neutron binary by yourself, `libwasmvm` should be loaded dynamically in your binary and somehow, the wrong `libwasmvm` library was present on your machine. You can change it by downloading the proper one and linking it to the Neutron binary manually:
1. download a proper version of `libwasmvm`:

```
$ wget https://github.com/CosmWasm/wasmvm/releases/download/v1.5.1/libwasmvm.x86_64.so
```

2. tell the linker where to find it:
```
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib/
```

3. check that libwasmvm version is correct:
```
$ neutrond q wasm libwasmvm-version
1.5.1
```

## Copy the new neutron (v2.0.1) binary to cosmovisor current directory
```shell
   cp $GOPATH/bin/neutrond ~/.neutrond/cosmovisor/current/bin
```

## Restore service file settings

If you are using a service file, restore the previous `Restart` settings in your service file: 
```
Restart=On-failure 
```
Reload the service control `sudo systemctl daemon-reload`.

# Revert `neutrond` configurations

Depending on which path you chose for Step 1, either:

* Reset `halt-height = 0` option in the `app.toml` or
* Remove it from start parameters of the neutrond binary and start node again
