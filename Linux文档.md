# Linux文档

#### 通配符

在 Shell 命令中，通常**用通配符表达式来匹配文件名**，而**用正则表达式来匹配一段文本内容**。以 `grep` 命令为例，`grep` 命令可以在指定的文件中，挑选出和表达式匹配的那些行，其中指定文件是用的通配符表达式，而文本内容的匹配用的是正则表达式。

**Shell 中可以使用的通配符如下：**

|   通配符    | 含义                                 | 实例                                                         |
| :---------: | ------------------------------------ | ------------------------------------------------------------ |
|      *      | 匹配 0 或多个字符                    | `a*b`，a与b之间可以有任意长度的任意字符, 也可以一个也没有, 如 aabcb, axyzb, a012b, ab |
|      ?      | 匹配任意单个字符                     | `a?b`，a与b之间有且只有一个字符, 可以是任意字符, 如 aab, abb, acb, a0b |
|   [list]    | 匹配 list 中的任意单个字符           | `a[xyz]b`，a与b之间必须也只能有一个字符, 但只能是 x 或 y 或 z, 如 axb, ayb, azb。 |
|   [^list]   | 匹配除 list 中的任意单一字符         | `a[^0-9]b`，a与b之间必须也只能有一个字符, 但不能是阿拉伯数字, 如 axb, aab, a-b。 |
|   [c1-c2]   | 匹配 c1-c2 中的任意单一字符          | `a[0-9]b`，匹配0与9之间其中一个字符，如 a0b, a1b... a9b      |
| {s1,s2,...} | 匹配 s1 或 s2 (或更多)中的一个字符串 | `a{abc,xyz,123}b`，a与b之间只能是abc或xyz或123这三个字符串之一 |

## 常用命令

#### 操作命令

```shell
# 进入目录
cd $(ls -t | grep -e "(termloan|consumerfinance)[a-zA-Z0-9-_]*" | head -1)
# 显示/etc/passwd中的账户[-F指定域分隔符为':',默认域分隔符是"空白键" 或 "[tab]键"]
cat /etc/passwd | awk -F ':' '{print $1}'
```



### 用户相关命令

```bash
#添加用户
sudo adduser 用户名
#修改用户登录名
sudo usermod -l 新用户名 旧用户名
#修改用户密码
sudo passwd 用户名
#删除用户
sudo userdel -r 用户名 
#锁定用户，用户无法登录
sudo usermod -L 用户名
#解除锁定
sudo usermod -U 用户名
#查看系统中所有用户
cat /etc/passwd
```

### 组相关命令

```bash
#新建组
sudo groupadd 组名
#修改用户所属组
sudo usermod -g 组名 用户名
#修改组名
sudo groupmod -n 新组名 旧组名
#删除组
sudo groupdel 组名
#查看系统中所有组
cat /etc/group
```

### 其他命令

```bash
#修改文件权限[-R表示对当前目录下的所有文件与子目录递归进行相同的所有者变更]
chmod -R 777 文件夹或文件名
```



## Windows登录Linux服务器

win+R-->输入cmd-->在cmd窗口输入ssh 用户名@IP