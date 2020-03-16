<center><h1>TAS开启HTTPS</h1></center>
## 1.概述

​	tas开启https除修改配置文件外，还需要准备好所需要的证书（根证书和服务器证书）以及给客户电脑添加信任

- 配置文件：start.ini
- 根证书：CA（用来生成服务器证书，并且还需要安装到客户电脑）
- 服务器证书：HTTPS中TAS用于来浏览器交互，配置文件需要指定其路径

## 2.准备工作

### 2.1.生成根证书（CA）

#### 2.1.1.生成根证书密钥

~~~shell
openssl genrsa -des3 -out ca.key 4096
~~~

​	需要输入密码，此密码很重要，后面签名证书时需要用到。例如：rootca

​	这一步会生成`ca.key`文件

#### 2.1.2.生成根证书

~~~shell
openssl req -sha256 -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/C=CN/ST=Beijing/L=Beijing/O=thunisoft/OU=thunisoft/CN=thunisoft.com/emailAddress=thunisoft@thunisoft.com"
~~~

​	需要验证上一步填的根证书密码。rootca

​	这一步会生成`ca.crt`文件

#### 2.1.3.创建创建 `/etc/pki/CA/index.txt`文件

~~~shell
touch /etc/pki/CA/index.txt
~~~

​	该文件不存在在签名服务器证书时会报错

#### 2.1.4.创建颁发证书的序列号

~~~shell
echo "01" > /etc/pki/CA/serial
~~~

### 2.2.生成服务器证书

#### 2.2.1.生成服务器证书密钥

~~~shell
openssl genrsa -des3 -out server.key 2048
~~~

​	需要输入密码，此密码很重要，后面签名证书和修改配置文件时需要用到（**建议所有服务器证书的密码都设置一样的**）。例如：server

​	这一步会生成server.key文件

#### 2.2.2.创建证书签名请求

~~~shell
openssl req -new -key server.key -sha256 -out server.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=thunisoft/OU=thunisoft/CN=bjcm/emailAddress=bjcm@thunisoft.com"
~~~

​	需要验证上一步输入的密码。例如：server

​	这一步会生成server.csr文件

#### 2.2.3.附加配置（签发IP证书需要）

~~~shell
vi ip.ext
~~~

填写下面的内容，根据需要修改

~~~text
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName=@SubjectAlternativeName

[SubjectAlternativeName]
##要给哪些ip签发证书就按照IP.n的格式填写
IP.1=172.16.34.106
IP.2=172.16.14.176
##要给哪些域名签发证书就按照DNS.n的格式填写
DNS.1=www.thunisoft.com

~~~

​	这一步会生成ip.ext文件

#### 2.2.4.签发证书

~~~shell
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -extfile ip.ext -CAcreateserial -CAserial ca.seq -sha256 -out server.crt
~~~

​	需要验证2.1.1输入的根证书密码：rootca

​	这一步会生成server.crt文件

### 2.3.格式转换

TAS需要jks格式的服务器证书

#### 2.3.1.转换crt证书为pfx证书

~~~shell
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt
~~~

​	需要先验证server.key的密码（2.2.1中输入的密码），然后再设置一个导出密码（**建议所有服务器证书的密码都设置一样的**）。例如：server

​	这一步会生成server.pfx文件

#### 2.3.2.转换pfx证书为jks证书

~~~shell
keytool -importkeystore -srckeystore  server.pfx -srcstoretype pkcs12 -destkeystore server.jks -deststoretype JKS
~~~

​	需要先输入密钥库密码，然后再验证pfx证书的密码（上一步设置的密码）。（**建议所有服务器证书的密码都设置一样的**）。例如：server

这一步会生成server.jks文件

## 3.修改配置文件

### 3.1.拷贝证书到TAS（2.8.6）

​	把上面转换完成的server.jks放入`TAS_HOME/etc`目录下（TAS_HOME表示tas的安装目录）

### 3.2.加密证书密码

1. ​	进入TAS/lib/server目录，找到jetty-util-xxxx.jar的文件

   ~~~shell
   ls lib | grep jetty.util
   ~~~

   ![image-20200310105159562](F:/notes/Linux/HTTPS证书申请/img/jetty-util.png)

2. 使用jetty提供的加密方法对证书的密码进行加密

~~~shell
## java -cp lib/jetty-util-9.4.19.v20190610.jar org.eclipse.jetty.util.security.Password 你server证书密码

/usr/local/jdk/jdk1.8.0_77/java -cp lib/jetty-util-9.4.19.v20190610.jar org.eclipse.jetty.util.security.Password 你server证书密码
~~~

​	需要注意：2.8.6版本的TAS必须使用jdk1.8以上的环境，所以上面的命令中指定了1.8.的jdk

### 3.3.修改配置文件

​	修改`TAS_HOME/start.ini`文件（TAS_HOME表示tas的安装目录）

​	放开下面几项的注释，并修改其中的配置

#### 3.3.1.配置证书信息

```text
--module=https
##服务器证书路径
jetty.sslContext.keyStorePath=etc/server.jks
##服务器信任的证书（颁发客户端的CA证书）路径
jetty.sslContext.trustStorePath=etc/server.jks
##服务端证书密码
jetty.sslContext.keyStorePassword=OBF:1yf41t331z0b1z0j1t331yf2
##服务端证书密码
jetty.sslContext.keyManagerPassword=OBF:1yf41t331z0b1z0j1t331yf2
####服务器信任的证书密码（颁发客户端的CA证书密码）
jetty.sslContext.trustStorePassword=OBF:1yf41t331z0b1z0j1t331yf2
```

#### 3.3.2.修改HTTPS端口

​	配置文件为：`TAS_HOME/conf/server/tas-server.xml`

~~~text
<Call name="add">
        <Arg>
          <New class="com.thunisoft.tas.core.model.server.PortInfo">
            <Set name="id">web_ssl</Set>
            <Set name="name">web_ssl</Set>
            <Set name="desc">ssl监听端口</Set>
            <!-- 此处修改端口 -->
            <Set name="port">20089</Set>
          </New>
        </Arg>
      </Call>

~~~

### 3.4.重启TAS

## 4.添加信任

### 4.1.添加浏览器信任

​	把**2.1**中生成的ca.crt文件拷贝到电脑上，双击打开安装证书，安装位置选择`受信任的根证书颁发机构`

### 4.2.添加服务器信任

​	**注意**：A服务器安装了SSL证书，B服务器通过`HTTS`调用A服务器接口时,需要信任根证书(**ca.crt**)，下面的安装步骤中的服务器指的是B服务器（也可以所有服务器都添加根证书信任）。

​			服务器上可能有多个java环境（多个jdk），在无法确认程序使用的是那个环境的情况下，最好每个java环境都把证书导入一遍（步骤相同，只需要把`$JAVA_HOME`改为改jdk的安装目录即可）。

​	**证书导入jdk命令**：

~~~shell
##	keytool -import -alias <证书别名> -keystore $JAVA_HOME/jre/lib/security/cacerts -file <证书路径>

keytool -import -alias Thunisoft -keystore $JAVA_HOME/jre/lib/security/cacerts -file /opt/ca.crt
~~~

​	**密钥库口令**：changeit

## 5.附录

​	有不明白的概念或者想看每一步的操作结果请看

[openssl证书申请]: ./openssl证书申请.md

