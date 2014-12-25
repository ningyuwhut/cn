---
layout: post 
title: 文件上传遇到的诡异错误
categories:
- java
---

今天在写图片上传时遇到一个诡异的错误:

    org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Catalina].StandardHost[localhost].StandardContext[/MMPLSA]]
	    at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:154)
	    at org.apache.catalina.core.StandardContext.reload(StandardContext.java:3954)
	    at org.apache.catalina.loader.WebappLoader.backgroundProcess(WebappLoader.java:426)
	    at org.apache.catalina.core.ContainerBase.backgroundProcess(ContainerBase.java:1345)
	    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.processChildren(ContainerBase.java:1530)
	    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.processChildren(ContainerBase.java:1540)
	    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.processChildren(ContainerBase.java:1540)
	    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.run(ContainerBase.java:1519)
	    at java.lang.Thread.run(Thread.java:744)
    Caused by: java.lang.NoClassDefFoundError: org/apache/commons/fileupload/FileItemFactory
	    at java.lang.Class.getDeclaredFields0(Native Method)
	    at java.lang.Class.privateGetDeclaredFields(Class.java:2397)
	    at java.lang.Class.getDeclaredFields(Class.java:1806)
	    at org.apache.catalina.util.Introspection.getDeclaredFields(Introspection.java:106)
	    at org.apache.catalina.startup.WebAnnotationSet.loadFieldsAnnotation(WebAnnotationSet.java:263)
	    at org.apache.catalina.startup.WebAnnotationSet.loadApplicationServletAnnotations(WebAnnotationSet.java:142)
	    at org.apache.catalina.startup.WebAnnotationSet.loadApplicationAnnotations(WebAnnotationSet.java:67)
	    at org.apache.catalina.startup.ContextConfig.applicationAnnotationsConfig(ContextConfig.java:405)
	    at org.apache.catalina.startup.ContextConfig.configureStart(ContextConfig.java:881)
	    at org.apache.catalina.startup.ContextConfig.lifecycleEvent(ContextConfig.java:376)
	    at org.apache.catalina.util.LifecycleSupport.fireLifecycleEvent(LifecycleSupport.java:119)
	    at org.apache.catalina.util.LifecycleBase.fireLifecycleEvent(LifecycleBase.java:90)
	    at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5322)
	    at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
	    ... 8 more
    Caused by: java.lang.ClassNotFoundException: org.apache.commons.fileupload.FileItemFactory
	    at org.apache.catalina.loader.WebappClassLoader.loadClass(WebappClassLoader.java:1714)
	    at org.apache.catalina.loader.WebappClassLoader.loadClass(WebappClassLoader.java:1559)
	    ... 22 more

原来问题出在没有把文件上传用的两个jar包放在`webcontent/lib`目录下.
