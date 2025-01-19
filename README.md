What is this?
--------------

This is a stand-alone P2P transaction submission tool for submitting transactions more privately.

By default, this application expects TOR to be installed. If you don't want to install and/or use TOR, use the `--proxy None` option.

The motivation for this command is the `-walletbroadcast=0` command line option introduced in Bitcoin Core 0.11 (see the [release notes](https://github.com/bitcoin/bitcoin/blob/v0.11.0rc1/doc/release-notes.md#privacy-disable-wallet-transaction-broadcast)). Using the `-walletbroadcast=0` command line option with bitcoin core in conjunction with this tool allows you to submit transactions over TOR, but still use clearnet for downloading public blockchain data and receiving unconfirmed transactions from others.


Install
------------

Install dependencies:

`sudo apt update`
`sudo apt install python3-pip tor`

Install bitcoin-submittx:

`pip3 install git+https://github.com/AndySchroder/bitcoin-submittx`

If this is the first time you've installed a python script with pip, `bitcoin-submittx` may not be found and you will need to add `~/.local/bin` to your PATH variable. To do this, you can use the following command:

`echo "export PATH=$HOME/.local/bin:$PATH">>~/.bashrc`

and then open a new terminal to reload the `.bashrc` file. Alternatively, you can just always use `~/.local/bin/bitcoin-submittx` to execute the script instead of `bitcoin-submittx`.


Usage
--------

```
usage: bitcoin-submittx [-h] [--loglevel {TRACE,DEBUG,INFO4,INFO3,INFO2,INFO,WARNING,ERROR}] [--proxy PROXY] [--proxyrandomize | --no-proxyrandomize] [--timeout TIMEOUT] [--network NETWORK] [--nodes NODES]
                        [--nodes-file NODES_FILE] [--max-nodes MAX_NODES] [--tx-file TX_FILE]
                        [transactions]

Bitcoin Transaction Submission Tool

positional arguments:
  transactions          Serialized transactions (encoded as hex, separated by commas) to broadcast. If None and `tx-file` is also None, you will be prompted to enter (useful if you do not want the
                        transactions stored in your `.bash_history` file). (default: None)

options:
  -h, --help            show this help message and exit
  --loglevel {TRACE,DEBUG,INFO4,INFO3,INFO2,INFO,WARNING,ERROR}
                        Set the log level. Choose WARNING or ERROR to receive no output under normal circumstances. Note: most log levels do not show connection failures as we can many times have a successful
                        broadcast when only a small number of the total nodes that were contacted received the broadcast transaction(s). Use TRACE, DEBUG, or INFO4 to see connection failures. (default: INFO)
  --proxy PROXY, -p PROXY
                        SOCKS5 proxy to connect through. Set to `None` to not use a proxy. (default: 127.0.0.1:9050)
  --proxyrandomize, --no-proxyrandomize
                        If SOCKS5 proxy is defined, assume it is TOR and use stream isolation. (default: True)
  --timeout TIMEOUT, -t TIMEOUT
                        Number of seconds to wait before disconnecting from nodes (default: 15)
  --network NETWORK     Network to connect to (mainnet, regtest, testnet). This also determines the default port (default: mainnet)
  --nodes NODES         List of nodes to connect to, denoted either host or host:port, separated by commas. If None and `nodes-file` is also None, DNS seeds will be used to populate the node list. (default:
                        None)
  --nodes-file NODES_FILE, -n NODES_FILE
                        Read list of nodes from file (either host or host:port, separated one per line) (default: None)
  --max-nodes MAX_NODES
                        Max number of nodes to use in the node list. Set to 0 to select all nodes. (default: 35)
  --tx-file TX_FILE, -r TX_FILE
                        Read list of transactions from file (encoded as hex, separated one per line) (default: None)
```

The tool will connect to the provided nodes and announce the transactions. If the
nodes subsequently request them within the timeout, they are sent.

The return status of the program will be 0 if at least one node requested the transaction, and 1
otherwise.


Segwit support
---------------

As of 2017-11-30, this requires the master version of python-bitcoinlib.
Version 0.8 fails with `bitcoin.core.serialize.DeserializationExtraDataError: Not all bytes consumed during deserialization`.

A known issue is that bitcoin-submittx invs the wtxid instead of the txid, but
when it sends the tx the node accepts it. This is probably not correct per
BIP141. Patches welcome!


Caveats
-------
- DNS seed resolution does not happen over TOR.
- Don't know how to test if the TOR stream isolation is actually working properly, but it should be.
- If multiple transactions are submitted and TOR stream isolation is used, they are batched together when submitted to all nodes and a seperate TOR circuit is not used for each transaction that is submitted to each node.


TODOs and contribution ideas
-----------------------------

- IPv6 support
- Provide feedback of transaction reach, e.g. when connected to multiple nodes, it could monitor if the transaction comes back via another route.
- Check inputs and give clearer error messages if bad inputs provided.


