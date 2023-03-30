# gm
**gm** is a blockchain built using Cosmos SDK and Tendermint and created with [Ignite CLI](https://ignite.com/cli).

## Get started

```
ignite chain serve
```

`serve` command installs dependencies, builds, initializes, and starts your blockchain in development.

### Configure

Your blockchain in development can be configured with `config.yml`. To learn more, see the [Ignite CLI docs](https://docs.ignite.com).

### Web Frontend

Ignite CLI has scaffolded a Vue.js-based web app in the `vue` directory. Run the following commands to install dependencies and start the app:

```
cd vue
npm install
npm run serve
```

The frontend app is built using the `@starport/vue` and `@starport/vuex` packages. For details, see the [monorepo for Ignite front-end development](https://github.com/ignite/web).

## Release
To release a new version of your blockchain, create and push a new tag with `v` prefix. A new draft release with the configured targets will be created.

```
git tag v0.1
git push origin v0.1
```

After a draft release is created, make your final changes from the release page and publish it.

### Install
To install the latest version of your blockchain node's binary, execute the following command on your machine:

```
curl https://get.ignite.com/username/gm@latest! | sudo bash
```
`username/gm` should match the `username` and `repo_name` of the Github repository to which the source code was pushed. Learn more about [the install process](https://github.com/allinbits/starport-installer).

## Learn more

- [Ignite CLI](https://ignite.com/cli)
- [Tutorials](https://docs.ignite.com/guide)
- [Ignite CLI docs](https://docs.ignite.com)
- [Cosmos SDK docs](https://docs.cosmos.network)
- [Developer Chat](https://discord.gg/ignite)


GM world rollup
☀️ Introduction
In this tutorial, we will build a sovereign gm-world rollup using Rollkit and Celestia’s data availability and consensus layer to submit Rollkit blocks.

This tutorial will cover setting up Ignite CLI, building a Cosmos-SDK application-specific rollup blockchain, and posting data to Celestia. First, we will test on a local DA network and then we will deploy to a live testnet.

The Cosmos SDK is a framework for building blockchain applications. The Cosmos Ecosystem uses Inter-Blockchain Communication (IBC) to allow blockchains to communicate with one another.

The development journey for your rollup will look something like this:

Part one: Run your rollup and post DA to a local devnet, and make sure everything works as expected
Part two: Deploy the rollup, posting to a DA testnet. Confirm again that everything is functioning properly
Coming soon: Deploy your rollup to the DA layer's mainnet
NOTE
This tutorial will explore developing with Rollkit, which is still in Alpha stage. If you run into bugs, please write a Github Issue ticket or let us know in our Telegram.

CAUTION
The scripts for this tutorial are built for Celestia's Blockspacerace testnet. If you choose to use Mocha testnet or Arabica devnet, you will need to modify the script manually.

🤔 What is GM?
GM means good morning. It's GM o'clock somewhere, so there's never a bad time to say GM, Gm, or gm. You can think of "GM" as the new version of "hello world".

Part one
This part of the tutorial will teach developers how to easily run a local data availability (DA) devnet on their own machine (or in the cloud). Running a local devnet for DA to test your rollup is the recommended first step before deploying to a testnet. This eliminates the need for testnet tokens and deploying to a testnet until you are ready.

NOTE
Part one of the tutorial has only been tested on an AMD machine running Ubuntu 22.10 x64.

Whether you're a developer simply testing things on your laptop or using a virtual machine in the cloud, this process can be done on any machine of your choosing. We tested it out on a machine with the following specs:

Memory: 1 GB RAM
CPU: Single Core AMD
Disk: 25 GB SSD Storage
OS: Ubuntu 22.10 x64
💻 Prerequisites
Docker installed on your machine
🏠 Running local devnet with a Rollkit rollup
First, run the local-celestia-devnet by running the following command:

docker run --platform linux/amd64 -p 26650:26657 -p 26659:26659 ghcr.io/celestiaorg/local-celestia-devnet:main


TIP
The above command is different than the command in the Running a Local Celestia Devnet tutorial by Celestia Labs. Port 26657 on the Docker container in this example will be mapped to the local port 26650. This is to avoid clashing ports with the Rollkit node, as we're running the devnet and node on one machine.

🔎 Query your balance
Open a new terminal instance. Check the balance on your account that you'll be using to post blocks to the local network, this will make sure you can post rollup blocks to your Celestia Devnet for DA & consensus:

curl -X GET http://0.0.0.0:26659/balance

You will see something like this, denoting your balance in TIA x 10-6:

{"denom":"utia","amount":"999995000000000"}

If you want to be able to transpose your JSON results in a nicer format, you can install jq:

sudo apt install jq

TIP
We'll need jq later, so install it!

Then run this to prettify the result:

curl -X GET http://0.0.0.0:26659/balance | jq

Here's what my response was when I wrote this:

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    43  100    43    0     0   1730      0 --:--:-- --:--:-- --:--:--  1791
{
  "denom": "utia",
  "amount": "999995000000000"
}


If you want to clean it up some more, you can use the -s option to run curl in silent mode and not print the progress metrics:

curl -s -X GET http://0.0.0.0:26659/balance | jq

Your result will now look like this, nice 🫡

{
  "denom": "utia",
  "amount": "999995000000000"
}

🟢 Start, stop, or remove your container
Find the Container ID that is running by using the command:

docker ps

Then stop the container:

docker stop CONTAINER_ID_or_NAME

You can obtain the container ID or name of a stopped container using the docker ps -a command, which will list all containers (running and stopped) and their details. For example:

docker ps -a

This will give you an output similar to this:

CONTAINER ID   IMAGE                                            COMMAND            CREATED         STATUS         PORTS                                                                                                                         NAMES
d9af68de54e4   ghcr.io/celestiaorg/local-celestia-devnet:main   "/entrypoint.sh"   5 minutes ago   Up 2 minutes   1317/tcp, 9090/tcp, 0.0.0.0:26657->26657/tcp, :::26657->26657/tcp, 26656/tcp, 0.0.0.0:26659->26659/tcp, :::26659->26659/tcp   musing_matsumoto


In this example, you can restart the container using either its container ID (d9af68de54e4) or name (musing_matsumoto). To restart the container, run:

docker start d9af68de54e4

or

docker start musing_matsumoto

If you ever would like to remove the container, you can use the docker rm command followed by the container ID or name.

Here is an example:

docker rm CONTAINER_ID_or_NAME

🏗️ Building your sovereign rollup
Now that you have a Celestia devnet running, you are ready to install Golang. We will use Golang to build and run our Cosmos-SDK blockchain.

The Ignite CLI comes with scaffolding commands to make development of blockchains quicker by creating everything that is needed to start a new Cosmos SDK blockchain.

Install Golang (these commands are for amd64/linux):

cd $HOME
ver="1.19.1"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version

Now, use the following command to install Ignite CLI:

curl https://get.ignite.com/cli! | bash

TIP
If you have issues with installation, the full guide can be found here or on docs.ignite.com. The above command was tested on amd64/linux.

Check your version:

ignite version

Open a new tab or window in your terminal and run this command to scaffold your rollup. Scaffold the chain:

cd $HOME
ignite scaffold chain gm --address-prefix gm

TIP
The --address-prefix gm flag will change the address prefix from cosmos to gm. Read more on the Cosmos docs.

The response will look similar to below:

jcs @ ~ % ignite scaffold chain gm

⭐️ Successfully created a new blockchain 'gm'.
👉 Get started with the following commands:

 % cd gm
 % ignite chain serve

Documentation: https://docs.ignite.com

This command has created a Cosmos SDK blockchain in the gm directory. The gm directory contains a fully functional blockchain. The following standard Cosmos SDK modules have been imported:

staking - for delegated Proof-of-Stake (PoS) consensus mechanism
bank - for fungible token transfers between accounts
gov - for on-chain governance
mint - for minting new units of staking token
nft - for creating, transferring, and updating NFTs
and more
Change to the gm directory:

cd gm

You can learn more about the gm directory’s file structure here. Most of our work in this tutorial will happen in the x directory.

🗞️ Install Rollkit
To swap out Tendermint for Rollkit, run the following command from inside the gm directory:

go mod edit -replace github.com/cosmos/cosmos-sdk=github.com/rollkit/cosmos-sdk@v0.46.7-rollkit-v0.7.2-no-fraud-proofs
go mod edit -replace github.com/tendermint/tendermint=github.com/celestiaorg/tendermint@v0.34.22-0.20221202214355-3605c597500d
go mod tidy
go mod download


▶️ Start your rollup
Download the init.sh script to start the chain:

# From inside the `gm` directory
wget https://raw.githubusercontent.com/rollkit/docs/main/docs/scripts/gm/init-local.sh


Run the init-local.sh script:

bash init-local.sh

This will start your rollup, connected to the local Celestia devnet you have running.

Now let's explore a bit.

🔑 Keys
List your keys:

gmd keys list --keyring-backend test

You should see an output like the following

- address: gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7
  name: gm-key-2
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AlXXb6Op8DdwCejeYkGWbF4G3pDLDO+rYiVWKPKuvYaz"}'
  type: local
- address: gm13nf52x452c527nycahthqq4y9phcmvat9nejl2
  name: gm-key
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AwigPerY+eeC2WAabA6iW1AipAQora5Dwmo1SnMnjavt"}'
  type: local


💸 Transactions
Now we can test sending a transaction from one of our keys to the other. We can do that with the following command:

gmd tx bank send [from_key_or_address] [to_address] [amount] [flags]

Set your keys as variables to make it easier to add the address:

export KEY1=gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7
export KEY2=gm13nf52x452c527nycahthqq4y9phcmvat9nejl2

So using our information from the keys command, we can construct the transaction command like so to send 42069stake from one address to another:

gmd tx bank send $KEY1 $KEY2 42069stake --keyring-backend test

You'll be prompted to accept the transaction:

auth_info:
  fee:
    amount: []
    gas_limit: "200000"
    granter: ""
    payer: ""
  signer_infos: []
  tip: null
body:
  extension_options: []
  memo: ""
  messages:
  - '@type': /cosmos.bank.v1beta1.MsgSend
    amount:
    - amount: "42069"
      denom: stake
    from_address: gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7
    to_address: gm13nf52x452c527nycahthqq4y9phcmvat9nejl2
  non_critical_extension_options: []
  timeout_height: "0"
signatures: []
confirm transaction before signing and broadcasting [y/N]:

Type y if you'd like to confirm and sign the transaction. Then, you'll see the confirmation:

code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: 677CAF6C80B85ACEF6F9EC7906FB3CB021322AAC78B015FA07D5112F2F824BFF

⚖️ Balances
Then, query your balance:

gmd query bank balances $KEY2

This is the key that received the balance, so it should have increased past the initial STAKING_AMOUNT:

balances:
- amount: "10000000000000000000042069"
  denom: stake
pagination:
  next_key: null
  total: "0"

The other key, should have decreased in balance:

gmd query bank balances $KEY1

Response:

balances:
- amount: "9999999999999999999957931"
  denom: stake
pagination:
  next_key: null
  total: "0"

Part two
🛠️ Setup
Operating systems: GNU/Linux, macOS, or Windows Subsystem for Linux (WSL)
Recommended GNU/Linux or macOS
Golang
Ignite CLI v0.25.1
Homebrew
wget
jq
A Celestia Light Node
Linux
Mac
TIP
If you already installed Ignite and Golang in Part one, feel free to skip to the next section.

Be sure to use the same testnet installation instructions through this entire tutorial.

🏃 Install Golang on Linux
Celestia-App, Celestia-Node, and Cosmos-SDK are written in the Golang programming language. You will need Golang to build and run them.

You can install Golang here.

🔥 Install Ignite CLI on Linux
First, you will need to create /usr/local/bin if you have not already:

sudo mkdir -p -m 775 /usr/local/bin

Run this command in your terminal to install Ignite CLI:

curl https://get.ignite.com/cli! | bash

TIP
✋ On some machines, you may run into permissions errors like the one below. You can resolve this error by following the guidance here or below.

# Error
jcs @ ~ % curl https://get.ignite.com/cli! | bash


  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3967    0  3967    0     0  16847      0 --:--:-- --:--:-- --:--:-- 17475
Installing ignite v0.25.1.....
######################################################################## 100.0%
mv: rename ./ignite to /usr/local/bin/ignite: Permission denied
============
Error: mv failed


The following command will resolve the permissions error:

sudo curl https://get.ignite.com/cli! | sudo bash

A successful installation will return something similar the response below:

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3967    0  3967    0     0  15586      0 --:--:-- --:--:-- --:--:-- 15931
Installing ignite v0.25.1.....
######################################################################## 100.0%
Installed at /usr/local/bin/ignite


Verify you’ve installed Ignite CLI by running:

ignite version

The response that you receive should look something like this:

jcs @ ~ % ignite version
Ignite CLI version: v0.25.1
Ignite CLI build date: 2022-10-20T15:52:00Z
Ignite CLI source hash: cc393a9b59a8792b256432fafb472e5ac0738f7c
Cosmos SDK version: v0.46.3
Your OS: darwin
Your arch: arm64
Your Node.js version: v18.10.0
Your go version: go version go1.19.2 darwin/arm64
Your uname -a: Darwin Joshs-MacBook-Air.local 21.6.0 Darwin Kernel Version 21.6.0: Mon Aug 22 20:20:07 PDT 2022; root:xnu-8020.140.49~2/RELEASE_ARM64_T8110 arm64
Your cwd: /Users/joshstein
Is on Gitpod: false


🪶 Run a Celestia light node
Follow instructions to install and start your Celestia Data Availalbility layer Light Node selecting the network that you had previously used. You can find instructions to install and run the node here.

After you have Go and Ignite CLI installed, and your Celestia Light Node running on your machine, you're ready to build, test, and launch your own sovereign rollup.

💬 Say gm world
Now, we're going to get our blockchain to say gm world! - in order to do so you need to make the following changes:

Modify a protocol buffer file
Create a keeper query function that returns data
Protocol buffer files contain proto RPC calls that define Cosmos SDK queries and message handlers, and proto messages that define Cosmos SDK types. The RPC calls are also responsible for exposing an HTTP API.

The Keeper is required for each Cosmos SDK module and is an abstraction for modifying the state of the blockchain. Keeper functions allow us to query or write to the state.

✋ Create your first query
Open a new terminal instance that is not the same that you started the chain in.

In your new terminal, cd into the gm directory and run this command to create the gm query:

ignite scaffold query gm --response text

Response:

modify proto/gm/gm/query.proto
modify x/gm/client/cli/query.go
create x/gm/client/cli/query_gm.go
create x/gm/keeper/query_gm.go

🎉 Created a query `gm`.

What just happened? query accepts the name of the query (gm), an optional list of request parameters (empty in this tutorial), and an optional comma-separated list of response field with a --response flag (text in this tutorial).

Navigate to the gm/proto/gm/gm/query.proto file, you’ll see that Gm RPC has been added to the Query service:

gm/proto/gm/gm/query.proto
service Query {
  rpc Params(QueryParamsRequest) returns (QueryParamsResponse) {
    option (google.api.http).get = "/gm/gm/params";
  }
    rpc Gm(QueryGmRequest) returns (QueryGmResponse) {
        option (google.api.http).get = "/gm/gm/gm";
    }
}

The Gm RPC for the Query service:

is responsible for returning a text string
Accepts request parameters (QueryGmRequest)
Returns response of type QueryGmResponse
The option defines the endpoint that is used by gRPC to generate an HTTP API
📨 Query request and response types
In the same file, we will find:

QueryGmRequest is empty because it does not require parameters
QueryGmResponse contains text that is returned from the chain
gm/proto/gm/gm/query.proto
message QueryGmRequest {
}

message QueryGmResponse {
  string text = 1;
}

👋 Gm keeper function
The gm/x/gm/keeper/query_gm.go file contains the Gm keeper function that handles the query and returns data.

gm/x/gm/keeper/query_gm.go
func (k Keeper) Gm(goCtx context.Context, req *types.QueryGmRequest) (*types.QueryGmResponse, error) {
    if req == nil {
        return nil, status.Error(codes.InvalidArgument, "invalid request")
    }
    ctx := sdk.UnwrapSDKContext(goCtx)
    _ = ctx
    return &types.QueryGmResponse{}, nil
}


The Gm function performs the following actions:

Makes a basic check on the request and throws an error if it’s nil
Stores context in a ctx variable that contains information about the environment of the request
Returns a response of type QueryGmResponse
Currently, the response is empty and you'll need to update the keeper function.

Our query.proto file defines that the response accepts text. Use your text editor to modify the keeper function in gm/x/gm/keeper/query_gm.go .

gm/x/gm/keeper/query_gm.go
func (k Keeper) Gm(goCtx context.Context, req *types.QueryGmRequest) (*types.QueryGmResponse, error) {
    if req == nil {
        return nil, status.Error(codes.InvalidArgument, "invalid request")
    }
    ctx := sdk.UnwrapSDKContext(goCtx)
    _ = ctx
    return &types.QueryGmResponse{Text: "gm world!"}, nil
}


🟢 Start your sovereign rollup
CAUTION
Before starting our rollup, we'll need to find and change FlagIAVLFastNode to FlagDisableIAVLFastNode:

gm/cmd/gmd/cmd/root.go
baseapp.SetIAVLDisableFastNode(cast.ToBool(appOpts.Get(server.FlagDisableIAVLFastNode))),


Also, if you are on macOS, you will need to install md5sha1sum:

brew install md5sha1sum

We have a handy init-testnet.sh found in this repo here.

We can copy it over to our directory with the following commands:

# From inside the `gm` directory
wget https://raw.githubusercontent.com/rollkit/docs/main/docs/scripts/gm/init-testnet.sh


This copies over our init-testnet.sh script to initialize our gm rollup.

You can view the contents of the script to see how we initialize the gm rollup.

Clear previous chain history
Before starting the rollup, we need to remove the old project folders:

cd $HOME
rm -r go/bin/gmd && rm -rf .gm

Start the new chain
Now, you can initialize the script with the following command:

bash init-testnet.sh

With that, we have kickstarted our second gmd network!

The query command has also scaffolded x/gm/client/cli/query_gm.go that implements a CLI equivalent of the gm query and mounted this command in x/gm/client/cli/query.go.

In a separate window, run the following command:

gmd q gm gm

We will get the following JSON response:

text: gm world!


Congratulations 🎉 you've successfully built your first rollup and queried it!
