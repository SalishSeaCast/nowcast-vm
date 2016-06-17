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
    apt-get install -y sshfs

    mkdir -p /data && chown vagrant:vagrant /data
    mkdir -p /ocean && chown vagrant:vagrant /ocean

    chown vagrant:vagrant /results
    chown vagrant:vagrant /results/nowcast-sys

    chown vagrant:vagrant /home/vagrant/nowcast
    chown vagrant:vagrant /home/vagrant/nowcast/www
    su vagrant -c ' \
        cd /home/vagrant/nowcast \
        && ln -sf /results/nowcast-sys/tools/SalishSeaNowcast/nowcast/nowcast.yaml \
    '
    su vagrant -c ' \
        cd /home/vagrant/nowcast/www \
        && ln -sf /results/nowcast-sys/tools/SalishSeaNowcast/nowcast/www/templates \
    '

    DEV_HOME=/home/vagrant
    MINICONDA_INSTALLER=http://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
    if [ -d $DEV_HOME/miniconda ]; then
      echo "Miniconda already installed"
    else
      echo "Downloading Miniconda installer from continuum.io"
      su vagrant -c " \
        wget -q $MINICONDA_INSTALLER -O $DEV_HOME/miniconda.sh \
      "
      echo "Installing $DEV_HOME/miniconda"
      su vagrant -c " \
        bash $DEV_HOME/miniconda.sh -b -p $DEV_HOME/miniconda \
      "
    fi

    CONDA_BIN=$DEV_HOME/miniconda/bin
    touch $DEV_HOME/.bash_aliases && chown vagrant:vagrant $DEV_HOME/.bash_aliases
    su vagrant -c " \
      echo export PATH=$CONDA_BIN:'$PATH' > $DEV_HOME/.bash_aliases \
    "
  SHELL
end
