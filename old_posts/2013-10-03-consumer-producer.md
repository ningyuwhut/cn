---
layout: post 
title: 生产者消费者问题
categories:
- operating system
---

问题描述参考[维基百科](http://zh.wikipedia.org/wiki/%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85%E9%97%AE%E9%A2%98)

问题的解决思路:

使用两个信号量,分别表示缓冲区中空元素的数目和已填充元素的数目.
另外,使用一个互斥量表示对缓冲区的互斥访问.程序中使用pthread_mutex_t类型的互斥量,其实也可以使用一个二值信号量

下面是linux下的实现,该实现是在[这篇文章](http://hi.baidu.com/shazi129/item/4d2a054626be7d17896d1088)的代码上稍微修改一下.

    #include <stdio.h>
    #include <stdlib.h>
    #include <pthread.h>
    #include <semaphore.h>
    #include <unistd.h>

    const int numberOfconsumer = 2;
    const int numberOfProducer = 5;

    #define bufferSize 10

    int producerPos = 0;
    int consumerPos = 0;
    int buffer[bufferSize] = { 0 };

    sem_t empty_sem;//表示缓冲区空元素数目的信号量
    sem_t full_sem;//表示缓冲区已填充元素数目的信号量

    pthread_mutex_t mutex;//控制对缓冲区的互斥访问

    int producerId = 0;
    int consumerId = 0;

    void* consumer( void* );
    void* producer( void* );
    void print();

    int main(){
	int ret1 = sem_init( &empty_sem, 0, bufferSize );
	int ret2 = sem_init( &full_sem, 0, 0 );

	if( ret1 != 0 || ret2 != 0 ){
	    printf("fail to initialize semaphore\n");
	    exit(1);
	}

	int ret3 = pthread_mutex_init(&mutex, NULL );
	if( ret3 != 0 ){
	    printf("fail to initialize mutex\n");
	    exit(1);
	}

	pthread_t consumers[numberOfconsumer];
	pthread_t producers[numberOfProducer];

	int retConsumer[numberOfconsumer];
	int retProducer[numberOfconsumer];

	for( int i = 0; i < numberOfconsumer; ++i ){
	    retConsumer[i] = pthread_create( consumers+i, NULL, consumer, NULL );
	    if( retConsumer[i] != 0 ){
		printf("failed to create consumer %d\n", i );
		exit(1);
	    }
	}

	for( int i = 0; i < numberOfProducer; ++i ){
	    retProducer[i] = pthread_create( producers+i, NULL, producer, NULL );
	    if( retProducer[i] != 0 ){
		printf("failed to create producers %d\n", i );
		exit(1);
	    }
	}

	for( int i = 0; i < numberOfconsumer; ++i ){
	    pthread_join( consumers[i] , NULL );
	}

	for( int i = 0; i < numberOfProducer; ++i ){
	    pthread_join( producers[i], NULL );
	}

	exit( 0 );
    }

    void* consumer( void* arg )
    {
	while( 1 ){

	    sleep(1);

	    sem_wait( &full_sem );

	    pthread_mutex_lock( &mutex );
	    printf("consumer %u consumes \n", (unsigned int )pthread_self() );

	    consumerPos %= bufferSize;
	    buffer[consumerPos] = 0;

	    print();
	    ++consumerPos;

	    pthread_mutex_unlock( &mutex );

	    sem_post( &empty_sem );
	}
    }

    /*
     *把相应的缓冲区中的元素设置为1,表示已填充
     */
    void* producer( void* arg )
    {
	while(1){
	    sleep(1);
	    sem_wait( &empty_sem );

	    pthread_mutex_lock( &mutex );

	    printf("producer %u produces \n", (unsigned int )pthread_self() );
	    producerPos %= bufferSize;
	    buffer[producerPos] = 1;
	    print();
	    ++producerPos;

	    pthread_mutex_unlock( &mutex );
	    sem_post( &full_sem );
	}
    }

    void print()
    {
	for( int i = 0; i< bufferSize; ++i ){
	    printf("%d    ", buffer[i] );
	}
	printf("\n");
    }
