## NOTE
We are currently working on an open source version of ceph-docker-plugin.
We will upload the source soon.

# ceph-docker-plugin

This is a repository about docker volume plugin to support ceph storage.
Ceph Docker Volume Plugin to enable management of Ceph Storage and OpenStack Cinder Block Service.


# Install Plugin

```bash
$ curl -LO https://.../ceph-docker-plugin/install.sh && sudo bash ./install.sh [OPTIONS] -t TYPE [-v VERSION]
sh install.sh [OPTIONS] -t TYPE [-v VERSION]
    -h         Print usages
    -u         enable upgrade
    -t TYPE    type
    -v VERSION Plugin Version (tag) (ND : latest)
```

#### Sample
```console
$ curl -LO https://.../ceph-docker-plugin/install.sh && sudo bash ./install.sh [OPTIONS] -t test -v 1.0.1
```


# Upgrade Plugin
```bash
$ curl -LO https://.../ceph-docker-plugin/install.sh && sudo bash ./install.sh [OPTIONS] -t TYPE [-v VERSION] -u
sh install.sh [OPTIONS] -t TYPE [-v VERSION]
    -h         Print usages
    -u         enable upgrade
    -t TYPE    type
    -v VERSION Plugin Version (tag) (ND : latest)
```

#### Sample
```console
$ curl -LO https://.../ceph-docker-plugin/install.sh && sudo bash ./install.sh [OPTIONS] -t test -v 1.0.2 -u
```

# Enable and Disable Plugin

#### Enable Plugin
```bash
$ docker plugin enable ceph
$ docker plugin ls
```

#### Disable Plugin
```bash
$ docker plugin disable -f ceph
$ docker plugin ls
```

# Try to test

Ceph Volume Plugin will support all of clis about `docker volume` command.

```bash
$ docker volume --help
Usage:	docker volume COMMAND

Manage Docker volumes

Options:
      --help   Print usage

Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  rm          Remove one or more volumes

Run 'docker volume COMMAND --help' for more information on a command.
```


### Create Service with Volume on swarm cluster
```bash
$ docker service create --name ceph-test \
--mount type=volume,src=ceph-test.{{.Task.Slot}},\
volume-opt=project-id={project-id},\
dst=/data,volume-driver=ceph \
--replicas 3 nginx
```

### Create Volume

```bash
$ docker volume create --driver=ceph --name {volume-name} -o size={size:gb} -o project-id={pasta-project-id}
```

### List Volume

```bash
$ docker volume ls
DRIVER              VOLUME NAME
ceph:latest         vol-test.1
ceph:latest         vol-test.2
ceph:latest         vol-test.3
ceph:latest         vol-test.4
```

### Inspect Volume

```bash
$ docker volume inspect vol-test.1
[
    {
        "Driver": "ceph:latest",
        "Labels": null,
        "Mountpoint": "/var/lib/ceph/mount/vol-test.1",
        "Name": "vol-test.1",
        "Options": {},
        "Scope": "global"
    }
]
```

### Delete Volume

```bash
$ docker volume rm {volume-name}
```

### Run Container with Volume

```bash
$ docker volume create --driver=ceph --name ${volumeName} -o size=${size} -o project-id=${projectId}
$ docker run --volume-driver=ceph --volume ${volumeName}:${mountDirInContainer} -it centos /bin/bash
[root@82df79f8777c /]#
[root@82df79f8777c /]# df -h
Filesystem                                                                                        Size  Used Avail Use% Mounted on
/dev/mapper/docker-253:0-526212-45fdd7ddaa7fe52c20be3fc9e61f24151aade654282ccb4f4c513e2b3a10abf9   10G  241M  9.8G   3% /
tmpfs                                                                                             1.9G     0  1.9G   0% /dev
tmpfs                                                                                             1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/rbd1                                                                                         2.0G   33M  2.0G   2% /mnt/foo
/dev/mapper/VolGroup00-LogVol00                                                                    37G  3.7G   32G  11% /etc/hosts
shm                                                                                                64M     0   64M   0% /dev/shm
```

### OpenStack Cinder CLIs

- cinder list
```bash
$ cinder list
+--------------------------------------+-----------+------+------+-------------+----------+-------------+
| ID                                   | Status    | Name | Size | Volume Type | Bootable | Attached to |
+--------------------------------------+-----------+------+------+-------------+----------+-------------+
| e8d3de7e-5f0d-4db2-aad7-74ba32130d04 | available | foo  | 2    | rbd-1       | false    |             |
+--------------------------------------+-----------+------+------+-------------+----------+-------------+
```

- cinder show
```bash
$ cinder show foo
+--------------------------------+--------------------------------------+
| Property                       | Value                                |
+--------------------------------+--------------------------------------+
| attachments                    | []                                   |
| availability_zone              | nova                                 |
| bootable                       | false                                |
| consistencygroup_id            | None                                 |
| created_at                     | 2016-10-19T08:10:18.000000           |
| description                    | volume test                          |
| encrypted                      | False                                |
| id                             | e8d3de7e-5f0d-4db2-aad7-74ba32130d04 |
| metadata                       | {}                                   |
| migration_status               | None                                 |
| multiattach                    | False                                |
| name                           | foo                                  |
| os-vol-host-attr:host          | pm-09@rbd-1#RBD                      |
| os-vol-mig-status-attr:migstat | None                                 |
| os-vol-mig-status-attr:name_id | None                                 |
| os-vol-tenant-attr:tenant_id   | 3c32ccf2836f412aaedd09225debb3db     |
| replication_status             | disabled                             |
| size                           | 2                                    |
| snapshot_id                    | None                                 |
| source_volid                   | None                                 |
| status                         | available                            |
| updated_at                     | 2016-10-19T08:10:19.000000           |
| user_id                        | 8b423ff8566a444fb4ee800607bad017     |
| volume_type                    | rbd-1                                |
+--------------------------------+--------------------------------------+
```
# How to test our own code in physical node
- copy overwrite ceph-docker-plugin and restart ceph-docker-plugin service
```bash
$ curl -sSl https://.../ceph-docker-plugin/install.sh | sh
$ cd ${GOPATH}/src/ceph-docker-plugin
$ go build .
$ sudo cp -f ${GOPATH}/src/ceph-docker-plugin/ceph-docker-plugin /usr/bin/ceph-docker-plugin
$ sudo systemctl restart ceph-docker-plugin
```
- test the function, for example:
```bash
$ docker volume create --driver=ceph --name ${volumeName} -o size=${size}  -o project-id=${projectId}
$ docker volume inspect ${volumeName}
[
    {
        "Name": "${volumeName}",
        "Driver": "ceph",
        "Mountpoint": "/var/lib/ceph/mount/06a937f9-36dc-4d52-b335-4963a5e5d887",
        "Labels": {
            "db": ""
        }
    }
]
$ docker run --volume-driver=ceph --volume ${volumeName}:${mountDirInContainer} -it centos /bin/bash
$ docker volume rm ${volumeName}
```

# Containerized ceph-docker-plugin
- Parepare keystone, cinder, ceph env, make sure they work well.
- Create volume type, create volume:save_project_id_volume to save volume's project-id, add member role to user and project
```bash
$ cinder type-create ceph
$ openstack user create --domain default  --password ${password} --enable ${user}
$ openstack role add --domain default --user ${user} ${role}
$ openstack role add  --project-domain  default --project ${project} --user ${user} ${role} # May be execute multiple times
```
- Build docker image and run ceph-docker-plugin using plugin mode
```bash
$ sh build.sh --envType  ${envTypeValue}  # envTypeValue:[ncloud,test,prod-dev,prod-prod]
$ sh install.sh --envType ${envTypeValue} # for developer
$ curl -sSl https://.../ceph-docker-plugin/install.sh | sudo -E envType=${env} bash # for terminal user
```
- Use the volume from ceph in docker container, for example:
```bash
$ docker volume create --driver=ceph --name ${volumeName} -o size=1 -o project-id=${projectId}
$ docker run --volume-driver=ceph --volume ${volumeName}:${mountDirInContainer} -it centos /bin/bash
```
