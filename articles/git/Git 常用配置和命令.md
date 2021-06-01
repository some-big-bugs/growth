<h1> Git | Git 常用配置和命令 </h1>

# 系列文章目录

---

**Table of Contents**

- [系列文章目录](#系列文章目录)
- [常用配置](#常用配置)
  - [git修改文件夹/文件名大小写](#git修改文件夹文件名大小写)
- [2. 配置](#2-配置)
  - [1. 初次运行 Git 前的配置](#1-初次运行-git-前的配置)
    - [小结](#小结)
  - [2. 用户信息](#2-用户信息)
  - [3. 别名](#3-别名)
  - [4. 查看已经存在的配置](#4-查看已经存在的配置)
- [3. 获取帮助](#3-获取帮助)
- [4. 参考与扩展阅读](#4-参考与扩展阅读)

---

# 常用配置

## git修改文件夹/文件名大小写

git config core.ignorecase false

# 2. 配置
## 1. 初次运行 Git 前的配置
既然已经在系统上安装了 Git，你会想要做几件事来定制你的 Git 环境。 每台计算机上只需要配置一次，程序升级时会保留配置信息。 你可以在任何时候再次通过运行命令来修改它们。

Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置：

1. `/etc/gitconfig` 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果使用带有 `--system` 选项的 `git config` 时，它会从此文件读写配置变量。

2. `~/.gitconfig` 或 `~/.config/git/config` 文件：只针对当前用户。 可以传递 `--global` 选项让 `Git` 读写此文件。 `Windows` 对应文件在用户目录下。

3. 当前使用仓库的 `Git` 目录中的 `config` 文件（就是 `.git/config`）：只针对该仓库。

每一个级别覆盖上一级别的配置，所以 .git/config 的配置变量会覆盖 /etc/gitconfig 中的配置变量。

在 `Windows` 系统中，`Git` 会查找 `$HOME` 目录下（一般情况下是 `C:\Users\$USER`）的 `.gitconfig` 文件。 Git 同样也会寻找 `/etc/gitconfig` 文件，但只限于 MSys 的根目录下，即安装 Git 时所选的目标位置。
### 小结
```
git config --system:操作:/etc/gitconfig文件,全局的.对所有用户都生效==系统级别
git config --global:操作:~/.gitconfig文件,仅仅对自己生效==用户级别
git config :操作仓库:.git/config文件==其实就是当前项目
```

## 2. 用户信息
当安装完 Git 应该做的第一件事就是设置你的用户名称与邮件地址。 这样做很重要，因为每一个 Git 的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改，通过该信息，其它开发者可以和你进行交流联系，当然要是你一直写`BUG`,别人也能知道是你在写，然后拿你`祭天`。
```bash
$ git config --global user.name "zyndev"
$ git config --global user.email "zyndev@gmail.com"
```
再次强调，如果使用了 --global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。此时你提交代码时就会带上该用户信息。

当你有多个仓库，例如你在 `github` 有个项目(例如：[manual](https://github.com/zyndev/manual))，这时你想使用个人邮箱(例如：gmail)，然后你还有个公司项目(例如：com-xxx-xxx),这时需要使用公司邮箱(例如：xxx.com)，这时你可以在那个项目目录下运行没有 --global 选项的命令来配置。
```
- 配置 github 仓库 manual
$ cd manual
$ git config user.name "zyndev"
$ git config user.email "zyndev@gmail.com"

- 配置 公司 仓库 com-xxx-xxx
$ cd com-xxx-xxx
$ git config user.name "张瑀楠"
$ git config user.email "yunan.zhang@xxx.com"
```

很多 GUI(git-extension) 工具都会在第一次运行时帮助你配置这些信息。

## 3. 别名
可以通过起别名缩短命令
例如：
```
给 commit 起个别名为 ci 则使用 git ci 等同于 git commit
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.br branch
```
这种方式对于组合长命令来说很有帮助，例如：

```
git config --global alias.graph "log --online --graph"
```
> 命令如果出现空格需要用引号引起来

## 4. 查看已经存在的配置
```
git config --system --list
git config --global --list
git config --list
```

# 3. 获取帮助
若你使用 Git 时需要获取帮助，有三种方法可以找到 Git 命令的使用手册：
```bash
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```
例如，要想获得 config 命令的手册，执行
```
$ git help config
```

# 4. 参考与扩展阅读
[GIT-SCM 官方文档](https://git-scm.com/doc)

[廖雪峰 GIT 教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)
