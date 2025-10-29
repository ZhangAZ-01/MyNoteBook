# Linux

## 常用命令

### `kill`

`kill -9 pid`: 强制退出进程
`kill -15 pid`: 优雅退出进程

### 查看当前端口是否被占用

`netstat -anp| grep 端口号`

### echo

单引号：引号里面的内容会原封不动的显示出来
双引号：里面的特殊符号会被解析，变量也会被替换（\ 符号、空格会被解析）
反引号：用于显示命令执行结果

#### 环境变量

设置环境变量
`export name=value`
查询临时变量
`echo $name`
取消环境变量
`unset $name`
查询所有变量
`env`

### dmesg

显示系统日志信息

-t 用于显示时间戳

### hdfs

`hdfs dfs -ls /...`
`hdfs dfs -get /...`
`hdfs dfs -put local_file /...`

### mv

mv source_file(文件) dest_file(文件)

### 查看本机 ip

`ifconfig | grep "inet " | grep -v 127.0.0.1`

### 压缩和解压缩

`zip -r zip_file.zip dir_name`
`unzip zip_file.zip`

`tar -cvf target_name.tar target_name`
`tar -xvf target_name.tar`

### 查看文件大小

`du -sh file_name`
