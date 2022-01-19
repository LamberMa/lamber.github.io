## 镜像制作




## 安装

```shell
virt-install \
--name=<domain_name> \
--os-variant=rhel7 \
--ram=8192 \
--vcpus=4 \
--disk path=<path_to_image>,format=qcow2,size=12,bus=virtio  \
--accelerate --import --noautoconsole --vnc
```
开启CPU直通以及网络直通（SR-IOV）
```shell
virt-install \
--name=init_centos \
--os-variant=rhel7 \
--ram=32768 \
--cpu host-passthrough \
--network network=passthrough_eno1,model=rtl8139 \
--vcpus=16 \
--disk path=<path_to_image>,format=qcow2,size=12,bus=virtio  \
--accelerate --import --noautoconsole --vnc
```