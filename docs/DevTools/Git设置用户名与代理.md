# Git设置用户名与代理

# Git设置用户名

## 全局指定用户

设全局的用户名与邮箱，这样就不需要为每个仓库单独设置了。有两种设置模式，通过命令或者通过文件，二选一即可。

### 通过命令

```shell
git config --global user.name  "xxx"  
git config --global user.email  "xxx@xxx.com"
```

### 通过文件

在用户目录下有一个隐藏文件`.gitconfig`，编辑该文件设置全局用户名与邮箱。

```
[user]
	email = xxx@xxx.com
	name = xxx
```

## 为仓库指定用户

有时我们并不一定每个仓库都使用相同的用户，最简单的例子就是我有的仓库是在GitHub上，有的仓库是在gitee上，这时候全局设置就不再适用，我们可以为每个仓库设置独立的用户名密码。同样有两种设置模式，通过命令或者通过文件，二选一即可。

### 通过命令

在项目目录下执行：

```shell
git config user.name  "xxx"  
git config user.email  "xxx@xxx.com"
```

其实与全局设置类似，就是去掉`--global`。

### 通过文件

打开项目根目录下的`.git/config`文件，这个文件是该项目的Git仓库配置文件。与全局配置相同，添加如下配置：

```
[user]
	email = xxx@xxx.com
	name = xxx
```

# Git设置代理

为Git设置代理时我们可以全局设置代理，也可以为指定地址配置特定的代理，这样解决了不同仓库使用不同代理的情况。

## 全局代理

### 通过命令

```
# socks
git config --global http.proxy 'socks5://127.0.0.1:socks5端口号' 
git config --global https.proxy 'socks5://127.0.0.1:socks5端口号'
# http
git config --global http.proxy 'http://127.0.0.1:http端口号' 
git config --global https.proxy 'https://127.0.0.1:https端口号'
```

### 通过文件

同样通过编辑`~/.gitconfig`，设置

```
[http]
	proxy = socks5://127.0.0.1:socks5端口号
	proxy = http://127.0.0.1:http端口号
 
[https] 
	proxy = socks5://127.0.0.1:socks5端口号
	proxy = https://127.0.0.1:http端口号
```

## 为特定地址设置代理

### 通过命令

```
git config --global http.仓库地址.proxy http://127.0.0.1:代理地址
git config --global https.仓库地址.proxy https://127.0.0.1:代理地址
```

这里以GitHub为例

```
git config --global http.https://github.com.proxy http://127.0.0.1:7890
git config --global https.https://github.com.proxy https://127.0.0.1:7890
```

### 通过文件

同样通过编辑`~/.gitconfig`，设置

```
[http "仓库地址"]
	proxy = socks5://127.0.0.1:socks5端口号
	proxy = http://127.0.0.1:http端口号
 
[https "仓库地址"] 
	proxy = socks5://127.0.0.1:socks5端口号
	proxy = https://127.0.0.1:http端口号
```

以GitHub为例

```
[http "https://github.com"]
	proxy = http://127.0.0.1:7890
[https "https://github.com"]
	proxy = https://127.0.0.1:7890
```

