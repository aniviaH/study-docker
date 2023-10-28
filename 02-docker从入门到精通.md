<a name="3be400ee"></a>
# docker安装部署
docker最核心的组件

- image镜像，构建容器（我们将应用程序运行所需的环境，打包为镜像文件）
- Container，容器（你的应用程序，就跑在容器中）
- 镜像仓库（dockerhub）（保存镜像文件，提供上传，下载镜像），作用好比github
- Dockerfile，将你部署项目的操作，写成一个部署脚本，这就是dockerfile，且该脚本还能构建出镜像文件
> 创建容器的过程

- 获取镜像，如 docker pull centos，从镜像仓库拉取
- 使用镜像创建容器
- 分配文件系统，挂载一个读写层，在读写层加载镜像
- 分配网络/网桥接口，创建一个网络接口，让容器和宿主机通信
- 容器获取IP地址
- 执行容器命令，如/bin/bash
- 反馈容器启动结果
<a name="7d20857b"></a>
# 安装docker--CentOS
提前准备好一个宿主机（vmware去创建一个linux机器，然后安装docker去使用）
<a name="bc865061"></a>
## 基础环境配置
<a name="927d0791"></a>
### 配置yum源
```perl
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all #清空一下本地缓存
yum makecache #生成新的缓存

iptables -F #清空现有规则

yum install -y bash-completion vim lrzsz wget expect net-tools nc nmap tree dos2unix htop iftop iotop unzip telnet sl psmisc nethogs glances bc ntpdate openldap-devel #安装常用工具

systemctl disable  firewalld #禁用防火墙
systemctl stop firewalld #关闭防火墙
```
<a name="0fae6bc1"></a>
### 安装docker
<a name="86ae1778"></a>
#### 开启linux内核的流量转发
```perl
# 通过cat重定向将内容写入到/etc/sysctl.d/docker.conf文件(该文件一开始并不存在)
cat <<EOF>> /etc/sysctl.d/docker.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
net.ipv4.ip_forward=1
EOF

# sysctl.d 是用于控制内核参数的一个目录，上面文件生成之后，要让它生效，需执行下下面这条命令
sysctl -p /etc/sysctl.d/docker.conf # 加载修改内核的参数，配置文件
# 上面命令有两个报文件不存在, 需执行下下面这条命令
# sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-ip6tables: No such file or directory
# sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-iptables: No such file or directory
# net.ipv4.conf.default.rp_filter = 0
# net.ipv4.conf.all.rp_filter = 0
# net.ipv4.ip_forward = 1

modprobe br_netfilter # 执行完这个，再重新执行上面的sysctl命令
```
利用yum快速安装docker
```perl
# 提前配置好yum仓库
# 1.阿里云自带仓库 2.阿里云提供的docker专属repo仓库

curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos

# 查看源中可用版本
yum list docker-ce --showduplicates | sort -r
```
<a name="1ed81b3b"></a>
# 我的云服务器上安装
参考阿里云的官方文档：[https://help.aliyun.com/zh/ecs/use-cases/deploy-and-use-docker-on-alibaba-cloud-linux-2-instances](https://help.aliyun.com/zh/ecs/use-cases/deploy-and-use-docker-on-alibaba-cloud-linux-2-instances)
```perl
[root@iZwz9c3vz32cadqqnzkj1cZ /]# lsb_release -a # 查看系统版本信息
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID: AlibabaCloud
Description:    Alibaba Cloud Linux release 3 (Soaring Falcon) 
Release:        3
Codename:       SoaringFalcon
[root@iZwz9c3vz32cadqqnzkj1cZ /]# sudo dnf config-manager --add-repo=https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
Adding repo from: https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@iZwz9c3vz32cadqqnzkj1cZ /]# ls /etc/yum.repos.d/
AliYun.repo  docker-ce.repo  epel.repo
[root@iZwz9c3vz32cadqqnzkj1cZ /]# sudo dnf -y install dnf-plugin-releasever-adapter --repo alinux3-plus
Last metadata expiration check: 0:39:00 ago on Sun 15 Oct 2023 05:52:47 PM CST.
Package dnf-plugin-releasever-adapter-1.0-1.4.al8.noarch is already installed.
Dependencies resolved.
==========================================================================================================================================================================================================================================
 Package                                                                Architecture                                    Version                                               Repository                                             Size
==========================================================================================================================================================================================================================================
Upgrading:
 dnf-plugin-releasever-adapter                                          noarch                                          1.0-2.al8                                             alinux3-plus                                           11 k

Transaction Summary
==========================================================================================================================================================================================================================================
Upgrade  1 Package

Total download size: 11 k
Downloading Packages:
dnf-plugin-releasever-adapter-1.0-2.al8.noarch.rpm                                                                                                                                                         64 kB/s |  11 kB     00:00    
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                      64 kB/s |  11 kB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                                                  1/1 
  Upgrading        : dnf-plugin-releasever-adapter-1.0-2.al8.noarch                                                                                                                                                                   1/2 
  Cleanup          : dnf-plugin-releasever-adapter-1.0-1.4.al8.noarch                                                                                                                                                                 2/2 
  Running scriptlet: dnf-plugin-releasever-adapter-1.0-1.4.al8.noarch                                                                                                                                                                 2/2 
  Verifying        : dnf-plugin-releasever-adapter-1.0-2.al8.noarch                                                                                                                                                                   1/2 
  Verifying        : dnf-plugin-releasever-adapter-1.0-1.4.al8.noarch                                                                                                                                                                 2/2 

Upgraded:
  dnf-plugin-releasever-adapter-1.0-2.al8.noarch                                                                                                                                                                                          

Complete!
[root@iZwz9c3vz32cadqqnzkj1cZ /]# sudo dnf -y install docker-ce --nobest
Last metadata expiration check: 0:01:12 ago on Sun 15 Oct 2023 06:31:49 PM CST.
Dependencies resolved.
==========================================================================================================================================================================================================================================
 Package                                                         Architecture                              Version                                                              Repository                                           Size
==========================================================================================================================================================================================================================================
Installing:
 docker-ce                                                       x86_64                                    3:24.0.6-1.el8                                                       docker-ce-stable                                     24 M
Installing dependencies:
 checkpolicy                                                     x86_64                                    2.9-1.2.al8                                                          alinux3-os                                          348 k
 container-selinux                                               noarch                                    2:2.173.0-1.al8                                                      alinux3-updates                                      56 k
 containerd.io                                                   x86_64                                    1.6.24-3.1.el8                                                       docker-ce-stable                                     34 M
 docker-ce-cli                                                   x86_64                                    1:24.0.6-1.el8                                                       docker-ce-stable                                    7.2 M
 docker-ce-rootless-extras                                       x86_64                                    24.0.6-1.el8                                                         docker-ce-stable                                    4.9 M
 fuse-overlayfs                                                  x86_64                                    1.11-1.0.1.al8                                                       alinux3-updates                                      75 k
 fuse3                                                           x86_64                                    3.2.1-12.2.al8                                                       alinux3-os                                           50 k
 fuse3-libs                                                      x86_64                                    3.6.1-2.el7                                                          epel                                                 82 k
 libcgroup                                                       x86_64                                    0.41-19.2.al8                                                        alinux3-os                                           70 k
 libslirp                                                        x86_64                                    4.4.0-1.al8                                                          alinux3-updates                                      71 k
 policycoreutils-python-utils                                    noarch                                    2.9-14.1.al8                                                         alinux3-updates                                     252 k
 python3-audit                                                   x86_64                                    3.0-0.17.20191104git1c2f876.1.al8                                    alinux3-os                                           86 k
 python3-libsemanage                                             x86_64                                    2.9-6.1.al8                                                          alinux3-updates                                     127 k
 python3-policycoreutils                                         noarch                                    2.9-14.1.al8                                                         alinux3-updates                                     2.2 M
 python3-setools                                                 x86_64                                    4.3.0-3.al8                                                          alinux3-updates                                     651 k
 slirp4netns                                                     x86_64                                    1.2.0-2.al8                                                          alinux3-updates                                      54 k
Installing weak dependencies:
 docker-buildx-plugin                                            x86_64                                    0.11.2-1.el8                                                         docker-ce-stable                                     13 M
 docker-compose-plugin                                           x86_64                                    2.21.0-1.el8                                                         docker-ce-stable                                     13 M

Transaction Summary
==========================================================================================================================================================================================================================================
Install  19 Packages

Total download size: 100 M
Installed size: 378 M
Downloading Packages:
(1/19): libslirp-4.4.0-1.al8.x86_64.rpm                                                                                                                                                                   2.3 MB/s |  71 kB     00:00    
(2/19): container-selinux-2.173.0-1.al8.noarch.rpm                                                                                                                                                        625 kB/s |  56 kB     00:00    
(3/19): fuse-overlayfs-1.11-1.0.1.al8.x86_64.rpm                                                                                                                                                          594 kB/s |  75 kB     00:00    
(4/19): policycoreutils-python-utils-2.9-14.1.al8.noarch.rpm                                                                                                                                              2.3 MB/s | 252 kB     00:00    
(5/19): python3-setools-4.3.0-3.al8.x86_64.rpm                                                                                                                                                             17 MB/s | 651 kB     00:00    
(6/19): python3-libsemanage-2.9-6.1.al8.x86_64.rpm                                                                                                                                                        1.5 MB/s | 127 kB     00:00    
(7/19): slirp4netns-1.2.0-2.al8.x86_64.rpm                                                                                                                                                                9.0 MB/s |  54 kB     00:00    
(8/19): checkpolicy-2.9-1.2.al8.x86_64.rpm                                                                                                                                                                 22 MB/s | 348 kB     00:00    
(9/19): libcgroup-0.41-19.2.al8.x86_64.rpm                                                                                                                                                                 13 MB/s |  70 kB     00:00    
(10/19): fuse3-3.2.1-12.2.al8.x86_64.rpm                                                                                                                                                                  678 kB/s |  50 kB     00:00    
(11/19): python3-policycoreutils-2.9-14.1.al8.noarch.rpm                                                                                                                                                   16 MB/s | 2.2 MB     00:00    
(12/19): python3-audit-3.0-0.17.20191104git1c2f876.1.al8.x86_64.rpm                                                                                                                                       445 kB/s |  86 kB     00:00    
(13/19): docker-buildx-plugin-0.11.2-1.el8.x86_64.rpm                                                                                                                                                     6.4 MB/s |  13 MB     00:02    
(14/19): docker-ce-cli-24.0.6-1.el8.x86_64.rpm                                                                                                                                                            4.7 MB/s | 7.2 MB     00:01    
(15/19): docker-ce-rootless-extras-24.0.6-1.el8.x86_64.rpm                                                                                                                                                5.0 MB/s | 4.9 MB     00:00    
(16/19): docker-ce-24.0.6-1.el8.x86_64.rpm                                                                                                                                                                4.1 MB/s |  24 MB     00:05    
(17/19): fuse3-libs-3.6.1-2.el7.x86_64.rpm                                                                                                                                                                519 kB/s |  82 kB     00:00    
(18/19): docker-compose-plugin-2.21.0-1.el8.x86_64.rpm                                                                                                                                                    5.8 MB/s |  13 MB     00:02    
(19/19): containerd.io-1.6.24-3.1.el8.x86_64.rpm                                                                                                                                                          4.6 MB/s |  34 MB     00:07    
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                      13 MB/s | 100 MB     00:07     
warning: /var/cache/dnf/docker-ce-stable-ab4061364e2cf0db/packages/containerd.io-1.6.24-3.1.el8.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
Docker CE Stable - x86_64                                                                                                                                                                                  10 kB/s | 1.6 kB     00:00    
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35
 From       : https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                                                  1/1 
  Installing       : docker-compose-plugin-2.21.0-1.el8.x86_64                                                                                                                                                                       1/19 
  Running scriptlet: docker-compose-plugin-2.21.0-1.el8.x86_64                                                                                                                                                                       1/19 
  Installing       : fuse3-libs-3.6.1-2.el7.x86_64                                                                                                                                                                                   2/19 
  Running scriptlet: fuse3-libs-3.6.1-2.el7.x86_64                                                                                                                                                                                   2/19 
  Installing       : docker-buildx-plugin-0.11.2-1.el8.x86_64                                                                                                                                                                        3/19 
  Running scriptlet: docker-buildx-plugin-0.11.2-1.el8.x86_64                                                                                                                                                                        3/19 
  Installing       : docker-ce-cli-1:24.0.6-1.el8.x86_64                                                                                                                                                                             4/19 
  Running scriptlet: docker-ce-cli-1:24.0.6-1.el8.x86_64                                                                                                                                                                             4/19 
  Installing       : python3-audit-3.0-0.17.20191104git1c2f876.1.al8.x86_64                                                                                                                                                          5/19 
  Running scriptlet: libcgroup-0.41-19.2.al8.x86_64                                                                                                                                                                                  6/19 
  Installing       : libcgroup-0.41-19.2.al8.x86_64                                                                                                                                                                                  6/19 
  Running scriptlet: libcgroup-0.41-19.2.al8.x86_64                                                                                                                                                                                  6/19 
  Installing       : fuse3-3.2.1-12.2.al8.x86_64                                                                                                                                                                                     7/19 
  Installing       : fuse-overlayfs-1.11-1.0.1.al8.x86_64                                                                                                                                                                            8/19 
  Running scriptlet: fuse-overlayfs-1.11-1.0.1.al8.x86_64                                                                                                                                                                            8/19 
  Installing       : checkpolicy-2.9-1.2.al8.x86_64                                                                                                                                                                                  9/19 
  Installing       : python3-setools-4.3.0-3.al8.x86_64                                                                                                                                                                             10/19 
  Installing       : python3-libsemanage-2.9-6.1.al8.x86_64                                                                                                                                                                         11/19 
  Installing       : python3-policycoreutils-2.9-14.1.al8.noarch                                                                                                                                                                    12/19 
  Installing       : policycoreutils-python-utils-2.9-14.1.al8.noarch                                                                                                                                                               13/19 
  Running scriptlet: container-selinux-2:2.173.0-1.al8.noarch                                                                                                                                                                       14/19 
  Installing       : container-selinux-2:2.173.0-1.al8.noarch                                                                                                                                                                       14/19 
  Running scriptlet: container-selinux-2:2.173.0-1.al8.noarch                                                                                                                                                                       14/19 
  Installing       : containerd.io-1.6.24-3.1.el8.x86_64                                                                                                                                                                            15/19 
  Running scriptlet: containerd.io-1.6.24-3.1.el8.x86_64                                                                                                                                                                            15/19 
  Installing       : libslirp-4.4.0-1.al8.x86_64                                                                                                                                                                                    16/19 
  Installing       : slirp4netns-1.2.0-2.al8.x86_64                                                                                                                                                                                 17/19 
  Installing       : docker-ce-rootless-extras-24.0.6-1.el8.x86_64                                                                                                                                                                  18/19 
  Running scriptlet: docker-ce-rootless-extras-24.0.6-1.el8.x86_64                                                                                                                                                                  18/19 
  Installing       : docker-ce-3:24.0.6-1.el8.x86_64                                                                                                                                                                                19/19 
  Running scriptlet: docker-ce-3:24.0.6-1.el8.x86_64                                                                                                                                                                                19/19 
  Running scriptlet: container-selinux-2:2.173.0-1.al8.noarch                                                                                                                                                                       19/19 
  Running scriptlet: docker-ce-3:24.0.6-1.el8.x86_64                                                                                                                                                                                19/19 
  Verifying        : container-selinux-2:2.173.0-1.al8.noarch                                                                                                                                                                        1/19 
  Verifying        : fuse-overlayfs-1.11-1.0.1.al8.x86_64                                                                                                                                                                            2/19 
  Verifying        : libslirp-4.4.0-1.al8.x86_64                                                                                                                                                                                     3/19 
  Verifying        : policycoreutils-python-utils-2.9-14.1.al8.noarch                                                                                                                                                                4/19 
  Verifying        : python3-libsemanage-2.9-6.1.al8.x86_64                                                                                                                                                                          5/19 
  Verifying        : python3-policycoreutils-2.9-14.1.al8.noarch                                                                                                                                                                     6/19 
  Verifying        : python3-setools-4.3.0-3.al8.x86_64                                                                                                                                                                              7/19 
  Verifying        : slirp4netns-1.2.0-2.al8.x86_64                                                                                                                                                                                  8/19 
  Verifying        : checkpolicy-2.9-1.2.al8.x86_64                                                                                                                                                                                  9/19 
  Verifying        : fuse3-3.2.1-12.2.al8.x86_64                                                                                                                                                                                    10/19 
  Verifying        : libcgroup-0.41-19.2.al8.x86_64                                                                                                                                                                                 11/19 
  Verifying        : python3-audit-3.0-0.17.20191104git1c2f876.1.al8.x86_64                                                                                                                                                         12/19 
  Verifying        : containerd.io-1.6.24-3.1.el8.x86_64                                                                                                                                                                            13/19 
  Verifying        : docker-buildx-plugin-0.11.2-1.el8.x86_64                                                                                                                                                                       14/19 
  Verifying        : docker-ce-3:24.0.6-1.el8.x86_64                                                                                                                                                                                15/19 
  Verifying        : docker-ce-cli-1:24.0.6-1.el8.x86_64                                                                                                                                                                            16/19 
  Verifying        : docker-ce-rootless-extras-24.0.6-1.el8.x86_64                                                                                                                                                                  17/19 
  Verifying        : docker-compose-plugin-2.21.0-1.el8.x86_64                                                                                                                                                                      18/19 
  Verifying        : fuse3-libs-3.6.1-2.el7.x86_64                                                                                                                                                                                  19/19 

Installed:
  checkpolicy-2.9-1.2.al8.x86_64         container-selinux-2:2.173.0-1.al8.noarch      containerd.io-1.6.24-3.1.el8.x86_64       docker-buildx-plugin-0.11.2-1.el8.x86_64         docker-ce-3:24.0.6-1.el8.x86_64                       
  docker-ce-cli-1:24.0.6-1.el8.x86_64    docker-ce-rootless-extras-24.0.6-1.el8.x86_64 docker-compose-plugin-2.21.0-1.el8.x86_64 fuse-overlayfs-1.11-1.0.1.al8.x86_64             fuse3-3.2.1-12.2.al8.x86_64                           
  fuse3-libs-3.6.1-2.el7.x86_64          libcgroup-0.41-19.2.al8.x86_64                libslirp-4.4.0-1.al8.x86_64               policycoreutils-python-utils-2.9-14.1.al8.noarch python3-audit-3.0-0.17.20191104git1c2f876.1.al8.x86_64
  python3-libsemanage-2.9-6.1.al8.x86_64 python3-policycoreutils-2.9-14.1.al8.noarch   python3-setools-4.3.0-3.al8.x86_64        slirp4netns-1.2.0-2.al8.x86_64                  

Complete!

[root@iZwz9c3vz32cadqqnzkj1cZ /]# sudo docker -v
Docker version 24.0.6, build ed223bc

# sudo systemctl start docker # 启动docker
# sudo systemctl enable docker # 设置开机自启

[root@iZwz9c3vz32cadqqnzkj1cZ /]# sudo systemctl status docker #查看Docker是否启动。
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2023-10-15 18:56:17 CST; 59s ago
     Docs: https://docs.docker.com
 Main PID: 670285 (dockerd)
    Tasks: 7
   Memory: 26.6M
   CGroup: /system.slice/docker.service
           └─670285 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Oct 15 18:56:16 iZwz9c3vz32cadqqnzkj1cZ systemd[1]: Starting Docker Application Container Engine...
Oct 15 18:56:16 iZwz9c3vz32cadqqnzkj1cZ dockerd[670285]: time="2023-10-15T18:56:16.992798331+08:00" level=info msg="Starting up"
Oct 15 18:56:17 iZwz9c3vz32cadqqnzkj1cZ dockerd[670285]: time="2023-10-15T18:56:17.094758684+08:00" level=info msg="Loading containers: start."
Oct 15 18:56:17 iZwz9c3vz32cadqqnzkj1cZ dockerd[670285]: time="2023-10-15T18:56:17.541104230+08:00" level=info msg="Loading containers: done."
Oct 15 18:56:17 iZwz9c3vz32cadqqnzkj1cZ dockerd[670285]: time="2023-10-15T18:56:17.576965885+08:00" level=warning msg="Not using native diff for overlay2, this may cause degraded performance for building images: kernel has CONFIG_OVE>
Oct 15 18:56:17 iZwz9c3vz32cadqqnzkj1cZ dockerd[670285]: time="2023-10-15T18:56:17.577162802+08:00" level=info msg="Docker daemon" commit=1a79695 graphdriver=overlay2 version=24.0.6
Oct 15 18:56:17 iZwz9c3vz32cadqqnzkj1cZ dockerd[670285]: time="2023-10-15T18:56:17.577231448+08:00" level=info msg="Daemon has completed initialization"
Oct 15 18:56:17 iZwz9c3vz32cadqqnzkj1cZ dockerd[670285]: time="2023-10-15T18:56:17.623477617+08:00" level=info msg="API listen on /run/docker.sock"
Oct 15 18:56:17 iZwz9c3vz32cadqqnzkj1cZ systemd[1]: Started Docker Application Container Engine.
lines 1-19/19 (END)

[root@iZwz9c3vz32cadqqnzkj1cZ /]# docker images # 查看镜像文件
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
[root@iZwz9c3vz32cadqqnzkj1cZ /]# 
[root@iZwz9c3vz32cadqqnzkj1cZ /]# docker ps # 查看容器
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@iZwz9c3vz32cadqqnzkj1cZ /]# 

[root@iZwz9c3vz32cadqqnzkj1cZ /]# docker version
Client: Docker Engine - Community
 Version:           24.0.6
 API version:       1.43
 Go version:        go1.20.7
 Git commit:        ed223bc
 Built:             Mon Sep  4 12:33:07 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.6
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.7
  Git commit:       1a79695
  Built:            Mon Sep  4 12:32:10 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.24
  GitCommit:        61f9fd88f79f081d64d6fa3bb1a0dc71ec870523
 runc:
  Version:          1.1.9
  GitCommit:        v1.1.9-0-gccaecfc
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
[root@iZwz9c3vz32cadqqnzkj1cZ /]# docker -v
Docker version 24.0.6, build ed223bc
[root@iZwz9c3vz32cadqqnzkj1cZ /]#
```
<a name="7a7573ee"></a>
# 配置docker加速器
用于加速镜像文件的下载<br />使用docker首要操作就是获取 镜像文件，默认下载是从Docker Hub下载，网速较慢，国内很多云服务商都提供了加速器服务，阿里云加速器，Daocloud加速器，灵雀云加速器。
```perl
# 1. 修改docker配置文件，我们选用七牛云镜像站
cat /etc/docker/daemon.json # 如果文件不存在，先创建或者touch一下

mkdir -p /etc/docker
vi /etc/docker/daemon.json
{
	"registry-mirrors": [
		"https://8xpk5wnt.mirror.aliyuncs.com"
	]
}
# 也可以
## 配置源加速
## https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

# 启动docker
systemctl enable docker
systemctl daemon-reload
```
<a name="c37a7d8d"></a>
# 启动第一个docker容器
```bash
# 1.获取镜像
# 2.运行镜像，生成容器，你想要的的应用，就运行在容器中
```
Nginx web服务器，运行出一个80端口的网站
```bash
# 在宿主机上，运行nginx
1. 开启服务器
2. 在服务器上安装好运行nginx所需的依赖关系
3. 安装nginx  yum install nginx -y
4. 修改nginx配置文件
5. 启动nginx
6. 客户端去访问nginx

比较耗时的。。。
```
如果让你用docker运行nginx，该怎么玩
```bash
1. 获取镜像，获取是从你配置好的docker镜像站中，去拉取nginx镜像

# 先搜索一下 ，镜像文件，是否存在
docker search nginx
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker search nginx
NAME                                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                                             Official build of Nginx.                        19127     [OK]       
unit                                              Official build of NGINX Unit: Universal Web …   15        [OK]       
nginxinc/nginx-unprivileged                       Unprivileged NGINX Dockerfiles                  125                  
nginx/nginx-ingress                               NGINX and  NGINX Plus Ingress Controllers fo…   81                   
nginx/nginx-prometheus-exporter                   NGINX Prometheus Exporter for NGINX and NGIN…   33                   
nginx/unit                                        NGINX Unit is a dynamic web and application …   64                   
nginxinc/nginx-s3-gateway                         Authenticating and caching gateway based on …   2                    
nginx/nginx-ingress-operator                      NGINX Ingress Operator for NGINX and NGINX P…   1                    
nginxinc/amplify-agent                            NGINX Amplify Agent docker repository           1                    
nginx/nginx-quic-qns                              NGINX QUIC interop                              1                    
nginxinc/ingress-demo                             Ingress Demo                                    4                    
nginxproxy/nginx-proxy                            Automated Nginx reverse proxy for docker con…   112                  
nginxproxy/acme-companion                         Automated ACME SSL certificate generation fo…   124                  
bitnami/nginx                                     Bitnami nginx Docker Image                      176                  [OK]
bitnami/nginx-ingress-controller                  Bitnami Docker Image for NGINX Ingress Contr…   30                   [OK]
ubuntu/nginx                                      Nginx, a high-performance reverse proxy & we…   102                  
nginxinc/nginmesh_proxy_debug                                                                     0                    
nginxproxy/docker-gen                             Generate files from docker container meta-da…   12                   
kasmweb/nginx                                     An Nginx image based off nginx:alpine and in…   6                    
nginxinc/mra-fakes3                                                                               0                    
rancher/nginx-ingress-controller                                                                  11                   
nginxinc/ngx-rust-tool                                                                            0                    
nginxinc/mra_python_base                                                                          0                    
nginxinc/nginmesh_proxy_init                                                                      0                    
rancher/nginx-ingress-controller-defaultbackend 

# 拉取，下载镜像
docker pull nginx
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete 
a9edb18cadd1: Pull complete 
589b7251471a: Pull complete 
186b1aaa4aa6: Pull complete 
b4df32aa5a72: Pull complete 
a0bcbecc962e: Pull complete 
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

# 如果已安装对应的images
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Image is up to date for nginx:latest
docker.io/library/nginx:latest

# 查看本地的docker镜像有哪些
docker image ls
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        latest    605c77e624dd   21 months ago   141MB

# 删除镜像的命令：docker rmi(rm images) <imageId>
# 等价于 docker image rm <imageId>
docker rmi 605c77e624dd
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker rmi 605c77e624dd
Untagged: nginx:latest
Untagged: nginx@sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Deleted: sha256:605c77e624ddb75e6110f997c58876baa13f8754486b461117934b24a9dc3a85
Deleted: sha256:b625d8e29573fa369e799ca7c5df8b7a902126d2b7cbeb390af59e4b9e1210c5
Deleted: sha256:7850d382fb05e393e211067c5ca0aada2111fcbe550a90fed04d1c634bd31a14
Deleted: sha256:02b80ac2055edd757a996c3d554e6a8906fd3521e14d1227440afd5163a5f1c4
Deleted: sha256:b92aa5824592ecb46e6d169f8e694a99150ccef01a2aabea7b9c02356cdabe7c
Deleted: sha256:780238f18c540007376dd5e904f583896a69fe620876cabc06977a3af4ba4fb5
Deleted: sha256:2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073b41ef9727da6c851f

# 再次拉取镜像
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete 
a9edb18cadd1: Pull complete 
589b7251471a: Pull complete 
186b1aaa4aa6: Pull complete 
b4df32aa5a72: Pull complete 
a0bcbecc962e: Pull complete 
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

# 再次查看镜像
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        latest    605c77e624dd   21 months ago   141MB

# 运行nginx进行，运行出，具体的容器，然后这个容器中，就跑着一个nginx服务了
# 运行镜像的命令，参数如下
docker run 参数    镜像的名字/id

# -d 后台运行容器(不会占用一个前台命令窗口)
# -p 80:80 端口映射， 宿主机端口：容器内端口， 你访问宿主机的这个端口，也就访问到了容器内的端口
docker run -d -p 80:80 nginx

# 查看端口有没有打开
netstat -tunlp 
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1003/sshd           
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:7778            0.0.0.0:*               LISTEN      1549/nginx: master  
tcp        0      0 0.0.0.0:55403           0.0.0.0:*               LISTEN      1022/rpc.statd      
tcp        0      0 0.0.0.0:5355            0.0.0.0:*               LISTEN      507/systemd-resolve 
tcp        0      0 0.0.0.0:46607           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd           
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1549/nginx: master  
tcp        0      0 0.0.0.0:20048           0.0.0.0:*               LISTEN      1029/rpc.mountd     
tcp6       0      0 :::43507                :::*                    LISTEN      -                   
tcp6       0      0 :::2049                 :::*                    LISTEN      -                   
tcp6       0      0 :::53033                :::*                    LISTEN      1022/rpc.statd      
tcp6       0      0 :::5355                 :::*                    LISTEN      507/systemd-resolve 
tcp6       0      0 :::111                  :::*                    LISTEN      1/systemd           
tcp6       0      0 :::20048                :::*                    LISTEN      1029/rpc.mountd     
udp        0      0 127.0.0.1:778           0.0.0.0:*                           1022/rpc.statd      
udp        0      0 0.0.0.0:43914           0.0.0.0:*                           -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           507/systemd-resolve 
udp        0      0 0.0.0.0:111             0.0.0.0:*                           1/systemd           
udp        0      0 0.0.0.0:5355            0.0.0.0:*                           507/systemd-resolve 
udp        0      0 127.0.0.1:323           0.0.0.0:*                           539/chronyd         
udp        0      0 0.0.0.0:46526           0.0.0.0:*                           1022/rpc.statd      
udp        0      0 0.0.0.0:20048           0.0.0.0:*                           1029/rpc.mountd     
udp6       0      0 :::41660                :::*                                1022/rpc.statd      
udp6       0      0 :::47028                :::*                                -                   
udp6       0      0 :::111                  :::*                                1/systemd           
udp6       0      0 :::5355                 :::*                                507/systemd-resolve 
udp6       0      0 ::1:323                 :::*                                539/chronyd         
udp6       0      0 :::20048                :::*                                1029/rpc.mountd  

docker run -d -p 81:80 nginx
# 运行结果如下，docker run 命令，会返回一个容器的id
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker run -d -p 81:80 nginx
7967bfc13affb3d005f781a23eff25d858fb59539c940f2a29a404f2ba4ddf28

# 通过 本台电脑(linux服务器的公网ip)的ip(有域名也可以通过域名) + 端口 查看运行的服务 - (需要开启安全组对应的端口权限)
# 或者通过：curl 172.17.0.1:81 发http请求查看响应
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# curl 172.17.0.1:81
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>

# 查看容器是否运行
docker ps
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                               NAMES
7967bfc13aff   nginx     "/docker-entrypoint.…"   8 minutes ago   Up 8 minutes   0.0.0.0:81->80/tcp, :::81->80/tcp   lucid_blackburn

# 查看运行的容器列表
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                               NAMES
7967bfc13aff   nginx     "/docker-entrypoint.…"   44 seconds ago   Up 43 seconds   0.0.0.0:81->80/tcp, :::81->80/tcp   lucid_blackburn

# 再查看服务器的端口情况，可以看到81端口运行的程序名并不是叫nginx，而是docker-proxy，说明是docker为nginx服务做了一层代理
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:81              0.0.0.0:*               LISTEN      676504/docker-proxy

# 可以尝试停止容器，看下结果
docker stop 容器id

[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                               NAMES
7967bfc13aff   nginx     "/docker-entrypoint.…"   18 minutes ago   Up 18 minutes   0.0.0.0:81->80/tcp, :::81->80/tcp   lucid_blackburn
# 停止具体的容器进程
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker stop 7967bfc13aff
7967bfc13aff
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@iZwz9c3vz32cadqqnzkj1cZ ~]#
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# curl 172.17.0.1:81
curl: (7) Failed to connect to 172.17.0.1 port 81: Connection refused

# 可以通过已停掉的容器id，重新运行
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker start 7967bfc13aff
7967bfc13aff
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS                               NAMES
7967bfc13aff   nginx     "/docker-entrypoint.…"   20 minutes ago   Up 4 seconds   0.0.0.0:81->80/tcp, :::81->80/tcp   lucid_blackburn
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# curl 172.17.0.1:81
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# 

# ps -ef 查看当前服务器运行的进程
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0  2022 ?        00:06:05 /usr/lib/systemd/systemd --switched-root --system --deserialize 17
root           2       0  0  2022 ?        00:00:02 [kthreadd]
root           3       2  0  2022 ?        00:00:00 [rcu_gp]
root           4       2  0  2022 ?        00:00:00 [rcu_par_gp]
root           6       2  0  2022 ?        00:00:00 [kworker/0:0H-events_highpri]
root           8       2  0  2022 ?        00:00:00 [mm_percpu_wq]
root           9       2  0  2022 ?        00:00:00 [rcu_tasks_rude_]
root          10       2  0  2022 ?        00:00:00 [rcu_tasks_trace]
root          11       2  0  2022 ?        00:02:19 [ksoftirqd/0]
root          12       2  0  2022 ?        00:20:54 [rcu_sched]
root          13       2  0  2022 ?        00:00:06 [migration/0]
root          14       2  0  2022 ?        00:00:00 [cpuhp/0]
root          16       2  0  2022 ?        00:00:00 [kdevtmpfs]
root          17       2  0  2022 ?        00:00:00 [netns]
root          18       2  0  2022 ?        00:00:00 [kauditd]
root          19       2  0  2022 ?        00:00:05 [khungtaskd]
root          20       2  0  2022 ?        00:00:00 [oom_reaper]
root          21       2  0  2022 ?        00:00:00 [writeback]
root          22       2  0  2022 ?        00:05:51 [kcompactd0]
root          23       2  0  2022 ?        00:00:00 [ksmd]
root          24       2  0  2022 ?        00:01:30 [khugepaged]
root          25       2  0  2022 ?        00:00:00 [memcg_wmark]
root          39       2  0  2022 ?        00:00:00 [cryptd]
root          74       2  0  2022 ?        00:00:00 [kintegrityd]
root          75       2  0  2022 ?        00:00:00 [kblockd]
root          76       2  0  2022 ?        00:00:00 [blkcg_punt_bio]
root          77       2  0  2022 ?        00:00:00 [tpm_dev_wq]
root          78       2  0  2022 ?        00:00:00 [md]
root          79       2  0  2022 ?        00:00:00 [edac-poller]
root          80       2  0  2022 ?        00:00:00 [watchdogd]
root          82       2  0  2022 ?        00:02:01 [kworker/0:1H-kblockd]
root          98       2  0  2022 ?        00:00:00 [kswapd0]
root          99       2  0  2022 ?        00:00:00 [zombie_memcg_re]
root         100       2  0  2022 ?        00:00:00 [kidled]
root         102       2  0  2022 ?        00:00:00 [kthrotld]
root         103       2  0  2022 ?        00:00:00 [acpi_thermal_pm]
root         105       2  0  2022 ?        00:00:00 [kmpath_rdacd]
root         106       2  0  2022 ?        00:00:00 [kaluad]
root         107       2  0  2022 ?        00:00:00 [ipv6_addrconf]
root         108       2  0  2022 ?        00:00:00 [kstrp]
root         124       2  0  2022 ?        00:00:00 [zswap-shrink]
root         129       2  0  2022 ?        00:06:09 [load_calc]
root         130       2  0  2022 ?        00:00:00 [kworker/u5:0-xprtiod]
root         303       2  0  2022 ?        00:00:00 [ata_sff]
root         304       2  0  2022 ?        00:00:00 [scsi_eh_0]
root         310       2  0  2022 ?        00:00:00 [scsi_tmf_0]
root         311       2  0  2022 ?        00:00:00 [scsi_eh_1]
root         312       2  0  2022 ?        00:00:00 [scsi_tmf_1]
root         338       2  0  2022 ?        00:04:38 [jbd2/vda1-8]
root         339       2  0  2022 ?        00:00:00 [jbd2-ckpt/vda1-]
root         340       2  0  2022 ?        00:00:00 [ext4-rsv-conver]
root         439       1  0  2022 ?        00:02:33 /usr/lib/systemd/systemd-journald
root         472       2  0  2022 ?        00:00:00 [rpciod]
root         473       2  0  2022 ?        00:00:00 [xprtiod]
root         498       1  0  2022 ?        00:00:33 /usr/lib/systemd/systemd-udevd
rpc          505       1  0  2022 ?        00:00:23 /usr/bin/rpcbind -w -f
systemd+     507       1  0  2022 ?        00:00:27 /usr/lib/systemd/systemd-resolved
root         512       1  0  2022 ?        00:00:00 /usr/sbin/rpc.idmapd
root         513       1  0  2022 ?        00:00:00 /usr/sbin/nfsdcld
dbus         522       1  0  2022 ?        00:01:17 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
nscd         527       1  0  2022 ?        00:01:13 /usr/sbin/nscd
root         529       1  0  2022 ?        00:00:15 /usr/local/cloudmonitor/CmsGoAgent.linux-amd64
root         530       1  0  2022 ?        00:00:51 /usr/lib/systemd/systemd-logind
chrony       539       1  0  2022 ?        00:01:27 /usr/sbin/chronyd
polkitd      540       1  0  2022 ?        00:00:06 /usr/lib/polkit-1/polkitd --no-debug
root         541       1  0  2022 ?        00:00:00 /usr/sbin/iprdump --daemon
rngd         546       1  0  2022 ?        00:00:13 /sbin/rngd -f --fill-watermark=0
root         549       1  0  2022 ?        00:00:00 /usr/sbin/iprupdate --daemon
root         551       1  0  2022 ?        00:00:00 /usr/sbin/iprinit --daemon
root         605     529  0  2022 ?        11:45:52 CmsGoAgent-Worker start
root         703       1  0  2022 ?        00:06:38 /usr/sbin/NetworkManager --no-daemon
root         706       1  0  2022 ?        00:00:00 /usr/sbin/oddjobd -n -p /run/oddjobd.pid -t 300
root         710       1  0  2022 ?        00:52:43 /usr/libexec/platform-python -Es /usr/sbin/tuned -l -P
root         719       1  0  2022 ?        00:00:00 /usr/sbin/gssproxy -D
root         997       1  0  2022 ?        00:18:05 /usr/sbin/rsyslogd -n
root        1003       1  0  2022 ?        00:00:11 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr,aes256-cbc,aes128-gcm@openssh.com,aes128-ctr,aes128-cbc -oMACs=hmac-sha2-256-etm@openssh.c
root        1008       1  0  2022 ?        00:00:00 /usr/sbin/atd -f
root        1009       1  0  2022 ?        00:00:33 /usr/sbin/crond -n
rpcuser     1022       1  0  2022 ?        00:00:00 /usr/sbin/rpc.statd
root        1029       1  0  2022 ?        00:00:00 /usr/sbin/rpc.mountd
root        1034       1  0  2022 tty1     00:00:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root        1035       1  0  2022 ttyS0    00:00:00 /sbin/agetty -o -p -- \u --keep-baud 115200,38400,9600 ttyS0 vt220
root        1051       2  0  2022 ?        00:00:00 [kworker/u5:1]
root        1054       2  0  2022 ?        00:00:00 [lockd]
root        1066       2  0  2022 ?        00:00:00 [nfsd]
root        1067       2  0  2022 ?        00:00:00 [nfsd]
root        1069       2  0  2022 ?        00:00:00 [nfsd]
root        1071       2  0  2022 ?        00:00:00 [nfsd]
root        1073       2  0  2022 ?        00:00:00 [nfsd]
root        1074       2  0  2022 ?        00:00:00 [nfsd]
root        1075       2  0  2022 ?        00:00:00 [nfsd]
root        1077       2  0  2022 ?        00:00:00 [nfsd]
root        1198       1  0  2022 ?        00:00:06 /usr/lib/systemd/systemd --user
root        1201    1198  0  2022 ?        00:00:00 (sd-pam)
root        1415       1  0  2022 ?        00:04:11 PM2 v5.2.0: God Daemon (/root/.pm2)
root        1549       1  0  2022 ?        00:00:00 nginx: master process ./nginx
nobody      3548    1549  0  2022 ?        00:00:14 nginx: worker process
root      579181       1  0 Aug24 ?        00:27:18 /usr/local/aegis/aegis_update/AliYunDunUpdate
root      579204       1  0 Aug24 ?        02:34:45 /usr/local/aegis/aegis_client/aegis_11_73/AliYunDun
root      579215       1  0 Aug24 ?        04:23:55 /usr/local/aegis/aegis_client/aegis_11_73/AliYunDunMonitor
root      579543       1  0 Aug24 ?        00:30:34 /usr/local/share/aliyun-assist/2.2.3.515/aliyun-service
root      579679       1  0 Aug24 ?        00:09:47 /usr/local/share/assist-daemon/assist_daemon
root      670274       1  0 Oct15 ?        00:00:47 /usr/bin/containerd
root      670285       1  0 Oct15 ?        00:00:30 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root      676274       2  0 22:07 ?        00:00:00 [kworker/u4:0-ext4-rsv-conversion]
root      676342    1003  0 22:50 ?        00:00:00 sshd: root [priv]
root      676344  676342  0 22:50 ?        00:00:00 sshd: root@pts/0
root      676345  676344  0 22:50 pts/0    00:00:00 -bash
root      676421       2  0 23:00 ?        00:00:00 [kworker/0:2-cgroup_destroy]
root      676504  670285  0 23:06 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 81 -container-ip 172.17.0.2 -container-port 80
root      676508  670285  0 23:06 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 81 -container-ip 172.17.0.2 -container-port 80
root      676535       1  0 23:06 ?        00:00:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 7967bfc13affb3d005f781a23eff25d858fb59539c940f2a29a404f2ba4ddf28 -address /run/containerd/containerd.sock
root      676553  676535  0 23:06 ?        00:00:00 nginx: master process nginx -g daemon off;
root      676559       2  0 23:06 ?        00:00:00 [kworker/0:3-events_power_efficient]
101       676609  676553  0 23:06 ?        00:00:00 nginx: worker process
root      676644       2  0 23:07 ?        00:00:00 [kworker/u4:1-ext4-rsv-conversion]
root      676736       2  0 23:14 ?        00:00:00 [kworker/u4:2-events_unbound]
root      676756       2  0 23:15 ?        00:00:00 [kworker/0:0-cgroup_pidlist_destroy]
root      676787  676345  0 23:16 pts/0    00:00:00 ps -ef
```
![linux公网id-81.png](https://cdn.nlark.com/yuque/0/2023/png/38602487/1698163851271-89cfd68d-451f-4f46-829f-786d3cc37176.png#averageHue=%23f7f5f4&clientId=uc4f951ca-bf18-4&from=paste&height=420&id=ue56a6c47&originHeight=420&originWidth=689&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34482&status=done&style=none&taskId=ub6bbd293-aa02-428f-af04-38e409fae4f&title=&width=689)<br />![dadifeihua.com-81.png](https://cdn.nlark.com/yuque/0/2023/png/38602487/1698163864502-f7f147e3-e383-43d7-8a62-67d6d4af9c62.png#averageHue=%23f7f5f4&clientId=uc4f951ca-bf18-4&from=paste&height=410&id=u20db0386&originHeight=410&originWidth=685&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34280&status=done&style=none&taskId=u3103c2ca-adb1-438f-b09d-b5e4950fe20&title=&width=685)
<a name="14340c30"></a>
# 镜像操作命令 - docker image --help
```
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker image --help

Usage:  docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Download an image from a registry
  push        Upload an image to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
```
镜像的管理<br />容器的管理<br />。。。
<a name="B8xtX"></a>
# docker生命周期
学习docker的核心要素、搞明白、镜像image、容器！<br />![docker-生命周期.png](https://cdn.nlark.com/yuque/0/2023/png/38602487/1698163762567-041b029a-adec-43f6-8f2d-8b8612ffabeb.png#averageHue=%23bfbdba&clientId=uc4f951ca-bf18-4&from=paste&height=1080&id=u8df92b6d&originHeight=1080&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=728687&status=done&style=none&taskId=ucef82730-4354-42f7-9553-2d78318ae2f&title=&width=1920)
<a name="yEzG2"></a>
# 彻底学明白，docker镜像的原理
我们在获取redis镜像的时候，发现是下载了多行信息，最终又得到了一个完整的镜像文件
```perl
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker pull redis
Using default tag: latest
latest: Pulling from library/redis
a2abf6c4d29d: Already exists 
c7a4e4382001: Pull complete 
4044b9ba67c9: Pull complete 
c8388a79482f: Pull complete 
413c8bb60be2: Pull complete 
1abfd3011519: Pull complete 
Digest: sha256:db485f2e245b5b3329fdc7eff4eb00f913e09d8feb9ca720788059fdc2ed8339
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```
来看一看docker的镜像原理
<a name="X5uKd"></a>
# 我们用的centos7系统长什么样-利用docker容器运行不同发行版镜像
我们一直以来，使用vmware虚拟机，安装的系统，是一个完整的系统文件，包括2部分

- linux内核，作用是提供操作系统的基本功能，和机器硬件进行交互，读取磁盘数据，管理网络
- centos7发行版，作用是提供软件功能，例如yum安装包管理等

因此，linux内核 + centos发行版，就组成了一个系统，让我们用户使用。<br />centos，ubuntu。。<br />是否有一个办法，可以灵活的替换发行版，让我们使用不同的【系统】？
> 那么docker就实现了这个功能，技术手段就是docker images
> 内核都公用宿主机的内核，上层的发行版，自由替换

```perl
# 查看系统的2大部分
# 发行版代码
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# cat /etc/redhat-release
Alibaba Cloud Linux release 3 (Soaring Falcon) 
# 内核代码
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# uname -r
5.10.60-9.al8.x86_64
```
![docker-linux系统分层.png](https://cdn.nlark.com/yuque/0/2023/png/38602487/1698460669793-4deacbd7-d533-4709-83fa-22a3caaf357d.png#averageHue=%23bab7b6&clientId=u1f928851-4832-4&from=paste&height=1079&id=ucc530136&originHeight=1079&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=590012&status=done&style=none&taskId=uda050ab2-caaa-4e73-ab4b-60d00df7b94&title=&width=1920)<br />![docker-linux分层-使用docker容器技术.png](https://cdn.nlark.com/yuque/0/2023/png/38602487/1698461138939-645c4fd9-1694-4603-8c56-75a5a67cde15.png#averageHue=%23f6f6f5&clientId=u1f928851-4832-4&from=paste&height=1079&id=u403a9949&originHeight=1079&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=478987&status=done&style=none&taskId=ud0da48a6-cec4-4062-bc07-8c84d56c17d&title=&width=1920)<br />快速实践，使用docker，来切换不同的发行版，内核使用的都是宿主机的内核
```perl
# 利用docker获取不同的发行版镜像
docker pull centos:7.8.2003
docker pull centos

docker pull ubuntu:

[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker pull centos
Using default tag: latest
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest

[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker pull centos:7.8.2003
7.8.2003: Pulling from library/centos
9b4ebb48de8d: Pull complete 
Digest: sha256:8540a199ad51c6b7b51492fa9fee27549fd11b3bb913e888ab2ccf77cbb72cc1
Status: Downloaded newer image for centos:7.8.2003
docker.io/library/centos:7.8.2003

[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
7b1a6ab2e44d: Pull complete 
Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest

[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker images
REPOSITORY   TAG        IMAGE ID       CREATED         SIZE
nginx        latest     605c77e624dd   22 months ago   141MB
redis        latest     7614ae9453d1   22 months ago   113MB
ubuntu       latest     ba6acccedd29   2 years ago     72.8MB
centos       latest     5d0da3dc9764   2 years ago     231MB
centos       7.8.2003   afb6fca791e0   3 years ago     203MB

# 确认当前的宿主机发行版
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# cat /etc/redhat-release
Alibaba Cloud Linux release 3 (Soaring Falcon) 
# 查看宿主机的内核版本
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# uname -r
5.10.60-9.al8.x86_64

# 运行centos7.8.2003发行版
# 运行容器，且进入容器内
# 参数 解释 
# docker run imageId 运行镜像，生成容器
# -i 交互式命令操作(进入到命令行中了，可以敲击命令进行交互，如ls)
# -t 开启一个终端(可以在后续的终端里输入命令，开启了终端才能在命令行输入命令)
# afb6fca791e0 是镜像的id
# bash 进入容器后，执行的命令 （进入容器后执行一个命令时，如ls，谁帮你去执行的，所以这就需要一个bash解释器，帮你解释ls命令是干嘛的）
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker run -it  afb6fca791e0 bash
[root@336019b68720 /]# 
# 上面的[root@xxx]发生变化，表明已进入到（交互式的）容器空间内了
[root@336019b68720 /]# ls
anaconda-post.log  dev  home  lib64  mnt  proc  run   srv  tmp  var
bin                etc  lib   media  opt  root  sbin  sys  usr
[root@336019b68720 /]# 

# 查看容器内的发行版本
[root@336019b68720 /]# cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)
# 查看容器的内核版本
[root@336019b68720 /]# uname -r
5.10.60-9.al8.x86_64

# 可以看到，发行版确实就是自己运行的镜像id:afb6fca791e0，对应的centos7.8.2003镜像，所以容器的发行版版本也是这个
# 而内核版本确实也是宿主机的内核版本

# 容器内的交互式命令行中，可以通过exit命令退出容器空间，回到宿主机
[root@336019b68720 /]# exit
exit
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# 

[root@iZwz9c3vz32cadqqnzkj1cZ ~]# cat /etc/redhat-release
Alibaba Cloud Linux release 3 (Soaring Falcon) 
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# 
```
![docker-使用docker的发行版镜像-centos7.8.2003.png](https://cdn.nlark.com/yuque/0/2023/png/38602487/1698462494323-23528ae6-8cf9-475c-b0b2-ae50d6349355.png#averageHue=%23f5f5f4&clientId=u1f928851-4832-4&from=paste&height=1080&id=u70527d45&originHeight=1080&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=494661&status=done&style=none&taskId=u009b41a0-7a06-4fc0-be34-3faa3c17bc8&title=&width=1920)
```perl
# 运维小王，想玩一玩ubuntu了
# 直接去获取ubuntu镜像即可

[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker images
REPOSITORY   TAG        IMAGE ID       CREATED         SIZE
nginx        latest     605c77e624dd   22 months ago   141MB
redis        latest     7614ae9453d1   22 months ago   113MB
ubuntu       latest     ba6acccedd29   2 years ago     72.8MB
centos       latest     5d0da3dc9764   2 years ago     231MB
centos       7.8.2003   afb6fca791e0   3 years ago     203MB
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker run -it  ba6acccedd29 bash
root@8d93a7a1fd53:/# ls
bin   dev  home  lib32  libx32  mnt  proc  run   srv  tmp  var
boot  etc  lib   lib64  media   opt  root  sbin  sys  usr
root@8d93a7a1fd53:/# cat /etc/redhat-release
cat: /etc/redhat-release: No such file or directory
# 这个时候想查看redhat-release，它就没了，因为它不是红帽系统
# 替换使用下面命令
root@8d93a7a1fd53:/# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.3 LTS"
root@8d93a7a1fd53:/# 

# 当然ubuntu发行版系统里，也没有yum命令，但有apt命令
root@8d93a7a1fd53:/# yum
bash: yum: command not found
root@8d93a7a1fd53:/# apt
apt 2.0.6 (amd64)
Usage: apt [options] command

apt is a commandline package manager and provides commands for
searching and managing as well as querying information about packages.
It provides the same functionality as the specialized APT tools,
like apt-get and apt-cache, but enables options more suitable for
interactive use by default.

Most used commands:
  list - list packages based on package names
  search - search in package descriptions
  show - show package details
  install - install packages
  reinstall - reinstall packages
  remove - remove packages
  autoremove - Remove automatically all unused packages
  update - update list of available packages
  upgrade - upgrade the system by installing/upgrading packages
  full-upgrade - upgrade the system by removing/installing/upgrading packages
  edit-sources - edit the source information file
  satisfy - satisfy dependency strings

See apt(8) for more information about the available commands.
Configuration options and syntax is detailed in apt.conf(5).
Information about how to configure sources can be found in sources.list(5).
Package and version choices can be expressed via apt_preferences(5).
Security details are available in apt-secure(8).
                                        This APT has Super Cow Powers.
root@8d93a7a1fd53:/# exit
exit
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# 
```
```perl
# 再来玩一玩opensuse系统

# 搜索一下镜像
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker search suse
NAME                                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
eclipse/platformreleng-opensuse-gtk3-metacity   Platform project for SuSE Linux                 0                    
susescc/notary                                  Docker notary                                   1                    [OK]
susedoc/ci                                      Now in OBS: https://registry.opensuse.org/cg…   1                    
mstormo/suse                                    Various SUSE baseimages which are not availa…   10                   [OK]
splatform/suse-os-image-stemcell-builder        Environment to build the openSUSE bosh os-im…   0                    
susesamples/backend-nodejs                                                                      0                    
splatform/suse-stratos                                                                          0                    
susesamples/myjenkins                                                                           0                    
susetest/hsc-proxy                                                                              0                    
susetest/hsc-postgres                                                                           0                    
susesamples/spring-petclinic-demo                                                               0                    
susetest/hsc-preflight-job                                                                      0                    
yuzhenpin/suse-11-sp3-x86_64-java               Add Java support to SUSE 11 SP3 x86_64          1                    [OK]
susetest/hsc-postflight-job                                                                     0                    
suseglobons/ws                                                                                  0                    
susetest/hsc-console                                                                            0                    
susedemo/vlc                                    vlc images for demos                            0                    
susesamples/sles15sp3-openjdk                                                                   0                    
suseru/rmt-airgap                                                                               0                    
susedemo/nginx                                                                                  0                    
susetest/uaa                                                                                    0                    
mstormo/suse_cuda                               SUSE with various CUDA versions for quick bu…   1                    [OK]
susesamples/sles15sp1-openjdk11                                                                 0                    
dhubtesting/suse                                                                                0                    
susetest/scf-etcd                                                                               0                    
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# 

# 并没有一个官方版本的镜像，因为suse是闭源的
# 搜索opensuse
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker search open-suse
NAME      DESCRIPTION   STARS     OFFICIAL   AUTOMATED
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker search opensuse
NAME                           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
opensuse                       DEPRECATED; for current images by the openSU…   336       [OK]       
opensuse/leap                  Official openSUSE Leap Images                   90                   
opensuse/tumbleweed            Official openSUSE Tumbleweed images             69                   
opensuse/portus                Authorization service and frontend for Docke…   92                   [OK]
kasmweb/opensuse-15-desktop    openSUSE Leap 15 desktop for Kasm Workspaces    8                    
opensuse/clair                 CoreOS Clair image based on openSUSE. Has th…   0                    [OK]
opensuse/python                openSUSE base image with python                 0                    [OK]
opensuse/archive               Archive of the old opensuse images from the …   3                    
opensuse/amd64                 x86_64 releases of openSUSE Docker images       9                    
opensuse/ruby                  Ruby Docker images based on openSUSE Leap 42…   3                    [OK]
opensuse/nailed                Collect and visualize product related data f…   2                    [OK]
opensuse/etcd                  etcd image                                      0                    [OK]
opensuse/salt-master           salt-master image                               2                    [OK]
opensuse/salt-minion           salt-minion image                               0                    [OK]
opensuse/salt-api              salt-api image                                  0                    [OK]
opensuse/s390x                 Experimental s390x images of openSUSE           0                    
dokken/opensuse-leap-15        openSUSE Leap 15.X image for kitchen-dokken     0                    
opensuse/dice                  dice - build container for KIWI appliances      0                    
opensuse/dex-operator          A Kubernetes operator for Dex                   1                    [OK]
opensuse/registries-operator   A Kubernetes operator for images registries     1                    [OK]
dokken/opensuse-leap-15.4      openSUSE 15.4 image for use with Test Kitche…   0                    
dokken/opensuse-leap-15.3      openSUSE 15.3 image for use with Test Kitche…   0                    
opensuse/arm64v8                                                               0                    
opensuse/ppc64le                                                               0                    
opensuse/registry              Docker registry based on openSUSE.              0                    [OK]
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# 

# 下载opensuse镜像
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker pull opensuse
Using default tag: latest
latest: Pulling from library/opensuse
cea5c22ff067: Pull complete 
Digest: sha256:0460dd92f97c35cd0875e9cd14b02321002fac14631ea682a0c9f31c4998844d
Status: Downloaded newer image for opensuse:latest
docker.io/library/opensuse:latest
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker images
REPOSITORY   TAG        IMAGE ID       CREATED         SIZE
nginx        latest     605c77e624dd   22 months ago   141MB
redis        latest     7614ae9453d1   22 months ago   113MB
ubuntu       latest     ba6acccedd29   2 years ago     72.8MB
centos       latest     5d0da3dc9764   2 years ago     231MB
centos       7.8.2003   afb6fca791e0   3 years ago     203MB
opensuse     latest     d9e50bf28896   4 years ago     110MB
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# 

# 运行镜像生成容器，并进入容器内部
[root@iZwz9c3vz32cadqqnzkj1cZ ~]# docker run -it d9e50bf28896 bash
59943d8289f8:/ # 
59943d8289f8:/ # 
59943d8289f8:/ #  

# 查看发行版信息
59943d8289f8:/ # cat /etc/
cat: /etc/: Is a directory
59943d8289f8:/ # ls
bin  dev  etc  home  lib  lib64  mnt  opt  proc  root  run  sbin  selinux  srv  sys  tmp  usr  var
59943d8289f8:/ # ls /etc 
HOSTNAME                cups           hosts.deny     modprobe.d            pki          skel
SuSE-release            dbus-1         hosts.equiv    modules-load.d        ppp          ssl
X11                     default        hosts.lpd      motd                  products.d   susehelp.d
YaST2                   defaultdomain  hushlogins     mtab                  profile      sysconfig
aliases                 depmod.d       init.d         netgroup              profile.d    sysctl.conf
aliases.d               dirmngr        inputrc        networks              protocols    sysctl.d
alternatives            environment    insserv.conf   news                  raw          systemd
bash.bashrc             ethers         issue          nsswitch.conf         rc.d         termcap
bash_completion.d       exports        issue.net      opt                   rc.splash    tmpdirs.d
bindresvport.blacklist  filesystems    java           os-release            rc.status    tmpfiles.d
binfmt.d                fstab          krb5.conf      pam.d                 resolv.conf  ttytype
blkid.conf              ftpusers       ld.so.cache    passwd                rpc          uucp
ca-certificates         gai.conf       ld.so.conf     passwd-               rpm          xattr.conf
cron.d                  gnupg          ld.so.conf.d   permissions           securetty    xdg
cron.daily              group          libaudit.conf  permissions.d         security     xinetd.d
cron.hourly             group-         localtime      permissions.easy      selinux      zypp
cron.monthly            host.conf      login.defs     permissions.local     services
cron.weekly             hostname       logrotate.d    permissions.paranoid  shadow
csh.cshrc               hosts          magic          permissions.secure    shadow-
csh.login               hosts.allow    mime.types     pkcs11                shells
59943d8289f8:/ # cat /etc/os-release
NAME="openSUSE Leap"
VERSION="42.3"
ID=opensuse
ID_LIKE="suse"
VERSION_ID="42.3"
PRETTY_NAME="openSUSE Leap 42.3"
ANSI_COLOR="0;32"
CPE_NAME="cpe:/o:opensuse:leap:42.3"
BUG_REPORT_URL="https://bugs.opensuse.org"
HOME_URL="https://www.opensuse.org/"
59943d8289f8:/ # cat /etc/SuSE-release
openSUSE 42.3 (x86_64)
VERSION = 42.3
CODENAME = Malachite
# /etc/SuSE-release is deprecated and will be removed in the future, use /etc/os-release instead
59943d8289f8:/ # 

# opensuse是一个最小化的发行版系统，它没有yum，也没有apt
59943d8289f8:/ # yum
bash: yum: command not found
59943d8289f8:/ # apt
bash: apt: command not found
59943d8289f8:/ # 

59943d8289f8:/ # cat /etc/hostname
59943d8289f8
59943d8289f8:/ # 

# 问题：命令行前面的那串数字是什么呢? 它是容器的主机名
# 容器的主机名一般都会以这个容器的id为命名

```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/38602487/1698464545635-7f58464a-f3f8-4d89-a3d4-5344333f3752.png#averageHue=%23f8f8f7&clientId=u1f928851-4832-4&from=paste&height=1079&id=u6385db3b&originHeight=1079&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=748818&status=done&style=none&taskId=uc16f15c3-460e-4c6a-8318-b46ee9ef8a0&title=&width=1920)<br />小结

1. 一个完整的系统，是由linux的内核+发行版，才组成了一个可以使用的完整的系统
2. 利用docker容器，可以获取不同的发行版镜像，然后基于该镜像，运行出各种容器去使用
> 那么下一节，就来学习，docker镜像的原理

为什么它能够基于我们一个宿主机本体，能够运行出那么多不同的系统来。
