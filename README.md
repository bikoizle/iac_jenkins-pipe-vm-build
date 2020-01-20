VM build
========

# Description

This Jenkins Pipe is used for automated Openstack Instance creation.

# Parameters

The following parameters are mandatory in order to launch the pipeline:

ANSIBLE_ROLE_NAME: Role to apply when creating the virtual machine.
GIT_ANSIBLE_ROLE_TAG: Git tag with the version of the role to download.
OS_IMAGE_NAME: Image name to use when building the virtual machine.
OS_VM_NAME: Virtual machine name.
OS_VM_VOL_SIZE: Virtual machine volume size in GB.
OS_VM_FLAVOUR: Virtual machine configuration flavour.
