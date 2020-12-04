### Apache Dolphin 如何调用 Windows 远程服务器上的命令



#### 前期准备

1. Apache Dolphin 环境搭建

Linux 宿主机，python3.7 环境（第三方包 pywinrm2==0.0.0），若涉及离线安装第三方 python 程序包，参考 https://www.cnblogs.com/staff/p/10538972.html

2. 开启 Windows 服务器 winrm 服务

https://blog.csdn.net/weixin_33816946/article/details/92569127

3. SapDataservice 环境，及 JOB bat 脚本获取

https://blogs.sap.com/2012/08/22/sap-bods-running-scheduling-bods-jobs-from-linux-command-line-using-third-party-scheduler/

#### 程序及配置

```python
import winrm
def cmd_views(ip,cmd_comand):
  win = winrm.Session('http://'+ip+':5985/wsman', auth=('Administrator', 'SuN!!!1699'))#参数为用户名和密码
  r = win.run_cmd(cmd_comand) # 执行cmd命令
  return r.std_out # 打印获取到的信息
   
ip="192.168.0.178"
cmd_comand=r'C:\PROGRA~2\SAPBUS~1\DATASE~1/bin/AL_RWJ~1.EXE  "C:\ProgramData\SAP BusinessObjects\Data Services/log/SUNGROWJOB/" -w "inet:SUNGROWDS:3500"  -C "C:\ProgramData\SAP BusinessObjects\Data Services/log/JOB_SHELL.txt"'#运行命令
 
a=cmd_views(ip, cmd_comand)
print(cmd_comand)
```

