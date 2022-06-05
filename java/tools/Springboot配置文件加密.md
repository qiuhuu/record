# Jasypt 加密

​		Jasypt 也即Java Simplified Encryption是Sourceforge.net上的一个开源项目

​		根据Jasypt文档，该技术可用于加密任务与应用程序，例如加密密码、敏感信息和数据通信、创建完整检查数据的sums. 其他性能包括高安全性、基于标准的加密技术、可同时单向和双向加密的加密密码、文本、数字和二进制文件。Jasypt也可以与Acegi Security整合也即Spring Security。Jasypt亦拥有加密应用配置的集成功能，而且提供一个开放的API从而任何一个Java Cryptography Extension都可以使用Jasypt。

​	Jasypt还符合RSA标准的基于密码的加密，并提供了无配置加密工具以及新的、高可配置标准的加密工具。

1、该开源项目可用于加密任务与应用程序，例如加密密码、敏感信息和数据通信

2、还包括高安全性、基于标准的加密技术、可同时单向和双向加密的加密密码、文本、数字和二进制文件。

3、Jasypt还符合RSA标准的基于密码的加密，并提供了无配置加密工具以及新的、高可配置标准的加密工具。

4、加密属性文件（encryptable properties files）、Spring work集成、加密Hibernate数据源配置、新的命令行工具、URL加密的Apache wicket集成以及升级文档。

5、Jasypt也可以与Acegi Security整合也即Spring Security。Jasypt亦拥有加密应用配置的集成功能，而且提供一个开放的API从而任何一个Java Cryptography Extension都可以使用Jasypt。

能够实现对明文进行加密，对密文进行解密。配置相关加密信息，就能够实现在项目运行的时候，自动把配置文件中已经加密的信息解密成明文，供程序使用



## Jasypt 配置详解

Jasypt 默认使用 StringEncryptor 解密属性，若是在 Spring 上下文中找不到自定义的 StringEncryptor，则使用以下默认值：

| 配置属性                                  | 是否必填项 | 默认值                              |
| :---------------------------------------- | :--------- | :---------------------------------- |
| jasypt.encryptor.password                 | **True**   | -                                   |
| jasypt.encryptor.algorithm                | False      | PBEWITHHMACSHA512ANDAES_256         |
| jasypt.encryptor.key-obtention-iterations | False      | 1000                                |
| jasypt.encryptor.pool-size                | False      | 1                                   |
| jasypt.encryptor.provider-name            | False      | SunJCE                              |
| jasypt.encryptor.provider-class-name      | False      | null                                |
| jasypt.encryptor.salt-generator-classname | False      | org.jasypt.salt.RandomSaltGenerator |
| jasypt.encryptor.iv-generator-classname   | False      | org.jasypt.iv.RandomIvGenerator     |
| jasypt.encryptor.string-output-type       | False      | base64                              |
| jasypt.encryptor.proxy-property-sources   | False      | false                               |
| jasypt.encryptor.skip-property-sources    | False      | empty list                          |

*若是想使用 “PBEWITHHMACSHA512ANDAES_256” 算法，须要 Java JDK 1.9 及以上支持，或者添加* [JCE 无限强度权限策略文件](http://www.javashuo.com/link?url=https://wangmaoxiong.blog.csdn.net/article/details/106170060#JCE 无限强度权限策略文件)，不然运行会报错：加密引起异常，一个可能的缘由是您正在使用强加密算法，而且您没有在这个Java虚拟机中安装Java加密扩展（JCE）无限强权限策略文件。*

## 使用说明

### 命令行使用

#### 1.命令行加密

进入maven仓库：org\jasypt\jasypt\1.9.3 ，版本根具自己仓库里jar包版本而定

`java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input=【待加密的数据】  password=【密码】 algorithm=PBEWithMD5AndDES`



```shell
D:\Java\maven-repos\org\jasypt\jasypt\1.9.3>java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input=root  password=123455 algorithm=PBEWithMD5AndDES

----ENVIRONMENT-----------------

Runtime: Oracle Corporation Java HotSpot(TM) 64-Bit Server VM 11.0.7+8-LTS



----ARGUMENTS-------------------

input: root
password: 123455
algorithm: PBEWithMD5AndDES



----OUTPUT----------------------

KJhFvdLA/+rlnjs87R7BqA==
```

OUTPUT：root字符串 加密后的密码



#### 2.命令行解密

`java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringDecryptionCLI input=【待加密的数据】 password=【密码】 algorithm=PBEWithMD5AndDES`

注意：加密方式`algorithm`需要保持一致

```
D:\Java\maven-repos\org\jasypt\jasypt\1.9.3>java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringDecryptionCLI input=KJhFvdLA/+rlnjs87R7BqA== password=123455 algorithm=PBEWithMD5AndDES

----ENVIRONMENT-----------------

Runtime: Oracle Corporation Java HotSpot(TM) 64-Bit Server VM 11.0.7+8-LTS



----ARGUMENTS-------------------

input: KJhFvdLA/+rlnjs87R7BqA==
password: 123455
algorithm: PBEWithMD5AndDES



----OUTPUT----------------------

root
```



### SpringBoot使用

#### 1.pom引入依赖

```xml
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.4</version>
</dependency>
```

#### 2.配置文件application.yaml

```yaml
jasypt:
  encryptor:
    #password: ${user.pwd:654321}  #获取系统环境变量${}
    #用来加密的salt值
    password: 123  #获取系统环境变量${}
    #版本默认：PBEWITHHMACSHA512ANDAES_256
    algorithm: PBEWithMD5AndDES #指定其他解密算法
    #新版本默认为org.jasypt.salt.RandomIVGenerator
    ivGeneratorClassname: org.jasypt.salt.RandomIVGenerator
    
```

#### 3.使用工具类进行加密字符串

```java
@SpringBootTest
class TestApplicationTests {

    @Autowired
    private StringEncryptor stringEncryptor;
    @Test
    void en(){
        String root = stringEncryptor.encrypt("root");
        String password = stringEncryptor.encrypt("password");
        System.out.println(root+":"+password);
    }
}


jJhTShPyy0QQXT4Tcioxp9hslwTzZyIin7qu7ZEFibiWi/4bCp1j0qLBP3TrO49i:UZX7n5kWovZyLHR6kPs7Mi2FaaebO/ybJGnSEhP84x3213NVt1VXUX1XT9WWRyui
```

上面的加密方式是采用`PBEWITHHMACSHA512ANDAES_256`

#### 4.在配置文件中使用

```yaml
user:
  username: ENC(jJhTShPyy0QQXT4Tcioxp9hslwTzZyIin7qu7ZEFibiWi/4bCp1j0qLBP3TrO49i)
  password: ENC(UZX7n5kWovZyLHR6kPs7Mi2FaaebO/ybJGnSEhP84x3213NVt1VXUX1XT9WWRyui)
```

注意：加密字符串的使用需要加上 ENC() 配套使用，也可以自定义前后缀

```yaml
jasypt:
 encryptor:
  property:
   prefix: "ENC{"
   suffix: "}"
```

此时，配置文件中，加密字符串应被ENC{}包含

```yaml
user:
  username: ENC{jJhTShPyy0QQXT4Tcioxp9hslwTzZyIin7qu7ZEFibiWi/4bCp1j0qLBP3TrO49i}
  password: ENC{UZX7n5kWovZyLHR6kPs7Mi2FaaebO/ybJGnSEhP84x3213NVt1VXUX1XT9WWRyui}
```



#### 额外配置

`@EnableEncryptableProperties`： 表示启用加密配置文件
`@EncryptablePropertySource` ：用来设置加密配置文件的路径
`@EncryptablePropertySources` ：如果有多个加密配置文件，则用花括号括起来，中间用逗号隔开每一个@EncryptablePropertySource



```
@EnableEncryptableProperties
@EncryptablePropertySource(value="classpath:/encrypt.yml")
//@EncryptablePropertySources({@EncryptablePropertySource("classpath:/encrypt.yml")})
```



## 注意

本身加解密过程都是通过`盐值`进行处理的，所以正常情况下<font color='red'>`盐值`和`加密串`是分开存储的。</font><font color='red'>**`盐值`应该放在`系统属性`、`命令行`或是`环境变量`来使用，而不是放在配置文件。**</font>

在生产环境中，希望通过一下几种方式来实现加密：

### 1、命令行

在启动jar包的命令中，加入密码配置

```bash
java -jar xxx.jar --jasypt.encryptor.password=xxx 
```

### 2、环境变量示例

设置环境变量：

```bash
# 打开/etc/profile文件
vi /etc/profile
# 文件末尾插入
export JASYPT_PASSWORD = xxxx
#使环境变量生效
source /etc/profile
```

注意：修改完成后不会马上生效。立即生效请输入命令：`source /etc/profile`

启动命令：

```shell
java -jar xxx.jar --jasypt.encryptor.password=${JASYPT_PASSWORD}
```



**其他配置方式，请自行琢磨。**