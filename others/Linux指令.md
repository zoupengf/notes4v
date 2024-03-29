# Linux指令


### 远程文件传输
```
 sshpass -p xxxpassws scp /home/scp/hi ubuntu@qw607.com:
```

### 递归删除
```
rm -rf /home/test
```

### 查看端口
```
lsof -i tcp:81
```

## 通过进程名查看对应进程
* 先用pgrep [str] 命令进行模糊匹配，找到匹配该特征串的进程ID；

* 其次根据进程ID显示指定的进程信息，ps --pid [pid]；

* 因为查找出来的进程ID需要被作为参数传递给ps命令，故使用xargs命令，通过管道符号连接；

* 最后显示进程详细信息，需要加上-u参数。
```
pgrep java | xargs ps -u --pid
```


### 查看进程信息
* ll是ls -l简写
```
ll /proc/pid
```

### 开启端口
```
firewall-cmd --zone=public --add-port=81/tcp --permanent;firewall-cmd --reload;
```


## 查看内存使用前10
```
ps auxw|head -1;ps auxw|sort -rn -k4|head -10 
```

### CPU占用最多的前10个进程： 
```
ps auxw|head -1;ps auxw|sort -rn -k3|head -10 
```
