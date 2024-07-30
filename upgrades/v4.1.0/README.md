# Upgrade from Neutron v4.0.1 to v4.1.0

> ## This is an important security update. IT IS CONSENSUS BREAKING UPGRADE! Please apply the fix only at height 11298600.

### Release Details
* https://github.com/neutron-org/neutron/releases/tag/v4.1.0
* Chain upgrade height: `TODO_HEIGHT`. Exact upgrade time can be checked [here](https://www.mintscan.io/neutron/block/TODO_HEIGHT).
* Go version has been frozen at `1.22`. If you are going to build Neutron binary from source, make sure you are using the right GO version!

# To upgrade neutron chain

### Backups

Prior to the upgrade, validators are encouraged to take a full data snapshot. Snapshotting depends heavily on infrastructure, but generally this can be done by backing up the `.neutrond` directory.
If you use Cosmovisor to upgrade, by default, Cosmovisor will backup your data upon upgrade.

It is critically important for validator operators to back-up the `.neutrond/data/priv_validator_state.json` file after stopping the neutrond process. This file is updated every block as your validator participates in consensus rounds. It is a critical file needed to prevent double-signing, in case the upgrade fails and the previous chain needs to be restarted.

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
halt-height = TODO_HEIGHT
```
* Start neutrond process

* Wait for the upgrade height and confirm that the node has halted

### Option 2: Restart the `neutrond` binary with command line flags

* Stop the neutrond process.

* Do not modify `app.toml`. Restart the `neutrond` process with the flag `--halt-height`:
```shell
neutrond --halt-height TODO_HEIGHT
```

* Wait for the upgrade height and confirm that the node has halted

After performing these steps, the upgrade will proceed as usual using Cosmovisor.

# Setup Cosmovisor
## Create the updated Neutron binary of v4.1.0

### Go to neutron directory if present else clone the repository

```shell
   git clone https://github.com/neutron-org/neutron.git
```

### Follow these steps if neutron repo already present

```shell
   cd $HOME/neutron
   git pull
   git fetch --tags
   git checkout v4.1.0
   make install
```

### Check the new neutron version, verify the latest commit hash
```shell
   $ neutrond version --long
   name: neutron
   server_name: neutrond
   version: 4.1.0
   commit: TODO_COMMIT
   ...
```

### Or check checksum of the binary if you decided to download it

```shell
$ shasum -a 256 neutrond-linux-amd64
TODO_HASH  neutrond-linux-amd64
```

## Copy the new neutron (v4.1.0) binary to cosmovisor current directory
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
