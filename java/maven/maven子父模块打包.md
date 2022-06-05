### maven子父模块打包

目录结构如下

```lua
main
├── common                               -- 公共模块
	├── pom.xml
├── test                            	 -- test模块
	├── pom.xml
├── pom.xml

```



common公共引用模块

test 引用common模块



main 下面的pom.xml

```xml
	<modelVersion>4.0.0</modelVersion>
    <groupId>com.xxx</groupId>
    <artifactId>xxx-main</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>   <!-- 父模块引用 使用pom -->
    <name>xxx-mian</name>

    <modules>
        <module>xxx-common</module>
        <module>xxx-test</module>
    </modules>


	<build>
        <resources>
            <!-- 指定 src/main/resources 下所有文件及文件夹为资源文件 -->
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
        <plugins>
            <!--<plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven-compiler.version}</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>${unified.encoding}</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>${maven-resources.version}</version>
            </plugin>
            <!-- 打包跳过测试文件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${maven-surefire.version}</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>

        </plugins>
    </build>
```

父模块中去掉`spring-boot-maven-pluging`打包插件



common pom.xml

```xml
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.xxx</groupId>
        <artifactId>xxx-main</artifactId>
        <version>1.0.0</version>
    </parent>
    <artifactId>xxx-common</artifactId>
    <name>xxx-commonr</name>
    <packaging>jar</packaging> <!-- 模块使用jar -->
    <description>通用组件启动器</description>
```



test pom.xml

```xml
	<parent>
        <artifactId>xxx-main</artifactId>
        <groupId>com.xxx</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>xxx-test</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    
    <build>
        <!-- 打包名称 -->
        <!-- 需要打包的jar引用maven打包插件 -->
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

