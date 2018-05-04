1. Create electrumx user
========================

        adduser electrumx

        usermod -aG sudo electrumx


2. Install Zcoin Core
=====================
Before you start, make sure to install a full Zcoin node first and set at least the following minimum options in zcoin.conf:

        server=1

        listen=1

        daemon=1

        txindex=1

        rpcuser=<random username>

        rpcpassword=<strong password>


3. Install leveldb
==================

        sudo apt-get install python3-leveldb libleveldb-dev


4. Install Python 3.6
=====================
ElectrumX developer decided to use newer Python 3 which isn't installed on many operating systems by default. Let's install it manually.

        sudo add-apt-repository ppa:jonathonf/python-3.6

        sudo apt-get update && sudo apt-get install python3.6 python3.6-dev python3-pip

To make python3 use the new installed python 3.6 instead of the default 3.5 release, run following 2 commands:

        sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.5 1

        sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 2

Finally switch between the two python versions for python3 via command:

        sudo update-alternatives --config python3


5. Install required Python packages
===================================
You will also need to install some Python 3.6 dependencies for ElectrumX.
Let's first upgrade setuptools:

        pip3 install --upgrade pip setuptools wheel

then install some required packages:

        pip3 install aiohttp pylru leveldb plyvel

If the above command gives you an error, try installing it with sudo (try to avoid it as much as possible, though).


6. Install and set up ElectrumX
===============================
Clone the ElectrumX code from a GitHub repository using git:

        mkdir ~/source

        cd ~/source

        git clone https://github.com/zcoinofficial/electrumx.git

        cd electrumx

Run the installation script (use sudo if it doesn't work):

        python3 setup.py install

Next, create a data folder where the blockchain data will be stored:

        mkdir ~/.electrumx


7. Create a self-signed certificate
===================================
To allow Electrum wallets to connect to your server over SSL you need to create a self-signed certificate.
Go to the data folder:

        cd ~/.electrumx

and generate your key:

        openssl genrsa -des3 -passout pass:x -out server.pass.key 2048

        openssl rsa -passin pass:x -in server.pass.key -out server.key

        rm server.pass.key

        openssl req -new -key server.key -out server.csr

Follow the on-screen information. It will ask for certificate details such as your country and password. You can leave those fields empty.
When done, create a certificate:

        openssl x509 -req -days 1825 -in server.csr -signkey server.key -out server.crt

These commands will create 2 files: server.key and server.crt.


8. Launch ElectrumX as service
==============================
First, make sure a fully validating Zcoin node is running:

        zcoin-cli getinfo

        curl --user <rpcuser>:<rpcpassword> --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getinfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:8888/

Open a sudo session and copy a service file from the ElectrumX repo to your Systemd directory:

        sudo -s

        cp ~/source/electrumx/contrib/systemd/electrumx.service /lib/systemd/system/

Edit the file to match your setup:

        nano /lib/systemd/system/electrumx.service

You need to edit at least ExecStart and User variables.

        ExecStart=/home/electrumx/source/electrumx/electrumx_server.py

        User=electrumx

Create a configuration file for the server:

        nano /etc/electrumx.conf

and configure it according to your environment.

Please refer to `ElectrumX's documentation`_ or have a look at my settings (`contrib/systemd/electrumx.conf.zcoinsample`_).

Start the service:

        systemctl start electrumx

and check the output:

        journalctl /usr/bin/python3 -f -n100

If it gives you no errors, enable the service:

        systemctl enable electrumx

You can exit the sudo session now:

        exit

.. _contrib/systemd/electrumx.conf.zcoinsample: https://github.com/zcoinofficial/electrumx/blob/master/contrib/systemd/electrumx.conf.zcoinsample
.. _ElectrumX's documentation: https://github.com/zcoinofficial/electrumx/blob/master/docs/ENVIRONMENT.rst
