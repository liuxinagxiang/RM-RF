# RM-RF
How to delete data and run ?
---
生产环境绝对禁止使用RM -RF！== NEVER！==

避免误删文件的13条建议：
1.操作前备份
  修改或删除数据前请务必备份，最好有异机备份，修改配置等先提交版本管理系统再发布到线上。
2.使用mv替代
  删除应使用mv命令替代rm命令，无用的文件不要着急删除，而是移动到回收站/tmp里观察一段时间。再写个定时shell定期清理，以模拟“回收站”功能
3.配置alias别名
  可以通过设置别名等手段屏蔽rm，这样一旦直接用到rm命令就是意识到。把rm配置成rm -i 或者 mv 之类的命令
4.让删除变得复杂（即精准删除）
  如果非要删除数据，还可用find结合rm替代单纯的rm，包括设定定时任务等动作执行清理。
5.必须用rm？
  如果非要使用rm删除数据，请尽量先切换目录到待删除数据所在的目录。
6.能不用通配符就不用通配符 例如：
```
[root@wellcom.cn /]# cd /wellcom.cn/
[root@wellcom.cn /]# rm -f test1 test2
```
7.必须用rm和通配符？
  如果非要使用rm删除并且要采用通配符，请按下面方法：
  ```
[root@wellcom.cn /]# cd /wellcom.cn/
[root@wellcom.cn wellcom.cn]# rm -fr *  #目标中最好不要带有“/”，因为“/”太危险， 原因请看第8条
```
8.通配符与rm -rf的结合是极其危险的
```
rm -rf /wellcom.cn/*
```
8.rm命令中，目标路径中的任意斜线前后如果多了空格可能会带来灾难
```
[root@wellcom.cn /]# rm -fr /wellcom.cn/*
#例如:rm -fr /wellcom.cn/空格*   # *前不小心多了空格，会删除当前目录下的所有内容。
[root@wellcom.cn /]# rm -fr /wellcom.cn/空格*  #会把当前目录根下全删了。
#更甚者， 如果在wellcom.cn多了一个空格， 那就大悲剧啦，根目录都删除了...
[root@wellcom.cn /]# rm -fr /空格wellcom.cn/*  #会把根目录全删了，所有文件，所有文件，所有文件!
```
9.习惯-tab补全
  如果必须要rm -rf /wellcom.cn/*   命令删除，最后的避免错误方法就是要用tab键去补全，不要手敲任何字符，防止误删。
10.不要高射炮打蚊子
  如果删除的不是目录，就不要用rm -fr，采用最小化的方法rm -f即可，甚至重要的少量文件，可以不用-f，以获得确认删除提示信息。
11.使用&&代替cd...rm
   我们常用命令:
```
cd ${log_path}
rm -rf *
```
在shell脚本中我们常用上述命令，合并成一个语句
```
cd ${log_path} && rm -rf *
```
当前半句执行失败的时候，后半句不再执行。更安全
12. rsync --delete 
```
慎用:rsync --delete 
```
13.重要文件加权限禁止修改
chattr [-RVf] [-+=aAcCdDeijsStTu] [-v version] files...
```
-R 递归处理，将指定目录下的所有文件及子目录一并处理。
-v<版本编号> 设置文件或目录版本。
-V 显示指令执行过程。
+<属性> 开启文件或目录的该项属性。
-<属性> 关闭文件或目录的该项属性。
=<属性> 指定文件或目录的该项属性
A：即Atime，告诉系统不要修改对这个文件的最后访问时间。
S：即Sync，一旦应用程序对这个文件执行了写操作，使系统立刻把修改的结果写到磁盘。
a：即Append Only，系统只允许在这个文件之后追加数据，不允许任何进程覆盖或截断这个文件。如果目录具有这个属性，系统将只允许在这个目录下建立和修改文件，而不允许删除任何文件。
b：不更新文件或目录的最后存取时间。
c：将文件或目录压缩后存放。
d：当dump程序执行时，该文件或目录不会被dump备份。
D:检查压缩文件中的错误。
i：即Immutable，系统不允许对这个文件进行任何的修改。如果目录具有这个属性，那么任何的进程只能修改目录之下的文件，不允许建立和删除文件。
s：彻底删除文件，不可恢复，因为是从磁盘上删除，然后用0填充文件所在区域。
u：当一个应用程序请求删除这个文件，系统会保留其数据块以便以后能够恢复删除这个文件，用来防止意外删除文件或目录。
t:文件系统支持尾部合并（tail-merging）。
X：可以直接访问压缩文件的内容。
#常用命令展示：
chattr +a  /etc/passwd #只能给文件添加内容,删除不了
chattr +i  /etc/passwd #文件不能删除，不能更改，不能移动
lsattr /etc/passwd     #文件加了一个参数 i 表示锁定
chattr -i /etc/passwd  #解锁
```
隐藏chattr命令：
```
which chattr
mv /usr/bin/chattr /opt/wellcom_ci/
cd /opt/wellcom_ci/
mv chattr ci                        #更改命令,利用别名隐藏身份
/opt/wellcom_ci/ci +i /etc/passwd   #利用c 行驶chattr命令
lsattr /etc/passwd                  #查看加密信息
#恢复隐藏命令：
mv ci /usr/bin/chattr
chattr -i /etc/passwd
lsattr /etc/passwd
```

##生产环境使用mv命令替代rm命令：
  任何时候都不要使用rm -rf，并做好数据备份，建议修改rm指令如下：
  可以把要删除的mv到~/.trash目录相当于进回收站，然后可以定期删除
  * 找个地方作为垃圾回收站（~/.trash)
  * rm改为mv，实现rm时把文件直接mv到这个地方
  * 提供清空回收站/查看回收站文件/找回误删文件等的操作
  * shell脚本：Bash_Trash.note

##如果很不幸手贱了，只能求助数据恢复工具-例：extundelete，ext3grep
恢复ext3,ext4文件系统-extundelete：
1.安装依赖e2fsprogs和e2fsprogs-libs库
```
$ sudo yum install e2fsprogs*
#在Ubuntu中安装ex2fslibs和e2fsprogs
$ sudo apt-get install e2fsprogs*
```
2.下载安装extundelete：yum install extundelete 
```
http://extundelete.sourceforge.net/
$ tar -xjf extundelete-0.2.4.tar.bz2
$ cd extunnelete-0.2.4
$ chmod u+x configure
$ ./configure
$ make       #执行文件将存储在extundelete-0.2.4文件夹中的src文件夹中,无需在系统中安装Extundelete即可运行它。如果发现有必要安装它，请运行命令安装
$ sudo make install
```
3.恢复数据
停止要从中恢复数据的磁盘上正在进行任何写操作的所有进程，然后卸下磁盘。你也可以将磁盘挂载为只读
```
$ umount /dev/sda6
```
要将磁盘卸载并重新装载为只读：
```
$ mount -o remount,ro /dev/sda6 #只读安装或卸载都可以
```
如果是系统根分区数据遭删除，就需要将系统进入单用户模式，然后将根分区以只读模式挂载
识别分区：
```
$ lsblk
```
运行Extundelete根据lsblk的输出，你可以看到分区的名称
```
$ src/extundelete  /dev/sda6 --restore-file home/filename 
```
注意：文件名是相对于分区的，而不是绝对路径。这就是为什么它们不以'/'开头的原因
如果你不知道文件的名称是什么，但是可以调出文件存储的目录，请运行以下命令。这将列出该目录中的文件，并指示是否删除了该文件：
```
$ src/extundelete /dev/sda6 --restore-file home/*
```
恢复分区所有数据：
```
$ src/extundelete /dev/sda6 --restore-all
$ src/extundelete /dev/sda6 --restore-all --log 0 #不显示log信息
$ src/extundelete /dev/sda6 --restore-all --log logdata.txt #输出到日志文件：
```

##模拟实战-extundelete：
模拟数据误删除
```
$ mkdir /data
$ mkfs.ext3 /dev/sdc1
$ mount /dev/sdc1 /data
$ cp /etc/passwd /data     
$ cp -r /app/nginx /data
$ mkdir /data/test
$ echo 'extundelete test \!' > /data/test/test.txt
$ cd /data
$ md5sum passwd
$ md5sum test/test.txt
$ rm -rf /data/*
```
卸载磁盘分区:数据误删后立即卸载磁盘分区
```
$ cd /mnt
$ umount /data
```
查询可恢复的数据信息
```
$ extundelete /dev/sdc1 --inode 2
标记为deleted为已删除的的文件或目录
```
恢复单个文件
```
$ extundelete /dev/sdc1 --restore-file passwd #注意：文件名是相对于原文件的存储路径，而不是绝对路径
$ cd RECOVERED_FILES/ && ls                 #查看恢复的文件,RECOVERED_FILES目录必须可写
$ md5sum passwd                             #对比两次的MD5,相同说明文件恢复成功
```
恢复单个目录
```
$ extundelete /dev/sdc1 --restore-directory /nginx
```
恢复所有误删除数据
```
$ extundelete /dev/sdc1 --restore-all
```
恢复某个时间段数据
```
$ extundelete --after date+%s --restore-all /dev/sdc1 #date+%s当前时间总秒数，起算时间为1970-01-01 00:00:00
```

##恢复ext3文件系统-ext3grep：
```
yum -y install e2fsprogs*
wget https://src.fedoraproject.org/lookaside/extras/ext3grep/ext3grep-0.10.1.tar.gz
tar zxvf ext3grep-0.10.1.tar.gz
./configure
make 
make install
ext3grep -v
```
模拟实战-ext3grep：
1.模拟数据误删除环境
```
/> mkdir /disk   #建立一个挂载点
/> cd /mydata
mydata> dd if=/dev/zero of=/mydata/disk1 count=102400 #模拟磁盘分区创建一个空设备
mydata> mkfs.ext3 /mydata/disk1                       #将空设备格式化为ext3格式
mydata> mount -o loop /mydata/disk1 /disk             #挂载设备到/disk目录
mydata> cd /disk/
disk> cp /etc/profile /disk                           #复制文件到模拟磁盘分区
disk> echo 'ext3grep test!' > ext3grep.txt
disk> mkdir /disk/ext3grep
disk> cp /etc/hosts /disk/ext3grep
disk> md5sum profile #获取文件校验码
disk> md5sum ext3grep.txt
disk> rm -rf /disk/* #模拟误删除操作
disk> ls
```
2.卸载磁盘分区
```
disk> cd /opt         #切换/opt目录下
opt> umount /disk     #卸载模拟磁盘分区
```
3.查询恢复数据信息
```
opt> ext3grep /mydata/disk1 --ls --inode 2
opt> ext3grep /mydata/disk1 --restore-inode inode_id #恢复所有--restore-all
opt> cd RESTORED_FILES
RESTORED_FILES>ls
```
4.误删除的mysql表.note

参考：Linux文件恢复利器-ext3grep与extundelete  
      https://my.oschina.net/goopand/blog/3003127
