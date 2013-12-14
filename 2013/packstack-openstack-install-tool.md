<!---
$"metadata"$
{
  "md": true,
  "title": "Packstack: Openstack Install tool",
  "draft": false,
  "slug": "packstack-openstack-install-tool",
  "tags": [
    "openstack",
    "packstack"
  ]
}
$"metadata"$
-->

For not a long time, a new tool that would help on getting Openstack installed either on bare metal or VMs, has been released. It's name is packstack and can be found on github under the fedora-openstack organization.

Packstack is currently in its first ages and still under heavy development but, it's already capable of installing, customize some parameters and distribute most of the Openstack modules on a single server or several as well. It currently supports Red Hat based distros but there's space for more.

Architecture
============

Packstack's architecture is based on weighted plugins, which are, python's modules packaged together with a weight in their names that gives them a priority over other modules. Each module is isolated from others and installs, configures or applies any Openstack's required dependency for modules marked for installation.

Packstack doesn't require any specific configuration but, it can read a so called answer file that species the configuration parameters for the given install process. Here's a quick overview of packstack's architecture:

Packstack -> Modules -> puppet

As you might have noticed, packstack uses puppet as the main provisioning tool and applies the required manifests for a given module to the destination host.

Hands on
========


Getting / Installing packstack
------------------------------

    $ git clone --branch folsom --recursive https://github.com/stackforge/packstack.git

    $ cd packstack

    $ python setup.py install


Using packstack
---------------

    $ packstack --gen-answer-file=<destination_path> #Review answers file


    $ packstack --answer-file=<path_to_answer_file>


Tunning answers file
--------------------

Even though Packstack supports live mode execution, I prefer and suggest to generate an answer file and tune it before provisioning. In the answer file, it is possible to change any exported parameter such like network interfaces, float ip classes, connection parameters and, may be one of the most important, enable / disable modules.

Answers file is well commented and allow users for configuring some of the most important parameters for Openstack to run well.

Troubleshooting
===============

Packstack applies every manifest on the destination server, which means, that logs for any issues can be found on the issued server under packstack's working directory `/var/tmp/packstack/%YYYY-%M-%d` and there's also a local log file under `/var/tmp/packstack/%YYYY-%M-%d/openstack-setup_2013_01_24_17_48_07.log`.

Remember, If something goes wrong or you find a bug, please, file a new report <a href="https://bugzilla.redhat.com/enter_bug.cgi?product=Red%20Hat%20OpenStack">here</a>


Future
======

* `sudo` support

* Quantum support

* More distros (Contribute back, plsssssssssss)

