# 第**24**章　弱联网游戏——**Cocos2d-x**客户端实现

本章要介绍的是签到功能的实现，该功能简单常见，通过该功能可以完整地了解一个弱联网请求的处理过程。只要掌握好该功能，其他如登录、公告、邮件、抽奖、商城之类的各种功能，都可以很轻易地实现出来。

本章的内容分为两部分，第一部分主要介绍客户端，如何在客户端**使用Libcurl**发起请求，以及处理请求，并进一步讨论Libcurl的使用问题。第25章主要介绍服务端，包含服务端PHP + Nginx的环境搭建，简单介绍MySQL数据库的搭建和使用，以及一些PHP语法和SQL语法，并用PHP和MySQL实现一个简单的签到服务。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　客户端请求流程。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Libcurl easy接口详解。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用多线程执行请求。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用Libcurl Multi接口进行非阻塞方式请求。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用非阻塞的Libcurl实现签到功能。

### 24.1　客户端请求流程

在客户端使用Libcurl发起请求时，需要顺序执行以下4个步骤。

（1）调用curl_easy_init初始化libcurl。

（2）调用curl_easy_setopt设置请求，如设置请求链接、参数以及接收回调等。

（3）调用curl_easy_perform发起请求。

（4）调用curl_easy_cleanup关闭Libcurl。

在使用Libcurl的时候需要**先调用curl_easy_init()函数创建CURL对象**，然后根据需要调用**curl_easy_setopt()函数对CURL对象进行设置**，如设置要请求哪一个链接、请求后的接收回调等。设置完了之后**调用curl_easy_perform()函数发起请求**，在请求结束之后，可以**调用curl_easy_cleanup()函数进行清理**，释放内存。简单的示例代码如下。

```
#include "curl/curl.h"
//在点击回调中发起请求
void CurlTest::onTouchesEnded(const std::vector<Touch*>& touches, Event
*event)
{
    //初始化libcurl
    CURL* curl = curl_easy_init();
    //设置要请求的连接
   curl_easy_setopt(curl, CURLOPT_URL, "http://webtest.cocos2d-x.org/
   curltest");
   //发起请求，该方法会阻塞程序
   CURLcode res = curl_easy_perform(curl);
   //清理libcurl
   curl_easy_cleanup(curl);
   //显示请求结果
   if(CURLE_OK == res)
   {
       CCLOG("request success");
   }
   else
   {
       CCLOG("request faile, code %d", res);
   }
}
```

如果需要频繁地发起请求，可以只调用一次curl_easy_init()和curl_easy_cleanup()函数，然后在这期间进行多次调用curl_easy_setopt()（设置请求参数）和curl_easy_perform()函数（发起请求）以提高请求效率。

### 24.2　Libcurl easy接口详解

首先是curl_easy_init()和curl_easy_cleanup()这对函数，它们负责libcurl的初始化和清理，必须成对出现，可以多次调用来创建多个Libcurl对象，在函数内部会调用curl_global_init()以及curl_global_cleanup()函数来执行全局的初始化和清理，这些函数都**不是线程安全的**。

curl_easy_perform()函数将使用一个配置好的CRUL指针发起请求，当返回CURLE_OK时表示请求成功，否则表示请求失败，失败的原因可以查看CURLcode枚举的定义。

curl_easy_setopt()方法是curl中最最复杂的一个方法，其函数原型如下。

```
CURLcode curl_easy_setopt(CURL *curl, CURLoption option, ...);
```

curl_easy_setopt()方法可以为CURL对象设置数10种选项，不同的选项对应不同的参数类型，可以输入整型、字符串、函数地址等。函数最后一个参数...表示可以输入任意类型和数量的参数，类似printf函数。下面介绍一些常用的选项。

#### 24.2.1　关于请求链接

CURLOPT_URL选项可以指定要请求的URL链接，该选项对应的参数为char*类型，程序员需要传入一个网址，如http://www.baidu.com。

CURLOPT_TIMEOUT选项可以设定**请求的超时时间**，该选项对应的参数为long类型，单位为秒，默认为0，表示不限定超时时间。如果需要更精确的时间，可以使用CURLOPT_TIMEOUT_MS选项来设置以毫秒为单位的超时时间。**当CURLOPT_ TIMEOUT和CURLOPT_TIMEOUT_MS都被设置时，最后调用的设置会生效**。

CURLOPT_HTTPGET选项可以设置请求类型为GET，该选项对应的参数为long类型，设置为0表示禁用，设置为1表示开启。当将该选项设置为1时，**程序会自动将CURLOPT_NOBODY和CURLOPT_UPLOAD设置为0**。默认就是Http Get模式，Get模式是最简单的HTTP请求模式，通过URL地址向服务器请求内容，可以在URL的尾部追加?key1=value1&key2=value2&key3=value3（**例如http://www.baidu.com?ie=utf-8**）来向服务器传递少量的key value参数（**URL最大长度大约是2000个字节**）。当设置了PUT或POST模式时，设置HTTP GET将重置回Get模式。

CURLOPT_PUT选项可以设置请求类型为HTTP PUT，该选项对应的参数为long类型，默认为0表示禁用，设置为1表示开启。PUT请求将上传数据到服务器（**如上传照片文件**），通过**设置CURLOPT_READDATA选项可以指定要传输的数据内容**。在Libcurl 7.12之后的版本，CURLOPT_PUT被**CURLOPT_UPLOAD替代**。

CURLOPT_POST选项可以设置请求类型为HTTP POST，提交表单数据到服务器，启用该选项后，可以**设置CURLOPT_POSTFIELDS和CURLOPT_COPYPOSTFIELDS选项来指定要传输到服务器的数据**。该选项对应的参数为long类型，默认为0表示禁用，设置为1表示开启。

#### 24.2.2　关于Post提交表单

CURLOPT_POSTFIELDS选项可以设置要POST到服务器的数据，当请求类型为POST时才会生效，对应的参数为char*，参数的值指定格式的字符串，如"key1=value1&key2= value2&key3=value3"。

CURLOPT_HTTPPOST选项可以设置要发送的Post数据，可以传输文本数据以及文件（**传输文件时需要指定一个可读的文件路径**），通过curl_formadd()方法填充一个curl_httppost结构体，在调用curl_easy_setopt()方法设置CURLOPT_HTTPPOST时，将curl_httppost结构体作为选项设置的参数传入。示例代码如下。

```
struct curl_httppost *formpost=NULL;
struct curl_httppost *lastptr=NULL;
curl_global_init(CURL_GLOBAL_ALL);
//添加一个key为sendfile，value为指定文件路径的表单信息
//在表单被提交时，指定的文件路径对应的文件postit2.c会被一起发送到服务器
curl_formadd(&formpost,
           &lastptr,
           CURLFORM_COPYNAME, "sendfile",
           CURLFORM_FILE, "postit2.c",
           CURLFORM_END);
//添加一个key为filename，value为postit2.c的表单文本信息
curl_formadd(&formpost,
           &lastptr,
           CURLFORM_COPYNAME, "filename",
           CURLFORM_COPYCONTENTS, "postit2.c",
           CURLFORM_END);
CURL* curl = curl_easy_init();
//设置要提交的表单
curl_easy_setopt(curl, CURLOPT_HTTPPOST, formpost);
//提交请求
curl_easy_perform(curl);
curl_easy_cleanup(curl);
//最后不要忘了清理表单对象占用的内存
curl_formfree(formpost);
```

curl_formadd的第一和第二个参数都是curl_httppost二级指针，第一次调用时参数应该被设置为NULL，接下来的参数“...”是可变参数，在这里必须成对地传入CURLformoption枚举成员和对应的值，在最后以CURLFORM_END结尾。

curl_formadd()函数可以添加字符串CURLFORM_COPYCONTENTS、文件CURLFORM_FILE以及二进制数据CURLFORM_PTRCONTENTS（**二进制数据需要再设置CURLFORM_CONTENTSLENGTH来指定数据的长度，默认为0，会调用strlen来判断长度**），如果传输的是文件，可以设置CURLFORM_CONTENTTYPE来指定文件类型，在http://tool.oschina.net/commons网址中可以找到常用的类型。

curl_formadd()函数所分配的curl_httppost指针对象必须使用curl_formfree()函数来释放，以避免内存泄漏。关于curl_formadd()函数更详细的内容可参阅http://curl.haxx.se/ libcurl/c/ curl_formadd.html。

#### 24.2.3　关于读写

CURLOPT_WRITEDATA可以传入一个指针，当设置CURLOPT_WRITEFUNCTION选项注册回调函数时，设置的指针会作为参数传入回调函数中。如果不注册回调函数，传入一个文件句柄，如**一个以可写方式打开的文件句柄**（**一个FILE指针转换为void***），那么接收到的数据将会被写入到这个文件中。

CURLOPT_WRITEFUNCTION可以传入一个函数地址，**在接收到数据之后会调用该函数**，回调函数的原型为size_t fun(void* ptr, size_t size, size_t nmemb, void *userdata)，第1个参数ptr是接收到的内容，第2和第3个参数相乘可以得出接收到的内容有多少字节。当设置了回调函数之后，在CURLOPT_WRITEDATA设置的指针会作为userdata传入（**如果有**）。需要注意的是，在一次请求中，这个回调可能会被多次调用，由于TCP传输的问题，一个响应的数据包可能会被分为多段进行传输（**半包粘包问题**），应该在回调中将数据缓存，当数据接收完整之后再进行处理。那么如何判断数据接收完整呢？如果是阻塞则调用curl_easy_perform()函数，在curl_easy_perform()函数返回之后再进行处理,否则需要自己根据协议来判断，一般是判断ptr内容是否包含协议的结束标识。

CURLOPT_READDATA和CURLOPT_WRITEDATA类似，可以传入一个void指针来作为CURLOPT_READFUNCTION设置的回调的第4个参数，也可以设置一个可读的文件句柄，当Libcurl读取文件上传到服务器时，从这个文件读取数据，默认值为stdin标准输入流。

CURLOPT_READFUNCTION和CURLOPT_WRITEFUNCTION类似，设置一个回调函数，在Libcurl上传文件到服务器时调用，回调函数的原型为size_t read_callback(char *buffer, size_t size, size_t nitems, void *instream)。默认的读取回调是标准函数fread()，buffer是要填充的缓存区，size * nitems可以得到要读取的内容大小，instream一般是一个文件句柄（**由CURLOPT_READDATA设置**），填充完缓存区后，返回成功读取的元素个数。

curl_easy_setopt()函数还可以设置其他数10个参数，包括下载进度回调、限速、传输安全、cookie等，具体可以参考官方对curl_easy_setopt的详细介绍，网址为http://curl.haxx.se/ libcurl/ c/curl_easy_setopt.html。

### 24.3　使用多线程执行请求

有时curl_easy_perform操作会造成一定的阻塞，当使用Libcurl从网上下载文件时（**很多情况下会用来动态更新资源**），或者在网络不佳的情况下，很容易碰到阻塞，一旦发生了阻塞，程序会整个停止，可以使用多线程或Libcurl的非阻塞方式来解决阻塞问题。下面是CURL中使用多线程发起多个请求的示例。

```
#include <stdio.h>
#include <pthread.h>
#include <curl/curl.h>
#define NUMT 4
//要访问的连接数组
const char * const urls[NUMT]= {
    "http://curl.haxx.se/",
    "ftp://cool.haxx.se/",
    "http://www.contactor.se/",
    "www.haxx.se"
};
//线程函数，执行一个请求
static void* pull_one_url(void *url)
{
    CURL *curl;
    curl = curl_easy_init();
    curl_easy_setopt(curl, CURLOPT_URL, url);
    curl_easy_perform(curl); /* ignores error */
    curl_easy_cleanup(curl);
    return NULL;
}
int main(int argc, char **argv)
{
    pthread_t tid[NUMT];
    int i;
    int error;
    //在创建线程之前先初始化CURL
    curl_global_init(CURL_GLOBAL_ALL);
    //for循环创建线程
    for(i=0; i< NUMT; i++) {
        error = pthread_create(&tid[i],
                            NULL, /* default attributes please */
                            pull_one_url,
                            (void *)urls[i]);
        if(0 != error)
            fprintf(stderr, "Couldn't run thread number %d, errno %d\n", i,
            error);
        else
           fprintf(stderr, "Thread %d, gets %s\n", i, urls[i]);
   }
   //在主线程中等待其他线程全部执行完，pthread_join()是一个阻塞函数
   for(i=0; i< NUMT; i++) {
       error = pthread_join(tid[i], NULL);
       fprintf(stderr, "Thread %d terminated\n", i);
   }
   curl_global_cleanup();
   return 0;
}
```

在Cocos2d-x 3.0中，也可以使用C++ 11的thread来替代pthread系列函数，例如，将pthread_create()替换为std::thread t(pull_one_url, urls[i])，将最后的pthread_join()替换为t.join();（可以选择定义N个线程对象，或者用一个数组来保存它们的指针）。

需要注意的是，**一个CURL对象，只能在一条线程中处理**，不能多条线程同时处理一个CURL对象，另外注意**curl_global_init()和culr_global_cleanup()是一对线程不安全的函数**，curl_global_init()函数应该在所有线程创建之前执行，而culr_global_cleanup()函数应该在所有线程结束后执行。

如果在线程中接收到消息，要处理消息时，涉及Cocos2d-x界面内容修改的部分必须在Cocos2d-x主线程执行，多个线程同时操作Cocos2d-x的场景，有可能导致黑屏或崩溃等异常。

通过调用Schedule的performFunctionInCocosThread()方法传入回调函数，可以在主线程中执行相应的代码。

### 24.4　使用Libcurl Multi接口进行非阻塞请求

24.3节介绍了easy接口，easy接口提供了最基础的CURL对象，能够以阻塞的方式处理单个请求，如果需要**同时发起多个请求**，需要借助线程。

而Multi接口能够**在同一线程内以非阻塞的方式处理多个并发请求**。由于multi接口是非阻塞的，所以使用起来比easy接口要复杂很多，并且在很多情况下需要结合select()函数来处理请求。

select()函数是用于并发管理通知的系统函数，在Windows和Linux以及Mac下都有。当使用非阻塞的方式处理请求时，请求的结果并不会立即返回。那么如何知道请求的结果是否返回了呢？答案是**在每一次主循环中调用接收数据的方法去接收数据**（这里接收数据的方法被Libcurl封装起来了），如果请求返回了，就可以接收到内容。

当有很多个并发请求的时候，这种方法就需要在每一次主循环中遍历所有的请求对象，让它们接收数据（**一般指Socket**），这种遍历的方式效率低下。而select()函数可以帮助程序员管理这些连接，当连接的数据到达可以读取的时候，通知应用程序来处理连接，这种监听——触发的方式与遍历的方式相比，效率提高了很多。程序员需要在每次主循环执行时调用select()方法来判断是否有消息可处理，如果是则对触发的消息进行处理。

#### 24.4.1　multi接口

以下是Libcurl提供的Multi系列接口。

```
    //初始化一个CURLM*指针，同curl_easy_init类似
    CURLM *curl_multi_init( );
    //清理CURLM*指针，同curl_easy_cleanup类似
    CURLMcode curl_multi_cleanup( CURLM *multi_handle );
    //添加一个easy对象到Multi对象中，由Multi对象进行管理
    CURLMcode curl_multi_add_handle(CURLM*multi_handle,CURL*easy_handle);
    //将一个已添加到Multi对象中的easy对象移除
    CURLMcode curl_multi_remove_handle(CURLM*multi_handle,CURL*easy_handle);
    //用于替代curl_easy_perform，发起以及处理非阻塞的请求
    //running_handles表示当前正在处理的请求数量
    CURLMcode curl_multi_perform(CURLM*multi_handle,int*running_handles);
    //从Multi对象中获取文件描述符集合，用于select()函数的参数
    //文件描述符在这里实际上是一个Socket网络连接对象
    //当Libcurl并未初始化任何Socket对象时，max_fd为-1
    CURLMcode curl_multi_fdset(CURLM *multi_handle,
                            fd_set *read_fd_set,
                            fd_set *write_fd_set,
                            fd_set *exc_fd_set,
                            int *max_fd);
    //curl_multi_tineout()函数用于询问libcurl超时时间，在超时时间结束之前，应该
    调用一次curl_multi_perform()方法
    //超时时间的单位为毫秒，为0表示应该立即处理，为-1表示没有设置超时时间，无须处理
    CURLMcode curl_multi_timeout(CURLM *multi_handle, long *timeout);
    //Multi对象会将每个请求结束后的信息存入Multi栈中，curl_multi_info_read()
    方法可以读取所有已完成请求的信息
    //这里的请求结束，包含了正常结束和异常结束。
    //只有通过curl_multi_info_read()方法才能知道请求的结果，因为perform并不会
    告知请求结束了或者发生了异常
    //curl_multi_info_read()每次调用，都会将信息从Multi栈中移除，msgs_in_
    queue()函数会告诉我们栈中还剩下多少消息
    //当Multi栈中没有消息时，该函数返回NULL
    CURLMsg *curl_multi_info_read( CURLM *multi_handle,   int *msgs_in_
    queue);
    //设置Multi选项，在大多数情况下不需要设置
    //大部分选项是为比较复杂的Multi Socket API提供
    CURLMcode curl_multi_setopt(CURLM * multi_handle, CURLMoption option,
param);
```

#### 24.4.2　Multi工作流程

##### 1．Multi接口工作流程

Multi接口工作流程如图24-1所示，使用非阻塞的Libcurl需要经历以下步骤。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121406.jpeg)

图24-1　Multi接口工作流程

首先是初始化阶段：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调用curl_multi_init()函数初始化返回一个CURLM指针。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调用curl_multi_add_handle()函数添加要请求的CURL对象。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调用curl_multi_perform()函数开始请求。

接下来是主循环阶段：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调用curl_multi_timeout()函数获取等待时间。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调用curl_multi_fdset()函数从CURL中获取maxfd以及fdset。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调用select()函数询问fdset中是否有数据可读或可写。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　如果是则调用curl_multi_perform()函数驱动CURL进行读写操作。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　操作完成后，循环调用curl_multi_info_read()函数来判断请求的结果。

所有的请求处理完之后，假设不需要再请求，可以进入清理阶段：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　先将结束的请求调用curl_multi_remove_handle()函数进行移除。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调用curl_easy_cleanup()函数清除CURL指针。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调用curl_multi_cleanup()函数清除CURLM指针。

##### 2．Multi接口使用详解

首先用curl_easy_init()函数和curl_easy_setopt()函数创建并设置好Libcurl连接句柄，并且用curl_multi_init()函数初始化一个CURLM句柄对象，然后调用curl_multi_add_handle()函数将CURL句柄添加到CURLM句柄中（**可以添加多个句柄到CURLM**），然后调用curl_multi_perform()函数将启动非阻塞的请求。

那些被添加到CURLM句柄中的Libcurl句柄将会被执行，curl_multi_perform()方法将会立即返回，但并不表示请求已经被处理完了，curl_multi_perform()函数的第二个参数是一个输出参数，表示还未处理完的句柄数，在每次调用完curl_multi_perform()函数的时候，假设某个请求已经处理结束，该参数会减1。

可以在程序的主循环中执行curl_multi_perform()函数，直到所有的请求都处理完毕。但更好的写法是在主循环中使用select()方法来进行检测，当有事件触发的时候，再去调用curl_multi_perform()函数，Libcurl也建议我们这么做。select()方法在第26章中会详细介绍。

如果需要处理服务器返回的内容，可以通过在创建CURL的时候（**不是CURLM**），设置CURL的CURLOPT_WRITEFUNCTION选项来指定处理回调，如果有多个请求，那么可以给每个请求设置不同的回调来处理，如果希望设置同一个函数来处理不同的请求，那么可以通过设置CURLOPT_WRITEDATA选项来设置回调函数的第4个参数，在函数中根据参数来区分请求。WriteFunction回调应该记录服务器返回的内容，当这个请求结束之后，再来进行处理。

在调用select()方法之前需要知道超时时间，curl_multi_timeout返回的超时时间是Libcurl内部计算的时间而不是手动设置的超时时间，表示需要在这个时间之内调用一次处理函数（**可以是curl_multi_perform()也可以是curl_multi_socket()函数**），超时时间的单位为毫秒，为0表示应该立即处理，为-1表示没有设置超时时间，无须处理。

在调用select()方法之前，还需要调用curl_multi_fdset()方法从CURL中获取文件描述符集合，作为select()方法的参数。当输出的maxfd为-1时表示Socket套接字还没有准备好，这时候如果调用select()函数可能会出错，Libcurl建议我们在这种情况下等待100毫秒之后，再重新操作。

在Multi接口中，**一个CURL请求的结果，不论成功还是失败，都会将请求的结果存储到一个Multi栈中，我们需要主动调用curl_multi_info_read()函数来获取请求的结果**。curl_multi_info_read()函数需要传入一个int指针参数来保存Multi栈剩下的信息数量，并返回一个CURLMsg指针，通过循环调用来取出所有结束的请求。

##### 3．Multi接口示例代码

Multi接口的示例代码如下。

```
#include <stdio.h>
#include <string.h>
#include <sys/time.h>
#include <curl/curl.h>
int main()
{
    int still_running;
    int msgs_in_queue;
    CURL *handle1 = curl_easy_init();
    CURL *handle2 = curl_easy_init();
    CURLM *multi_handle = curl_multi_init();
    //初始化两个普通的请求
    curl_easy_setopt(handle1, CURLOPT_URL, "http://www.baidu.com/");
    curl_easy_setopt(handle2, CURLOPT_URL, "http://www.google.com/");
    //将两个请求添加到multi_handle()函数中
    curl_multi_add_handle(multi_handle, handle1);
    curl_multi_add_handle(multi_handle, handle2);
    //still_running是当前还在执行的句柄数
    curl_multi_perform(multi_handle, &still_running);
    do
    {
        //根据still_running来判断是否有请求未执行完
        if(still_running <= 0)
        {
            break;
        }
        struct timeval timeout;
        int rc;
        fd_set fdread;
        fd_set fdwrite;
        fd_set fdexcep;
        int maxfd = -1;
        long curl_timeo = -1;
        FD_ZERO(&fdread);
        FD_ZERO(&fdwrite);
        FD_ZERO(&fdexcep);
        timeout.tv_sec = 1;
        timeout.tv_usec = 0;
        //获取要等待的超时时间，并设置到timeout中
        curl_multi_timeout(multi_handle, &curl_timeo);
        if(curl_timeo >= 0)
        {
            timeout.tv_sec = curl_timeo / 1000;
            if(timeout.tv_sec > 1)
                timeout.tv_sec = 1;
            else
                timeout.tv_usec = (curl_timeo % 1000) * 1000;
        }
        //从CURLM中取出fdset，并返回maxfd
        curl_multi_fdset(multi_handle,&fdread, &fdwrite, &fdexcep, &maxfd);
        if(maxfd == -1)
        {
            //如果maxfd为-1，说明网络未完成初始化，建议等待100毫秒
            //Windows和其他平台的等待方法不同
#ifdef _WIN32
            Sleep(100);
            rc = 0;
#else
            struct timeval wait = { 0, 100 * 1000 };
            rc = select(0, NULL, NULL, NULL, &wait);
#endif
        }
        else
        {
            //select()函数监听是否有读写事件触发，此时的select()函数是阻塞函数，
            会阻塞timeout所指定的时间
            //如果不希望select阻塞，可以将timeout的两个成员变量都设置为0，传入
            NULL表示一直阻塞
            //当有连接可读或可写时，select()函数会立即返回可读写连接的数量，如果超时
            则返回0，返回-1表示异常
            rc = select(maxfd+1, &fdread, &fdwrite, &fdexcep, &timeout);
        }
       //根据select()函数的结果来处理
        switch(rc)
        {
            //select()函数发生了错误
            case -1:
                still_running = 0;
                printf("select() returns error, this is badness\n");
                break;
            //有数据可读或可写，调用curl_multi_perform()函数驱动数据读写
            //curl_multi_perform()函数会将剩余正在执行的请求输出到still_running中
           //CURL句柄设置的回调
           case 0:
           default:
               curl_multi_perform(multi_handle, &still_running);
               break;
       }
       CURLMsg * msg = NULL;
       do
       {
           msg = curl_multi_info_read( multi_handle, msgs_in_queue);
           if(msg)
           {
               //打印请求结果，并清理CURL对象
               printf("%d request finish result %d ", msg->easy_handle,
               msg->data.result);
               curl_multi_remove_handle( multi_handle, msg->easy_handle);
               curl_easy_cleanup( msg->easy_handle);
           }
           else
           {
               break;
           }
       } while(true);
   }while(true);
   //清理Multi对象
   curl_multi_cleanup( multi_handle );
   return 0;
}
```

### 24.5　使用非阻塞的Libcurl实现签到功能

24.4节的例子关于请求的使用并不是在Cocos2d-x中使用的，easy接口以及多线程请求这两种方式都可以在Cocos2d-x中直接使用，但是Multi接口要在Cocos2d-x中使用则需要一些改变。因为24.4节的例子实现了程序的主循环，以及等待100毫秒、等待超时等阻塞操作，而在Cocos2d-x中则不需要去实现程序的主循环。

程序要实现的签到功能非常简单，首先客户端发起签到请求，将自己的用户ID提交，那么用户ID从哪来呢？因为我们并没有注册和登录这样的服务，所以这个ID也由签到服务来生成，这个做法只是为了演示，存在安全隐患，请勿模仿。程序默认的ID是-1，保存在UserDeafult中，当请求服务器时，如果服务器发现程序的ID是-1，那么将视为一个新的用户并分配一个ID，然后我们将ID保存到UserDeafult中，下次再请求，从UserDeafult中获取ID，发送的就是服务器返回的ID。

为什么要这么复杂地创建和维护这个ID呢？因为程序要实现每日签到的功能，必须知道用户今天是否签到了，服务器需要一个ID来记录用户今天是否已经签到了。正常的签到请求是不应该由客户端发送ID的，其存在的安全隐患就是，客户端可以发送任何ID过来，帮任何一个人进行签到。这种模式称为信任客户端，也就是客户端说了算。在这种模式下的服务器开发会轻松很多，但换来的是外挂的有机可乘。所以安全的做法是对客户端最少的信任，将数据和逻辑控制在服务端。

请求完之后，会接收到服务器返回的信息，首先把ID记录下来，然后判断签到成功与否，弹出对应的对话框即可。当然，如签到成功的话可以给出各种奖励的，大家可以自由发挥。

#### 24.5.1　初始化界面

在场景的init()函数中，先进行初始化，首先初始化CURLM对象及一些成员变量，并在场景中添加一个签到按钮，当用户单击按钮时，执行signin()函数。m_multi_handle和m_still_running是场景的成员变量。

```
//初始化变量
m_multi_handle = curl_multi_init();
m_still_running = 0;
//单击signin按钮发起登录请求
auto item = MenuItemFont::create("signin", [this](Ref*)->void
{
    int uid = UserDefault::getInstance()->getIntegerForKey("uid", -1);
    this->signin(uid);
});
Menu* menu = Menu::create(item, NULL);
addChild(menu);
//注册update回调
scheduleUpdate();
```

#### 24.5.2　发起请求

发起签到请求的代码很简单，只是添加一个CURL对象到Multi对象中，比较关键的地方是在接收数据处为每个CURL对象分配了一个string对象，在接收数据时将接收到的数据追加到string对象的尾部。

```
size_t writeFun(void* ptr, size_t size, size_t nmemb, void *userdata)
{
    *((string*)(userdata)) += (char*)ptr;
    return nmemb;
}
void HelloWorld::signin(int uid)
{
    CURL* request = curl_easy_init();
    m_requestMap[request] = "";
    char buf[128];
    curl_easy_setopt(request, CURLOPT_URL, "http://localhost/signin.php");
    snprintf(buf, sizeof(buf), "uid=%d", uid);
    curl_easy_setopt(request, CURLOPT_POSTFIELDS, buf);
    curl_easy_setopt(request, CURLOPT_WRITEFUNCTION, writeFun);
    curl_easy_setopt(request, CURLOPT_WRITEDATA, &m_requestMap[request]);
    curl_multi_add_handle(m_multi_handle, request);
   curl_multi_perform(m_multi_handle, &m_still_running);
}
```

在signin成员函数中发起请求，根据传入的uid来发起请求，请求的URL是http://localhost/signin.php，在第25章中会介绍服务器的搭建以及服务器代码的编写。

我们要传入的uid是从UserDeafult中获取的一个数值，默认为-1，表示一个新用户，当收到签到结果时会将uid保存，在这里将uid作为POST参数设置进去。

在这里设置了CURLOPT_WRITEDATA和CURLOPT_WRITEFUNCTION，首先传入了writeFun指针作为数据接收回调函数，并设置m_requestMap[request]的地址作为回调函数的userdata参数，m_requestMap是场景的成员变量，其类型是std::map<CURL*, std::string>，以CURL*为key，以string为value。

当我们发起一个请求时，为这个CURL*对象分配一个string用于接收服务器返回的数据，并在writeFun中把数据追加到字符串中。这样做的目的是当知道某个CURL对象请求完毕时，可以根据这个对象来获取其接收到的数据。

将数据接收和数据处理分离，对于一些请求过长的内容，会分为多次下发，这样在接收函数中只需要将接收到的内容保存追加即可，不需要去判断是否接收完成。注意！**这里为每个CURL都分配了一个独立string来接收数据，这个string的指针作为writeFun的userdata传入**。

最后将CURL对象添加到CURLM中，并调用curl_multi_perform()函数开始执行，此时会输出正在执行的请求数量到m_still_running中。

#### 24.5.3　处理结果

在场景的update()函数中判断是否有请求正在执行，如果有则调用curl_multi_perform，然后调用process()成员函数进行处理。在process()中判断是否有请求结束了，如果有则进行处理。首先判断请求是否成功，如果请求失败，则弹出对话框net work error，在访问一个不存在的网页时会执行，如果请求成功，则判断服务器返回的结果，看是否签到成功，如成功则弹出sign in success对话框，如失败则弹出sign in faile对话框（**在同一天重复签到，服务器会返回签到失败**）。

最后设置服务器返回的uid，并清理CURL对象及m_requestMap中对应的响应内容。

```
void HelloWorld::update(float dt)
{
    //根据m_still_running来判断是否有请求未执行完
    if (m_still_running <= 0)
    {
        return;
    }
    curl_multi_perform(m_multi_handle, &m_still_running);
    process();
}
void HelloWorld::process()
{
    CURLMsg * msg = NULL;
    int msgs_in_queue;
    while (msg = curl_multi_info_read(m_multi_handle, &msgs_in_queue))
        {
            CURL* easy = msg->easy_handle;
            //打印请求结果，并清理CURL对象
            if (msg->data.result == CURLE_OK)
            {
                //调用getMapFromResult()方法解析返回结果，后面会介绍到该方法的实现
                auto dict = getMapFromResult(m_requestMap[easy]);
                if (dict["result"] == 1)
                {
                    cocos2d::MessageBox("sign in success", "signin");
                }
                else
                {
                    cocos2d::MessageBox("sign in faile", "signin");
                }
                UserDefault::getInstance()->setIntegerForKey("uid",
dict["uid"]);
            }
            else
            {
                cocos2d::MessageBox("net work error", "signin");
            }
            m_requestMap.erase(easy);
            curl_multi_remove_handle(m_multi_handle, easy);
            curl_easy_cleanup(easy);
        }
    }
```

当运行服务器并单击sign in按钮时，会弹出签到成功或失败的提示信息，如图24-2所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121407.jpeg)

图24-2　签到成功

#### 24.5.4　解析结果

在24.5.3节的代码中使用了getMapFromResult()函数将服务器返回的字符串，解析到一个Map中，Map的key是string类型，value是int类型。服务器下发的内容会是这样的"uid=122\nresult=1"，在getMapFromResult()函数中先用\n作为分隔符，抽出所有的a=b这样的字符串，然后再以=为分隔符，将a和b分离，最后进行转换并存到要返回的Map中。

```
vector<string> splitString(const string& str, char p)
{
    size_t offset = 0;
    vector<string> vec;
    do
    {
        size_t pos = str.find(p, offset);
        if (pos == string::npos)
        {
            vec.push_back(str.substr(offset, str.length() - offset));
            break;
        }
        else
        {
            vec.push_back(str.substr(offset, pos - offset));
            offset = pos + 1;
        }
    } while (true);
    return vec;
}
map<string, int> getMapFromResult(const string& result)
{
    map<string, int> ret;
    vector<string> vec = splitString(result, '\n');
    for (auto iter = vec.begin(); iter != vec.end(); ++iter)
    {
        vector<string> vecKV = splitString(*iter, '=');
        if (vecKV.size() == 2)
        {
            ret[vecKV[0]] = atoi(vecKV[1].c_str());
        }
    }
    return ret;
}
```

在实际开发过程中，用Json和Protobuffer来作为传输的协议格式是比较常见的，在例中没有使用而是使用了一段自定义的文本协议。