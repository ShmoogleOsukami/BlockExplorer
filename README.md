An open-source block explorer written in node.js.

Tested on Ubuntu Xenial/Bionic.

### Requires

*  node.js >= 0.10.28
*  npm >= 1.4.9
*  mongodb >= 2.6.x
*  gridcoinresearchd

Installing node.js/npm:

https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions

Installing mongodb:

https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

You can quickly install gridcoinresearchd, including the latest snapshot, using the handy Autonode script located here:

https://github.com/gridcoin-community/Autonode

### Wallet

The wallet must be running with the following flags:

    -daemon -txindex

Or set the following in your gridcoinresearch.conf file:

	daemon=1
	txindex=1

### Create Database

Enter MongoDB cli:

    $ mongo

Create database:

    use explorerdb

Create user with read/write access:

    db.createUser( { user: "gridcoin", pwd: "u9WdcVgMVqvHcDz7", roles: [ "readWrite" ] } )

### Get The Source

    git clone https://github.com/gridcoin-community/BlockExplorer.git explorer

### Install Node Modules

    cd explorer && sudo npm install --production
	
### Edit settings.json

	vi settings.json
	
Set the RPC ID and password for your gridcoinresearchd instance (found in your gridcoinresearch.conf file) on lines 46-47.

### Start Explorer

    sudo npm start

*Note: mongod must be running to start the explorer.*

The explorer defaults to cluster mode, forking an instance of its process to each CPU core. This results in increased performance and stability. Load balancing is automatic, and any instances that die will be restarted automatically. For testing/development a single instance can be launched with:

    node --stack-size=10000 bin/instance

To stop the cluster you can use:

    sudo npm stop

### Syncing Databases With The Blockchain

sync.js (located in scripts/) is used for updating the local databases. This script must be called from the explorer's root directory.

    Usage: node scripts/sync.js [database] [mode]

    database: (required)
    index [mode] Main index: coin info/stats, transactions & addresses
    market       Market data: summaries, orderbooks, trade history & chartdata

    mode: (required for index database only)
    update       Updates index from last sync to current block
    check        checks index for (and adds) any missing transactions/addresses
    reindex      Clears index then resyncs from genesis to current block

    notes:
    * 'current block' is the latest created block when script is executed.
    * The market database only supports (& defaults to) reindex mode.
    * If check mode finds missing data (ignoring new data since last sync),
      index_timeout in settings.json is set too low.


*It is recommended to have this script launched via a cronjob at 1+ min intervals.*

**crontab**

*Example crontab; update index every minute and market data every 2 minutes*

    */1 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1
    */2 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js market > /dev/null 2>&1
    */5 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1


### Known Issues

**script is already running.**

If you receive this message when launching the sync script either a) a sync is currently in progress, or b) a previous sync was killed before it completed. If you are certain a sync is not in progress, remove the index.pid from the tmp folder in the explorer root directory.

    rm tmp/index.pid

**exceeding stack size**

    RangeError: Maximum call stack size exceeded

Node's default stack size may be too small to index addresses with many tx's. If you experience the above error while running sync.js, the stack size needs to be increased.

To determine the default setting run:

    node --v8-options | grep -B0 -A1 stack_size

To run sync.js with a larger stack size launch with:

    node --stack-size=[SIZE] scripts/sync.js index update

Where [SIZE] is an integer higher than the default.

### [License](LICENSE)
