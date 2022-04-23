ipfs_config
===========

Deploy `go-ipfs` configuration with Ansible, including `systemd` service files.

Requirements
------------

`go-ipfs` must be present in the system.

Role Variables
--------------

The primary variable used by this role is `ipfs_config_default` that contains the whole `go-ipfs` config (as found int `.ipfs/config`) in YAML format:

```yaml

# 
ipfs_config_default:
  API:
  HTTPHeaders: {}
  Addresses:
    API:
    - /ip6/::/tcp/5001
    - /ip4/0.0.0.0/tcp/5001
    ...
```

`go-ipfs` config documentation can be found here https://github.com/ipfs/go-ipfs/blob/master/docs/config.md

In addition the role uses the following variables:

```yaml
ipfs_config_debug: true
ipfs_mem_hi: 800M
ipfs_mem_max: 1G
ipfs_home: /home/ipfs
```

Dependencies
------------

For installation of the actual `go-ipfs` binaries:

* [andrewrothstein.ipfs](https://galaxy.ansible.com/andrewrothstein/ipfs)

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

GPL3

Author Information
------------------

Pawel Krawczyk https://krvtz.net/