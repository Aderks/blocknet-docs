# DX Maker Bot
A market making bot for Blocknet's decentralized exchange protocol, built with the [XBridge API](https://api.blocknet.co).



## Prerequisites
1. [Latest Blocknet wallet installed](https://github.com/BlocknetDX/blocknet/releases/latest).
1. The wallet of any assets you will be trading. See list of [compatible assets](https://docs.blocknet.co/protocol/xbridge/compatibility/#supported-digital-assets).
1. The Blocknet wallet and any other wallet you're trading out of must but fully synced and fully unlocked.
1. The wallets used for trading must be configured. For simple setup, use [Block DX's automated configuration setup wizard](https://docs.blocknet.co/blockdx/configuration/). Having Block DX installed and opened is also useful to visually monitor the market and your open orders.
1. Funds are split into multiple UTXOs. If you have an order for 1 LTC and you only have a single 10 LTC input, all 10 LTC will be locked in this order. Having multiple, preferrably smaller, UTXOs will allow a better distribution of funds across orders.
1. Make sure funds are in legacy addresses (Eg. LTC funds should be in a "L" address).



## Installation

#### Linux
1. Install Python 3: ```apt-get install python3```
	* Or update Python 3: ```apt-get upgrade python3```
1. Install pip:```apt-get install python3-pip```
	* Or update pip: ```apt-get upgrade python3-pip```
1. Navigate to your project directory.
1. Download DX Maket Bot: ```git clone https://github.com/BlocknetDX/dxmakerbot```
1. Navigate into *dxmakerbot*: ```cd dxmakerbot```
1. Install required DX Maket Bot packages: ```pip3 install -r requirements.txt```

#### Windows
1. Install Python 3:
	1. Navigate to [https://www.python.org/downloads/](https://www.python.org/downloads/) and select *Download Python 3.7.x*.
	1. Run the installer.
	1. Check off *Add Python 3.7 to PATH*.
	1. Click *Install Now*.
1. Download DX Maker Bot:
	1. Navigate to [https://github.com/BlocknetDX/dxmakerbot](https://github.com/BlocknetDX/dxmakerbot).
	1. Click the green *Clone or download* button and select *Download ZIP* from the dropdown.
	1. Save the file and extract the contents to a folder.
1. Right-click the taskbar *Start* menu and select *Command Prompt (Admin)*.
1. Update pip: ```py -m pip install --upgrade pip```
1. Navigate into *dxmakerbot*: ```cd C:\Users\%USERNAME%\Downloads\dxmakerbot```
1. Install required DX Maket Bot packages: ```pip3 install -r requirements.txt```



## Configuration
1. Open `dxmaketbot/utils/dxsettings.py`.
1. Edit the trading addresses to match the wallet addresses containing funds split into multiple UTXOs.
	* Make sure funds are in legacy addresses (Eg. LTC funds should be in a "L" address).
1. Edit `rpcuser =`, `rpcpassword =`, and `rpcport =` to the same values used in the Blocknet client's `blocknetdx.conf` file.
1. Save and close the file. 

Example `dxsettings.py` file:

```
tradingaddress = {}
tradingaddress['SYS'] = "SbaRDo3taq1zmAba2Ak54fRxAz8Z9rkyPw"
tradingaddress['LTC'] = "LZdTPLFqQzxuoSfyRioUSMyPvlsrnpGVWR"
tradingaddress['BLOCK'] = "B2cBqpuEXu4tVS5prBd5zyeqvBTSi6g398"
tradingaddress['DGB'] = "D3F4pdn7CGZ4kxFUvdMbflyJ8cRqfwsurj"
rpcport = 41414
rpcuser = 'blocknetbot'
rpcpassword = 'blocknetbot123'
```



## Supported Arguments

* `--maker` = asset being sold (default='BLOCK')
* `--taker` = asset being bought (default='LTC')
* `--sellmin` = minimum quantity to sell per order (default=0.001)
* `--sellmax` = maximum quantity to sell per order (default=1)
* `--slidemin` = minimum order price multipler (market price * slidemin = min price) (default=1.000001)
* `--slidemax` = maximum order price multipler (marker price * slidemax = max price) (default=1.019999)
* `--cancelall` = cancels all open orders
?* `--cancelmarket` = cancel all orders in a given market
?* `--delay` = sleep delay value (default=3)
* `--maxloop` = maximum number of order loops (default=7)
* `--maxopen` = maximum number of open orders (default=5)
?* `--minbalance` = minimum balance needed for the loop (default=10)
* `--usecb` = enables CryptoBridge pricing (without this argument Bittrex pricing will be used)



## Running the Bot

1. Run the wallets of any assets being traded (fully synced, unlocked).
1. Run the Blocknet wallet (fully synced, unlocked).
1. *Optional*: Run [Block DX](https://docs.blocknet.co/blockdx/setup) for visual reference that the bot is working.
	* At this stage it would be a good idea to test making/taking an order without using the bot to ensure everything is setup properly
1. Navigate to the *dxmakerbot* directory in the terminal.

### Maker Bot
* At the moment this bot only supports pricing from Bittrex and CryptoBridge markets for Blocknet [compatible assets](https://docs.blocknet.co/protocol/xbridge/compatibility/#supported-digital-assets).
* Add any of the supported arguments to customize the dxmakerbot

Use the following basic command format to start the bot:

```
dxmakerbot.py --maker [] --taker [] --sellmin [] --sellmax [] --slidemin [] --slidemax []
```

Example command:

```
dxmakerbot.py --maker SYS --taker LTC --sellmin 5 --sellmax 15 --slidemin 1.00111 --slidemax 1.1111
```

If Bittrex does not have available markets to retrieve pricing information from, use argument '--usecb' to retrieve pricing information from CryptoBridge markets.

Example command:

```
dxmakerbot.py --maker LTC --taker BTC --sellmin 1 --sellmax 5 --slidemin 1.00111 --slidemax 1.1111 --usecb
````

### Cancel Open Orders

If you would like to cancel all open orders, use argument '--cancelall'.

Example command:

```
dxmakerbot.py --cancellall
```

If you would like to cancel all open orders in any given market, use argument '--cancelmarket'

Example command:

```
dxmakerbot.py --cancelmarket [?]
```



### Taker Bot

*Guide coming...*
