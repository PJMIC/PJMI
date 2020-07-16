# Docker+GitLab+Jenkins自动化管理项目


0x01：👏搭建项目自动化部署

<!-- more -->


### 一、环境配置：**

> - `System`：CentOS 7
> - `GitLab Version`：beginor/gitlab-ce:11.0.1-ce.0
> - `JenKins Version`：官网最新 [jenkins.war](http://mirrors.jenkins.io/war-stable/latest/jenkins.war)
> - `JDK Version`: 1.8 
> - `Maven Version`:  3.6.1 

### 二、Dokcer:

1. #### **安装需要的库**

   - ```shell
     sudo yum install -y yum-utils device-mapper-persistent-data lvm2
     ```

2. #### **添加docker的源**

   - ```shell
     sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
     ```

3. #### **安装Docker**

   - ```shell
     sudo yum install docker-ce
     ```

4. #### **启动docker服务和开机自启**

   - ```shell
     sudo systemctl start docker
     sudo systemctl enable docker
     ```

5. #### **`Docker Installed End`**

### 三、Gitlab：
   - #### PS：
   
     - 关闭防火墙：systemctl stop firewalld
     - 关闭防火墙自启：systemctl disable firewalld

1. #### **创建映射文件夹**（可自定义，待会儿自己修改命令相关的参数）

   - ```shell
     mkdir -p /data/gitlab/etc
     mkdir -p /data/gitlab/log
     mkdir -p /data/gitlab/data
     ```

2. #### **安装并启动GitLab**

   - > #### **`参数解析：`**

     ```shell
     --detach 表示后台运行
     --publish 表示容器类与宿机映射和端口
     --name 表示容器的名字
     --restart 重启策略：退出容器是重启容器
     ```

   - > #### `构建容器命令`

     ```shell
     docker run \
      --detach \
      --publish 998:22 \
      --publish 8443:443 \
      --publish 80:80 \
      --name gitlab \
      --restart unless-stopped \
      -v /data/gitlab/etc:/etc/gitlab \
      -v /data/gitlab/log:/var/log/gitlab \
      -v /data/gitlab/data:/var/opt/gitlab \
      beginor/gitlab-ce:11.0.1-ce.0
     ```

   - > **在命令的最后的一行为镜像的名称(`beginor/gitlab-ce:11.0.1-ce.0`)或者ID(需要先`docker pull 镜像`，后通过 `docker images`，可以获取到镜像的ID)，运行上面的命令即可安装并启动gitlab（这样他会先检查本地有没有此镜像，没有就会先pull，然后再带参数run）。**
   
     ```shell
     docker ps： 查看正在运行的容器
     docker ps -a：查看所有创建的容器
     ```
     


### 四、Jenkins：

1. #### **下载相关的包**:

   - `Jenkins.war包（这儿是用java启动，你也可以通过docker）、maven、jdk`

2. #### **运行参数**:

   - ```shell
     java  -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --httpPort=8088 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
     ```

   - > `参数解析：`
     >
     > ```shell
     > java  -Djava.awt.headless=true #解释：https://www.cnblogs.com/wudi-dudu/p/7871405.html
     >    -DJENKINS_HOME=/var/lib/jenkins #定义Jenkins的根目录
     >    -jar /usr/lib/jenkins/jenkins.war #war包位置，根据你的实际情况填写参数
     >    --logfile=/var/log/jenkins/jenkins.log #Jenkins的日志输出
     >    --webroot=/var/cache/jenkins/war #Jenkins缓存路径
     >    --httpPort=8088 #Jenkins的端口（看自己的端口情况修改）
     >    --debug=5 #
     >    --handlerCountMax=100 #
     >    --handlerCountMaxIdle=20 #
     > ```

3. #### **访问 http://ip:httpPort 即可开始`Jenkins`之旅**:
   
   - 首先去提示的路径复制密码到 Input 框;
   - 然后选择推荐插件，这儿可能有点慢，看各自的网速;
   - 安装完后即可创建账户;
   - Jenkins搭建结束。

### 五、实现从Gitlab自动拉取代码并部署：

1. #### **登录界面**

   [![KrPuwD.md.png](https://s2.ax1x.com/2019/10/26/KrPuwD.md.png)](https://imgchr.com/i/KrPuwD)

2. #### **登录后的界面**（新的没有任何的任务）
   
   ![KrP20U.png](https://s2.ax1x.com/2019/10/26/KrP20U.png)
   
3. #### **新建工作空间**

   1. **创建任务**![KriOK0.png](https://s2.ax1x.com/2019/10/27/KriOK0.png)

   2. **通用配置**

      1. 这儿配置两个参数：第一个参数 **Discard old builds** ![KretoV.png](https://s2.ax1x.com/2019/10/27/KretoV.png)

      2. **第二个参数 This project is  parameterized** 

      ![KrlguF.png](https://s2.ax1x.com/2019/10/27/KrlguF.png) 

   3. **源码管理**（我们用gitlab，这儿需要自己现在Gitlab中去创建项目和获取项目地址）![KrtNRA.png](https://s2.ax1x.com/2019/10/27/KrtNRA.png)

   4. **构建触发器**（构建方式有好多种，具体看实际情况，日程表可以直接copy）

      ![KrtWMq.png](https://s2.ax1x.com/2019/10/27/KrtWMq.png)

      `可参考SCDN`：【[Poll SCM]( https://blog.csdn.net/xueyingqi/article/details/53216506 )】

   5. **构建环境**：不用设置，跳过

   6. **构建**：因为启动项目的方式有很多种方案，所以这儿可以使用骨灰级的方式（shell）

      - ps：**本次演示的是在搭建的Jenkins的服务器中，启动项目，如果你需要将项目到第三方服务器是上启动，就得在第三方服务器的路径下写启动脚本（start.sh）,这是一种解决方案，但不是绝对的。**

      - 添加构建方式： `Execute shell`

      - ```shell
        #$：这儿是在引用刚刚在 This project is  parameterized 配置的参数
        case $test_dev in
           deploy)
           #当选择的是 deploy 就执行下面呢的命令,注意绝对路径
             /usr/local/maven/bin/mvn clean package --settings /usr/local/maven/conf/settings.xml -Dmaven.test.skip=true
             ;;
           rollback)
           #当选择的是这个即是回滚
           	 echo "${version}"
             rm -rf target
             #${JENKINS_HOME}此参数是Jenkins默认的俄路径，自己和自己的核对改成对应的路径
             cp -R ${JENKINS_HOME}/jobs/你的任务名/builds/${version}/archive/target .
             pwd && ls
             ;;
           *)
             exit
             ;;
        esac
        jenkinsjarpath=${JENKINS_HOME}/workspace/你的任务名 #如test_dev
        jarpath=/data/test/target #这是你要启动项目的路径（将jar包复制到此目录）
        cp ${jenkinsjarpath}/target/test.jar /data/test/ #将jar包复制到jarpath
        sh /data/test/start.sh ${port} #执行jarpath的shell脚本（类容为启动jar包的命令）
        echo "Start Run......"
        ```

      - start.sh

        ```shell
        #!/bash/bin
        jarName=test.jar
        function shutDown(){
        		#获取上一次的构建的项目pid
                pid=`ps -ef|grep ${jarName}|grep -v grep|awk '{print $2}'`
                     kill -9 ${pid} #干掉
                     echo "shut down" #打印信息
        }
        
        function startUp(){
        		#2:是打印错误到屏幕，&: 打印到后台
                nohup java -jar test.jar --server.port=$1  >> ./logs.log 2>&1 & #$1是运用上面参数${port}
                echo "start up"
        }
        #先关闭上一个项目
        shutDown()
        #启动新构建的项目
        startUp()
        ```

   7. **构建后操作**

      ![KrwuBq.png](https://s2.ax1x.com/2019/10/27/KrwuBq.png)

3. 回到开始：

   ![KrwCHP.png](https://s2.ax1x.com/2019/10/27/KrwCHP.png)

4. **现在你可以是实现自动Gitlab+jenkins 构建项目了。**

4. **Shell 脚本方面可以参考下** 【[菜鸟教程]( https://www.runoob.com/linux/linux-shell.html )】

