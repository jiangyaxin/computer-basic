# 参数

常用参数：

| 参数  | 说明                                                                                  |
|-----|-------------------------------------------------------------------------------------|
| -h  | 帮助                                                                                  |
| -U  | 强制更新releases、snapshots类型的插件或依赖库(否则maven一天只会更新一次snapshot依赖)                          |
| -o  | 运行offline模式,不联网进行依赖更新                                                               |
| -P  | 激活指定的profile文件列表(用逗号,隔开)                                                            |
| -pl | 手动选择需要构建的项目,项目间以逗号分隔，或者使用 ! 表示不构建。<br />例如`mvn -pl "!signal-infra,!signal-gateway"` |
| -rf | 从指定的项目(或模块)开始继续构建                                                                   |
| -X  | 输出详细信息，debug模式                                                                      |

全量参数：

```text
-h,--help                              Display help information
-am,--also-make                        构建指定模块,同时构建指定模块依赖的其他模块;
-amd,--also-make-dependents            构建指定模块,同时构建依赖于指定模块的其他模块;
-B,--batch-mode                        以批处理(batch)模式运行;
-C,--strict-checksums                  检查不通过,则构建失败;(严格检查)
-c,--lax-checksums                     检查不通过,则警告;(宽松检查)
-D,--define <arg>                      Define a system property
-e,--errors                            显示详细错误信息
-emp,--encrypt-master-password <arg>   Encrypt master security password
-ep,--encrypt-password <arg>           Encrypt server password
-f,--file <arg>                        使用指定的POM文件替换当前POM文件
-fae,--fail-at-end                     最后失败模式：Maven会在构建最后失败（停止）。如果Maven refactor中一个失败了，Maven会继续构建其它项目，并在构建最后报告失败。
-ff,--fail-fast                        最快失败模式： 多模块构建时,遇到第一个失败的构建时停止。
-fn,--fail-never                       从不失败模式：Maven从来不会为一个失败停止，也不会报告失败。
-gs,--global-settings <arg>            替换全局级别settings.xml文件(Alternate path for the global settings file)
-l,--log-file <arg>                    指定输出日志文件
-N,--non-recursive                     仅构建当前模块，而不构建子模块(即关闭Reactor功能)。
-nsu,--no-snapshot-updates             强制不更新SNAPSHOT(Suppress SNAPSHOT updates)
-U,--update-snapshots                  强制更新releases、snapshots类型的插件或依赖库(否则maven一天只会更新一次snapshot依赖)
-o,--offline                           运行offline模式,不联网进行依赖更新
-P,--activate-profiles <arg>           激活指定的profile文件列表(用逗号[,]隔开)
-pl,--projects <arg>                   手动选择需要构建的项目,项目间以逗号分隔
-q,--quiet                             安静模式,只输出ERROR
-rf,--resume-from <arg>                从指定的项目(或模块)开始继续构建
-s,--settings <arg>                    替换用户级别settings.xml文件(Alternate path for the user settings file)
-T,--threads <arg>                     Thread count, for instance 2.0C where C is core multiplied
-t,--toolchains <arg>                  Alternate path for the user toolchains file
-V,--show-version                      Display version information WITHOUT stopping build
-v,--version                           Display version information
-X,--debug                             输出详细信息，debug模式。
```

# 内置属性

1. 内置属性（Maven预定义属性，用户可以直接使用）

    * `${basedir}`表示项目的根路径，即包含pom.xml文件的目录
    * `${version}`表示项目版本
    * `${project.basedir}`同${basedir}
    * `${project.baseUri}`表示项目文件地址
    * `${maven.build.timestamp}`表示项目构建开始时间
    * `${maven.build.timestamp.format}`表示`${maven.build.timestamp}`的展示格式，默认值为yyyyMMdd-HHmm

2. pom属性（使用pom属性可以引用到pom.xml文件中对应元素的值)

    * `${project.build.sourceDirectory}`表示主源码路径，默认为`src/main/java/`
    * `${project.build.testSourceDirectory}`表示测试源码路径，默认为`src/test/java/`
    * `${project.build.directory}`表示项目构建输出目录，默认为`target/`
    * `${project.outputDirectory}`表示项目测试代码编译输出目录，默认为`target/classes/`
    * `${project.groupId}`表示项目的groupId
    * `${project.artifactId}`表示项目的artifactId
    * `${project.version}`表示项目的version，同`${version}`
    * `${project.build.finalName}`表示项目打包输出文件的名称,默认为`${project.artifactId}-${project.version}`

3. 自定义属性（在pom.xml文件的<properties>标签下定义的Maven属性）
4. setting.xml文件属性（与pom属性同理，用户可以用settings.开头的属性引用setting.xml文件的xml元素值）

    * ${settings.localRepository}表示本地仓库的地址

5. Java系统属性（所有的Java属性都可以使用Maven属性引用）
   `mvn help:system` 可以查看所有的Java属性 即 System.getProperties() 可以得到所有的Java属性
   `${user.home}` 表示用户目录

6. 环境变量属性（所有的环境变量都可以以env.开头的Maven属性引用）
   ` mvn help:system` 可查看所有的环境变量
   `${env.JAVA_HOME}` 表示JAVA_HOME环境变量的值

# profile

用于拆分不同环境的构建，不同的环境需要使用的依赖、插件、属性等存在不同的配置。

```xml

<project>
    <profiles>
        <profile>
            <id>...</id>
            <activation>...</activation>
            <properties>...</properties>
            <build>
                <defaultGoal>...</defaultGoal>
                <finalName>...</finalName>
                <resources>...</resources>
                <testResources>...</testResources>
                <plugins>...</plugins>
            </build>
            <reporting>...</reporting>
            <modules>...</modules>
            <dependencies>...</dependencies>
            <dependencyManagement>...</dependencyManagement>
            <distributionManagement>...</distributionManagement>
            <repositories>...</repositories>
            <pluginRepositories>...</pluginRepositories>
        </profile>
    </profiles>
</project>
```

激活profile的方式：

1. 命令行激活：`mvn clean install -Ptest1,test2`
2. settings.xml 文件激活，属于全局配置,每个项目都会生效。

   ```xml
   <activeProfiles>
       <activeProfile>dev</activeProfile>
   </activeProfiles>
   ```

3. 通过配置条件激活

   ```xml
   
   <profile>
       <id>prod</id>
       <properties>
           <envFlag>prod</envFlag>
       </properties>
       <activation>
          <!-- 1. 默认激活 -->
          <activeByDefault>true</activeByDefault>
   
          <!-- 2. jdk版本 激活 -->
          <jdk>1.5</jdk>
   
          <!-- 3. 操作系统 激活 -->
          <os>
             <name>Windows XP</name>
             <family>Windows</family>
             <arch>x86</arch>
             <version>5.1.2600</version>
          </os>
   
          <!-- 4. System.getProperties系统属性 激活，可通过 -Dxxx 传入 -->
          <property>
             <name>a</name>
             <value>dev</value>
          </property>
          <property>
             <name>!b</name>
          </property>
   
       </activation>
   </profile>
   ```

# classifier

用于区分从同一POM构建的具有不同内容的构件（artifact）。它是可选的，任意的字符串，附加在版本号之后。
classifier 不能直接定义，需通过 maven-jar-plugin 等插件生成。

例如：通过profile可以打出两个不同的版本 aa.bb.cc-0.1.jar 和 aa.bb.cc-0.1-jdk15.jar

```xml

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>aa.bb</groupId>
    <artifactId>cc</artifactId>
    <packaging>jar</packaging>
    <version>0.1</version>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.0.2</version>
                <configuration>
                    <source>${jar.source}</source>
                    <target>${jar.target}</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <!-- 这里省略依赖 -->
    </dependencies>

    <properties>
        <!-- 这里省略属性 -->
    </properties>

    <profiles>
        <profile>
            <id>default</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <jar.source>1.6</jar.source>
                <jar.target>1.6</jar.target>
            </properties>
        </profile>
        <profile>
            <id>jdk15</id>
            <build>
                <plugins>
                    <plugin>
                        <artifactId>maven-jar-plugin</artifactId>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                                <configuration>
                                    <!-- 指定 classifier -->
                                    <classifier>jdk15</classifier>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
            <properties>
                <jar.source>1.5</jar.source>
                <jar.target>1.5</jar.target>
            </properties>
        </profile>
    </profiles>
</project>  
```

# Scope

用来指定包的依赖范围和依赖的传递性，依赖范围用于控制三种classpath：编译classpath、测试classpath、运行classpath。

| scope取值  | 说明               | 有效范围          | 作用域传递 | 例子          |
|----------|------------------|---------------|-------|-------------|
| compile  | 默认值              | all           | 是     | spring-core |
| provided | 运行时无效表示不打包       | compile, test | 否     | servlet-api |
| runtime  | 编译时不需要           | runtime, test | 是     | JDBC驱动      |
| test     | 测试时有效            | test          | 否     | JUnit       |
| system   | 运行时无效，往往时本地运行时需要 | compile, test | 是     |             |

system范围的依赖时必须通过systemPath元素显式地指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析的，而且往往与本机系统绑定。

例如：

```xml

<dependency>
    <groupId>javax.sql</groupId>
    <artifactId>jdbc-stdext</artifactId>
    <version>2.0</version>
    <scope>system</scope>
    <systemPath>${java.home}/lib/rt.jar</systemPath>
</dependency>
```

# 仓库与镜像

## 添加仓库和认证

存在一个默认的id为central的重要仓库，如果repository命名为central会将其覆盖掉。

修改 maven 的 settings 文件：

```xml

<settings>
    <repositories>
        <repository>
            <id>nexus</id>
            <url>http://xxxxx:8081/nexus/content/groups/public/</url>
            <releases>
                <!-- 支持发布版本下载 -->
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <!-- 支持快照版本下载 -->
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>

    <servers>
        <server>
            <!-- 配置认证信息，id必须与repository id 一致 -->
            <id>nexus</id>
            <username>xxx</username>
            <password>xxx</password>
        </server>
    </servers>
</settings>
```

## 添加镜像

```xml

<settings>
    <mirrors>
        <mirror>
            <id>alimaven</id>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <!-- 表示任何请求central的请求都会转至该镜像 -->
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
</settings>
```

# 生命周期与插件

## 生命周期

maven 拥有三套互相独立的生命周期，clean、default、site,每个生命周期包含一些阶段 phase。

clean: 清理项目

* pre-clean: 执行一些清理前需要完成的工作。
* clean: 清理上一次构建生成的文件。
* post-clean: 执行一些清理后需要完成的工作。

default: 定义构建时锁需要执行的所有步骤

* validate: 验证项目是否正确，所有必要信息是否可用（很少单独使用）
* initialize
* generate-sources
* process-sources: 处理主资源文件，一般来说是对src/main/resources 目录内容进行变量替换等工作后，复制到输出的主classpath目录
* generate-resources
* process-resources
* compile: 编译
* process-classes
* generate-test-sources
* process-test-sources: 处理测试资源文件
* generate-test-resources
* process-test-resources
* test-compile: 编译
* process-test-classes
* test: 使用单元测试框架运行测试，测试代码不会被打包或部署
* prepare-package
* package: 打包成可发布的格式
* pre-integration-test
* integration-test
* post-integration-test
* verify
* install: 安装到本地仓库
* deploy: 部署到远程仓库

site: 生命周期

* pre-site
* site: 生成WEB站点HTML
* post-site
* site-deploy：WEB站点部署到服务器

## 插件

插件和生命周期互相绑定，用以描述在某个生命周期的某个阶段（phase）构建任务，一个插件有多个功能，每个功能就是一个插件目标(
goal)。
生命周期阶段与插件目标相互绑定， 例如：clean周期的clean阶段和 maven-clean-plugin:clean 目标绑定。

default 生命周期的内置插件绑定关系：

| 生命周期阶段                 | 插件目标                                 | 执行任务           |
|------------------------|--------------------------------------|----------------|
| process-resources      | maven-resources-plugin:resources     | 复制主资源文件至主输出目录  |
| compile                | maven-compiler-plugin:compile        | 编译主代码至主输出目录    |
| process-test-resources | maven-resources-plugin:testResources | 复制测试资源文件至主输出目录 |
| test-compile           | maven-compiler-plugin:testCompile    | 编译测试代码至测试输出目录  |
| test                   | maven-surefire-plugin:test           | 执行测试用例         |
| package                | maven-jar-plugin:jar                 | 创建项目jar包       |
| install                | maven-install-plugin:install         | 将项目输出构件安装到本地仓库 |
| deploy                 | maven-deploy-plugin:deploy           | 将项目输出构件部署到远程仓库 |

### 常用插件

#### maven-compiler-plugin

有两个目标：compile、testCompile，参数配置文档: https://maven.apache.org/plugins/maven-compiler-plugin/compile-mojo.html

常用参数：

```xml

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.13.0</version>
    <configuration>

        <!-- 一般而言，target与source是保持一致的，但是，有时候为了让程序能在其他版本的jdk中运行
             (对于低版本目标jdk，源代码中需要没有使用低版本jdk中不支持的语法)，
             会存在target不同于source的情况 -->
        <!-- 源代码使用的开发版本，通常使用 maven.compiler.source 属性设置 -->
        <source>1.8</source>
        <!-- 需要生成的目标class文件的编译版本，通常使用 maven.compiler.target 属性设置 -->
        <target>1.8</target>

        <!-- 这下面的是可选项 -->
        <!-- 默认值为false,为 true 时下面三个参数可用 -->
        <fork>true</fork>
        <meminitial>128m</meminitial>
        <maxmem>512m</maxmem>
        <compilerVersion>1.3</compilerVersion>

        <!-- 传递参数给jvm，compilerArguments 已废弃 -->
        <compilerArgs>
            <arg>-verbose</arg>
            <arg>-Xlint:unchecked</arg>
            <arg>-Xlint:deprecation</arg>
            <arg>-bootclasspath</arg>
            <arg>${env.JAVA_HOME}/jre/lib/rt.jar${path.separator}${env.JAVA_HOME}/lib/jce.jar</arg>
            <arg>-extdirs</arg>
            <arg>${project.basedir}/src/main/webapp/WEB-INF/lib</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

#### maven-jar-plugin

有两个目标：jar、testJar，参数配置文档: https://maven.apache.org/plugins/maven-jar-plugin/jar-mojo.html

常用参数：

```xml

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.4.1</version>

    <!-- 过滤掉不希望包含在jar中的文件  -->
    <excludes>
        <exclude>*.xml</exclude>
        <exclude>spring/**</exclude>
        <exclude>config/**</exclude>
    </excludes>
    <!-- 希望包含的文件 -->
    <includes>
        <!-- 打jar包时，打包class文件和config目录下面的 properties文件 -->
        <!-- 有时候可能需要一些其他文件，这边可以配置，包括剔除的文件等等 -->
        <include>**/*.class</include>
        <include>**/*.properties</include>
    </includes>

    <archive>
        <!-- 默认生成的jar中，包含pom.xml和pom.properties这两个文件 -->
        <addMavenDescriptor>true</addMavenDescriptor>
        <!-- 默认压缩 -->
        <compress>true</compress>
        <!-- 默认强制重新创建存档 -->
        <forced>true</forced>

        <!-- 配置清单（MANIFEST）-->
        <manifest>
            <!-- 默认false，添加 Class-Path 到 MANIFEST -->
            <addClasspath>true</addClasspath>
            <!-- 默认""，为classpath依赖包添加前缀 -->
            <classpathPrefix>lib/</addClasspath>
            <!-- jar启动入口类 -->
            <mainClass>xxx</mainClass>
            <!-- 添加 Class-Path 时，带时间戳 -->
            <useUniqueVersions>true</useUniqueVersions>
        </manifest>
        <manifestEntries>
            <!-- 在Class-Path下添加配置文件的路径 -->
            <Class-Path>../config/</Class-Path>
        </manifestEntries>
    </archive>
</plugin>
```

#### maven-resources-plugin

参数配置文档: https://maven.apache.org/plugins/maven-resources-plugin/copy-resources-mojo.html

有三个目标：

* resources：拷贝main resources到main output directory。绑定process-resources阶段，执行Compiler:compile插件目标前执行此阶段。
* testResources：拷贝test resources到test output directory。绑定process-test-resources阶段，执行surefire:test插件目标前执行此阶段。
* copy-resources：手动拷贝资源到输出目录

常用参数：

```xml

<build>
    <filters>
        <!-- 引入外置配置文件,替换掉resource文件中的属性 -->
        <filter>filters_file.properties</filter>
    </filters>

    <resources>
        <!--默认配置-->
        <resource>
            <directory>${project.basedir}/src/main/resources</directory>
        </resource>

        <!--手动配置-->
        <resource>
            <directory>src/my-resources</directory>
            <includes>
                <include>**/*.txt</include>
                <include>**/*.rtf</include>
            </includes>
            <excludes>
                <exclude>**/*test*.*</exclude>
                <exclude>**/*.pdf</exclude>
            </excludes>
        </resource>
    </resources>

    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.3.1</version>
            <!-- 手动资源拷贝-->
            <executions>
                <execution>
                    <id>copy-resources</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>copy-resources</goal>
                    </goals>
                </execution>
            </executions>

            <configuration>
                <encoding>UTF-8</encoding>

                <!-- 手动资源拷贝-->
                <outputDirectory>${basedir}/target/extra-resources</outputDirectory>
                <resources>
                    <resource>
                        <directory>src/non-packaged-resources</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </configuration>
        </plugin>
    </plugins>
</build>

```

#### maven-source-plugin

参数配置文档: https://maven.apache.org/plugins/maven-source-plugin/jar-no-fork-mojo.html

常用参数：

```xml

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>3.3.1</version>
    <executions>
        <execution>
            <id>attach-sources</id>
            <!-- 绑定verify插件到Maven的生命周期 -->
            <phase>verify</phase>
            <!--在生命周期后执行绑定的source插件的goals -->
            <goals>
                <goal>jar-no-fork</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

#### maven-shade-plugin

maven-shade-plugin 相较于 maven-jar-plugin，会将将依赖的jar包解压后打包到当前jar包，当class包名出现冲突的时候可以修改包路径。
参数配置文档: https://maven.apache.org/plugins/maven-shade-plugin/shade-mojo.html

常用参数：

```xml

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.5.3</version>
    <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>xxx</mainClass>
            </transformer>
        </transformers>
        <!-- 自动将所有不使用的类全部排除掉 -->
        <minimizeJar>true</minimizeJar>
        <artifactSet>
            <includes>
                <include>org.apache.maven:*</include>
            </includes>
            <excludes>
                <exclude>*:maven-core</exclude>
            </excludes>
        </artifactSet>
        <filters>
            <filter>
                <artifact>junit:junit</artifact>
                <includes>
                    <include>org/junit/**</include>
                </includes>
                <excludes>
                    <exclude>org/junit/experimental/**</exclude>
                </excludes>
            </filter>
        </filters>
        <!-- 将 org.codehaus.plexus.util 包前缀修改为 org.shaded.plexus.util -->
        <relocations>
            <relocation>
                <pattern>org.codehaus.plexus.util</pattern>
                <shadedPattern>org.shaded.plexus.util</shadedPattern>
                <excludes>
                    <exclude>org.codehaus.plexus.util.xml.Xpp3Dom</exclude>
                    <exclude>org.codehaus.plexus.util.xml.pull.*</exclude>
                </excludes>
            </relocation>
        </relocations>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

#### maven-assembly-plugin

参数配置文档: https://maven.apache.org/plugins/maven-assembly-plugin/single-mojo.html

常用参数：

```xml

<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-shade-plugin</artifactId>
   <version>3.7.1</version>
   <configuration>
      <!-- 打包文件名字不包含 assembly.xml 中 id -->
      <appendAssemblyId>false</appendAssemblyId>
      <descriptors>
         <descriptor>assembly/assembly.xml</descriptor>
      </descriptors>
      <archive>
         <manifest>
            <!-- springboot项目，会由springboot插件自动生成，无需配置 -->
            <mainClass>xxx</mainClass>
         </manifest>
      </archive>
   </configuration>
   <executions>
      <execution>
         <id>assembly-package</id>
         <phase>package</phase>
         <goals>
            <goal>single</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

assembly.xml配置：
```xml

<assembly>
   <id>assembly-package</id>

   <formats>
      <format>tar.gz</format>
   </formats>
   <!--是否生成和项目名相同的根目录 -->
   <includeBaseDirectory>true</includeBaseDirectory>

   <!--第三方依赖设置-->
   <!-- <dependencySets>
           <dependencySet>
               &lt;!&ndash;使用项目中的artifact,第三方包打包进tar.gz文件的lib目录下&ndash;&gt;
               <useProjectArtifact>true</useProjectArtifact>
               <outputDirectory>lib</outputDirectory>
           </dependencySet>
       </dependencySets>
   -->

   <fileSets>
      <!-- assembly/bin文件下的所有脚本文件输出到打包后的bin目录下 -->
      <fileSet>
         <directory>assembly/bin</directory>
         <outputDirectory>/</outputDirectory>
         <!-- 权限设置:
          0755->即用户具有读/写/执行权限，组用户和其它用户具有读写权限；
          0644->即用户具有读写权限，组用户和其它用户具有只读权限；
         -->
         <fileMode>0755</fileMode>
         <lineEnding>unix</lineEnding>
         <filtered>true</filtered>
      </fileSet>

      <!-- src/main/resources/config目录下配置文件打包到config目录下 -->
      <fileSet>
         <directory>src/main/resources</directory>
         <!--       <includes>
                         <include>application*.yml</include>
                         <include>application-${profile.env}.yml</include>
                         <include>application*.yaml</include>
                         <include>logback*.xml</include>
                     </includes>
                     <filtered>true</filtered>
         -->
         <outputDirectory>config</outputDirectory>
      </fileSet>

      <!-- 将target目录下的启动jar打包到目录下-->
      <fileSet>
         <directory>target</directory>
         <outputDirectory>/</outputDirectory>
         <includes>
            <include>*.jar</include>
         </includes>
      </fileSet>
   </fileSets>
</assembly>
```