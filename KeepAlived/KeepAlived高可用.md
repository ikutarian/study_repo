# 准备工作

## 安装 killall 的包

编写的 shell 脚本需要用到 `killall` 命令，但是最小化安装 CentOS 7 时，这个命令是不存在的，所以需要安装一下相关的包

```
yum install -y psmisc
```

## 关闭 SELinux

SELinux 太严格了，会导致 vrrp_script 指定的脚本无法执行。而且一般我们都是关闭它的，所以就把它关掉吧。打开 SELinux 配置文件

```
vi /etc/selinux/config
```

把 `SELINUX=permissive` 它改成 `SELINUX=disabled`

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

然后重启服务器

```
reboot
```

## 关闭防火墙

执行以下命令关闭防火墙

```
systemctl stop firewalld
```

准备两台服务器，一台作为 MASTER 使用，一台作为 BACKUP 使用

# 查看服务器的网卡和 IP

在 MASTER 节点服务器上输入 `ip a` 命令

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:64:e4:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.107/24 brd 192.168.1.255 scope global noprefixroute dynamic ens32
       valid_lft 7163sec preferred_lft 7163sec
    inet6 fe80::d908:55b8:5beb:d7e3/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

可以看到网卡名称是 `ens32`，ip 地址是 `192.168.1.107`

在 BACKUP 节点服务器上输入 `ip a` 命令

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:25:48:1f brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.109/24 brd 192.168.1.255 scope global noprefixroute dynamic ens32
       valid_lft 6852sec preferred_lft 6852sec
    inet6 fe80::d908:55b8:5beb:d7e3/64 scope link tentative noprefixroute dadfailed
       valid_lft forever preferred_lft forever
    inet6 fe80::4ed9:3fdc:7d01:2296/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

可以看到网卡名称是 `ens32`，ip 地址是 `192.168.1.109`

# 约定 VIP

我们约定 VIP（虚拟 IP）是 `192.168.1.108`。而且通过上面的 `ip a` 命令输入结果得知

|节点|网卡|IP|
|:--:|:--:|:--:|
|MASTER|ens32|192.168.1.107|
|BACKUP|ens32|192.168.1.109|

# 安装 keepalived

在 MASTER 和 BACKUP 服务器上都执行以下命令安装 keepalived

```
yum install -y keepalived
```

# 查看配置文件的位置

可以用这条命令查看安装 keepalived 之后成了哪些文件

```
rpm -ql keepalived
```

输出

```
[root@localhost ~]# rpm -ql keepalived
/etc/keepalived
/etc/keepalived/keepalived.conf
/etc/sysconfig/keepalived
/usr/bin/genhash
/usr/lib/systemd/system/keepalived.service
/usr/libexec/keepalived
/usr/sbin/keepalived
/usr/share/doc/keepalived-1.3.5
/usr/share/doc/keepalived-1.3.5/AUTHOR
/usr/share/doc/keepalived-1.3.5/CONTRIBUTORS
/usr/share/doc/keepalived-1.3.5/COPYING
/usr/share/doc/keepalived-1.3.5/ChangeLog
/usr/share/doc/keepalived-1.3.5/NOTE_vrrp_vmac.txt
/usr/share/doc/keepalived-1.3.5/README
/usr/share/doc/keepalived-1.3.5/TODO
/usr/share/doc/keepalived-1.3.5/keepalived.conf.SYNOPSIS
/usr/share/doc/keepalived-1.3.5/samples
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.HTTP_GET.port
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.IPv6
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.SMTP_CHECK
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.SSL_GET
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.fwmark
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.inhibit
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.misc_check
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.misc_check_arg
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.quorum
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.sample
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.status_code
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.track_interface
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.virtual_server_group
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.virtualhost
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.vrrp
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.vrrp.localcheck
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.vrrp.lvs_syncd
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.vrrp.routes
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.vrrp.rules
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.vrrp.scripts
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.vrrp.static_ipaddress
/usr/share/doc/keepalived-1.3.5/samples/keepalived.conf.vrrp.sync
/usr/share/doc/keepalived-1.3.5/samples/sample.misccheck.smbcheck.sh
/usr/share/man/man1/genhash.1.gz
/usr/share/man/man5/keepalived.conf.5.gz
/usr/share/man/man8/keepalived.8.gz
/usr/share/snmp/mibs/KEEPALIVED-MIB.txt
/usr/share/snmp/mibs/VRRP-MIB.txt
/usr/share/snmp/mibs/VRRPv3-MIB.txt
```

从第二条输入结果可以看到 keepalived 的配置文件是 `/etc/keepalived/keepalived.conf`

# keepalived 配置文件的结构

```
! Configuration File for keepalived

# 全局配置
global_defs {
   # 邮件通知的配置，当 keepalived 切换 VIP 会发邮件通知
   notification_email { 
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   # 邮件通知的配置
   
   # 路由ID：当前安装 keepalived 的主机的标识符，必须是全局唯一的
   router_id LVS_DEVEL
   
   # 严格遵守 VRRP 协议
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   # 严格遵守 VRRP 协议
}

# vrrp 实例，并且名称是 VI_1
vrrp_instance VI_1 {
    state MASTER # 节点的状态，要么是 MASTER 要么是 BACKUP
    interface eth0 # 绑定的网卡名称
    virtual_router_id 51 # 要确保 MASTER 节点和 BACKUP 节点中相同名称的 vrrp 实例，它的 virtual_router_id 要相同
    priority 100 # 优先级/权重，值越大，优先级越高。MASTER 挂了之后，剩下的 BACKUP 谁的优先级高，谁就可以成为 MASTER。MASTER 节点设置为 100，其他的 BACKUP 设置比 100 小就行
    advert_int 1 # MASTER 与 BACKUP 之间的同步检查的时间间隔，默认是1s
    authentication { # 认证授权的密码，防止非法节点的进入。MASTER 与 BACKUP 要设置成一样
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress { # 虚拟IP列表，可以配置多个虚拟IP
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}

# 下面的内容没用，不用关注
virtual_server 192.168.200.100 443 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.201.100 443 {
        weight 1
        SSL_GET {
            url {
              path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    sorry_server 192.168.200.200 1358

    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

根据实际情况：

- 当 keepalived 切换 VIP 不需要发邮件通知
- 不需要严格遵守 VRRP 协议

精简之后的配置文件如下

对于 MASTER 节点

```
! Configuration File for keepalived

global_defs {
   # 107 是 IP 的后三位数
   router_id keep_107
}

vrrp_instance VI_1 {
    state MASTER
    interface ens32
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.108
    }
}
```

对于 BACKUP 节点

```
! Configuration File for keepalived

global_defs {
   # 107 是 IP 的后三位数
   router_id keep_109
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens32
    virtual_router_id 51
    priority 98 # BACKUP 节点的优先级要比 MASTER 小
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.108
    }
}
```

# 启动 keepalived

在 MASTER 和 BACKUP 节点上都输入

```
systemctl start keepalived
```

如果要查看启动日志，可以再开一个窗口输入以下命令实时查看

```
tail -f /var/log/messages
```

# 验证 keepalived 是否生效

在 MASTER 节点的服务器上输入 `ip a`，可以看到 ens32 网卡上多了一个 `192.168.1.108` 的 IP

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:64:e4:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.107/24 brd 192.168.1.255 scope global noprefixroute dynamic ens32
       valid_lft 6739sec preferred_lft 6739sec
    inet 192.168.1.108/32 scope global ens32
       valid_lft forever preferred_lft forever
    inet6 fe80::d908:55b8:5beb:d7e3/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

在 BACKUP 节点的服务器上只有一个 IP

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:25:48:1f brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.109/24 brd 192.168.1.255 scope global noprefixroute dynamic ens32
       valid_lft 6519sec preferred_lft 6519sec
    inet6 fe80::d908:55b8:5beb:d7e3/64 scope link tentative noprefixroute dadfailed
       valid_lft forever preferred_lft forever
    inet6 fe80::4ed9:3fdc:7d01:2296/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

如果在 MASTER 节点上把 keepalived 关掉，VIP 能迁移到 BACKUP 节点上的话，就说明 keepalived 配置成功了。现在就来试一下，输入以下命令停掉 MASTER 节点的 keepalived

```
systemctl stop keepalived
```

然后在 MASTER 节点上输入 `ip a` 查看 VIP 的情况

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:64:e4:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.107/24 brd 192.168.1.255 scope global noprefixroute dynamic ens32
       valid_lft 6739sec preferred_lft 6739sec
    inet 192.168.1.108/32 scope global ens32
       valid_lft forever preferred_lft forever
    inet6 fe80::d908:55b8:5beb:d7e3/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

怎么回事？为什么 VIP 还在 MASTER 节点上？是不是 keepalived 没有彻底关掉？输入命令 `ps -ef | grep keepalived` 查看一下进程信息

```
root      11040      1  0 22:16 ?        00:00:00 keepalived
root      11041  11040  0 22:16 ?        00:00:00 keepalived
root      11042  11040  0 22:16 ?        00:00:00 keepalived
root      11054   1335  0 22:17 pts/0    00:00:00 grep --color=auto keepalived
```

可以看到 keepalived 进程还是存在的

## 解决无法彻底关闭 keepalived 的问题

打开 `/usr/lib/systemd/system/keepalived.service` 文件

```
[Unit]
Description=LVS and VRRP High Availability Monitor
After=syslog.target network-online.target

[Service]
Type=forking
PIDFile=/var/run/keepalived.pid
KillMode=process
EnvironmentFile=-/etc/sysconfig/keepalived
ExecStart=/usr/sbin/keepalived $KEEPALIVED_OPTIONS
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

把这一行注释掉 `KillMode=process`

```
[Unit]
Description=LVS and VRRP High Availability Monitor
After=syslog.target network-online.target

[Service]
Type=forking
PIDFile=/var/run/keepalived.pid
# KillMode=process
EnvironmentFile=-/etc/sysconfig/keepalived
ExecStart=/usr/sbin/keepalived $KEEPALIVED_OPTIONS
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

然后再执行 `systemctl daemon-reload` 命令

现在在 MASTER 节点上执行 `systemctl stop keepalived` 关掉 keepalived 之后，VIP 就迁移到了 BACKUP 节点上了

# KeepAlived + Nginx 高可用

## 双机主备：两台机器 + 1 个 VIP

**在 MASTER 节点**编写一个监测 nginx 状态的脚本 `/etc/keepalived/chk_nginx.sh`

```bash
#!/bin/bash

A=`ps -C nginx --no-header |wc -l`
if [ $A -eq 0 ];then
    killall keepalived
fi
```

并赋予执行权限

```
chmod +x chk_nginx.sh
```

打开 keepalived 配置文件，配置脚本

```
! Configuration File for keepalived

global_defs {
   router_id keep_107
}

## 添加
vrrp_script chk_nginx {
    script "/etc/keepalived/chk_nginx.sh"
    interval 2
    weight 2
}
## 添加

vrrp_instance VI_1 {
    state MASTER
    interface ens32
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.108
    }
	## 添加
    track_script  {
        chk_nginx
    }
	## 添加
}
```

## 双主热备：两台机器 +  2 个 VIP

### 前提

一个域名比如 `www.example.com` 可以配置多个 IP，并且可以配置每个 IP 的解析权重。具体的配置可以在 DNS 解析厂商的后台上配置，每家厂商的方式大同小异

### 2 个 VIP

- 192.168.1.108
- 192.168.1.110

### MASTER 节点的 keepalived 的配置

增加一个 VIP（192.168.1.110）的 BACKUP 的 VRRP 实例

```
! Configuration File for keepalived

global_defs {
   router_id keep_107
}

vrrp_script chk_nginx {
    script "/etc/keepalived/chk_nginx.sh"
    interval 2
    weight 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens32
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.108
    }
    track_script  {
        chk_nginx
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface ens32
    virtual_router_id 52
    priority 98
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.110
    }
}
```

### BACKUP 节点的 keepalived 的配置

增加一个 VIP（192.168.1.110）的 MASTER 的 VRRP 实例

```
! Configuration File for keepalived

global_defs {
   router_id keep_109
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens32
    virtual_router_id 51
    priority 98
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.108
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface ens32
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.110
    }
}
```

