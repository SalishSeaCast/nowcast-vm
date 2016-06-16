# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrant VM configuration to emulate Salish Sea Nowcast system
# deployment on skookum.ubc.ca


# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "nowcast" do |nowcast|
    # Ubuntu 14.04 LTS
    nowcast.vm.box = "ubuntu/trusty64"

    config.ssh.forward_agent = true
  end

  config.vm.synced_folder "../tools/", "/results/nowcast-sys/tools",
    create: true
  config.vm.synced_folder "../salishsea-site/",
    "/home/vagrant/nowcast/www/salishsea_site",
    create: true

  # Provisioning
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y mg
    apt-get install sshfs

    mkdir -p /data && chown vagrant:vagrant /data
    mkdir -p /ocean && chown vagrant:vagrant /ocean

    chown vagrant:vagrant /results
    chown vagrant:vagrant /results/nowcast-sys

    chown vagrant:vagrant /home/vagrant/nowcast
    chown vagrant:vagrant /home/vagrant/nowcast/www
    su vagrant -c 'cd /home/vagrant/nowcast \
        && ln -sf /results/nowcast-sys/tools/SalishSeaNowcast/nowcast/nowcast.yaml'
    su vagrant -c 'cd /home/vagrant/nowcast/www \
        && ln -sf /results/nowcast-sys/tools/SalishSeaNowcast/nowcast/www/templates'
  SHELL
end
