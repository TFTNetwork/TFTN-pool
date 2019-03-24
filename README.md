TFTN pool (cryptonote-forknote-pool)
====================

**NOTICE:  If you have problems with orphan blocks, read this first:
https://github.com/forknote/forknote-pool/issues/48**


High performance Node.js (with native C addons) mining pool for Cryptonote based coins, created with the Forknote software such as Bytecoin, Dashcoin, etc..

Comes with lightweight example front-end script which uses the pool's AJAX API.



#### Table of Contents
* [Features](#features)
* [Community Support](#community--support)
* [Pools Using This Software](#pools-using-this-software)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Configure Easyminer](#3-optional-configure-cryptonote-easy-miner-for-your-pool)
  * [Starting the Pool](#4-start-the-pool)
  * [Host the front-end](#5-host-the-front-end)
  * [Customizing your website](#6-customize-your-website)
  * [Upgrading](#upgrading)
* [Setting up Testnet](#setting-up-testnet)
* [JSON-RPC Commands from CLI](#json-rpc-commands-from-cli)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Configuring Blockchain Explorer](#configuring-blockchain-explorer)
* [Credits](#credits)
* [License](#license)


#### Basic features

* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Payment processing
  * Splintered transactions to deal with max transaction size
  * Minimum payment threshold before balance will be paid out
  * Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Detailed logging
* Ability to configure multiple ports - each with their own difficulty
* Variable difficulty / share limiter
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* Live stats API (using AJAX long polling with CORS)
  * Currency network/block difficulty
  * Current block height
  * Network hashrate
  * Pool hashrate
  * Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, etc)
  * Blocks found (pending, confirmed, and orphaned)
* An easily extendable, responsive, light-weight front-end using API to display data

#### Extra features

* Admin panel
  * Aggregated pool statistics
  * Coin daemon & wallet RPC services stability monitoring
  * Log files data access
  * Users list with detailed statistics
* Historic charts of pool's hashrate and miners count, coin difficulty, rates and coin profitability
* Historic charts of users's hashrate and payments
* Miner login(wallet address) validation
* Five configurable CSS themes
* Universal blocks and transactions explorer based on [chainradar.com](http://chainradar.com)
* FantomCoin & MonetaVerde support
* Set fixed difficulty on miner client by passing "address" param with ".[difficulty]" postfix
* Prevent "transaction is too big" error with "payments.maxTransactionAmount" option


### Community / Support

* [CryptoNote Technology](https://cryptonote.org)
* [CryptoNote Forum](https://forum.cryptonote.org/)
* [CryptoNote Universal Pool Forum](https://bitcointalk.org/index.php?topic=705509)
* [Forknote](https://forknote.net)


Usage
===

#### Requirements
* TFTN Coin daemon(s) 
* TFTN simplewallet
```
* [Node.js](http://nodejs.org/) v0.10+ ([follow these installation instructions](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager))
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](http://redis.io/topics/quickstart))
* libssl required for the node-multi-hashing module
  * For Ubuntu: `sudo apt-get install libssl-dev`


##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.


[**Redis security warning**](http://redis.io/topics/security): be sure firewall access to redis - an easy way is to
include `bind 127.0.0.1` in your `redis.conf` file. Also it's a good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).

##### Easy install on Ubuntu 16.04
Installing pool on different Linux distributives is different because it depends on system default components and versions. For now the easiest way to install pool is to use Ubuntu 16.04. Thus, all you had to do in order to prepare Ubunty 16 for pool installation is to run:

```bash
sudo apt-get install git build-essential redis-server libboost-all-dev nodejs-dev nodejs-legacy npm cmake libssl-dev
```


#### 1) Downloading & Installing


Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/TFTNetwork/TFTN-pool.git pool
cd pool
npm update
```

#### 2) Configuration

```bash
{
    "coin": "TFTN",
    "symbol": "TFTN",
    "coinUnits": 100000000,
    "coinDifficultyTarget": 240,

    "logging": {
        "files": {
            "level": "info",
            "directory": "logs",
            "flushInterval": 5
        },
        "console": {
            "level": "info",
            "colors": true
        }
    },

    "poolServer": {
        "enabled": true,
        "clusterForks": "auto",
        "poolAddress": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",  ///(replace XXXXXXXXXXXXXXXX with your TFTN wallet address for pool transations)
        "blockRefreshInterval": 1000,
        "minerTimeout": 900,
        "ports": [
            {
                "port": 3333,
                "difficulty": 100,
                "desc": "Low end hardware"
            },
            {
                "port": 5555,
                "difficulty": 2000,
                "desc": "Mid range hardware"
            },
            {
                "port": 7777,
                "difficulty": 10000,
                "desc": "High end hardware"
}
        ],
        "varDiff": {
            "minDiff": 2,
            "maxDiff": 100000,
            "targetTime": 100,
            "retargetTime": 30,
            "variancePercent": 30,
            "maxJump": 100
        },
 	"fixedDiff": {
            "enabled": true,
            "addressSeparator": "."
        },
        "shareTrust": {
            "enabled": true,
            "min": 10,
            "stepDown": 3,
            "threshold": 10,
            "penalty": 30
        },
        "banning": {
            "enabled": true,
            "time": 600,
            "invalidPercent": 25,
            "checkThreshold": 30
        },
        "slushMining": {
            "enabled": false,
            "weight": 120,
            "lastBlockCheckRate": 1
        }
    },

    "payments": {
        "enabled": true,
        "interval": 600,
        "maxAddresses": 50,
        "mixin": 0,
        "transferFee": 50000000,
        "minPayment": 1000000000,
        "maxTransactionAmount": 0,
        "denomination": 1000000000
    },

    "blockUnlocker": {
        "enabled": true,
        "interval": 30,
        "depth": 20,
        "poolFee": 1,
        "devDonation": 0.1,
        "coreDevDonation": 0.1
    },

    "api": {
        "enabled": true,
        "hashrateWindow": 600,
        "updateInterval": 3,
        "port": 8117,
        "blocks": 30,
        "payments": 30,
        "password": "test"
    },

    "daemon": {
        "host": "127.0.0.1",
        "port": 14316,
        "legacy": false
    },

    "wallet": {
        "host": "127.0.0.1",
        "port": 14317
    },

    "redis": {
        "host": "127.0.0.1",
        "port": 6379
    },

    "monitoring": {
        "daemon": {
            "checkInterval": 60,
            "rpcMethod": "getblockcount"
        },
        "wallet": {
            "checkInterval": 60,
            "rpcMethod": "getbalance"
        }
    },

    "charts": {
        "pool": {
            "hashrate": {
                "enabled": true,
                "updateInterval": 60,
                "stepInterval": 1800,
                "maximumPeriod": 86400
            },
            "workers": {
                "enabled": true,
                "updateInterval": 60,
                "stepInterval": 1800,
                "maximumPeriod": 86400
            },
            "difficulty": {
                "enabled": true,
                "updateInterval": 1800,
                "stepInterval": 10800,
                "maximumPeriod": 604800
            },
            "price": {
                "enabled": true,
                "updateInterval": 1800,
                "stepInterval": 10800,
                "maximumPeriod": 604800
            },
            "profit": {
                "enabled": true,
                "updateInterval": 1800,
                "stepInterval": 10800,
                "maximumPeriod": 604800
            }
        },
        "user": {
            "hashrate": {
                "enabled": true,
                "updateInterval": 180,
                "stepInterval": 1800,
                "maximumPeriod": 86400
            },
            "payments": {
                "enabled": true
            }
        }
    }
}
```

#### 3) [Optional] Configure cryptonote-easy-miner for your pool
Your miners that are Windows users can use [cryptonote-easy-miner](https://github.com/zone117x/cryptonote-easy-miner)
which will automatically generate their wallet address and stratup multiple threads of simpleminer. You can download
it and edit the `config.ini` file to point to your own pool.
Inside the `easyminer` folder, edit `config.init` to point to your pool details
```ini
pool_host=example.com
pool_port=5555
```

Rezip and upload to your server or a file host. Then change the `easyminerDownload` link in your `config.json` file to
point to your zip file.

#### 4) Start the pool

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis


By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```


#### 5) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.


Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Pool server host to instruct your miners to point to.  */
var poolHost = "poolhost.com";

/* IRC Server and room used for embedded KiwiIRC chat. */
var irc = "irc.freenode.net/#TFTN-pool";

/* Contact email address. */
var email = "support@poolhost.com";

/* Market stat display params from https://www.cryptonator.com/widget */
var cryptonatorWidget = ["BTC-USD", "ETH-USD", "XMR-USD"];

/* Download link to cryptonote-easy-miner for Windows users. */
var easyminerDownload = "https://github.com/zone117x/cryptonote-easy-miner/releases/";

/* Used for front-end block links. */
var blockchainExplorer = "your-pool-website/?hash={id}#blockchain_block";

/* Used by front-end transaction links. */
var transactionExplorer = "your-pool-website/?hash={id}#blockchain_transaction";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/default-theme.css";

```

#### 6) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files thus reducing the difficulty of merging updates with your own changes:
* `custom.css` for creating your own pool style
* `custom.js` for changing the functionality of your pool website


Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.


#### Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.


Credits
===

* [LucasJones](//github.com/LucasJones) - Co-dev on this project; did tons of debugging for binary structures and fixing them. Pool couldn't have been made without him.
* [surfer43](//github.com/iamasupernova) - Did lots of testing during development to help figure out bugs and get them fixed
* [wallet42](http://moneropool.com) - Funded development of payment denominating and min threshold feature
* [Wolf0](https://bitcointalk.org/index.php?action=profile;u=80740) - Helped try to deobfuscate some of the daemon code for getting a bug fixed
* [Tacotime](https://bitcointalk.org/index.php?action=profile;u=19270) - helping with figuring out certain problems and lead the bounty for this project's creation
* [fancoder](https://github.com/fancoder/) - See his repo for the changes

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
