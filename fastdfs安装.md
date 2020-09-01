安装fastDFS（我的安装目录为/root/software/fastdfs）

- 安装编译环境

  ```
  yum install git gcc gcc-c++ make automake  vim   wget  libevent -y 
  ```

- 安装公用函数包

  ```
  mkdir /root/software/fastdfs
  cd /root/software/fastdfs
  git clone https://github.com/happyfish100/libfastcommon.git --depth 1 
  cd libfastcommon/ 
  ./make.sh && ./make.sh install 
  ```

- 安装fastDFS

  ```
  cd /root/software/fastdfs
  wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz 
  tar -zxvf V5.11.tar.gz 
  cd fastdfs-5.11 
  ./make.sh && ./make.sh install 
  ```

- 配置fastDFS

  配置文件准备

  ```
  cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf 
  cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf 
  cp /etc/fdfs/client.conf.sample  /etc/fdfs/client.conf   
  cp /root/fastdfs/fastdfs-5.11/conf/http.conf  /etc/fdfs 
  cp /root/fastdfs/fastdfs-5.11/conf/mime.types  /etc/fdfs
  
  ```

  修改追踪器配置

  ```
  vim /etc/fdfs/tracker.conf 
  #需要修改的内容如下 
  port=22122   #默认端口
  base_path=/home/xcl/fastdfs #自定义基础路径
  ```

  修改存储节点配置

  ```
  vim /etc/fdfs/storage.conf 
  #需要修改的内容如下 
  port=23000   
  base_path=/home/xcl/fastdfs  # 数据和日志文件存储根目录 store_path0=/home/xcl/fastdfs  # 第一个存储目录 tracker_server=192.168.6.132:22122   # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致) http.server_port=8888
  ```

- 启动

  ```
  /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start /usr/bin/fdfs_storaged /etc/fdfs/storage.conf start 
  查看所有运行的端口 
  netstat -ntlp
  ```

- 测试上传

  修改客户端配置

  ```
  vim /etc/fdfs/client.conf 
  #需要修改的内容如下 
  base_path=/home/xcl/fastdfs 
  #tracker服务器IP和端口 tracker_server=192.168.6.132:22122 
  ```

  上传

  ```
  #保存后测试,返回ID表示成功 如：group1/M00/00/00/xxx.png /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /root/fastdfs/1.png group1/M00/00/00/wKjTiF7h5EWASb5aAACGZa9JdFo611.png
  ```

fastDFS整合nginx

- 安装fastdfs-nginx-module模块

  ```
  cd /root/software/fastdfs
  wget https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.20.tar.gz 
  tar -xvf V1.20.tar.gz 
  cd fastdfs-nginx-module-1.20/src 
  vim config 修改第5 行 和 15 行 修改成 ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/" 
  CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
  
  ```

- 配置fastdfs-nginx-module

  拷贝配置文件至/etc/fdfs下

  ```
  cp mod_fastdfs.conf /etc/fdfs/
  ```

  修改配置文件

  ```
  vim /etc/fdfs/mod_fastdfs.conf 
  #需要修改的内容如下 
  tracker_server=192.168.6.132:22122   url_have_group_name=true 
  store_path0=/home/xcl/fastdfs
  
  ```

  创建nginx需要的目录

  ```
  mkdir -p /var/temp/nginx/client
  ```

- 安装nginx

  ```
  cd /root/software/fastdfs
  wget http://nginx.org/download/nginx-1.15.6.tar.gz 
  tar -zxvf nginx-1.15.6.tar.gz
  
  cd nginx-1.15.6/
  yum -y install pcre-devel openssl openssl-devel 
  # 添加fastdfs-nginx-module模块 
  ./configure  --add-module=/root/fastdfs/fastdfs-nginx-module-1.20/src 
  ```

  ```
  编译安装 
  make && make install 
  查看模块是否安装上
  /usr/local/nginx/sbin/nginx -V
  
  ```

- 配置nginx

  ```
  vim /usr/local/nginx/conf/nginx.conf
  
  #添加如下配置 
  server {        
  	listen       8888;       
      server_name  localhost;       
      location ~/group[0-9]/ {      
      	ngx_fastdfs_module;      
       } 
   }
  ```

  注：记得关闭防火墙

- 启动nginx

  ```
  /usr/local/nginx/sbin/nginx 
  ```

- 测试

![image-20200901151036688](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200901151036688.png)

整合GraphicsMagick工具（安装目录/usr/local/src）

- 下载Lua、LuaJIT、GraphicsMagick

  下载LuaJIT

  ```
  wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
  ```

  下载Lua

  ```
  wget http://www.lua.org/ftp/lua-5.3.3.tar.gz
  ```

  下载GraphicsMagick

  ```
  wget https://sourceforge.net/projects/graphicsmagick/files/graphicsmagick/1.3.35/GraphicsMagick-1.3.35.tar.gz
  ```

  下载nginx使用的一些模块（zlib、ngx_devel_kit、lua-nginx-module）

  ```
  wget http://www.zlib.net/fossils/zlib-1.2.11.tar.gz
  git clone https://github.com/simpl/ngx_devel_kit.git
  wget https://github.com/openresty/lua-nginx-module/archive/v0.10.9rc7.tar.gz
  
  解压
  tar -xvf zlib-1.2.11.tar.gz
  tar -xvf v0.10.9rc7.tar.gz
  ```

- 编译安装LuaJIT：LuaJIT是Lua语言编写的即时编译器

  ```
  tar -zxf LuaJIT-2.0.4.tar.gz
  cd LuaJIT-2.0.4/
  make && make install
  export LUAJIT_LIB=/usr/local/lib
  export LUAJIT_INC=/usr/local/include/luajit-2.0
  ln -s /usr/local/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2
  ```

- 编译安装Lua

  ```
  tar -xvf lua-5.3.3.tar.gz
  cd lua-5.3.3/
  make linux && make install
  
  #编译时遇到错误lua.c:80:31: fatal error: readline/readline.h: No such file or directory
  yum install readline-devel
  ```

- 编译安装GraphicsMagick

  ```
  #安装graphicsmagick支持的图片格式
  yum install libjpeg libjpeg-devel libpng libpng-devel giflib giflib-devel freetype freetype-devel
  tar -xvf GraphicsMagick-1.3.35.tar.gz
  cd GraphicsMagick-1.3.35/
  ./configure --prefix=/usr/local/GraphicsMagick --enable-shared
  make && make install
  ```

- 配置GraphicsMagick环境变量

  ```
  export GMAGICK_HOME="/usr/local/GraphicsMagick"
  export PATH="$GMAGICK_HOME/bin:$PATH"
  LD_LIBRARY_PATH=$GMAGICK_HOME/lib:$LD_LIBRARY_PATH
  export LD_LIBRARY_PATH
  ```

- 验证GraphicsMagick

  ```
  gm -version
  ```

  ![image-20200901152305778](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200901152305778.png)

- 下载lua脚本

  ```
  git clone https://github.com/hpxl/nginx-lua-fastdfs-GraphicsMagick.git
  cd nginx-lua-fastdfs-GraphicsMagick/lua
  cp ./* /usr/local/nginx/conf/lua
  ```

- 修改fastdfs.lua

  ```
  vim /usr/local/nginx/conf/lua/fastdfs.lua
  
  #在46行
  fdfs:set_tracker("你的ip", 22122)
  #在72行
  local command = "/usr/local/GraphicsMagick/bin/gm convert " .. originalFile  .. " -thumbnail " .. area .. " -background gray -gravity center -extent " .. area .. " " .. ngx.var.file;
  ```

  

- 配置nginx

  添加模块（nginx源码目录下执行）

  ```
  ./configure --add-module=/root/software/fastDFS/fastdfs-5.11/fastdfs-nginx-module-1.20/src/  --prefix=/usr/local/nginx --with-zlib=/usr/local/src/zlib-1.2.11 
  --add-module=/usr/local/src/lua-nginx-module-0.10.9rc7 
  --add-module=/usr/local/src/ngx_devel_kit
  ```

  ```
  make && make install
  ```

  修改配置

  ```
  #安装并启动nginx后，使用lua1来验证nginx是否启动成功并支持lua脚本
  		#浏览器输入http://192.168.6.132:8888/lua1，可以输出“Hello, Lua!”即启动成功
  		location /lua1 {
              default_type 'text/plain';
              content_by_lua 'ngx.say("Hello, Lua!")';
  		}
  		
  		
          # fastdfs 缩略图生成
          location /group1/M00 {
                  alias /home/xcl/fastdfs/data;
   
                  set $image_root "/home/xcl/fastdfs/data";
                  if ($uri ~ "/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/(.*)") {
                    set $image_dir "$image_root/$3/$4/";
                    set $image_name "$5";
                    set $file "$image_dir$image_name";
                  }
   
                  if (!-f $file) {
          #         # 关闭lua代码缓存，方便调试lua脚本
                    #lua_code_cache off;
                    content_by_lua_file "/usr/local/nginx/conf/lua/fastdfs.lua";
                  }
                 # ngx_fastdfs_module;
          }
  ```

- 重启或启动

  ```
  启动
  /usr/local/nginx/sbin/nginx
  
  重启
  /usr/local/nginx/sbin/nginx -s reload
  ```

  