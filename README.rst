************************************
Salish Sea Nowcast System Vagrant VM
************************************

This repo contains a `Vagrant`_ virtual machine configuration that emulates
the Salish Sea Nowcast system deployment on skookum.eos.ubc.ca.
The VM is intended for development and testing of elements of the nowcast system.
The default Vagrant ``virtualbox`` provider is used,
so it should be possible to use this VM configuration on any machine with
Vagrant installed and sufficient resources to run a 512 Mb Ubuntu 14.04 LTS
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

The local ``../tools/`` directory is mounted as a ``synced_folder`` at
``/results/nowcast-sys/tools/``.
That is done on the assumption that you have cloned this repo beside your
``tools/`` repo.

``sshfs`` is installed on the VM and ssh agent forwarding is enabled
so that directories from ``skookum`` can be mounted,
for example::

  sshfs dlatorne@skookum.eos.ubc.ca:/data /data

will use the default ssh key for `dlatorne`` to mount the `/data/` directory
on ``skookum`` as ``/data/`` on the VM.
