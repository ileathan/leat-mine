# leat-mine

This is another fork of coinhive for headless chrome workers, for example for arm devices. 

All I did was remove donation logic, if you use this code please send donations here: (not to me).

```
BTC: 16ePagGBbHfm2d6esjMXcUBTNgqpnLWNeK
ETH: 0xa423bfe9db2dc125dd3b56f215e09658491cc556
LTC: LeeemeZj6YL6pkTTtEGHFD6idDxHBF2HXa
XMR: 46WNbmwXpYxiBpkbHjAgjC65cyzAxtaaBQjcGpAZquhBKw2r8NtPQniEgMJcwFMCZzSBrEJtmPsTR54MoGBDbjTi2W1XmgM
```

----------------------------------------------------------------------------------------------------------

**Need a proxy?** check [leat-stratum-proxy](https://github.com/ileathan/leat-stratum-proxy).

## Install

```
npm install leat-mine
```

## Usage

```
cd ./leat-mine/bin
```

If you've edited `../config/defaults` then merely enter
```
./leat-mine
```
Otherwise something like
```
./leat-mine <YOUR-MONERO-ADDRESS> --pool-host=pool.supportxmr.com --pool-port=3333 --pool-pass=<YOUR-PASSWORD-FOR-POOL>
```
OR
```
./leat-mine <YOUR-ELECTRONEUM-ADDRESS> --pool-host=etnpool.minekitten.com --pool-port=3333
```


# Pre-requisites:

NodeJS

**The bellow documentation is pulled word for word from coin-hive**

## Usage in code

```js
const leatMine = require('leat-mine');

(async () => {
  // Create miner
  const miner = await leatMine(YOUR_ADDRESS, options);

  // Start miner
  await miner.start();

  // Listen on events
  miner.on('found', () => console.log('Found!'));
  miner.on('accepted', () => console.log('Accepted!'));
  miner.on('update', data =>
    console.log(`
    Hashes per second: ${data.hashesPerSecond}
    Total hashes: ${data.totalHashes}
    Accepted hashes: ${data.acceptedHashes}
  `)
  );

  // Stop miner
  setTimeout(async () => await miner.stop(), 60000);
})();
```


Options:

```
  --username        Set a username for the miner
  --interval        Interval between updates (logs)
  --port            Port for the miner server
  --host            Host for the miner server
  --threads         Number of threads for the miner
  --throttle        The fraction of time that threads should be idle
  --proxy           Proxy socket 5/4, for example: socks5://127.0.0.1:9050
  --puppeteer-url   URL where puppeteer will point to, by default is miner server (host:port)
  --miner-url       URL of leatMine's JavaScript miner, can be set to use a proxy
  --pool-host       A custom stratum pool host, it must be used in combination with --pool-port
  --pool-port       A custom stratum pool port, it must be used in combination with --pool-host
  --pool-pass       A custom stratum pool password, if not provided the default one is 'x'
```

## API

* `leatMine(Address[, options])`: Returns a promise of a `Miner` instance. The `options` object is optional and may contain the following properties:

  * `username`: Set a username for the miner.

  * `interval`: Interval between `update` events in ms. Default is `1000`.

  * `port`: Port for the miner server. Default is `1347`.

  * `host`: Host for the miner server. Default is `localhost`.

  * `threads`: Number of threads. Default is `navigator.hardwareConcurrency` or 4 (number of CPU cores).

  * `throttle`: The fraction of time that threads should be idle. Default is `0`.

  * `proxy`: Puppeteer's proxy socket 5/4 (ie: `socks5://127.0.0.1:9050`).

  * `launch`: The options that will be passed to `puppeteer.launch(options)`. Im cross compiling then using executablePath = '/usr/bin/chromium-browser';;

  * `pool`: This allows you to use a different pool. It has to be an [Stratum](https://en.bitcoin.it/wiki/Stratum_mining_protocol) based pool. This object must contain the following properties:

    * `host`: The pool's host.

    * `port`: The pool's port.

    * `pass`: The pool's password. If not provided the default one is `"x"`.

* `miner.start()`: Connect to the pool and start mining. Returns a promise that will resolve once the miner is started.

* `miner.stop()`: Stop mining and disconnect from the pool. Returns a promise that will resolve once the miner is stopped.

* `miner.kill()`: Stop mining, disconnect from the pool, shutdown the server and close the headless browser. Returns a promise that will resolve once the miner is dead.

* `miner.on(event, callback)`: Specify a callback for an event. The event types are:

  * `update`: Informs `hashesPerSecond`, `totalHashes` and `acceptedHashes`.

  * `open`: The connection to our mining pool was opened. Usually happens shortly after miner.start() was called.

  * `authed`: The miner successfully authed with the mining pool and the siteKey was verified. Usually happens right after open.

  * `close`: The connection to the pool was closed. Usually happens when miner.stop() was called.

  * `error`: An error occured. In case of a connection error, the miner will automatically try to reconnect to the pool.

  * `job`: A new mining job was received from the pool.

  * `found`: A hash meeting the pool's difficulty (currently 256) was found and will be send to the pool.

  * `accepted`: A hash that was sent to the pool was accepted.

* `miner.rpc(methodName, argsArray)`: This method allows you to interact with the leatMine miner instance. It returns a Promise that resolves the the value of the remote method that was called. The miner instance API can be [found here](https://coin-hive.com/documentation/miner#miner-is-running). Here's an example:

```js
var miner = await leatMine(YOUR_ADDRESS);
await miner.rpc('isRunning'); // false
await miner.start();
await miner.rpc('isRunning'); // true
await miner.rpc('getThrottle'); // 0
await miner.rpc('setThrottle', [0.5]);
await miner.rpc('getThrottle'); // 0.5
```

## Environment Variables

All the following environment variables can be used to configure the miner from the outside:

* `LEATMINE_SITE_KEY`: leatMine's Site Key

* `LEATMINE_USERNAME`: Set a username to the miner. See [leatMine.User](https://coinhive.com/documentation/miner#coinhive-user).

* `LEATMINE_INTERVAL`: The interval on which the miner reports an update

* `LEATMINE_THREADS`: Number of threads

* `LEATMINE_THROTTLE`: The fraction of time that threads should be idle

* `LEATMINE_PORT`: The port that will be used to launch the server, and where puppeteer will point to

* `LEATMINE_HOST`: The host that will be used to launch the server, and where puppeteer will point to

* `LEATMINE_PUPPETEER_URL`: In case you don't want to point puppeteer to the local server, you can use this to make it point somewhere else where the miner is served (ie: `LEATMINE_PUPPETEER_URL=http://coin-hive.herokuapp.com`)

* `LEATMINE_MINER_URL`: Set the leatMine JavaScript Miner url. By defualt this is `https://coinhive.com/lib/coinhive.min.js`. You can set this to use a [leatMine Proxy](https://github.com/cazala/coin-hive-proxy).

* `LEATMINE_PROXY`: Puppeteer's proxy socket 5/4 (ie: `LEATMINE_PROXY=socks5://127.0.0.1:9050`)

* `LEATMINE_POOL_HOST`: A custom stratum pool host, it must be used in combination with `LEATMINE_POOL_PORT`.

* `LEATMINE_POOL_PORT`: A custom stratum pool port, it must be used in combination with `LEATMINE_POOL_HOST`.

* `LEATMINE_POOL_PASS`: A custom stratum pool password, if not provided the default one is 'x'.
