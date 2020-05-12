---
title: 5.NSURLConnection使用
date: 2016-05-17 21:39:40
tags:
---

NSURLConnection是apple提供的网络请求类.



* 作用:
      1、负责发送请求，建立客户端和服务器的连接发送数据给服务器
      2、并收集来自服务器的响应数据



<Excerpt in index | 首页摘要>
<!-- more -->
<The rest of contents | 余下全文>

* 步骤:
      1、创建一个NSURL对象，设置请求路径
      2、传入NSURL并创建一个NSURLRequest对象，设置请求头和请求体
      3、使用NSURLConnection发送请求

* 常见类:
      1、NSURL:收纳请求的地址
      2、NSURLRequest:一个NSURLRequest对象就代表一个请求，它包含的信息有一个NSURL对象、请求方法、请求头、请求体等等
      3、NSMutableURLRequest是NSURLRequest的子类

* 发送同步请求:(不建议使用)

```
- (void)sendSynchronousRequest{
  //1、创建一个URL
  //协议头+主机地址+接口名称+?+参数1&参数2&参数3
  NSURL *url = [NSURL URLWithString:@"http://xxxxx/login"];

  //2、创建请求(Request)对象(默认为GET请求)；
  NSURLRequest *requst = [[NSURLRequest alloc]initWithURL:url];

  //3、发送请求
  /*
   第一个参数:请求对象
   第二个参数:响应头
   第三个参数:错误信息
   返回值:NSData类型,响应体信息
   */
  NSError *error = nil;
  NSURLResponse *response = nil;
  //发送同步请求(sendSynchronousRequest)
  NSData *data = [NSURLConnection sendSynchronousRequest:requst returningResponse:&response error:&error];
  //如果没有错误就执行
  if (!error) {
      //打印的服务端返回的信息以及错误信息
      NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
      NSLog(@"%@",error);
  }
}
```

* 发送异步请求:
```
-(void)sendAsynchronousRequest{
        //1、创建一个URL
        NSURL *url = [NSURL URLWithString:@"http://xxxxx/login"];

        //2、创建请求(Request)对象 这里使用的是它的子类NSMutableURLRequest,因为子类才具有设置方法和设置请求体的属性
        NSMutableURLRequest *requst = [[NSMutableURLRequest alloc]initWithURL:url];
        //2.1、设置请求方法
        requst.HTTPMethod = @"POST";
        //2.2、设置请求体,因为传入的为Data数据所有这里需要转换
        requst.HTTPBody = [@"username=xxx&pwd=xxx" dataUsingEncoding:NSUTF8StringEncoding];
        //2.3、设置请求超时时间，如果超过这个时间，请求为失败
        requst.timeoutInterval = 10;

        //3、发送请求

        /*
         第一个参数:请求对象
         第二个参数:队列
         第三个参数:Block回调函数
            response:响应头
            data:响应体信息
            connectionError:错误信息
         */

        //发送异步请求(sendAsynchronousRequest)
        [NSURLConnection sendAsynchronousRequest:requst queue:[[NSOperationQueue alloc]init] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
            NSLog(@"----%@",[NSThread currentThread]);

            //解析数据
            NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
        }];
    }

```

* 代理请求:
```
    1、当接受到服务器响应的时候会调用:response(响应头)
        -(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response;
    2、当接受到服务器返回数据的时候调用(会调用多次)
        - (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data;
    3、当请求失败的时候调用
        - (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error;
    4、当请求结束(成功|失败)的时候调用
        - (void)connectionDidFinishLoading:(NSURLConnection *)connection;
```

具体代码如下 :
```
1、首先实现代理,并定义一个NSData对象初始化,在请求结束的时候查看服务器传来的内容，
    @interface ViewController ()<NSURLConnectionDataDelegate>
    /** 可变的二进制数据 */
    @property (nonatomic, strong) NSMutableData *fileData;
    @end

    /*
    懒加载
    */
    -(NSMutableData *)fileData{
        if (!_fileData) {
            _fileData = [[NSMutableData alloc]init];
        }
        return _fileData;
    }
2、实现代理中的四个方法
    //1.当接受到服务器响应的时候会调用:response(响应头)
    -(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
    {
        NSLog(@"接受到相应");
    }

    //2.当接受到服务器返回数据的时候调用(会调用多次)
    -(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
    {
        //
        NSLog(@"接受到数据");
        //拼接数据
        [self.fileData appendData:data];
    }

    //3.当请求失败的时候调用
    -(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
    {
        NSLog(@"请求失败");
    }

    //4.当请求结束(成功|失败)的时候调用
    -(void)connectionDidFinishLoading:(NSURLConnection *)connection
    {
        NSLog(@"请求结束");

        //解析数据
        NSLog(@"%@",[[NSString alloc]initWithData:self.fileData encoding:NSUTF8StringEncoding]);
    }
3、最后编写点击时调用的方法
    -(void)sendRequestWithDelegate{
        //1.确定请求路径
        NSURL *url = [NSURL URLWithString:@"http://xxxxx/login?username=xxx&pwd=xxx"];
        //2.创建请求对象
        NSURLRequest *request = [NSURLRequest requestWithURL:url];

        //3、代理请求
        /*
         第一个参数:请求对象
         第二个参数:谁成为代理
         第三个参数:startImmediately :是否立即开始发送网络请求
         */
        NSURLConnection *connect = [[NSURLConnection alloc]initWithRequest:request delegate:self startImmediately:NO];
        //[connect cancel]; 取消
        [connect start];
    }

```
