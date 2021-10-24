# Docker XCloud Services Tutorial

## Requirements:

* Operating Service Node on main-net (test-net if you want to test functionality)
* Docker installed
* Windows Sub Linux System if working on Windows (recommended)
* Notepad++ or other text editor capable of Unix EOL converting

---

## Common Docker Commands

```
docker ps                                  # list running containers
docker ps -a                               # list all containers
docker stop container_name                 # stop container
docker start container_name                # start container
docker run -d --name=container_name        # create and run new container in background
docker exec -it container_name /bin/bash   # login to container
docker rm container_name                   # delete stopped container
docker cp host_file_loc container:/.../	   # copy from host to container
```

---

## Running a XWallet Container

* Docker runs the XWallet securely in a non-GUI container

* Official Blocknet Docker Images: https://hub.docker.com/u/blocknetdx/

* Official Blocknet Docker Images GitHub: https://github.com/BlocknetDX/dockerimages

* There are different options on how to run the Docker XWallet container:

> Running a simple XWallet node
>
> Running a XWallet that persists the blockchain on a host directory
>
> Running a XWallet node with restart features


### Running a simple XWallet node

* For an example we will create a Syscoin Docker container

* Open a command prompt/terminal window

* Type: `docker run -d --name=syscoin -p 8369:8369 blocknetdx/syscoin3:3.1.3`

* This will pull the syscoin image slated version 3.1.3

* This container will run on port 8369 and be called 'syscoin'

* Type: `docker ps -a` to see that status of the container

* Type: `docker exec -it syscoin /bin/bash` to login to the container

* Once logged into the container you can type: `syscoin-cli <cmd>`

* Type: `exit` to return back to the command prompt/terminal


### Running a XWallet that persists the blockchain on a host directory

* For an example we will create a Syscoin Docker container using an existing data directory

* Find the location of your data directory
	* For Eg. C:\Users\Administrator\AppData\Roaming\SyscoinCore

* The docker container will utilize your existing data directory

* Open a command prompt/terminal window

* This is an example run command:

```
docker run -d --name=syscoin -p 8369:8369 -v=C:\crypto\syscoin\config:/opt/blockchain/config -v=C:\crypto\syscoin\data:/opt/blockchain/data blocknetdx/syscoin3:3.1.3
```

* Replace `C:\crypto\syscoin\config` with the location to your `syscoin.conf` file

* Replace `C:\crypto\syscoin\data` with the location to your Syscoin datadir

* For this example the command would be:

```
docker run -d --name=syscoin -p 8369:8369 -v=C:\Users\Administrator\AppData\Roaming\SyscoinCore:/opt/blockchain/config -v=C:\Users\Administrator\AppData\Roaming\SyscoinCore:/opt/blockchain/data blocknetdx/syscoin3:3.1.3
```

* This will pull the syscoin image slated version 3.1.3

* This container will run on port 8369 and be called 'syscoin'

* Type: `docker ps -a` to see that status of the container

* Type: `docker exec -it syscoin /bin/bash` to login to the container

* Once logged into the container you can type: `syscoin-cli <cmd>`

* Type: `exit` to return back to the command prompt/terminal


### Running a XWallet node with restart features (WIP)

* See: https://docs.docker.com/engine/admin/start-containers-automatically/

* `--restart=no|on-failure:retrycount|always|unless-stopped`

* Examples:

```
docker run -d --restart=no --name=syscoin -p 8369:8369 blocknetdx/syscoin:3.0.4 syscoind -daemon=0 -rpcuser=sys -rpcpassword=sys123
docker run -d --restart=on-failure:10 --name=syscoin -p 8369:8369 blocknetdx/syscoin:3.0.4 syscoind -daemon=0 -rpcuser=sys -rpcpassword=sys123
docker run -d --restart=unless-stopped --name=syscoin -p 8369:8369 blocknetdx/syscoin:3.0.4 syscoind -daemon=0 -rpcuser=sys -rpcpassword=sys123
docker run -d --restart=always --name=syscoin -p 8369:8369 blocknetdx/syscoin:3.0.4 syscoind -daemon=0 -rpcuser=sys -rpcpassword=sys123
```

---

## Configuring XWallet conf file

* For this example we will be using Syscoin

* Once the XWallet container is running RPC works like normal

* If you want to run the XWallet on local host you need to change `syscoin.conf` to allow Dockers local host IP

* Navigate to your `syscoin.conf`

* Replace `rpcallowip=127.0.0.1` with `rpcallowip=0.0.0.0/0`

* Example `syscoin.conf`:

```
server=1
listen=1
rpcuser=sys
rpcpassword=sys123
rpcallowip=0.0.0.0/0
port=8369
rpcport=8370
```

* In you `xbridge.conf` leave `Ip=127.0.0.1`

* Example `xbridge.conf`:

```
[SYS]
Title=Syscoin
Address=
Ip=127.0.0.1
Port=8370
Username=sys
Password=sys123
AddressPrefix=63
ScriptPrefix=5
SecretPrefix=128
COIN=100000000
MinimumAmount=0
TxVersion=1
DustAmount=0
CreateTxMethod=BTC
MinTxFee=0
BlockTime=60
GetNewKeySupported=false
ImportWithNoScanSupported=false
FeePerByte=210
Confirmations=0
TxWithTimeField=false
LockCoinsSupported=false
```

* Ensure Blocknet's XBridge is connects to the Syscoin container

---

* If you would like a more secure setting use these settings in your `syscoin.conf`:

```
rpcallowip=172.16.0.0/16   # syscoin container subnet
rpcallowip=192.168.1.0/24  # your lan subnet
```

* Lookup the Syscoin container subnet by typing: `docker inspect syscoin`

* Find the 'IPAddress'

* Login to your router to find your lan subnet

---

## Docker XCloud Service (Syscoin Example)

* This tutorial will create a single Syscoin Docker XCloud service

* Create a new `.conf` file. Use the naming factor `<coin_ticker><rpc_call>.conf` and place it in the 'plugins' folder in the Blocknet datadir

* For this example we will create a Docker XCloud service using Syscoin's RPC call: `listassetallocationtransactions`

---

* Create `SYSlistassetallocationtransactions.conf` and place it in the 'plugins' folder

* Edit the `.conf` with the following:

```
parameters=int,int,string
fee=0
clientrequestlimit=-1
disabled=0
fetchlimit=50

#private:: Equivalent to: docker exec -t syscoin syscoin-cli listassetallocationtransactions [count] [from] [{options}]
private::type=docker
private::containername=syscoin
private::quoteargs=1
private::command=
private::args=$1 $2 $3
```

* Save and close

* Navigate and edit you `xrouter.conf`

* Add the new plugin: `plugins=SYSlistassetallocationtransactions`

* Restart the Service Node



---

# XCloud API Services

## Install Ubuntu Docker Image

* https://hub.docker.com/_/ubuntu/

* Type `docker pull ubuntu`

* Docker will get the necessary files

* Once the files are downloaded type `docker run -it ubuntu`

* It will output the Docker Ubuntu Container ID# once the container is running

* Type `docker ps -a` to view the status of all containers

---

## Install Ubuntu updates, curl, jq

* Type `docker exec <docker_container_id#> apt-get update`

* Type `docker exec <docker_container_id#> apt-get install -y curl wget`

* Type `docker exec <docker_container_id#> apt-get install -y jq`

---

## Linux Bash Script (.sh) Creation

* We will use an easy XCloud Services example, Global Stock pricing info.

* Sign up for a free or paid API with any given provider. Get familiar with the providers API documentation.

* Most API providers require authorization via a API key. This is sent to you when you sign up. Depending on the level of membership you may have max API call limits and/or rate limiting. If you exceed the call limit there is a chance the API provider will ban your API key for excessive use.

* Open text editor (Notepad++) and save the file. For this example we will call it `GlobalStock_Latest.sh`

* You need to ensure the file is converted to Unix EOL. 
	* In Notepad++ click Edit > EOL Conversion > Unix (LF)

* Start coding your bash script. To call the API we will be using `curl`.

* Below is an example bash script:

```
#! /bin/sh

symbol=$1
apikey=<enter_your_api_key_here>

curl -s "https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=${symbol}&apikey=${apikey}"
```

* This script when ran will accept 1 user parameter. To run this script, go to a Linux environment and type: `bash GlobalStock_Latest.sh AMZN`. This will make the API call and output the response.

* If you are happy with the reponse head to the next section to copy this script to the Docker container.

---

## Copy Files from Host to Docker Container

* Copy the bash script or file on the host computer

* Here is an example to copy this file from the host computer to the Docker container. Type `docker cp C:\Scripts\GlobalStock_Latest.sh <docker_container_id#>:/bin/`. You can place these files anywhere in the container, doesn't have to be `/bin/`.

* You should test that the script works in Docker. Type `docker exec -it <docker_container_id#> /bin/bash GlobalStock_Latest.sh AMZN`

* If the output is correct head to the next step.

---

## Plugin .conf Creation

* Create a new .conf and name it what you want to call your service. For example: `GlobalStockLatestInfo.conf`

* Edit the file with this template:

```
parameters=
fee=
clientrequestlimit=
disabled=
help=

#private:: Equivalent to: docker exec -t <docker_container_id#> /bin/bash Script.sh [parameter1]
private::type=docker
private::containername=<docker_container_id#>
private::quoteargs=1
private::command=/bin/bash Script.sh
private::args=$1 $2 $3 10
```

* Here is how this Global Stock example should be configured:

```
parameters=string
fee=0
clientrequestlimit=12000
disabled=0
help=Outputs latest price and volume information for any given stock; GlobalStockLatestInfo [symbol]; Example: xrService xrs::GlobalStockLatestInfo AAPL

#private:: Equivalent to: docker exec -t <docker_container_id#> /bin/bash GlobalStock_Latest.sh [symbol]
private::type=docker
private::containername=<docker_container_id#>
private::quoteargs=1
private::command=/bin/bash GlobalStock_Latest.sh
private::args=$1
```

* Choose a `fee=` based on what you want to charge, `fee=0` is free.

* Choose a `clientrequestlimit=` (in milliseconds) that a single request is time limited to.

* Type up a help area to help guide users with parameters. Users can see this by calling `xrShowConfigs`.

* Now add this service (GlobalStockLatestInfo) or any service in your Service Node's `xrouter.conf`

---

## Filtering API Responses w/ 'jq'

* WIP

* https://programminghistorian.org/en/lessons/json-and-jq#create-new-json--and-

---

## Script Output Format

* Running these scripts through Blocknet's console and CLI output require the correct formatting or it won't display properly.

* The script response needs to be structured in JSON format

* If the script response is not structured in JSON format it will contain lots of "/n" and look all cluttered.

* If the script response is in JSON format, but contain's multiple arrays "[ ]", it tends to only output the first array.

* Sometimes you have to use `sed` and `tr` to remove/add commas and brackets to force the correct JSON structure.

* The format to aim for is:

```
[
	{
		"ticker": BTC
	},
	{
		"price": 0.001
	},
	{
		"percent_change": 46
	}
]
```
