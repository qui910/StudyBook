# 1 概述

在Ubuntu中要执行定时任务，可以通过linux自带的crontab工具进行。在Crontab中调用普通的shell脚本一样，需要注意的就是shell脚本中的环境变量在crontab中失效，必须在脚本中添加如下类型语句:

```shell
source ~/.bashrc
```



# 2 安装Cron

* 检查，主机上是否安装crontab：

```shell
crontab -l
```

* 如检查无命令，则通过`apt-get`进行安装：

```shell
sudo apt-get install cron
```



#  3 配置cron启动

1. 打开ubuntu的cron日志

```shell
vim /etc/rsyslog.d/50-default.conf
# 打开文件，在文件中找到cron.*，把前面的#去掉，保存退出，输入

sudo service rsyslog restart
```

2. 安装postfix邮箱

```shell
sudo apt-get install postfix
# 注：crontab执行脚本时是不会直接错误的信息输出，而是会以邮件的形式发送到你的邮箱里，这时候就需要邮件服务器了
# 此步可以跳过
```

3. 运行日志检查

```shell
tail -f /var/log/cron.log
# 检查cron的运行日志
```

检查结果类似：

```shell
Jun 18 10:35:01 master CRON[10442]: (CRON) info (No MTA installed, discarding output)
Jun 18 10:40:01 master CRON[10788]: (pangrd) CMD (/usr/bin/curl -o /home/test/logs/job_1.log http://192.168.31.185:8093/dmp/doImport?jobId=1\&uuid=`date +)
Jun 18 10:40:01 master CRON[10786]: (CRON) info (No MTA installed, discarding output)
Jun 18 10:40:10 master crontab[10792]: (pangrd) BEGIN EDIT (pangrd)
Jun 18 10:40:52 master crontab[10792]: (pangrd) REPLACE (pangrd)
Jun 18 10:40:52 master crontab[10792]: (pangrd) END EDIT (pangrd)
Jun 18 10:40:59 master crontab[10852]: (pangrd) LIST (pangrd)
Jun 18 10:41:01 master cron[10259]: (pangrd) RELOAD (crontabs/pangrd)
Jun 18 10:41:14 master cron[10871]: (CRON) INFO (pidfile fd = 3)
Jun 18 10:41:14 master cron[10871]: (CRON) INFO (Skipping @reboot jobs -- not system startup)
Jun 18 10:45:01 master CRON[11013]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Jun 18 10:45:01 master CRON[11014]: (pangrd) CMD (/home/test/crontab_shell/runPfDev.sh)
Jun 18 10:45:01 master CRON[11007]: (CRON) info (No MTA installed, discarding output)

```

4. 查看报错信息

```shell
cat /var/mail/root
# 配置mail后，错误信息可以从mail中查看
# 因为脚本是使用root账户cron执行的，所以在mail中查看root文件的输出
```

# 4 配置Crontab任务

* 添加crontab任务

```shell
crontab -e
# 打开编辑crontab命令
```

​		编辑后内容类似：

```shell
# 每5分钟执行一次
*/5 * * * * /home/test/crontab_shell/runPfDev.sh
```

​		如果你的主机安装过GUN nano编辑器，那`crontab -e`默认使用的就是nano编辑器，非常不好用，可以修改配置，将系统的默认编辑器改为VIM。

```shell
test@master:~$ select-editor
Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed
  
Choose 1-4 [1]: 2

```

​		定时任务语句格式为：执行周期+命令。

​		周期有5个域，分别是分钟，小时，日(day of month)，月（month of year），周几（day of week）。每个域不加限制任意的话用*。



*  查看定时任务清单

```shell
crontab -l
```



# 5 Cron服务操作

* 检查是否已经开启 cron

```shell
sudo service cron status
```

* 重启服务 cron

```shell
sudo service cron restart
```

* 关闭服务

```shell
service cron stop
```

* 重新载入配置

```shell
service cron reload
```

* 启动服务

```shell
service cron start
```

也可以使用如下命令操作：

```shell
sudo /etc/init.d/cron start
sudo /etc/init.d/cron stop
sudo /etc/init.d/cron restart
sudo /etc/init.d/cron status
```



# 6 常用周期配置格式

```shell
# 每五分钟执行 
*/5 * * * *
# 每小时执行    
0 * * * *
# 每天执行        
0 0 * * *
# 每周执行       
0 0 * * 0
# 每月执行        
0 0 1 * *
# 每年执行       
0 0 1 1 *
# 每分钟执行一次  
* * * * * user command
# 每隔2小时执行一次
* */2 * * * user command (/表示频率)
#每天8:30分执行一次
30 8 * * * user command
# 每小时的30和50分各执行一次   
30,50 * * * * user command（,表示并列）
# 每个月的3号到6号的8:30执行一次  
30 8 3-6 * * user command （-表示范围）
# 每个星期一的8:30执行一次  
30 8 * * 1 user command（周的范围为0-7,0和7代表周日）
```

