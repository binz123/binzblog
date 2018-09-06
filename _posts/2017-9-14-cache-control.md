---
layout: post
title: 常用缓存机制优先与缓存解决方案
categories: cache
description:  缓存详解以及解决方案
keywords: 缓存 cache
---
## 缓存
### 强缓存：
1.**Expires**   
代表过期日期  
2.**Cache-control**  
代表多少时间以后过期（单位毫秒）
### 协商缓存
1.**Last-Modified**  
上一次修改时间  
2.**Etag**  
被请求变量实体值

### 缓存优先级别
#### 强缓存与协商缓存同时存在
强缓存有效期内：强缓存 > 协商缓存
强缓存有效期外：强缓存 < 协商缓存
#### 强缓存对比
Cache-control > Expires
### 协商缓存
ETag > Last-Modified

## 缓存的基本机制
之前看到很多对缓存的理解其实都是有一定误区对的,最后决定自己总结一下，主要根据强缓存跟协商缓存二者在请求方面的区别。

### 强缓存
浏览器缓存可用

![cache3](http://binzhome.com/assets/images/cachecontrol/strongcachesuccess.png)

浏览器缓存不可用

![cache1](http://binzhome.com/assets/images/cachecontrol/strongcachefailed.png)

### 协商缓存
浏览器缓存有效  
![cache3](http://binzhome.com/assets/images/cachecontrol/weakcachesuccess.png)  
浏览器缓存无效  
![cache2](http://binzhome.com/assets/images/cachecontrol/weakcachefailed.png)

**总结：**可以看出来强缓存与协商缓存机制的区别在于是否需要通过与服务器进行协商确认来区分，强缓存判断该缓存是否有效可以直接通过判断浏览器缓存当前是否在可用期内，若可用则直接返回对应数据。而协商缓存获取的只是一个标识，该标识此时是否有效，还需要与服务器进行请求判断，服务器最后根据标识有效性返回对应的信息（有的人会认为若有效，不是也已经发起了一个请求进行了一次通信了么，这有什么意义呢？其实如果标识有效，服务器返回的只是一个非常小响应头，并不包含响应的主体部分，所以相比来说是大大减少了消耗）


### 缓存导致问题
&emsp;&emsp;之前刚到做奔驰项目的啥时候，缓存问题一直无法解决，主要现象是当我们改完代码后发版，第二天用户并没有如期的拿到修改后代码，客户端展示还是原来的代码。  
&emsp;&emsp;后来用了各种缓存机制还是存在很大的问题，最后还是用版本号Md5解决的，至于如何加版本号，网上一搜一大堆，各种打包工具也提供了很好的支持，这里就不再赘述了。

