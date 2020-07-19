# tomcat 配置https

#### 使用使用java自带keytool创建本地密钥库

创建本地密钥库命令:

    keytool -genkey -alias testkey -keyalg RSA -keystore "C:\Program Files\Java\Testkey"
    or
    keytool -genkey -alias caskeystore -keypass 123456 -keyalg RSA -keystore thekeystore
    
* -alias ： 这里的testkey是别名，根据个人自定义
* -keystore  ：这里是声明生成key的位置（稍后会用到）

输入上述命令后，会让你输入密钥库口令（记住密码）


