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

```
HOST cnio.es
#***** HOST seth.cnio.es

#BASE DC=cnio,DC=es
BASE OU=Usuarios,OU=Programa Biologia Estructural,OU=CNIO.ES,DC=cnio,DC=es

binddn cn=bioldap,cn=Users,dc=cnio,dc=es
bindpw #####REDACTED######

nss_base_passwd OU=Usuarios,OU=Programa Biologia Estructural,OU=CNIO.ES,DC=cnio,DC=es?sub
nss_base_passwd OU=Usuarios,OU=Programa Biotecnologia,OU=CNIO.ES,DC=cnio,DC=es
nss_base_passwd OU=Usuarios,OU=Programa Terapias Experimentales,OU=CNIO.ES,DC=cnio,DC=es
nss_base_passwd DC=cnio,DC=es?sub
nss_base_shadow OU=Usuarios,OU=Programa Biologia Estructural,OU=CNIO.ES,DC=cnio,DC=es?sub
nss_base_shadow OU=Usuarios,OU=Programa Biotecnologia,OU=CNIO.ES,DC=cnio,DC=es
nss_base_shadow OU=Usuarios,OU=Programa Terapias Experimentales,OU=CNIO.ES,DC=cnio,DC=es
nss_base_shadow DC=cnio,DC=es?sub
nss_base_group OU=PBE,OU=ACLs,DC=cnio,DC=es?sub
nss_map_objectclass posixAccount User
nss_map_objectclass shadowAccount User
nss_map_attribute uid sAMAccountName
nss_map_attribute uidNumber msSFU30UidNumber
nss_map_attribute gidNumber msSFU30GidNumber
nss_map_attribute homeDirectory msSFU30HomeDirectory
nss_map_attribute loginShell msSFU30LoginShell
nss_map_objectclass posixGroup Group
nss_map_attribute cn sAMAccountName
pam_login_attribute sAMAccountName
pam_filter objectclass=user
pam_password ad  
nss_initgroups_ignoreusers avahi,avahi-autoipd,backup,bin,colord,daemon,debian-spamd,dhcpd,dnsmasq,elasticsearch,games,gdm,gnats,guest-ngHhtq,hplip,irc,kernoops,libuuid,lightdm,list,lp,mail,man,messagebus,mysql,nagios,news,postfix,postgres,proxy,pulse,root,rstudio-server,rtkit,saned,sgeadmin,speech-dispatcher,sshd,statd,sync,sys,syslog,tftp,usbmux,uucp,whoopsie,www-data
```

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
