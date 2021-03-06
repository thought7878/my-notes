# 4.systemctl：Linux 服务管理

# systemd/systemctl

### systemd

systemd 这个东西是很多 linux 系统的一种管理方法。比如 CentOS、Ubuntu，都会使用它去管理系统。操作系统启动以后会第一个启动 systemd 这个进程。在系统运行期间，systemd 可以去管理一些其它的一些东西。比如它可以管理系统上的服务，我们可以让某些服务在系统启动以后自动启动，或者也可以手工的去启动或停止服务。

### systemctl

`systemctl` 这个工具提供了一些方法可以让我们去控制 systemd 这个系统。比如用它去启动服务，停止服务等等。

# units

systemd 可以管理的东西叫 `units` ，中文可以翻译成单元，单元就是 systemd 可以管理的一些资源。

### units 类型

单元有不同的类型，比如`服务类型的单元`、`设备类型的单元`等等。

### units 单元文件-配置文件

- 定义

这些单元都会有一个配置文件，叫`单元文件`。

在这个文件里描述了单元的一些信息，比如单元的名字，在什么时候启动它，启动它的话要执行哪个东西，启动它之前要先做什么等等。

- 文件后缀名

这些单元文件有不同的后缀，比如 `.service` 就是服务类型的单元，`.device` 就是设备类型的单元。

### 查看系统当前载入的一些单元

可以使用命令：

```bash
systemctl list-units
```

#### 返回的内容分成了几栏：

- UNIT 就是单元的名字

注意它们都有一个特别的后缀，.device ，.mount，.service 等等。

- LOAD 这栏表示这个单元是不是被加载了

也就是 systemd 是不是正确的载入了单元的配置文件，因为我们执行的命令就是显示所有加载的单元，所以这栏内容都会是 loaded。

- ACTIVE 这栏内容表示单元的状态

一般我们可以使用它来判断这个单元是不是启动了，active 表示启动，inactive 表示没启动。

- SUB 这栏是更底层的状态

它会显示更详细的关于单元状态的信息，不同类型的单元，它的 SUB 这栏内容会有一些不一样。比如这个 .device，设备类型的单元，它的 SUB 这栏内容显示 plugged ，应该就表示这个设备正常接入了，下面的这个 .mount ，挂载类型的单元，显示 mounted ，表示正常挂载。这个 service，服务类型的单元，显示 running ，表示这个服务正常的运行了。

- DESCRIPTION 这栏内容是关于这个单元的一个简介

现在我们执行的这个命令显示的就是 systemd 已经加载的一些单元。

### 查看系统里面全部的可以使用的单元文件

可以使用这个命令：

```bash
systemctl list-unit-files
```

- 只想查看服务类型的单元：

```bash
systemctl list-unit-files --type service
```

# 管理服务

## 查看服务状态

### 查看关于服务的更详细的状态(\*)

- basic

```bash
systemctl status serviceName
```

- 获得更详细的信息

```bash
sudo systemctl status serviceName
```

- 显示全部的信息

```bash
sudo systemctl status serviceName -l
```

### 查看服务是不是在运行中

```bash
systemctl is-active serviceName

```

表示要查看单元的状态是不是在运行中，后面加上要查看的单元的名字。平时在使用 systemctl 的时候可以不需要指定单元的后缀。

返回的是 active，表示这个服务正在运行。返回的是 unknown，意思是这个服务没有启动。

### 查看服务是不是开机自动启动

```bash
systemctl is-enabled serviceName
```

服务是不是开机自动启动，后面加上要查看的服务。

返回的是 enabled，表示这个服务会开机自启动。返回的都是 disabled，表示这个服务不会开机自启动。

## 启动与停止服务

### 启动服务

```bash
(sudo) systemctl start serviceName
```

### 停止服务

```bash
(sudo) systemctl stop serviceName
```

## 启用与禁用自启动

### 启用开机自启动

```bash
sudo systemctl enable serviceName
```

### 禁用开机自启动

```bash
sudo systemctl disable serviceName
```

## 重启或重新载入服务

### 重新载入服务

不停止服务，就可以让服务的配置生效。

```bash
sudo systemctl reload serviceName
```

### 重启服务

完全停止服务，再重新启动。

```bash
sudo systemctl restart serviceName
```

## 不让服务启动

既不能开机自启动，也不能手工启动。

```bash
sudo systemctl mask serviceName
```

- 恢复

```bash
sudo systemctl unmask serviceName
```
