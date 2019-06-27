---
  layout: post
  title: rsync 错误
  categories: 开发
  tags:笔记
---

今天在测试流程的时候发现rsync在拉取数据时遇到如下错误:

    protocol version mismatch -- is your shell clean?
    (see the rsync man page for an explanation)
    rsync error: protocol incompatibility (code 2) at compat.c(171) [receiver=3.0.6]

然后在网上搜了一下，原来在rsync 的man page 有如下描述:

> rsync  occasionally  produces error messages that may seem a little cryptic. The one that seems to cause the most confusion is "protocol version mismatch       -- is your shell clean?".

>       This message is usually caused by your startup scripts or remote shell facility producing unwanted garbage on the stream that  rsync  is  using  for  its transport. The way to diagnose this problem is to run your remote shell like this:

>              ssh remotehost /bin/true > out.dat

>       then  look  at  out.dat. If everything is working correctly then out.dat should be a zero length file. If you are getting the above error from rsync then you will probably find that out.dat contains some text or data. Look at the contents and try to work out what is producing it. The most common  cause  is incorrectly configured shell startup scripts (such as .cshrc or .profile) that contain output statements for non-interactive logins. If  you are having trouble debugging filter patterns, then try specifying the -vv option.  At this level of verbosity rsync will show why each individual file is included or excluded.


按照这里的提示，执行`ssh remotehost /bin/true > out.dat`,out.dat果然有几个字符。根据提示，在远程机器的.bashrc中找到了输出这几个字符的代码。原来刚才为了调试另外的脚本我在.bashrc中加了几行输出代码，然后再同步数据的时候就遇到上面的错误了。看来.bashrc这些文件不能随便加输出。
