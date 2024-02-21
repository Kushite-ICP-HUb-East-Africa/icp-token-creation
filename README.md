# ICP TOKEN CREATION

## Introduction 
The standard for creating tokens on Internet Computer is called ```ICRC-1 token standard``` 

``ICRC``: means Internet Computer Request Comments 

The ``ICRC-1 Standard`` defines a universally accepted standard for creating and recording token transactions on Internet Computer Protocol 

Tokens that are to be/created are to fullfil all requirements that are in the standard 

You can check the detailed account of the standard [here](https://github.com/dfinity/ICRC-1/tree/main/standards/ICRC-1)

## Using ICP Ledger
ICP Ledger is a canister that is used for interacting with the ICP Token i.e sending to another account, converting it to cycles 

Deploying a local ICP Ledger is very important as your local replica cannot access the mainnet ICP ledger

#### Note: The ICP Ledger can only be used to interact with ICP Token, in order to interact with other tokens i.e ICRC Tokens, you will need an ICRC Ledger, which we'll be coverning in this tutorial 

Main difference between ICP Ledger and ICRC-1 Ledger is the implementation of accounts: 
* ICRC-1 specifies ``account`` which is a struct that contains a ``principal`` and optional `subaccount`
* On the other hand, ICP Ledger uses ``AccountIdentifier`` which is a hash of the ICRC-1 value 

## Deploying ICRC-1 Ledger Locally: 
You will need to download the wasm and candid files for the ICRC-1 standard 

These are the URLs you will use: 
* Wasm module: https://download.dfinity.systems/ic/d87954601e4b22972899e9957e800406a0a6b929/canisters/ic-icrc1-ledger.wasm.gz 

* Candid file: https://raw.githubusercontent.com/dfinity/ic/d87954601e4b22972899e9957e800406a0a6b929/rs/rosetta-api/icrc1/ledger/ledger.did 

## Step 1: Installing dependencies
Run the commands: 
```
dfx new my_token_project
``` 

```
cd my_token_project
``` 

## Step 2: Confuguring your dfx.json file
Open your dfx json and replace the existing content with: 
```
{
  "canisters": {
    "my_token_project": {
      "type": "custom",
      "candid": "https://raw.githubusercontent.com/dfinity/ic/d87954601e4b22972899e9957e800406a0a6b929/rs/rosetta-api/icrc1/ledger/ledger.did",
      "wasm": "https://download.dfinity.systems/ic/d87954601e4b22972899e9957e800406a0a6b929/canisters/ic-icrc1-ledger.wasm.gz",
    }
  },
  "defaults": {
    "build": {
      "args": "",
      "packtool": ""
    }
  },
  "output_env_file": ".env",
  "version": 1
}
```

Note that we've completely removed the frontend configuration and also changed the backend configuration. 

For the backend configuration, the ``type`` has been changed from ``motoko`` to ``custom``, since we're not using motoko language to create the token 

We've also added a ``candid`` and ``wasm`` urls for the modules, since these will be using the pre-configured modules to create our token 

## Step 3: Creating a minter identity
In order for us to interact with the token, we'll create a new identity that will act as the ``minting`` account. We will then export the the account's ID to be used as the environment variable for ``MINTER``

For those that don't remember, identity is like the authentication service for internet computer

First before creating the identity you'll check out the identities that you currently have using the command 

```
dfx identity list
```

This will print a list of all identities that you currently have. Once you've seen the identities you have and verified you don't have an identity called  ``minter`` you can now proceed to create one using the command below: 

```
dfx identity new minter 

dfx identity use minter 

export MINTER=$(dfx identity get-principal)
```

Transfer from the minting account will make a ``Mint`` transaction, while transfer to the minting account will make a  ``Burn`` transaction 

## Step 4: Exporting the name and symbol of your token 
You can do that using the commands below 

```
export TOKEN_NAME="My First IC Token" 
export TOKEN_SYMBOL="MFICT" 
```

These are the environment variables that will be used for token creation 

## Step 5: Set the identity that you'll use to deploy the ledger 
We will use the default ``DevJourney`` identity 

```
dfx identity use DevJourney 
export DEPLOY_ID=$(dfx identity get-principal)
```

## Step 6: Set the amount of pre-minted tokens that you'll mint when deploying the ledger and the transfer fees 
Use the command below: 

```
export PRE_MINTED_TOKENS=10_000_000_000
export TRANSFER_FEE=10_000
```

## Step 7: Set the values for the ledger's archiving options
Use the following commands: 

```
dfx identity new archive_controller 
dfx identity use archive_controller 

export ARCHIVE_CONTROLLER=$(dfx identity get-principal)
export TRIGGER_THRESHOLD=2000 
export NUM_OF_BLOCK_TO_ARCHIVE=1000 
export CYCLE_FOR_ARCHIVE_CREATION=10000000000000 

```

Here are the uses of the above variables: 
* ``TRIGGER_THRESHOLD``: number of blocks that will be archived once the trigger threshold is achieved 
* ``NUM_OF_BLOCK_TO_ARCHIVE``: The amount of blocks to be archived 
* ``CYCLE_FOR_ARCHIVE_CREATION``: amount of cycles to be sent to the archive canister when it is deployed 


## Step 8: Specify which standards you want your ledger to support: 

```
export FEATURE_FLAGS=true 
``` 

If you set this to true it will support ICRC-2 standard extension, but if it is set to false it will only support ICRC-1 standard 

## Step 9: Deploy the ICRC-1 Ledger canister locally: 

You will use this command: 

```
dfx deploy my_token_project --argument "(variant {Init = 
record {
     token_symbol = \"${TOKEN_SYMBOL}\";
     token_name = \"${TOKEN_NAME}\";
     minting_account = record { owner = principal \"${MINTER}\" };
     transfer_fee = ${TRANSFER_FEE};
     metadata = vec {};
     feature_flags = opt record{icrc2 = ${FEATURE_FLAGS}};
     initial_balances = vec { record { record { owner = principal \"${DEPLOY_ID}\"; }; ${PRE_MINTED_TOKENS}; }; };
     archive_options = record {
         num_blocks_to_archive = ${NUM_OF_BLOCK_TO_ARCHIVE};
         trigger_threshold = ${TRIGGER_THRESHOLD};
         controller_id = principal \"${ARCHIVE_CONTROLLER}\";
         cycles_for_archive_creation = opt ${CYCLE_FOR_ARCHIVE_CREATION};
     };
 }
})"
```

Once your ledger deploys succesfully this is the url you'll see: 
```
  my_token_project: http://127.0.0.1:4943/?canisterId=bd3sg-teaaa-aaaaa-qaaba-cai&id=bkyz2-fmaaa-aaaaa-qaaaq-cai
```





