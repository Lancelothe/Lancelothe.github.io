---
title: IntelliJ IDEA使用
date: 2018-11-30 22:10:28
categories:
  - 工具
tags:
---
#### 好用的插件

- **lokbok**
项目中经常使用bean，entity等类，绝大部分数据类类中都需要get、set、toString、equals和hashCode方法，虽然eclipse和idea开发环境下都有自动生成的快捷方式，但自动生成这些代码后，如果bean中的属性一旦有修改、删除或增加时，需要重新生成或删除get/set等方法，给代码维护增加负担。而使用了lombok则不一样，使用了lombok的注解(@Setter,@Getter,@ToString,@@RequiredArgsConstructor,@EqualsAndHashCode或@Data)之后，就不需要编写或生成get/set等方法，很大程度上减少了代码量，而且减少了代码维护的负担。故强烈建议项目中使用lombok，去掉bean中get、set、toString、equals和hashCode等方法的代码。

1. IDEA安装插件lombok
2. 添加lombok的maven的pom.xml依赖：
```Java
  <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.16.10</version>
  </dependency>
```
3. 添加注解
```Java
package com.lombok.demo;


import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

/**
 * Created by zhangzh on 2017/2/8.
 */
@Setter
@Getter
@ToString
@EqualsAndHashCode
public class Student {

    private String name;
    private int age;
    private String male;
    private String studentNo;
}
```
更多注解等你发现，比如：`@Data`, `@Log`


### 参数优化
Mac 下的地址
> vim /Applications/IntelliJ\ IDEA.app/Contents/bin/idea.vmoptions
```
-Xms128m
-Xmx750m
-XX:ReservedCodeCacheSize=240m
-XX:+UseCompressedOops
-Dfile.encoding=UTF-8
-XX:+UseConcMarkSweepGC
-XX:SoftRefLRUPolicyMSPerMB=50
-ea
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djdk.http.auth.tunneling.disabledSchemes=""
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Xverify:none

-XX:ErrorFile=$USER_HOME/java_error_in_idea_%p.log
-XX:HeapDumpPath=$USER_HOME/java_error_in_idea.hprof
```

更新下面文件后重启，不会太卡了
```
-Xms2200m
-Xmx2200m
-XX:ReservedCodeCacheSize=680m
-XX:+UseCompressedOops
-Dfile.encoding=UTF-8
-XX:+UseConcMarkSweepGC
-XX:SoftRefLRUPolicyMSPerMB=50
-ea
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Xverify:none
-XX:ErrorFile=$USER_HOME/java_error_in_idea_%p.log
-XX:HeapDumpPath=$USER_HOME/java_error_in_idea.hprof
-Xbootclasspath/a:../lib/boot.jar
```
或者
```
-Xms4400m
-Xmx4400m
-Xmn1000m
-XX:PermSize=768m
-XX:MaxPermSize=768m
-Xss512K
-XX:ReservedCodeCacheSize=128m
-XX:SurvivorRatio=1
-XX:+UseParNewGC
-XX:+UseConcMarkSweepGC
-XX:+UseCMSCompactAtFullCollection
-XX:+UseCMSInitiatingOccupancyOnly
-XX:CMSInitiatingOccupancyFraction=70
-XX:+CMSParallelRemarkEnabled
-XX:+CMSClassUnloadingEnabled
-XX:CMSFullGCsBeforeCompaction=0
-XX:LargePageSizeInBytes=200M
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Dsun.rmi.dgc.client.gcInterval=10800000
-Dsun.rmi.dgc.server.gcInterval=10800000
-XX:SoftRefLRUPolicyMSPerMB=0
-XX:+DisableExplicitGC
-XX:+PrintClassHistogram
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamp
-XX:+PrintHeapAtGC
-Xloggc:gc.log
```
