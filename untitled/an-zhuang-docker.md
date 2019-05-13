---
description: 很多安装包括官方脚本感觉不全面，缺少一些安装前设置和例如场景的补全脚本设置，这里讲下安装步骤
---

# 安装docker

从前面的namespace各项技术来看是很早出现在Linux的，但是实际上近几年才有大规模的基于它的应用开发导致这些内核技术的发展，所以安装docker建议使用各个较新的发行版Linux和使用较新的docker版本。建议使用ubuntu，centos的话进来不要使用低版本特别7.2坑非常多，尽量最新的7.6。这里我是写的centos的安装，同时也推荐使用年份命名版本的docker

环境设置\(对NetworkManager熟悉的话可以不关闭它\):

```
systemctl disable --now firewalld NetworkManager
setenforce 0
sed -ri '/^[^#]*SELINUX=/s#=.+$#=disabled#' /etc/selinux/config
```

如果是图形界面Linux安装了dnsmasq的话可能dns server会被设置为127.0.0.1，这会导致容器无法解析域名，建议关闭它

```text
systemctl disable --now dnsmasq
```

默认下系统的User namespaces是没开的，需要我们手动开启

```text
grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"
```

这个开启设置了内核的，所以需要reboot生效

```text
reboot
```

设置下系统内核参数

```text
cat<<EOF > /etc/sysctl.d/docker.conf
# 要求iptables不对bridge的数据进行处理
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
EOF
sysctl --system
```

使用官方检查脚本检查下环境是否需要要求:

```text
curl -s https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh | bash
```

使用官方脚本安装

```text
export VERSION=18.09
curl -fsSL "https://get.docker.com/" | bash -s -- --mirror Aliyun
```

配置docker，默认docker0的段是`172.17.0.1/16`，如果和vpc或者局域网冲突可以修改下

```text
mkdir -p /etc/docker/
cat>/etc/docker/daemon.json<<EOF
{
  "bip": "172.17.0.1/16",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://fz5yth0r.mirror.aliyuncs.com"],
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
EOF
```

存储驱动建议使用现在的overlay2，以前的devicemapper坑非常多不建议使用

设置docker补全脚本并启动docker

```text
yum install -y epel-release bash-completion 
cp /usr/share/bash-completion/completions/docker /etc/bash_completion.d/
systemctl enable --now docker
```

如果docker启动失败删掉配置文件下面的内容，然后删掉/var/lib/docker后再重启，这个原因在于根分区是lvm的时候很大几率出现，原因没去关注过

```text
  "storage-driver": "overlay2",
```

