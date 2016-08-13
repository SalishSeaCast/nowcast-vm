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
    nowcast.vm.network :forwarded_port, guest: 80, host: 4567

    config.ssh.forward_agent = true
  end

  config.vm.synced_folder "../tools/", "/results/nowcast-sys/tools",
    create: true
  config.vm.synced_folder "../NEMO-forcing/", "/results/nowcast-sys/NEMO-forcing",
    create: true
  config.vm.synced_folder "../salishsea-site/",
    "/home/vagrant/nowcast/www/salishsea_site",
    create: true

  # Provisioning
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update

    TIMEZONE=Canada/Pacific
    echo "Set timezone to ${TIMEZONE}"
    timedatectl set-timezone ${TIMEZONE}

    apt-get install -y mg
    apt-get install -y sshfs
    apt-get install -y apache2 libapache2-mod-proxy-html libxml2-dev

    mkdir -p /data && chown vagrant:vagrant /data
    mkdir -p /ocean && chown vagrant:vagrant /ocean
    mkdir -p /var/www/html && chgrp vagrant /var/www/html && chmod 775 /var/www/html

    cat << EOF > /etc/apache2/sites-available/salishsea.eos.ubc.ca.conf
<VirtualHost *:80>
    ServerAdmin admin@salishsea.eos.ubc.ca
    ServerName localhost
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    ProxyRequests Off
    ProxyPreserveHost On
    ProxyPass / http://0.0.0.0:6543/
    ProxyPassReverse / http://0.0.0.0:6543/
</VirtualHost>
EOF
    /usr/sbin/a2enmod proxy_http rewrite cache headers
    /usr/sbin/a2ensite salishsea.eos.ubc.ca.conf
    /usr/sbin/a2dissite 000-default.conf
    service apache2 restart

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

    VAGRANT_HOME=/home/vagrant
    MINICONDA_INSTALLER=http://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
    if [ -d $VAGRANT_HOME/miniconda ]; then
      echo "Miniconda already installed"
    else
      echo "Downloading Miniconda installer from continuum.io"
      su vagrant -c " \
        wget -q $MINICONDA_INSTALLER -O $VAGRANT_HOME/miniconda.sh \
      "
      echo "Installing $VAGRANT_HOME/miniconda"
      su vagrant -c " \
        bash $VAGRANT_HOME/miniconda.sh -b -p $VAGRANT_HOME/miniconda \
      "
    fi

    CONDA_BIN=$VAGRANT_HOME/miniconda/bin
    touch $VAGRANT_HOME/.bash_aliases && chown vagrant:vagrant $VAGRANT_HOME/.bash_aliases
    su vagrant -c " \
      echo export PATH=$CONDA_BIN:\$PATH > $VAGRANT_HOME/.bash_aliases \
    "

    CONDA=$CONDA_BIN/conda
    NOWCAST_SYS=/results/nowcast-sys
    NOWCAST_ENV=$NOWCAST_SYS/nowcast-env
    PIP=$NOWCAST_ENV/bin/pip
    if [ -d $NOWCAST_ENV ]; then
      echo "$NOWCAST_ENV conda env already exists"
    else
      echo "Creating $NOWCAST_ENV conda env"
      su vagrant -c " \
        $CONDA create --yes --prefix $NOWCAST_ENV \
            bottleneck \
            lxml \
            mako \
            matplotlib \
            netCDF4 \
            numpy \
            pandas \
            paramiko \
            pillow \
            pyyaml \
            pyzmq \
            pip \
            python=3 \
            requests \
            scipy \
            sphinx \
            xarray \
      "
      echo "Installing pip packages into $NOWCAST_ENV conda env"
      su vagrant -c " \
        $PIP install \
          angles \
          arrow \
          BeautifulSoup4 \
          driftwood \
          feedgen \
          retrying \
          sphinx-bootstrap-theme \
        "
      echo "Installing editable tools/SalishSea* packages into $NOWCAST_ENV conda env"
      su vagrant -c " \
        $PIP install --editable $NOWCAST_SYS/tools/SalishSeaTools/ \
        && $PIP install --editable $NOWCAST_SYS/tools/SalishSeaCmd/ \
        && $PIP install --editable $NOWCAST_SYS/tools/SalishSeaNowcast/ \
      "
    fi

    su vagrant -c " \
      echo source activate $NOWCAST_ENV >> $VAGRANT_HOME/.bash_aliases \
    "

    SALISHSEA_SITE_ENV=$NOWCAST_SYS/salishsea-site-env
    PIP=$SALISHSEA_SITE_ENV/bin/pip
    if [ -d $SALISHSEA_SITE_ENV ]; then
      echo "$SALISHSEA_SITE_ENV conda env already exists"
    else
      echo "Creating $SALISHSEA_SITE_ENV conda env"
      su vagrant -c " \
        $CONDA create --yes --prefix $SALISHSEA_SITE_ENV \
            pip \
            python=3 \
      "
      echo "Installing pip packages into $SALISHSEA_SITE_ENV conda env"
      su vagrant -c " \
        $PIP install \
          pyramid \
          pyramid-crow \
          pyramid-debugtoolbar \
          pyramid_mako \
          waitress \
        "
    fi
  SHELL
end
