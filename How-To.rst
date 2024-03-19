.. _How-To:

============
How-To Guide
============

Prerequisites
=============

**ElectrumX-Dime** should run on any flavour of unix but this guide assumes the
user is running Ubuntu 22.04 LTS (x64) with a user named `electrumx` that has sudo priviliges. Feel free to use
whatever username you prefer, however, make sure to adjust your configuration files accordingly.  

ElectrumX-Dime has also been succesfully run on MacOS and DragonFlyBSD.  It won't run out-of-the-box
on Windows, but the changes required to make it do so should be small - pull requests are welcomed.

Dependencies
------------------
================ ========================
Package          Notes
================ ========================
Python3          ElectrumX-Dime uses asyncio.  Python version >= 3.8 is
                 **required**.
`aiohttp`_       Python library for asynchronous HTTP.  Version >=
                 2.0 required.
`pylru`_         Python LRU cache package.
DB Engine        A database engine package is required; two are
                 supported (see `Database Engine`_ below).
================ ========================

Additionally, there is another package required to handle Dimecoins block hash
functions. Instructions on installing the additional package for Dimecoin to follow below.

Dimecoin Core
------------------------

If you have not done so already, you will need to download and fully sync the
latest version of Dimecoin Core which can be found here: `Dimecoin Core`_

Dimecoin Core needs to be running with the following configurations:

You **must** be running a non-pruning dimecoin daemon with::

  txindex=1

set in its configuration file.  If you have an existing installation
of dimecoind and have not previously set this you will need to reindex
the blockchain with::

  dimecoind -reindex

which can take some time.

Here is a sample of what your configuration file may contain for running your dimecoin node:

.. code-block:: bash

    rpcuser=dimerpcuser
    rpcpassword=dimerpcpwd
    rpcallowip=127.0.0.1
    rpcport=8332
    rpcbind=127.0.0.1
    daemon=1
    logtimestamps=1
    maxconnections=256
    txindex=1

Once you have ensured that all prerequisites and dependencies are met, you can 
proceed to the installation of ElectrumX and its configuration for use with Dimecoin.

Installation and Setup of ElectrumX-Dime (Not as a Service)
==============================================================

Clone the ElextrumX-Dime repository
------------------------------------

.. code-block:: bash

    git clone https://github.com/dime-coin/electrumx-dimecoin.git

Change into the directory of the freshly cloned repository:

.. code-block:: bash

    cd electrumx-dimecoin

Create a new directory for the ElectrumX-Dime database:

.. code-block:: bash

    mkdir db

Build ElectrumX-Dime

.. code-block:: bash

    sudo python3 setup.py install

Change back to home directory

.. code-block:: bash

    cd

Install Dime Quark Hash
------------------------

ElectrumX-Dime requires an additional dependency to handle its hashing functions.

.. code-block:: bash

    git clone https://github.com/dime-coin/dime_quark_hash.git \
    cd dime_quark_hash \
    sudo python3 setup.py install

After completing these steps, ElectrumX and the necessary Dimecoin hashing algorithm are installed 
on your system. You are now ready to configure ElectrumX to communicate with your Dimecoin Core node.

.. _SSL certificates:
Create Self-Signed SSL Certificate
----------------------------------
To ensure secure connections to your ElectrumX server, you'll need to create self-signed SSL certificates. 
Follow these steps within the `electrumx-dimecoin` folder:

.. code-block:: bash

    cd \
    cd electrumx-dimecoin

Now that you are in the electrumx-dimecoin directory, create a certificate directory

.. code-block:: bash

    mkdir cert \
    cd cert

Use OpenSSL to generate a new SSL certificate and private key with the following command:

.. code-block:: bash

    openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem

During the process, you will be prompted to provide details for your certificate:

* Country Name (2 letter code): Enter the two-letter code for your country.
* State or Province Name (full name): Provide the full name of your state or province.
* Locality Name (eg, city): Enter your city's name.
* Organization Name (eg, company): You can skip this by pressing enter.
* Organizational Unit Name (eg, section): Skip this as well by pressing enter.
* Common Name (e.g. server FQDN or YOUR name): Enter electrumx as the common name.
* Email Address: Skip the email address by pressing enter.

If you run into an issue and the :command:`openssl` command does not run, you may need to install OpenSSL on your
system.

These steps create a private key (key.pem) and a public certificate (cert.pem) in the cert directory within 
your electrumx-dimecoin folder. The certificate will be valid for 3650 days (~10 years), ensuring you 
don't need to repeat this process frequently. Feel free to modify the number of days your certificat is valid.
It is good practice to frequently update certs. If an SSL cert becomes compromised, it will persist until expiry.

Create a Shell Script for ElectrumX-Dime
----------------------------------------------

To simplify the process of starting your ElectrumX-Dime server, you will create a shell script. 
This script initializes ElectrumX-Dime with the correct settings for your setup.

Use a text editor like nano to create your script:

.. code-block:: bash

    sudo nano start_elecx.sh

Copy and paste the following lines into the editor. This script sets the necessary environment variables 
and starts ElectrumX-Dime with your Dimecoin configuration:

.. code-block:: bash

    #!/bin/bash
    echo "Starting up Electrumx-dimecoin"
    echo "sudo ALLOW_ROOT=1 COIN=Dimecoin DAEMON_URL=http://dimerpcuser:dimerpcpwd@localhost:8332 SERVICES=ssl://:50002 SSL_CERTFILE=/home/electrumx/electrumx-dimecoin/cert/cert.pem SSL_KEYFILE=/home/electrumx/electrumx-dimecoin/cert/key.pem COST_SOFT_LIMIT=1000000 COST_HARD_LIMIT=10000000000000 DB_DIRECTORY=/home/electrumx/electrumx-dimecoin/db ./electrumx_server -v"
    ALLOW_ROOT=1 COIN=Dimecoin DAEMON_URL=http://dimerpcuser:dimerpcpwd@localhost:8332 SERVICES=ssl://:50002 SSL_CERTFILE=/home/electrumx/electrumx-dimecoin/cert/cert.pem SSL_KEYFILE=/home/electrumx/electrumx-dimecoin/cert/key.pem COST_SOFT_LIMIT=1000000 COST_HARD_LIMIT=10000000000000 DB_DIRECTORY=/home/electrumx/electrumx-dimecoin/db ./electrumx_server -v


After pasting the script into nano, press Ctrl+O to save the file, then Ctrl+X to exit nano.

To ensure the script can be run, make it executable:

.. code-block:: bash

    sudo chmod 774 start_elecx.sh

Now, you have a script ready to start your ElectrumX-Dime server with a simple command.

Install Screen and Launch ElectrumX-Dime
----------------------------------------
Using screen is a practical way to keep ElectrumX-Dime running in the background. Here's how to install screen 
and use it to launch your ElectrumX-Dime server.

If screen is not already installed on your system, you can install it using the following command:

.. code-block:: bash

    sudo apt update && sudo apt install screen -y  

Create a new screen session named `electrumxdime`:

.. code-block:: bash

    screen -S electrumxdime

Within the screen session, start your ElectrumX-Dime server by executing the startup script you created earlier:

.. code-block:: bash

    sudo ./start_elecx.sh
    
.. note:: 

    The first time you start your ElectrumX-Dime server, it will take about ~30 minutes to get in sync with your 
    Dimecoin Core node. You can monitor the screen session to see its progress. Once ElectrumX-Dime is near the end 
    of sync, it will appear to hangup. Do not fear, it is not frozen. After a few minutes, you will start seeing 
    new messages populate the log history.

After confirming that ElectrumX-Dime has started successfully, you can detach from the screen session and leave ElectrumX running in the background:

Press Ctrl + A then D to detach from the screen session.
To reattach to the screen session later and check on ElectrumX-Dime, use the following command:

.. code-block:: bash

    screen -r electrumxdime

This command reconnects you to your `electrumxdime` screen session, where you can view logs, issue commands,
or shut down ElectrumX-Dime properly.

Running ElectrumX-Dime (As a Service)
=====================================

While not a requirement for running ElectrumX-Dime, it is intended to be
run with supervisor software such as Daniel Bernstein's
`daemontools`_, Gerrit Pape's `runit`_ package or :command:`systemd`.
These make administration of secure unix servers very easy, and it is
strongly recommend you install one of these and familiarise yourself
with them.  The instructions below and sample run scripts assume
``daemontools``; adapting to ``runit`` should be trivial for someone
used to either.

Running
-------

Install the prerequisites above.

Check out the code from Github::

    git clone https://github.com/dime-coin/electrumx-dimecoin.git
    cd electrumx

You can install with::

    pip3 install .

There are many extra Python dependencies available to fit the needs of your
system or coins. For example, to install the RocksDB dependencies and a faster
JSON parsing library::

    pip3 install .[rocksdb,ujson]

see setup.py's ``extra_requires`` for a complete list.

You can also run the code from the source tree or a copy of it.

You should create a standard user account to run the server; using your own account is generally 
sufficient for most users. For those who prefer an additional layer of security, it might be advisable 
to create a separate user account specifically for the daemontools logging process. The example scripts 
and instructions provided here assume that everything is operated under a single account, which we've 
named ``electrumx`` in the examples below.

Next create a directory where the database will be stored and make it
writeable by the ``electrumx`` account.  It is recommended this directory
live on an SSD::

    mkdir /path/to/db_directory
    chown electrumx /path/to/db_directory


Using daemontools
-----------------

Next create a daemontools service directory; this only holds symlinks
(see daemontools documentation).  The :command:`svscan` program will
ensure the servers in the directory are running by launching a
:command:`supervise` supervisor for the server and another for its
logging process.  You can run :command:`svscan` under the *electrumx*
account if that is the only one involved (server and logger) otherwise
it will need to run as root so that the user can be switched to
electrumx.

Assuming this directory is called :file:`service`, you would do one
of::

    mkdir /service       # If running svscan as root
    mkdir ~/service      # As electrumx if running svscan as that a/c

Next create a directory to hold the scripts that the
:command:`supervise` process spawned by :command:`svscan` will run -
this directory must be readable by the :command:`svscan` process.
Suppose this directory is called :file:`scripts`, you might do::

    mkdir -p ~/scripts/electrumx

Then copy the all sample scripts from the ElectrumX-Dime source tree there::

    cp -R /path/to/repo/electrumx/contrib/daemontools ~/scripts/electrumx

This copies 3 things: the top level server run script, a :file:`log/`
directory with the logger :command:`run` script, an :file:`env/`
directory.

You need to configure the :ref:`environment variables <environment>`
under :file:`env/` to your setup.  ElectrumX-Dime server currently takes no
command line arguments; all of its configuration is taken from its
environment which is set up according to :file:`env/` directory (see
:manpage:`envdir` man page).  Finally you need to change the
:command:`log/run` script to use the directory where you want the logs
to be written by multilog.  The directory need not exist as
:command:`multilog` will create it, but its parent directory must
exist.

Now start the :command:`svscan` process.  This will not do much as the
service directory is still empty::

    svscan ~/service & disown

svscan is now waiting for services to be added to the directory::

    cd ~/service
    ln -s ~/scripts/electrumx electrumx

Creating the symlink will kick off the server process almost immediately.
You can see its logs with::

    tail -F /path/to/log/dir/current | tai64nlocal


Using systemd
-------------

This repository contains a sample systemd unit file that you can use
to setup ElectrumX-Dime with systemd. Simply copy it to
:file:`/etc/systemd/system`::

    cp contrib/systemd/electrumx.service /etc/systemd/system/

The sample unit file assumes that the repository is located at
:file:`/home/electrumx/electrumx`. If that differs on your system, you
need to change the unit file accordingly.

You need to set a few :ref:`environment variables <environment>` in
:file:`/etc/electrumx.conf`.

Now you can start ElectrumX-Dime using :command:`systemctl`::

    systemctl start electrumx

You can use :command:`journalctl` to check the log output::

    journalctl -u electrumx -f

Once configured you may want to start ElectrumX-Dime at boot::

    systemctl enable electrumx

.. Warning:: systemd is aggressive in forcibly shutting down
   processes.  Depending on your hardware, ElectrumX-Dime can need several
   minutes to flush cached data to disk during initial sync.  You
   should set TimeoutStopSec to *at least* 10 mins in your
   :file:`.service` file.


Installing on Raspberry Pi 3
----------------------------

To install on the Raspberry Pi 3 you will need to update to the
``stretch`` distribution.  See the full procedure in
`contrib/raspberrypi3/install_electrumx.sh`_.

See also `contrib/raspberrypi3/run_electrumx.sh`_ for an easy way to
configure and launch electrumx.

Additional Considerations
=========================

Database Engine
---------------

When building the database from the genesis block, ElectrumX-Dime has to
flush large quantities of data to disk and its DB.  You will have a
better experience if the database directory is on an SSD rather than on an
HDD.  Currently to around height 5,550,000 of the Dimecoin blockchain the
final size of the leveldb database, and other ElectrumX-Dime file metadata
comes to just over XX.XXB (XX.X GiB).  LevelDB needs a bit more for
brief periods, and the block chain is only getting longer, so it is
recommend having at least 70-80GB of free space before starting.

You can choose from LevelDB and RocksDB to store transaction
information on disk.  The time taken and DB size is not significantly
different.  We tried to support LMDB, but its history write performance
was much worse.

You will need to install one of:

+ `plyvel <https://plyvel.readthedocs.io/en/latest/installation.html>`_ for LevelDB.

  Included as part of a regular pip or ``setup.py`` installation of ElectrumX-Dime.
+ `python-rocksdb <https://pypi.python.org/pypi/python-rocksdb>`_ for RocksDB

  ``pip3 install python-rocksdb`` or use the rocksdb extra install option to ElectrumX-Dime.
+ `pyrocksdb <http://pyrocksdb.readthedocs.io/en/v0.4/installation.html>`_ for an unmaintained version that doesn't work with recent releases of RocksDB

Process Limits
--------------

You must ensure the ElectrumX-Dime process has a large open file limit.
During sync it should not need more than about 1,024 open files.  When
serving it will use approximately 256 for LevelDB plus the number of
incoming connections.  It is not unusual to have 1,000 to 2,000
connections being served, so it is suggested you set your open files limit
to at least 2,500.

Note that setting the limit in your shell does *NOT* affect ElectrumX-Dime
unless you are invoking ElectrumX-Dime directly from your shell.  If you
are using :command:`systemd`, you need to set it in the
:file:`.service` file (see `contrib/systemd/electrumx.service`_).

Sync Progress
--------------

Time taken to index the blockchain depends on your hardware of course.
As Python is single-threaded most of the time only 1 core is kept
busy.  ElectrumX-Dime uses Python's :mod:`asyncio` to prefill a cache of
future blocks asynchronously to keep the CPU busy processing the chain
without pausing.

Consequently there will probably be only a minor boost in performance
if the daemon is on the same host.  It may even be beneficial to have
the daemon on a *separate* machine so the machine doing the indexing
has its caches and disk I/O tuned to that task only.

The :envvar:`CACHE_MB` environment variable controls the total cache
size ElectrumX-Dime uses; see :ref:`here <CACHE>` for caveats.

.. note:: ElectrumX-Dime will not serve normal client connections until it
          has fully synchronized and caught up with your daemon.
          However LocalRPC connections are served at all times.


Terminating ElectrumX-Dime
--------------------------

The preferred way to terminate the server process is to send it the
``stop`` RPC command::

  electrumx_rpc stop

or alternatively on Unix the ``INT`` or ``TERM`` signals.  For a
daemontools supervised process this can be done by bringing it down
like so::

    svc -d ~/service/electrumx

ElectrumX-Dime will note receipt of the signals in the logs, and ensure the
block chain index is flushed to disk before terminating.  You should
be patient as flushing data to disk can take many minutes.

ElectrumX-Dime uses the transaction functionality, with fsync enabled, of
the databases.  It is written with the intent that, to the extent
the atomicity guarantees are upheld by the DB software, the operating
system, and the hardware, the database should not get corrupted even
if the ElectrumX-Dime process if forcibly killed or there is loss of power.
The worst case should be having to restart indexing from the most
recent UTXO flush.

Once the process has terminated, you can start it up again with::

    svc -u ~/service/electrumx

You can see the status of a running service with::

    svstat ~/service/electrumx

:command:`svscan` can of course handle multiple services
simultaneously from the same service directory, such as a testnet or
altcoin server.  See the man pages of these various commands for more
information.

Understanding the Logs
----------------------

You can see the logs usefully like so::

    tail -F /path/to/log/dir/current | tai64nlocal

Here is typical log output on startup::

  INFO:BlockProcessor:switching current directory to /crucial/server-good
  INFO:BlockProcessor:using leveldb for DB backend
  INFO:BlockProcessor:created new database
  INFO:BlockProcessor:creating metadata diretcory
  INFO:BlockProcessor:software version: ElectrumX 0.10.2
  INFO:BlockProcessor:DB version: 5
  INFO:BlockProcessor:coin: Dimecoin
  INFO:BlockProcessor:network: mainnet
  INFO:BlockProcessor:height: -1
  INFO:BlockProcessor:tip: 0000000000000000000000000000000000000000000000000000000000000000
  INFO:BlockProcessor:tx count: 0
  INFO:BlockProcessor:sync time so far: 0d 00h 00m 00s
  INFO:BlockProcessor:reorg limit is 200 blocks
  INFO:Daemon:daemon at 192.168.0.2:8332/
  INFO:BlockProcessor:flushing DB cache at 1,200 MB
  INFO:Controller:RPC server listening on localhost:8000
  INFO:Prefetcher:catching up to daemon height 447,187...
  INFO:Prefetcher:verified genesis block with hash 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
  INFO:BlockProcessor:our height: 9 daemon: 447,187 UTXOs 0MB hist 0MB
  INFO:BlockProcessor:our height: 52,509 daemon: 447,187 UTXOs 9MB hist 14MB
  INFO:BlockProcessor:our height: 85,009 daemon: 447,187 UTXOs 12MB hist 31MB
  INFO:BlockProcessor:our height: 102,384 daemon: 447,187 UTXOs 15MB hist 47MB
  [...]
  INFO:BlockProcessor:our height: 133,375 daemon: 447,187 UTXOs 80MB hist 222MB
  INFO:BlockProcessor:our height: 134,692 daemon: 447,187 UTXOs 96MB hist 250MB
  INFO:BlockProcessor:flushed to FS in 0.7s
  INFO:BlockProcessor:flushed history in 16.3s for 1,124,512 addrs
  INFO:BlockProcessor:flush #1 took 18.7s.  Height 134,692 txs: 941,963
  INFO:BlockProcessor:tx/sec since genesis: 2,399, since last flush: 2,400
  INFO:BlockProcessor:sync time: 0d 00h 06m 32s  ETA: 1d 13h 03m 42s

Under normal operation these cache stats repeat once or twice a
minute.  UTXO flushes can take several minutes and look like this::

  INFO:BlockProcessor:our height: 378,745 daemon: 447,332 UTXOs 1,013MB hist 184MB
  INFO:BlockProcessor:our height: 378,787 daemon: 447,332 UTXOs 1,014MB hist 194MB
  INFO:BlockProcessor:flushed to FS in 0.3s
  INFO:BlockProcessor:flushed history in 13.4s for 934,933 addrs
  INFO:BlockProcessor:flushed 6,403 blocks with 5,879,440 txs, 2,920,524 UTXO adds, 3,646,572 spends in 93.1s, committing...
  INFO:BlockProcessor:flush #120 took 226.4s.  Height 378,787 txs: 87,695,588
  INFO:BlockProcessor:tx/sec since genesis: 1,280, since last flush: 359
  INFO:BlockProcessor:sync t ime: 0d 19h 01m 06s  ETA: 3d 21h 17m 52s
  INFO:BlockProcessor:our height: 378,812 daemon: 447,334 UTXOs 10MB hist 10MB

The ETA shown is just a rough guide and in the short term can be quite
volatile.  It tends to be a little optimistic at first; once you get
to height 1,200,000 is should be fairly accurate.

Running on a Pivileged Port
-----------------------------

You may choose to run Electrumx-Dime on a different port than 50001
/ 50002.  If you choose a privileged port ( < 1024 ) it makes sense to
make use of a iptables NAT rule.

An example, which will forward Port 110 to the internal port 50002 follows::

    iptables -t nat -A PREROUTING -p tcp --dport 110 -j DNAT --to-destination 127.0.0.1:50002

You can then set the port as follows and advertise the service externally on the privileged port::

    REPORT_SSL_PORT=110

Compact DB History and Required Maintenance
-------------------------------------------

The ElectrumX-Dime server stores transaction history and other blockchain data in a database. Over time, this database can grow significantly in size due to the accumulation of transaction records. Running the compaction script will be
required periodically once the flushcount reaches its limits. This will help:

* Reduce the disk space used by the database.
* Improving the performance of the server by optimizing data storage.
* Ensuring query response times are maintained for Electrum clients.

The process involves just a few steps, they are as follows (remember you need sudo priviliges to execute these commands):

* **Step 1:** Stop the ElectrumX-Dime server

.. code-block::

    sudo ./electrumx_server stop

* **Step 2:** Run the `electrumx_compact_script` included in the Electrumx-Dime root directory of your installation. 

.. note:: 

    This script needs to be run with the same start parameters you use to start your ElectrumX-Dime server. For example, if you start your ElectrumX-Dime server with:

    .. code-block::

        `sudo ALLOW_ROOT=1 COIN=Dimecoin DAEMON_URL=http://testuser:testpass@127.0.0.1:8332 SERVICES=ssl://:50002 SSL_CERTFILE=/root/cert/cert.pem SSL_KEYFILE=/root/cert/key.pem COST_SOFT_LIMIT=1000000 COST_HARD_LIMIT=10000000000000 DB_DIRECTORY=/root/electrumx-dimecoin/db ./electrumx_server -v`

    You will need to start the compact script with the following:

    .. code-block::

        `sudo ALLOW_ROOT=1 COIN=Dimecoin DAEMON_URL=http://testuser:testpass@127.0.0.1:8332 SERVICES=ssl://:50002 SSL_CERTFILE=/root/cert/cert.pem SSL_KEYFILE=/root/cert/key.pem COST_SOFT_LIMIT=1000000 COST_HARD_LIMIT=10000000000000 DB_DIRECTORY=/root/electrumx-dimecoin/db ./electrumx_compact_history -v`
    
Remember to change the start parameters used to run the script to match your setup and installation.

* **Step 3:** Once the compact script finishes, restart your ElectrumX-Dime server. As noted above, starting the server takes several minutes and it will appear to hang. Understand it is not frozen. Let it finish startup.

* **Step 4:** Finished. Your ElectrumX-Dime server will now run as expected. When the db flushcount reaches its limit again, you will need to run this script again to compact the db history.






.. _`contrib/systemd/electrumx.service`: https://github.com/spesmilo/electrumx/blob/master/contrib/systemd/electrumx.service
.. _`daemontools`: http://cr.yp.to/daemontools.html
.. _`runit`: http://smarden.org/runit/index.html
.. _`aiohttp`: https://pypi.python.org/pypi/aiohttp
.. _`pylru`: https://pypi.python.org/pypi/pylru
.. _`dash_hash`: https://pypi.python.org/pypi/dash_hash
.. _`contrib/raspberrypi3/install_electrumx.sh`: https://github.com/spesmilo/electrumx/blob/master/contrib/raspberrypi3/install_electrumx.sh
.. _`contrib/raspberrypi3/run_electrumx.sh`: https://github.com/spesmilo/electrumx/blob/master/contrib/raspberrypi3/run_electrumx.sh
.. _`Dimecoin Core`: https://github.com/dime-coin/dimecoin/releases