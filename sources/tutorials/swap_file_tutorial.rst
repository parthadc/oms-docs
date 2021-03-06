:title: Swap File Tutorial
:description: Tutorial for setting up a swap file
:keywords: swap, file, development, developer,


.. _swap_file_tutorial:

Swap File Tutorial
==================

This tutorial is a guide for using swap on your Open Mustard Seed (OMS) host.
Using swap can improve system performance, especially in low memory
environments.

.. note::

  This tutorial should be used on either the :ref:`importable virtual
  appliance <deploy_development_vm>` or an :ref:`OMS Host in the Cloud
  <kickstart_oms>`.


Introduction
------------

A swap file is a file that can be used to improve system performance by freeing
up RAM. In this scheme, the kernel temporarily moves unused memory pages into
the swap file, making room for more actively used ones.

The kernel's "swappiness" (tendency to use swap) can be configured.


Setting up the swap file
------------------------

OMS includes rudimentary support for swap files, with a utility helper that
simplifies creating swapfiles.

Let's first step through the manual process, and you can choose what will work
best for the specifics of your situation.


Manual Setup
~~~~~~~~~~~~

Start by determining the appropriate size for your swap file.  A common
approach is to use 1.5X or 2X of your machine's RAM size.

To create an empty 1 GB file for swap use, use ``dd`` with 1,048,576 blocks
of size 1,024 (1,048,576 x 1,024 = 1,073,741,824):


.. code:: bash

  oms% dd if=/dev/zero of=/var/1GB.swap bs=1024 count=1048576
  1048576+0 records in
  1048576+0 records out
  1073741824 bytes (1.1 GB) copied, 10.3472 s, 104 MB/s


Next, make sure the file's permissions (mode) are locked down:

.. code:: bash

  oms% chmod 600 /var/1GB.swap


Convert the file into a swap file:

.. code:: bash

  oms% mkswap /var/1GB.swap
  Setting up swapspace version 1, size = 1048572 KiB
  no label, UUID=3b3585aa-08cb-4820-8d39-b2cbb74c7493


Enable the file for swapping:

.. code:: bash

  oms% swapon /var/1GB.swap


Verify that the swap file is enabled:

.. code:: bash

  oms% swapon -s
  Filename				Type		Size	Used	Priority
  /dev/xvdc1				partition	499996	56	-1
  /var/1GB.swap				file		1048572	0	-2


You can make the use of the swap file persist across reboots by updating the
filesystem table.

``/etc/fstab``:

.. code::

  # <file system> <mount point>   <type>  <options>       <dump>  <pass>
  # ...
  /var/1GB.swap   none            swap    sw              0       0


Automated
~~~~~~~~~

If you are bootstrapping a new host with oms-kickstart, the default build will
include creating a 1GB swap file (and enabling it) for you.

The basic OMS Root VRC, installed onto an OMS Host, includes a formula which
can be applied as so:

.. code::

   # salt-call --local state.sls swap


This will create a script ``/root/mkswap.sh`` that will run through the manual
process detailed above. It is simple, but functional. You can also edit the
script if you wish to change the size of the swapfile.

.. todo::

   We should update the formula, to update the script, to use a bash variable
   to define the size of the swap file to use. but then we might introduce
   command injection vulnerability :(


Adjusting kernel swappiness
---------------------------

Kernel swappiness can be configured, with a range of 0 (low) to 100 (high). A
setting of 60 is typical. You can view and set swappiness as follows:

.. code:: bash

  oms% sysctl vm.swappiness
  vm.swappiness = 10
  oms% sysctl vm.swappiness=25
  vm.swappiness = 25


You can make this setting persist across reboots by adding it to a
configuration file:

``/etc/sysctl.conf``: 

.. code::

  # ...
  vm.swappiness = 25


These values can be set higher on physical systems. For VMs, it is recommended
to set swappiness with smaller, incremental updates, and performance evaluations
between.
