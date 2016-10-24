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
    nowcast.vm.provider "virtualbox" do |v|
      v.memory = 1024
    end

    nowcast.vm.network :forwarded_port, guest: 80, host: 4567

    config.ssh.forward_agent = true
  end

  config.vm.synced_folder "../SalishSeaNowcast", "/results/nowcast-sys/SalishSeaNowcast",
    create: true
  config.vm.synced_folder "../NEMO_Nowcast", "/results/nowcast-sys/NEMO_Nowcast",
    create: true
  config.vm.synced_folder ".", "/results/nowcast-sys/salishsea_site",
    create: true
  config.vm.synced_folder "../salishsea_site", "/home/vagrant/nowcast/www/salishsea_site",
    create: true
  config.vm.synced_folder "../tools/", "/results/nowcast-sys/tools",
    create: true
  config.vm.synced_folder "../private-tools/", "/results/nowcast-sys/private-tools",
    create: true
  config.vm.synced_folder "../NEMO-forcing/", "/results/nowcast-sys/NEMO-forcing",
    create: true
  config.vm.synced_folder "../SS-run-sets/", "/results/nowcast-sys/SS-run-sets",
    create: true
  config.vm.synced_folder "../NEMO-3.6-code/", "/results/nowcast-sys/NEMO-3.6-code",
    create: true
  config.vm.synced_folder "../XIOS/", "/results/nowcast-sys/XIOS",
    create: true

  # Provisioning
  config.vm.provision "shell", inline: <<-SHELL
    add-apt-repository -y ppa:mercurial-ppa/releases
    apt-get update

    TIMEZONE=Canada/Pacific
    echo "Set timezone to ${TIMEZONE}"
    timedatectl set-timezone ${TIMEZONE}

    apt-get install -y mg
    apt-get install -y sshfs
    apt-get install -y apache2 libapache2-mod-proxy-html libxml2-dev
    apt-get install -y gfortran nco mercurial

    mkdir -p /data && chown vagrant:vagrant /data
    mkdir -p /ocean && chown vagrant:vagrant /ocean
    mkdir -p /var/www/html && chgrp vagrant /var/www/html && chmod 775 /var/www/html

    addgroup sallen
    adduser vagrant sallen

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
    su vagrant -c ' \
      mkdir -p /results/forcing/atmospheric/GEM2.5/GRIB/ \
      mkdir -p /results/forcing/atmospheric/GEM2.5/operational/fcst \
      mkdir -p /results/forcing/rivers \
      mkdir -p /results/forcing/sshNeahBay/fcst \
      mkdir -p /results/forcing/sshNeahBay/obs \
      mkdir -p /results/forcing/sshNeahBay/txt \
      mkdir -p /results/SalishSea/nowcast \
      mkdir -p /results/SalishSea/nowcast-green \
      mkdir -p /results/SalishSea/forecast \
      mkdir -p /results/SalishSea/forecast2 \
      mkdir -p /results/nowcast-sys/runs \
      mkdir -p /results/nowcast-sys/figures/nowcast \
      mkdir -p /results/nowcast-sys/figures/nowcast-green \
      mkdir -p /results/nowcast-sys/figures/forecast \
      mkdir -p /results/nowcast-sys/figures/forecast2 \
      mkdir -p /results/nowcast-sys/figures/storm-surge \
    '

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


 ########################################################
 # Environment for nowcast-v2 (tools/SalishSeaNowcast)
 ########################################################
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

    # su vagrant -c " \
    #   echo source activate $NOWCAST_ENV >> $VAGRANT_HOME/.bash_aliases \
    # "

    su vagrant -c " \
      mkdir -p /results/observations/ONC/CTD/SCVIP \
    "

    echo "Setting up ${NOWCAST_ENV} activate/deactivate hooks that export/unset environment variables"
    su vagrant -c " \
      mkdir -p ${NOWCAST_ENV}/etc/conda/activate.d \
      && cat << EOF > ${NOWCAST_ENV}/etc/conda/activate.d/envvars.sh
export ONC_USER_TOKEN=
export SENTRY_DSN=
EOF"
    su vagrant -c " \
      mkdir -p ${NOWCAST_ENV}/etc/conda/deactivate.d \
      && cat << EOF > ${NOWCAST_ENV}/etc/conda/deactivate.d/envvars.sh
unset ONC_USER_TOKEN
unset SENTRY_DSN
EOF"


 ########################################################
 # Environment for salishsea-site Pyramid app
 ########################################################
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
          pyyaml \
      "
      echo "Installing pip packages into $SALISHSEA_SITE_ENV conda env"
      su vagrant -c " \
        $PIP install \
          chaussette \
          circus \
          pyramid \
          pyramid-crow \
          pyramid-debugtoolbar \
          pyramid_mako \
          waitress \
        "
    fi

    # su vagrant -c " \
    #   echo source activate $SALISHSEA_SITE_ENV >> $VAGRANT_HOME/.bash_aliases \
    # "

    su vagrant -c " \
      mkdir -p /results/nowcast-sys/logs/salishsea-site \
    "

    echo "Setting up ${SALISHSEA_SITE_ENV} activate/deactivate hooks that export/unset environment variables"
    su vagrant -c " \
      mkdir -p ${SALISHSEA_SITE_ENV}/etc/conda/activate.d \
      && cat << EOF > ${SALISHSEA_SITE_ENV}/etc/conda/activate.d/envvars.sh
export SALISHSEA_SITE_ENV=${SALISHSEA_SITE_ENV}
export SALISHSEA_SITE=/home/vagrant/nowcast/www/salishsea_site
export SENTRY_DSN=
EOF"
    su vagrant -c " \
      mkdir -p ${SALISHSEA_SITE_ENV}/etc/conda/deactivate.d \
      && cat << EOF > ${SALISHSEA_SITE_ENV}/etc/conda/deactivate.d/envvars.sh
unset SALISHSEA_SITE_ENV
unset SALISHSEA_SITE
unset SENTRY_DSN
EOF"


 ################################################################
 # Environment for nowcast-v3 (NEMO_Nowcast & SalishSeaNowcast)
 ################################################################
    NEMO_NOWCAST_ENV=${NOWCAST_SYS}/nemo_nowcast-env
    PIP=${NEMO_NOWCAST_ENV}/bin/pip
    if [ -d ${NEMO_NOWCAST_ENV} ]; then
      echo "${NEMO_NOWCAST_ENV} conda env already exists"
    else
      echo "Creating ${NEMO_NOWCAST_ENV} conda env"
      su vagrant -c " \
        $CONDA create --yes \
          --channel gomss-nowcast --channel defaults --channel conda-forge \
          --prefix ${NEMO_NOWCAST_ENV} \
          arrow \
          attrs \
          beautifulsoup4 \
          bottleneck \
          circus \
          cliff \
          matplotlib \
          netcdf4 \
          numpy \
          pandas \
          paramiko \
          pillow \
          pip \
          python=3 \
          pyyaml \
          pyzmq \
          retrying \
          requests \
          schedule \
          scipy \
          xarray \
          \
          coverage \
          pytest \
          sphinx \
      "
      echo "Installing pip packages into ${NEMO_NOWCAST_ENV} conda env"
      su vagrant -c " \
        ${PIP} install \
          angles \
          driftwood \
          raven \
          \
          sphinx-rtd-theme \
        "
      echo "Installing editable NEMO_Nowcast, SalishSeaTools & SalishSeaNowcast packages into ${NEMO_NOWCAST_ENV} conda env"
      su vagrant -c " \
        ${PIP} install --editable ${NOWCAST_SYS}/NEMO_Nowcast/ \
        && ${PIP} install --editable ${NOWCAST_SYS}/tools/SalishSeaTools/ \
        && $PIP install --editable $NOWCAST_SYS/tools/SalishSeaCmd/ \
        && ${PIP} install --editable ${NOWCAST_SYS}/SalishSeaNowcast/ \
      "
    fi

    su vagrant -c " \
      echo source activate ${NEMO_NOWCAST_ENV} >> ${VAGRANT_HOME}/.bash_aliases \
    "

    echo "Setting up ~/.ssh/config"
    su vagrant -c "\
      cat << EOF > $HOME/.ssh/config \
Host orcinus-nowcast
  HostName     orcinus.westgrid.ca
  User         dlatorne
  IdentityFile    /home/dlatorne/.ssh/SalishSeaNEMO-nowcast_id_rsa
  ForwardAgent no
EOF"
    echo "Don't forget to install the SalishSeaNEMO-nowcast_id_rsa key pair in /home/dlatorne/.ssh/"

    echo "Setting up ${NOWCAST_SYS}/runs/ directory"
    su vagrant -c " \
      cp ${NOWCAST_SYS}/SS-run-sets/SalishSea/nemo3.6/namelist.time ${NOWCAST_SYS}/runs/ \
      ln -s ${NOWCAST_SYS}/SS-run-sets/SalishSea/nemo3.6/nowcast/iodef.xml ${NOWCAST_SYS}/runs/iodef.xml \
    "

    echo "Setting up mock nemo.exe and xios_server.exe executables"
    su vagrant -c " \
      mkdir -p ${NOWCAST_SYS}/NEMO-3.6-code/NEMOGCM/CONFIG/SOG/BLD/bin/ \
      touch ${NOWCAST_SYS}/NEMO-3.6-code/NEMOGCM/CONFIG/SOG/BLD/bin/nemo.exe \
      mkdir -p ${NOWCAST_SYS}/XIOS/bin/ \
      touch ${NOWCAST_SYS}/XIOS/bin/xios_server.exe \
    "

    NOWCAST_CONFIG=${NOWCAST_SYS}/SalishSeaNowcast/config
    NOWCAST_YAML=${NOWCAST_CONFIG}/nowcast.yaml

    NOWCAST_LOGS=${NOWCAST_SYS}/logs/nowcast
    mkdir -p ${NOWCAST_LOGS} && chown vagrant:vagrant ${NOWCAST_LOGS}

    echo "Setting up ${NEMO_NOWCAST_ENV} activate/deactivate hooks that export/unset environment variables"
    su vagrant -c " \
      mkdir -p ${NEMO_NOWCAST_ENV}/etc/conda/activate.d \
      && cat << EOF > ${NEMO_NOWCAST_ENV}/etc/conda/activate.d/envvars.sh
export NOWCAST_ENV=${NEMO_NOWCAST_ENV}
export NOWCAST_CONFIG=${NOWCAST_CONFIG}
export NOWCAST_YAML=${NOWCAST_YAML}
export NOWCAST_LOGS=${NOWCAST_LOGS}
export ONC_USER_TOKEN=
export SENTRY_DSN=
EOF"
    su vagrant -c " \
      mkdir -p ${NEMO_NOWCAST_ENV}/etc/conda/deactivate.d \
      && cat << EOF > ${NEMO_NOWCAST_ENV}/etc/conda/deactivate.d/envvars.sh
unset NOWCAST_ENV
unset NOWCAST_CONFIG
unset NOWCAST_YAML
unset NOWCAST_LOGS
unset ONC_USER_TOKEN
unset SENTRY_DSN
EOF"

  SHELL
end
