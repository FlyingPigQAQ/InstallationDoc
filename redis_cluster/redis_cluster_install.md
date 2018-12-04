### redis集群安装文档
- 新建用户
```shell
useradd ncl  
passwd ncl 密码:ncl@1234
```
- 赋予/app目录ncl用户和组
```shell
chown -R ncl:ncl /app
```
- 切换到ncl用户执行
```shell
su - ncl
```  
- 部署准备环境
```shell
mkdir /app/software
cp redis-4.0.2.gem /app/software/redis-4.0.2.gem
cp ruby-2.2.7.tar.gz /app/software/ruby-2.2.7.tar.gz
cp redis-4.0.11.tar.gz /app/software/redis-4.0.11.tar.gz
mkdir -p /app/data/master/redispid/
mkdir -p /app/data/master/redislog/
mkdir -p /app/data/master/redisdump/
mkdir -p /app/config/redisconfig/master/
mkdir -p /app/data/slave/redispid/
mkdir -p /app/data/slave/redislog/
mkdir -p /app/data/slave/redisdump/
mkdir -p /app/config/redisconfig/slave/
```  
- 编译和安装  
```shell
 cd /app/software
 tar -zxvf redis-4.0.11.tar.gz
 cd redis-4.0.11/src
 make PREFIX=/app/redis install
```
- 复制配置文件到指定目录中  
```shell
#10.1.52.46  
cd /app/software/
cp 46/redis-master.conf /app/config/redisconfig/master/
cp 46/redis-slave.conf /app/config/redisconfig/slave/
#10.1.52.47
cd /app/software/
cp 47/redis-master.conf /app/config/redisconfig/master/
cp 47/redis-slave.conf /app/config/redisconfig/slave/
#10.1.52.48  
cd /app/software/
cp 48/redis-master.conf /app/config/redisconfig/master/
cp 48/redis-slave.conf /app/config/redisconfig/slave/
#10.1.52.49  
cd /app/software/
cp 49/redis-master.conf /app/config/redisconfig/master/
cp 49/redis-slave.conf /app/config/redisconfig/slave/
```
**运行脚本目前为46环境，47、48、49环境只是选择对应目录的配置文件**  

- 启动redis
```shell
#启动master节点
/app/redis/bin/redis-server /app/config/redisconfig/master/redis-master.conf
#启动slave节点
/app/redis/bin/redis-server /app/config/redisconfig/slave/redis-slave.conf
```
- 检查启动情况  
```shell
#主服务端口  
netstat -lntp | grep 6379  
#从服务端口  
netstat -lntp | grep 7000
```


- 当所有环境安装完毕，则继续向下执行安装集群配置  

### 集群安装(只在46环境执行)
- ruby 只在一台服务安装 10.1.52.46,用户为root
- 切换到root用户执行
```shell
su - root
```
- 准备环境
```shell
cd /app/software/
tar -zxvf ruby-2.2.7.tar.gz
cd ruby-2.2.7
./configure --prefix=/usr/local --with-openssl-dir=/etc/pki/tls  --with-zlib-dir=/usr
make && make install
ln -sf /usr/local/bin/ruby /usr/bin/ruby
ln -sf /usr/local/bin/gem /usr/bin/gem
cd ..
gem install redis-4.0.2.gem
```
- 验证redis.gem安装成功,WARNING警告可以忽略
```shell
ruby 和 redis 下面是安装成功 1 gem installed
[root@erds1 software]# gem install redis-4.0.2.gem
Successfully installed redis-4.0.2
Parsing documentation for redis-4.0.2
Installing ri documentation for redis-4.0.2
Done installing documentation for redis after 1 seconds
WARNING:  Unable to pull data from 'https://rubygems.org/': no such name (https://rubygems.org/specs.4.8.gz)
1 gem installed
```
- 编辑client.rb配置文件
```shell
vi /usr/local/lib/ruby/gems/2.2.0/gems/redis-4.0.2/lib/redis/client.rb
#修改password中的nil为配置文件的密码
password
      :password => "sinosoft",
    def password
      @options[:password]
        call [:auth, password] if password
          defaults[:password] = CGI.unescape(uri.password) if uri.password
          @options[:password] = DEFAULTS.fetch(:password)
```

- 集群建立
```shell
cd /app/software/redis-4.0.11/src
./redis-trib.rb create --replicas 1 10.1.52.46:6379 10.1.52.47:6379 10.1.52.48:6379  10.1.52.49:6379 10.1.52.46:7000 10.1.52.47:7000  10.1.52.48:7000   10.1.52.49:7000
```
- 集群建立成功显示结果
```shell
[root@erds1 src]# ./redis-trib.rb create --replicas 1 10.1.52.46:6379 10.1.52.47:6379 10.1.52.48:6379  10.1.52.49:6379 10.1.52.46:7000 10.1.52.47:7000  10.1.52.48:7000 10.1.52.49:7000
>>> Creating cluster
>>> Performing hash slots allocation on 8 nodes...
Using 4 masters:
10.1.52.46:6379
10.1.52.47:6379
10.1.52.48:6379
10.1.52.49:6379
Adding replica 10.1.52.47:7000 to 10.1.52.46:6379
Adding replica 10.1.52.48:7000 to 10.1.52.47:6379
Adding replica 10.1.52.49:7000 to 10.1.52.48:6379
Adding replica 10.1.52.46:7000 to 10.1.52.49:6379
M: fc44d0aa4d87520c42f43da2d98fafed7241d2ad 10.1.52.46:6379
   slots:0-4095 (4096 slots) master
M: 7c24b2a903dae6e365adbe5aa41e3c06706f7d92 10.1.52.47:6379
   slots:4096-8191 (4096 slots) master
M: 52b15328eab888a93f13665dc26a812e54d46bed 10.1.52.48:6379
   slots:8192-12287 (4096 slots) master
M: 598135a0373681c0501c9011bd85e4811120f871 10.1.52.49:6379
   slots:12288-16383 (4096 slots) master
S: 933a63086fed850a5ecbfc8f517484ab6b843eaf 10.1.52.46:7000
   replicates 598135a0373681c0501c9011bd85e4811120f871
S: 4385279e0b767f2e5e189efe51a05c4e57f5dd94 10.1.52.47:7000
   replicates fc44d0aa4d87520c42f43da2d98fafed7241d2ad
S: 0791d487247d953f7ad025c4799dccc53454035d 10.1.52.48:7000
   replicates 7c24b2a903dae6e365adbe5aa41e3c06706f7d92
S: c886f865d2087683e51d17974cbf804772b99115 10.1.52.49:7000
   replicates 52b15328eab888a93f13665dc26a812e54d46bed
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.....
>>> Performing Cluster Check (using node 10.1.52.46:6379)
M: fc44d0aa4d87520c42f43da2d98fafed7241d2ad 10.1.52.46:6379
   slots:0-4095 (4096 slots) master
   1 additional replica(s)
S: 4385279e0b767f2e5e189efe51a05c4e57f5dd94 10.1.52.47:7000
   slots: (0 slots) slave
   replicates fc44d0aa4d87520c42f43da2d98fafed7241d2ad
S: 933a63086fed850a5ecbfc8f517484ab6b843eaf 10.1.52.46:7000
   slots: (0 slots) slave
   replicates 598135a0373681c0501c9011bd85e4811120f871
M: 7c24b2a903dae6e365adbe5aa41e3c06706f7d92 10.1.52.47:6379
   slots:4096-8191 (4096 slots) master
   1 additional replica(s)
S: 0791d487247d953f7ad025c4799dccc53454035d 10.1.52.48:7000
   slots: (0 slots) slave
   replicates 7c24b2a903dae6e365adbe5aa41e3c06706f7d92
S: c886f865d2087683e51d17974cbf804772b99115 10.1.52.49:7000
   slots: (0 slots) slave
   replicates 52b15328eab888a93f13665dc26a812e54d46bed
M: 598135a0373681c0501c9011bd85e4811120f871 10.1.52.49:6379
   slots:12288-16383 (4096 slots) master
   1 additional replica(s)
M: 52b15328eab888a93f13665dc26a812e54d46bed 10.1.52.48:6379
   slots:8192-12287 (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
- 验证集群
```shell
./redis-cli -c -h 10.1.52.46 -p 6379 -a sinosoft
10.1.52.46:6379> set key a
-> Redirected to slot [12539] located at 10.1.52.49:6379
OK
10.1.52.49:6379> get key
"a"
```
### 相关问题描述
- 配置文件描述
 - bind 绑定ip地址
 - masterauth 主机之间通信密码
 - requirepass 客户端认证密码
 - protected-mode no 关闭保护模式，以便客户端连接
 - port 绑定端口号
 - timeout 超时时间设置
 - daemonize yes 默认以后台方式运行
 - pidfile pid文件位置
 - loglevel 日志级别
 - logfile redis日志文件目录
 - dir redis dump文件保存位置
 - appendonly no 关闭aof配置
 - cluster-enabled yes开启集群模式
 - cluster-config-file 指定cluster配置文件位置
- redis密码更改，应用程序也得随之更改配置文件进行重启
