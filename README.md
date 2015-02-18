# Samba AD/DC

This repository contains a Vagrantfile that describes the
setup of a Samba Active Directort Domain Controller.

In short:

* setup controlled by vagrant
* LXC-Containers as basis
* node OS: Ubuntu Trusty (for a start)

## Configuration

This Vagrantfile is parametrized: The options for configuring
the number of nodes and the nodes' network interfaces and addresses
are stored in the config file 'vagrant.yml' when running any
vagrant command (except for vagrant help). The whole setup can
then conveniently be reconfigured modifying that file.

## Prerequisites

* Linux host, attached to the network (tested on Fedora 21, should work on other hosts as well)
* lxc installed
* vagrant installed
* vagrant-lxc plugin installed
* possibly preparation of bridge interfaces on the host (the example uses libvirt's `virbr` interfaces)

## Running

* Run `vagrant status` to create the `vagrant.yaml` config file.
* Adjust the config file.
* Run `vagrant up`.

## TODO

* make provisioning reentrant/idempotent
* maybe do that using ansible?

## Author

Michael Adam (obnox at samba dot org)
