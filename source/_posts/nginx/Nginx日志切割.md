---
layout: post
title: Nginx日志切割
date: 2016-12-03 19:45:33
tags: Nginx
categories: Nginx
---

## 信号控制

| TERM,INT   | Quick shutdown   | 
| --------   | -----  | 
| QUIT        |  Graceful shutdown  优雅的关闭进程,即等请求结束后再关闭 |   
| HUP         |  Configuration reload ,Start the new worker processes with a new configuration Gracefully shutdown the old worker processes 改变配置文件,平滑的重读配置文件   |   
| USR1        |   Reopen the log files 重读日志,在日志按月/日分割时有用    |  
| USR2        |  Upgrade Executable on the fly 平滑的升级                |
| WINCH       |  Gracefully shutdown the worker processes 优雅关闭旧的进程(配合USR2来进行升级)     |

示例

```shell
#具体语法:
Kill -信号选项 nginx的主进程号
Kill -HUP 4873
 
Kill -信号控制 `cat /xxx/path/log/nginx.pid`
 
Kill -USR1 `cat /xxx/path/log/nginx.pid`
```


## shell+定时任务+nginx信号管理,完成日志按日期存储

分析思路

- 凌晨00:00:01,把昨天的日志重命名,放在相应的目录下
- 再USR1信息号控制nginx重新生成新的日志文件

脚本一

```shell
#!/bin/bash
base_path='/usr/local/nginx/logs'
log_path=$(date -d yesterday +"%Y%m")
day=$(date -d yesterday +"%d")
mkdir -p $base_path/$log_path
mv $base_path/access.log $base_path/$log_path/access_$day.log
# echo $base_path/$log_path/access_$day.log
kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`

```

脚本二

```shell
#!/bin/bash
LOGPATH=/usr/local/nginx/logs/z.com.access.log
BASEPATH=/data/$(date -d yesterday +%Y%m)
mkdir -p $BASEPATH
bak=$BASEPATH/$(date -d yesterday +%d%H%M).zcom.access.log
mv $LOGPATH $bak
touch $LOGPATH
kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`

```

### Crontab 编辑定时任务

示例

```shell
01 00 * * * /xxx/path/b.sh  #每天0时1分(建议在02-04点之间,系统负载小)
```

## 参考链接

- Nginx官网, [Starting, Stopping, and Restarting NGINX](https://www.nginx.com/resources/wiki/start/topics/tutorials/commandline/)