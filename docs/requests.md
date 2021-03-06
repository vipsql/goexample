## requests

`requests` 是一个用golang 语言clone python版本的requests库。
golang 自带的net/http功能已经非常完善。它和python里面的urllib系列库一样，功能足够，但是使用起来非常繁琐。

python版本的requests简直就是神器，让很多人非常敬仰。

因此我就动手按照python的requests的思想封装了一个 requests。

动手之前，我想尽量兼容 python的写法。但是golang不支持函数名称小写，
也不支持 参数命名。所以我只能用参数类型的方案来支持动态参数。

## 安装

golang 1.11之前

```
go get -u github.com/asmcos/requests
```

golang 1.11之后，编辑一个go.mod文件

```
module github.com/asmcos/requests
```

!!! 特别说明
    由于requests代码在更新，所有例子都有是当前有效。
    并且部分片段有上下文。请大家阅读时参考原代码的requests_test.go


## 开始使用(带Auth)

``` go
package main

import (
        "github.com/asmcos/requests"
        "fmt"
)

func main (){

        req := requests.Requests()
        resp,_ := req.Get("https://api.github.com/user",requests.Auth{"asmcos","password...."})
        println(resp.Text())
        fmt.Println(resp.R.StatusCode)
        fmt.Println(resp.R.Header["Content-Type"])
}

```

第一个例子为什么是它？ 因为python requests第一个例子是它。。。呵呵

    注意：：： `密码` 和用户名要用github真实用户才能测试。


## 创建请求的方法

### 例子1

极简使用

``` go
package main

import "github.com/asmcos/requests"

func main (){

        resp,_ := requests.Get("http://go.xiulian.net.cn")
        println(resp.Text())
}
```

其实 requests实现都是先用Requests()函数创建一个 request 和 client，
再用req.Get去请求。

requests.Get 是一个封装，对Requests()和req.Get的一个封装。

### 例子2

这个例子是分成2个步骤，来操作，这样的好处是可以通过req来设置各种请求参数。
后面的例子会展示各种设置。

``` go
package main

import "github.com/asmcos/requests"


func main (){

        req := requests.Requests()

        resp,_ := req.Get("http://go.xiulian.net.cn",requests.Header{"Referer":"http://www.jeapedu.com"})

        println(resp.Text())

}
```

## 设置Header

```
req := Requests()

req.Header.Set("accept-encoding", "gzip, deflate, br")
req.Get("http://go.xiulian.net.cn", requests.Header{"Referer": "http://www.jeapedu.com"})
```

!!! 设置头的2种方法
    1. 通过req.Header.Set函数直接设置
    2. 调用req.Get 函数时，在函数参数里填写上

Get 支持动态参数，但是参数前面要加类型标识。

函数里面根据类型判断参数的含义。

其实 函数的参数里关于Header参数是可以多次设置的。

``` go
url1 := "http://go.xiulian.net.cn"
req.Get(url1, requests.Header{"k0": "v0"},requests.Header{"k1":"v1"},requests.Header{"k2":"v2"})

h := requests.Header{
  "k3":"v3",
  "k4":"v4",
  "k5":"v5",
}
req.Get(url1,h)
```

这些都可以。灵活增加头。


## 设置参数


``` go
p := requests.Params{
  "title": "The blog",
  "name":  "file",
  "id":    "12345",
}
resp,_ := Requests().Get("http://www.cpython.org", p)
```

其实参数设置。参数设置也是支持多次的。

类似如下：

``` go
resp,_ := Requests().Get("http://www.cpython.org", p,p1,p2)
```

## POST 的form表单

``` go
data := Datas{
    "comments": "ew",
    "custemail": "a@231.com",
    "custname": "1",
    "custtel": "2",
    "delivery": "12:45",
    "size": "small",
    "topping": "bacon",
  }

resp,_ := req.Post("https://www.httpbin.org/post",data)

fmt.Println(resp.Text())
```

!!!! `注意` Post的form是放在body里面，而get的params 是放在url里面。


## Proxy

目前不支持带密码验证的代理。

``` go
req = Requests()
req.Proxy("http://192.168.1.190:8888")
resp,_ = req.Get("https://www.sina.com.cn")
```

## 设置Cookies

requests 支持自身传递cookies。本质上是把cookies存在client.jar里面。
用户设置的cookies也会随着client.jar来传递。

``` go
req = Requests()

cookie := &http.Cookie{}
cookie.Name   = "anewcookie"
cookie.Value  = "20180825"
cookie.Path   = "/"

req.SetCookie(cookie)

fmt.Println(req.Cookies)
req.Get("https://www.httpbin.org/cookies/set?freeform=1234")
req.Get("https://www.httpbin.org")
req.Get("https://www.httpbin.org/cookies/set?a=33d")
```


！！！ 过程说明
      代码中 首先使用http.Cookie生成一个用户自定义的cooie,
      req.SetCookie 实际上还没有把cookie放在client.jar里面。
      在Get的时候requests会把req.Cookies里面的内容复制到client.jar里面，并且清空req.cookies
      再一次Get的时候，requests都会处理Cookies。

## debug

当设置了Debug = 1，请求的时候会把request和response都打印出来，

包含request的cookie， 返回的cookie没有打印。

``` go
req := Requests()
req.Debug = 1

data := Datas{
    "comments": "ew",
    "custemail": "a@231.com",
    "custname": "1",
    "custtel": "2",
    "delivery": "12:45",
    "size": "small",
    "topping": "bacon",
  }

resp,_ := req.Post("https://www.httpbin.org/post",data)

fmt.Println(resp.Text())
```

### Debug 结果如下

``` json
===========Go RequestDebug ============
POST /post HTTP/1.1
Host: www.httpbin.org
User-Agent: Go-Requests 0.5
Content-Length: 96
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip


===========Go ResponseDebug ============
HTTP/1.1 200 OK
Content-Length: 560
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Type: application/json
Date: Sun, 02 Sep 2018 09:40:32 GMT
Server: gunicorn/19.9.0
Via: 1.1 vegur


{
  "args": {},
  "data": "",
  "files": {},
  "form": {
    "comments": "ew",
    "custemail": "a@231.com",
    "custname": "1",
    "custtel": "2",
    "delivery": "12:45",
    "size": "small",
    "topping": "bacon"
  },
  "headers": {
    "Accept-Encoding": "gzip",
    "Connection": "close",
    "Content-Length": "96",
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "www.httpbin.org",
    "User-Agent": "Go-Requests 0.5"
  },
  "json": null,
  "origin": "219.143.154.50",
  "url": "https://www.httpbin.org/post"
}
```

## Authentication 验证用户名和密码。

``` go
req = Requests()
url1 := "https://www.httpwatch.com/httpgallery/authentication/authenticatedimage/default.aspx?0.45874470316137206"
resp,_ = req.Get(url1,Auth{"httpwatch","foo"})
fmt.Println(resp.httpresp)
```

用户名和密码的参数是加密存放在Header里面，

和params是不同的。

## saveFile

接着上面的例子，如果成功了。返回的数据实际是一个jpeg格式文件。

我们可以直接存文件。

```
resp.SaveFile("auth.jpeg")
```

## JSON

``` go
req = Requests()
req.Header.Set("Content-Type","application/json")
resp,_ = req.Get("https://httpbin.org/json")

var json map[string]interface{}
resp.Json(&json)

for k,v := range json{
  fmt.Println(k,v)
}
```

## 支持GZIP格式

``` go
req = Requests()
req.Debug = 1
resp,_ = req.Get("https://httpbin.org/gzip")

fmt.Println(resp.Text())
```


## 支持传文件POST

``` go
req = Requests()
req.Debug = 1
path, _ := os.Getwd()
path1 := path +  "/../examples/net1.go"
path2 := path +  "/../examples/net2.go"

resp,_ = req.Post("https://www.httpbin.org/post",data,Files{"a":path1,"b":path2})

fmt.Println(resp.Text())

```

header，params，是不分 GET和POST的。


## 返回cookies

其实requests的cookies 一直存在client.jar里面，

所以需要从client.jar读取。

``` go
resp,_ = req.Get("https://www.httpbin.org")
coo := resp.Cookies()
// coo is [] *http.Cookies
println("********cookies*******")
for _, c:= range coo{
  fmt.Println(c.Name,c.Value)
}
```
