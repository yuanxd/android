android
=======
http://www.ikoding.com/build-android-project-with-maven/
使用Maven构建Android项目 2
3 四 2013   | 技术 Tags:android-maven-plugin · jarsigner · Maven archetype · proguard
之前一直在做WEB前端项目，前段时间接手第一个Android项目，拿到代码之后，先试着run起来再说，导入eclipse，一堆错误，设置classpath依赖，折腾半天，还是编译错误，于是联系项目接口人，得知他有一个Android库项目没有提交到SVN，晕。。。

对于习惯使用Maven管理Java项目的我来说，自然想到能否用Maven构建Android项目呢？于是开始Google、百度，发现已经有前人做过这样的实践了，不过在使用过程中还是遇到不少问题，后面经过各种努力终于能比较顺地使用了，这篇文章对如何使用Maven构建Android项目作了简要总结。如果你和我一样饱受项目依赖管理的折磨，和我一样讨厌项目打包发布时的繁琐，希望能通过Maven让这一切自动化完成。那么，这篇文章或许对你有用。

1. 环境搭建

JDK与Android SDK安装
做Android开发，这里无需多说，但安装完成后需要正确设置JAVA_HOME、CLASSPATH、ANDROID_HOME等环境变量。其中ANDROID_HOME为Android SDK安装的根目录。并将%ANDROID_HOME%\tools和%ANDROID_HOME%\platform-tools值添加到Path变量中。

Maven安装
这里无需多说，下载安装Maven并正确设置环境变量即可。

IDE支持
Eclipse
大多数人都在使用Eclipse开发Android应用，如果你用Eclipse做Android开发，推荐下载eclipse-with-m2e-and-adt，该版本已经安装了ADT、m2e、m2e-android等重要插件，支持在Eclipse中使用Maven进行Android应用开发。当时为了安装这些插件费了好大劲，所以，如果你看到这里，可以直接下载它，不用像我一样去做那些没有意义又浪费时间的事情。

IntelliJ IDEA
做Java开发，你不能不知道的神器，完美支持使用Maven构建Android应用，强烈推荐。即使是装了插件的Eclipse对使用Maven构建Android应用仍然支持不好。如果你是Eclipse的信徒，你也可以试试它，如果你能习惯它，你一定会被它的强大所吸引。

NetBeans IDE
NetBeans也支持Android开发，但没怎么了解，用的人应该也比较少。

2. 项目构建

以下是我的项目中使用的pom文件，因为涉及保密，部分地方做过修改，但整体结构没有改变，可以清楚地说明问题。

<?xml version="1.0" encoding="UTF-8"?>
<project 
   xmlns="http://maven.apache.org/POM/4.0.0" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.ikoding.android</groupId>
    <artifactId>android-app-quickstart</artifactId>
    <version>1.0.0</version>
    <packaging>apk</packaging>
    <name>Android Application Quick Satrt</name>

    <dependencies>
        <dependency>
            <groupId>com.google.android</groupId>
            <artifactId>android</artifactId>
            <version>4.1.1.4</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.google.android</groupId>
            <artifactId>support-v4</artifactId>
            <version>r7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.james</groupId>
            <artifactId>apache-mime4j</artifactId>
            <version>0.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpmime</artifactId>
            <version>4.2.2</version>
        </dependency>
        <dependency>
            <groupId>com.ikoding.android</groupId>
            <artifactId>android-base</artifactId>
            <version>2.0.0</version>
            <type>apklib</type>
        </dependency>
    </dependencies>

    <properties>
        <keystore.filename>app-quickstart.keystore</keystore.filename>
        <keystore.storepass>2013@ikoding</keystore.storepass>
        <keystore.keypass>2013@ikoding</keystore.keypass>
        <keystore.alias>ikoding-android-app</keystore.alias>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <build>
        <finalName>${project.artifactId}-${project.version}-${manifest.metadata.id}</finalName>
        <sourceDirectory>src</sourceDirectory>
        <resources>
            <resource>
                <directory>.</directory>
                <filtering>true</filtering>
                <targetPath>../filtered-resources</targetPath>
                <includes>
                    <include>AndroidManifest.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>resources</directory>
                <filtering>true</filtering>
                <includes>
                    <include>**/*</include>
                </includes>
                <excludes>
                    <exclude>**/env-*.properties</exclude>
                </excludes>
            </resource>
        </resources>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                    <artifactId>android-maven-plugin</artifactId>
                    <version>3.5.0</version>
                    <extensions>true</extensions>
                    <executions>
                        <execution>
                            <id>run</id>
                            <goals>
                                <goal>deploy</goal>
                                <goal>run</goal>
                            </goals>
                            <phase>install</phase>
                        </execution>
                    </executions>
                    <configuration>
                        <proguardConfig>proguard.cfg</proguardConfig>
                        <proguardSkip>${project.build.proguardSkip}</proguardSkip>
                        <manifestDebuggable>${manifest.debuggable}</manifestDebuggable>
                        <androidManifestFile>target/filtered-resources/AndroidManifest.xml</androidManifestFile>
                        <release>${project.build.release}</release>
                        <run>
                            <debug>${project.build.debug}</debug>
                        </run>
                        <runDebug>${project.build.runDebug}</runDebug>
                        <sign>
                            <debug>${project.build.sign.debug}</debug>
                        </sign>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                <artifactId>android-maven-plugin</artifactId>
                <configuration>
                    <sdk>
                        <platform>15</platform>
                    </sdk>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>keytool-maven-plugin</artifactId>
                <version>1.2</version>
                <!--<executions>-->
                    <!--<execution>-->
                        <!--<goals>-->
                            <!--<goal>clean</goal>-->
                            <!--<goal>generateKeyPair</goal>-->
                        <!--</goals>-->
                        <!--<phase>generate-resources</phase>-->
                    <!--</execution>-->
                <!--</executions>-->
                <configuration>
                    <keystore>${keystore.filename}</keystore>
                    <storepass>${keystore.storepass}</storepass>
                    <keypass>${keystore.keypass}</keypass>
                    <alias>${keystore.alias}</alias>
                    <dname>CN=iKoding, OU=iKoding, O=iKoding, C=CN</dname>
                    <sigalg>SHA1withDSA</sigalg>
                    <validity>10000</validity>
                    <keyalg>DSA</keyalg>
                    <keysize>1024</keysize>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <encoding>UTF8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-eclipse-plugin</artifactId>
                <version>2.9</version>
                <configuration>
                    <projectnatures>
                        <projectnature>org.eclipse.m2e.core.maven2Nature</projectnature>
                        <projectnature>com.android.ide.eclipse.adt.AndroidNature</projectnature>
                        <projectnature>org.eclipse.jdt.core.javanature</projectnature>
                    </projectnatures>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.6</version>
                <executions>
                    <execution>
                        <phase>initialize</phase>
                        <goals>
                            <goal>resources</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <profile>
            <id>debug</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <filters>
                    <filter>resources/env-debug.properties</filter>
                </filters>
            </build>
            <properties>
                <project.build.debug>true</project.build.debug>
                <project.build.runDebug>false</project.build.runDebug>
                <project.build.proguardSkip>true</project.build.proguardSkip>
                <project.build.release>false</project.build.release>
                <project.build.sign.debug>true</project.build.sign.debug>
                <manifest.debuggable>true</manifest.debuggable>
            </properties>
        </profile>
        <profile>
            <id>release</id>
            <properties>
                <project.build.debug>false</project.build.debug>
                <project.build.runDebug>false</project.build.runDebug>
                <project.build.proguardSkip>false</project.build.proguardSkip>
                <project.build.release>true</project.build.release>
                <project.build.sign.debug>false</project.build.sign.debug>
                <manifest.debuggable>false</manifest.debuggable>
            </properties>
            <build>
                <filters>
                    <filter>resources/env-release.properties</filter>
                </filters>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-jarsigner-plugin</artifactId>
                        <version>1.2</version>
                        <executions>
                            <execution>
                                <id>sign</id>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                                <phase>package</phase>
                                <inherited>true</inherited>
                                <configuration>
                                    <includes>
                                        <include>${project.build.outputDirectory}/*.apk</include>
                                    </includes>
                                    <keystore>${keystore.filename}</keystore>
                                    <storepass>${keystore.storepass}</storepass>
                                    <keypass>${keystore.keypass}</keypass>
                                    <alias>${keystore.alias}</alias>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <!-- 渠道profiles -->
        <profile>
            <id>channel-test</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <manifest.metadata.id>test</manifest.metadata.id>
                <manifest.metadata.channel>test</manifest.metadata.channel>
            </properties>
        </profile>
        <profile>
            <id>channel-91</id>
            <properties>
                <manifest.metadata.id>91-market</manifest.metadata.id>
                <manifest.metadata.channel>91 market</manifest.metadata.channel>
            </properties>
        </profile>
        <profile>
            <id>channel-yingyonghui</id>
            <properties>
                <manifest.metadata.id>yingyonghui-market</manifest.metadata.id>
                <manifest.metadata.channel>yingyonghui market</manifest.metadata.channel>
            </properties>
        </profile>
        <profile>
            <id>channel-tongbutui</id>
            <properties>
                <manifest.metadata.id>tongbutui-market</manifest.metadata.id>
                <manifest.metadata.channel>tongbutui market</manifest.metadata.channel>
            </properties>
        </profile>
        <profile>
            <id>channel-tengxun</id>
            <properties>
                <manifest.metadata.id>tengxun-market</manifest.metadata.id>
                <manifest.metadata.channel>tengxun market</manifest.metadata.channel>
            </properties>
        </profile>
        <profile>
            <id>channel-anzhi</id>
            <properties>
                <manifest.metadata.id>anzhi-market</manifest.metadata.id>
                <manifest.metadata.channel>anzhi market</manifest.metadata.channel>
            </properties>
        </profile>
        <profile>
            <id>channel-gfan</id>
            <properties>
                <manifest.metadata.id>gfan</manifest.metadata.id>
                <manifest.metadata.channel>gfan</manifest.metadata.channel>
            </properties>
        </profile>
    </profiles>
</project>
以上的pom文件基本上涉及到了一个Android应用构建过程中的各个方面，以下针对其中比较重要的一些点作简要说明。

依赖管理
这也是我们使用Maven构建Android项目的重要原因之一，值得注意的是以下部分：

<dependency>
    <groupId>com.ikoding.android</groupId>
    <artifactId>android-base</artifactId>
    <version>2.0.0</version>
    <type>apklib</type>
</dependency>
以上依赖的type为apklib，这就是我们项目中的Library Project，此外，我们还可以使用Maven管理Native库，亦即我们项目中所依赖的那些so库文件。

资源过滤
项目中总会有一些和环境相关的配置，比如线下测试环境和线上环境的配置可能不一样，为此我们在pom中分别定义了id为debug和release两个profile，并使用不同的filter进行资源过滤。

生成keystore
我们使用了keytool-maven-plugin来生成签名时所需的keystore文件，我在properties中定义了生成keystore文件所需的信息，如下所示：

<properties>
    <keystore.filename>app-quickstart.keystore</keystore.filename>
    <keystore.storepass>2013@ikoding</keystore.storepass>
    <keystore.keypass>2013@ikoding</keystore.keypass>
    <keystore.alias>ikoding-android-app</keystore.alias>
</properties>
可以运行mvn keytool:generateKeyPair命令来生成keystore文件，之后在应用正式打包发布时候就会使用该keystore文件进行签名。

生成渠道包
项目最终发布时需要根据渠道号生成不同的渠道包，我们可以利用Maven的资源过滤对AndroidManifest.xml文件中的渠道信息进行替换，例如我们上面的pom文件对Manifest文件中${manifest.metadata.channel}的渠道信息进行了替换，还有一些其他信息如Manifest文件的debuggable属性也可以针对不同配置进行替换，比如我们在日常开发中需要将该值设为true，而在项目正式发布时需要将该值设为false，我们以上的pom文件中就定义了多个针对不同渠道的profile。

代码混淆
项目正式发布时需要对代码进行混淆处理，我们只需在android-maven-plugin配置中指定proguardConfig文件，比如我们在上面的pom中指定proguard文件为proguard.cfg，项目构建过程中会自动运行代码混淆，并且可以通过proguardSkip属性指定是否运行代码混淆任务，这样我们就可以在日常开发调试过程中关闭混淆，而在项目发布时开启混淆。

签名
我们使用了maven-jarsigner-plugin对apk文件进行签名，签名使用的证书就是之前生成的keystore文件。

调试
我在使用Maven构建Android项目之初遇到的问题就是无法进行断点调试，这显然会降低开发效率，后来发现可以通过android-maven-plugin的runDebug属性开启调试，这样可以在应用程序启动时连接debugger进行调试。

IDE支持
再次回到这个话题上，在导入eclipse之前，你需要先运行mvn eclipse:eclipse命令，生成eclipse项目文件，因为eclipse对android-maven-plugin支持不好。

3. 常用命令

以下是使用Maven构建Android项目中常用的一些命令，你可以根据需要选择合适的命令。

mvn clean package
打包，但不部署。

mvn clean install
打包，部署并运行。

mvn clean package android:redeploy android:run
这个命令通常用于手机上已经安装了要部署的应用，但签名不同，所以我们打包的同时使用redeploy命令将现有应用删除并重新部署，最后使用run命令运行应用。

mvn android:redeploy android:run
不打包，将已生成的包重新部署并运行。

mvn android:deploy android:run
部署并运行已生成的包，与redeploy不同的是，deploy不会删除已有部署和应用数据。

4. 总结

使用Maven构建Android应用很好地解决了依赖管理的问题，而且我们也可以将很多繁琐的任务自动化完成。在使用过程中主要的问题是Eclipse的m2e-android插件支持不够理想，而IntelliJ IDEA在这方便显得非常强大，让我得以将Maven和IDE的优势结合在一起。你可以在此基础上开发出一些常用类型项目的maven archetype，这样你就可以更快的开始一个新的项目。

5. 坚持？改变？

事实上我所在的团队成员最初都在使用Eclipse进行开发，后来在我的一再“忽悠”下，大家都接受了这种新的开发方式，并逐渐转向IntelliJ IDEA。这需要你有做出改变的勇气，毕竟改变是痛苦的，没有充分的理由就很难说服别人改变，但当你适应后就会有新的收获。

附件

下面是Android Library Project的pom文件，它比文中的Application Project的pom文件看起来要简单很多：

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.ikoding.android</groupId>
    <artifactId>android-lib-quickstart</artifactId>
    <version>1.0.0</version>
    <packaging>apklib</packaging>
    <name>Android Library Project Quick Start</name>

    <dependencies>
        <dependency>
            <groupId>com.google.android</groupId>
            <artifactId>android</artifactId>
            <version>4.1.1.4</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <build>
        <sourceDirectory>src</sourceDirectory>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                    <artifactId>android-maven-plugin</artifactId>
                    <version>3.4.1</version>
                    <extensions>true</extensions>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                <artifactId>android-maven-plugin</artifactId>
                <configuration>
                    <sdk>
                        <platform>15</platform>
                    </sdk>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-javadoc-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-eclipse-plugin</artifactId>
                <version>2.9</version>
                <configuration>
                    <projectnatures>
                        <projectnature>org.eclipse.m2e.core.maven2Nature</projectnature>
                        <projectnature>com.android.ide.eclipse.adt.AndroidNature</projectnature>
                        <projectnature>org.eclipse.jdt.core.javanature</projectnature>
                    </projectnatures>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

http://peirenlei.iteye.com/blog/1546413
maven集成eclipse android项目办法总结

博客分类： android
 
1.安装m2eclipse-android-integration
https://svn.codespot.com/a/eclipselabs.org/m2eclipse-android-integration/updates/m2eclipse-android-integration/
其实这个不需要安，不过下面的是必须要安装的。
 
安装maven-android-sdk-deployer
https://github.com/mosabua/maven-android-sdk-deployer
下载完后，执行：
 
Java代码  收藏代码
cd mosabua-maven-android-sdk-deployer-df824df  
mvn clean install  
  
#可选择安装：  
mvn install -P 1.5  
mvn install -P 1.6  
mvn install -P 2.1  
mvn install -P 2.2  
mvn install -P 2.3.3  
mvn install -P 3.0  
mvn install -P 3.1  
mvn install -P 3.2  
mvn install -P 4.0  
mvn install -P 4.0.3  
 如果报错，说明sdk没有安装完整（google api等等），要在android sdk manager里面下载完整，
如果运行正常的话，这一步操作就是将sdk的包放入到svn的本地仓库里了。
 
 
2.Create Eclipse project
 
If you already have an Android project please make sure you have created a POM for your project using version 3.0.0 or greater of the maven-android-plugin.
 
Then right-click on your project and select Configuration -> Convert to Maven Project.
 
If you are starting with a new project you can use the Maven Android archetypes to create Android projects completely within Eclipse:
 
Create a new Maven Project (File -> New -> Project... then select Maven -> Maven Project).
When prompted to Select Archetype click Add Archetype...
In the dialog that appears enter "de.akquinet.android.archetypes" for Archetype Group Id.
In Archetype Artifact Id enter "android-quickstart".
In Archetype Version enter "1.0.8" and continue.
When prompted enter your desired project group and artifact ID, version and, optionally, set the "platform" property for the Android version (defaults to '10').
Click Finish
Right-click on new project and select Maven -> Update Project Configuration.
 
 
3.Install Android Connector
Windows->Preferences->Maven->Discovery->Open Catalog->安装m2e connector for android
 
4.至此新建的项目应该不会再报错了。
 
5.配置maven插件，自动打包签名，附上我的完整的pom：
 
Xml代码  收藏代码
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <groupId>com.olivephone</groupId>  
    <artifactId>olive-browser</artifactId>  
    <version>1.0.10</version>  
    <packaging>apk</packaging>  
    <name>olive-browser</name>  
      
    <properties>  
        <platform.version>2.2.1</platform.version>  
    </properties>  
      
    <dependencies>  
        <dependency>  
            <groupId>com.google.android</groupId>  
            <artifactId>android</artifactId>  
            <version>${platform.version}</version>  
            <scope>provided</scope>  
        </dependency>  
        <dependency>  
            <groupId>com.google.android</groupId>  
            <artifactId>support-v4</artifactId>  
            <version>r7</version>  
        </dependency>  
        <dependency>  
            <groupId>commons-io</groupId>  
            <artifactId>commons-io</artifactId>  
            <version>2.0.1</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.commons</groupId>  
            <artifactId>commons-lang3</artifactId>  
            <version>3.1</version>  
        </dependency>  
        <dependency>  
            <groupId>org.jsoup</groupId>  
            <artifactId>jsoup</artifactId>  
            <version>1.6.1</version>  
        </dependency>  
        <dependency>  
            <groupId>com.google.android</groupId>  
            <artifactId>libGoogleAnalytics</artifactId>  
            <version>1.0</version>  
        </dependency>  
    </dependencies>  
          
    <build>  
        <plugins>  
            <plugin>  
                <groupId>com.jayway.maven.plugins.android.generation2</groupId>  
                <artifactId>android-maven-plugin</artifactId>  
                <version>3.1.1</version>  
                <configuration>  
                    <androidManifestFile>${project.basedir}/AndroidManifest.xml</androidManifestFile>  
                    <assetsDirectory>${project.basedir}/assets</assetsDirectory>  
                    <resourceDirectory>${project.basedir}/res</resourceDirectory>  
                    <nativeLibrariesDirectory>${project.basedir}/src/main/native</nativeLibrariesDirectory>  
                    <sdk>  
                        <platform>8</platform>  
                    </sdk>  
                    <undeployBeforeDeploy>true</undeployBeforeDeploy>  
                </configuration>  
                <extensions>true</extensions>  
            </plugin>  
  
            <plugin>  
                <artifactId>maven-compiler-plugin</artifactId>  
                <version>2.3.2</version>  
                <configuration>  
                    <source>1.6</source>  
                    <target>1.6</target>  
                    <encoding>UTF8</encoding>  
                </configuration>  
            </plugin>  
              
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-jarsigner-plugin</artifactId>  
                <version>1.2</version>  
                <executions>  
                  <execution>  
                    <id>sign</id>  
                    <goals>  
                      <goal>sign</goal>  
                    </goals>  
                  </execution>  
                  <execution>  
                    <id>verify</id>  
                    <goals>  
                      <goal>verify</goal>  
                    </goals>  
                  </execution>  
                </executions>  
                <configuration>  
                    <encoding>UTF-8</encoding>  
                    <includes>  
                            <include>target/${artifactId}.apk</include>  
                            </includes>     
                            <removeExistingSignatures>true</removeExistingSignatures>  
                            <keystore>${keyFilePath}</keystore>  
                    <storepass>${storePassword}</storepass>  
                    <keypass>${keyPassword}</keypass>  
                    <alias>${keyAlias}</alias>  
                </configuration>  
            </plugin>  
                
        <plugin>  
                 <groupId>org.codehaus.mojo</groupId>  
                 <artifactId>exec-maven-plugin</artifactId>  
                 <version>1.2.1</version>  
                 <executions>  
                     <execution>  
                         <id>zipalign</id>  
                         <goals>  
                             <goal>exec</goal>  
                         </goals>  
                         <phase>install</phase>  
                         <configuration>  
                             <executable>${ANDROID_HOME}/tools/zipalign</executable>  
                              <arguments>  
                                 <argument>-f</argument>  
                                 <argument>4</argument>  
                                 <argument>target/${project.build.finalName}.apk</argument>  
                                 <argument>target/${project.build.finalName}-zipped.apk</argument>  
                             </arguments>  
                         </configuration>  
                     </execution>  
                 </executions>  
        </plugin>  
        </plugins>   
    </build>  
      
</project>  
 
 
setting.xml
 
Xml代码  收藏代码
<settings xmlns="http://maven.apache.org/POM/4.0.0"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  
                               http://maven.apache.org/xsd/settings-1.0.0.xsd">  
  
<!-- add by peirenlei here one line-->  
<localRepository>D:/3rdlib/java/m2/repository</localRepository>  
  
<profiles>  
    <profile>  
        <id>sign</id>  
        <activation>  
            <activeByDefault>true</activeByDefault>  
        </activation>  
        <properties>  
            <keyFilePath>E:/MyDocument/xxxx.keystore</keyFilePath>  
            <storePassword>就不告诉你</storePassword>  
            <keyPassword>就不告诉你</keyPassword>  
            <keyAlias>就不告诉你</keyAlias>  
        </properties>  
    </profile>  
  
</profiles>  
<activeProfiles>  
   <activeProfile>sign</activeProfile>  
</activeProfiles>  
  
</settings>  
 
 
特别提醒：
1.<removeExistingSignatures>true</removeExistingSignatures>这个一定要配上，我之前参考网上的都没有这一项，结果就是签名的apk里面有两个签名，一个是自己的一个是android debug的，so...
 
2.记住源代码的路径要更改为：src/main/java,否则在package的时候会抱错，并且就算不抱错，打好的包在真机上也运行不了，谨记，为这个事情，我折磨了一整天。
 
 
6.配置这这一切，接下来就是爽歪歪的时候了： 
 
Java代码  收藏代码
mvn clean package  
 进入target目录，看到的mvndemo-1.0.9.apk就是已经签过名的apk,怎么？不相信，自己看看：
 
Java代码  收藏代码
jarsigner -verify -verbose -certs "F:\workspace\android_workspace\mvndemo\target\mvndemo-1.0.9.apk"  
 
看到没？打印出来的信息里，是不是有自己的签名信息。
 
收工了。。。。
