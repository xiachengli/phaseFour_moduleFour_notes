#### FastDFS基础

##### 什么是FastDFS

- FastDFS是C语言编写的一款开源的轻量级分布式文件系统
- 功能包括：文件存储、文件同步、文件上传/下载等
- 解决了文件大容量存储和高性能访问问题
- 适合以文件为载体的在线服务，如图片、视频、文档服务等
- 基于文件的key value存储系统，key为文件ID，value为文件内容

##### FastDFS特性

- 分组存储，简单灵活；对等结构，不存在单点
- 文件不分块存储，上传的文件和OS文件系统中的文件一一对应
- 文件ID由FastDFS生成，作为文件访问凭证
- 提供了nginx扩展模块
- 支持海量小文件存储
- 一台storage支持多块磁盘，支持单盘数据恢复
- 支持相同内容的文件只保存一份，节约磁盘空间
- 存储服务器上可以保存文件属性
- 支持多线程上传和下载文件，支持断点续传

##### FastDFS的构成

![image-20200827075551166](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200827075551166.png)

- client客户端

  请求的发起方。通过提供的API，使用TCP/IP协议与跟踪服务器或存储节点进行交互

- tracker跟踪器

  负载均衡和服务调度。文件上传时跟踪器根据配置的策略找到可用的存储节点提供文件上传服务

- storage存储节点

  文件存储。利用操作系统的文件系统来管理文件

  存储服务器的上下线不会影响线上服务

##### 安装

1. 安装编译环境

   ```
   yum install git gcc gcc-c++ make automake  vim   wget  libevent -y 
   ```

2. 安装libfastcommon基础库

   ```
   mkdir /root/fastdfs
   cd /root/fastdfs
   git clone https://github.com/happyfish100/libfastcommon.git --depth 1 
   cd libfastcommon/ 
   ./make.sh && ./make.sh install 
   ```

3. 安装FastDFS

   ```
   cd  /root/fastdfs 
   wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz 
   tar -zxvf V5.11.tar.gz 
   cd fastdfs-5.11 
   ./make.sh && ./make.sh install 
   
   #配置文件准备 
   cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf 
   cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf 
   cp /etc/fdfs/client.conf.sample  /etc/fdfs/client.conf   
   cp /root/fastdfs/fastdfs-5.11/conf/http.conf  /etc/fdfs
   cp /root/fastdfs/fastdfs-5.11/conf/mime.types  /etc/fdfs
   ```

   ```
   vim /etc/fdfs/tracker.conf 
   #需要修改的内容如下 
   port=22122   
   base_path=/home/fastdfs 
   ```

   ```
   vim /etc/fdfs/storage.conf 
   #需要修改的内容如下 
   port=23000   
   base_path=/home/fastdfs  
   # 数据和日志文件存储根目录
   store_path0=/home/fastdfs  
   # 第一个存储目录
   tracker_server=192.168.211.136:22122   
   # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致) 
   http.server_port=8888
   ```

4. 启动

   ```
   mkdir  /home/fastdfs  -p 
   /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start 
   /usr/bin/fdfs_storaged /etc/fdfs/storage.conf start 
   
   查看所有运行的端口 
   netstat -ntlp
   
   ```

5. 测试上传

   ```
   vim /etc/fdfs/client.conf 
   #需要修改的内容如下 
   base_path=/home/fastdfs 
   #tracker服务器IP和端口
   tracker_server=192.168.211.136:22122   
   #保存后测试,返回ID表示成功 
   /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /root/fastdfs/1.png 
   ```

6. 安装fastdfs-nginx-module

   ![image-20200827080947467](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200827080947467.png)

   ```
   cd  /root/fastdfs 
   wget https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.20.tar.gz 解压
   tar -xvf V1.20.tar.gz cd fastdfs-nginx-module-1.20/src
   
   vim config 
   修改第5 行 和 15 行 修改成 
   ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/" CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
   
   ```

   ```
   cp mod_fastdfs.conf /etc/fdfs/
   ```

   ```
   vim /etc/fdfs/mod_fastdfs.conf 
   #需要修改的内容如下 
   tracker_server=192.168.211.136:22122   
   url_have_group_name=true 
   store_path0=/home/fastdfs
   
   ```

   ```
   nginx默认需要的目录
   mkdir -p /var/temp/nginx/client
   ```

7. 安装nginx

   ```
   cd  /root/fastdfs 
   wget http://nginx.org/download/nginx-1.15.6.tar.gz tar -zxvf nginx-1.15.6.tar.gz cd nginx-1.15.6/
   
   yum -y install pcre-devel openssl openssl-devel 
   # 添加fastdfs-nginx-module模块 
   ./configure  --add-module=/root/fastdfs/fastdfs-nginx-module-1.20/src 
   
   编译安装 
   make && make install 
   查看模块是否安装上 
   /usr/local/nginx/sbin/nginx -V
   
   
   vim /usr/local/nginx/conf/nginx.conf
   #添加如下配置 
   server {        
   	listen       8888;        
   	server_name  localhost;       
       location ~/group[0-9]/ {           
       	ngx_fastdfs_module;        
       } 
   }
   
   启动
   /usr/local/nginx/sbin/nginx
   ```

8. 测试下载

   ```
   关闭防火墙
   systemctl stop firewalld  
   上传时返回的id group1/M00/00/00/wKjTiF7h5EWASb5aAACGZa9JdFo
   http://192.168.211.136:8888/group1/M00/00/00/wKjTiF7h5EWASb5aAACGZa9JdFo611.png
   
   ```

##### java访问FastDFS

- 引maven坐标

  ```xml
  <!--fastdfs的java客户端--> 
  <dependency>    
  	<groupId>cn.bestwu</groupId>   
      <artifactId>fastdfs-client-java</artifactId>   
      <version>1.27</version> 
  </dependency>
  
  ```

- 设置配置文件

  ```properties
  #fastdfs-client.properties 
  fastdfs.connect_timeout_in_seconds = 5 
  fastdfs.network_timeout_in_seconds = 30 
  fastdfs.charset = UTF-8 
  fastdfs.tracker_servers = 192.168.211.136:22122
  ```

- 测试

  ```java
  @Test 
  public void testUpload() {    
  	try {        
  		//加载配置文件        
  		ClientGlobal.initByProperties("fastdfs-client.properties");        
  		//创建tracker客户端        
  		TrackerClient tc = new TrackerClient();       
          //根据tracker客户端创建连接  获取到跟踪服务器对象        
          TrackerServer ts = tc.getConnection();        
          StorageServer ss = null;        
          //定义storage客户端        
          StorageClient1 client = new StorageClient1(ts, ss);       
          //文件元信息       
          NameValuePair[] list = new NameValuePair[1];        
          list[0] = new NameValuePair("fileName", "1.png");        
          // 上传，返回fileId        
          String fileId = client.upload_file1("****.png", "png", list); 
          System.out.println(fileId);    
          
      } catch (Exception e) {       
     		 e.printStackTrace();   
      } 
  }
  
  
  
  @Test 
  public void testDownload() {    
      try {         
          //加载配置文件         
          ClientGlobal.initByProperties("config/fastdfs-client.properties");        
          // 创建tracker客户端        
          TrackerClient tc = new TrackerClient();        
          //  根据tracker客户端创建连接       
          TrackerServer ts = tc.getConnection();       
          StorageServer ss = null;
  
          //  定义storage客户端        
          StorageClient1 client = new StorageClient1(ts, ss);        
          // 下载        
          byte[] bs = client.download_file1           ("group1/M00/00/00/****.png");        
          FileOutputStream fos = new FileOutputStream(new File("xxxx.png"));        fos.write(bs);       
          fos.close();    
      } catch (Exception e) {      
          e.printStackTrace();  
      } 
  }
  
  ```

#### FastDFS系统架构和功能原理

##### 架构详解

![image-20200827083554320](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200827083554320.png)

- storage server：存储服务器（又称存储节点或数据服务器），文件和文件属性（meta data）都保存 到存储服务器上。Storage server直接利用OS的文件系统调用管理文件
- tracker server：跟踪服务器，主要做调度工作，起负载均衡的作用。在内存中记录集群中所有存储组 和存储服务器的状态信息，是客户端和数据服务器交互的枢纽。因为不记录文件索引信息，所以占用的 内存量很少
- client：客户端，作为业务请求的发起方，通过专有接口，使用TCP/IP协议与跟踪器服务器或存储节点 进行数据交互。FastDFS向使用者提供基本文件访问接口，比如upload、download、append、delete 等，以客户端库的方式提供给用户使用

##### 设计理念

###### 轻量级

- FastDFS 服务端只有两个角色： Tracker server 和 Storage server 
  - Tracker server 在内存中记录 分组 和 Storage server 的状态等信息，不记录文件索引信息，占用 的内存量很少
  -  Storage server 直接利用 OS 的文件系统存储文件； Storage server 可以根据文件ID 直接定位到文件，不需要存储文件索引信息；
- 代码量较小

###### 分组存储

- FastDFS 采用了 分组存储 方式。集群由一个或多个组构成，集群存储总容量为集群中所有组的存储容 量之和
- 同组内的多台 Storage server 之间是对等的互备关系
- 文件上传、下载、删除等操作可以在组内任意一台 Storage server 上进行
- 短板效应，一个组的存储容量为该组内存储服务器容量最小的那个
- 增加服务器来扩充服务能力（纵向扩容）；增加组来扩充存储容量（横向扩容）

###### 对等结构

Tracker server 之间是对等关系，组内的 Storage server 之间也是对等关系

###### 文件HTTP访问支持

FastDFS的tracker和storage都内置了http协议的支持，客户端可以通过http协议来下载文件，tracker 在接收到请求时，通过http的redirect机制将请求重定向至文件所在的storage上。除了内置的http协议外，FastDFS还提供了通过apache或nginx扩展模块下载文件的支持

#### FastDFS集群和配置优化

##### 详细配置

- 配置tracker集群

  ```
  vi /etc/fdfs/tracker.conf
  store_loopup=0 #0轮询 1指定组 2剩余存储空间大的group优先
  ```

- 配置storage集群

  ```
  vi /etc/fdfs/storage.conf
  
  tracker_server=192.168.211.130:22122 tracker_server=192.168.211.136:22122 tracker_server=192.168.211.135:22122 group_name=group1 #注意组名 192.168.211.135 配置是 group2 port=23000 #storage 的端口号,同一个组的 storage 端口号必须相同
  ```

- 启动

  ```
  /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start /usr/bin/fdfs_storaged /etc/fdfs/storage.conf start
  ```

- 查看storage日志，查看tracker集群信息

  ```
  cd /home/xcl/fastdfs/
  cat storaged.log
  cat tracker.log
  ```

- 测试上传

  ```
  vim /etc/fdfs/client.conf tracker_server=192.168.211.130:22122 tracker_server=192.168.211.136:22122 tracker_server=192.168.211.135:22122
  ```

  ```
  /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /root/fastdfs/1.png
  
  ```

##### nginx与FastDFS集群配合使用

```
vi /etc/fdfs/mod_fastdfs.conf
tracker_server=192.168.211.130:22122 
tracker_server=192.168.211.136:22122 
tracker_server=192.168.211.135:22122 
group_name=group1 #注意组名 


vim /usr/local/nginx/conf/nginx.conf
#添加如下配置 
server {        
	listen       8888;        
	server_name  localhost;       
    location ~/group[0-9]/ {           
    	ngx_fastdfs_module;        
    } 
}

//启动
/usr/local/nginx/sbin/nginx 
```

测试：（注意填自己的主机和文件id）

http://192.168.211.130/group1/M00/00/00/wKjIZVyLMi6AH08jAADtXa53YW0605.png

##### FastDFS配置优化

- 最大连接数设置

  ```
  配置文件：tracker.conf 和 storage.conf 
  参数名：max_connections 
  缺省值：256
  ```

- 工作线程数设置

  ```
  配置文件：tracker.conf 和 storage.conf 
  参数名： work_threads 
  缺省值：4 
  
  说明：为了避免CPU上下文切换的开销，以及不必要的资源消耗，不建议将本参数设置得过大。为了发挥出 多个CPU的效能，系统中的线程数总和，应等于CPU总数。 对于tracker server，公式为：     work_threads + 1 = CPU数 对于storage，公式为：    work_threads + 1 + (disk_reader_threads  + disk_writer_threads) * store_path_count  = CPU数
  ```

- storage目录树设置

  ```
  配置文件： storage.conf 
  参数名：subdir_count_per_path 
  缺省值：256
  ```

- storage磁盘读写线程设置

  ```
  配置文件： storage.conf 
  参数名:disk_rw_separated：磁盘读写是否分离 
  参数名:disk_reader_threads：单个磁盘读线程数 
  参数名:disk_writer_threads：单个磁盘写线程数 
  
  如果磁盘读写混合，单个磁盘读写线程数为读线程数和写线程数之和，对于单盘挂载方式，磁盘读写线程分 别设置为   1即可 如果磁盘做了RAID，那么需要酌情加大读写线程数，这样才能最大程度地发挥磁盘性能
  ```

- storage同步延迟相关设置

  ```
  配置文件： storage.conf 
  参数名:sync_binlog_buff_interval：将binlog buffer写入磁盘的时间间隔，取值大于0，
  缺省值 为60s 
  参数名:sync_wait_msec：如果没有需要同步的文件，对binlog进行轮询的时间间隔，取值大于0，缺省 值为200ms 
  参数名: sync_interval：同步完一个文件后，休眠的毫秒数，缺省值为0
  
  ```

#### FastDFS实战（结合SpringBoot）

- 引入Maven坐标

  ```xml
  <parent>    
      <groupId>org.springframework.boot</groupId>   
      <artifactId>spring-boot-starter-parent</artifactId>    <version>2.0.5.RELEASE</version>
  </parent> 
  <dependencies>   
      <dependency>      
          <groupId>org.springframework.boot</groupId>        <artifactId>spring-boot-starter-web</artifactId>   
      </dependency>   
      <dependency>       
          <groupId>com.github.tobato</groupId>     
          <artifactId>fastdfs-client</artifactId>    
          <version>1.26.1-RELEASE</version>    </dependency>
  </dependencies>
  
  ```

  

- 修改配置文件application.yml

  ```yml
  fdfs:  
    connectTimeout: 600  
    trackerList:    
      - 192.168.211.130:22122   
      - 192.168.211.135:22122   
      - 192.168.211.136:22122
      
  server:  port: 8899
  ```

- 编写上传下载服务类

  ```java
  import com.github.tobato.fastdfs.domain.StorePath;
  import com.github.tobato.fastdfs.proto.storage.DownloadByteArray;
  import com.github.tobato.fastdfs.service.FastFileStorageClient;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.stereotype.Component;
  
  import java.io.ByteArrayInputStream;
  
  @Component
  public class FastDFSClientService {
      @Autowired
      private FastFileStorageClient  fastFileStorageClient;
  	
      public String  uploadFile(byte[] bytes,long fileSize,String extension){
          ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
          StorePath storePath = fastFileStorageClient.uploadFile(byteArrayInputStream,fileSize,extension,null);
          System.out.println(storePath.getGroup() + ":" + storePath.getPath() + ":"+storePath.getFullPath());
          return  storePath.getFullPath();
      }
      public  byte[] downloadFile(String fileUrl){
          String group = fileUrl.substring(0,fileUrl.indexOf("/"));
          String path = fileUrl.substring(fileUrl.indexOf("/")+1);
          DownloadByteArray  downloadByteArray = new DownloadByteArray();
          byte[] bytes  = fastFileStorageClient.downloadFile(group,path,downloadByteArray);
          return   bytes;
      }
  }
  ```

  启动类

  ```java
  import com.github.tobato.fastdfs.FdfsClientConfig;
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.context.annotation.EnableMBeanExport;
  import org.springframework.context.annotation.Import;
  import org.springframework.jmx.support.RegistrationPolicy;
  // 解决jmx 重复注册bean
  @EnableMBeanExport(registration = RegistrationPolicy.IGNORE_EXISTING)
  @Import(FdfsClientConfig.class)
  @SpringBootApplication
  public class FastDFSApplicationMain {
      public static void main(String[] args) {
          SpringApplication.run(FastDFSApplicationMain.class,args);
      }
  }
  
  ```

  

避免文件重复上传：上传成功后计算文件对应的MD5然后存入MySQL,添加文件时把文 件MD5和之前存入MYSQL中的存储的信息对比 。参考:DigestUtils.md5DigestAsHex(bytes)

