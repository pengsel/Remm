# Ubuntu软件安装



### 安装JDK

1. 下载.tar.gz格式的jdk文件
2. 解压到一个目录

```
sudo tar xvf jdk-filename
```

3. 将jdk/bin配置到环境变量，注意替换相应目录

```
#jdk1.8 config
export JAVA_HOME=/usr/java/jdk1.8.0_211
export JRE_HOME=${JAVA_HOME}/jre 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:$PATH

```



### 安装Chrome

deb是Ubuntu系统的标准安装文件格式

```text
sudo dpkg -i ./google-chrome-stable_current_i386.deb
sudo apt-get install -f 
```



### 安装搜狗输入法

```
sudo dpkg  -i   sogou_pinyin_linux_1.0.0.0033_amd64.deb      
sudo apt-get install -f 
```

完成安装后需要重启电脑



### Tomcat安装

1. 下载相应的tar.gz包
2. 解压

```
sudo tar -zxvf apache-tomcat-8.5.40.gz
```

3. 修改文件修改权限

```
sudo chmod 755 -R tomcat
```

4. 修改startup.sh和shutdown.sh，在最后一行前添加，注意改成相应jdk和Tomcat路径

```
#set java environment
export JAVA_HOME=/usr/java/jdk1.8.0_211
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:%{JAVA_HOME}/lib:%{JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

#tomcat
export TOMCAT_HOME=/opt/tomcat/apache-tomcat-8.5.40
```



### 忽略系统程序错误报告

```
sudo apt install gksu
gksu gedit /etc/default/apport
```

将enable=1改成0



### 更改环境变量

编辑/etc/environment文件；

source /etc/environment使其生效