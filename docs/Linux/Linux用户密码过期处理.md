# Linux用户密码过期处理


重置用户密码过期标志，密码可更改的最小天数设置为0，密码保持有效的最大天数设置为99999，停滞时期设置为-1即never，帐号到期的日期设置为-1即never：

````
chage -m 0 -M 99999 -I -1 -E -1 user
````

再次查看该用户的密码有效期：

````
[dolphinscheduler@node01 ~]$ chage -l dolphinscheduler
Last password change                                    : Sep 17, 2021
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
````

# 附 chage命令详解
## chage
- 修改帐号和密码的有效期限

## 语法

````
chage [选项] 用户名
````

## 选项

````
-m：密码可更改的最小天数。为零时代表任何时候都可以更改密码。
-M：密码保持有效的最大天数。
-w：用户密码到期前，提前收到警告信息的天数。
-E：帐号到期的日期。过了这天，此帐号将不可用。
-d：上一次更改的日期。
-i：停滞时期。如果一个密码已过期这些天，那么此帐号将不可用。
-l：例出当前的设置。由非特权用户来确定他们的密码或帐号何时过期
````