<center><h1>SSL证书申请</h1></center>

## 1. 阿里云免费证书申请

1.1. 网址：https://www.aliyun.com/

1.2.  进入证书购买页面

![image-20200306110607607](.\img\index.png)

1.3.  按步骤选择并购买（免费证书0元）

![image-20200306111003577](.\img\buy.png)

1.4. 点击证书申请

![image-20200306114442125](.\img\apply.png)

1.5 填写信息(**必须是公网ip或者域名**)

![image-20200306135252361](.\img\ali_edit.png)

1.6 验证

![image-20200306135507758](.\img\ali_validate.png)

## 2. openssl自制证书

2.1. 创建密钥

~~~shell
openssl genrsa -des3 -out server.key 2048
~~~

注意：生成私钥，需要提供一个至少4位，最多1023位的密码。

2.2. 生成CSR（证书签名请求）

~~~shell
openssl req -new -key server.key -out server.csr -config /etc/pki/tls/openssl.cnf
~~~

说明：需要依次输入国家，地区，城市，组织，组织单位，Common Name和Email。其中Common Name，可以写自己的名字或者域名，如果要支持https，Common Name应该与域名保持一致，否则会引起浏览器警告。

​	可以将证书发送给证书颁发机构（CA），CA验证过请求者的身份之后，会出具签名证书，需要花钱。另外，如果只是内部或者测试需求，也可以使用OpenSSL实现自签名。

2.3. 删除密钥中的密码

~~~shell
openssl rsa -in server.key -out
~~~

说明：如果不删除密码，在应用加载的时候会出现输入密码进行验证的情况，不方便自动化部署。

