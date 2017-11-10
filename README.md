# ansible-gentoo

This role demonstrates the automatic configuration of a gentoo system from
minimal install to first boot.

First record the gentoo iso in your USB, connect it to the desired device and
[configure the network](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Networking)

You should also shred the `main_disk` couldn't add this task due to idempotence
problems

```bash
shred -uv {{ main_disk }}
```

# Disclaimer

This role is not tested for `disk.encrypt_with_luks == false` so it won't work,
the path is open for you to make a Merge request :)

Role Variables
--------------

The role defines several defaults that an be overridden by the user

* `disk`: dictionary with information of the disks layout
  * `main`: Path to the main device (Default: `/dev/sda`)
  * `main_size`: Size of the main partition (Default: `100%`)
  * `swap_size`: Size of the swap partition (Default: `4GiB`)
  * `filesystem`: Filesystem of the main partition (Default: `ext4`)
  * `encrypt_with_luks`: Encrypt with LUKS the main device and swap (Default:
    `true`)

pubkey
    The public key to deploy to the root user on the *minimal cd*. Primarily to
    negate the need for a password if you need to run twice.
    default: ~/.ssh/id_rsa.pub

mirror
    The mirror to fetch the gentoo stage3 sources from.
    default: http://mirror.mcs.anl.gov/pub/gentoo

timezone
    The timezone to set for the new system.
    default: America/Los_Angeles

domain
    main domain for the new system
    default: met.tfoundry.com

kernel
    The kernel configuration to use.
    default: config-3.13.0-24-generic

make_opts
    any make options you wish to pass for ebuilds.
    default: -j4

management_interface
    the main interface to configure
    default: ansible_default_ipv4.interface

root_passwd
    the initial root password for the newly installed system.
    default: gentoo-root

gateway
    the default gateway for the newly installed system.
    default: ansible_default_ipv4.gateway

netmask
    the netmask for the newly installed system
    default: ansible_default_ipv4.netmask


# What it does

## Storage
* Configure the disks layout
* Create a boot MBR partition
* Encrypt the main partition with LUKS
* Backup the LUKS header in the boot partition
* Create an LVM partition for the main filesystem
* Create an LVM partition for swap
* Format the LVM partitions
* Mount them

##
Example Playbook
----------------

Including an example of how to use your role (for instance, with variables
passed in as parameters) is always nice for users too:

.. code:: yaml

    - hosts: all
      roles:
         - gentoo

Also please see  `blog post`_  for an example of using this role to deploy
gentoo.

License
-------

BSD

Author Information
------------------

James A. Kyle
james@jameskyle.org
http://blog.jameskyle.org

.. _`blog post`: http://blog.jameskyle.org/2014/08/automated-stage3-gentoo-install-using-ansible/
