---
layout: post
title: Linux命令-1
date: 2016-11-05 08:32:24.000000000 +09:00
categories:
- 技术
tags:
- JavaScript
toc: true
---
**
摘要：Linux命令
**
<!-- more -->
<The rest of contents | 余下全文>
---
### 1. 查看目录下的文件信息
```
    > ls            //查看当前目录下的文件信息(list)
    > ls  -l 或 -ll //把文件信息以详细列表形式展示出来
    > ls  -a        //查看全部文件(包括隐藏文件)
    > ls  -al       //以详细列表形式查看全部文件
    > ls –F         //在列出的文件 目录名称后加一符号例如可执行文件加"*", 目录则加 "/"
```

### 2. 查看当前所处位置
```
    > pwd      //打印工作目录(print working directory)
```

### 3. 目录切换
```
    > cd ..     //切换到上级目录
    > cd  目录  //切换到当前目录下的某个"目录"里边
    > cd        //切换到当前用户的“家目录”里边
    > cd ~      //切换到当前用户的“家目录”里边
```

### 4. 查看自己是谁
```
    > whoami       //查看正在操作的用户
    > who am i     //查看到当前登录系统的用户信息
```
### 5. 账号切换
```
    su  账号名称
    > su - root  //切换到root管理员账号(权限也是root)
    > su -       //切换到root管理员账号(权限也是root)

    > su  root   //切换到root管理员账号(普通用户权限)
    > su  普通账号名称
    > exit       //退出为原先的账号
    注意：多次连续使用su指令会使得账号出现累加效果
          如果需要退出当前账号，回到原先账号就使用exit指令
    wangxiong---->root---->wangxiong---->root
```
### 6. su (需要是切换到root管理员账号（权限为root）su - root 或者 su -)
```
    ># init 3         //切换到命令窗口
    ># init 5        //切换为桌面窗口
    >$               //普通用户（没有管理员权限）
    >#               //超级用户
    >Ctrl+Alt+F1至F7 //文本和图形界面的切换
```
### 7. 目录操作
```
    ① 创建目录 make directory
    > mkdir  dirname                    //创建一个目录
    > mkdir -p dirname/dirname/dirname [-p] //递归方式创建多级目录
    > mkdir  dir/dir/newdir
    > mkdir  dir/newdir/newdir  -p
    创建目录，新目录的个数多于2个，就需要加-p参数，进行递归创建

    ② 改名字
    > mv diroldname  dirnewname     //同目录下改名字
    > mv dir/oldname  dir/newname  // 子目录下改名字

    ③ 移动操作move
    > mv  dir1   dir2   // dir1移动到dir2目录下边
    mv既可以改名字、也可以移动操作
    改名字：第二个参数是“不存在”的目录名字，就是改名字
    移动操作：第二个参数是“存在”的目录名字，就是移动
    该mv内部有两个实现：移动、改名字

    ④ 复制操作copy
    > cp  filename  dirname   //filename文件被复制到dirname目录里边
    > cp  filename  dirname/newname   //复制文件到目标地址后起一个新名字
    > cp -r dir  dirname      //recursive复制目录需要设置-r参数
    > cp -r dir1/dir2/  dir3   //dir2被复制到dir3目录下边
    > cp -r  dir1/dir2  newdir  //dir2被复制到当前目录并给改一个新名字

    ⑤ 删除操作
    > rm  filename    //删除单个文件,需要确认
    > rm -f filename  //删除单个文件，不需要确认
    > rmdir  dirname  //删除单个空目录
    > rm -r  dirname  //recursive递归删除非空目录
    > rm -rf filename //recursive and force 递归强制删除任何文件
    > rm -rf /        //root执行  kill you  by yourself
```

### 8. 文件操作
```
    ① 文件查看
    > cat filename      //文件内容一次性输出到页面上
    > head -n filename  //查看文件的前n行内容
    > tail -n filename  //查看文件的后n行内容
    > wc filename       //查看文件内容行数
    > more  filename   //敲回车方式“逐行”查看文件内容
                         不支持回看，q键退出
    > less  filename   //通过“上下左右”键查看文档的各个部分内容
                        q键退出查看
    ② 文件创建
    > touch  filename  //创建文件

    ③ 给文件追加内容
    > echo 内容  >  filename  //把内容以"覆盖"方式存入filename文件
                               如果对应的文件不存在会自动创建
    > echo 内容 >>  filename  //把内容以"追加"方式存入filename文件
                               如果对应的文件不存在会自动创建
    > cat file1 > / >>  file2 //把file1内容给存入file2里边
```

### 9. 用户操作user，
```
    用户信息存放在配置文件/etc/passwd
    ① 创建用户useradd
    > useradd  用户名           //给系统增加一个普通用户
                                同时要创建一个同名的组
    > useradd -g gid组编号  -d 家目录  -u 用户id  username
    (自定义的家目录会被系统自动创建出来)

    ② 修改用户usermod
    > usermod -g gid组编号  -d  家目录  -u 用户id编号  -l 新名字  username
    (修改的家目录不会自动创建，需要手动创建)

    ③ 删除用户userdel
    > userdel username      //在/etc/passwd里边删除用户信息（家目录给保留）
    > userdel username -r   //在/etc/passwd里边删除用户信息同时也会删除该用户家目录
```

### 10. 组别操作group
```
    用户信息存放在配置文件/etc/group
    ① 创建组groupadd
    > groupadd groupname

    ② 修改组groupmod
    > groupmod -g gid组编号  -n 新名字  groupname

    ③ 删除组groupdel
    > groupdel  groupname
    注意:组别里边有关联的用户则禁止删除
```
### 11. 查看指令可以设置何种参数
```
    > man 指令
```

### 12. 给文件设置权限
```
    ① 字母相对方式设置权限
    chmod  u+/-rwx[,g+/-rwx][,o+/-rwx]  filename
    > chmod  u+rw   filename    //给文件主人增加读、写的权限
    > chmod  u+wx,g-rx,o+rwx  filename  //主人增加写、执行权限，
                                          同组用户取消读、执行权限，
                                          其他组用户增加读、写、执行权限
    > chmod  u+r,u-x filename       //给主人增加“读”权限、取消“执行”权限
    > chmod u=rwx,g=rw,o=r filename //赋予给定权限,并取消其他所有权限

    ② 数字绝对方式设置权限
    读：  4
    写:   2
    执行: 1

    6-------> 读、写
    1-------> 执行权限
    3-------> 写、执行
    2-------> 写
    4-------> 读
    5-------> 读、执行
    7-------> 读、写、执行
    0-------> 空权限

    chmod  ABC  filename     //ABC三个数字分别代表主人、同组、其他组用户权限信息
    > chmod 751  filename    //主人：读、写、执行
                               同组用户：读、执行
                               其他组用户：执行
    问：字母和数字如何选取使用？
    答：权限改动“较少”使用字母方式
        权限改动“较多”使用数字方式
```

### 13. 在指定文件里边搜索指定的内容
```
    grep  被搜寻内容   文件
    > grep  login   passwd    //在passwd文件里边搜寻login字样
                                会把有login字样的行的内容都打印出来
```

### 14. 查看文件占据磁盘空间大小
```
    磁盘有block块(小格子)，磁盘内部最小的空间是一个block块，大小是4k(可以修改)

    > du -h  目标文件
    问：1000k的磁盘可以存放1k的图片，存放多少张。
    答：block为4k情况，可以存放250张，利用率为25%.
```
### 15. 管道的使用
```
    > ls -l | grep passwd           //在列出的文件信息里边获得passwd内容
    > grep sbin passwd | grep abc   //在passwd文件里边找到符合既有abc，还有sbin字样的行内容
    > grep sbin passwd | grep abc | wc  //多个管道同时使用
                                    //在passwd文件里边找到符合既有abc，还有sbin字样的行数目

    > ls -l | head -15 | tail -5    //查看当前目录下第11-15个文件的内容
```
### 16. 文件查找指令
```
    find  目录   选项  选项值
    > find  ./  -name  passwd      //在当前目录下边查找一个文件的名字为passwd
    > find  /  -name  passwd       //遍历系统的全部目录，搜寻名字为passwd的文件(效率低)

    > find  /  -mindepth 3  -maxdepth 4  -name passwd   //查找文件处于目录的层次在3-4之间

    选项：
        -name           通过文件名字查找文件
        -name  "pass*"  通过模糊查找方式搜寻以pass开始的文件

        -maxdepth  n    限制查找目录的最深层次
        -mindepth  n    限制查找目录的最浅层次

        -size           通过文件大小进行查找
                        默认单位：512字节
                        100c  单位：字节
                        2k    单位：千字节

                        -100c  条件是文件大小小于100字节
                        +100c  条件是文件大小大于100字节
    > find ./  -size 52c     //在当前目录查找文件大小为52字节的文件
    > find ./  -size  +3k    //在当前目录查找文件大小大于3k
    > find ./  -size  -150c  //在当前目录查找文件大小小于150字节
    > find ./  -size 3       //在当前目录查找文件大小为1536字节
```

### 17.关闭系统
```
    shutdown [选项] [时间] [警告信息] //需要特别说明的是该命令只能由超级用户使用
    - k 并不真正关机而只是发出警告信息给所有用户
    - r 关机后立即重新启动 //reboot
    - h 关机后不重新启动 //halt
    - f 快速关机重启动时跳过fsck
    - n 快速关机不经过init 程序
    - c 取消一个已经运行的shutdown
    > # shutdown -r +10 //系统在十分钟后关机并且马上重新启动
    > # shutdown –h now //系统马上关机并且不重新启动
```
### 18. 更改属主和属组
```
    chown [用户:组] 文件
    > chown wangxiong:linux filename //更改文件的属主和属组
```
### 19.进程查看命令
```
    ps [选项]
    -e 显示所有进程
    -f 全格式
    -l 长格式
    > ps -ef |grep wangxiong   //查询该用户的所有进程
```

### 20.磁盘及文件系统管理命令
```
    df 目前磁盘剩余的磁盘空间
    > df -k  //显示各分区的磁盘空间使用情况
```
### 21.软件安装命令
```
    tar [选项] 文件名
    -c 创建一个新的档案文件 create
    -t 查看档案文件的内容  tarfile
    -x 分解档案文件的内容
    -f 指定档案文件的名称
    -v 显示过程信息
    -z 采用压缩方式
    linux压缩：
    tar czf a.tar.gz a
    linux解压：
    tar xzf a.tar.gz
```

### 22.查看cpu型号
```
uname -a
uname -m //i686
```
### 23.优化选项编译程序
```
make CFLAGS="-march-i686"//编译成优化的二进制编码
```
