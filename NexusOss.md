## Nexus Oss 安装文档
#### 1. 配置用户权限（网页操作）
#### 2. 配置settings.xml
增加,
```xml
<server>
    <id>ID</id>
    <username>USERNAME</username>
    <password>PASSWORD</password>
</server>
```
#### 3. 上传依赖到远程代码库
>注意：**不能将jar文件或者pom文件指定为本地代码仓库的文件，否则会报错Cannot deploy artifact from the local repository。具体原因待查**  
##### - 上传单个依赖到远程代码库
```shell
#第一种方式
mvn deploy:deploy-file -DgroupId=mysql   -DartifactId=mysql-connector-java   -Dversion=5.1.45   -Dpackaging=jar   -Dfile=/Users/tobbyquinn/Desktop/mysql-connector-java-5.1.45.jar   -DrepositoryId=sinosoft   -Durl=http://10.1.105.169:8081/repository/maven-releases/
#第二种方式
mvn deploy:deploy-file -Durl=http://10.1.105.169:8081/repository/maven-releases/ -Dfile="/Users/tobbyquinn/Desktop/mysql-connector-java-5.1.45.jar" -DgeneratePom=false -DpomFile="/Users/tobbyquinn/Desktop/mysql-connector-java-5.1.45.pom"
```

<font color='blue'>**一定记住：url一定要为maven-relases或maven-snapshot,不能为group组如maven-central，否则报错 HTTP.405,PUT method not allowed **


##### - 上传工程所有依赖到本地代码库
1. 导出pom的所有依赖
```shell
mvn -Dmdep.copyPom=true dependency:copy-dependencies
```
2. 遍历并上传
```shell
for pom in target/dependency/*.pom; do mvn deploy:deploy-file -Durl=http://10.1.105.169:8081/repository/maven-releases/ -Dfile="${pom%%.pom}.jar" -DgeneratePom=false -DpomFile="$pom" -DrepositoryId=sinosoft;done
```  

##### -  部署工程到远程代码仓库
修改pom.xml
```xml
<distributionManagement>
        <repository>
            <id>sinosoft</id>
            <name>部署release到nexus</name>
            <url>http://10.1.105.169:8081/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>sinosoft</id>
            <name>部署snapshot到NEXUS</name>
            <url>http://10.1.105.169:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
</distributionManagement>
```
##### -  从远程代码仓库下载依赖
同上,
```xml
<profiles>
        <profile>
            <id>dev</id>
            <repositories>
                <repository>
                    <id>sinosoft</id>
                    <name>repo-public</name>
                    <url>http://10.1.105.169:8081/repository/maven-public/</url>
                    <releases>
                        <checksumPolicy>warn</checksumPolicy>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </releases>
                    <snapshots>
                        <checksumPolicy>warn</checksumPolicy>
                        <updatePolicy>never</updatePolicy>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
        </profile>
</profiles>
```  
开启activeProfiles=&lt;RepositoryId>
