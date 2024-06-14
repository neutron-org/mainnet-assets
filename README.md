# `neutron-1`

The `neutron-1` chain will be launched as a consumer chain with Cosmos Hub network as provider chain.

**⚠️ If you have not used key assignment for your consumer node yet, please wait until Neutron is live and receiving valset updates from the Hub before you do so.**

## Upgrades history

| Version    | Value                             | Height                                                                                                                                     |
|------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| **v1.0.1** | Genesis version                   | From start                                                                                                                                 |
| **v1.0.2** | Security upgrade ([more info][1]) | Anytime, not breaks the consensus                                                                                                          |
| **v1.0.3** | Security upgrade ([more info][2]) | Coordinated **consensus breaking** upgrade without proposal at height 1236300                                                              |
| **v1.0.4** | Security upgrade ([more info][3]) | Anytime before height #1909000, **consensus breaking**                                                                                     |
| **v2.0.0** | Upgrade ([more info][4])          | Coordinated **consensus breaking** upgrade with a [proposal 25](https://governance.neutron.org/proposals/25) at height 5416000             |
| **v2.0.1** | Upgrade ([more info][5])          | Coordinated **consensus breaking security** upgrade without a proposal on height 5971800 approximately at 3 PM UTC on 10th of January 2024 |
| **v2.0.3** | Upgrade ([more info][6])          | Coordinated **consensus breaking security** upgrade without a proposal on height 7818500 approximately at 2:30 PM UTC on 5th of March 2024 |
| **v3.0.1** | Upgrade ([more info][7])          | Coordinated **consensus breaking** upgrade with a [proposal 35](https://governance.neutron.org/proposals/35) at height 9034900             |
| **v3.0.5** | Upgrade ([more info][8])          | Coordinated **consensus breaking** upgrade with a [proposal 37](https://governance.neutron.org/proposals/37) at height 10525000            |
| **v3.0.6** | Upgrade ([more info][8])          | Coordinated **consensus breaking security** upgrade without a proposal on height <HEIGHT> approximately at 13:30 UTC on 14th of June 2024  |            |

## Parameters

Below are the `neutron-1` chain parameters:

| Name                   | Value                         |
| ---------------------- | ----------------------------- |
| **chain-id**           | `neutron-1`                   |
| **denom**              | `untrn`                       |
| **minimum-gas-prices** | `0.01untrn`                   |
| **timeout_commit**     | `1s`                          |
| **genesis_time**       | `2023-05-10T15:00:00.000000Z` |

**The `minimum-gas-prices` parameter must be set to `0.01untrn`.** At chain launch (and until the end of the Token Generation Event) the only address that will have `untrn`s will be the initial Hermes relayer. This relayer will be configured to **only** process `Transfer` messages between Neutron and Cosmos Hub. As soon as the chain starts, some `uatoms` will be transferred from Cosmos Hub to Neutron and the bridged ATOM denom will be communicated to the validators. After that, the validators will be able to set the `minimum-gas-prices` in bridged ATOMs.

## Binary

The release binary information is provided below:

| Item                  | Description                                                            |
| --------------------- | ---------------------------------------------------------------------- |
| **GitHub repo**       | [neutron-org/neutron](https://github.com/neutron-org/neutron.git)      |
| **Release**           | [`v1.0.1`](https://github.com/neutron-org/neutron/releases/tag/v1.0.1) |
| **Reference binary**  | [neutrond-linux-amd64](./neutrond-linux-amd64)                         |
| **Checksum (sha256)** | b628d3eb1e0e12617b2c905f07dc39bd91d5dd3bd284a2a51d47c04cc3aa2e6d       |

> The `neutrond-linux-amd64` binary is only provided to verify the SHA256. It was built with Interchain Security release [v1.2.0-multiden](https://github.com/cosmos/interchain-security/tree/v1.2.0-multiden). You can generate the binary following the build instructions in the [neutron-org/neutron](https://github.com/neutron-org/neutron.git) repo.

The Cosmos Hub was recently upgraded to [v9.1.0](https://github.com/cosmos/gaia/releases/tag/v9.1.0), which bumps ICS to [v1.1.0-multiden](https://github.com/cosmos/interchain-security/tree/v1.1.0-multiden). This version introduces two new parameters to prevent Replicated Security’s logic for handling rewards to be abused as a DOS vector.

This required a release of a new version of Neutron, `v1.0.1`, which must be used for the mainnet launch. The only difference between `v1.0.0-rc1`, which was used in the proposal, and `v1.0.1` is the upgrade of Interchain Security to the latest release, [`v1.2.0-multiden`](https://github.com/cosmos/interchain-security/tree/release/v1.2.0-multiden), which is compatible with the new parameters.
You can check the difference [here](https://github.com/neutron-org/neutron/compare/v1.0.0-rc1..v1.0.1).

**⚠️ All validators are required to use Neutron `v1.0.1` during the mainnet launch. The instructions on how to build `v1.0.1` can be found in the [How to Join](#how-to-join) section.**

## Genesis

The final `genesis.json` information is provided below:

| Item                  | Description                                                                                              |
| --------------------- | -------------------------------------------------------------------------------------------------------- |
| **Genesis**           | [genesis.json](https://raw.githubusercontent.com/neutron-org/mainnet-assets/main/neutron-1-genesis.json) |
| **Checksum (sha256)** | 9496492c81b31befb59a4336d5ae4444c24b863721d21be143d7d4fdf8072c84                                         |

**⚠️ All validators are required use the [genesis.json](https://raw.githubusercontent.com/neutron-org/mainnet-assets/main/neutron-1-genesis.json) file provided in this instruction.**

If you want to get a better idea of what was changed in the genesis since Proposal 792, please read the sections below.

---

#### `ccvconsumer` section updates

Due to the upgrade of the Neutron's ICS dependency to [v1.2.0-multiden](https://github.com/cosmos/interchain-security/tree/v1.2.0-multiden), and due to using a version of ICS with soft opt-out, there is 3 parameters that were added to the `ccvconsumer` section of the genesis:

```json
{
  "soft_opt_out_threshold": "0.05",
  "reward_denoms": ["untrn"],
  "provider_reward_denoms": ["uatom"]
}
```

1. `rewards_denoms`: rewards denominations from the Consumer chain (e.g. uNTRN or IBC denoms used on the consumer chain);
2. `provider_rewards_denoms`: rewards denomination from the Provider chain (e.g. uATOM);
3. `soft_opt_out_threshold`: this parameter was set to `0.05` to allow the smaller validators to opt out of validating Neutron without being slashed.

#### `slashing` parameters

In the pre-genesis, we used incorrect `slashing` parameters. Below you can see the `slashing` parameters diff:

```
+ "signed_blocks_window": "140000",
+ "min_signed_per_window": "0.050000000000000000",
+ "slash_fraction_downtime": "0.000100000000000000"
- "signed_blocks_window": "100",
- "min_signed_per_window": "0.500000000000000000",
- "slash_fraction_downtime": "0.010000000000000000"
```

1. `signed_blocks_window` was increased to approximately 4 days given a `2.5s` block production time that we are expecting. This is done to give the validators more time to recover a failed node;
2. `min_signed_per_window` was adjusted to match the value used by he Cosmos Hub;
3. `slash_fraction_downtime` was adjusted to match the value used by he Cosmos Hub.

#### `wasm` artifacts

Neutron genesis instantiates dozens of smart contracts that are used by the Neutron DAO and the Neutron Token Generation Event. Since the time when Proposal 792 was published, we fixed some bugs and introduced some improvements to our smart contracts.

We have a [script](https://github.com/neutron-org/tools/blob/mainnet/genesis/genesis.sh) that you can copy to an empty directory and run to get the final genesis. We provide this script **for information purposes only**, and we do not guarantee that it will work on your machine (although if you have a Mac and the Docker daemon is running, it should produce the final `genesis.json` within approximately 30 minutes because it will build all the `wasm` binaries).

**⚠️ Please use the [genesis.json](https://raw.githubusercontent.com/neutron-org/mainnet-assets/main/neutron-1-genesis.json) file provided above during the coordinated launch.**

## Endpoints

Seed nodes:

1. `24f609fb5946ca3a979f40b7f54132c00104433e@p2p-erheim.neutron-1.neutron.org:26656`
2. `20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:19156`
3. `b1c6fa570a184c56d0d736d260b8065d887e717c@p2p-kralum.neutron-1.neutron.org:26656`
4. `f80e07ebb0ae3f11d938e8e224705b1039f000b5@neutron.dhk.org:26656`
5. `ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:19156`

Persistent nodes:

1. `e5d2743d9a3de514e4f7b9461bf3f0c1500c58d9@neutron.peer.stakewith.us:39956`
2. `982f968cd6ac567fdddf2170f7da9725ba21693d@51.210.209.49:15600`
3. `8e2af33b3fc9fee81dda7d351a65d93e8ac97409@65.108.71.163:2480`
4. `b119a600d082963d27a9ba9cd762c7c83f00e7d1@81.196.190.108:10083`
5. `9cede4e9f58fc3dfc80ad643c0e1f1cbab26b8ba@138.201.135.251:26656`
6. `2ee64f9f128e8a9f15f47a8f8d0e9cde7351fd17@46.101.193.151:26656`
7. `ce526dc00568900d54dedf2fd2680ddfc7a58c59@138.201.124.215:36656`
8. `5865c2bae7c0403c0c6d90c01556b1b2bb437ef8@51.89.195.173:26656`

Also you can use this [addrbook.json](./addrbook.json) to bootstrap your nodes.

The following state sync node serve snapshots every 2000 blocks:

1. `http://rpc-kralum.neutron-1.neutron.org:26657`

## How to Join

### Hardware Requirements

- 4 Cores
- 32 GB RAM
- 2x512 GB SSD

### Software Versions

| Name    | Version |
| ------- | ------- |
| Neutron | v2.0.3  |
| Go      | =1.20   |

### Node manual installation

Build and install the Neutron binary.

```bash
$ git clone -b v2.0.3 https://github.com/neutron-org/neutron.git
$ cd neutron
$ make install
```

After installation, please check installed version by running:

```bash
$ neutrond version --long
name: neutron
server_name: neutrond
version: 2.0.3
commit: <COMMIT>
```

You can also download binary directly from our [official release](https://github.com/neutron-org/neutron/releases/tag/v1.0.1).

[1]: ./upgrades/v1.0.2/README.md
[2]: ./upgrades/v1.0.3/README.md
[3]: ./upgrades/v1.0.4/README.md
[4]: ./upgrades/v2.0.0/README.md
[5]: ./upgrades/v2.0.1/README.md
[6]: ./upgrades/v2.0.3/README.md
[7]: ./upgrades/v3.0.1/README.md
[8]: ./upgrades/v3.0.5/README.md
[9]: ./upgrades/v3.0.6/README.md