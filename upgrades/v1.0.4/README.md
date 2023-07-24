> ## This is just a QoL upgrade. It is not consensus breaking.


# Upgrade Neutron from v1.0.3 to v1.0.4

### Release Details
* https://github.com/neutron-org/neutron/releases/tag/v1.0.4


# Create the updated Neutron binary of v1.0.4

## Go to neutron directory if present else clone the repository

```shell
   git clone https://github.com/neutron-org/neutron.git
```

## Follow these steps if neutron repo already present

```shell
   cd $HOME/neutron
   git pull
   git fetch --tags
   git checkout v1.0.4
   make install
```

## Check the new neutron version, verify the latest commit hash
```shell
   $ neutrond version --long
    name: neutron
    server_name: neutrond
    version: 1.0.4
    commit: 780486095bf657f7b94b4474cb39fd137cf93c98
    ...
```

## Copy the new neutron (v1.0.4) binary to cosmovisor current directory

```shell
   cp $GOPATH/bin/neutrond ~/.neutrond/cosmovisor/current/bin
```