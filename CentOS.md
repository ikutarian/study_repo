## 使用网易源

首先备份 `/etc/yum.repos.d/CentOS-Base.repo`

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

下载对应版本 repo 文件, 放入 `/etc/yum.repos.d/` 中

```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
```

生成缓存

```bash
yum clean all
yum makecache
```

### 参考

- [CentOS镜像使用帮助](http://mirrors.163.com/.help/centos.html)

## 安装 OpenJDK

输入以下命令

```bash
yum install -y java-1.8.0-openjdk-devel
```

验证是否安装成功

```bash
javac
```

如果有输出信息说明 OpenJDK 安装成功了

### 参考

- [How to download and install prebuilt OpenJDK packages](http://openjdk.java.net/install/)