---
title: 'Mybatis select返回值为List<Map>并将表字段的两列为K,V'
date: 2020-01-07 23:34:56
tags: []
published: true
hideInList: false
feature: /post-images/mybatis-select-fan-hui-zhi-wei-listlessmapgreaterbing-jiang-biao-zi-duan-de-liang-lie-zuo-wei-keyvalue.jpg
isTop: false
---
0x03：`mybatis`

<!-- more -->

## 场景重现
<hr/>
### 表中有这样的数据

![](https://pjmic.github.io//post-images/1578412769529.jpg)
### 需要实现

![](https://pjmic.github.io//post-images/1578412975303.jpg)

### 思路是重写返回Mybatis select默认映射Map的返回接口，通过源码追踪，发现 “ResultHandler”这个接口;
```java
package org.apache.ibatis.session;

/**
 * @author Clinton Begin
 */
public interface ResultHandler<T> {

  void handleResult(ResultContext<? extends T> resultContext);

}
```

### 通过观察他的实现有两个实现：

- DefaultMapResultHandler :  返回的是 Map

- DefaultResultHandler ：返回的是 List<Object>

#待续。。。。
