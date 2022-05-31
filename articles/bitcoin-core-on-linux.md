---
title: Installing Bitcoin Core on Linux
tags: debian, linux, bitcoincore
---

This article will guide you through installing Bitcoin Core on Linux. My preferred distribution is
[Debian](https://www.debian.org), so examples will be for Debian, but Bitcoin Core can be run on
other Linux distributions as well.

## Prerequisites
You will need a Linux system, with enough storage to store the blockchain, currently around 450G.
This system will need an internet connection.

## Install dependencies
```bash
sudo apt install wget tar
```

## Create bitcoin user
We'll be creating a bitcoin user for Bitcoin core, to keep things secure and separate from the rest
of the system.

```bash
sudo adduser bitcoin
```

Provide a password for the new user when prompted. The other information can be completed or left
empty, as you wish.

## Download and install Bitcoin Core
Browse to https://bitcoin.org/en/download and copy the link for the Linux (tgz) file on the left of
the download page. At the time of writing, this link is
https://bitcoin.org/bin/bitcoin-core-22.0/bitcoin-22.0-x86_64-linux-gnu.tar.gz

Next we'll switch to the *bitcoin* user and download the file using *wget*. Provide the password
when prompted, and substitute the URL for the one you copied from the download page.

```bash
su - bitcoin
wget https://bitcoin.org/bin/bitcoin-core-22.0/bitcoin-22.0-x86_64-linux-gnu.tar.gz
```

Next we'll extract the tar archive.

```bash
tar xf bitcoin-22.0-x86_64-linux-gnu.tar.gz
```

Substitute the correct filename if you downloaded a newer version.
This will create a directory which includes the version number, something like _bitcoin-22.0_.
I like to create a symlink to this directory to make it easy to upgrade in future.

```bash
ln -s bitcoin-22.0 bitcoin
```

We now have a _bitcoin/bin_ directory with the Bitcoin core binaries, we'll add this directory to our
path for convenience.

```bash
echo "PATH=$PATH:~/bitcoin/bin" >> ~/.bashrc
source ~/.bashrc
```

The _.bashrc_ file gets sourced when you log in, we're manually sourcing it here so that we don't have to log
out and back in.

Next we're going to create a *systemd* startup script, so that we can control the bitcoin daemon with the _systemct_ command,
and have it start up automatically at bootup. We'll have to switch to the *root* user to perform these steps.

```bash
su -

cat <<EOF >> /etc/systemd/system/bitcoind.service
[Unit]
Description=Manage bitcoind running as bitcoin user
After=network.target

[Service]
Type=oneshot
User=bitcoin
ExecStart=/home/bitcoin/bitcoin/bin/bitcoind
RemainAfterExit=yes
ExecStop=/home/bitcoin/bitcoin/bin/bitcoin-cli stop

[Install]
WantedBy=multi-user.target
EOF

systemctl enable bitcoind
systemctl start bitcoind &
```

That last command will take a while to run the first time bitcoind is started, which is why we're running it in the background
with *&*.

Give it a minute or two to start up, then you can use *bitcoin-cli* to check the progress of the blockchain download.

```bash
bitcoin-cli getblockchaininfo | less
```

Among the many lines of output, you should see lines like these

> "verificationprogress": 1.387586886309934e-09,
> "initialblockdownload": true,

*verificationprogress* will approach one as your node catches up to the blockchain. And *initialblockdownload* will change to *false* when your node has caught up.

## Congratulations!
Your full node is now up and running, and busy downloading the blockchain. You can find more information on
configuration of bitcoin core at https://bitcoin.org

