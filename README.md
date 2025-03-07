# zkBitcoin

Use **zero-knowledge applications (zkapps)** on Bitcoin! (**Currently only on testnet.**)

![image](https://github.com/sigma0-xyz/zkbitcoin/assets/1316043/5fe31a43-1775-4ebb-b6ac-510651e8b08a)

**How does it work**? Write your zkapp in [circom](https://github.com/iden3/circom) and deploy it on Bitcoin by sending a transaction to our multi-party wallet run by a committee of nodes.

To use a zkapp, provide a correct proof of execution using [snarkjs](https://github.com/iden3/snarkjs) to our multi-party wallet which will trigger a threshold signature, eventually allowing funds to move out of the zkapp.

[Check the whitepaper here](./whitepaper.pdf). You can also [watch a demo here](https://www.youtube.com/watch?v=2a0UYT5nbEA).

## Installation

Jump straight to [usage](#usage) if you want to see some examples, but make sure to read this section otherwise things won't work!

### Circom/snarkjs

We build on top of the well-known [circom](https://github.com/iden3/circom)/[snarkjs](https://github.com/iden3/snarkjs) stack.

To install `circom`, please follow [their guide](https://docs.circom.io/getting-started/installation/).

To install `snarkjs`, just run:

```
npm install -g snarkjs@latest
```

### Bitcoin wallet

On top that, you'll need your own Bitcoin node/wallet. This application will perform queries to your node/wallet in order to fund your zkapp transactions.

All the following commands expects the following environment variables to be set so that it can communicate with your node/wallet:

```shell
export RPC_WALLET="walletname"
export RPC_ADDRESS="http://127.0.01:18331"
export RPC_AUTH="username:password"
```

### zkbtc: the zkBitcoin CLI

To install `zkbtc`, run the following command:

```shell
cargo install --git https://github.com/sigma0-xyz/zkbitcoin.git
```

## Usage

There are two types of zkapps: [stateless](#stateless-zkapps) and [stateful](#stateful-zkapps).

### Stateless zkapps

A stateless zkapp is single-use, and the bitcoin it locks can be redeemed by anyone who can provide a proof of correct execution. An example of a stateless zkapp is in [`examples/circuit/stateless.circom`](examples/circuit/stateless.circom) (which releases funds to anyone who can find the preimage of a hash function).
A stateless zkapp must always contains one public input that authenticates the transaction that spends it:

```circom
// circom code
template Main() {
    signal input truncated_txid;
    // TRUNCATED...
}
component main{public [truncated_txid]} = Main();
```

The zkapp doesn't have to do anything with the `truncated_txid` field (although it can if it wants to).

You can deploy a stateless zkapp with the following command:

```shell
$ zkbtc deploy-zkapp --circom-circuit-path examples/circuit/stateless.circom --satoshi-amount 1000
```

This will lock 1,000 satoshis in the zkapp and return the transaction ID of the transaction that deployed the zkapp. A stateless zkapp can be referenced by that transaction ID.

Bob can then unlock the funds from the stateless zkapp with the following command:

```shell
$ zkbtc use-zkapp --txid "e793bdd8dfdd9912d971790a5f385ad3f1215dce97e25dbefe5449faba632836" --circom-circuit-path examples/circuit/stateless.circom --proof-inputs '{"preimage":["1"]}' --recipient-address "tb1q6nkpv2j9lxrm6h3w4skrny3thswgdcca8cx9k6"
```

### Stateful zkapps

A stateful zkapp is a zkapp that has a state, and which state can be updated without consuming the zkapp.

An example of a stateful zkapp is in [`examples/circuit/stateful.circom`](examples/circuit/stateful.circom). A stateful zkapp must always contains a number of additional public inputs, allowing an execution to authenticate the zkapp state transition, as well as the amounts being withdrawn and deposited:

```circom
// circom code
template Main() {
    signal output new_state;
    signal input prev_state;
    signal input truncated_txid; // this should not affect output
    signal input amount_out;
    signal input amount_in;
    // TRUNCATED...
}
component main{public [prev_state, truncated_txid, amount_out, amount_in]} = Main();
```

You can deploy a stateful zkapp with the following command:

```shell
$ zkbtc deploy-zkapp --circom-circuit-path examples/circuit/stateful.circom --initial-state "1" --satoshi-amount 1000
```

You can use a stateful zkapps with the following command:

```shell
$ zkbtc use-zkapp --circom-circuit-path examples/circuit/stateful.circom --proof-inputs '{"amount_in":["1000"], "amount_out":["1000"]}' --recipient-address "tb1q6vjawwska63qxf77rrm5uwqev0ma8as8d0mkrt" --txid "76763d6130ee460ede2739e0f38ea4d61cc940b00af5eab83e5afb0fcc837b91"
```

specifying the following inputs:

- `amount_out`: amount being withdrawn
- `amount_in`: amount being deposited

Other inputs will be automatically filled in (for example, it will use the zkapp's state as `prev_state` input).

## Get information about a zkapp

You can retrieve information about a specific zkapp by running the following command with the zkapp's transaction id:

```shell
$ zkbtc get-zkapp 7f08eeb5a4cba9bed161ba54bb28db4fc6ce51273e48d40969d5d89fdab61770
```

## List all deployed zkapps

You can list all currently-deployed zkapps in the following way:

```shell
$ zkbtc list-zkapps
```

## Tell me more

You can read more about zkBitcoin in [our whitepaper](./whitepaper.pdf), [our documentation](docs/), and about advanced usage in [our developer documentation](DEVELOPER.md).
