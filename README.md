# ansible-gentoo

This playbook demonstrates the automatic configuration of a gentoo system from
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

# Role Variables

The role defines several defaults that an be overridden by the user

* `disk`: dictionary with information of the disks layout
  * `main`: Path to the main device (Default: `/dev/sda`)
  * `main_size`: Size of the main partition (Default: `100%`)
  * `swap_size`: Size of the swap partition (Default: `4GiB`)
  * `filesystem`: Filesystem of the main partition (Default: `ext4`)
  * `encrypt_with_luks`: Encrypt with LUKS the main device and swap (Default:
    `true`) **You have to change this**
* `mirror`: The mirror to fetch the gentoo stage3 sources from.(Default:
  http://mirror.mdfnet.se/gentoo)


* `portage`: Dictionary with information of portage
  For more information setting this flags, read the
  [PORTDIR](https://wiki.gentoo.org/wiki//etc/portage/make.conf#PORTDIR) article
  * `cflags`: Optimization flags for GCC C (Default: `-march=native -O2 -pipe`)
    Although the [GCC
    optimization](https://wiki.gentoo.org/wiki/GCC_optimization) article has
    more information on how the various compilation options can affect a system,
    the Safe [CFLAGS](https://wiki.gentoo.org/wiki/Safe_CFLAGS) article may be
    a more practical place for beginners to start optimizing their systems.
  * `cxxflags`: Optimization flags for GCC C++ (Default: `${CFLAGS}`)
  * `chost`: Tells portage which platform code should be built for (Default:
    `x86_64-pc-linux-gnu`). Should not be changed lightly, for more information
    check [this](http://www.gentoo.org/doc/en/change-chost.xml)
  * `use`: Default use flags (Default: `bindist dbus alsa -X -gnome -dvd -cdr
    -ipv6 -systemd -gtk -gtk3`)
  * `cpu_flags_x86`: Informs Portage about the CPU flags (features) permitted by
    the CPU (Default: `mmx sse sse2 mmxext`)
  * `makeopts`: Used to specify arguments passed to make when packages are built
    from source (Default: `-j{{ ansible_processor_count + 1 }}`)
  * `features`: Contains a list of Portage features that the user wants enabled
    on the system, effectively influencing Portage's behavior (Default: `sandbox
    parallel-fetch candy userfetch webrsync-gpg`).

    For more information, please see [Portage
    features](https://wiki.gentoo.org/wiki/Handbook:AMD64/Working/Features) in
    the Gentoo Handbook and the
    [FEATURES](https://wiki.gentoo.org/wiki/FEATURES) article. For a complete
    list of available features, see man 5 make.conf.
  * `portage_gpg_dir`: Directory where the gpg keys are stored to verify the
    downloaded trees (Default: `/var/lib/gentoo/gkeys/keyrings/gentoo/release`)
* `timezone`: The timezone to set for the new system  (Default:
  `Europe/Brussels`)
* `hostname`: Hostname for the new system (Default: `create-a-super-nice-name`)
  **You have to change this**

pubkey
    The public key to deploy to the root user on the *minimal cd*. Primarily to
    negate the need for a password if you need to run twice.
    default: ~/.ssh/id_rsa.pub


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

## Stage3
* Correct the date with `ntpd`
* Download the stage3
* Verify it's integrity with gpg and sha512
* Configure the verification of the portage tree with gpg
* Configure compile options (`/etc/portage/make.conf`)

## Base system unchrooted
* Mount the required mountpoints

## Base system
* Update the `@world`

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
