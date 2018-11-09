# ceph-ansible-test

Just a wrapper for helping me test ceph-ansible


1. On hypervisor make VM and checkout [podman branch](https://github.com/ceph/ceph-ansible/compare/podman-support)
```
ansible-playbook -i inventory.yml prep.yml
```

2. On VM symlink docker to podman (this should be unecessary in time)
```
yum install docker -y
pushd /usr/bin/
mv docker docker.old
ln -s podman docker
popd
```

3. On VM run ceph-ansible
```
cd ceph-ansible
ansible-playbook -i inventory.yml site-docker.yml.sample
```

It will fail but the monitor does get running with podman

```
[root@overcloud1 ~]# podman images
REPOSITORY              TAG      IMAGE ID       CREATED          SIZE
docker.io/ceph/daemon   latest   c99cb4f46f67   27 minutes ago   674MB
[root@overcloud1 ~]# podman ps
CONTAINER ID   IMAGE                          COMMAND          CREATED              STATUS                  PORTS   NAMES
ffa7fd684e36   docker.io/ceph/daemon:latest   /entrypoint.sh   About a minute ago   Up About a minute ago           ceph-mon-overcloud1
[root@overcloud1 ~]# 
```
