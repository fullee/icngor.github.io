

## systemd系列大纲







目录：

- 前奏
  - Linux登录前显示IP地址，登录后设置公告
- 建立全局观
- 深入要害
- 实战案例
  - springboot项目开机启动
  - 内网穿透软件frp开机启动
- 开机自动挂在磁盘分区
  - 定时器
- 总结
- 附录
- 番外1 系统日志工具journalctl











```
[root@localhost compose]# cat /usr/bin/issue.sh
#!/bin/sh

ipaddr=`ip addr | grep "dynamic" | awk '{print $2}' |awk -F/ '{print $1}'`
cp /etc/issue.bak /etc/issue
echo IP: $ipaddr >> /etc/issue
echo 系统访问地址: http://$ipaddr >> /etc/issue
echo "" >> /etc/issue



[root@localhost compose]# cat /usr/lib/systemd/system/issue.service
[Unit]
Description=Issue Banner
After=network-online.target

[Service]
Type=oneshot
User=nobody
ExecStart=/usr/bin/issue.sh

[Install]
WantedBy=multi-user.target
```






## 简介 & 特点




Unit类型：

          service unit：扩展名为 .service 不需要执行权限，只是配置文件，用于定义系统服务
    
          target unit   : 扩展名为  . target 用于模拟实现"运行级别"
    
          Divice unit   ：  .divice  用于定义内核识别的设备 
          Mount unit    ：  .mount
    
                 定义文件系统挂载点
    
          Socket unit: .socket
    
            用于标识进程间通信用的socket文件，也可在系统启动时，延迟启动服务，实现按需启动
    
          Snapshot unit: .snapshot,
    
                管理系统快照
    
          Swap unit: .swap,
    
                用于标识swap设备
    
          Automount unit: .automount，
    
                文件系统的自动挂载点
    
         Automount unit: .automount，
    
                文件系统的自动挂载点
    
       Path unit: .path，
    
             用于定义文件系统中的一个文件或目录使用,常用于当文件系统变化时，延迟激活服务，如： spool



基于socket的激活机制： socket与服务程序分离

      #为每个服务预先创建激活socket，systemd监听对应socket，当需要使用时启动服务
    
        基于d-bus的激活机制：
    
        基于device的激活机制：
    
        基于path的激活机制：
    
        系统快照：保存各unit的当前状态信息于持久存储设备中 
    
        向后兼容sysv init脚本（在cenos7上不建议init级别和init切换运行级别）


​        
不兼容：

        systemctl命令固定不变，不可扩展
    
        非由systemd启动的服务， systemctl无法与之通信和控制

   系统服务不会读取标准输入流，系统服务启动不会读取任何用户环境变量，服务中需要使用绝对路径超过5分钟，就会强制退出


服务管理命令：
tab键自动补全
```shell
root@localhost# yum install -y bash-completion

root@localhost# systemctl
[root@localhost system]# systemctl list-dependencies network.service

systemctl kill 进程名   
```

挖坑：
dnf包命令