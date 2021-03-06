# iptables 防火墙

- why

学会使用 iptables 防火墙保护服务器。

## iptables

### 定义

iptables 是一种用在 Linux 操作系统上的防火墙。你可以用它去设置一些规则，数据包经过网卡的时候都会被检查，系统会根据你设置的防火墙规则来决定到底怎么样处理这些要进来的，要出去的，或者要转发的数据包。

### 规则

规则的英文是 rules ，在每条防火墙的规则里面，你可以使用一些特性去描述一下数据包，比如数据包使用的传输协议的类型，数据包的来源，去向，或者端口号等等。

如果到网卡上的数据包符合我们设置的某条规则，就会执行在这条规则里面指定的动作。

### 动作

这个动作的英文名叫 target。

- 它可以是 ACCEPT ，授受这个数据包，也就是让数据包通过，去它想去的地方。

- 可以是 DROP ，丢掉这个数据包，就是直接把数据包扔掉，比如有人想连接我们的服务器，如果我们把他发过来的数据包扔了，他那边儿的连接会一直等着，直到连接超时，因为 DROP 这个动作不会作出回应。

- 也可以是 REJECT，拒绝数据包，这个动作会给对方一个回应，告诉对方被拒绝了。

## Chain：防火墙规则的分组

Chain 有点像是防火墙规则的分组，不过要注意每个 Chain 里的规则的顺序很重要。

### 三种默认 Chain

我们可以根据需求自己去创建 Chain ，默认有三个：

- INPUT，表示输入，这个 Chain 里面处理的是进入到服务器上的数据包。

- OUTPUT，输出，它处理的是服务器向外面发送的数据包。比如你想安装或者升级服务器上的某个服务，服务器本身就会向外面去发送一些数据包。

- FORWARD，转发，这个分组里的数据包就是要从一个地方转发到另一个地方。

### Chain 下面的规则

这些 chain，也就是规则的分组里面，可以包含多条规则。

#### iptables 如何检查规则

在检查数据包的时候，分组里的规则的顺序是很重要的。

比如一个数据包会从第一条规则开始检查。
如果它符合某个 chain 里的某条规则的描述，就会执行对应的动作，也就是接受，丢弃或者拒绝。这样就不会再继续去检查这个数据包是不是符合其它的规则了。

#### 如何编写规则

- 所以在分组里排在前面的规则，它的优先级比后面的规则的优先级更高一些，这样我们就需要在设计规则的时候，把一些比较通用的规则放到前面。

- 然后把更具体的规则放在后面，比如你想禁用从某个 IP 地址发送进来的数据 ，这应该就是比较具体的规则。

- 每个分组都可以设置一个默认的动作，这样如果数据包不符合分组里的所有的防火墙规则，就会执行这个默认的动作，这个动作可以是接受，也可以是丢弃。

## 端口

### 端口扫描

```bash
nmap -sT [ip/localhost]
```

- s：scan
- T：tcp

### 查看端口上的连接

#### 系统上的端口文件

```bash
cat /etc/services
```

- 过滤

```bash
cat /etc/services | grep 80
```

#### 查看端口是哪个 service 打开的

```bash
netstat -anp [| grep 80]
```

- a:all
- n:number，使用数字的表示方式
- p:progress，显示进程 id 和名字

#### 查看当前正在使用 tcp 的连接

```bash
netstat -ntp
```

## 管理规则

### 查看防火墙规则

#### 查看现有的防火墙分组和规则

- 基础

```bash
iptables --list
```

```bash
iptables -L
```

- 加行号

```bash
iptables -L --line-numbers
```

- 查看指定的 chain

```bash
iptables -L INPUT
```

- 查看更详细的信息（例如被拦截的数据包数量、大小）

```bash
iptables -L -v
```

-v:--verbose,Provide more information when used with the list option.

- 清除数据包信息

```bash
sudo iptables -Z
```

-Z：zero

#### 查看现有的规则是执行什么样的 iptables 命令生成的

```bash
iptables --list-rules
```

```bash
iptables -S
```

### 修改

#### 改变 chain 默认 policy

- 默认的 policy

也就是如果一个数据包不符合所有规则里的描述就会执行这个默认的 Policy 指定的动作。

- 比如我们改变 FORWARD 这个 Chain 的默认的 policy。

```bash
iptables -P FORWARD DROP
```

- -P 这个选项表示 policy
- FORWARD，是 Chain 的名字
- DROP

### 增加（追加/插入）

#### 追加：禁止指定来源（IP）的访问

```bash
sudo iptables -A INPUT -s 192.168.33.161 -j DROP
```

- -A 就是 append ，追加的意思，添加到现有规则的底部。
- INPUT 表示规则应用的 Chain， INPUT 就是进入到服务器的数据
- -s ，表示 source ，用它指定一下数据包的来源，后面的 ip 就是数据包的来源
- -j 表示 jump，后面是要执行的动作

这条规则的意思就是，扔掉来自 192.168.33.161 这个 IP 地址发送过来的所有的数据。

#### 追加：禁止指定来源访问指定协议的端口号

```bash
sudo iptables -A INPUT -s 192.168.33.0/24 -p tcp --dport 80 -j DROP
```

- 192.168.33.0/24：从 192.168.33.1 到 192.168.33.254 的 ip。
- -p：指定传输数据包用的协议（protocol），设置成 tcp ，意思就是数据包是用 tcp 这种协议传输的。
- --dport ，它表示 destination-port，目标端口，这里设置成 80。

这条规则的意思就是，禁止 192.168.66.1 到 192.168.66.254 这些 IP 地址使用 tcp 协议在 80 端口传输数据。

#### 插入：增加到指定位置上

```bash
sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

- -I：insert，插入规则
- INPUT：插入哪个 chain
- 1：插入到第一行
- -i： --in-interface，命名接收数据包的接口

### 删除

#### 删除指定的规则

```bash
iptables -D INPUT 4
```

- -D：delete，删除规则
- INPUT：删除哪个 chain 的规则
- 4：删除第 4 行的规则

#### 清空所有规则

```bash
iptables -F INPUT
```

- F：--flush，冲洗的意思，删除所有规则，Delete all the rules one-by-one.
- INPUT：删除 INPUT 中的所有规则

### 保存规则

添加完规则，需要保存，否则系统重启，规则就没了。

- 保存

```bash
service iptables save
```

会保存到如下位置：

```bash
/etc/sysconfig/iptables
```

## 参考资料

- [资料 1](https://ninghao.net/video/3542#info)

- [资料 2](https://www.linode.com/docs/security/firewalls/control-network-traffic-with-iptables/#basic-iptables-rulesets-for-ipv4-and-ipv6)
