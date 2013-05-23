.. Hey, Emacs this is -*- rst -*-

   This file follows reStructuredText markup syntax; see
   http://docutils.sf.net/rst.html for more information.

.. include:: global.inc

.. _playbooks:
  
============================================
  Playbooks distributed with elasticluster  
============================================

After the requested number of Virtual Machines have been started,
elasticluster uses `Ansible`_ to configure them based on the
configuration options defined in the configuration file.

We distribute a few playbooks together with elasticluster to configure
some of the most wanted clusters. The playbooks are available at the
``share/elasticluster/providers/ansible-playbooks/`` directory inside
your virtualenv if you installed using `pip`_, or in the
``elasticluster/providers/ansible-playbooks`` directory of the github
source code. You can copy, customize and redistribute them freely
under the terms of the GPLv3 license.

A list of the most used playbooks distributed with elasticluster and
some explanation on how to use them follows.

Slurm
=====

Tested on:

* Ubuntu 12.04
* Ubuntu 13.04

+------------------+--------------------------------------+
| ansible groups   | role                                 |
+==================+======================================+
|``slurm_master``  | Act as scheduler and submission host |
+------------------+--------------------------------------+
|``slurm_clients`` | Act as compute node                  |
+------------------+--------------------------------------+

This playbook will install the `SLURM`_ queue manager using the
packages distributed with Ubuntu and will create a basic, working
configuration.

You are supposed to only define one ``slurm_master`` and multiple
``slurm_clients``. The first will act as login node and will run the
scheduler, while the others will only execute the jobs.

The ``/home`` filesystem is exported *from* the slurm server to the compute nodes.

A *snippet* of a typical configuration for a slurm cluster is::

    [cluster/slurm]
    frontend_nodes=1
    compute_nodes=5
    ssh_to=frontend
    setup_provider=ansible_slurm
    ...

    [setup/ansible_slurm]
    frontend_groups=slurm_master
    compute_groups=slurm_clients
    ...

You can combine the slurm playbooks with ganglia. In this case the ``setup`` stanza will look like::

    [setup/ansible_slurm]
    frontend_groups=slurm_master,ganglia_master
    compute_groups=slurm_clients,ganglia_monitor
    ...


Gridengine
==========

Tested on:
 
* Ubuntu 12.04
* CentOS 6.3

+-----------------------+--------------------------------------+
| ansible groups        | role                                 |
+=======================+======================================+
|``gridengine_master``  | Act as scheduler and submission host |
+-----------------------+--------------------------------------+
|``gridengine_clients`` | Act as compute node                  |
+-----------------------+--------------------------------------+

This playbook will install `Grid Engine`_ using the packages
distributed with Ubuntu or CentOS and will create a basic, working
configuration.

You are supposed to only define one ``gridengine_master`` and multiple
``gridengine_clients``. The first will act as login node and will run the
scheduler, while the others will only execute the jobs.

The ``/home`` filesystem is exported *from* the gridengine server to
the compute nodes. If you are running on a CentOS, also the
``/usr/share/gridengine/default/common`` directory is shared from the
gridengine server to the compute nodes.

A *snippet* of a typical configuration for a gridengine cluster is::

    [cluster/gridengine]
    frontend_nodes=1
    compute_nodes=5
    ssh_to=frontend
    setup_provider=ansible_gridengine
    ...

    [setup/ansible_gridengine]
    frontend_groups=gridengine_master
    compute_groups=gridengine_clients
    ...

You can combine the gridengine playbooks with ganglia. In this case the ``setup`` stanza will look like::

    [setup/ansible_gridengine]
    frontend_groups=gridengine_master,ganglia_master
    compute_groups=gridengine_clients,ganglia_monitor
    ...


Ganglia
=======

Tested on:

* Ubuntu 12.04
* CentOS 6.3

+--------------------+---------------------------------+
| ansible groups     | role                            |
+====================+=================================+
|``ganglia_master``  | Run gmetad and web interface.   |
|                    | It also run the monitor daemon. |
+--------------------+---------------------------------+
|``ganglia_monitor`` | Run ganglia monitor daemon.     |
+--------------------+---------------------------------+

This playbook will install `Ganglia`_ monitoring tool using the
packages distributed with Ubuntu or CentOS and will configure frontend
and monitors.

You should run only one ``ganglia_master``. This will install the
``gmetad`` daemon to collect all the metrics from the monitored nodes
and will also run apache.

If the machine in which you installed ``ganglia_master`` has IP
``10.2.3.4``, the ganglia web interface will be available at the
address http://10.2.3.4/ganglia/

This playbook is supposed to be compatible with all the other available playbooks.

Hadoop
======

Tested on:

* Ubuntu 12.04
* CentOS 6.3

+----------------------+-----------------------------------+
| ansible groups       | role                              |
+======================+===================================+
|``hadoop_namenode``   | Run the Hadoop NameNode service   |
+----------------------+-----------------------------------+
|``hadoop_jobtracker`` | Run the Hadoop JobTracker service |
+----------------------+-----------------------------------+
|``hadoop_datanode``   | Act as datanode for HDFS          |
+----------------------+-----------------------------------+
|``hadoop_tasktracker``| Act as tasktracker node accepting |
|                      | jobs from the JobTracker          |
+----------------------+-----------------------------------+

Hadoop playbook will install a basic hadoop cluster using the packages
available on the Hadoop website. The only supported version so far is
**1.1.2 x86_64** and it works both on CentOS and Ubuntu.

You must define only one ``hadoop_namenode`` and one
``hadoop_jobtracker``. Configuration in which both roles belong to the
same machines are not tested. You can mix ``hadoop_datanode`` and
``hadoop_tasktracker`` without problems though.

A *snippet* of a typical configuration for an Hadoop cluster is::

    [cluster/hadoop]
    hadoop-name_nodes=1
    hadoop-jobtracker_nodes=1
    hadoop-task-data_nodes=10
    ssh_to=hadoop-name
    ...

    [setup/ansible_hadoop]
    hadoop-name_groups=hadoop_namenode
    hadoop-jobtracker_groups=hadoop_jobtracker
    hadoop-task-data_groups=hadoop_tasktracker,hadoop_datanode
    ...