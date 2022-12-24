<p align="center">
<img width="836" alt="image" src="https://user-images.githubusercontent.com/3147812/209434273-ff5eb5e6-0b32-4bb0-854b-dda2693e0175.png">
</p>

# 🔥🛠️ Foundry Storage Upgrade Seatbelt

- Protect your Smart Contract Proxy from storage collisions upon upgrading, by running this action in a CI on each of your Pull Requests!
- Feel safe when extending your storage layout by trusting this action to check that extended layout is zero-ed out on-chain!

## Live Example

Check out [PR #21](/pulls/21) for a live example:

- Action is ran on [contracts/Example.sol:Example](./contracts/Example.sol)
- Warnings & errors appear on the [Pull Request changes](https://github.com/Rubilmax/foundry-storage-check/pull/21/files)

## Getting started

### Automatically generate & compare to the previous storage layout on every PR

Add a workflow (`.github/workflows/foundry-storage-check.yml`):

```yaml
name: Check storage layout

on:
  push:
    branches:
      - main
  pull_request:
    # Optionally configure to run only for changes in specific files. For example:
    # paths:
    # - src/**
    # - test/**
    # - foundry.toml
    # - remappings.txt
    # - .github/workflows/foundry-storage-check.yml

jobs:
  check_storage_layout:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive

      - name: Install Foundry
        uses: onbjerg/foundry-toolchain@v1
        with:
          version: nightly

      - name: Check storage layout
        uses: Rubilmax/foundry-storage-check@v3
        with:
          contract: src/Contract.sol:Contract
          # settings below are optional, but allows to check whether the added storage slots are empty on the deployed contract
          rpcUrl: wss://eth-mainnet.g.alchemy.com/v2/<YOUR_ALCHEMY_KEY> # the RPC url to use to query the deployed contract's storage slots
          address: 0x0000000000000000000000000000000000000000 # the address at which the contract check is deployed
          failOnRemoval: true # fail the CI when removing storage slots (default: false)
```

> :information_source: **An error will appear at first run!**<br/>
> 🔴 <em>**Error:** No workflow run found with an artifact named "..."</em><br/>
> As the action is expecting a comparative file stored on the base branch and cannot find it (because the action never ran on the target branch and thus has never uploaded any storage layout)

---

## How it works

Everytime somebody opens a Pull Request, the action runs [Foundry](https://github.com/foundry-rs/foundry) `forge` to generate the storage layout of the Smart Contract you want to check.

Once generated, the action will fetch the comparative storage layout stored as an artifact from previous runs and compare them, to perform a series of checks at each storage byte, and raise a notice accordingly:

- Variable changed: `error`
- Type definition changed: `error`
- Type definition removed: `warning`
- Different variable naming: `warning`
- Variable removed (optional): `error`

The action automatically checks for:
- All canonic storage bytes
- Array value (32 bytes) at index `#0`
- Mapping value (32 bytes) at key `0x00`
- Zero-ed bytes for added storage variables

---

## Options

### `contract` _{string}_

The path and name of the contract of which to inspect storage layout (e.g. src/Contract.sol:Contract).

_Required_

### `address` _{string}_

The address at which the contract is deployed on the EVM-compatible chain queried via `rpcUrl`.

### `rpcUrl` _{string}_

The HTTP/WS url used to query the EVM-compatible chain for storage slots to check for clashing.

### `failOnRemoval` _{string}_

Whether to fail the CI when removing a storage slot (to only allow added or renamed storage slots).

_Defaults to: `false`_

### `base` _{string}_

The gas diff reference branch name, used to fetch the previous gas report to compare the freshly generated gas report to.

_Defaults to: `${{ github.base_ref || github.ref_name }}`_

### `head` _{string}_

The gas diff target branch name, used to upload the freshly generated gas report.

_Defaults to: `${{ github.head_ref || github.ref_name }}`_

### `token` _{string}_

The github token allowing the action to upload and download gas reports generated by foundry. You should not need to customize this, as the action already has access to the default Github Action token.

_Defaults to: `${{ github.token }}`_

This repository is maintained independently from [Foundry](https://github.com/foundry-rs/foundry) and may not work as expected with all versions of `forge`.
