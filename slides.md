## Ceph is a distributed storage system

+ designed for scalability, reliability and performance
+ can be run on commodity servers in a common network like Ethernet
+ scales up well to thousands of servers
+ automates management tasks such as data distribution and redistribution, data replication, failure detection and recovery
+ both self-healing and self-managing

--

### Reliable Autonomic Distributed Object Store

![RADOS](img/rados-structure.png)

--

### Controlled Replication Under Scalable Hashing

![CRASH](img/data-structure-example.png)

---

## [rook.io](https://rook.io)

![Design](img/kubernetes.png)

--

### Rook Design

![Design](img/rook-architecture.png)

---

## openSUSE Ceph

+ [openSUSE:Ceph](https://en.opensuse.org/openSUSE:Ceph) wiki page
+ OBS projects [filesystems:ceph](https://build.opensuse.org/project/show/filesystems:ceph)
+ [subprojects](https://build.opensuse.org/project/subprojects/filesystems:ceph) like [filesystems:ceph:nautilus](https://build.opensuse.org/project/show/filesystems:ceph:nautilus)
+ Process
  + development version of Ceph [filesystems:ceph:octopus](https://build.opensuse.org/project/show/filesystems:ceph:octopus)
  + submitted to [filesystems:ceph](https://build.opensuse.org/project/show/filesystems:ceph)
  + [filesystems:ceph](https://build.opensuse.org/project/show/filesystems:ceph) is submitted to Factory
  + stable version now is [filesystems:ceph:nautilus](https://build.opensuse.org/project/show/filesystems:ceph:nautilus)

--

### openSUSE Ceph containers

+ rook-ceph-image is Rook container for Ceph
+ ceph-image is a Ceph container that contains all need packages
  + ceph (osd, mon, mgr, mds)
  + ceph-mgr-dashboard
  + ceph-mgr-rook
  + ceph-radosgw
  + nfs-ganesha

--

### openSUSE Ceph containers

+ Process plan is to follow [openSUSE Building derived containers](https://en.opensuse.org/Building_derived_containers)
  + submit containers from [filesystems:ceph](https://build.opensuse.org/project/show/filesystems:ceph) to openSUSE:Factory
  + submit containers from [filesystems:ceph:nautilus](https://build.opensuse.org/project/show/filesystems:ceph:nautilus) to openSUSE:Leap:15.1:Images

---

## Development cluster on Vagrant

+ [vagrant-ceph](https://github.com/openSUSE/vagrant-ceph) openSUSE project
+ [openSUSE:Ceph](https://en.opensuse.org/openSUSE:Ceph) wiki page
+ [Rook in Vagrant cluster](https://en.opensuse.org/openSUSE:Ceph#Using_Rook_in_Vagrant_cluster)

--

### vagrant-ceph

+ follow instruction on wiki or github to use [vagrant-ceph](https://github.com/openSUSE/vagrant-ceph)
+ box could be found in [openSUSE:Factory/openSUSE-MicroOS](https://build.opensuse.org/package/binaries/openSUSE:Factory/openSUSE-MicroOS:Kubic-kubeadm-Vagrant/images)
+ `vagrant box add --provider libvirt --name opensuse/Kubic-kubeadm-cri-o Kubic.box`
+ `BOX="opensuse/Kubic-kubeadm-cri-o" vagrant up`
+ `vagrant ssh admin`
+ `sudo su`
+ `kubectl get nodes`

--

### vagrant-ceph k8s deployment

```yaml
      admin:
        - getent hosts admin-management | awk '{ print $1}' | xargs kubeadm init --cri-socket=/var/run/crio/crio.sock --pod-network-cidr=10.244.0.0/16 --token 56fa9a.705a6001db6a6756 --skip-token-print --apiserver-advertise-address

        - mkdir -p $HOME/.kube; cp -i /etc/kubernetes/admin.conf $HOME/.kube/config; chown $(id -u):$(id -g) $HOME/.kube/config
        - kubectl apply -f https://0y.at/kubicflannel
        - kubectl taint nodes admin node-role.kubernetes.io/master::NoSchedule-

        - curl -LkSs https://api.github.com/repos/SUSE/rook/tarball/suse-master -o /home/vagrant/rook.tar.gz
        - tar -xzf /home/vagrant/rook.tar.gz -C /home/vagrant

      all:
        - ( ( if [ "$HOSTNAME" != "admin" ]; then kubeadm join --token 56fa9a.705a6001db6a6756 --discovery-token-unsafe-skip-ca-verification admin-management:6443 >>/home/vagrant/join 2>&1; fi; ) & )
```

--

### vagrant-ceph Rook deployment

+ `cd SUSE-rook*/cluster/examples/kubernetes/ceph/`
+ `kubectl create -f common.yaml -f psp.yaml -f operator.yaml`
+ `kubectl create -f cluster.yaml -f toolbox.yaml`
+ `kubectl -n rook-ceph get pod`

--

### vagrant-ceph Rook deploys Ceph cluster

```
# kubectl -n rook-ceph exec $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') -- ceph -s
  cluster:
    id:     3d3eb9ab-7416-474d-8e28-fe673f23bd3e
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c (age 4m)
    mgr: a(active, since 3m)
    osd: 6 osds: 6 up (since 3s), 6 in (since 3s)
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   6.0 GiB used, 108 GiB / 114 GiB avail
    pgs:
```

--

### vagrant-ceph Rook defaults

+ by default it is _tiny_ cluster from 3 nodes
  + check _Vagrant_ file and _config.yml_ for
+  Vagrant changes default containers in yaml to
  + `image: registry.opensuse.org/filesystems/ceph/images/rook/ceph:latest` in _operator.yaml_
  + `image: registry.opensuse.org/filesystems/ceph/images/ceph:latest` in _cluster.yaml_
+ [SUSE Rook fork](http://github.com/suse/rook) is at _github.com/suse/rook_ and _suse-master_ branch

---

## CI cluster on Openstack

+ Terraform, Heat Orchestration Template, and etc
+ Heat is not supported on all Cloud instances
+ HOT approach has less tools in the middle so less failures and easier to analyze them
+ Templates examples could be found [here](https://github.com/denisok/ceph-kubic-stack)

---

## other tools

+ [terraform-kubic-kvm](https://github.com/kubic-project/kubic-terraform-kvm)
+ Heat is not supported on all Cloud instances
+ HOT approach has less tools in 

---

## Bugs

+ bug for image
+ bug for cloud-init

---

## Contribute

- https://github.com/rook/rook/ Rook
- https://github.com/ceph/ceph Ceph
- https://build.opensuse.org/project/show/filesystems:ceph:octopus openSUSE Ceph, Rook and other tools



