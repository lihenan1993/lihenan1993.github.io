---
title: java maven 搭建gRPC环境
category: Java
---



## java maven 搭建gRPC环境

### 依赖

```xml
<dependencyManagement>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.google.protobuf/protobuf-java -->
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>3.12.4</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

有时候需要升级版本来解决代码报错问题

例如，我遇到的报错：

```
1. UnusedPrivateParameter missing
2. @java.lang.Override 父类中不存在可复写方法
```

### 插件

- 将.proto文件生成java需要的类

  ```xml
  <plugin>
      <groupId>org.xolstice.maven.plugins</groupId>
      <artifactId>protobuf-maven-plugin</artifactId>
      <version>0.5.1</version>
      <configuration>
          <protocArtifact>com.google.protobuf:protoc:3.5.1-1:exe:${os.detected.classifier}</protocArtifact>
          <pluginId>grpc-java</pluginId>
          <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.17.1:exe:${os.detected.classifier}</pluginArtifact>
      </configuration>
      <executions>
          <execution>
              <goals>
                  <goal>compile</goal>
                  <goal>compile-custom</goal>
              </goals>
          </execution>
      </executions>
  </plugin>
  ```

  

- 设置classifier， Windows在 **pom.xml** 的 **properties** 中新增：

  ```xml
  <os.detected.classifier>windows-x86_64</os.detected.classifier>
  ```

  

> ```
> ${os.detected.classifier} 设置方法参考
> https://github.com/trustin/os-maven-plugin
> ```



### 插件使用

1. 在IDE的maven面板上分别点击下面两个任务。

   - **protobuf:compile**：protobuf序列化相关的文件，也就相当于是数据交换时的java bean。

   - **protobuf:compile-custom**：生成grpc相关的，主要用于与服务端通信的。

![1134128-20181211222247995-2015401751](/assets/img/1134128-20181211222247995-2015401751-1607524337623.png)

2. 自动生成的代码在**target/generated-sources/protobuf**里