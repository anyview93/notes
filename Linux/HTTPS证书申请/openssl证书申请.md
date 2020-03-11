<center><h1>SSL证书申请</h1></center>
## 1. openssl自签证书

### 1.1. 自建CA

#### 1.1.1.生成根密钥

~~~shell
openssl genrsa -des3 -out ca.key 4096
~~~

注意：生成私钥，需要提供一个至少4位，最多1023位的密码（此密码后面需要多次用到）。

![image-20200310094633810](.\img\cakey.png)

#### 1.1.2.生成根 CA 证书

~~~shell
openssl req -sha256 -new -x509 -days 3650 -key ca.key -out ca.crt
~~~

- **sha256**:指定加密算法为sha256（默认为sha1,谷歌浏览器会提示证书为弱算法）
- **x509**:证书格式（只有公钥，不包含私钥）
- **days**：证书有效天数
- **key**：指定密钥文件
- **out**：指定输出文件

需要填写一些信息

- **Country Name** ：国家名称简写，填`CN`
- **State or Province Name**:省份名称。填`Beijing`
- **Locality Name**：所在位置名称/城市名称。填`Beijing`
- **Organization Name**：机构名称。填`Thunisoft`
- **Organizational Unit Name**:组织单位名称。填`Thunisoft`
- **Common Name**：通用名称。可以填域名，IP或其他名称。填`Thunisoft`
- **Email Address** ：证书使用者email，填邮箱地址

![image-20200310095334552](.\img\cacrt.png)



### 1.2. 颁发证书

#### 1.2.1.生成服务端密钥文件

~~~shell
openssl genrsa -des3 -out server.key 2048
~~~

注意：生成私钥，需要提供一个至少4位，最多1023位的密码（这是server证书的密码，跟上面ca证书的密码没有关系）。

![image-20200310095720107](.\img\server_key.png)

#### 1.2.2.创建证书签名请求

~~~shell
openssl req -new -key server.key -sha256 -out server.csr
~~~

- **Country Name** ：国家名称简写，填`CN`

- **State or Province Name**:省份名称。填`Beijing`

- **Locality Name**：所在位置名称/城市名称。填`Beijing`

- **Organization Name**：机构名称。填`Thunisoft`

- **Organizational Unit Name**:组织单位名称。填`Thunisoft`

- **Common Name**：通用名称。可以填域名，IP或服务器名称

- **Email Address** ：证书使用者email，填邮箱地址

- A challenge password：为证书设置密码（私钥保护密码），设置一个密码（可以不填）（没发现这个密码的用处，可能是1.2.3中把它给清除了）

- An optional company name：可选的公司名称（可以不填）。填`Thunisoft`

  说明：需要依次输入国家，地区，城市，组织，组织单位，Common Name和Email。其中State or Province Name和 Organization Name必须和CA填写的信息一致，否则无法签名。

![image-20200310100128034](.\img\server_csr.png)

查看证书请求文件

~~~shell
openssl req -in server.csr -text
~~~

![image-20200311183151373](.\img\view_csr.png)

#### 1.2.3. 删除密钥中的密码

~~~shell
openssl rsa -in server.key -out server.key
~~~

说明：如果不删除密码，在应用加载的时候会出现输入密码进行验证的情况，不方便自动化部署。

这里需要输入在**1.2.1**中输入的密码

![image-20200310100308157](.\img\server_rmkey.png)

#### 1.2.4.附加用途（签发IP证书）

创建一个额外的配置文件，下面签发证书的时候需要指定此配置文件

~~~shell
vi ip.ext
~~~

~~~text
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName=@SubjectAlternativeName

[SubjectAlternativeName]
IP.1=172.16.34.106
IP.2=172.16.14.176
~~~

与签发域名证书的区别就在于此步骤，在 **不改 openssl.cnf 的情况** （方便签发不同证书）下如果是要签发 IP 证书**必须**参照上述格式执行此步骤。

如果要通过 **修改 openssl.cnf** 来签发证书，除将上述配置直接改到 openssl.cnf 相应位置外，必须将配置中的`basicConstraints = CA:FLASE` 改为 `basicConstraints = CA:TRUE`，否则修改不生效。

如果是域名证书，也可以在此可以添加`多域名`，如：

~~~text
...
subjectAltName=@SubjectAlternativeName

[ SubjectAlternativeName ]
DNS.1 = www.local.domain
DNS.2 = localdomain
IP.1 = 1.1.1.1
IP.2 = 127.0.0.1
~~~

#### 1.2.5.签发证书

~~~shell
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -extfile ip.ext -CAcreateserial -CAserial ca.seq -sha256 -out server.crt
~~~

##### 1.2.5.1.创建创建 `/etc/pki/CA/index.txt`文件

**注意**：只在签发第一张证书的时候执行即可（签发证书报错再执行这一步）0

~~~shell
touch /etc/pki/CA/index.txt
~~~

该文件不存在会出现下图中的错误：

![image-20200310101337171](.\img\ca_index.png)

##### 1.2.5.2.创建最后一次颁发证书的序列号

**注意**：只在签发第一张证书的时候执行即可（签发证书报错再执行这一步）

~~~shell
echo "01" > /etc/pki/CA/serial
~~~

否则会出现下图两个错误：

![image-20200310101539467](.\img\serial_err.png)

##### 1.2.5.3.签名成功

解决上面两个问题后，再次执行签名命令

~~~shell
##openssl ca -keyfile ca.key -cert ca.crt -in server.csr -extfile ip.ext -days 365 -out server.crt
openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -extfile ip.ext -CAcreateserial -CAserial ca.seq -sha256 -out server.crt
~~~

![image-20200310102431844](.\img\sign_success.png)

##### 1.2.5.4.查看证书

~~~shell
openssl x509 -in server.crt -text
~~~

![image-20200311182942382](.\img\view_crt.png)

### 1.3.证书格式转换

java环境需要jks格式证书

##### 1.3.1.转换crt证书为pfx证书

~~~shell
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt
~~~

需要设置一个导出密码:

![image-20200310102615084](.\img\pfx_pwd.png)

##### 1.3.2.转换pfx证书为jks证书

~~~shell
keytool -importkeystore -srckeystore  server.pfx -srcstoretype pkcs12 -destkeystore server.jks -deststoretype JKS
~~~

需要设置密码（**必须和上一步设置的导出密码相同**，否则应用启动时会出现密码解析失败的错误），并且需要校验源文件密钥(**上一步设置的导出密码**)
![image-20200310102731580](.\img\jks_pwd.png)

### 1.4.TAS（2.8.6）安装证书

#### 1.4.1.把生成的证书放入TAS

​	把上面转换完成的server.jks放入`TAS_HOME/etc`目录下（TAS_HOME表示tas的安装目录）

#### 1.4.2.生成TAS支持的证书加密密码（下一步需要用到）

1. ​	进入TAS/lib/server目录，找到jetty-util-xxxx.jar的文件

   ~~~shell
   ls lib | grep jetty.util
   ~~~

   ![image-20200310105159562](.\img\jetty-util.png)

2. 使用jetty提供的加密方法对证书的密码进行加密

~~~shell
## java -cp lib/jetty-util-9.4.19.v20190610.jar org.eclipse.jetty.util.security.Password 你server证书密码

/usr/local/jdk/jdk1.8.0_77/java -cp lib/jetty-util-9.4.19.v20190610.jar org.eclipse.jetty.util.security.Password 你server证书密码
~~~

需要注意：2.8.6版本的TAS必须使用jdk1.8以上的环境，所以上面的命令中指定了1.8.的jdk

![image-20200310105600583](.\img\enc_pwd.png)

#### 1.4.3.修改配置文件

​	修改`TAS_HOME/start.ini`文件（TAS_HOME表示tas的安装目录）

​	去掉下面几项的注释，并修改其中的配置

~~~text
--module=https
jetty.sslContext.keyStorePath=etc/server.jks
jetty.sslContext.trustStorePath=etc/server.jks
jetty.sslContext.keyStorePassword=OBF:1yf41t331z0b1z0j1t331yf2
jetty.sslContext.keyManagerPassword=OBF:1yf41t331z0b1z0j1t331yf2
jetty.sslContext.trustStorePassword=OBF:1yf41t331z0b1z0j1t331yf2
~~~

- **keyStorePath**：1.4.1中放到`TAS_HOME/etc`目录下的server.jks路径
- **trustStorePath**：1.4.1中放到`TAS_HOME/etc`目录下的server.jks路径
- **keyStorePassword**：1.4.2中加密后的密码
- **keyManagerPassword**：1.4.2中加密后的密码
- **trustStorePassword**：1.4.2中加密后的密码

![image-20200310110233639](.\img\vi_cnf.png)

#### 1.4.4.修改SSL端口

##### 1.4.4.1.方式一：控制台修改端口

```text
http://ip:port/tas-console

用户名：tas2admin
密码：tusc@99
```

##### 1.4.4.2.方式二：修改配置文件

​	配置文件为：`TAS_HOME/conf/server/tas-server.xml`

![image-20200310150806111](.\img\cnf_port.png)

### 1.5.验证证书

#### 1.5.1.重启TAS

~~~shell
##关闭TAS
sh bin/StopTAS.sh
##启动TAS
sh bin/StartTAS.sh
~~~

#### 1.5.2.页面访问验证

需要开放所使用的端口或者关闭防火墙

~~~href
https://ip:port
~~~

- **ip**:配置证书的服务器ip
- **port**:SSL连接开放的端口

![image-20200310133447145](.\img\view_err.png)

出现上面的图片表示证书配置成功，浏览器显示不安全是因为需要把根证书(**CA**)添加到`受信任的根证书颁发机构`

### 1.6.添加信任

#### 1.6.1.客户端添加信任(客户电脑)

1. 把1.1.2生成的根证书(**ca.crt**)下载到桌面，双击打开,然后点击安装证书

   ![image-20200310134541822](.\img\ca_install_1.png)

2. 选择本地计算机，点击下一步

   ![image-20200310134946684](.\img\ca_install_2.png)

3. 按下图步骤，选择`受信任的根证书颁发机构`

   ![image-20200310135421740](.\img\ca_install_3.png)

4. 点击下一步

   ![image-20200310135613710](.\img\ca_install_4.png)

5. 点击完成，显示导入成功

   ![image-20200310135806202](.\img\ca_install_5.png)

6. 验证导入情况

   重新访问1.5.2中的连接

~~~href
https://ip:port
~~~

此时页面访问正常。页面无警告，地址栏显示小锁标志

![image-20200310142829525](.\img\view_success.png)

#### 1.6.2.服务端添加信任(服务器之间互信)

**注意**：A服务器安装了SSL证书，B服务器通过`HTTS`调用A证书接口时,需要信任根证书(**ca.crt**)，下面的安装步骤中的服务器指的是B服务器（也可以所有服务器都添加根证书信任，没有影响）。

​			服务器上可能有多个java环境（多个jdk），在无法确认程序使用的是那个环境的情况下，最好每个java环境都把证书导入一遍（步骤相同，只需要把`$JAVA_HOME`改为改jdk的安装目录即可）。

------



1. 把1.1.2生成的根证书(**ca.crt**)上传到服务器

   例如把**ca.crt**上传到/opt/目录下

   ![image-20200310155539814](.\img\upload_CA.png)

2. 查看当前java环境所信任的证书

   ~~~shell
   keytool -list -keystore $JAVA_HOME/jre/lib/security/cacerts
   ~~~

   **密钥库口令**：changeit

   ![image-20200310160842946](.\img\importCA_before.png)

   可以看到，导入之前java环境所信任证书有92条目

3. 导入证书

   ~~~shell
   ##	keytool -import -alias <证书别名> -keystore $JAVA_HOME/jre/lib/security/cacerts -file <证书路径>
   
   keytool -import -alias Thunisoft -keystore $JAVA_HOME/jre/lib/security/cacerts -file /opt/ca.crt
   ~~~

   **密钥库口令**：changeit

   ![image-20200310160429037](.\img\importCA.png)

4. 查看导入后信任证书条目

   ~~~shell
   keytool -list -keystore $JAVA_HOME/jre/lib/security/cacerts
   ~~~

   **密钥库口令**：changeit

   ![image-20200310160753793](.\img\upload_CA_after.png)

   可以看到，现在密钥库包含93个条目，刚刚导入的那条已经存在了

5. 证书删除命令（**备用，正常操作不需要执行**）

   ~~~shell
   ##	keytool -delete -alias <证书别名> -keystore $JAVA_HOME/jre/lib/security/cacerts -file
   ~~~

   **密钥库口令**：changeit

6. HttpURLConnection调用接口验证

   上面的导入步骤都是在服务器上操作的，但为了测试方便，测试的时候是在本地测试的（重新在本地又导入了一遍ca.crt证书），用jdk自带的`HttpURLConnection`进行模拟调用测试

   - `GET`接口测试

     ![image-20200310165117648](.\img\test_get.png)

   - POST接口测试

     ![image-20200310165222924](.\img\test_post.png)

#### 1.6.3.前后端分离（不需要修改）

​	**前后端分离项目中，前端使用ajax调用后端的SSL接口，不需要额外添加信任**

- 测试代码

  ~~~html
  <!DOCTYPE html>
  <html lang="zh-CN">
  <head>
  	<title>SSL前端调用测试</title>
  </head>
  <body>
  	<div id="app">
  		<h1>GET请求：{{getData}}</h1>
  		<h1>POST请求：{{postData}}</h1>
  	</div>
  </body>
  <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  <script src="https://cdn.staticfile.org/vue-resource/1.5.1/vue-resource.min.js"></script>
  <script type="text/javascript">
  	var vm = new Vue({
  		el: '#app',
  		data: function() {
  			return {
  				getData: '',
  				postData: ''
  				}
  		},
  		methods: {
  			getInterface:function(){
                  //发送get请求
                  this.$http.get('https://172.16.34.106:20089/demo/getData').then(function(res){
                  	console.log(res);
                  	this.getData=res.bodyText; 
                  },function(){
                      console.log('请求失败处理');
                  });
              },
              postInterface:function(){
              	//发送post请求
                  this.$http.post('https://172.16.34.106:20089/demo/postData').then(function(res){
                  	console.log(res);
                  	this.postData=res.bodyText;   
                  },function(){
                      console.log('请求失败处理');
                  });
              }
  		},
  		mounted: function(){
  			this.getInterface();
  			this.postInterface();
  		}
  	})	
  </script>
  </html>
  ~~~

  

- 测试结果如下：

![image-20200310173939387](.\img\html_result.png)