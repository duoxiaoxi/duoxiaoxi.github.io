# Git

## 安装

### 源码包 安装

```shell
# 依赖插件
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev
# 下载源代码 官方网站
http://git-scm.com/
# 下载源码包 镜像推荐
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.3.tar.gz
# 解压缩
tar -zxf git-1.7.2.2.tar.gz
# 编译 & 安装
make prefix=/usr/local all
make prefix=/usr/local install
```

### Linux 安装

```shell
# CentOS
yum install git-core
# Ubuntu
apt-get install git
```

### Windows 安装

[下载]( https://git-scm.com/ ) > 下一步 > 下一步 ... 即可

## 配置

### 配置文件

| 配置文件         | 设置命令                            | 查看命令                    |
| ---------------- | ----------------------------------- | --------------------------- |
| `/etc/gitconfig` | `git config --system {key} {value}` | `git config --system{key}`  |
| `~/.gitconfig`   | `git config --global {key} {value}` | `git config --global {key}` |
| `.git/config`    | `git config {key} {value}`          | `git config {key}`          |

### 基本配置

```shell
# 配置用户提交代码时候使用的用户名和邮箱
git config [--global,--system,] user.name "duoxiaoxi"
git config [--global,--system,] user.email 2862580679@qq.com
# 配置用户默认使用的编辑器, 例如commit的时候会输入信息
git config [--global,--system,] core.editor emacs
# 设置Git默认使用的差异分析工具
git config [--global,--system,] merge.tool [vimdiff,xxdiff,opendiff]
# 查看已经拥有的所有配置
git config [--global,--system,] --list
# 查看具体某个变量的配置
git config [--global,--system,] {key}
```

## 基础

### 初始化

```shell
# 新建一个本地裆裤
git init
# 克隆已经有的仓库
git clone https://github.com/duoxiaoxi/test.git
```

### 文件状态

```shell

# On branch master
    nothing to commit (working directory clean)
    
# On branch master
    
    nothing added to commit but untracked files present (use "git add" to track)

    
    
    # On branch master
    nothing to commit (working directory clean)
    # Changes to be committed:
    # (use "git reset HEAD <file>..." to unstage)
    #
    # new file: README
    # modified: benchmarks.rb
    # deleted: grit.gemspec
    #
    # Changes not staged for commit:
    # (use "git add <file>..." to update what will be committed)
    #
    # modified: benchmarks.rb
    # deleted: grit.gemspec
    #
    # Untracked files:
    # (use "git add <file>..." to include in what will be committed)
    #
    # README

```

























