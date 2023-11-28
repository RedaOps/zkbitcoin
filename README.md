# zkBitcoin

## Set up bitcoind

Both users and nodes need access to Bitcoin.

1. download latest release on https://bitcoincore.org/en/releases/ (for example `wget https://bitcoincore.org/bin/bitcoin-core-25.1/bitcoin-25.1-x86_64-linux-gnu.tar.gz`)
2. you can run bitcoind with `./bin/bitcoind -testnet -server=1 -rpcuser=root -rpcpassword=hello`
    - or you can set these in `bitcoin.conf` and copy that file in `~/.bitcoin/bitcoin.conf`
3. you can query it for testing with `./bin/bitcoin-cli -rpcport=18332 -rpcuser=root -rpcpassword=hellohello getblock`

you can also use curl to query it:

```console
curl --user root --data-binary '{"jsonrpc": "1.0", "id": "curltest", "method": "getblock", "params": ["00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09"]}' -H 'content-type: text/plain;' http://127.0.0.1:18332/
```

If you want to expose this to the internet, you can setup a reverse proxy like nginx. This is the config I use (in `/etc/nginx/sites-enabled/bitcoind-proxy.conf`):

```
server {
    listen 18331;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18332;
    }
}
```

## Our own bitcoind

I'm running one on digitalocean at [146.190.33.39](http://146.190.33.39)

You can query it with:

```console
curl --user root:hellohello --data-binary '{"jsonrpc": "1.0", "id": "curltest", "method": "getblockchaininfo", "params": []}' -H 'content-type: text/plain;' http://146.190.33.39:18331
```
