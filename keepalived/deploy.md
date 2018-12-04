### Keepalived组件部署
1. 下载  
[官方链接](http://htdfd)
2. 安装  
```shell
#解压并进入安装目录
tar zxvf keepalived-2.0.10.tar.gz && cd keepalived-2.0.10
#编译前配置
#prefix指定安装目录，若目录不存在会自动创建
./configure --prefix=/usr/local/keepalived
#编译、安装以及启动
make && make install
#==================安装分割线=====================
# 使用sysv启动(适用于centos6以及以下版本)
#将sbin/keepalived可执行文件配置为用户变量中
ln -s /usr/local/keepalived/sbin/keepalived /usr/bin/keepalived
#复制配置文件到/etc/keepalived/
mkdir -p /etc/keepalived
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
#复制执行参数文件
cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
#配置系统自启动
cp /opt/keepalived-2.0.10/keepalived/etc/init.d/keepalived /etc/init.d/
chkconfig --add keepalived
chkconfig keepalived on
chkconfig --list | grep keepalived
#运行
service keepalived start
```

3. 注意事项  
keepalived组件运行在多个节点，只是配置文件发生了变化
