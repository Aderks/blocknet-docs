# **Blocknet Enterprise XRouter Proxy**

## Pre-requisites:

* Blocknet Service Node (5000 BLOCK)
* Docker


## Setting up the inital `uwsgi.ini` configuration file:

* Create a new directory and a `uwsgi.ini` file
   * This example will use: `/xrproxy/uwsgi.ini`

* Sample `uwsgi.ini`:

```
## SPV sample configuration file
[uwsgi]
processes = 8
threads = 2

# Place your Service Node private key here (this is not a wallet private key!)
# Allows the XRouter Proxy to sign packets on your Service Node's behalf
# DO NOT SHARE THIS KEY
set-ph = SERVICENODE_PRIVKEY=cV1bo3ME3qvw9Sxzo72skbFsAQ6ihyT6F8VXMe8mzv6EJoqFVXMV

#  mainnet or testnet
set-ph = BLOCKNET_CHAIN=mainnet

# Handle XRouter payments
set-ph = HANDLE_PAYMENTS=true
set-ph = HANDLE_PAYMENTS_RPC_HOSTIP=192.168.1.25
set-ph = HANDLE_PAYMENTS_RPC_PORT=41419
set-ph = HANDLE_PAYMENTS_RPC_USER=user
set-ph = HANDLE_PAYMENTS_RPC_PASS=pass
set-ph = HANDLE_PAYMENTS_RPC_VER=2.0

# Sample SPV RPC configuration
set-ph = RPC_BLOCK_HOSTIP=192.168.1.25
set-ph = RPC_BLOCK_PORT=41419
set-ph = RPC_BLOCK_USER=user
set-ph = RPC_BLOCK_PASS=pass
set-ph = RPC_BLOCK_VER=2.0

# Sample XCloud plugin configuration
set-ph = URL_SomeCustomPlugin_HOSTIP=192.168.1.5
set-ph = URL_SomeCustomPlugin_PORT=8080
```

* Adjust `processes` and `threads` related to your setup

* Change `SERVICENODE_PRIVKEY=` to your servicenode privkey
   * On your Service Node client type `servicenodestatus` and retrieve your `snodeprivkey`

* Change `RPC_BLOCK_...=` HOSTIP/PORT/USER/PASS to your settings in your service node `blocknet.conf`

* Save and close the file

* Anytime you edit this file, you need to restart the container. (or just `service nginx restart` within the container??)


## Setting up `xrouter.conf`

* To use the XRouter Proxy you have to user a port that is not `port=41412`
   * Set `port=80` or any other port you have designated eg. 9090

* Set `host=` to the server IP or server domain name
   * `host=10.0.0.1` or `host=api.domainnamehere.com`

* Save and close the file.

* In your servicenode client type `xrReloadConfigs` for the settings to take. Send the new information out to the network by using `servicenodesendping`


## Pull and run the XRproxy

* In a console where Docker is available type: `docker pull blocknetdx/xrouterproxy:latest`

* Running the xrproxy in a container:

* `docker run -d --name xrproxy -p 80:80 -p 443:443 -v=/xrproxy:/opt/uwsgi/conf`
   * `--name xrproxy` names the container 'xrproxy'
   * `-p 80:80` exposes the container to port 80. Use this if you set `port=80` in your `xrouter.conf`
      * `-p 9090:80` Use this if you set `port=9090` in your `xrouter.conf`
   * Optional: `-p 443:443` Add this if you wish to provide a HTTPS API end point.
      * Note: This proxy does not support 443 through blocknet client.
   * `-v=/xrproxy:/opt/uwsgi/conf` This is connecting a volume to the xrproxy container.
      * `-v=/dir/to/conf:/opt/uwsgi/conf` change `/dir/to/conf` to where the `uwsgi.ini` file is saved locally

* To access inside the container type: `docker exec -it xrproxy /bin/bash`

* To access logs type: `docker logs xrproxy`

* Test to see if the XRProxy is working:
   * `curl -X POST -H "content-type: application/json" --data-binary '[]' localhost:9090/xr/BLOCK/xrGetBlockCount`
      * Adjust `localhost:9090` to your settings
         * If you used `port=80` you can just use your IP. `10.0.0.1/xr/BLOCK/xrGetBlockCount` or `api.domainnamehere.com/xr/BLOCK/xrGetBlockCount`


## Setting up additional XWallets

* Any additional XWallets are added based on this SPV template.
   * `set-ph = RPC_XWALLET_...=` for HOSTIP, PORT, USER, PASS, VER

* Litecoin example:

```
set-ph = RPC_LTC_HOSTIP=127.0.0.1
set-ph = RPC_LTC_PORT=9332
set-ph = RPC_LTC_USER=litecoin
set-ph = RPC_LTC_PASS=litecoin1234
set-ph = RPC_LTC_VER=2.0
```

* Ensure to have a `LTC` entry in the `wallets=` area of your `xrouter.conf`   


## Setting up XCloud RPC plugins

* XCloud RPC plugins are setup similar to XWallets, but you state which RPC call to invoke with `set-ph RPC_..._METHOD=`

* Example: XCloud RPC plugin for Litecoin's `getbestblockhash`

* `uwsgi.ini` entry:

```
# Litecoin RPC XCloud Services

## getbestblockhash
set-ph = RPC_ltc_getbestblockhash_HOSTIP=127.0.0.1
set-ph = RPC_ltc_getbestblockhash_PORT=9332
set-ph = RPC_ltc_getbestblockhash_METHOD=getbestblockhash
set-ph = RPC_ltc_getbestblockhash_USER=litecoin
set-ph = RPC_ltc_getbestblockhash_PASS=litecoin1234
set-ph = RPC_ltc_getbestblockhash_VER=2.0
```

* Ensure to enter `ltc_getbestblockhash` in the `plugins=` area of your `xrouter.conf`
   * You will need a `ltc_getbestblockhash.conf` entry for your Blocknet client to load the service

* Test to see if this XCloud RPC is working:
   `curl -X POST -H "content-type: application/json" --data-binary '[]' localhost:9090/xrs/ltc_getbestblockhash`


## Setting up XCloud URL plugins

* You will need to setup a container or webserver for non-rpc based calls

* Currency Exchange plugin example:

```
set-ph = URL_CurrencyExchangeRate_HOSTIP=172.17.0.1
set-ph = URL_CurrencyExchangeRate_PORT=9195
```

   * The webserver needs to be setup so that you can cURL `172.17.0.1:9195/xrs/CurrencyExchangeRate`

   * Ensure to enter `CurrencyExchangeRate` in the `plugins=` area of your `xrouter.conf`
      * You will need a `CurrencyExchangeRate.conf` entry for your Blocknet client to load the service

      ```
      parameters=string,string
	  fee=0
	  clientrequestlimit=30000
      disabled=0
	  help=Outputs the exchange rate between two currency pairs; CurrencyExchangeRate [currency_id1] [currency_id2]; Example: xrService xrs::CurrencyExchangeRate USD CAD; To see a list of currency ID's use xrService xrs::WorldCurrencyList

	  private::type=rpc
	  private::rpcip=172.17.0.1
	  private::rpcport=9195
	  ```

* Test to see if this XCloud plugin is working:
   `curl -X POST -H "content-type: application/json" --data-binary '["CAD","USD"]' localhost:9090/xrs/CurrencyExchangeRate`


## Use Domain Name Instead of Server IP

* Allow `port 80` through your firewall

* Register a domain name
   * Point the A record to your server IP (this may take up to 24hrs for the change to happen)
   * A record, @, Server IP address

* Enter the container: `docker exec -it xrproxy /bin/bash`

* Edit the `nginx.conf` file located at `/etc/nginx/nginx.conf`
	* Find the `server` block in the conf
	* Adjust the conf to point to your domain name

	```
	server {
		listen 80;
		server_name api.domainnamehere.com;

	...	

	}
	```

* While inside the container restart nginx for the settings to take place: `service nginx restart`

* Adjust your `xrouter.conf` `host=` to point to the domain name and `port=80` will need to be set.
   * `xrReloadConf` and `servicenodesendpring` in client 
