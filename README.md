# Doginals

A minter and protocol for inscriptions on Dogecoin. 

## ⚠️⚠️⚠️ Important ⚠️⚠️⚠️

Use this wallet for inscribing only! Always inscribe from this wallet to a different address, e.g. one you created with Woof Wallet. This wallet is not meant for storing funds or inscriptions.

## Prerequisites

To use this, you'll need to use your console/terminal and install Node.js on your computer. So please ensure, that you have your 

### Install NodeJS

Please head over to [https://nodejs.org/en/download](https://nodejs.org/en/download) and follow the installation instructions.

### Launch your own RPC 

In order to inscribe, you will need to have access to a Dodgecoin RPC. For example: [https://getblock.io/](https://getblock.io/) provides a service to get access to an RPC.
You will need that for the configuration.

## Setup

### git clone and install

Install by git clone (requires git and node on your computer) 

#### git clone
```
git clone https://github.com/verydogelabs/inscription-wallet.git
```

**or** 

download this [zip file](https://github.com/verydogelabs/inscription-wallet/archive/refs/heads/main.zip) and upack in a directory.

Now open your terminal and change to the directory the sources are installed.
####

```
cd <path to your download / installation>
npm install
``` 

After all dependencies are solved, you can configure the environment:

### Configure environment

Copy a `.env.example` to `.env` and add your node information:

```
NODE_RPC_URL=http://<ip>:<port>
# This is optional if you have an RPC from getblock.io
NODE_RPC_USER=<username>
NODE_RPC_PASS=<password>
TESTNET=false
FEE_PER_KB=500000000
```

You can get the current fee per kb from [here](https://blockchair.com/).

## Funding

Generate a new `.wallet.json` file:

```
node . wallet new
```

Then send DOGE to the address displayed. Once sent, sync your wallet:

```
node . wallet sync
```

If you are minting a lot, you can split up your UTXOs:

```
node . wallet split <count>
```

When you are done minting, send the funds back:

```
node . wallet send <address> <optional amount>
```

## Minting

From file:

```
node . mint <address> <path>
```

From data:

```
node . mint <address> <content type> <hex data>
```

Examples:

```
node . mint DSV12KPb8m5b6YtfmqY89K6YqvdVwMYDPn dog.jpeg
```

```
node . mint DSV12KPb8m5b6YtfmqY89K6YqvdVwMYDPn "text/plain;charset=utf-8" 576f6f6621 
```

**Note**: Please use a fresh wallet to mint to with nothing else in it until proper wallet for doginals support comes. You can get a paper wallet [here](https://www.fujicoin.org/wallet_generator?currency=Dogecoin).

## DRC-20

```
node . drc-20 mint <address> <ticker> <amount>
```

Examples: 

```
node . drc-20 mint DSV12KPb8m5b6YtfmqY89K6YqvdVwMYDPn dogi 1000
```

## Viewing

Start the server:

```
node . server
```

And open your browser to:

```
http://localhost:3000/tx/15f3b73df7e5c072becb1d84191843ba080734805addfccb650929719080f62e
```

## Protocol

The doginals protocol allows any size data to be inscribed onto subwoofers.

An inscription is defined as a series of push datas:

```
"ord"
OP_1
"text/plain;charset=utf-8"
OP_0
"Woof!"
```

For doginals, we introduce a couple extensions. First, content may spread across multiple parts:

```
"ord"
OP_2
"text/plain;charset=utf-8"
OP_1
"Woof and "
OP_0
"woof woof!"
```

This content here would be concatenated as "Woof and woof woof!". This allows up to ~1500 bytes of data per transaction.

Second, P2SH is used to encode inscriptions.

There are no restrictions on what P2SH scripts may do as long as the redeem scripts start with inscription push datas.

And third, inscriptions are allowed to chain across transactions:

Transaction 1:

```
"ord"
OP_2
"text/plain;charset=utf-8"
OP_1
"Woof and "
```

Transaction 2

```
OP_0
"woof woof!"
```

With the restriction that each inscription part after the first must start with a number separator, and number separators must count down to 0.

This allows indexers to know how much data remains.

## FAQ

### I'm getting ECONNREFUSED errors when minting

There's a problem with the node connection. Your `dogecoin.conf` file should look something like:

```
rpcuser=ape
rpcpassword=zord
rpcport=22555
server=1
```

Make sure `port` is not set to the same number as `rpcport`. Also make sure `rpcauth` is not set.

Your `.env file` should look like:

```
NODE_RPC_URL=http://127.0.0.1:22555
NODE_RPC_USER=ape
NODE_RPC_PASS=zord
TESTNET=false
```

### I'm getting "insufficient priority" errors when minting

The miner fee is too low. You can increase it up by putting FEE_PER_KB=300000000 in your .env file or just wait it out. The default is 100000000 but spikes up when demand is high.
