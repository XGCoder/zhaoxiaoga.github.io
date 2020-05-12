---
title: 6.NSURLSession的使用
date: 2016-05-27 14:53:48
tags:
---
NSURLSession 基本使用:

### 使用步骤:

使用NSURLSession创建`NSURLSessionTask`,然后执行`NSURLSessionTask`的子类`NSURLSessionDataTask` \ `NSURLSessionUploadTask` \ `NSURLSessionDownloadTask`

<Excerpt in index | 首页摘要>
 <!-- more -->
<The rest of contents | 余下全文>

* 发送第一种`get`请求
```
//1.创建NSURLSession对象（可以获取单例对象）
    NSURLSession *session = [NSURLSession sharedSession];

    //2.根据NSURLSession对象创建一个Task

    NSURL *url = [NSURL URLWithString:@"http://xxxx/login?username=xxx&pwd=xxx&type=JSON"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    //方法参数说明
    /*
    注意：该block是在子线程中调用的，如果拿到数据之后要做一些UI刷新操作，那么需要回到主线程刷新
    第一个参数：需要发送的请求对象
    block:当请求结束拿到服务器响应的数据时调用block
    block-NSData:该请求的响应体
    block-NSURLResponse:存放本次请求的响应信息，响应头，真实类型为NSHTTPURLResponse
    block-NSErroe:请求错误信息
     */
   NSURLSessionDataTask * dataTask =  [session dataTaskWithRequest:request completionHandler:^(NSData * __nullable data, NSURLResponse * __nullable response, NSError * __nullable error) {

        //拿到响应头信息
        NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;

        //4.解析拿到的响应数据
        NSLog(@"%@\n%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding],res.allHeaderFields);
    }];

    //3.执行Task
    //注意：刚创建出来的task默认是挂起状态的，需要调用该方法来启动任务（执行任务）
    [dataTask resume];
```


* 发送第二种`get`请求
```
//注意：该方法内部默认会把URL对象包装成一个NSURLRequest对象（默认是GET请求）
    //方法参数说明
    /*
    //第一个参数：发送请求的URL地址
    //block:当请求结束拿到服务器响应的数据时调用block
    //block-NSData:该请求的响应体
    //block-NSURLResponse:存放本次请求的响应信息，响应头，真实类型为NSHTTPURLResponse
    //block-NSErroe:请求错误信息
     */
- (nullable NSURLSessionDataTask *)dataTaskWithURL:(NSURL *)url completionHandler:(void (^)(NSData * __nullable data, NSURLResponse * __nullable response, NSError * __nullable error))completionHandler;
```

* 发送post请求
```
//1.创建NSURLSession对象（可以获取单例对象）
    NSURLSession *session = [NSURLSession sharedSession];

    //2.根据NSURLSession对象创建一个Task

    NSURL *url = [NSURL URLWithString:@"http://xxxxx/login"];

    //创建一个请求对象，并这是请求方法为POST，把参数放在请求体中传递
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    request.HTTPMethod = @"POST";
    request.HTTPBody = [@"username=xxx&pwd=xxx&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];

    NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * __nullable data, NSURLResponse * __nullable response, NSError * __nullable error) {
        //拿到响应头信息
        NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;

        //解析拿到的响应数据
        NSLog(@"%@\n%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding],res.allHeaderFields);
    }];

    //3.执行Task
    //注意：刚创建出来的task默认是挂起状态的，需要调用该方法来启动任务（执行任务）
    [dataTask resume];
```

### NSURLSession下载文件
1. 创建NSURLSession对象，设置代理（默认配置）

```
    1.创建NSURLSession,并设置代理
    /*
     第一个参数：session对象的全局配置设置，一般使用默认配置就可以
     第二个参数：谁成为session对象的代理
     第三个参数：代理方法在哪个队列中执行（在哪个线程中调用）,如果是主队列那么在主线程中执行，如果是非主队列，那么在子线程中执行
     */
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
```


2. 根据Session对象创建一个NSURLSessionDataTask任务（post和get选择）

```
//创建task
NSURL *url = [NSURL URLWithString:@"http://xxxx/img/hehe.png"];

//注意：如果要发送POST请求，那么请使用dataTaskWithRequest,设置一些请求头信息
NSURLSessionDataTask *dataTask = [session dataTaskWithURL:url];
```

3. 执行任务（其它方法，如暂停、取消等）

```
    //启动task
    //[dataTask resume];
    //其它方法，如取消任务，暂停任务等
    //[dataTask cancel];
    //[dataTask suspend];
```
4. 遵守代理协议，实现代理方法（3个相关的代理方法）

```
/*
 1.当接收到服务器响应的时候调用
     session：发送请求的session对象
     dataTask：根据NSURLSession创建的task任务
     response:服务器响应信息（响应头）
     completionHandler：通过该block回调，告诉服务器端是否接收返回的数据
 */
- (void)URLSession:(nonnull NSURLSession *)session dataTask:(nonnull NSURLSessionDataTask *)dataTask didReceiveResponse:(nonnull NSURLResponse *)response completionHandler:(nonnull void (^)(NSURLSessionResponseDisposition))completionHandler

/*
 2.当接收到服务器返回的数据时调用
 该方法可能会被调用多次
 */
- (void)URLSession:(nonnull NSURLSession *)session dataTask:(nonnull NSURLSessionDataTask *)dataTask didReceiveData:(nonnull NSData *)data

/*
 3.当请求完成之后调用该方法
 不论是请求成功还是请求失败都调用该方法，如果请求失败，那么error对象有值，否则那么error对象为空
 */
- (void)URLSession:(nonnull NSURLSession *)session task:(nonnull NSURLSessionTask *)task didCompleteWithError:(nullable NSError *)error
```

5. 当接收到服务器响应的时候，告诉服务器接收数据（调用block）
```
//默认情况下，当接收到服务器响应之后，服务器认为客户端不需要接收数据，所以后面的代理方法不会调用
    //如果需要继续接收服务器返回的数据，那么需要调用block,并传入对应的策略

    /*
        NSURLSessionResponseCancel = 0, 取消任务
        NSURLSessionResponseAllow = 1,  接收任务
        NSURLSessionResponseBecomeDownload = 2, 转变成下载
        NSURLSessionResponseBecomeStream NS_ENUM_AVAILABLE(10_11, 9_0) = 3, 转变成流
    */

    completionHandler(NSURLSessionResponseAllow);
```

### NSURLSessionDownloadTask 大文件下载
1. 使用NSURLSession和NSURLSessionDownload可以很方便的实现文件下载操作

```
/*
     第一个参数：要下载文件的url路径
     第二个参数：当接收完服务器返回的数据之后调用该block
     location:下载的文件的保存地址（默认是存储在沙盒中tmp文件夹下面，随时会被删除）
     response：服务器响应信息，响应头
     error：该请求的错误信息
     */
    //说明：downloadTaskWithURL方法已经实现了在下载文件数据的过程中边下载文件数据，边写入到沙盒文件的操作
    NSURLSessionDownloadTask * downloadTask = [session downloadTaskWithURL:url completionHandler:^(NSURL * __nullable location, NSURLResponse * __nullable response, NSError * __nullable error)
```
2. downloadTaskWithURL内部默认已经实现了变下载边写入操作，所以不用开发人员担心内存问题
3. 文件下载后默认保存在tmp文件目录，需要开发人员手动的剪切到合适的沙盒目录
4. 缺点：没有办法监控下载进度

### 使用 NSURLSessionDownloadTask 大文件下载-监听下载进度

1. 创建NSURLSession并设置代理，通过NSURLSessionDownloadTask并以代理的方式来完成大文件的下载
```
//1.创建NSULRSession,设置代理
    self.session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];

    //2.创建task
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"];
    self.downloadTask = [self.session downloadTaskWithURL:url];

    //3.执行task
    [self.downloadTask resume];
```

2. 常用代理方法

```
/*
 1.当接收到下载数据的时候调用,可以在该方法中监听文件下载的进度
 该方法会被调用多次
 totalBytesWritten:已经写入到文件中的数据大小
 totalBytesExpectedToWrite:目前文件的总大小
 bytesWritten:本次下载的文件数据大小
 */
- (void)URLSession:(nonnull NSURLSession *)session downloadTask:(nonnull NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
/*
 2.恢复下载的时候调用该方法
 fileOffset:恢复之后，要从文件的什么地方开发下载
 expectedTotalBytes：该文件数据的总大小
 */
- (void)URLSession:(nonnull NSURLSession *)session downloadTask:(nonnull NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes
/*
 3.下载完成之后调用该方法
 */
- (void)URLSession:(nonnull NSURLSession *)session downloadTask:(nonnull NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(nonnull NSURL *)location
/*
 4.请求完成之后调用
 如果请求失败，那么error有值
 */
- (void)URLSession:(nonnull NSURLSession *)session task:(nonnull NSURLSessionTask *)task didCompleteWithError:(nullable NSError *)error

```

3. 实现断点下载相关代码

```
//如果任务，取消了那么以后就不能恢复了
    //    [self.downloadTask cancel];

    //如果采取这种方式来取消任务，那么该方法会通过resumeData保存当前文件的下载信息
    //只要有了这份信息，以后就可以通过这些信息来恢复下载
    [self.downloadTask cancelByProducingResumeData:^(NSData * __nullable resumeData) {
        self.resumeData = resumeData;
    }];

    -----------
    //继续下载
    //首先通过之前保存的resumeData信息，创建一个下载任务
    self.downloadTask = [self.session downloadTaskWithResumeData:self.resumeData];

     [self.downloadTask resume];
```

4. 计算当前下载进度
```
//获取文件下载进度
    self.progress.progress = 1.0 * totalBytesWritten/totalBytesExpectedToWrite;
```

5. 局限性
```
    如果用户点击暂停之后退出程序，那么需要把恢复下载的数据写一份到沙盒，代码复杂度更
    如果用户在下载中途未保存恢复下载数据即退出程序，则不具备可操作性
```

### 使用 NSURLSessionDataTask 大文件离线断点下载
1. 关于NSOutputStream的使用
```
//1. 创建一个输入流,数据追加到文件的屁股上
    //把数据写入到指定的文件地址，如果当前文件不存在，则会自动创建
    NSOutputStream *stream = [[NSOutputStream alloc]initWithURL:[NSURL fileURLWithPath:[self fullPath]] append:YES];

    //2. 打开流
    [stream open];

    //3. 写入流数据
    [stream write:data.bytes maxLength:data.length];

    //4.当不需要的时候应该关闭流
    [stream close];
```

2. 关于网络请求请求头的设置（可以设置请求下载文件的某一部分
```
//1. 设置请求对象
    //1.1 创建请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"];

    //1.2 创建可变请求对象
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];

    //1.3 拿到当前文件的残留数据大小
    self.currentContentLength = [self FileSize];

    //1.4 告诉服务器从哪个地方开始下载文件数据
    NSString *range = [NSString stringWithFormat:@"bytes=%zd-",self.currentContentLength];
    NSLog(@"%@",range);

    //1.5 设置请求头
    [request setValue:range forHTTPHeaderField:@"Range"];
```

3. NSURLSession对象的释放
```
- (void)dealloc
{
    //在最后的时候应该把session释放，以免造成内存泄露
    //    NSURLSession设置过代理后，需要在最后（比如控制器销毁的时候）调用session的invalidateAndCancel或者resetWithCompletionHandler，才不会有内存泄露
    //    [self.session invalidateAndCancel];
    [self.session resetWithCompletionHandler:^{

        NSLog(@"释放---");
    }];
}
```

### NSURLSession 文件上传
1. 实现文件上传的方法
```
/*
     第一个参数：请求对象
     第二个参数：请求体（要上传的文件数据）
     block回调：
     NSData:响应体
     NSURLResponse：响应头
     NSError：请求的错误信息
     */
    NSURLSessionUploadTask *uploadTask =  [session uploadTaskWithRequest:request fromData:data completionHandler:^(NSData * __nullable data, NSURLResponse * __nullable response, NSError * __nullable error)
```
2. 设置代理，在代理方法中监听文件上传进度
```
/*
 调用该方法上传文件数据
 如果文件数据很大，那么该方法会被调用多次
 参数说明：
     totalBytesSent：已经上传的文件数据的大小
     totalBytesExpectedToSend：文件的总大小
 */
- (void)URLSession:(nonnull NSURLSession *)session task:(nonnull NSURLSessionTask *)task didSendBodyData:(int64_t)bytesSent totalBytesSent:(int64_t)totalBytesSent totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend
{
    NSLog(@"%.2f",1.0 * totalBytesSent/totalBytesExpectedToSend);
}
```

3. 关于NSURLSessionConfiguration相关
```
    作用：可以统一配置NSURLSession,如请求超时等
         创建的方式和使用
```

```
//创建配置的三种方式
+ (NSURLSessionConfiguration *)defaultSessionConfiguration;
+ (NSURLSessionConfiguration *)ephemeralSessionConfiguration;
+ (NSURLSessionConfiguration *)backgroundSessionConfigurationWithIdentifier:(NSString *)identifier NS_AVAILABLE(10_10, 8_0);

//统一配置NSURLSession
- (NSURLSession *)session
{
    if (_session == nil) {

        //创建NSURLSessionConfiguration
        NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];

        //设置请求超时为10秒钟
        config.timeoutIntervalForRequest = 10;

        //在蜂窝网络情况下是否继续请求（上传或下载）
        config.allowsCellularAccess = NO;

        _session = [NSURLSession sessionWithConfiguration:config delegate:self delegateQueue:[NSOperationQueue mainQueue]];
    }
    return _session;
}
```
一直在使用中,可一直没总结,借鉴[这里](http://www.jianshu.com/p/04050398c602).
