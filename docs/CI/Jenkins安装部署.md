## 1. 安装JDK

``` 
yum install java-1.8.0-openjdk* -y
```



## 2. 安装Jenkins

### 2.1 安装

```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum upgrade
# Add required dependencies for the jenkins package
sudo yum install jenkins
```

### 2.2 修改Jenkins配置

```
vi /etc/sysconfig/jenkins
```

端口修改为：

JENKINS_PORT="9090"

【踩坑】这里修改端口后不会生效，Jenkins还是会启动再8080端口，完成Jenkins初始化后再回来做如下修改：

```
vi /usr/lib/systemd/system/jenkins.service
```

修改为：Environment="JENKINS_PORT=9090"

```
systemctl daemon-reload
systemctl restart jenkins
```

现在才是运行在9090端口

![image-20220517094743207](D:\ResilioSync\myBlog\docs\CI\img\image-20220517094743207.png)

### 2.3 插件管理

启动Jenkins：

```
systemctl start jenkins
```

安装git插件

在Linux实例中安装git软件

```
yum install git -y
```



### 2.4 添加凭证

![image-20220517100150423](D:\ResilioSync\myBlog\docs\CI\img\image-20220517100150423.png)

测试拉取代码成功（Build Now）

![image-20220517100831990](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517100831990.png)

## 3. 安装Maven

### 3.1 安装Maven

上传Maven安装包到Linux实例，解压，版本最好与开发使用版本一致。

![image-20220517101125649](D:\ResilioSync\myBlog\docs\CI\img\image-20220517101125649.png)

### 3.2 配置环境变量

```
vi /etc/profile
```

添加：

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk

export MAVEN_HOME=/opt/maven

export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin

```
source /etc/profile
```

### 3.3 在Jenkins配置JDK和Maven

![image-20220517102413506](D:\ResilioSync\myBlog\docs\CI\img\image-20220517102413506.png)

![image-20220517102521524](D:\ResilioSync\myBlog\docs\CI\img\image-20220517102521524.png)

![image-20220517102745448](D:\ResilioSync\myBlog\docs\CI\img\image-20220517102745448.png)

### 3.4 添加阿里云Maven镜像

```
vi /opt/maven/conf/settings.xml
```

添加：

```
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```



### 3.5 测试

添加build命令：

![image-20220517103417137](D:\ResilioSync\myBlog\docs\CI\img\image-20220517103417137.png)

控制台信息，依赖下载中：

![image-20220517103505658](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517103505658.png)

构建成功：

![image-20220517103800075](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517103800075.png)

启动项目：

```
java -jar /var/lib/jenkins/workspace/training_hub_freestyle/training_plan/target/training_plan-0.0.1-SNAPSHOT.jar
```

成功!!!

![image-20220517104108714](D:\ResilioSync\myBlog\docs\CI\img\image-20220517104108714.png)