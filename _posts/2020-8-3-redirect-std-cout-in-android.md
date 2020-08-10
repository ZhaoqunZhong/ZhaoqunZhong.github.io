---
layout: post
title: Redirect std::cout to Android log 
categories: [Android, cpp]
---

We know that std::cout prints nothing in Android's logcat. Sometimes we use third-party libraries that was originally developed in desktop environment, thus have tons of std::cout debug output in their code. It's tedious to find them one by one and change the code to replace them with android_log_write() function. We can of course uncomment them all, but what if there are important debug imformation we still need to print. Here comes the rescue from an awsome [answer](https://stackoverflow.com/questions/8870174/is-stdcout-usable-in-android-ndk) to a StackOverflow question. 

## How to use

1. Include this source code in your project. It seems that no overhead will be caused since the stream redirection is launched in a new thread. 

```cpp
#include <iostream>
#include <unistd.h>
#include <pthread.h>
#include <android/log.h>

static int pfd[2];
static pthread_t thr;
static const char *tag = "from-std-cout";

static void *thread_func(void*)
{
    ssize_t rdsz;
    char buf[128];
    while((rdsz = read(pfd[0], buf, sizeof buf - 1)) > 0) {
        if(buf[rdsz - 1] == '\n') --rdsz;
        buf[rdsz] = 0;  /* add null-terminator */
        __android_log_write(ANDROID_LOG_DEBUG, tag, buf);
    }
    return 0;
}

int start_logger(const char *app_name)
{
    tag = app_name;

    /* make stdout line-buffered and stderr unbuffered */
    setvbuf(stdout, 0, _IOLBF, 0);
    setvbuf(stderr, 0, _IONBF, 0);

    /* create the pipe and redirect stdout and stderr */
    pipe(pfd);
    dup2(pfd[1], 1);
    dup2(pfd[1], 2);

    /* spawn the logging thread */
    if(pthread_create(&thr, 0, thread_func, 0) == -1)
        return -1;
    pthread_detach(thr);
    return 0;
}
```

2. Call this function where you want the redirection to begin.
```cpp
start_logger("some log tag you define");
```