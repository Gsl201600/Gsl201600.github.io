---
title: iOS和Unity交互之参数传递
date: 2019-03-07 14:23:00
tags: 代码库
---

**1. 调用方法一**

* Unity调方法传参,有返回值
```
// Unity代码
[DllImport("__Internal")]
// 给iOS传string参数,有返回值,返回值通过iOS的return方法返回给Unity
private static extern string getIPv6(string mHost, string mPort)
```
```
// iOS代码
extern "C" const char * getIPv6(const char *mHost, const char *mPort)
{
    // strdup(const char *__s1) 复制mHost字符串,通过Malloc()进行空间分配 
    // return strdup(mHost);
    return makeStringCopy(mHost);
}

char* makeStringCopy(const char* string)
{
    if (NULL == string) {
        return NULL;
    }
    char* res = (char*)malloc(strlen(string)+1);
    strcpy(res, string);
    return res;
}
```

* 如果Unity传参为string类型,不执行strdup()方法而直接使用return方法,导致mHost没有分配内存空间而报错
* 这里的const char* 会被C#自动转换成string因为在.m文件中使用了内存申请，该段内存自然是处在堆内存中，这样转成string符合c#的内存管理机制，我们不用担心它的释放问题
* 如果Unity传参为int等基础数据类型,可以直接使用return方法
* 调用DllImport(“”)方法,需要引入命名空间:`using System.Runtime.InteropServices`

**2. 调用方法二**

* Unity调方法传参,无返回值
```
// Unity代码
// 传数据给iOS
[DllImport("__Internal")]
// 给iOS传string参数,无返回值,返回值通过iOS的UnitySendMessage方法返回给Unity
private static extern void setDate(string date);

// 接收iOS的数据
public void GetDate(string date)
{
}
```
```
// iOS代码
extern "C" void setDate(const char *date)
{
    /**
    发送数据给Unity
    @param obj 模型名
    @param method Unity接收iOS数据的方法名
    @param msg 传给Unity的数据
    UnitySendMessage(const char* obj, const char* method, const char* msg);
    */
    UnitySendMessage("PublicGameObject", "GetDate", date);
}
```
