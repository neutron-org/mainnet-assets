# Upgrade from Neutron v2.0.0 to v2.0.1

> ## This is an important security update. It **is consensus breaking**, so please apply it to your validator ASAP.

### Release Details
* https://github.com/neutron-org/neutron/releases/tag/v2.0.1
* Chain upgrade time: 10th of January 2024 3 PM UTC.
* Go version has been frozen at `1.20`. If you are going to build Neutron binary from source, make sure you are using the right GO version!

# Create the updated Neutron binary of v2.0.1

## Go to neutron directory if present else clone the repository

```shell
   git clone https://github.com/neutron-org/neutron.git
```

## Follow these steps if neutron repo already present

```shell
   cd $HOME/neutron
   git pull
   git fetch --tags
   git checkout v2.0.1
   make install
```

## Check the new neutron version, verify the latest commit hash
```shell
   $ neutrond version --long
   name: neutron
   server_name: neutrond
   version: 2.0.1
   commit: <TODO_COMMIT>
   ...
```

## Or check checksum of the binary if you decided to download it

```shell
$ shasum -a 256 neutrond-linux-amd64
<TODO_HASH>  neutrond-linux-amd64
```

## Copy the new neutron (v2.0.1) binary to cosmovisor current directory

```shell
   cp $GOPATH/bin/neutrond ~/.neutrond/cosmovisor/current/bin
```

## Make sure you are using the proper version of libwasm

You can check the version you are currently using by running the following command:
```
$ neutrond q wasm libwasmvm-version

1.5.1
```
The proper version is `1.5.1`.

**If the version on your machine is different you MUST change it immediately!**

### Ways to change libwasmvm

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

4. Stop the node;
5. Restart the node.