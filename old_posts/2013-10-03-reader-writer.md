---
layout: post 
title: 读者写者问题
categories:
- operating system 
---

问题描述

> 有一个许多进程共享的数据区，这个数据区可以是一个文件或者主存的一块空间；有一些只读取这个数据区的进程（Reader）和一些只往数据区写数据的进程(Writer)，此外还需要满足以下条件：

> （1） 任意多个读进程可以同时读这个文件；

> （2） 一次只有一个写进程可以往文件中写；

> （3） 如果一个写进程正在进行操作，禁止任何进程读文件。

思路:

使用一个变量表示当前有多少读者在读,一个互斥量表示对缓冲区的互斥访问

当第一个读者读的时候,对缓冲区加锁,后面的读者来的时候只需要增加读者数目.
当最后一个读者离开的时候,对缓冲区解锁.

写者只需要在写入前获得缓冲区的锁,写完解锁就可以


    #include <stdio.h>
    #include <stdlib.h>
    #include <pthread.h>
    #include <semaphore.h>
    #include <unistd.h>

    const int readerNumber = 5;
    const int writerNumber = 2;

    pthread_mutex_t bufferMutex;//控制对缓冲区的互斥访问
    pthread_mutex_t readerNumberMutex;//控制对当前读者数目的互斥访问

    int currentReaderNumber = 0;

    #define bufferSize  10

    int buffer[bufferSize] = { 0 };

    void* readBuffer( void* );
    void* writeBuffer( void* );
    void modify();
    void print();

    int main(){
	int ret1 = pthread_mutex_init( &bufferMutex, NULL );
	if( ret1 != 0 ){
	    printf("fail to initialize buffer mutex\n");
	    exit( 1);
	}

	pthread_t readers[readerNumber];
	pthread_t writers[writerNumber];

	int retReader[readerNumber];
	int retWriter[writerNumber];

	for( int i = 0; i < readerNumber; ++i ){
	    retReader[i] = pthread_create( readers+i, NULL, readBuffer, NULL );

	    if( retReader[i] != 0 ){
		printf("failed to create reader %d\n", i );
		exit( 1 );
	    }
	}

	for( int i = 0; i < writerNumber; ++i ){
	    retWriter[i] = pthread_create( writers+i, NULL, writeBuffer, NULL );
	    if( retWriter[i] != 0 ){
		printf("failed to create writer %d\n", i );
		exit(1);
	    }
	}

	for( int i = 0; i < readerNumber; ++i ){
	    pthread_join( readers[i], NULL );
	}

	for( int i = 0; i < writerNumber; ++i ){
	    pthread_join( writers[i], NULL );
	}
	exit(0);
    }

    void* readBuffer( void* arg )
    {
	while( 1 ){
	    pthread_mutex_lock( &readerNumberMutex );

	    ++currentReaderNumber;

	    if( currentReaderNumber == 1 )
		pthread_mutex_lock( &bufferMutex );

	    pthread_mutex_unlock( &readerNumberMutex );

	    //访问数据
		printf("reader %u begins to read\n", (unsigned int)pthread_self() );
		print();
		sleep( 2 );
		printf("reader %u finished reading\n", (unsigned int)pthread_self() );

	    //访问结束
	    
	    pthread_mutex_lock( &readerNumberMutex );

	    --currentReaderNumber;

	    if( currentReaderNumber == 0 )//没有读者,释放对缓冲区的锁
		pthread_mutex_unlock( &bufferMutex );

	    pthread_mutex_unlock( &readerNumberMutex );
	    sleep(1);
	}
    }

    void* writeBuffer( void* arg )
    {
	while( 1 ){
	    pthread_mutex_lock( &bufferMutex );

	    printf("writer %u begins to write\n", (unsigned int)pthread_self() );
	    modify();
	    sleep( 1 );
	    printf("writer %u finished writing\n", (unsigned int)pthread_self() );
	    pthread_mutex_unlock( &bufferMutex );
	    sleep(1);
	}
    }

    void modify()
    {
	for( int i = 0; i< bufferSize; ++i ){
	    buffer[i] += 1;
	}
    }

    void print()
    {
	for( int i = 0; i< bufferSize; ++i ){
	    printf("%d    ", buffer[i] );
	}
	printf("\n");
    }

