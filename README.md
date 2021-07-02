# Vagrant GVM/Openvas

## What is Vagrant?

Vagrant is a tool that uses Oracle's VirtualBox to dynamically build configurable, lightweight, and portable virtual machines. Vagrant supports the use of either Puppet or Chef for managing the configuration. Much more information is available on the [Vagrant web site](http://www.vagrantup.com).

## What is this project?

This is the Vagrant configuration used in my boxes for [GVM/Openvas](https://www.greenbone.net/en/) at [VagrantCloud](https://app.vagrantup.com/isaqueprofeta). They start out with GVM full instaled and updated definitions database in 06/2021. The Operating System is an Alpine Linux 2.13.

## How do I install Vagrant?

The VirtualBox version used is 6.1 and Vagrant version is v2.2.14.

- Download VirtualBox 6.1: https://www.virtualbox.org/wiki/Downloads
- Download Vagrant 2.2.14: https://www.vagrantup.com/downloads

## How do I run?

Two options:

1. Using the VagrantCloud box: From an empty directory, Create a Vagrantfile and put in it the contents:

config.vm.box could be "isaqueprofeta/gvm-openvas"

```
Vagrant.configure("2") do |config|
  config.vm.box = "isaqueprofeta/gvm-openvas"
  config.vm.box_version = "1.0.0"
  config.vm.network "forwarded_port", guest: 9392, host: 9392, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 5432 , host: 5432, host_ip: "127.0.0.1"
end
```

Save, exit, and then type:

```
vagrant init isaqueprofeta/gvm-openvas
vagrant up
```

2. Build ground up from Alpine Linux: Clone this repo and inside the wanted OS folder, run vagrant up.

## How do I work?

0. Greenbone Assitant Frontend is forwarded from TCP/9392 of guest to TCP/9392 on the host, so just go for http://localhost:9392 (User: admin, Pass:Vu1nCh3ck3R), and you should be fine

1. Port 5432 are directly mapped from the guest to the host to connect and make queries using SQL.

2. gvmd Database Password: Vu1nCh3ck3R

3. Crontab is already configured to update Database Definitions everyday from 3 to 6 AM using the config below:

```
0 3 * * * sudo -u gvm greenbone-feed-sync --type GVMD_DATA
0 4 * * * sudo -u gvm greenbone-feed-sync --type SCAP
0 5 * * * sudo -u gvm greenbone-feed-sync --type CERT
0 6 * * * sudo -u gvm greenbone-nvt-sync
```

## Quick reference:

1. How to stop/turn off:

```
vagrant halt
```

1. How to clean/delete all data:

```
vagrant destroy
vagrant box prune isaqueprofeta/gvm-openvas
```

2. Create a snapshot for tests:

```
vagrant snapshot create MySnapshot
```

2. Recover the snapshot:

```
vagrant snapshot restore MySnapshot
```
