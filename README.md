gandi-agent
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

For the time being, this role is only compatible with Debian installations.

Role Variables
--------------

| _Defaults_ variable         	      | Default value	| Comments 	|
|---------------------	              |---------	|----------	|
| `gandi_fix_buster_restrictaddressfamilies` | `False`   | Should we fix `RestrictAddressFamilies` in systemd-udev or not. Additional condition when run on Debian 10 is applied.  |
| `gandi_patch_udev_last_rule`        | `False`   | Patch udev `86-gandi.rules` to use GOTO instead of `last_rule` option |
| `gandi_config_motd`                 | `True`    |   |
| `gandi_config_timezone`             | `True`    |   |
| `gandi_config_sysctl`               | `True`    |   |
| `gandi_config_sshd`                 | `True`    |   |
| `gandi_config_console`              | `True`    |   |
| `gandi_config_hostname`             | `True`    |   |
| `gandi_config_nameserver`           | `True`    |   |
| `gandi_config_allow_mount`          | `True`    |   |
| `gandi_config_cron`                 | `True`    |   |
| `gandi_config_mount_params`         | `"rw,nosuid,nodev,noatime"` |   |
| `gandi_config_disk_root`            | `"/srv"`  |   |
| `gandi_config_nodhcp`               | `""`      |   |
| `gandi_config_multiqueue`           | `True`    |   |
| `gandi_config_network`              | `True`    |   |

The following vars are defined to not be hardcoded in the gandi agent configuration file template. Nevertheless they shouldn't be changed.

| _Vars_ variable       | Value   	               |
|---------------------	|---------	               |
| `gandi_plugin_dir`    | `"/etc/gandi/plugins.d"` |
| `gandi_temp_dir_conf` | `"/var/gandi"`           |
| `gandi_syslog_target` | `"gandi"`                |
| `gandi_serverurl`     | `"mirrors.gandi.net"`    |
| `gandi_hook_dir`      | `"/etc/gandi/hooks/"`    |


Dependencies
------------

None


Listeners/Handlers
----------------

- Most of gandi agent parameters are read at boot time when gandi `systemd` services are started. As a consequence, and for the time being, there is no particular handler when gandi agent parameters are changed. This means that you may have to reboot your VM for new parameters to be taken into account.

- `systemd` daemon and `udev` service are restarted when `RestrictAddressFamilies` fix is applied.

- `udev` service is restarted when _udev_ `86-gandi.rules` is applied.

Example Playbook
----------------

```yaml
    - hosts: servers
      roles:
        - role: kinou74.gandi-agent
          when: ansible_facts['distribution'] == "Debian"
```

License
-------

Licensed under the 2-clause "Simplified BSD License". See [LICENSE.md](/LICENSE.md) for details.

Contributor
------------------

- [Nicolas Garin](https://github.com/kinou74)