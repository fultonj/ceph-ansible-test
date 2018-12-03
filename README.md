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

#### Fedora28

Inputs:
- fedora28
- docker not installed
- podman installed
- `container_binary: podman`

Results:
- ceph-ansible installs docker 
- get a working ceph-mon container but with docker
- playbook fails as it cannot start ceph-mgr contianer

You get the same results however if you use docker in place of podman,
so I think I am hitting fedora28 issues.

#### Fedora29 Atomic

Inputs:
- fedora29atomic 
- docker and podman preinstalled
- `container_binary: podman`

Results
- get a working ceph-mon container with podman (not docker, yay!)
- playbook fails on `create ceph mgr keyring(s)` task


##### Outputs

podman
```
[root@f29atomic ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@f29atomic ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@f29atomic ~]# podman ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED         STATUS             PORTS   NAMES
a25cae51769c   docker.io/ceph/daemon:latest   /opt/ceph-container...   6 minutes ago   Up 6 minutes ago           ceph-mon-f29atomic
[root@f29atomic ~]# podman images
REPOSITORY              TAG      IMAGE ID       CREATED      SIZE
docker.io/ceph/daemon   latest   89af895e7d5b   3 days ago   674MB
[root@f29atomic ~]# 
```

Playbook output
```
TASK [ceph-mon : create ceph mgr keyring(s)] *************************************************************
Monday 03 December 2018  17:46:38 -0500 (0:00:01.314)       0:00:53.841 ******* 
failed: [192.168.122.241] (item=192.168.122.241) => {"changed": true, "cmd": ["podman", "run", "--rm", "--net=host", "-v", "/etc/ceph:/etc/ceph:z", "-v", "/var/lib/ceph/:/var/lib/ceph/:z", "-v", "/var/log/ceph/:/var/log/ceph/:z", "--entrypoint=ceph", "docker.io/ceph/daemon:latest", "-n", "client.admin", "-k", "/etc/ceph/ceph.client.admin.keyring", "--cluster", "ceph", "auth", "import", "-i", "/etc/ceph//ceph.mgr.f29atomic.keyring"], "delta": "0:00:06.234163", "end": "2018-12-03 22:46:44.954082", "item": "192.168.122.241", "msg": "non-zero return code", "rc": 1, "start": "2018-12-03 22:46:38.719919", "stderr": "2018-12-03 22:46:44.852 7f43dfe2c700 -1 auth: unable to find a keyring on /etc/ceph/ceph.client.admin.keyring: (2) No such file or directory\n2018-12-03 22:46:44.855 7f43dfe2c700 -1 monclient: authenticate NOTE: no keyring found; disabled cephx authentication\n[errno 95] error connecting to the cluster", "stderr_lines": ["2018-12-03 22:46:44.852 7f43dfe2c700 -1 auth: unable to find a keyring on /etc/ceph/ceph.client.admin.keyring: (2) No such file or directory", "2018-12-03 22:46:44.855 7f43dfe2c700 -1 monclient: authenticate NOTE: no keyring found; disabled cephx authentication", "[errno 95] error connecting to the cluster"], "stdout": "", "stdout_lines": []}
failed: [192.168.122.241] (item=192.168.122.241) => {"changed": true, "cmd": ["podman", "run", "--rm", "--net=host", "-v", "/etc/ceph:/etc/ceph:z", "-v", "/var/lib/ceph/:/var/lib/ceph/:z", "-v", "/var/log/ceph/:/var/log/ceph/:z", "--entrypoint=ceph", "docker.io/ceph/daemon:latest", "-n", "client.admin", "-k", "/etc/ceph/ceph.client.admin.keyring", "--cluster", "ceph", "auth", "import", "-i", "/etc/ceph//ceph.mgr.f29atomic.keyring"], "delta": "0:00:06.212102", "end": "2018-12-03 22:46:51.504912", "item": "192.168.122.241", "msg": "non-zero return code", "rc": 1, "start": "2018-12-03 22:46:45.292810", "stderr": "2018-12-03 22:46:51.409 7f29db70e700 -1 auth: unable to find a keyring on /etc/ceph/ceph.client.admin.keyring: (2) No such file or directory\n2018-12-03 22:46:51.413 7f29db70e700 -1 monclient: authenticate NOTE: no keyring found; disabled cephx authentication\n[errno 95] error connecting to the cluster", "stderr_lines": ["2018-12-03 22:46:51.409 7f29db70e700 -1 auth: unable to find a keyring on /etc/ceph/ceph.client.admin.keyring: (2) No such file or directory", "2018-12-03 22:46:51.413 7f29db70e700 -1 monclient: authenticate NOTE: no keyring found; disabled cephx authentication", "[errno 95] error connecting to the cluster"], "stdout": "", "stdout_lines": []}
```

There's that command again
```
[root@f29atomic ~]# podman run --rm --net=host -v /etc/ceph:/etc/ceph:z -v /var/lib/ceph/:/var/lib/ceph/:z -v /var/log/ceph/:/var/log/ceph/:z --entrypoint=ceph docker.io/ceph/daemon:latest -n client.admin -k /etc/ceph/ceph.client.admin.keyring --cluster ceph auth import -i /etc/ceph//ceph.mgr.f29atomic.keyring
2018-12-03 22:54:13.111 7feaeae4a700 -1 auth: unable to find a keyring on /etc/ceph/ceph.client.admin.keyring: (2) No such file or directory
2018-12-03 22:54:13.114 7feaeae4a700 -1 monclient: authenticate NOTE: no keyring found; disabled cephx authentication
[errno 95] error connecting to the cluster
[root@f29atomic ~]# ls -l /etc/ceph/
total 16
-rw-r--r--. 1 root root 789 Dec  3 22:46 ceph.conf
-r--------. 1 ceph ceph 140 Dec  3 22:46 ceph.mgr.f29atomic.keyring
-r--------. 1 root root  77 Dec  3 22:46 ceph.mon.keyring
-rw-r--r--. 1 root root  92 Nov 26 03:18 rbdmap
[root@f29atomic ~]# 
```

Good to see podman progress, I'm more worried about getting it working
on f28 than then f29atomic issues I'm seeing.
