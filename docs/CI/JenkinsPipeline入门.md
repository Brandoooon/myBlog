## 1. 了解pipeline script代码生成器

以拉取代码为例，使用Jenkins自带的groovy代码生成器：

![image-20220517112839145](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517112839145.png)

生成脚本：

```groovy
checkout([$class: 'GitSCM', branches: [[name: '*/dev']], extensions: [], userRemoteConfigs: [[credentialsId: '45291ce0-d5ac-40c4-86e3-006b086d0110', url: 'git@jihulab.com:xiantangjiao/traininghub.git']]])
```

添加到pipeline script中：

![image-20220517112941748](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517112941748.png)

拉取测试成功：

![image-20220517113039684](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517113039684.png)

## 2. 使用Gitlab hook自动触发构建

### 2.1 Jenkins安装、配置Gitlab插件

在Pipeline项目中添加gitlab的builder trigger：

![image-20220517143401796](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517143401796.png)

在系统配置中关闭Enable authentication for '/project' end-point：

![image-20220517143559372](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517143559372.png)

### 2.2 Gitlab项目开启Jenkins Integration

![image-20220517143452835](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220517143452835.png)

## 3. 添加邮件提醒

## 4. SonarQube部署

​		节省资源，暂未部署。

## 5. Docker安装

```
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```
yum list docker-ce --showduplicates | sort -r
sudo yum install docker-ce-<VERSION_STRING>
```

修改/etc/docker/daemon.json文件，如果没有先建一个即可:

```
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

```
sudo systemctl daemon-reload
sudo service docker restart
```

## 6. Harbor安装

修改harbor.yml：

1. 修改hostname和port

![image-20220518111250345](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220518111250345.png)

2. 修改admin密码

   - 需要进入harbor-db容器的postgresql数据库中修改；

   - 密码采用pbkdf2加密，可以使用在线的pbkdf2 generator生成password和salt；

   - 修改后登录web ui，再选择change password，修改为刚保存在数据库中的密码。

harbor重启方法：

在harbor安装目录下运行

```
docker-compose stop
docker-compose up -d
```

添加harbor仓库到docker信任列表，修改

```
vi /etc/docker/daemon.json
```

添加：

```
"insecure-registries": ["127.0.0.1:85"]
```

从Linux登录Harbor：

```
docker login 127.0.0.1:85
```

push镜像到harbor：

```
docker push 127.0.0.1:85/traininghub/traininghub:v2
```

下载镜像：

```
docker pull 127.0.0.1:85/traininghub/traininghub:v2
```

## 7.使用Jenkins打包镜像

### 7.1 制作dockerfile

### 7.2 （踩坑）为jenkins添加操作docker的权限

将jenkins用户添加到docker用户组中：

```
sudo gpasswd -a jenkins docker
sudo service jenkins restart
```

## 8. 多模块构建和镜像打包的配置

### 8.1 构建

1. 父工程

​		父工程不用打包镜像，但是需要在common模块安装前先进行安装，Jenkins脚本配置如下：

​		mvn install -N

​		-N用户忽略子模块的构建。

2. common模块

3. 业务模块

   如果没有做第一步父工程的安装，那么在业务模块构建的时候会报错：

   ```
   [ERROR] Failed to execute goal on project training_plan: Could not resolve dependencies for project cn.xiantangjiao:training_plan:jar:0.0.1-SNAPSHOT: Failed to collect dependencies at cn.xiantangjiao:common:jar:0.0.1-SNAPSHOT: Failed to read artifact descriptor for cn.xiantangjiao:common:jar:0.0.1-SNAPSHOT: Could not find artifact cn.xiantangjiao:training_hub:pom:0.0.1-SNAPSHOT -> [Help 1]
   ```

完整的Jenkinsfile如下：

```groovy
pipeline {
    agent any

    stages {
        stage('拉取代码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], extensions: [], userRemoteConfigs: [[credentialsId: ${git_auth}, url: ${git_url}]]])
            }
        }
        stage('安装父工程模块') {
            steps {
                sh 'mvn clean install -N'
            }
        }
        stage('编译，安装common模块') {
            steps {
                sh 'mvn -f common clean install'
            }
        }
        stage('编译，打包training_plan模块') {
            steps {
                sh 'mvn -f training_plan clean package dockerfile:build'
            }
        }
    }
}
```

### 8.2 dockerfile打包

1. 父工程

   pom.xml中引入依赖，并且设置skip为false，跳过打包。

   ```xml
   <plugin>
       <groupId>com.spotify</groupId>
       <artifactId>dockerfile-maven-plugin</artifactId>
       <version>1.3.6</version>
       <inherited>false</inherited>
       <configuration>
           <skip>true</skip>
       </configuration>
   </plugin>
   ```

2. common模块

   pom.xml中引入依赖，并且设置skip为false，跳过打包。

   ```xml
   <plugin>
       <groupId>com.spotify</groupId>
       <artifactId>dockerfile-maven-plugin</artifactId>
       <version>1.3.6</version>
       <inherited>false</inherited>
       <configuration>
           <skip>true</skip>
       </configuration>
   </plugin>
   ```

   

3. 业务模块

​		pom.xml中引入依赖，并且设置skip为true，打包镜像，在Jenkinsfile中添加脚本dockerfile:build。

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.3.6</version>
    <configuration>
        <skip>false</skip>
        <repository>${project.artifactId}</repository>
        <buildArgs>
            <JAR_FILE>target/training_plan-0.0.1-SNAPSHOT.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

## 9. 部署服务

创建deploy.sh脚本，存放在主机上，例如/opt。

```shell
#! /bin/sh
#接收外部参数
harbor_url=127.0.0.1:85
harbor_project_name=traininghub
project_name=training_plan
tag=latest
port=8080

imageName=$harbor_url/$harbor_project_name/$project_name:$tag

echo "$imageName"

#查询容器是否存在，存在则删除
containerId=`docker ps -a | grep -w ${project_name}:${tag}  | awk '{print $1}'`
if [ "$containerId" !=  "" ] ; then
    #停掉容器
    docker stop $containerId

    #删除容器
    docker rm $containerId
	
	echo "成功删除容器"
fi

#查询镜像是否存在，存在则删除
imageId=`docker images | grep -w $project_name  | awk '{print $3}'`

if [ "$imageId" !=  "" ] ; then
      
    #删除镜像
    docker rmi -f $imageId
	
	echo "成功删除镜像"
fi

# 登录Harbor
docker login --username traininghub --password 7shY6tBb $harbor_url

# 下载镜像
docker pull $imageName

# 启动容器
docker run -di -p $port:$port $imageName

echo "容器启动成功"

```



由于实在本机上部署，只需要访问执行本机的deploy.sh脚本即可，在Jenkinsfile中增加stage：

```groovy
stage('部署服务') {
    steps {
        sh 'sh /opt/deploy.sh'
    }
}
```

