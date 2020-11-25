# Setting up workstation users

!!! Note

    If it's a new user, you'll need to send a ticket (incidencia) to IT for them to a) enable the user for Linux and b) create the shared home (specify both).

!!! Warning

    The cnio username ("xxxx" in "xxxx@cnio.es") should not exist locally on the machine or it will clash with the remote one.
    If you used the same username for the installation of the OS you will first need to remove it from the local machine.

## LDAP

### Install required packages (leave all options as default when prompted)

    $ sudo apt-get update
    $ sudo apt-get -y install libnss-ldap libpam-ldap ldap-utils nscd
  
### Update /etc/nsswitch.conf

    passwd:         compat ldap
    group:          compat ldap
    shadow:         compat ldap

### Replace contents in /etc/ldap.conf

Use the contents of [this file](https://gitlab.com/bu_cnio/bu_cnio.gitlab.io/-/snippets/1999595).

### Restart nscd service

    $ sudo service nscd restart

### Verify LDAP login
    $ getent passwd ldapuser

    ldapuser:x:9999:100:Test LdapUser:/home/ldapuser:/bin/bash

## CNIO shared homes

Home mounting was originally done with autofs. It does not play well with systemd and the long delays on network uplinks, so it's a bit of a mess in Ubuntu (tested on 18). The "classic" fstab mounting is therefore preferred.

!!! Warning

    The "classic" and the autofs mounts are mutually exclusive. You need to set up one or the other. Otherwise you'll have conflicts and problems will arise.

### Classic mount

#### Install nfs

Install nfs-common if not already available:

    apt-get install nfs-common

#### Edit /etc/fstab

    lando.cnio.es:/ifs/data/CNIO/Homes/<user> /home/<user> nfs  auto,noatime,nolock,bg,nfsvers=3,tcp,intr,_netdev,x-systemd.automount,x-systemd.after=network-online.target,x-systemd.device-timeout=240      0       0

### autofs

#### Install required packages

    $ sudo apt-get -y install autofs

#### Add a /home alias to /etc/auto.master (if it's not there yet)

    $ vi /etc/auto.master
    ...
    /home /etc/auto.home
    ...

#### Add the user to the home automount list

    $ vi /etc/auto.home
    ...
    username      lando.cnio.es:/ifs/data/CNIO/Homes/username
    ...
