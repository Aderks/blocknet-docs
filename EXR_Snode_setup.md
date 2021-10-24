title: Enterprise XRouter Proxy description: This guide explains how to setup and configure your Service Node to utilize Blocknet's Enterprise XRouter Proxy.

# Enterprise XRouter Proxy
This guide explains how to setup and configure your [Service Node](/service-nodes/introduction) to utilize Blocknet's Enterprise XRouter Proxy (EXR).

The XRouter Proxy acts as a reverse proxy (similar to a load balancer) for all your XRouter services and oracles. The reverse proxy is comprised of an nginx web server and a uwsgi python handler.

XRouter Proxy is designed to work in Docker a environment, although this is not a mandatory requirement. This guide will be following a Docker environment and assumes you have Docker installed.

To setup your Enterprise XRouter Proxy, follow these steps:

1. [EXR Configuration File Setup](/service-nodes/exr/#exr-configuration-file-setup)
1. [Service Node Configuration File Setup](/service-nodes/exr/#service-node-configuration-file-setup)
1. [Deploy EXR Container](/service-nodes/exr/#deploy-exr-container)
1. [SPV Wallets Setup](/service-nodes/exr/#spv-wallets-setup)
1. [SPV RPC Type Plugins Setup](/service-nodes/exr/#spv-rpc-type-plugins-setup)
1. [URL Type Plugins Setup](/service-nodes/exr/#url-type-plugins-setup)
1. [Domain Name Setup](/service-nodes/exr/#domain-name-setup)

---

## EXR Configuration File Setup
A `uwsgi.ini` configuration file is required to hook up the services and oracles that need to be exposed through the proxy.

Example `uwsgi.ini`:
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

1. Create a new directory and a `uwsgi.ini` file. This guide will use `/xrproxy/uwsgi.ini`.
1. Adjust `processes` and `threads` related to your setup.
1. Change `SERVICENODE_PRIVKEY=` to your servicenode privkey.
   1. On your Service Node client type `servicenodestatus` and retrieve your `snodeprivkey`.
1. Change `RPC_BLOCK_...=` HOSTIP/PORT/USER/PASS to your settings in your service node `blocknet.conf`
1. Save and close the `uwsgi.ini` file.
1. Anytime you edit the `uwsgi.ini` file, you need to restart the container. (or just `service nginx restart` within the container??)

---

## Service Node Configuration File Setup

1. To use the XRouter Proxy you have to use a port that is not `port=41412`.
	1. Set `port=80` or any other port you have designated Eg. `port=9090`.

1. Set `host=` to the server IP or server domain name.
	1. `host=10.0.0.1` or `host=api.domainnamehere.com`.

1. Save and close the file.

1. In your servicenode client type `xrReloadConfigs` for the settings to take.
	1. Send the new information out to the network by using `servicenodesendping`.

---

## Deploy EXR Container

1. In a console where Docker is available type: `docker pull blocknetdx/xrouterproxy:latest`.

1. Running the xrproxy in a container:
	1. `docker run -d --name xrproxy -p 80:80 -p 443:443 -v=/xrproxy:/opt/uwsgi/conf`
		1. `--name xrproxy` names the container 'xrproxy'
		1. `-p 80:80` exposes the container to port 80. Use this if you set `port=80` in your `xrouter.conf`
			1. Use `-p 9090:80` if you set `port=9090` in your `xrouter.conf`
		1. Optional: Add `-p 443:443` if you wish to provide a HTTPS API end point.
			1. Note: This proxy does not support 443 through the Blocknet client.
		1. `-v=/xrproxy:/opt/uwsgi/conf` connects a host volume to the xrproxy container.
			1. `-v=/dir/to/conf:/opt/uwsgi/conf` change `/dir/to/conf` to where the `uwsgi.ini` file is saved locally

1. To access inside the container type: `docker exec -it xrproxy /bin/bash`.

1. To access xrproxy container logs type: `docker logs xrproxy`.

1. Test to see if the XRProxy is working:
	1. `curl -X POST -H "content-type: application/json" --data-binary '[]' localhost:9090/xr/BLOCK/xrGetBlockCount`
		1. Adjust `localhost:9090` to your settings
			1. If you used `port=80` you can just use your IP. `10.0.0.1/xr/BLOCK/xrGetBlockCount` or `api.domainnamehere.com/xr/BLOCK/xrGetBlockCount`


---

## SPV Wallets Setup

1. Any additional SPV wallets are added based on this SPV template. Use the asset's ticker for each additional SPV wallet you're adding.
	1. `set-ph = RPC_WALLET_...=` for HOSTIP, PORT, USER, PASS, VER

	Litecoin example:

	```
	set-ph = RPC_LTC_HOSTIP=127.0.0.1
	set-ph = RPC_LTC_PORT=9332
	set-ph = RPC_LTC_USER=litecoin
	set-ph = RPC_LTC_PASS=litecoin1234
	set-ph = RPC_LTC_VER=2.0
	```

1. Add each asset's ticker in the `wallets=` section in your `xrouter.conf`. 
	1. For Litecoin add `LTC` in your `xrouter.conf`. 
		1. `wallets=BLOCK,LTC`

1. In your servicenode client type `xrReloadConfigs` for the settings to take.
	1. Send the new information out to the network by using `servicenodesendping`.


---

## SPV RPC Type Plugins Setup

1. SPV RPC plugins are setup similar to SPV Wallets, but you state which RPC call to invoke with `set-ph RPC_..._METHOD=`.

	1. SPV RPC plugin for Litecoin's `getbestblockhash` example:

		1. `uwsgi.ini` entry:

		```
		# Litecoin RPC SPV Services
		## getbestblockhash
		set-ph = RPC_ltc_getbestblockhash_HOSTIP=127.0.0.1
		set-ph = RPC_ltc_getbestblockhash_PORT=9332
		set-ph = RPC_ltc_getbestblockhash_METHOD=getbestblockhash
		set-ph = RPC_ltc_getbestblockhash_USER=litecoin
		set-ph = RPC_ltc_getbestblockhash_PASS=litecoin1234
		set-ph = RPC_ltc_getbestblockhash_VER=2.0
		```

1. Add `ltc_getbestblockhash` to the `plugins=` area of your `xrouter.conf`
	1. You will need a `ltc_getbestblockhash.conf` entry for your Blocknet client to load the service.

1. In your servicenode client type `xrReloadConfigs` for the settings to take.
	1. Send the new information out to the network by using `servicenodesendping`.

1. Test to see if this XCloud RPC is working:
	1. `curl -X POST -H "content-type: application/json" --data-binary '[]' localhost:9090/xrs/ltc_getbestblockhash`


---

## URL Type Plugins Setup

1. You will need to setup a container or webserver for non-RPC based calls.

1. Currency Exchange plugin example:

	```
	set-ph = URL_CurrencyExchangeRate_HOSTIP=172.17.0.1
	set-ph = URL_CurrencyExchangeRate_PORT=9195
	```
	1. The webserver needs to be setup so that you can cURL `172.17.0.1:9195/xrs/CurrencyExchangeRate`

	1. Add `CurrencyExchangeRate` to the `plugins=` area of your `xrouter.conf`.
		
	1. You will need a `CurrencyExchangeRate.conf` entry for your Blocknet client to load the service:

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

1. In your servicenode client type `xrReloadConfigs` for the settings to take.
	1. Send the new information out to the network by using `servicenodesendping`.

1. Test to see if this XCloud plugin is working:
	1. `curl -X POST -H "content-type: application/json" --data-binary '["CAD","USD"]' localhost:9090/xrs/CurrencyExchangeRate`

---

## Domain Name Setup

1. Allow `port 80` through your firewall.

1. Register a domain name.
	1. Point the A record to your server IP (Note: this may take up to 24hrs for the change to happen).
		1. A record, @, Server IP address

1. Enter the container by typing: `docker exec -it xrproxy /bin/bash`.

1. Edit the `nginx.conf` file located at `/etc/nginx/nginx.conf`.
	1. Find the `server` block in the `nginx.conf`.
	1. Adjust `nginx.conf` to point to your domain name:

	```
	server {
		listen 80;
		server_name api.domainnamehere.com;
	...	
	}
	```

1. For the settings to take, while you're inside the container, restart nginx by typing: `service nginx restart`.
	1. You should see a successfully restarted service response.

1. Adjust your `xrouter.conf` `host=` to point to the domain name and set `port=80` if you haven't already done so.

1. In your servicenode client type `xrReloadConfigs` for the settings to take.
	1. Send the new information out to the network by using `servicenodesendping`.



