---
  layout: post
  title: 一些常用的bash命令
  categories: Linux
  tags:
---

#### ssh

    ssh root@ip

1.http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html
2.https://www.bennythink.com/ssh-1.html
3.http://skx926.com/2017/11/30/understanding-of-ssh/

#### scp

    #复制远程机器的文件到本地
    scp root@10.10.10.10:/opt/soft/nginx-0.5.38.tar.gz /opt/soft/

    将本地机器发送到远程机器
    scp timeline_01.json root@172.18.138.143:/Users/sw/Downloads

    scp -r 选项表示复制目录

参考：

http://man.linuxde.net/scp

#### rsync

--bwlimit 限速


#### ifconfig


#### lsof -i


#### sed

将逗号替换为制表符

    sed $'s/,/\t/g' a.txt

前面需要加$' ',否则\t不会被解析

打印第10到20行

    sed -n '10,20p' a.txt


#### find

#### grep

#### xargs

#### kill

kill -0 $pid 查看进程是否还在运行

参考： https://stackoverflow.com/questions/11012527/what-does-kill-0-pid-in-a-shell-script-do

#### ll

In the order of output;

    -rwxrw-r--    1    root   root 2048    Jan 13 07:11 afile.exe

file permissions,

number of links,

owner name,

owner group,

file size,

time of last modification, and

file/directory name

File permissions is displayed as following;

first character is - or l or d, d indicates a directory, a line represents a file, l is a symlink (or soft link) - special type of file

three sets of characters, three times, indicating permissions for owner, group and other:

    r = readable
    w = writable
    x = executable

参考： https://unix.stackexchange.com/questions/103114/what-do-the-fields-in-ls-al-output-mean

#### bash快捷键

    ^a^b
    #将上一个命令第一个出现的a替换为b

    !!:gs/a/b
    #将上一个命令中所有的a全部替换为b  
