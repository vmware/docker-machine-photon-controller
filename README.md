<!--[metadata]>
+++
title = "Photon Controller Driver Plugin”
description = "Photon Controller driver for machine"
keywords = ["machine, Photon Controller, driver"]
+++
<![end-metadata]-->

# Photon Controller Driver Plugin

Photon Controller driver plugin is developed to create machines on [Photon Controller](http://vmware.github.io/photon-controller/). In order to create a docker machine you must supply parameters 
using the following command line / environment variable options.

## Options

-   `--photon-endpoint`: Your Photon Controller deployment endpoint. It includes both IP and protocol like: https://192.0.2.2
-   `--photon-project`: Your project Id already defined in the Photon Controller.
-   `--photon-vmflavor`: Your VM flavor already defined in Photon Controller.
-   `--photon-diskflavor`: Your disk flavor already defined in Photon Controller.
-   `--photon-image`: Your image Id for the image already uploaded in Photon Controller for consumption.
-   `--photon-diskname` : Your boot disk name. This is optional.
-   `--photon-bootdisksize`: Your boot disk size in GB.
-   `--photon-iso-path`: Your cloud-init ISO file path that is used to retrieve SSH keys to configure machine for SSH connection.
-   `--photon-ssh-keypath`: This is required if you have provided ISO path. This will be the path to the same SSH keys that are used in cloud-init.iso.
-   `--photon-ssh-user-password`: This is required if you want to enable SSH connection through password instead of SSH keys. Otherwise it is optional.
-   `--photon-ssh-user`: SSH user that will be used to connect the machine.

Environment variables and default values:

| CLI option                               | Environment variable       | Default          |
| ---------------------------------------- | -------------------------- | ---------------- |
| `--photon-endpoint`                      | `PHOTON_ENDPOINT`          | -                |
| `--photon-project`                       | `PHOTON_PROJECT`           | -                |
| `--photon-vmflavor`                      | `PHOTON_VM_FLAVOR`         | -                |
| `--photon-diskflavor`                    | `PHOTON_DISK_FLAVOR`       | -                |
| `--photon-image`                         | `PHOTON_IMAGE`             | -                |
| `--photon-diskname`                      | `PHOTON_DISK_NAME`         | `boot-disk`      |
| `--photon-bootdisksize`                  | `PHOTON_BOOT_DISK_SIZE`    | `2`              |
| `--photon-iso-path`                      | `PHOTON_ISO_PATH`          | -                |
| `--photon-ssh-keypath`                   | `PHOTON_SSH_KEYPATH`       | -                |
| `--photon-ssh-user-password`             | `PHOTON_SSH_USER_PASSWORD` | -                |
| `--photon-ssh-user`                      | `PHOTON_SSH_USER`          | `docker`         |

## Configuring setup data in Photon Controller

Before using the photoncontroller driver, ensure that you've configured the following setup data in Photon Controller.
Sample data here uses Photon Controller CLI (https://github.com/vmware/photon-controller-cli) to connect to and configure data in Photon Controller:

1.  Create a tenant in Photon Controller.

    Following sample command will create a tenant with name 'DockerTenant' and return the tenant Id as output.

    $ photon --non-interactive tenant create 'DockerTenant'

    DockerTenant	d468507d-467b-42e1-a8f5-ebb91f5ddbe5

2.  Create a resource ticket in Photon Controller.

    Following sample command will create a resource ticket named 'DockerRTicket' having the VM memory and count specs as specified and will return the resource ticket Id as output.

    $ photon --non-interactive resource-ticket create -n 'DockerRTicket' -t 'DockerTenant' -l 'vm.memory 2000 GB, vm 1000 COUNT'

    6228475f-f91f-49e3-a2d5-1e685cf4fe53

3.  Create a project in Photon Controller.

    Following sample command will create a project named 'DockerProject' associated with the tenant and resource ticket already defined.
    It will again specify the VM memory and count specs. Here we are assigning all the resources of a resource ticket to the same project but it can be split in multiple projects. It will return project Id as output.

    $ photon --non-interactive project create -n 'DockerProject' -t 'DockerTenant' -r 'DockerRTicket' -l 'vm.memory 2000 GB, vm 1000 COUNT'

    71a4dd83-763e-4d7e-91fe-8b7e1f6a0719

4.  Create a VM flavor in Photon Controller.

    Following command creates a VM flavor that layouts the specs for the VM that will be created using the flavor. It will also return VM flavor Id.

    $ photon --non-interactive flavor create -k 'vm' -n 'DockerFlavor' -c 'vm 1.0 COUNT, vm.flavor.core-100 1.0 COUNT, vm.cpu 1.0 COUNT, vm.memory 2.0 GB, vm.cost 1.0 COUNT'

    ec6e341c-d1a1-49dd-9162-24a6cf8f1d84

5.  Create a disk flavor in Photon Controller.

    Following command creates a disk flavor that layouts the specs for the disks that will be created using the flavor. It will also return disk flavor Id.

    $ photon --non-interactive flavor create -k 'ephemeral-disk' -n 'DockerDiskFlavor' -c 'ephemeral-disk 1.0 COUNT, ephemeral-disk.flavor.core-100 1.0 COUNT, ephemeral-disk.cost 1.0 COUNT'

    324890f2-7485-46a8-b8f9-14f12fb2e0e6

6.  Upload an image in Photon Controller.

    Following command uploads a Debian in Photon Controller that can be used to create VMs in future.
    You can download the sample Debian image from: https://s3.amazonaws.com/photon-platform/artifacts/OS/debian/debian-8.2.vmdk

    $ photon --non-interactive image create '/data/debian-8.2.vmdk' -n 'image-debian' -i 'EAGER'

    7b36947c-5387-404c-8123-2d71fd6baea7

7.  You have option to either provide the user password for SSH connection or to generate an ISO to attach with VM on start up to configure SSH keys for connection.

    Following is the process for generating ISO using the same sample Debian image as mentioned above:
    -   Create a user-data.txt file which lists SSH connection information to generate ISO from it. 

	 Sample: [user-data.txt](https://github.com/vmware/docker-machine-photon-controller/blob/master/sample/user-data.txt)

    -   Use mkisofs tool to generate ISO. It will take user-data.txt as input and generate ISO file for you to be used for SSH connections:

        $ mkisofs -rock -o cloud-init.iso user-data.txt

	You can install mkisofs by running following command:

	$ brew install cdrtools

Sample Environment Variables:

Following is the sample of environment variables that can be formed based on the data generated using the above sample Photon Controller CLI calls:

-   PHOTON_ENDPOINT = https://192.0.2.2
-   PHOTON_PROJECT = 71a4dd83-763e-4d7e-91fe-8b7e1f6a0719
-   PHOTON_DISK_FLAVOR = DockerDiskFlavor
-   PHOTON_IMAGE = 7b36947c-5387-404c-8123-2d71fd6baea7
-   PHOTON_VM_FLAVOR = DockerFlavor
-   PHOTON_ISO_PATH = /data/cloud-init.iso
-   PHOTON_SSH_KEYPATH=/data/id_rsa
