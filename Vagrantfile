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
    :provider => {
      :lxc => {
        :container_name => 's4dc-180',
      },
    },
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


SCRIPT_INSTALL = <<SCRIPT
set -e

apt-get -y update
apt-get -y upgrade
apt-get -y install samba
SCRIPT

SCRIPT_PROVISION = <<SCRIPT
set -e

stop smbd || true
stop nmbd || true
stop samba-ad-dc || true

BACKUP_SUFFIX=".orig.$(date +%Y%m%d-%H%M%S)"

FILE=/etc/samba/smb.conf
mv -f ${FILE} ${FILE}${BACKUP_SUFFIX}

samba-tool domain provision \
	--domain=S4AD \
	--realm=s4ad.private \
	--adminpass=Passw0rd \
	--server-role=dc \
	--function-level=2008_R2 \
	--use-rfc2307

FILE=/etc/resolv.conf
mv -f ${FILE} ${FILE}${BACKUP_SUFFIX}
cat <<EOF > ${FILE}
nameserver 127.0.0.1
domain s4ad.private
search s4ad.private
EOF

sudo sed -i${BACKUP_SUFFIX} -e 's/passwd: .*/passwd:         files winbind/g' -e 's/group: .*/group:          files winbind/g' /etc/nsswitch.conf

start samba-ad-dc
SCRIPT


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  vms.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]
      node.vm.provider :lxc do |lxc|
        lxc.container_name = machine[:provider][:lxc][:container_name]

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

  #
  # Provisioning common to all machines:
  #
  config.vm.provision :shell, inline: SCRIPT_INSTALL
  config.vm.provision :shell, inline: SCRIPT_PROVISION

end
