Knowledge base

#### Binaries overview

Every CryptoNote coin interacts with the network through 2 binaries.

##### 1. Daemon

Default name is “cryptonotecoind”. Daemon is responsible for any communication with the network:

- Mining
- Interaction with the blockchain, e.g. blocks relaying, getting info about the block, etc.
- Peer list look up
- Connections look up
- Transaction pool information and relaying

##### 2. Wallet

Default name is “simplewallet”. It carries out basic wallet's functions:

- Wallet creation
- Financial operations, e.g. sending, receiving, checking for incoming transfers, etc.

Simplewallet also allows user to start/stop mining without a need to specify the wallet in the daemon.

In CryptoNote only the simplewallet knows about user's private keys and funds. The daemon is used solely for the purpose of network communication. Simplewallet and daemon binaries work through JSON RPC. You may run the binaries from the separate machines, but for the security purposes you should make sure that daemon won't be accesible from anywhere but the simplewallet's ip address.










Processing payments (3rd party integration)

Daemon & Wallet as RPC

If you want your website or service to work with CryptoNote currencies you will have to tweak a few things in the daemon and wallet.

First of all you should start both daemon and simplewallet in RPC mode. For the daemon to work as RPC you don’t have to do much; simply start it as usual.

To start the wallet JSON RPC API server you should specify the port to which the server binds (additionally to standard wallet's arguments). You may choose any free port. Execute the following command from the command line:

Example:
simplewallet --wallet-file=example_wallet.bin --pass=12345 --rpc-bind-port=8082

Note: if you'd like your daemon to accept incoming connections from the outside of your local machine, you should start the daemon with the following arguments: --rpc-bind-ip=0.0.0.0 --rpc-bind-port=arg. You should also specify an IP address of the daemon when starting simplewallet. There are a few ways to do that:

--daemon-address=arg
simplewallet connects to the daemon instance at host:port
--daemon-host=arg
simplewallet cinnects to the daemon instance at host arg instead of localhost
--daemon-port=arg
simplewallet connects to the localhost daemon instance at port arg instead of 8081
To sum it up, you should start simplewallet as follows:

Example:
simplewallet --wallet-file=example_wallet.bin --pass=12345 --daemon-address=123.123.1.1:8082
simplewallet --wallet-file=example_wallet.bin --pass=12345 --daemon-host=123.123.1.1
simplewallet --wallet-file=example_wallet.bin --pass=12345 --daemon-port=8082

Important notes:
1. For simplewallet operating in RPC mode the "refresh" command is not working. The balance is automatically refreshed every 20 seconds.
2. You should not launch more than 1 instance of the same wallet in RPC mode.


Payment identification

Services accepting deposits in the CryptoNote currency should run only 1 instance of the simplewallet for each of the currencies. Due to the CryptoNote’s high cryptographic security and anonymity the process of reaching out to the multiple addresses would require a lot of resources. In order to solve this problem a new feature called payment_id was introduced. Payment_id allows the service to run only 1 wallet for all the incoming payments of all its users. The unique payment_id argument of the transfer should be generated for each of the website's users in order to identify deposits.

To create a payment_id you should generate random 32 bytes of data and then convert it to HEX. All you have to do is assign payment_id to each of your users. Users will be sending you the funds using “transfer mixin_count address amount payment_id” command. We suggest you help your users by providing them with the complete transfer command including the payment_id.

To check for user’s payments you should use get_payments method. This method returns all the payments with the corresponding payment_ids that were deposited to the wallet.

Example:
{
  "jsonrpc":"2.0",
  "method":"get_payments",
  "params":
    {
      "payment_id": "78cc4b76a48bd50ab955ac61a0c04e4a82079fbcf27298f87b39c76aefccbcc9"
    }
}

Note: please, avoid crediting user balances until they deposits are confirmed and unlocked in your wallet. The number of blocks for the funds to get unlocked may vary across CryptoNote currencies.


Cold and hot storage

The whole cold/hot wallet concept is based on the fact that it’s easier to hack the wallet that is connected to the internet.

Thus, we strongly recommend you to create a new wallet, synchronize it with a network, transfer the majority of your funds to it, disconnect it from the network, and upload this wallet to an encrypted flash drive or any other secure offline device.

For everyday transfers we recommend you to create a different wallet that will contain amount of funds that are required for your normal operations.


Transfer payments through the wallet RPC

Transfer method looks as follows:

{
  "jsonrpc":"2.0",
  "method":"transfer",
  "params":
    {
      "destinations":
        [
          {
            "amount":11111,
            "address":"248iCFTjpRKGdoD34B6nJVfgLLgnoucNhMYWeTVAAkpoQQV6yVve2sxh6QVxSh4HW213Wy5siw8eBa5k2G1sV639QzFWqGD"
          },
          {
            "amount":22222,
            "address":"248iCFTjpRKGdoD34B6nJVfgLLgnoucNhMYWeTVAAkpoQQV6yVve2sxh6QVxSh4HW213Wy5siw8eBa5k2G1sV639QzFWqGD"
           }
        ],
      "payment_id":"",
      "fee":1000000,
      "mixin":0,
      "unlock_time":0
    }
}

Please note, payment_id is an optional argument and may be left out.
