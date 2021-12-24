# samba - (servercontainers/samba) [x86 + arm]

samba on alpine

with timemachine, zeroconf (`avahi`) and WSD (Web Services for Devices) (`wsdd2`) support

## Changelogs

* 2021-12-24
    * experienced some issues - now use only `interactive` flag to start `smbd`
    * start `wsdd2` after 10 seconds
* 2021-12-02
    * made `wsdd2` service optional
    * updated version
* 2021-09-27
    * added support for `wsdd2` parameterization
* 2021-08-30
    * added support for groups
* 2021-08-27
    * removed old multi arch build dockerfiles - `builx is used`
    * added `wsdd2` for service discovery on windows
* 2021-08-23
    * fixed `SAMBA_GLOBAL_CONFIG_...` missing key.
* 2021-08-08
    * added env to contorl `log level` - default value `1` 
    * fixed `SAMBA_GLOBAL_CONFIG_...` with spaces in the key.
* 2021-03-16
    * added support for specifing the `uid` for each `ACCOUNT_` using `UID_username=1234214` env.
* 2021-03-09
    * updated healthcheck to work with external avahi server
* 2020-12-22
    * added support for samba password hashes instead of just plaintext passwords
* 2020-12-10
    * added Timemachine Multiuser Support (samba config path needs to end with `%U`)
* 2020-12-09
    * bug fix: `</service-group>` gets removed with multiple timemachine volumes
* 2020-11-08
    * fixed samba user creation
    * custom avahi service name
* 2020-11-05
    * multiarch build
    * rewrite from debian to alpine
    * enhanced timemachine support

## Info

This is a Samba Server Container running on `_/alpine`.

## Environment variables and defaults

### Samba

*  __SAMBA\_GLOBAL\_CONFIG\_someuniquevalue__
    * add any global samba config to `smb.conf`
    * example value: `key = value`
    * important if the SAMBA key contains a ` ` space replace it with `_SPACE_`
        * e.g. `foo_SPACE_bar`

* __ACCOUNT\_username__
    * multiple variables/accounts possible
    * adds a new user account with the given username and the env value as password or samba hash
        * either you add a simple plaintext password as value (can't start with `:`username`:[0-9]*:` or it will be detected as hash)
        * to add a samba hash e.g. `user:1002:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX:8846F7EAEE8FB117AD06BDD830B7586C:[U          ]:LCT-5FE1F7DF:` (user: `user` / password: `password`) add the line from `/var/lib/samba/private/smbpasswd`
        * create hash using this command `docker run -ti --rm --entrypoint create-hash.sh servercontainers/samba`
        * see `docker-compose.yml` user `foo` for an example how it's used/configured.
    * to restrict access of volumes you can add the following to your samba volume config:
        * `valid users = alice; invalid users = bob;`

* __UID\_username__
    * optional
    * specify the `uid` explicitly for each user account.
    * the `username` part must match to a specified `ACCOUNT_username` environment variable

* __GROUP\_groupname__
    * optional
    * value will be `gid`
    * example: `GROUP_devops=1500` will create group `devops` with id `1500`

* __GROUPS\_username__
    * optional
    * additional groups for the user
    * to create groups look at `GROUP_groupname` or mount/inject /etc/groups file (can cause problems)
    * the `username` part must match to a specified `ACCOUNT_username` environment variable
    * one or more groups to add seperated by a `,`
    * example: `GROUPS_johndoe=musican,devops`

* __MODEL__
    * _optional_ model value of avahi samba service
    * _default:_ `TimeCapsule`
    * some available options are Xserve, PowerBook, PowerMac, Macmini, iMac, MacBook, MacBookPro, MacBookAir, MacPro, MacPro6,1, TimeCapsule, AppleTV1,1 and AirPort.

* __AVAHI\_NAME__
    * _optional_ name of avahi samba service
    * _default:_ _hostname_

* __SAMBA\_CONF\_LOG\_LEVEL__
    * default: _1_

* __SAMBA\_CONF\_WORKGROUP__
    * default: _WORKGROUP_

* __SAMBA\_CONF\_SERVER\_STRING__
    * default: _Samba Server_

* __SAMBA\_CONF\_MAP_TO_GUEST__
    * default: _Bad User_

* __SAMBA\_VOLUME\_CONFIG\_myconfigname__
    * adds a new samba volume configuration
    * multiple variables/confgurations possible by adding unique configname to SAMBA_VOLUME_CONFIG_
    * take a look at https://wiki.samba.org/index.php/Configure_Samba_to_Work_Better_with_Mac_OS_X -> EXPLANATION OF VOLUME PARAMETERS
    * for timemachine only add `fruit:time machine = yes` and all other needed settings are automatically added
        * you can also use `fruit:time machine max size = 500G;` to limit max size of time machine volume
        * if your path variable ends with `%U` e.g. `path = /shares/timemachine/%U;` multi user mode gets activated and each user gets their own subdirectory for their own share.

* __WSDD2\_DISABLE__
    * _optional_
    * default not set - set to any value to disable wsdd2 Service
* __WSDD2\_PARAMETERS__
    * _optional_ specify parameters for wsdd2
    * default not set - wsdd2 starts without any parameters
    * e.g. `-l`

### Volumes

* __your shares__
    * by default I recommend mounting all shares beneath `/shares` and configure them using the `path` property

* __/external/avahi__
    * mount your avahi service folder e.g. `/etc/avahi/services/` to this spot
    * the container now maintains the service file `samba.service` for you - __it will be overwritten!__
    * when mounted, the internal avahi daemon will be disabled


## Some helpful indepth informations about TimeMachine and Avahi / Zeroconf 

### General Infos

- Samba
    - https://github.com/willtho89/docker-samba-timemachine/
    - https://github.com/sp00ls/SambaConfigs very interessting multi user timemachine setup
    - https://wiki.samba.org/index.php/Configure_Samba_to_Work_Better_with_Mac_OS_X
    - https://serverfault.com/questions/1010822/samba4-issues-with-time-machine-cannot-create-new-backup-on-samba-share

- Avahi
    - https://openwrt.org/docs/guide-user/services/nas/samba_configuration#zeroconf_advertising
    - http://samba.sourceforge.net/wiki/index.php/Bonjour_record_adisk_adVF_values
    - https://linux.die.net/man/5/avahi.service


You can't proxy the zeroconf inside the container to the outside, since this would need routing and forwarding to your internal docker0 interface from outside.
So you need to use the `network=host` mode to enable zeroconf from within the container

You can just expose the needed Port 548 to the docker hosts port and install avahi.
After that just add a new service which fits to your config.


## Windows 10 Network Discovery

For the Windows 10 Network Discovery the `hostname` of the container is used.
If you use `network_mode: host` then it's the docker-host `hostname`.

If you use any other `network_mode` and want to avoid the autogenerated cryptic hostname of the container, you can specify
a explicit hostname using: `hostname: my-samba-containers-hostname`

- WSD
    - https://devanswers.co/discover-ubuntu-machines-samba-shares-windows-10-network/
    - https://github.com/Netgear/wsdd2

Note: _This `wsdd2` service seems to need `CAP_NET_ADMIN` as a capability. (more info: https://github.com/ServerContainers/samba/issues/50)_
```
    cap_add:
      - CAP_NET_ADMIN
```
