<center>VMware安装Debian</center>
================


> 存在以下问题：  
> * vim编辑异常
> * 软件源不合适
> * 挂载vmware共享文件夹


# vim编辑异常

大概有这些异常：  
* 键盘字母不能对应正常显示
* 上下左右移动键不能正确移动光标
* Backspace和Delete不能正确删除

解决办法：  
**重新安装vim**  
````
apt-get install vim
````


# 软件源存在问题

报错：  
````
Package vim has no installation candidate
````

**可能是debian版本有点旧，也可能在国内网络的问题，总之不能找到源。**

解决办法：  
**vim编辑/etc/apt/sources.list，在其中添加：**  
````
deb http://mirrors.aliyun.com/debian/ wheezy main
deb-src http://mirrors.aliyun.com/debian wheezy-updates main
````

然后更新软件列表：  
````
apt-get update
````

然后就可以愉快地玩耍了：  
````
apt-get install vim
````


# 挂载共享文件夹

命令：  
````
sudo mount -t vmhgfs .host:/Share  /mnt/share
````

其中Share是在vmware中设置的主机共享文件夹，/mnt/share是虚拟系统中的对应共享文件夹（自己建的）。


# 附

### vim中常用命令：  

| 命令名  | 命令含义 |
| :------ | :------ |
| dd      | 删除行  |
| j/k/h/l  | 上下左右移动  |
| i        | 插入内容  |
| esc     | 退出编辑  |
| :q!     | 强制退出（不保存changes）  |
| :wq     | 保存并退出  |

### debain常用命令：  

| 命令名  | 命令含义 |
| :------ | :------ |
| reboot  | 重启(root权限)  |
| shutdown -h now   | 关机(root权限)  |
| alt + f2    | 启动run工具  |

### bash命令：  

| 命令名  | 命令含义 |
| :------ | :------ |
| pwd     |  列出当前目录  |
| cd ~    |  快速进入用户根目录  |
| cd /    |  快速进入文件系统根目录 |
| ls -la  |  以列表形式列出所有文件详细信息 |

### word快捷键： 

| 命令名  | 命令含义 |
| :------ | :------ | 
| f4      |  重复上一次操作，比格式刷还好用  |
