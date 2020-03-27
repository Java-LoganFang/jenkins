# Jenkins 笔记

## jenkins安装

docker run   -v /root/jenkins/apache-maven-3.6.3:/usr/local/maven  -p 8080:8080 -p 50000:50000 --name jenkins -d jenkins/jenkins:jdk11

## 设置镜像加速

sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/lib/jenkins/updates/default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /var/lib/jenkins/updates/default.json

https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

## 必备插件

Role-based Authorization Strategy

Credentials Binding

Git

Maven  Integration    支持Maven工程

Email Extension Template   邮箱配置

SonarQube 代码审查

Extended Choice Paramete  多选框



## apt-get加速

 sudo vim /etc/apt/sources.list 

```txt
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
```

## 安装Sonarqube

```
docker run -d  --link db -p 9000:9000 -e "SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonar" -e "SONARQUBE_JDBC_USERNAME=odoo" -e "SONARQUBE_JDBC_PASSWORD=odoo" --name sonarqube sonarqube:7.9.1-community
```

```jsx
docker run --name sonarqube -d \
-p 9002:9000 -p 9092:9092 \
sonarqube
```

jenkins操作Docker

```xml
<!--    Docker 自动构建        -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.3.6</version>
                <configuration>
                    <repository>${project.artifactId}</repository>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
```

## NFS的安装

yum install -y nfs-utils

创建共享目录   mkdir -p  /opt/nfs/jenkins 

编写NFS共享配置   vi /etc/exports  

/opt/nfs/jenkins    *(rw,no_root_squash)  *代表对所有的IP都开放此目录，rw表示读写

启动服务

systemctl  enable  nfs     sudo service nfs-server start 开机启动

systemctl  start nfs   systemctl  enable  nfs-server   启动





showmount -e 120.26.38.228  查看远程机器共享的目录



dockerfile自制jenkins从节点

```dockerfile
FROM jenkins/jnlp-slave:latest
MAINTAINER itcast
#切换到root权限
USER root

#安装maven
COPY apache-maven-3.6.2-bin.tar.gz .

RUN tar -zxf apache-maven-3.6.2-bin.tar.gz && \
     mv apache-maven-3.6.2 /user/local && \
     rm -f apache-maven-3.6.2-bin.tar.gz && \
     ln -s  /user/local/apache-maven-3.6.2/bin/mvn  /usr/bin/mvn &&\
     ln -s /user/local/apache-maven-3.6.2 /usr/local/apache-maven & \
     mkdir -p /user/local/apache-maven/repo
  COPY  setting.xml /user/local/apache-maven/conf/setings.xml
  
  
  USER jenkins
```



sources.list

```shell
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse  
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse  
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse  
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse  
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse  
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse  
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse  
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse  
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse  
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse 

```



```dockerfile
FROM jenkins/jnlp-slave:latest
MAINTAINER itcast
#切换到root权限
USER root

#安装maven
COPY sources.list  /etc/apt/sources.list

RUN  apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 3B4FE6ACC0B21F32 && \
     apt-get update  && \
     apt-get install maven -y  
#     rm -f apache-maven-3.6.3-bin.tar.gz && \
#     ln -s  /usr/local/apache-maven-3.6.3/bin/mvn  /usr/bin/mvn && \
#     ln -s /usr/local/apache-maven-3.6.3 /usr/local/apache-maven && \
#     mkdir -p /usr/local/apache-maven/repo
#COPY  setting.xml /usr/local/apache-maven-3.6.3/conf/setings.xml
#ENV MAVEN_HOME=/usr/local/maven/apache-maven-3.6.3/
#ENV M2_HOME=/usr/local/maven/apache-maven-3.6.3/
#ENV JAVA_HOME=/usr/local/openjdk-8/
#ENV PATH=\${PATH}:\${MAVEN_HOME}/bin
#ENV export PATH

```



## 私服登录

vim /etc/docker/daemon.json

```json
{
  "registry-mirrors":["https://mc9glgxz.mirror.aliyuncs.com"],
  "insecure-registries":["39.108.190.246"]
}
```

systemctl daemon-reload

systemctl restart docker

## k8s建立与私服的凭证

docker login 39.108.190.246

kubectl create secret docker-registry registry-auth-secret --docker-server=39.108.190.246 --docker-username=admin --docker-password=123456







```tcl
stage('拉取代码'){
            checkout([$class: 'GitSCM', branches: [[name: "*/${branch}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: "${git_url}"]]])

    }

    stage('安装jar包'){

                        echo "开始安装jar包"
                        sh "ls"

                        sh "mvn -f ./Itoken/${project_name}   clean  package "
                }
        stage('制作镜像'){

                            echo "开始制作镜像"


                            sh "mvn -f ./Itoken/${project_name}  dockerfile:build"
                    }



        stage('镜像上传'){

                                echo "镜像打标签"

                                //定义镜像名称
                                def imageName = "${project_name}:${tag}"

                                //镜像打标签
                                sh "docker tag ${imageName} ${harbor_url}/${harbor_project}/${imageName}"

                                sh "docker login -u admin -p 123456 ${harbor_url}"
                                sh "docker push ${harbor_url}/${harbor_project}/${imageName}"
                                sh "echo 镜像上传成功"
                        }

        stage('镜像发布'){

                                    echo "开始发布镜像"
                                    sshPublisher(publishers: [sshPublisherDesc(configName: 'zcm 101.200.91.110', transfers: [sshTransfer(cleanRemote: false, excludes: '',execCommand: "/root/shell/deploy.sh $harbor_url $harbor_project $project_name $tag $port",execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])

                            }
```

```yml
---
apiVersion: v1
kind: Service
metadata:
  name: eureka
  labels:
    app: eureka
spec:
  type: NodePort
  ports:
    - port: 10086
      name: eureka
      targetPort: 10086
  selector:
    app: eureka
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eureka
spec:
  #serviceName: "eureka"
  replicas: 1
  selector:
    matchLabels:
      app: eureka
  template:
      metadata:
        labels:
          app: eureka
      spec:
        #imagePullSecrets:
          #- name: $SECRET_NAME
        containers:
          - name: eureka
            #image: $IMAGE_NAME
            image: nginx
            ports:
              - containerPort: 10086
  podManageementPolicy: "Parallel"



```

