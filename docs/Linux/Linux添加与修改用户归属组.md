# Linux 添加与修改用户归属组
参考资源：https://cnzhx.net/blog/linux-add-user-to-group/

## 已存在的用户
1. 以root进行登录
2. 打开终端
3. 修改分组
````
usermod -a -G root huang  
````
说明：这里的usermod指修改，-a就相当于append，root指用户组，huang指存在的用户 -G 标识组的意思

4. 将用户分配给主要用户组
````
usermod -g root huang   
````
说明：-g 用户初始化为指定为登录组

5. 在某个组删除用户
````
gpasswd -d huang root  
````
注意：这个时候需要保证 group 不是 user 的主组。

## 没有存在的用户
1. 新建用户并归属组
````
useradd -G root huang
````

2. 分配密码
````
passwd huang123
````

3. 查看是否成功
````
id huang
````