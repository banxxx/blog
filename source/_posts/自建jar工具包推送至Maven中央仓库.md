---
title: 自建jar工具包推送至Maven中央仓库
tags:
  - java
  - maven
categories:
  - JAVA相关
abbrlink: f5f6bbca
date: 2023-12-01 11:29:15
---

本篇文章详细介绍如何自建java工具包并推送至maven中央仓库供所有人引入使用
<!-- more -->
## 1. 注册账号
登录[Sign up for Jira - Sonatype JIRA](https://issues.sonatype.org/secure/Signup!default.jspa "Sign up for Jira - Sonatype JIRA")网站,注册账号
密码要求比较高,大小写数字符号啥的都要有,注意做好备份
## 2. 创建Issue
用上面创建的账号密码登录后,点击页面上方导航栏的Create按钮创建工单Issue:
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_02-13-51.png)
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_02-17-09.png)
通过以上两个步骤,你的工单就被创建完成了,接下来就耐心等待工作人员审核,注意留意你注册账号时填写的邮箱,回复内容会在邮箱里通知你.
## 3. 确认邮件
在收到工作人员的确认邮件后,在邮件内容中他们通常会要求验证你填写的groupId中的域名或github账户是不是你本人的
+ 如果你填写的是域名,他们会要去你用该域名的邮箱给他们发一封邮件进行确认
+ 如果你填写的是github账户,他们会要求你在你的github中创建一个指定名称的仓库进行验证(如下图,按要求去创建即可)
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_18-59-49.png)
创建好并确认无误后,你需要将此Iusse的状态置为Open,然后他们系统会自动去完成验证.
## 4.上传至中央仓库
官方验证此域名/账户确实是你本人的之后,会再次发送邮件给你,告诉你你的仓库已经被激活了,此时你需要按照他们的要求,分别上传你项目的SNAPSHOT和release版本至指定地址:
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_19-03-47.png)
### 4.1 上传SNAPSHOT版本的步骤
如果步骤走不下去,请务必查阅官方文档,以官方文档为准,因为可能会有更新.需要一定的英文文档阅读能力,如果英文不好可以使用谷歌翻译及其他插件.
参考文档地址: [OSSRH Guide - The Central Repository Documentation](https://central.sonatype.org/publish/publish-guide/#deployment)
{% note color:orange 流程: 配置秘钥 -> 配置settings.xml -> 配置pom.xml -> 上传%}
#### 1. 配置秘钥:
下载并安装GPG:
[https://www.gnupg.org/download/index.html](https://www.gnupg.org/download/index.html)
根据你电脑当前系统选择合适的安装包下载并安装,安装按默认勾选的配置一路安装即可.
生成密钥有两种方式,敲命令和UI界面两种,这里演示两种方法
##### a. 敲命令的方式
 ```Shell
 生成：
 gpg --gen-key

 Real name: 名字
 Email address: 邮箱
 You selected this USER-ID:
 "xxx[xxx@qq.com](mailto:xxx@qq.com)"
 Change (N)ame, (E)mail, or (O)kay/(Q)uit?
 之后往下，会让你输入用户名和邮箱，还有一个Passphase
 (输入两次,务必牢记,建议先找个地方记下来,后续要用到)
```
```Shell
查看公钥
gpg --list-keys
 
查询结果：
C:/Users/Laohan/AppData/Roaming/gnupg/pubring.kbx
--------------------------------------------------
pub   rsa2048 2020-06-06 [SC] [expires: 2022-06-06]
      85B594371E0A38D70243B1E927EDC1D952E45334
uid           [ultimate] banxx <xxxxxx@qq.com>
sub   rsa2048 2020-06-06 [E] [expires: 2022-06-06]
```
```
其中 85B594371E0A38D70243B1E927EDC1D952E45334 就是你的公钥key
```
```
发布公钥：
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 85B594371E0A38D70243B1E927EDC1D952E45334
```
```
查询发布公钥是否成功
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --recv-keys 85B594371E0A38D70243B1E927EDC1D952E45334
 
成功的话会有如下结果
gpg: key 27EDC1D952E45891: "xingpengcheng <252645816@qq.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
```

##### b. 应用程序的方式
先去GPG官网下载windows版本应用
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_20-27-52.png)
打开应用软件,按图所示
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_20-35-07.png)
新建密钥对成功后,将密钥对发送至服务器上
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_20-38-47.png)
等待一会后在服务器上查找,找到证书即为成功,然后将此证书导出


#### 2. 配置settings.xml
找到你IDEA自带或者你安装的Maven所在的目录:
File->settings->Build->Maven->MavenHomePath,进入此目录在conf文件夹下就可以看到settings.xml文件,如果没有的话可以自己新建一个.
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_19-36-47.png)
然后在其中加入配置:
```XML
  <servers>
	  <server>
        <id>ossrh</id>
        <username>xxxxx@qq.com(SonaType账号)</username>
        <password>填你注册SonaType时填写的密码</password>
	  </server>
  </servers>
 
  <profiles>
    <profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <!--这里填你安装的GnuPG位置-->
        <!--可视化Klepatra也会安装GnuPG,耐心找一找-->
        <gpg.executable>C:/Program Files (x86)/GnuPG/bin/gpg.exe</gpg.executable>
        <gpg.passphrase>填写你生成秘钥时输入的密码</gpg.passphrase>
        <!--这里填你秘钥在磁盘上的位置,可通过上面步骤的 gpg --list-keys找到,也可不填-->
        <gpg.homedir>C:/Users/laohan/.gnupg</gpg.homedir>
      </properties>
    </profile>
  </profiles>
  ```
#### 3. 配置pom.xml
以下配置信息已为最精简版,省略了无关内容,且加了注释,方便读者理解:
```xml
    <!--gav信息,此处填域名或者github仓库域名,与上面注册时一致-->
    <groupId>io.github.xxx</groupId>
    <artifactId>banxx</artifactId>
    <!--需要特别注意,你上传的是SNAPSHOT仓库,所以此处版本号后缀必须带SNAPSHOT-->
    <version>0.9.3-SNAPSHOT</version>
 
    <!--项目信息-->
    <name>easy-es-parent</name>
    <!--说明文档-->
    <description>use for myself</description>
    <!--代码托管在github仓库的地址-->
    <url>https://github.com/xxxxx/banxx</url>
 
    <!--开源协议-->
    <licenses>
        <license>
            <name>The Apache Software License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>
 
    <!--开发者信息-->
    <developers>
        <developer>
            <id>xpc</id>
            <name>xpc</name>
            <email>252645816@qq.com</email>
            <roles>
                <role>Project Manager</role>
                <role>Architect</role>
            </roles>
            <timezone>+8</timezone>
        </developer>
    </developers>
    
    <!--项目在github或其它托管平台的地址-->
    <scm>
        <connection>https://github.com/xxxxx/banxx.git</connection>
        <developerConnection>scm:git:ssh://git@github.com/xxxxx/banxx.git</developerConnection>
        <url>https://github.com/xxxxx/banxx</url>
    </scm>
 
    <profiles>
        <profile>
            <!--注意,此id必须与setting.xml中指定的一致,不要改变名字-->
            <id>ossrh</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <!--发布到中央SNAPSHOT仓库插件-->
                <plugins>
                    <plugin>
                        <groupId>org.sonatype.plugins</groupId>
                        <artifactId>nexus-staging-maven-plugin</artifactId>
                        <version>1.6.7</version>
                        <extensions>true</extensions>
                        <configuration>
                            <serverId>ossrh</serverId>
                            <nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
                            <autoReleaseAfterClose>true</autoReleaseAfterClose>
                        </configuration>
                    </plugin>
                       
                    <!--生成源码插件-->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>2.2.1</version>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    
                    <!--生成API文档插件-->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>2.9.1</version>
                        <executions>
                            <execution>
                                <id>attach-javadocs</id>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
 
                    <!--gpg插件-->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.5</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
 
                </plugins>
            </build>
            
            <distributionManagement>
                <snapshotRepository>
                   <!--注意,此id必须与setting.xml中指定的一致-->
                   <id>ossrh</id>
                   <url>https://s01.oss.sonatype.org/content/repositories/snapshots</url>
                </snapshotRepository>
                <repository>
                    <id>ossrh</id>
              <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/</url>
                </repository>
            </distributionManagement>
        </profile>
 
    </profiles>
```
完成上述配置,你已经基本上走完了80%的步骤了,此时,你可以通过Maven尝试打包一下你的项目:
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_19-45-10.png)
#### 4. 上传
接着点击{% emp clean %},{% emp deploy %}然后会弹出对话框让你输入生成gpg时填写的密码,然后弹出的确认对话框都敲回车,耐心等待上传,完成后你可以在控制台看到SUCCESS,至此,SNAPSHOT版本已被上传至中央SNAPSHOT仓库,可以在浏览器访问[Nexus Repository Manager](https://s01.oss.sonatype.org/#welcome),登录后查看.
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_19-46-59.png)

### 4.2 上传release版本的步骤
整体流程和上传SNAPSHOT版本一致,下面仅贴出差异点:
#### 1. settings.xml
```XML
<!--将原来server标签和profile标签中的的ossrh替换为release-->
<id>release</id>
```
如下图
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_19-57-57.png)
#### 2. pom.xml
```XML
<!--修改GAV中的版本号,把SNAPSHOT后缀去掉-->
<version>0.9.3</version>
 
<!--将原来server标签和profile标签中的的ossrh替换为release-->
<id>release</id>
<!--移除此段发布到中央SNAPSHOT仓库插件代码,并替换为下面代码发布到中央release仓库的插件-->
    <plugin>
        <groupId>org.sonatype.plugins</groupId>
        <artifactId>nexus-staging-maven-plugin</artifactId>
        <version>1.6.7</version>
        <extensions>true</extensions>
        <configuration>
            <serverId>ossrh</serverId>
            <nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
            <autoReleaseAfterClose>true</autoReleaseAfterClose>
        </configuration>
    </plugin>

    <!--替换代码-->
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-release-plugin</artifactId>
        <version>2.5.3</version>
        <configuration>
            <autoVersionSubmodules>true</autoVersionSubmodules>
            <useReleaseProfile>false</useReleaseProfile>
            <releaseProfiles>release</releaseProfiles>
            <goals>deploy</goals>
        </configuration>
    </plugin>
```
#### 3. 发布
点击clean,deploy 然后耐心等待成功,因为这是你第一次上传release包至中央仓库,
所以他们平台的机器人会自动帮你同步(在他们回复你的ISSUE里也提到了这点,同步触发的前提是SNAPSHOT和release版本都已正确上传至中央仓库)
上传成功后,可在官网查看到:
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_20-06-49.png)
在完成了SNAPSHOT和release版本上传之后,你会收到系统发来的邮件:
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_20-10-00.png)
大致就是告诉你你已经成功完成了首次上传,你上传的项目会在30分钟以内同步到中央仓库中,
届时你可以在Central Repository:中找和下载.
4小时内你可以在  Maven Central Repository Search中搜索到.

至此,恭喜你已首次完成将项目发布至中央仓库.
## 5. 后续
在之后,随着你项目的迭代,你可能需要发布新的版本至中央仓库,此时你可以跳过上传SNAPSHOT,
直接上传release版本至中央仓库,由于非首次发布,他们的机器人也不再会帮助你自动同步了,
此过程需要你手动触发,在上传完release包之后,打开官方页面,进入操作,具体流程如下图:
[Nexus Repository Manager](https://s01.oss.sonatype.org/#welcome)
![](https://cos.banxx.cn/Blog/PixPin_2023-12-03_20-19-22.png)

