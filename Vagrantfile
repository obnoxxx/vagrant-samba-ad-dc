# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = 2

PROVISION_SCRIPT = <<SCRIPT
apt-get -y install make samba

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  config.vm.define "s4dc" do |machine|
    machine.vm.box      = "local-trusty64"
    machine.vm.hostname =  "s4dc-180"
    machine.vm.provider :lxc do |lxc|
      lxc.container_name = "s4dc-180"
      lxc.customize "network.type", "veth"
      lxc.customize "network.link", "virbr1"
      lxc.customize "network.flags", "up"
      lxc.customize "network.ipv4", "172.16.1.180"
    end
    machine.vm.provision :shell, inline: PROVISION_SCRIPT
  end
end
