# Linux设置无密码sudo

- [root@localhost ~]# visudo
- 在`root  ALL=(ALL)   ALL`这一行下面，再加入一行：
- 将原来的：`admin   ALL=(ALL)   ALL` 修改为 ：`admin   ALL=(ALL)   NOPASSWD:ALL`