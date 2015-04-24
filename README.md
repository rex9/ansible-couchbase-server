# Couchbase Server

               .CCCCC                       CCCCC.
              .CCCCCC                       CCCCCC.
              .CCCCCC                       CCCCCC.
              .CCCCCC                       CCCCCC.
              .CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC.
              .CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC.
              .CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC.
               .CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC.

[Couchbase Server](http://www.couchbase.com/couchbase-server/overview) is a
high performance NoSQL document database available in Community and Enterprise
editions for several supported operating systems.

This Ansible role can be used to install Couchbase Server on cluster nodes,
initialize working clusters, create buckets, and load buckets with test
documents with the playbooks included in the `examples` directory.

## Requirements

This role is tested for basic functionality with the following software:

* Couchbase Server (versions 2.5.2-3.0.3)
* Ansible (version 1.9.0.1)
* CentOS (versions 6.2-6.7)
* Ubuntu (versions 12.04-13.10)

## Designed for Ansible Galaxy

This role is designed to be installed via the `ansible-galaxy` command
instead of being directly run from the git repository.

You should install it like this:

```
ansible-galaxy install couchbase.couchbase-server
```

You'll want to make sure you have write access to `/etc/ansible/roles/` since
that is where the role will be installed by default, or define your own
Ansible role path by creating a `$HOME/.ansible.cfg` file with these contents:

```
[defaults]
roles_path = <path_to_your_preferred_role_location>
```

Change `<path_to_your_preferred_role_location>` to a directory you have write
access to.

## Role Variables

In cases where you want simple clusters for development or other
non-production use, the default values for the Couchbase Server role's common
variables can usually be left as-is.

However, should you need specific performance or otherwise wish to tweak them
for your particular purpose, this section describes all of the user editable
variables in detail including their default values for your reference.

### Default Variables

| Name                                 | Default                                                                                   | Description                             |
| ------------------------------------ | ----------------------------------------------------------------------------------------- | --------------------------------------- |
| couchbase_server_edition | enterprise | Couchbase Server edition to install: community or enterprise |
| couchbase_server_admin | Administrator | Couchbase Server administrator user name |
| couchbase_server_password | couchbase | Couchbase Server administrator user password |
| couchbase_server_ram | 3072 | The per server RAM quota specified in megabytes |
| couchbase_server_admin_port          | 8091                                                                                      | Administration and web console port     |
| couchbase_server_api_port            | 8092                                                                                      | Couchbase Server API port               |
| couchbase_server_internal_ports      | 11209:11211                                                                               | Memcached and client ports              |
| couchbase_server_node_data_ports     | 21100:21299                                                                               | Distributed Erlang communication ports  |
| couchbase_server_config_file         | /opt/couchbase/var/lib/couchbase/config/config.dat                                        | Full path to config.dat                 |
| couchbase_server_filesystem | ext4 | Default filesystem for data and index volumes |
| couchbase_server_mountpoint | / | Logical volume mountpoint |
| couchbase_server_partition | /dev/mapper/VolGroup-lv_root | Logical volume partition |
| couchbase_server_mount_options | 'noatime,barrier=0,errors=remount-ro' | Additional mount options |
| couchbase_server_data_path           | /opt/couchbase/var/lib/couchbase/data                                                     | Path to data files                      |
| couchbase_server_home_path           | /opt/couchbase                                                                            | Couchbase Server installation base path |
| couchbase_server_index_path          | /opt/couchbase/var/lib/couchbase/data                                                     | Path to index files                     |
| couchbase_server_log_path            | /opt/couchbase/var/lib/couchbase/logs                                                     | Path to log files                       |
| couchbase_server_cbcollect_path | /tmp |  Path to cbcollect_info output |
| couchbase_server_tmpdir | /tmp | System wide TMPDIR for cbcollect_info |

### Special Variables

*The following variables should be set with caution* as they have potential
negative performance implications; they should not be used without knowledge
of the changes which are made to the operating system configuration:

| Name                                 | Default  | Description                                    |
| ------------------------------------ | -------- | ---------------------------------------------- |
| couchbase_server_tune_os          | false    | Whether to tune OS with optimized settings |

## Examples

The `examples` directory contains some basic playbooks, host inventory
examples, and Vagrant bits (primarily for Mac OS X development use)
as follows:

* `cluster_install.yml` prepares OS and installs Couchbase Server only
* `cluster_init.yml` installs Couchbase Server and initializes the cluster
* `create_bucket.yml` creates an example bucket
* `load_bucket.yml` loads sample JSON data into a bucket
* `example_hosts` example hosts inventory in format required by this project
* `Vagrantfile` example Vagrant development cluster definition
* `centos` CentOS hosts inventory for Vagrant based development cluster
* `ubuntu` Ubuntu hosts inventory for Vagrant based development cluster

### Create Buckets

The example playbook `create_bucket.yml` for bucket creation can be used
as follows:

Upon first execution without specifying variable arguments via the `ansible-playbook` extra vars ('-e') option, the playbook will generate a
bucket with the following properties:

* Bucket name: *default*
* Bucket type: *couchbase*
* Bucket port: *11211*
* Bucket RAM size: 256MB
* Bucket replica number: 1

If you'd like to create your own buckets, then use the `ansible-playbook`
extra vars ('-e') option and specify values for the
*b_name*, *b_type*, *b_port*, *b_ramsize*, and *b_replica* variables like so:

```
ansible-playbook -i centos create_bucket.yml \
-e "b_name=danika b_type=couchbase b_port=11223 b_ramsize=256 b_replica=2"
```

or perhaps you'd like to make a memcached based bucket? No problem:

```
ansible-playbook -i centos create_bucket.yml \
-e "b_name=breandon b_type=memcached b_port=11224 b_ramsize=512 b_replica=0"
```

Additional playbooks are planned for future versions of this project as well.


### Quick Start for Simple Mac OS X Cluster

Follow these steps to have a simple 3 node development or evaluation
cluster on a >= 8GB Mac with OS X, VirtualBox and Vagrant:

1. export ROLEPATH=<ansible_role_path>
2. `ansible-galaxy install couchbase.couchbase-server`
3. cd $ROLEPATH/couchbase.couchbase-server/examples
4. vagrant up

This will install three (3) CentOS 6.5 nodes with 1.5GB RAM each and cluster
them together. The nodes will be available at 10.1.42.10, 10.1.42.20, and
10.1.42.30 as defined in the `Vagrantfile`.

To install Debian based nodes, change the command in step 4 to:

```
BOX_NAME=chef/debian-7.4 CLUSTER_HOSTS=debian vagrant up
```

To install Ubuntu based nodes, change the command in step 4 to:

```
BOX_NAME=ubuntu/trusty64 CLUSTER_HOSTS=ubuntu vagrant up
```


## Dependencies

None

## License

Apache

## Author Information

- Brian Shumate (brian at couchbase.com)
