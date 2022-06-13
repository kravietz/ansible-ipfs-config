ipfs_config
===========

Deploy `go-ipfs` configuration with Ansible, including `systemd` service files.

Requirements
------------

`go-ipfs` must be present in the system.

# Role Variables

## go-ipfs
If no `ipfs_config` is found in Ansible variables for the host, the role will run `ipfs init`.

When `ipfs_config` is defined, it must as a minimum contain `Identity.PeerID` and `Identity.PrivKey`. It's then merged with
`ipfs_config_default` which contains default configuration produced by `ipfs init`, per [go-ipfs` config reference](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md). Sample:

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

The role also configured `systemd` service for `ipfs` using the following variables (enforced using [systemd.resource-control](https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html)):

```yaml
ipfs_config_debug: true
ipfs_mem_hi: 800M
ipfs_mem_max: 1G
ipfs_home: /home/ipfs
```

## ipfs-cluster-service

IPFS clusters _require_ `ipfs_cluster_secret` variable explicitly defined. If the secret is not defined, the cluster configuration will be skipped entirely.

The secret needs to be shared among all hosts that belong to a cluster (e.g. as a group variable), and can be generated with `openssl rand -hex 32`:

```yaml
# protect with ansible-vault
ipfs_cluster_secret: 0d26e71c48e84890c89ef73bc26a45b167df03177df3ca399244e630513fbf53
```
On the _first_ cluster node, you should also define the following:
```yaml
ipfs_cluster_bootstrap: []`
```
All remaining cluster configuration on the _first_ cluster node is optional, and the role will run `ipfs-cluster-service init` on the node, creating new configuration with the configured secret.

All _subsequent_ hosts in the cluster must also declare `ipfs_cluster_bootstrap` with address and public key of at least one existing cluster member, allowing them to bootstrap configuration and establish connections to other members, for example:

```yaml
ipfs_cluster_bootstrap:
  - /ip4/192.168.144.200/tcp/9096/p2p/12D3KooWPd39DaEUVdaEHaJhKb3nDBA2SPjgwVA3YsrsSXH7XGa3
```

This leads to an interesting chicken-and-egg problem which can be resolved by running Ansible twice:

* run the playbook with `ipfs_cluster_bootstrap: []` on all servers - this will install daemons and initialise config even though nodes will be initally disconnected;
* run `sudo -u ipfs ipfs-cluster-ctl id` on any of the initialised servers to retrieve its IPFS cluster address;
* configure that address in `ipfs_cluster_bootstrap` for all the other hosts and re-run Ansible.

**Optional:** IPFS cluster identifier and private key are also configurable and can be controlled with the following variables - they end up in `~/.ipfs-cluster/identity.json`:

```yaml
ipfs_cluster_identity:
  id: 12D3KooWPd39DaEUVdaEHaJhKb3nDBA2SPjgwVA3YsrsSXH7XGa3
  # protect with ansible-vault
  private_key: CAESQLDHjjm8oMlXz5CAI1l40ytMyoJfEBANEfP3AO3RhzI0zRy3BfXYwZaiRtCx9odFzW7dRrdj3oD/kJLIhTiHE6g=
```
Any other configuration options can be set in `ipfs_cluster_config` and they will be merged with `ipfs_cluster_config_default` to form a full IPFS cluster service configuration file written to `~/.ipfs-cluster/service.json`.

All `systemd` settings set for `ipfs` (see above) will be also applied to the `ipfs-cluster` service. 

Dependencies
------------

For installation of the actual `go-ipfs` binaries:

* [andrewrothstein.ipfs](https://galaxy.ansible.com/andrewrothstein/ipfs)

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
        - role: kravietz.ipfs_config

License
-------

GPL3

Author Information
------------------

Pawel Krawczyk https://krvtz.net/