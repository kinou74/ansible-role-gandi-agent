gandi-hosting-vm
=========

The main goal of this role is to configure the [Gandi agent](https://docs.gandi.net/fr/cloud/gestion_serveur/agent_gandi.html) installed by `gandi-hosting-vm2` package on your [Gandi Hosting IaaS](https://www.gandi.net/en/cloud) virtual machine.

It also helps to fix an issue with ipv4 interfaces not getting its IP address from DHCP on vm freshly installed with Debian 10 _Buster_ image. This problem doesn't exist when _dist-upgrading_ from Debian 9 _Strech_ to Debian 10 _Buster_ . The root cause is the lack of the `/etc/systemd/system/systemd-udevd.service.d/local.conf` file with `RestrictAddressFamilies` parameter as below:

```properties
[Service]
RestrictAddressFamilies=AF_UNIX AF_NETLINK AF_INET AF_INET6 AF_PACKET
```

A small fix for _udev_ `86-gandi.rules` is included as well: support for `last_rule` option has been removed since version [v147](https://github.com/pavlinux/udev/blob/master/ChangeLog). As a consequence, when attaching an network interface, the `manage_iface` process may be called twice. The workaround is to add a `GOTO` instruction to a `LABEL`

Requirements
------------

Please ensure you've hard-rebooted your virtual machine at least once, from [Gandi-Cli](https://cli.gandi.net/) or your [Gandi admin console](https://admin.gandi.net/cloud/), before playing with this role: in fact, the `gandi-boostrap` may overwrite your configuration again and again if you only soft-reboot the vm (i.e. `shutdown -r`) and until you perform a hard-reboot.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

| Variable                  	| Default 	| Comments 	|
|---------------------	      |---------	|----------	|
| `gandi_config_motd`         | `1`       |   |
| `gandi_config_timezone`     | `1`       |   |
| `gandi_config_sysctl`       | `1`       |   |
| `gandi_config_sshd`         | `1`       |   |
| `gandi_config_console`      | `1`       |   |
| `gandi_config_hostname`     | `1`       |   |
| `gandi_config_nameserver`   | `1`       |   |
| `gandi_config_allow_mount`  | `1`       |   |
| `gandi_config_cron`         | `1`       |   |
| `gandi_config_mount_params` | `"rw,nosuid,nodev,noatime"` |   |
| `gandi_config_disk_root`    | `"/srv"`  |   |
| `gandi_config_nodhcp`       | `""`      |   |
| `gandi_config_multiqueue`   | `1`       |   |
| `gandi_config_network`      | `1`       |   |

Dependencies
------------

None


Listeners/Handlers
----------------

This role will restart `systemd` daemon and `udev` service if `RestrictAddressFamilies` fix is applied.

`udev` service is restarted if _udev_ `86-gandi.rules` is patched.

See role description above for more details.

Example Playbook
----------------

```
    - hosts: servers
      roles:
         - role: kinou74.gandi-hosting-vm
```

License
-------

To be defined

Contributor
------------------

- [Nicolas Garin](https://github.com/kinou74)