# ceph-ansible-test

Just a wrapper for helping me test ceph-ansible. 

1. On hypervisor make VM:
```
ansible-playbook -i inventory.yml prep.yml
```

2. On VM run ceph-ansible
```
cd ceph-ansible
ansible-playbook -i inventory.yml site-container.yml.sample
```

## Podman Testing

The ([initial podman changes have merged](https://github.com/ceph/ceph-ansible/pull/3308))
so it is no longer necessary to do this hack:
```
yum install docker -y
pushd /usr/bin/
mv docker docker.old
ln -s podman docker
popd
```
Originally from the hack I was able to get to this:
```
[root@overcloud1 ~]# podman images
REPOSITORY              TAG      IMAGE ID       CREATED          SIZE
docker.io/ceph/daemon   latest   c99cb4f46f67   27 minutes ago   674MB
[root@overcloud1 ~]# podman ps
CONTAINER ID   IMAGE                          COMMAND          CREATED              STATUS                  PORTS   NAMES
ffa7fd684e36   docker.io/ceph/daemon:latest   /entrypoint.sh   About a minute ago   Up About a minute ago           ceph-mon-overcloud1
[root@overcloud1 ~]# 
```

### Current State (3 Dec 2018)

Inputs:
- fedora28
- docker not installed
- podman installed
- `container_binary: podman`

Results:
- ceph-ansible installs docker 
- get a working ceph-mon container but with docker
- playbook fails as it cannot start ceph-mgr contianer

