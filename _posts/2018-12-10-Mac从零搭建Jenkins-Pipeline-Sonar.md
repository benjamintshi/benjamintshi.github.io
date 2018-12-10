## 一、安装Java SDK （若安装则略过）
> Jenkins 依赖 Java环境

###### 1、官网下载Java SDK
[https://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-macosx-x64.dmg?AuthParam=1543903057_a7a990b991bf4ee1842a7b80a5c4b086](https://note.youdao.com/)

###### 2、配置Java环境


```
//环境配置文件：~/.bash_profile文件 （如无该文件，则需要手动创建）
open ~/.bash_profile
//新增如下配置
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home
export CLASSPATH=./$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
//保存环境变量
source ~/.bash_profile
```


###### 二、安装Jenkins
> 通过命令行，不要直接下载官网的.dmg文件，权限问题会导致无法从gitlab上拉取代码
###### 1、安装brew

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

###### 2、安装Jenkins

```
brew install jenkins
```

###### 3、验证Jenkins
jenkins --version
出现对应版本号则安装成功
###### 4、启动Jenkins
控制台直接输入：jenkins
浏览器输入：http://localhost:8080
###### 5、首次进入Jenkins需要Password

```
tail /Users/shitao/.jenkins/secrets/initialAdminPassword（查看密码）
```

###### 6、设置admin帐户
###### 7、Jenkins配置完成

## 三、安装 Jenkins Pipeline 插件
###### 1、通过brew安装Jenkins会默认把Pipeline插件安装上。

## 四、安装 Jenkins BlueOcean插件
###### 1、在插件管理搜索BlueOcean选择安装即可。

> PS：到该阶段Jenkins 管道建设所需要的插件已安装完成。

## 五、安装Sonarqube及Sonar-scanner
> Sonarqube：代码扫描Web站点
> Sonar-scanner：代码扫描工具
###### 1、官网下载对应的压缩文件
[https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.2.0.1227-macosx.zip](https://note.youdao.com/)
[https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.4.zip](https://note.youdao.com/)
###### 2、把对应文件存放到/usr/local/目录
###### 3、配置Sonar (sonar.propertis)
目录：/usr/local/sonarqube/conf
// 配置MySql数据库连接
sonar.jdbc.username=dev
sonar.jdbc.password=dev
//配置jdbc连接
sonar.jdbc.url=jdbc:mysql://10.100.1.73:3306/sonar_web?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false

###### 4、配置Sonar-scanner（sonar-scanner.properties ）

```
//Sonar服务Web站点地址
sonar.host.url=http://localhost:9000
//编码格式
sonar.sourceEncoding=UTF-8
// 配置MySql数据库连接
sonar.jdbc.username=dev
sonar.jdbc.password=dev
//配置jdbc连接
sonar.jdbc.url=jdbc:mysql://10.100.1.73:3306/sonar_web?useUnicode=true&characterEncoding=utf8
```

###### 5、启动Sonar

```
/usr/local/sonarqube/bin/macosx-universal-64/sonar.sh start
```

> 启动：start
> 停止：stop
> 重启：restart

###### 6、浏览器访问Sonar站点
http://localhost:9000

> PS：到该阶段Sonar所需要的配置已完成。

## 六、Jenkins 如何跟Sonar结合起来
###### 1、Jenkins-配置Sonar

- 进入Jenkins首页-系统设置-配置SonarQube名称即可，其他位默认（本机Sonar服务为localhost:9000）。

- 进入Jenkins首页-全局工具配置-配置SonarQube Scanner

> PS：到该阶段Jenkins与Sonar结合所需要的配置已完成。

## 七、Jenkins 配置Gitlab代码拉取

###### 1、新建Jenkins任务-选择流水线

###### 2、配置流水线脚本-点击流水线语法

> 进入流水线语法生产界面

> 示例步骤：选择-checkout:Check out from version control

> 输入对应Gitlab项目地址

> 配置Gitlab账号权限--点击添加

###### Gitlab添加SSH认证

> 配置SSH认证权限：~/.ssh/config 文件（新增config文件）

```
Host *
UseKeychain yes
AddKeysToAgent yes
IdentityFile ~/.ssh/id_rsa
```


> 配置完成如下图：

> 点击生成流水线脚本-复制到流水线脚本中

> PS：到该阶段Jenkins与Gitlab代码拉取所需要的配置已完成。

## 八、Jenkins流水线脚本

```
node {
    stage('拉取代码'){
        checkout([$class: 'GitSCM', branches: [[name: '*/v1.4.2']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'your credentialsid id', url: 'git@your git lab address.git']]])
    }
    stage('Sonar') {
        def sonarqubeScannerHome = tool name: 'SonarQube Scanner'
        withSonarQubeEnv('SonarQube') {
        sh "${sonarqubeScannerHome}/bin/sonar-scanner"
    }
    timeout(time: 1, unit: 'HOURS') {
        def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        }
    }
    stage('构建'){
        //... 自动打包项目
        //... 将打包后的文件传输到对应部署服务器
    }
    stage('构建完成'){
        //...构建完成
    }
}
```


## 九、项目中添加Sonar配置文件（Sonar-project.properties）

```
sonar.projectName=Invault-Wallet-Web
sonar.projectVersion=1.0
sonar.sources=./src
sonar.language=js
sonar.sourceEncoding=UTF-8
```


- 上传文件到gitlab
- 点击立即构建
- 出现如下结果，说明管道建设及代码扫描已经完成
- 进入Sonar站点查看扫描结果
