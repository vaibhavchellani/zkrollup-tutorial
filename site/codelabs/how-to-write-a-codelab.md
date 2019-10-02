summary: Scaling your favourite dapp using zkRollup
id: scaling-using-zkRollup
categories: Sample
tags: medium
status: Published
authors: Vaibhav Chellani
Feedback Link: https://twitter.com/vaibhavchellani

# ZkRollup Tutorial

<!-- ------------------------ -->

## Overview

Duration: 1

### What you'll build

In this workshop we will build circuits for zkRollup using

### What you'll learn

You will learn how to build with zkRollup

### What you'll need

You will need circom and snarkjs

## ZkRollup in a nutshell

ZkSnarks have been used for privacy applications and for having private transactions on chains like ZCash for a few years now. Rollup utilises zkSnarks to scale ethereum by taking advantage of the succinctness and privacy provided by snarks.

We achive scalability by not having to send user signatures on-chain and instead send one proof which signifies 1000's of signature verifications and other transaction validation checks have been done correctly off-chain.

We send minified form of transactions on-chain as input to circuit to have data-available.

### How zkRollup works?

There are two actors involved in a rollup chain: Coordinator and User.

- We maintain account merkel tree on chain which can be updated by sending snark proof.
- Users sends transaction to coordinator with signatures via RPC/API's.
- Coordinator collects all the transactions and creates a batch which is processed by the snark circuit, if the circuit finds all transactions in the batch to be valid it emits a proof.
- This validity proof will be submitted and verified on chain which signifies that state was updated properly off-chain.
- This updates the onchain account merkel tree.

> We will dive into specifics of the construction in the next parts.

### Why zkRollup is Amazing ?

- No exit games
- No liveliness assumptions
- No data availability issues
- Sub cent fees for transactions
- Can create rollup chain for any state transitions not just transfers( fairWin anyone?! )

<!-- ------------------------ -->

## Using Circom and Snarkjs

Duration: 2

Circom and snarkjs are these compiler and helper packages by IDEN3 to compile and use circuits. They are JS based and hence very easy to use for small circuits. We will go through some commands which we will be using in this tutorial.

#### Compiling circom circuits

Compiles your circuit and generates compiled circuit in json format for snarkjs to use.

```bash
circom <your_circuit_name>.circom -o circuit.json`
```

#### Generate inputs for your circuit

Generates public private inputs for your circuit

```bash
node generate_circuit_input.js
```

> Note: This exists only for the course of our workshop

#### Calculate witness

Generates witness for your circuit using the compiled circuit and input file created in the above step

```bash
snarkjs calculatewitness -c circuit.json -i input.json
```

#### Performing trusted setup

Remember the infamous "Trusted Setup", to perform trusted setup on your device for a circuit use the below command. It creates a Proving and Verification key for your circuit.

```bash
snarkjs setup -c circuit.json --protocol groth
```

#### Generate the proof

To generate proof for your circuit use the below command. It uses witness and proving key as inputs and generates a proof for you which can be passed around to certify correcct computation of circuit

```bash
snarkjs proof -w witness.json --pk proving_key.json
```

## Part 1 - Simple arithmetic constraints

This is a contrived example to familiarise ourselves with the syntax of `circom` and how it works with `snarkjs`.

Let's write a circuit to check:

- that the sum of two private inputs `a + b` is equal to a public input `c`;
- that the product `b * c` is equal to private input `d`;

Let's get started 🎊🎉

```bash
- $ cd 1_simple_arithmetic
- $ touch circuit.circom

```

```C##

// create template
template SimpleChecks() {
    // declare a variables which will store private inputs
    signal private input a;
    signal private input b;
    signal private input d;


    // declare public input variables
    signal input c;

    // declate output variables
    signal output out;

    // force a + b = c
    a + b === c;

    // force b * c = d
    // TODO : Fill in the constraint

    // assign output to output variable
    out <== c + d;
}

// export the template
component main = SimpleChecks();
```

Now that we are done with this small circuit let's see how we can generate the proof for this circuit

1. Compile it: `circom circuit.circom -o circuit.json`
2. Generate your input for the circuit: `node generate_circuit_input.js`
3. Calculate the witness for the circuit: `snarkjs calculatewitness -c circuit.json -i input.json`
4. Generate the proof: `snarkjs setup -c circuit.json --protocol groth`
5. Verify the proof: `snarkjs verify`

## Part-1 Challenge

It's time for a challenge guys!

**Goal** : Modify the circuit and input to take in length-4 arrays of `a`, `b`, `c`, and `d`, and perform the checks in a `for` loop. Output the sums of `c` and `d` arrays.

To get you started, here's some help

```C##
template SimpleChecks(k) {
    signal private input a[k];
    signal private input b[k];
    signal input c[k];
    signal private input d[k];
    signal output out;

    var sum = 0;
    for (var i = 0; i < k; i++){
        // force a + b = c
        a[i] + b[i] === c[i];

        // force b * c = d
        // fill this in

        // add up c and d arrays
        // use the variable 'sum' defined outside the for loop
    }
    // output sum of c and d arrays
    out <== sum;
}

component main = SimpleChecks(4);
```

## Part-2 Verifying an EdDSA signature

Signing data and verifying signatures are an essential part of any rollup chain you want to design. We use the EdDSA signature because it's much more efficient.

This example works with useful libraries in `circomlib`. Note: we are using v0.0.6 of `circomlib`.

Let's get started 🎊🎉

```bash
- $ cd 2_verify_eddsa
- $ touch circuit.circom

```

```C##
include "../circomlib/circuits/eddsamimc.circom";

template VerifyEdDSAMiMC() {
    signal input from_x;
    signal input from_y;
    signal input R8x;
    signal input R8y;
    signal input S;
    signal input M;

    component verifier = EdDSAMiMCVerifier();
    verifier.enabled <== 1;
    verifier.Ax <== from_x;
    verifier.Ay <== from_y;
    verifier.R8x <== R8x
    verifier.R8y <== R8y
    verifier.S <== S;
    verifier.M <== M;
}

component main = VerifyEdDSAMiMC();
```

You know the next steps, but here's some help

1. Compile it: `circom circuit.circom -o circuit.json`
2. Generate your input for the circuit: `node generate_circuit_input.js`
3. Calculate the witness for the circuit: `snarkjs calculatewitness -c circuit.json -i input.json`
4. Generate the proof: `snarkjs setup -c circuit.json --protocol groth`
5. Verify the proof: `snarkjs verify`

## Part-2 Challenge

Modify the circuit and input to take in a length-3 preimage of the message as a private input, and hash them inside the circuit. To get you started:

```
include "../circomlib/circuits/eddsamimc.circom";
include "../circomlib/circuits/mimc.circom";

template VerifyEdDSAMiMC(k) {
    signal input from_x;
    signal input from_y;
    signal input R8x;
    signal input R8y;
    signal input S;
    signal private input preimage[k];

    component M = MultiMiMC7(k,91);
    M.in[0] <== // the first element of your preimage
    M.in[1] <== // the second element of your preimage
    M.in[2] <== // the third element of your preimage

    component verifier = EdDSAMiMCVerifier();
    verifier.enabled <== 1;
    verifier.Ax <== from_x;
    verifier.Ay <== from_y;
    verifier.R8x <== R8x;
    verifier.R8y <== R8y;
    verifier.S <== S;
    verifier.M <== M.out;
}

component main = VerifyEdDSAMiMC(3);
```

## Part-3 Verifying a Merkle proof

`cd 3_verify_merkle`

This example shows how to write smaller templates and use them as components in the main circuit. To verify a Merkle proof, we need to take in a leaf and its Merkle root, along with the path from the leaf to the root. Let's break this down into two circuits:

1. `get_merkle_root.circom`: this takes a leaf and a Merkle path and returns the computed Merkle root.
2. `leaf_existence.circom`: this compares an expected Merkle root with a computed Merkle root.

Create new file named `get_merkle_root.circom` and paste this code in:

```
include "../circomlib/circuits/mimc.circom";

template GetMerkleRoot(k){
// k is depth of tree

    signal input leaf;
    signal input paths2_root[k];
    signal input paths2_root_pos[k];

    signal output out;

    // hash of first two entries in tx Merkle proof
    component merkle_root[k];
    merkle_root[0] = MultiMiMC7(2,91);
    merkle_root[0].in[0] <== leaf - paths2_root_pos[0]* (leaf - paths2_root[0]);
    merkle_root[0].in[1] <== paths2_root[0] - paths2_root_pos[0]* (paths2_root[0] - leaf);

    // hash of all other entries in tx Merkle proof
    for (var v = 1; v < k; v++){
        merkle_root[v] = MultiMiMC7(2,91);
        merkle_root[v].in[0] <== paths2_root[v] - paths2_root_pos[v]* (paths2_root[v] - merkle_root[v-1].out);
        merkle_root[v].in[1] <== //can you figure this one out?
    }

    // output computed Merkle root
    out <== merkle_root[k-1].out;

}

component main = GetMerkleRoot(2);

```

Try to fill in the second line of the `for` loop using the pattern from the lines before. (The solution is in `sample_get_merkle_root.circom`.)

Now, make the second file `leaf_existence.circom` and paste this in:

```
include "./get_merkle_root.circom";

// checks for existence of leaf in tree of depth k

template LeafExistence(k){
// k is depth of tree

    signal input leaf;
    signal input root;
    signal input paths2_root_pos[k];
    signal input paths2_root[k];

    component computed_root = GetMerkleRoot(k);
    computed_root.leaf <== leaf;

    for (var w = 0; w < k; w++){
        computed_root.paths2_root[w] <== // assign elements from paths2_root
        computed_root.paths2_root_pos[w] <== // assign elements from paths2_root_pos
    }

    // equality constraint: input tx root === computed tx root
    root === computed_root.out;

}

component main = LeafExistence(2);
```

Make sure to REMOVE `component main = GetMerkleRoot(2)` from `get_merkle_root.circom`.

Modify your input to work with `leaf_existence.circom`.

## Part-3 Challenge

Like you did in the EdDSA verification exercise, provide the preimage of the leaf hash as private inputs to `leaf_existence.circom`, and hash them in the circuit.

## Data Structures

### Transaction

```js
class Transaction = {
    from: eddsa_pubkey,
    to: eddsa_pubkey,
    amount: integer
}
```

### Account

and an account as:

```js
class Account = {
    pubkey: eddsa_pubkey,
    balance: integer
}
```

NOTE: we also have a nonce for protection against replay attacks, but for simplicity let's consider it in the next example.

In ZkRollup, processing a single transaction involves:

- checking that the sender account existsin a tree of accounts, `accounts_root`
- checking that the hash of the transaction was signed by the sender
- debiting the sender account
- updating the `accounts_root` to get `intermediate_root`
- checking that the receiver account exists in `intermediate_root`
- crediting the receiver account
- updating the `accounts_root`to get `final_root`

## Part 3 - Processing a single transaction

Create a file called `circuit.circom` and put in this code. Fill in the signals for each component. Then, compile your circuit and test it against the `input.json` generated by running `node generate_circuit_input.js`.

```C##
include "./leaf_existence.circom";
include "./verify_eddsamimc.circom";
include "./get_merkle_root.circom";
include "../circomlib/circuits/mimc.circom";

template ProcessTx(k){
    // k is depth of accounts tree

    // accounts tree info
    signal input accounts_root;
    signal private input intermediate_root;
    signal private input accounts_pubkeys[2**k, 2];
    signal private input accounts_balances[2**k];

    // transactions info
    signal private input sender_pubkey[2];
    signal private input sender_balance;
    signal private input receiver_pubkey[2];
    signal private input receiver_balance;
    signal private input amount;
    signal private input signature_R8x;
    signal private input signature_R8y;
    signal private input signature_S;
    signal private input sender_proof[k];
    signal private input sender_proof_pos[k];
    signal private input receiver_proof[k];
    signal private input receiver_proof_pos[k];

    // output
    signal output new_accounts_root;

    // verify sender account exists in accounts_root
    component senderExistence = LeafExistence(k, 3);
    senderExistence.preimage[0] <== sender_pubkey[0];
    senderExistence.preimage[1] <== sender_pubkey[1];
    senderExistence.preimage[2] <== sender_balance;
    senderExistence.root <== accounts_root;
    for (var i = 0; i < k; i++){
        senderExistence.paths2_root_pos[i] <== sender_proof_pos[i];
        senderExistence.paths2_root[i] <== sender_proof[i];
    }

    // check that transaction was signed by sender
    component signatureCheck = VerifyEdDSAMiMC(5);
    signatureCheck.from_x <== sender_pubkey[0];
    signatureCheck.from_y <== sender_pubkey[1];
    signatureCheck.R8x <== signature_R8x;
    signatureCheck.R8y <== signature_R8y;
    signatureCheck.S <== signature_S;
    signatureCheck.preimage[0] <== sender_pubkey[0];
    signatureCheck.preimage[1] <== sender_pubkey[1];
    signatureCheck.preimage[2] <== receiver_pubkey[0];
    signatureCheck.preimage[3] <== receiver_pubkey[1];
    signatureCheck.preimage[4] <== amount;

    // debit sender account and hash new sender leaf
    component newSenderLeaf = MultiMiMC7(3,91){
        newSenderLeaf.in[0] <== sender_pubkey[0];
        newSenderLeaf.in[1] <== sender_pubkey[1];
        newSenderLeaf.in[2] <== sender_balance - amount;
    }

    // update accounts_root
    component computed_intermediate_root = GetMerkleRoot(k);
    computed_intermediate_root.leaf <== newSenderLeaf.out;
    for (var i = 0; i < k; i++){
        computed_intermediate_root.paths2_root_pos[i] <== sender_proof_pos[i];
        computed_intermediate_root.paths2_root[i] <== sender_proof[i];
    }

    // check that computed_intermediate_root.out === intermediate_root
    computed_intermediate_root.out === intermediate_root;

    // verify receiver account exists in intermediate_root
    component receiverExistence = LeafExistence(k, 3);
       // provide the appropriate signals to this component! see senderExistence for reference

    // credit receiver account and hash new receiver leaf
    component newReceiverLeaf = MultiMiMC7(3,91){
       // provide the appropriate signals to this component! see newSenderLeaf for reference
    }

    // update accounts_root
    component computed_final_root = GetMerkleRoot(k);
       // provide the appropriate signals to this component! see computed_intermediate_root for reference

    // output final accounts_root
    new_accounts_root <== computed_final_root.out;
}

component main = ProcessTx(1);
```

## Processing multiple transactions

Processing multiple transactions requires us to update the `accounts_root` many times before we arrive at the final one. This means we have to pre-compute all the `intermediate_roots` and pass them to the circuit to use in validating Merkle proofs.

Check out https://github.com/therealyingtong/RollupNC/blob/master/snark_circuit/multiple_tokens_transfer_and_withdraw.circom to see how it was implemented.
