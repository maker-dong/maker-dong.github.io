# Crontab定时任务


## 安装crontab
````
yum install vixie-cron
yum install crontabs
````

## 开启crontab服务
````
service crond start //启动服务
````
设置开机启动：
查看crontab服务是否已设置为开机启动，运行命令：ntsysv 开机启动管理
加入开机自己主动启动: 
````
chkconfig --level 35 crond on

#设置开机自己主动启动crond服务: 
chkconfig crond on 

#查看各个开机级别的crond服务运行情况 
chkconfig --list crond 
#crond 0:关闭 1:关闭 2:启用 3:启用 4:启用 5:启用 6:关闭 
#能够看到2、3、4、5级别开机会自己主动启动crond服务 

#取消开机自己主动启动crond服务: 
chkconfig crond off
````

## 设置要运行的脚本
两种方式：
- 在命令行输入: crontab -e 然后加入对应的任务，wq存盘退出。 
- 直接编辑/etc/crontab 文件。即vi /etc/crontab，加入对应的任务。
> crontab -e配置是针对某个用户的。而编辑/etc/crontab是针对系统的任务 

### 查看调度任务 
````
crontab -l //列出当前的全部调度任务 
crontab -l -u jp //列出用户jp的全部调度任务 
````

### 删除任务调度工作 
````
crontab -r //删除全部任务调度工作 
````

直接编辑 `vim /etc/crontab` ,默认的文件形式例如以下：
````
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed

````
这个文本解释的已经比较直观了。其中 
- 星号（*）：代表全部可能的值。比如month字段假设是星号。则表示在满足其他字段的制约条件后每月都运行该命令操作。 
- 逗号（,）：能够用逗号隔开的值指定一个列表范围，比如。“1,2,5,7,8,9” 
- 中杠（-）：能够用整数之间的中杠表示一个整数范围，比如“2-6”表示“2,3,4,5,6” 
- 正斜线（/）：能够用正斜线指定时间的间隔频率，比如“0-23/2”表示每两小时运行一次。同一时候正斜线能够和星号一起使用。比如*/10，假设用在minute字段，表示每十分钟运行一次。

这里举几个样例，基本涵盖了常见的一些情况：

实例1
````
5      *       *           *     *     ls         指定每小时的第5分钟运行一次ls命令
30     5       *           *     *     ls         指定每天的 5:30 运行ls命令
30     7       8           *     *     ls         指定每月8号的7：30分运行ls命令
30     5       8           6     *     ls         指定每年的6月8日5：30运行ls命令
30     6       *           *     0     ls         指定每星期日的6:30运行ls命令
30     3     10,20         *     *     ls         每月10号及20号的3：30运行ls命令
25     8-11    *           *     *     ls         每天8-11点的第25分钟运行ls命令
*/15   *       *           *     *     ls         每15分钟运行一次ls命令
30     6     */10          *     *     ls         每一个月中。每隔10天6:30运行一次ls命令
22     4       *           *     *     root     run-parts     /etc/cron.daily
#每天4：22以root身份运行/etc/cron.daily文件夹中的全部可运行文件，run-parts參数表示。运行后面文件夹中的全部可运行文件。
````

实例2
````
#每晚的21:30 重新启动apache
30 21 * * * /usr/local/etc/rc.d/lighttpd restart

#每月1、10、22日的4 : 45重新启动apache
45 4 1,10,22 * * /usr/local/etc/rc.d/lighttpd restart

#每周六、周日的1 : 10重新启动apache
10 1 * * 6,0 /usr/local/etc/rc.d/lighttpd restart

#每天18 : 00至23 : 00之间每隔30分钟重新启动apache
0,30 18-23 * * * /usr/local/etc/rc.d/lighttpd restart

#每星期六的11 : 00 pm重新启动apache
0 23 * * 6 /usr/local/etc/rc.d/lighttpd restart

#晚上11点到早上7点之间，每隔一小时重新启动apache
0 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart

#每一小时重新启动apache
0 */1 * * * /usr/local/etc/rc.d/lighttpd restart

#每月的4号与每周一到周三的11点重新启动apache
0 11 4 * mon-wed /usr/local/etc/rc.d/lighttpd restart

#一月一号的4点重新启动apache
0 4 1 jan * /usr/local/etc/rc.d/lighttpd restart

#每半小时同步一下时间
0/30 * * * * /usr/sbin/ntpdate 210.72.145.44
注意 
* *1 * * * 命令表示是每小时之内的每一分钟都运行。
````


必须指定在每一个小时的第几分钟运行。也就是说第一个*号必须改成一个数值。 
由于*号表示的就是每一分钟。 
另外小时位的/1和没有差别，都是每小时一次。 
假设是设置*/2，实际上是能被2整除的小时数而不是从定时设置開始2小时后运行。比方9点设的到10点就会运行。

## 可能遇到的问题 

root用户下 输入 crontab -l 显示 
````
no crontab for root 
````

这个问题非常easy，相同在 root 用户下输入 `crontab -e `

按 Esc 按`：wq` 回车 ，在此输入 `crontab -l` 就没有问题了 

主要原因是由于这个liunxserver 第一次使用 crontab ，还没有生成对应的文件导致的，运行了 编辑（crontab -e）后 就生成了这个文件
