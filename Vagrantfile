# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = 2


require 'yaml'

#
# Defaults for Configuration data.
# Will be overridden from the settings file
# and (possibly later) from commandline parameters.
#

net_default = {
  :type   => 'veth',
  :flags  => 'up',
  :hwaddr => '',
  :name   => '',
  :ipv4   => '',
  :ipv6   => '',
}

network_opts = [ :type, :link, :flags, :hwaddr, :name, :ipv4, :ipv6 ]

vms = [
  {
    :hostname => 's4dc-180',
    :box => 'local-trusty64',
    :container_name => 's4dc-180',
    :networks => [
      {
        :link => 'virbr1',
        :ipv4 => '172.16.1.180',
      },
      #{
      #  :link => 'virbr2',
      #  #:ipv4 => '10.111.222.201',
      #},
    ],
  },
]

#
# Load the config, if it exists,
# possibly override with commandline args,
# (currently none supported yet)
# and then store the config.
#

projectdir = File.expand_path File.dirname(__FILE__)
f = File.join(projectdir, 'vagrant.yaml')
if File.exists?(f)
  settings = YAML::load_file f

  if settings[:vms].is_a?(Array)
    vms = settings[:vms]
  end
  puts "Loaded settings from #{f}."
end

# TODO(?): ARGV-processing

settings = {
  :vms  => vms,
}

File.open(f, 'w') do |file|
  file.write settings.to_yaml
end
puts "Wrote settings to #{f}."

# apply net defaults:

vms.each do |vm|
  vm[:networks].each do |net|
    net_default.keys.each do |key|
      if not net.has_key?(key)
        net[key] = net_default[key]
      end
    end
  end
end


PROVISION_SCRIPT = <<SCRIPT
apt-get -y install make samba
SCRIPT

#Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
#  if Vagrant.has_plugin?("vagrant-cachier")
#    config.cache.scope = :box
#  end
#
#  config.vm.define "s4dc" do |machine|
#    machine.vm.box      = "local-trusty64"
#    machine.vm.hostname =  "s4dc-180"
#    machine.vm.provider :lxc do |lxc|
#      lxc.container_name = "s4dc-180"
#      lxc.customize "network.type", "veth"
#      lxc.customize "network.link", "virbr1"
#      lxc.customize "network.flags", "up"
#      lxc.customize "network.ipv4", "172.16.1.180"
#    end
#    machine.vm.provision :shell, inline: PROVISION_SCRIPT
#  end
#end


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  #
  # Provisioning common to all machines:
  #
  config.vm.provision :shell, inline: PROVISION_SCRIPT

  vms.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]
      node.vm.provider :lxc do |lxc|
        lxc.container_name = machine[:container_name]

        machine[:networks].each do |net|
          network_opts.each do |key|
            if not net[key] == ''
             lxc.customize "network.#{key}", net[key]
            end
          end
        end
      end
      #node.vm.synced_folder "shared/", "/shared"
    end
  end

end
