# Setup Zebra Daemon (Zcash Blockchain Node) on a Development Workstation


This document is a guide for installing zebra, on BOTH **mainnet** and **testnet**, on a development workstation.

**As of the creation date of this document a Debian package does not appear to be generally available in the (zcash apt repository)[https://apt.z.cash/].** 



## Preamble


This guide has been generated using notes I have collected from various sources. Some of these notes relate to system administration tasks on Debian Linux and how they relate to running the Zebra Daemon (zebrad) on Debian. 

The single best information resource regarding zebra can be found on its [homepage](https://zebra.zfnd.org/). The source code for zebra can be found on [github](https://github.com/ZcashFoundation/zebra). 

The single best information resource for (Debian)[https://www.debian.org] is, of course, the (Debian Home Page)[https://www.debian.org] more specifically the (Debian Users' Manuals)[https://www.debian.org/doc/user-manuals]. (The Debian Administrator's Handbook)[https://debian-handbook.info/] is also an excellent resource. 

This zebrad setup "recipe" assumes that your Debian development workstation already has a copy of the ORIGINAL (zcashd)[https://github.com/zcash/zcash] full node software installed and configured according to the instruction published in my (github repository here)[https://github.com/gesker/zcashd_config]. Completing the instructions found in those instructions is a prerequisite to completing this recipe. This document will also follow the same pattern found in the earlier guide of installing the node a Linux Service so that the node software is controlled by the systemd software provided by Debian. The zebra node operates on BOTH the zcash mainnet and testnet blockchains. Because, upon completion of BOTH sets of instructions, your workstation will have *zcashd* running on ports *8233* (mainnet) and *18233* (testnet) and zebrad running on the NON-standard ports of *8243* (for mainnet) and *18243* (for testnet).


It is helpful when the zebra and/or zcashd software is communicating and synchronizing on BOTH the main (mainnet) and test (testnet) networks and the start/stop life cycle is controlled by the operating system. While there is a great deal of good information resources available for zebra I hope this recipe/guide can bring much of that together, along with a touch of sysadmin, to lower the barrier of entry for developers wishing to build applications utilizing the zcash blockchain and more specifically use zebra for their development efforts.



## License


    Copyright (c) 2023 Dennis R. Gesker.
    Permission is granted to copy, distribute and/or modify this documenet spe
    under the terms of the GNU Free Documentation License, Version 1.3
    or any later version published by the Free Software Foundation;
    with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
    A copy of the license is included in the section entitled "GNU

## Goals


- Install zebra (zebrad)[https://github.com/ZcashFoundation/zebra]
  - Using the (zebra)[https://github.com/ZcashFoundation/zebra] repository
- Configure zcashd to automatically start using systemd
  - Mainnet (mainnet)
  - Testnet (testnet)
- Configure zcashd
  - Mainnet (mainnet)
    - Use NON-standard port 8243 and 8242 (RPC)
        - Default port is 8233 for mainnet; 8232 for RPC
  - Testnet (testnet)
    - Use NON-standard port 18243 and 18242 (RPC)
        - Default port is 18233 for mainnet; 18232 for RPC
- Configure systemd


## Non-Goals


- Guarantee Security of the configuration
- This is for a personal workstation only!
- Generate a wallet.dat
    - See the guide on (lightwalletd)[https://github.com/gesker/lightwalletd_config]


## Miscellaneous 


- I am not part of the (zcash)[https://github.com/zcash/zcash] project
- I am not part of the (zebra)[https://github.com/ZcashFoundation/zebra] project
- I DO use this configuration on my own workstation
  - Use at your own risk!
- Pull requests to improve these notes are welcome


## Requirements


- Debian 11 (Bookworm)
- Ability to sudo on the system
- Ability to use a text editor and navigate directories
- About 400 GB of disk space in your home *~/* directory


## Step 1 - Install some prerequisites

```
sudo apt update
sudo apt install wget git libclang-dev protobuf-compiler 
```


## Step 2 - Install Rust

We will need the rust compiler and some associated rust development tools to compile the source code. Download and install rust. 

```
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustc --version
```


## Step 2 - Download & Compile Source Code

As of the creation of this document zebra is in *release candidate* status at version (v1.0.0-rc.5)[https://github.com/ZcashFoundation/zebra/releases/tag/v1.0.0-rc.5] and as far as I have been able to determine a Debian apt package is not yet available. We will install the software from the source code.


```bash
mkdir ~/Development/
cd ~/Development/
git clone git@github.com:ZcashFoundation/zebra.git
cd zebra
git checkout v1.0.0-rc.5;   
cargo build --all-features --locked
```


The build process will complete and the new zebrad binary can be found in the target directory.


```
tree -L 1 ~/Development/zebra/target/debug
~/Development/zebra/target/debug
├── ...
├── zebrad  <-- This is the executable for Zebra Daemon
└── ...
```


## Step 4 - Create Storage Location for Zcash Blockchain



The zcash blockchain is pretty large. Let's store these files under the ~/zebrad directory.

*Tip:* If you have a LARGER physical drive mounted on the system create a symbolic link from ~/zebrad to that larger drive. 


```bash
mkdir ~/zebrad;
```


## Step 5 - Copy and UPDATE the configuration files


Copy the configuraiton files to this new folder.

```bash
sudo wget https://github.com/gesker/zebrad_config/zebrad_mainnet.toml -O ~/zebrad/zebrad_mainnet.toml
sudo wget https://github.com/gesker/zebrad_config/zebrad_testnet.toml -O ~/zebrad/zebrad_testnet.toml
```

**Important:** In BOTH of the files above CHANGE *yourUserName* to your actual username on the system.

*Tip:* If you DO NOT have a local copy of zcashd, the original zcash full node, copy down zebra_mainnet-alternate.toml and zebra_testnet-alternate.toml from my repository, in the above commands instead. These "alternate" files are configured to use the standard ports of 8233 & 18233 BUT you will still need to update *yourUserName.



## Step 6 - Copy and UPDATE the systemd files


Copy down the configuration files for systemd, enable the services, and start the services


```bash
sudo wget https://github.com/gesker/zebrad_config/zebrad_testnet.service -O /etc/systemd/system/zebrad_testnet.service
sudo wget https://github.com/gesker/zebrad_config/zebrad_mainnet.service -O /etc/systemd/system/zebrad_mainnet.service
```


**Important:** In BOTH of the files above CHANGE *yourUserName* to your actual username on the system.


Reload the systemd daemon so that the new files are recognized and start zebrad on BOTH mainnet and testnet.

```bash
sudo systemctl daemon-reload 
sudo systemctl start zebrad_testnet;
sudo systemctl start zebrad_mainnet;
```

Tail the log files to confirm that the services are running

```bash
tail -f /var/lib/zebrad/state/v25/testnet/LOG  # Ctrl-C to exit tail
tail -f /var/lib/zebrad/state/v25/mainnet/LOG  # Ctrl-c to exit tail
```

Instruct systemd to start zebrad when when your workstation is restarted.
```bash
sudo systemctl enable zebrad_testnet; # Optional but useful
sudo systemctl enable zebrad_mainnet; # Optional but useful
```

It will take some time for zebrad to fully syncronize the mainnet and testnet blockchains!




## Additional Information Sources


- [Zcash Documentation](https://zcash.readthedocs.io/en/latest/)
- [Electric Coin Company](https://electriccoin.co/)
- [Zcash Forums](https://forum.zcashcommunity.com/)
- [Debian Adminsitrator's Handbook](https://debian-handbook.info/)


## Discussion

The original software for working with the zcash blockchain (zcashd) was forked from bitcoin and performed two main functions; communicating/operating with the zcash blockchain and managing the wallet.dat file. Zebra (zebrad) is a newer full node software tool that ONLY communicates/operates on the blockchain. It appears that there will be a forthcoming zebra-cli program that will handle wallet.dat operations. In the meantime if you use Zebra install the "companion" lightwalletd software and wallet software designed to work with the zebrad/lightwalletd combination.



## This Document



- Feedback welcome
- Pull requests welcome
- Original Located at [Github](https://github.com/gesker/zebrad_config)

I hope you found these notes useful, getting zcashd setup relatively quickly and can back to coding your zcash application!


