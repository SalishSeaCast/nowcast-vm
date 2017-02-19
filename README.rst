************************************
Salish Sea Nowcast System Vagrant VM
************************************

This repo contains a `Vagrant`_ virtual machine configuration that emulates
the Salish Sea Nowcast system deployment on skookum.eos.ubc.ca.
The VM is intended for development and testing of elements of the nowcast system.
The default Vagrant ``virtualbox`` provider is used,
so it should be possible to use this VM configuration on any machine with
Vagrant installed and sufficient resources to run a 1024 Mb Ubuntu 14.04 LTS
VM.
The VM has been extensively used on 2015-vintage laptops running Ubuntu 16.04.

.. _Vagrant: https://www.vagrantup.com/

The VM is configured so that it can be used the ``vagrant`` user,
the default user when you ``vagrant ssh`` into the VM.


Provisioning
============

Directories similar to those on ``skookum`` are created:

* ``/data/``
* ``/ocean/``
* ``/results/``
* ``/var/www/html``
* ``$HOME/nowcast`` (in place of ``/home/dlatorne/public_html/MEOPAR/nowcast/``)

The Vagrant ``synced_folder`` feature is used to mount local directories
on the VM:

* the local ``../tools/`` directory is mounted at
``/results/nowcast-sys/tools/``
* the local ``../salishsea-site/`` directory is mounted at ``$HOME/nowcast/www/salishsea-site/``

That is done on the assumption that you have cloned this repo beside your
``tools/`` repo.

``sshfs`` is installed on the VM and ssh agent forwarding is enabled
so that directories from ``skookum`` can be mounted,
for example::

  sshfs dlatorne@skookum.eos.ubc.ca:/data /data

will use the default ssh key for ``dlatorne`` to mount the ``/data/`` directory
on ``skookum`` as ``/data/`` on the VM.

`Miniconda`_ is installed in ``/home/vagrant/miniconda/``,
and ``/home/vagrant/miniconda/bin`` is prefixed to ``PATH`` in ``/home/vagrant/.bash_aliases``.

.. _Miniconda: http://conda.pydata.org/miniconda.html

The `SalishSeaTools`_, `SalishSeaCmd`_, and `SalishSeaNowcast`_ packages are
installed in the ``/results/nowcast-sys/nowcast-env`` conda environment.

.. _SalishSeaTools: http://salishsea-meopar-tools.readthedocs.io/en/latest/SalishSeaTools/index.html
.. _SalishSeaCmd: http://salishsea-meopar-tools.readthedocs.io/en/latest/SalishSeaCmd/index.html
.. _SalishSeaNowcast: http://salishsea-meopar-tools.readthedocs.io/en/latest/SalishSeaNowcast/index.html

The ``/results/nowcast-sys/nowcast-env`` conda environment is activate by default
on ``vagrant ssh`` login via a ``source activate /results/nowcast-sys/nowcast-env``
command  in ``/home/vagrant/.bash_aliases``.
