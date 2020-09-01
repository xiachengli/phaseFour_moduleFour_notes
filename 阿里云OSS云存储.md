#### 阿里云OSS云存储简介

Object Storage Service，简称OSS对象存储服务，是阿里云提供的海量、安全、低成本、高可靠的云存储服务

#### OSS存储基本概念

##### Bucket存储空间

存储空间是用户用于存储对象的容器。其具有各种配置属性，如地域、访问权限、存储类型等

特点：

- 同一个存储空间的内部是扁平的，没有文件系统目录的概念，所有的对象都直接隶属于存储空间
- 每个用户可以拥有多个存储空间
- 存储空间的名称在OSS范围内必须是全局唯一的，且一旦创建后无法修改
- 存储空间内部的对象数目没有限制

命名规范：

- 只能包括小写字母、数字和短横线-
- 必须以小写字母或者数字开头和结尾
- 长度必须在3-63个字节之间

##### Object对象/文件

存储的基本单元，由元信息，用户数据和文件名组成，通过存储空间内部唯一的key来标识。对象元信息是一组键值对，表示对象的属性，如最后修改时间、大小等，用户可自定义元信息

 对象的生命周期是从上传成功到删除。在整个生命周期内，可追加上传的对象可以继续写入数据，其他上传方式上传的对象无法修改（可通过上传同名对象进行覆盖）

命名规范：

- 使用UTF-8编码
- 长度在1-1023个字节之间
- 不能以斜线/ 或 \开头

##### Region地域

表示OSS的数据中心所在物理位置

地域是在创建存储空间的时候指定的，一旦指定后就不允许更改。该存储空间下的所有对象都存储在对应的数据中心

##### EndPoint访问域名

表示OSS对外服务的访问域名

##### AccessKey访问密钥

用于身份验证的AccessKeyId和AccessKeySecret，OSS 通过使用 AccessKeyId 和 AccessKeySecret 对称加密的方法来验证某个请求的发送者身份

AccessKeyId：用户标识

AccessKeySecret：密钥

AccessKey的来源有以下三种：

- Bucket 的拥有者申请的 AccessKey
- 被 Bucket 的拥有者通过 RAM 授权给第三方请求者的 AccessKey
- 被 Bucket 的拥有者通过 STS 授权给第三方请求者的 AccessKey

##### Service

OSS提供给用户的虚拟存储空间，Service中可以包含多个Bucket

#### OSS功能详解

##### 基本功能

- 创建存储空间
- 上传文件
- 下载文件
- 删除文件
- 删除存储空间

##### Object外链地址的构成规则

```
http:// <你的bucket名字>.<数据库中心服务域名>/<你的object名字>
```

![image-20200827172254612](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200827172254612.png)



##### OSS防盗链

为了防止用户在OSS上的数据被人盗链，OSS支持基于HTTP header中表头字段referer的防盗链方法

通过 OSS 的控制台--权限管理--防盗链,可以对一个 bucket 设置 referer 字段的白名单和是否允许 referer 字段为空的请求访问。例如， 对于一个名为 ossexample 的 bucket，设置其 referer 白名单为 http://www.aliyun.com。则所有 referer 为 http://ww w.aliyun.com 的请求才能访问 oss-example 这个 bucket 中的 Object

##### 自定义域名绑定

OSS支持用户将自定义的域名绑定在指定的Bucket

##### 访问日志记录

OSS为用户提供自动保存访问日志记录功能。Bucket的拥有者可以通过OSS控制台（http://oss.aliyu n.com）日志管理，为其所拥有的bucket开启访问日志记录功能。当一个bucket（源Bucket，Source Bucket）开启访问日志记录功能后，OSS自动将访问这个bucket的请求日志，以小时为单位，按照固定 的命名规则，生成一个Object写入用户指定的bucket（目标Bucket，Target Bucket）

存储访问日志记录的object命名规则：

```
<TargetPrefix><SourceBucket>-YYYY-mm-DD-HH-MM-SS-UniqueString

命名规则中，TargetPreﬁx由用户指定；YYYY, mm, DD, HH, MM和SS分别是该Object被创建时的阿拉伯数 字的年，月，日，小时，分钟和秒（注意位数）；UniqueString为OSS系统生成的字符串。一个实际的 用于存储OSS访问日志的Object名称例子如下

```

#### OSS云存储的权限控制

##### 权限控制方式

- ACL

  Bucket ACL是 Bucket 级别的权限访问控制。目前有三种访问权限：public-read-write，public-read 和 private

  | 权限值            | 名称     | 限制                                                         |
  | ----------------- | -------- | ------------------------------------------------------------ |
  | public-read-write | 公共读写 | 任何人（包括匿名访问）都可以对该 Bucket 中的 Object 进行读/写/删 除操作；所有这些操作产生的费用由该 Bucket 的 Owner 承担，请慎 用该权限 |
  | public-read       | 公共读   | 只有该 Bucket 的 Owner 或者授权对象可以对存放在其中的 Object 进 行写/删除操作；任何人（包括匿名访问）可以对 Object 进行读操作 |
  | private           | 私有读写 | 只有该 Bucket 的 Owner 或者授权对象可以对存放在其中的 Object 进 行读/写/删除操作；其他人在未经授权的情况下无法访问该 Bucket 内 的 Object |

  Object ACL是Object 级别的权限访问控制。目前有四种访问权限：private、public-read、publicread-write、default。PutObjectACL                           操作通过 Put 请求中的 x-oss-object-acl 头 来设置，这个操作只有 Bucket Owner 有权限执行

  | 权限              | 名称           | 限制                                                         |
  | ----------------- | -------------- | ------------------------------------------------------------ |
  | public-read-write | 公共读写       | 表明某个 Object 是公共读写资源，即所有用户拥有对该 Object 的读写权限 |
  | public-read       | 公有读，私有写 | 表明某个 Object 是公共读资源，即非 Object Owner 只有该 Object 的读权限，而 Object Owner 拥有该 Object                                       的读写权限 |
  | private           | 私有读写       | 表明某个 Object 是私有资源，即只有该 Object 的 Owner 拥有 该 Object 的读写权限，其他的用户没有权限操作该 Object |
  | default           | 默认权限       | 表明某个 Object 是遵循 Bucket 读写权限的资源，即 Bucket 是 什么权限，Object 就是什么权限 |

- RAM Policy（Resource Access Management）

  RAM（Resource Access Management）是阿里云提供的资源访问控制服务，RAM Policy是基于用户 的授权策略。使用RAM您可以创建、管理RAM用户，并可以控制这些RAM用户对资源的操作权限

  https://help.aliyun.com/document_detail/102600.html?spm=a2c4g.11186623.6.693.143258f6f33HST 

- Bucket Policy

  Bucket Policy是基于资源的授权策略。相比于RAM Policy，Bucket Policy支持在控制台直接进行图形 化配置操作，并且Bucket拥有者直接可以进行访问授权

#### SpringBoot整合OSS

- 引入maven

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>com.xcl</groupId>
      <artifactId>OSS_springBoot</artifactId>
      <version>1.0-SNAPSHOT</version>
  
      <!--spring boot的支持-->
      <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.1.0.RELEASE</version>
      </parent>
  
      <dependencies>
          <!--springboot 测试支持-->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
          </dependency>
          <dependency>
              <groupId>com.aliyun.oss</groupId>
              <artifactId>aliyun-sdk-oss</artifactId>
              <version>2.8.3</version>
          </dependency>
          <dependency>
              <groupId>org.apache.commons</groupId>
              <artifactId>commons-lang3</artifactId>
              <version>3.7</version>
          </dependency>
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <version>1.18.4</version>
          </dependency>
          <dependency>
              <groupId>joda-time</groupId>
              <artifactId>joda-time</artifactId>
              <version>2.9.9</version>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  
      </dependencies>
  
  </project>
  ```

- 配置

  application.properties

  ```
  server.port = 8999 
  spring.servlet.multipart.maxFileSize = 10MB
  spring.servlet.multipart.maxRequestSize=100MB
  ```

  aliyun.properties

  ```
  aliyun.endpoint=http://oss-cn-beijing.aliyuncs.com
  aliyun.accessKeyId=xxx
  aliyun.accessKeySecret=xxx
  aliyun.bucketName=weiyi33
  aliyun.urlPrefix=https://weiyi33.oss-cn-beijing.aliyuncs.com/
  ```

- 编写代码

  aliyun配置类

  ```java
  package com.xcl.config;
  
  import com.aliyun.oss.OSSClient;
  import lombok.Data;
  import org.springframework.boot.context.properties.ConfigurationProperties;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.PropertySource;
  
  /**
   * 阿里云配置
   */
  @Configuration
  @PropertySource("classpath:aliyun.properties")
  @ConfigurationProperties(prefix = "aliyun")
  public class AliyunConfig {
      private String endpoint;
      private String accessKeyId;
      private String accessKeySecret;
      private String bucketName;
      private String urlPrefix;
  
      public String getEndpoint() {
          return endpoint;
      }
  
      public void setEndpoint(String endpoint) {
          this.endpoint = endpoint;
      }
  
      public String getAccessKeyId() {
          return accessKeyId;
      }
  
      public void setAccessKeyId(String accessKeyId) {
          this.accessKeyId = accessKeyId;
      }
  
      public String getAccessKeySecret() {
          return accessKeySecret;
      }
  
      public void setAccessKeySecret(String accessKeySecret) {
          this.accessKeySecret = accessKeySecret;
      }
  
      public String getBucketName() {
          return bucketName;
      }
  
      public void setBucketName(String bucketName) {
          this.bucketName = bucketName;
      }
  
      public String getUrlPrefix() {
          return urlPrefix;
      }
  
      public void setUrlPrefix(String urlPrefix) {
          this.urlPrefix = urlPrefix;
      }
  
      // 生成OSSClient
      @Bean
      public OSSClient ossClient(){
          return   new OSSClient(endpoint,accessKeyId,accessKeySecret);
      }
  }
  
  ```

  service类

  ```java
  package com.xcl.service;
  
  import com.aliyun.oss.OSSClient;
  import com.aliyun.oss.model.GetObjectRequest;
  import com.aliyun.oss.model.OSSObject;
  import com.xcl.config.AliyunConfig;
  import org.apache.commons.lang3.StringUtils;
  import org.joda.time.DateTime;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.stereotype.Service;
  import org.springframework.web.multipart.MultipartFile;
  import com.xcl.pojo.UploadResult;
  
  import java.io.*;
  import java.util.UUID;
  
  @Service
  public class FileUploadService {
  
      @Autowired
      private AliyunConfig aliyunConfig;
  
      @Autowired
      private OSSClient ossClient;
  
      // 允许上传的格式
      private static final String[] IMAGE_TYPE = new String[]{ ".jpg",
              ".jpeg",".png"};
  
      private static final Long FILE_LENGTH = 5 * 1024 * 1024L;
  
      public UploadResult  upload(MultipartFile multipartFile){
          // 校验图片格式
          boolean  isLegal = false;
          for (String type:IMAGE_TYPE){
              if(StringUtils.endsWithIgnoreCase(multipartFile.getOriginalFilename(),type)){
                  isLegal = true;
                  break;
              }
          }
          UploadResult upLoadResult = new UploadResult();
          if (!isLegal){
              upLoadResult.setStatus("error");
              return  upLoadResult;
          }
  
          //限制大小5M
          long size = multipartFile.getSize();
          if (FILE_LENGTH.compareTo(size) < 0) {
              upLoadResult.setStatus("大小超过5M");
              return upLoadResult;
          }
          String fileName = multipartFile.getOriginalFilename();
          String filePath = getFilePath(fileName);
          try {
              //上传
              ossClient.putObject(aliyunConfig.getBucketName(),filePath,new ByteArrayInputStream(multipartFile.getBytes()));
          } catch (IOException e) {
              e.printStackTrace();
              // 上传失败
              upLoadResult.setStatus("error");
              return  upLoadResult;
          }
          upLoadResult.setStatus("done");
          upLoadResult.setName(aliyunConfig.getUrlPrefix()+filePath);
          upLoadResult.setUid(filePath);
          return  upLoadResult;
      }
      // 生成不重复的文件路径和文件名
      private String getFilePath(String sourceFileName) {
          DateTime dateTime = new DateTime();
          return "images/" + dateTime.toString("yyyy")
                  + "/" + dateTime.toString("MM") + "/"
                  + dateTime.toString("dd") + "/" + UUID.randomUUID().toString() + "." +
                  StringUtils.substringAfterLast(sourceFileName, ".");
      }
  
      /**
       * 下载
       * @param objectName
       * @return
       */
      public UploadResult download(OutputStream os, String objectName) throws IOException {
          UploadResult upLoadResult = new UploadResult();
          OSSObject result = ossClient.getObject(aliyunConfig.getBucketName(), objectName);
  
          BufferedInputStream in = new BufferedInputStream(result.getObjectContent());
          BufferedOutputStream out = new BufferedOutputStream(os);
          byte[] buffer = new byte[1024];
          int len = 0;
          while((len = in.read(buffer)) != -1) {
              out.write(buffer,0,len);
          }
          if (out != null) {
              out.flush();
              out.close();
          }
          if (in != null) {
              in.close();
          }
  
          upLoadResult.setStatus("done");
          upLoadResult.setName(aliyunConfig.getUrlPrefix()+result.getKey());
          upLoadResult.setUid(result.toString());
          return upLoadResult;
      }
  
      public UploadResult delete(String objectName) {
          // 根据BucketName,objectName删除文件
          ossClient.deleteObject(aliyunConfig.getBucketName(), objectName);
          UploadResult uploadResult = new UploadResult();
          uploadResult.setName(objectName);
          uploadResult.setStatus("removed");
          uploadResult.setResponse("success");
  
          return uploadResult;
      }
  }
  
  ```

  消息类

  ```java
  package com.xcl.pojo;
  
  import lombok.Data;
  
  public class UploadResult {
      // 文件唯一标识
      private String uid;
      // 文件名
      private String name;
      // 状态有：uploading done error removed
      private String status;
      // 服务端响应内容，如：'{"status": "success"}'
      private String response;
  
      public String getUid() {
          return uid;
      }
  
      public void setUid(String uid) {
          this.uid = uid;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getStatus() {
          return status;
      }
  
      public void setStatus(String status) {
          this.status = status;
      }
  
      public String getResponse() {
          return response;
      }
  
      public void setResponse(String response) {
          this.response = response;
      }
  }
  
  ```

  



