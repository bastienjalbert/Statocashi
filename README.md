Statocashi: Bitcoin ABC Core + statistics logging
=====================================

What is Statocashi ?
----------------

Statocashi is a fork of JLOPP Statoshi (http://statoshi.info) adapted to work with
bitcoin cash. It use bitcoin abc core and most added code from statoshi.

https://github.com/jlopp/statoshi + https://github.com/Bitcoin-ABC/bitcoin-abc = Statocashi

Version
-------
16 August 2017 : First fork from Bitcoin ABC and Statoshi functions implementation.

19 August 2017 : Fix the mining chart issue (configuration problem). New features (like price) will be added soon, they are already available on the preview website.
                   

TODO : Need to fix 
- Recommanded Transaction Fee for Target Confirmation in X Blocks  

Preview 
-------
[https://statocashi.info/] : demonstration website.  


How to install ?
-------

**How does it work ?** 

Statocashi node -> statsd -> graphite <- grafana (web interface)
Legend : <-/-> = dataflux

### Dependencies :
> **Node.js:** Download node [http://nodejs.org/download/] and install node.js, on ubuntu just run ``` root@bash: sudo apt-get install nodejs npm ```.

> **Forever:** just run ``` root@bash: sudo npm install forever ```.

> **Statsd:** Download [https://github.com/etsy/statsd] (the github repo of statsd) and copy all files in a folder (like /opt/statsd/).

> **Graphite:** There are many way to install graphite. I used this one : [http://graphite.readthedocs.io/en/latest/install-pip.html]. If you had dependencies problems you can try this : [https://pastebin.com/raw/a9v5UkbK].

> **Apache2:** Debian/ubuntu based ``` root@bash: sudo apt-get install apache2 ```.

> **Memcached:** Debian/ubuntu based ``` root@bash: sudo apt-get install memcached ```.

> **Grafana:** Here too, this should be enough ``` root@bash: sudo apt-get install grafana ``` or check at [http://docs.grafana.org/installation/].

### Configuration :

**Statsd** configuration :
> ``` root@bash: sudo nano /opt/statsd/config.js ``` and paste the following configuration :
> ``` 
> {
>     graphitePort: 2003, 
>     graphiteHost: "127.0.0.1", 
>     port: 8125,
>     backends: [ "./backends/graphite" ]
> }  
> ``` 
> Save the file (ctrl + x on nano) and it should be ok !


**Graphite** configuration :
> ``` root@bash: sudo nano /opt/graphite/conf/storage-aggregation.conf ``` and paste the following configuration :
> ``` 
>[min]
>pattern = \.min$
>xFilesFactor = 0.0
>aggregationMethod = min
>
>[max]
>pattern = \.max$
>xFilesFactor = 0.0
>aggregationMethod = max
>
>[sum]
>pattern = \.count$
>xFilesFactor = 0.0
>aggregationMethod = sum
>
>[default_average]
>pattern = .*
>xFilesFactor = 0.0
>aggregationMethod = average
>  
> ``` 
> ctrl + x and then : ``` root@bash:  sudo nano /opt/graphite/conf/storage-schemas.conf ``` and paste this configuration :
> ``` 
>[carbon]
>pattern = ^carbon\.
>retentions = 10:2160,60:10080,600:262974
>
>[stats]
>priority = 110
>pattern = ^stats\..*
>retentions = 10:2160,60:10080,600:262974
>
>[default]
>pattern = .*
>retentions = 10:2160,60:10080,600:262974
> ``` 
> and again ctrl + x ... ! Now enable memcached on graphite : ``` root@bash: sudo nano /opt/graphite/webapp/graphite/local_settings.py ``` add in the file the following lines
> ``` 
>MEMCACHE_HOSTS = ['127.0.0.1:11211']
>DEFAULT_CACHE_DURATION = 600
> ``` 


### Statocashi node :

**Clone the github projet into a folder where you want to install the node :**
>``` root@bash: sudo git clone https://github.com/bastienjalbert/Statocashi.git ```

**Check dependencies** Take a loot at build notes : [https://github.com/bastienjalbert/Statocashi/blob/master/build-unix.md]

**Compile the project**
> ``` root@bash: cd /path/to/cloned/statocashi/repo ```
> ``` root@bash:/statocashi/repo/$ ./autogen.sh ```
> ``` root@bash:/statocashi/repo/$ ./configure.sh --disable-wallet ``` (--disable-wallet is not necessary, but I don't care on my node !)
> ``` root@bash:/statocashi/repo/$ make ```

Now be patient, it depend of your computer (15 mins ~ with 4 Cores i5, more than 40minutes with 2 Cores cpu).

**Create a bitcoin.conf file**

First create a file to notify mining info and txout info :
> ``` root@bash: nano /statocashi/where_you_want/notify.sh ``` then paste this :
> ```#!/bin/bash
>/statocashi/repo/src/bitcoin-cli getmininginfo 
>sleep 30
>/statocashi/repo/src/bitcoin-cli gettxoutsetinf
> ```

You can create the file like I do below, or if you had already downloaded the blockchain you can add the following conf lines to you current bitcoin.conf

> ``` root@bash: nano /statocashi/node_files/bitcoin.conf ``` then paste this :
>```
>rpcuser=unguessableUser3256
>rpcpassword=someRandomPasswordWithEnt
>server=1
>rpcbind=127.0.0.1
># this is my node IP  ;)
>addnode=163.172.219.62:8333 
># note that you may want to use blocknotify after the node be totally synced (all blocks downloaded) to avoid slow down when downloading the blockchain
>blocknotify=/statocashi/where_you_want/notify.sh
>```


### Run :

**Start statsd** continually (if crashed or exited)
> ``` root@bash: sudo forever start /opt/statsd/stats.js /opt/statsd/config.js ```

**Start graphite collector** 
> ``` root@bash: sudo /opt/graphite/bin/carbon-cache.py start ```

**Start graphite webapp** with the *8181* port
> ``` root@bash: sudo /opt/graphite/bin/run-graphite-devel-server.py --port 8181 /opt/graphite/ ```

**Start grafana**
> ``` root@bash: sudo service grafana-server start ```

**Start the Statocashi node** as a daemon 
> ``` root@bash: cd /path/to/cloned/statocashi/repo/src ```
> ``` root@bash: ./bitcoind -conf=/statocashi/node_files/bitcoin.conf -daemon ```

At this stage, you can access to your grafana node (http://ip_addr:3000) (default login admin/admin) and began to copy my Dashboard (from preview website) to your grafana. Don't forget to add the Datasource (graphite : localhost:8181) to your Grafana configuration file.

Notice that you may have to totally sync your node to see coherent data ...

If you get trouble dont hesitate to open an issue !

License
-------

Statocashi is released under the terms of the MIT license. See http://opensource.org/licenses/MIT for more information.

Valuable links :
----------------

http://docs.grafana.org/installation/configuration Some information about grafana configuration



