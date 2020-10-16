# license的使用

## 一、生成密匙对

1. 首先要用KeyTool工具来生成密钥库：（-alias别名 –validity 3650表示10年有效） keytool -genkey -alias privatekeys -keysize 1024 -keystore privateKeys.store -validity 3650
2. 然后将密钥库中名称为‘privatekeys’的证书条目导出到证书文件certfile.cer中： keytool -export -alias privatekeys -file certfile.cer -keystore privateKeys.store
3. 然后再把这个证书文件的信息导入到公钥库中别名为publiccert的证书条目中： keytool -import -alias publiccert -file certfile.cer -keystore publicCerts.store
4. 最后生成的文件privateKeys.store和publicCerts.store拷贝出来备用。
5. 密码要一致。

## 二、下载licenseDemo，生成lic文件

1. github地址：https://github.com/kobeyk/license  自己fork的项目地址：https://github.com/git-tree/license

2. 把privateKeys.store，放在license-create的resource目录下

3. 把publicCerts.store，放在license-verify的resource目录下

4. 修改license-app目录下的application.properties文件，把crater取消注释，verify注释(先创建，后验证)

5. 启动项目(启动文件在license-app)里面

6. 打开postman，调用接口获取本机硬件信息，接口地址:localhost:8080/license/getServerInfos 请求方式为get请求,请求成功后可以看到本机硬件信息

7. ```json
   {
       "status": 200,
       "message": "成功",
       "data": {
           "ipAddress": [
               "172.16.28.6",
               "192.168.75.1",
               "192.168.139.1"
           ],
           "macAddress": [
               "BC-30-5B-AC-3E-DF",
               "00-50-56-C0-00-01",
               "00-50-56-C0-00-08"
           ],
           "cpuSerial": "BFEBFBFF0001067A",
           "mainBoardSerial": "..CN137400BQ023X.",
           "registerAmount": null,
           "macCheck": false,
           "ipCheck": false,
           "cpuCheck": false,
           "boardCheck": false,
           "registerCheck": false
       },
       "timeStamp": "2020-10-16 13:55:10"
   }
   ```

8. postman调用生成接口：127.0.0.1:8080/license/generate  方式为post请求，参数为json格式，参数如下：

9. ```json
   {
       "subject": "landi",
       "privateAlias": "privateKeys",
       "keyPass": "123456a",
       "storePass": "123456a",
       "privateKeysStorePath": "/privateKeys.store",
       "issuedTime": "2020-10-16 08:30:00",
       "expiryTime": "2021-10-16 08:30:00",
       "description": "系统软件许可证书",
       "licenseCheck": {
          "ipAddress": [
               "172.16.28.6",
               "192.168.75.1",
               "192.168.139.1"
           ],
           "macAddress": [
               "BC-30-5B-AC-3E-DF",
               "00-50-56-C0-00-01",
               "00-50-56-C0-00-08"
           ],
           "cpuSerial": "BFEBFBFF0001067A",
           "mainBoardSerial": "..CN137400BQ023X.",
           "registerAmount": 1000,
           "macCheck": false,
           "boardCheck": false,
           "cpuCheck": false,
           "ipCheck": false,
           "registerCheck": true
       }
   }
   ```

10. 修改参数信息，时间、密码、等等。（postman body需要修改为raw，并选择json）

11. 请求成功后，会返回如下信息：

12. ```json
    {
        "status": 200,
        "message": "成功",
        "data": {
            "subject": "landi",
            "privateAlias": "privateKeys",
            "keyPass": "123456a",
            "privateKeysStorePath": "/privateKeys.store",
            "storePass": "123456a",
            "licensePath": "E:\\licdemo/license/20201016141901/license.lic",
            "issuedTime": "2020-10-16 08:30:00",
            "expiryTime": "2021-10-16 08:30:00",
            "consumerType": "user",
            "consumerAmount": 1,
            "description": "系统软件许可证书",
            "licenseCheck": {
                "ipAddress": [
                    "172.16.28.6",
                    "192.168.75.1",
                    "192.168.139.1"
                ],
                "macAddress": [
                    "BC-30-5B-AC-3E-DF",
                    "00-50-56-C0-00-01",
                    "00-50-56-C0-00-08"
                ],
                "cpuSerial": "BFEBFBFF0001067A",
                "mainBoardSerial": "..CN137400BQ023X.",
                "registerAmount": 1000,
                "macCheck": false,
                "ipCheck": false,
                "cpuCheck": false,
                "boardCheck": false,
                "registerCheck": true
            },
            "licUrl": "http://localhost:8066/license/download?path=E:\\licdemo/license/20201016141901/license.lic"
        },
        "timeStamp": "2020-10-16 14:19:01"
    }
    ```

## 三、生成的lic添加到自己的项目中

1. 自己项目添加maven依赖：

2. ```xml
   		<dependency>
               <groupId>com.appleyk.spring.boot</groupId>
               <artifactId>license-verify-spring-boot-starter</artifactId>
               <version>0.2.1-SNAPSHOT</version>
           </dependency>
   ```

3. 修改自己项目配置文件(注意账号密码)

4. ```yaml
   springboot:
     license:
       verify:
         # 证书名称
         subject: haifeng20200927
         # 公钥库证书条目别名
         publicAlias: publiccert
         # 证书公钥库存储路径(公钥库可以公开出去，但是私有密钥库一定要自己保存好，即私有密钥库要保存在creator模块中)
         publicKeysStorePath: /publicCerts.store
         # 证书公钥库访问密码
         storePass: 123456a
         # 证书存放路径
         licensePath: classpath:license.lic
   ```

5. 启动项目，查看是否有安装证书成功log 打印

6. ```
   destination(s)], registry[0 sessions]]]]
   14:52:59 [restartedMain] INFO  o.s.m.s.b.SimpleBrokerMessageHandler - Started.
   14:52:59 [restartedMain] INFO  com.appleyk.core.helper.LoggerHelper - ++++++++ 开始安装证书 ++++++++
   14:53:02 [restartedMain] INFO  com.appleyk.core.helper.LoggerHelper - 证书安装成功，证书有效期：2020-10-16 08:30:00 - 2021-10-16 08:30:00
   14:53:02 [restartedMain] INFO  com.appleyk.core.helper.LoggerHelper - ++++++++ 证书安装成功 ++++++++
   14:53:02 [restartedMain] INFO  o.a.coyote.http11.Http11NioProtocol - Starting ProtocolHandler ["http-nio-80"]
   14:53:02 [restartedMain] INFO  o.a.tomcat.util.net.NioSelectorPool - Using a shared selector for servlet write/read
   ```

7. 需要的方法上添加注解 @VLicense

8. ```java
    @GetMapping("/login")
    @VLicense
       String login(Model model) {
           model.addAttribute("username", bootdoConfig.getUsername());
           model.addAttribute("password", bootdoConfig.getPassword());
           return "login";
       }
   ```

