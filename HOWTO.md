How to run your own LTCLectrum server
===================================

Abstract
--------

This document is an easy to follow guide to installing and running your own
LTCLectrum server on Linux. It is structured as a series of steps you need to
follow, ordered in the most logical way. The next two sections describe some
conventions we use in this document and hardware, software and expertise
requirements.

The most up-to date version of this document is available at:

    https://github.com/wregab/ltclectrum-server/blob/ltcmaster/HOWTO.md

This document is derived from:
    https://github.com/spesmilo/electrum-server/blob/master/HOWTO.md

Conventions
-----------

In this document, lines starting with a hash sign (#) or a dollar sign ($)
contain commands. Commands starting with a hash should be run as root,
commands starting with a dollar should be run as a normal user (in this
document, we assume that user is called 'litecoin'). We also assume the
litecoin user has sudo rights, so we use '$ sudo command' when we need to.

Strings that are surrounded by "lower than" and "greater than" ( < and > )
should be replaced by the user with something appropriate. For example,
<password> should be replaced by a user chosen password. Do not confuse this
notation with shell redirection ('command < file' or 'command > file')!

Lines that lack hash or dollar signs are pastes from config files. They
should be copied verbatim or adapted, without the indentation tab.

apt-get install commands are suggestions for required dependencies.
They conform to an Ubuntu 13.04 system but may well work with Debian
or earlier and later versions of Ubuntu.

Prerequisites
-------------

**Expertise.** You should be familiar with Linux command line and
standard Linux commands. You should have basic understanding of git,
Python packages. You should have knowledge about how to install and
configure software on your Linux distribution. You should be able to
add commands to your distribution's startup scripts. If one of the
commands included in this document is not available or does not
perform the operation described here, you are expected to fix the
issue so you can continue following this howto.

**Software.** A recent Linux 64-bit distribution with the following software
installed: `python`, `easy_install`, `git`, standard C/C++
build chain. You will need root access in order to install other software or
Python libraries. 

**Hardware.** The lightest setup is a pruning server with diskspace 
requirements well under 1 GB growing very moderately and less taxing 
on I/O and CPU once it's up and running. However note that you also need
to run litecoind and keep a copy of the full blockchain, which is roughly
9 GB in April 2013. If you have less than 2 GB of RAM make sure you limit
litecoind to 8 concurrent connections. If you have more ressources to 
spare you can run the server with a higher limit of historic transactions 
per address. CPU speed is also important, mostly for the initial block 
chain import, but also if you plan to run a public LTCLectrum server, which 
could serve tens of concurrent requests. Any multi-core x86 CPU ~2009 or
newer other than Atom should do for good performance.

Instructions
------------

### Step 1. Create a user for running litecoind and LTCLectrum server

This step is optional, but for better security and resource separation I
suggest you create a separate user just for running `litecoind` and LTCLectrum.
We will also use the `~/bin` directory to keep locally installed files
(others might want to use `/usr/local/bin` instead). We will download source
code files to the `~/src` directory.

    # sudo adduser litecoin --disabled-password
    # su - litecoin
    $ mkdir ~/bin ~/src
    $ echo $PATH

If you don't see `/home/litecoin/bin` in the output, you should add this line
to your `.bashrc`, `.profile` or `.bash_profile`, then logout and relogin:

    PATH="$HOME/bin:$PATH"

### Step 2. Download and install LTCLectrum

We will download the latest git snapshot for LTCLectrum and 'install' it in
our ~/bin directory:

    $ mkdir -p ~/src/ltclectrum
    $ cd ~/src/ltclectrum
    $ sudo apt-get install git
    $ git clone https://github.com/wregab/ltclectrum-server.git server
    $ chmod +x ~/src/ltclectrum/server/server.py
    $ ln -s ~/src/ltclectrum/server/server.py ~/bin/ltclectrum-server

### Step 3. Download litecoind

Older versions of LTCLectrum used to require a patched version of litecoind. 
This is not the case anymore since litecoind supports the 'txindex' option.
We currently recommend litecoind 0.8.6 stable.

If your package manager does not supply a recent litecoind and prefer to compile
here are some pointers for Ubuntu:

    $ cd ~/src && wget https://download.litecoin.org/litecoin-0.8.6.1/linux/litecoin-0.8.6.1-linux.tar.xz
    $ tar xfz litecoin-0.8.6.1-linux.tar.gz
    $ cd litecoin-0.8.6.1-linux/src/src
    $ sudo apt-get install make g++ python-leveldb libboost-all-dev libssl-dev libdb++-dev python-setproctitle
    $ make USE_UPNP= -f makefile.unix
    $ strip ~/src/litecoin-0.8.6.1-linux/src/src/litecoind
    $ ln -s ~/src/litecoin-0.8.6.1-linux/src/src/litecoind ~/bin/litecoind

### Step 4. Configure and start litecoind

In order to allow LTCLectrum to "talk" to `litecoind`, we need to set up a RPC
username and password for `litecoind`. We will then start `litecoind` and
wait for it to complete downloading the blockchain.

    $ mkdir ~/.litecoin
    $ $EDITOR ~/.litecoin/litecoin.conf

Write this in `litecoin.conf`:

    rpcuser=<rpc-username>
    rpcpassword=<rpc-password>
    daemon=1
    txindex=1
    addnode=<one of the addresses listed in https://litecoin.org/downloads/SUPERNODES.txt >


If you have an existing installation of litecoind and have not previously
set txindex=1 you need to reindex the blockchain by running

    $ litecoind -reindex

If you have a fresh copy of litecoind start `litecoind`:

    $ litecoind

Allow some time to pass, so `litecoind` connects to the network and starts
downloading blocks. You can check its progress by running:

    $ litecoind getinfo

You should also set up your system to automatically start litecoind at boot
time, running as the 'litecoin' user. Check your system documentation to
find out the best way to do this.

### Step 5. Install LTCLectrum dependencies

LTCLectrum server depends on various standard Python libraries. These will be
already installed on your distribution, or can be installed with your
package manager. LTCLectrum also depends on two Python libraries which we will
need to install "by hand": `JSONRPClib`.

    $ sudo apt-get install python-setuptools
    $ sudo easy_install jsonrpclib
    $ sudo apt-get install python-openssl

### Step 6. Install leveldb

    $ sudo apt-get install python-leveldb
 
See the steps in README.leveldb for further details, especially if your system
doesn't have the python-leveldb package.

### Step 7. Select your limit

LTCLectrum server uses leveldb to store transactions. You can choose
how many spent transactions per address you want to store on the server.

In original Electrum server pruning limit for spent transactions 
was set to 100 by default.
Since litecoin has significantly less transactions than bitcoin,
in LTCLectrum the default limit is 100000 - what should be considered
an equivalent to "full" server.

You can set this limit to lower number (100/1000/1000),
although this doesn't have big impact on initial blockchain import time.
The difference in database size shouldn't be big neither (TODO study).

The section in the ltclectrum server configuration file (see step 10) looks like this:

     [leveldb]
     path = /path/to/your/database
     # for each address, history will be pruned if it is longer than this limit
     pruning_limit = 100000

### Step 8. Import blockchain into the database or download it

It's recommended to fetch a pre-processed leveldb from the net,
but it's not available yet :)

You can fetch recent copies of ltclectrum leveldb databases and further instructions 
from the LTCLectrum full archival server foundry at:
<to be done>

Alternatively if you have the time and nerve you can import the blockchain yourself.

TODO: replace sync time below
As of April 2013 it takes between 6-24 hours to import 230k of blocks, depending
on CPU speed, I/O speed and selected pruning limit.

It's considerably faster to index in memory. You can use /dev/shm or
or create a tmpfs which will also use swap if you run out of memory:

    $ sudo mount -t tmpfs -o rw,nodev,nosuid,noatime,size=3000M,mode=0777 none /tmpfs

Figures from December 2013:
At limit 100000 the database comes to 1.6 GB with 480k blocks



### Step 9. Create a self-signed SSL cert

To run SSL / HTTPS you need to generate a self-signed certificate
using openssl. You could just comment out the SSL / HTTPS ports in the config and run 
without, but this is not recommended.

Use the sample code below to create a self-signed cert with a recommended validity 
of 5 years. You may supply any information for your sign request to identify your server.
They are not currently checked by the client except for the validity date.
When asked for a challenge password just leave it empty and press enter.

    $ openssl genrsa -des3 -passout pass:x -out server.pass.key 2048
    $ openssl rsa -passin pass:x -in server.pass.key -out server.key
    writing RSA key
    $ rm server.pass.key
    $ openssl req -new -key server.key -out server.csr
    ...
    Country Name (2 letter code) [AU]:US
    State or Province Name (full name) [Some-State]:California
    Common Name (eg, YOUR name) []: ltclectrum-server.tld
    ...
    A challenge password []:
    ...

    $ openssl x509 -req -days 730 -in server.csr -signkey server.key -out server.crt

The server.crt file is your certificate suitable for the ssl_certfile= parameter and
server.key corresponds to ssl_keyfile= in your ltclectrum server config

Starting with LTCLectrum 1.9 the client will learn and locally cache the SSL certificate 
for your server upon the first request to prevent man-in-the middle attacks for all
further connections.

If your certificate is lost or expires on the server side you currently need to run
your server with a different server name along with a new certificate for this server.
Therefore it's a good idea to make an offline backup copy of your certificate and key
in case you need to restore it.

### Step 10. Configure LTCLectrum server

LTCLectrum reads a config file (/etc/ltclectrum.conf) when starting up. This
file includes the database setup, litecoind RPC setup, and a few other
options.

    $ sudo cp ~/src/ltclectrum/server/ltclectrum.conf.sample /etc/ltclectrum.conf
    $ sudo $EDITOR /etc/ltclectrum.conf

Go through the sample config options and set them to your liking.
If you intend to run the server publicly have a look at README-IRC.md 

### Step 11. Tweak your system for running ltclectrum

LTCLectrum server currently needs quite a few file handles to use leveldb. It also requires
file handles for each connection made to the server. It's good practice to increase the
open files limit to 16k. This is most easily achived by sticking the value in .bashrc of the
root user who usually passes this value to all unprivileged user sessions too.

    $ sudo sed -i '$a ulimit -n 16384' /root/.bashrc

We're aware the leveldb part in ltclectrum server may leak some memory and it's good practice to
to either restart the server once in a while from cron (preferred) or to at least monitor 
it for crashes and then restart the server. Weekly restarts should be fine for most setups.
If your server gets a lot of traffic and you have a limited amount of RAM you may need to restart
more often.

Two more things for you to consider:

1. To increase security you may want to close litecoind for incoming connections and connect outbound only

2. Consider restarting litecoind (together with ltclectrum-server) on a weekly basis to clear out unconfirmed
   transactions from the local the memory pool which did not propagate over the network

### Step 12. (Finally!) Run LTCLectrum server

The magic moment has come: you can now start your LTCLectrum server:

    $ ltclectrum-server

You should see this on the screen:

    starting LTCLectrum server
    cache: yes

If you want to stop LTCLectrum server, open another shell and run:

    $ ltclectrum-server stop

You should also take a look at the 'start' and 'stop' scripts in
`~/src/ltclectrum/server`. You can use them as a starting point to create a
init script for your system.

### Step 13. Test the LTCLectrum server

We will assume you have a working LTCLectrum client, a wallet and some
transactions history. You should start the client and click on the green
checkmark (last button on the right of the status bar) to open the Server
selection window. If your server is public, you should see it in the list
and you can select it. If you server is private, you need to enter its IP
or hostname and the port. Press Ok, the client will disconnect from the
current server and connect to your new LTCLectrum server. You should see your
addresses and transactions history. You can see the number of blocks and
response time in the Server selection window. You should send/receive some
litecoins to confirm that everything is working properly.

### Step 14. Join us on IRC, subscribe to the server thread

Say hi to the dev crew, other server operators and fans on 
irc.freenode.net #ltclectrum and we'll try to congratulate you
on supporting the community by running an LTCLectrum node

If you're operating a public LTCLectrum server please subscribe
to or regulary check the following thread:
<TODO>
It'll contain announcements about important updates to LTCLectrum
server required for a smooth user experience.
