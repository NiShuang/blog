---
title: 使用 jar 包运行项目时的资源文件定位问题
comments: true
date: 2018-12-15 14:51:40
tags:
 - java 
 - 实习
categories: java笔记
---
## 背景
把 java 项目打包成 jar 包时，资源文件夹 resources 下的文件会被打包进 jar 包里面。当使用 jar 运行整个项目时， 通过 getResource() 方法获得的资源文件路径会变成 xxx.jar!/xxx 的格式，这种格式的路径，不能通过 new File 的方式找到文件。这里我提供两个思路解决这个问题。
<!-- more --> 

## 使用 getResourceAsStream 
在这种情况下，如果想让 jar 读取到自己的资源文件，可以通过类加载器的 getResourceAsStream() 方法来解决。
```
InputStream is = this.getClass().getResourceAsStream("geolite/GeoLite2-City.mmdb")
```

## 想办法让资源文件出现在 jar 包同级目录下
既然无法读取 jar 包内部的文件，那可以转变思路，使资源文件出现在 jar 包的同级目录下，然后可以获取 jar 包的路径，从而获得资源文件的路径。

由于我的项目是多模块，所以我把资源文件放在项目根目录下，同主模块处在同级。这样打包以后，资源文件不会被打包进 jar 包内，而是和主模块同级。然后通过 Docker 把资源文件移动到 jar 包所在的目录下，这样就可以轻松访问到资源文件了。

Docker 指令如下：

```
RUN mkdir /data_cleaner && mkdir /data_cleaner/lib && mkdir /data_cleaner/resources

COPY resources/ /data_cleaner/resources/

COPY runner/target/lib/* /data_cleaner/lib/

COPY runner/target/*.jar /data_cleaner

CMD java -jar /data_cleaner/data-cleaning-runner.jar
```

需要注意的是，上述方法只适用于 Docker 环境下，所以我们需要区分项目是在 IDE 环境运行，还是 Docker 环境下使用 jar 包运行。可以通过以下代码来判断，同时获取 jar 包的所在目录

```
String filePath = "/resources/geolite/GeoLite2-City.mmdb";
String path;
// 判断 ide 运行还是 jar 运行
File file = new File(this.getClass().getProtectionDomain().getCodeSource().getLocation().getPath());

if (file.isFile()) {	// jar 运行
	// 获取 jar 包所在目录	
    path = System.getProperty("java.class.path");
    int firstIndex = path.lastIndexOf(System.getProperty("path.separator")) + 1;
    int lastIndex = path.lastIndexOf(File.separator);
    path = path.substring(firstIndex, lastIndex);
} else {				// IDE 运行
    path = System.getProperty("user.dir");
}
path += filePath;
System.out.println(path);
File myFile = new File(path);
```




## 参考资料

 - [获取jar包内部的资源文件
](https://blog.csdn.net/luo_jia_wen/article/details/50057191)

---

> 文章标题：[使用 jar 包运行项目时的资源文件定位问题](http://www.cielni.com/2018/12/15/docker-jar-file/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2018/12/15/docker-jar-file/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！