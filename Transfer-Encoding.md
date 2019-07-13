# HTTP服务器响应数据引发的思考

## 来源
最近在学习网络编程相关的信息，突发奇想自己手编http post请求数据去请求公司的数据

```java
try {
    Socket socket = new Socket();
    socket.connect(new InetSocketAddress("...", 80)); // 这里是host, 保密

    BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));

    writer.write("POST / HTTP/1.1\r\n");
    writer.write("Host: ...\r\n");  // 保密
    writer.write("User-Agent: Socket\r\n");
    writer.write("Accept: application/json\r\n");
    writer.write("Connection: close\r\n");
    writer.write("Content-Type: application/x-www-form-urlencoded\r\n");

    String body = "action=adv_index";
    byte[] bytes = body.getBytes();

    writer.write("Content-Length: " + bytes.length);
    writer.write("\r\n");
    writer.write("\r\n");
    writer.write(body);

    writer.flush();
    BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }

    writer.close();
    reader.close();
    socket.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

下面是得到的响应数据
```http
HTTP/1.1 200 OK
Server: nginx
Date: Sat, 27 Apr 2019 06:32:53 GMT
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: close
X-Powered-By: PHP/5.5.38
Access-Control-Allow-Origin: *

70c
//... 这里是实体数据
0

```

一开始我觉得很奇怪，为什么会在entity body前后添加一个字符串，同时我又去请求了其他的接口，前面的字符串会改变，后面的字符串永远是一个0

## 发现
在仔细观察了这个响应数据之后，我发现其中有两个点，1. 这个post响应头中没有`Content-Length`，2. 这个字符串是`entity body`的16进制长度。所以在搜索了一些资料，同时又回顾了一下http协议的内容后，发现是`Transfer-Encoding`这个header在作用。

## http请求的长度

网络请求一般会有两种表示内容结束的方法，这样接受数据的一段就不用一直盲目的等待：
1. 一开始就标志内容的长度，http一般就使用这种方式，在header中使用`Content-Length`来标识
2. 在内容前后添加开始和结束的表示，如当传输文件时，我们会在其数据前后添加`------`的字符串表示开始和结束

第一种方法通常是我们在传输数据之前就知道其内容的长度，而第二种方法，是我们在传输数据之前不知道内容长度时采用的。
如，当我们使用`post`上传文件时，设置`Content-Type`为`multipart/form-data`会在其后添加一个`boundary`属性，用来标志`entity body`的开始、参数分割、结束。

而`Transfer-Encoding`则是采用另外的方法，当我们一开始不知道内容的长度时，我们自然不能使用`Content-Length`。这时，我们就需要另外一种方式知道内容的边界。`Transfer-Encoding`提供的就是一种分块方法，我们不知道内容总长度，但是我们知道每一次发送的内容的长度，那么在发送内容前，我们就在内容前面添加一个长度。

### 具体方法
1. 在头部添加`Transfer-Encoding: chunked`
2. 每个分块包含一个十六进制的长度值，这个值独占一行，长度不包含其后面的`CRLF(/r/n)`，也不包括后面分块内容的`CRLF`
3. 最后一个分块一定是`0`, 同时也有分块，但是分块没有内容，只是`CRLF`

## 重点

所以`Transfer-Encoding`和`Content-Type`是不共存的。


## 参考

[分块编码（Transfer-Encoding: chunked）](https://www.cnblogs.com/xuehaoyue/p/6639029.html)

[HTTP协议中Content-Length的详细解读。](https://blog.csdn.net/xxdddail/article/details/20995411)

[
multipart/form-data请求与文件上传的细节](https://blog.csdn.net/zshake/article/details/77985757)

[Multipart/form-data POST文件上传详解](https://blog.csdn.net/xiaojianpitt/article/details/6856536)