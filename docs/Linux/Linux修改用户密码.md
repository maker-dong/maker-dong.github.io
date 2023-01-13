
Linux修改密码用 `passwd` 命令，用root用户运行`passwd` ，`passwd user_name`可以设置或修改任何用户的密码，普通用户运行`passwd`只能修改它自己的密码。

````
[root@localhost ~]#  passwd  ##修改root用户密码
Changing password for user root..
New password: ##输入新密码
Retype new password:  ##再次确认新密码
passwd: all authentication tokens updated successfully.
````