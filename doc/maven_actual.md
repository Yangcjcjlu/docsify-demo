```
<?xml version="1.0" encoding="UTF-8"?>                                          
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <groupId>com.yang.mvnbook</groupId>
        <artifactId>hello-world</artifactId>
        <version>1.0-SNAPSHOT</version>
        <name>Maven Hello World Project</name>
/project>
```

这个是第一次实战编写的```POM```文件,第一行是xml头，指定了该xml文档的版本和编码方式，project元素是所有pom.xml文件的根元素，指明了POM的命名空间和xsd元素，能让ide中的xml编辑器帮助我们编辑xmL。

 groupId定义了项目属于哪个组，artifactId指明了项目在组中的唯一Id,version表明了项目的版本，name 并非必要，但其是对用户一个更为友好的项目名称
 
 1. maven 假设项目的主代码位于```src/main/java``` 目录， 然后我们在该目录下创建文件```com/yang/mvnbook/helloworld/HelloWorld.java``` 这样与pom 中的groupId 和 artifactId 相符
 2. 运行```mvn clean complie```, ```clean```告诉maven 清理target/目录，complie 告诉maven编译项目主代码
 3. maven 默认测试代码目录为```src/test/java```,在pom.xml中新增代码
 ```
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>  
            <version>4.7</version>
            <scope>test</scope>                                
         </dependency>
</dependencies>
```
 其中```scope``` 表示依赖范围，若依赖范围为test则表示该依赖只对测试有效，如果不声明scope，则默认```scope```为```complie```,表示依赖对主代码与测试代码同样有效
 4. 在src/test/java 下创建文件 HelloWorldTest.java
 ```
 package com.yang.mvnbook.helloworld;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class HelloWorldTest
{
   @Test
   public void testSayHello(){
     HelloWorld helloWorld = new HelloWorld();
     String result =  helloWorld.sayHello();
     assertEquals("Hello Maven",result);                                        
   }   
}
 ```
 之后在src目录执行```mvn clean test```，编译成功！
 书籍还给出一种编译失败的可能性，更新maven核心插件compiler支持1.8
 ```
 <build>
                 <plugins>
                        <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <configuration>
                                <source>1.8</source>
                                <target>1.8</target>
                        </configuration>
                </plugin>
                 </plugins>
                </build>
 ```
 5. 打包： ```mvn clean package``` 其中 jar: jar 任务负责打包，就是jar插件的jar目标将项目主代码打包成一个名为hello-world-1.0-SNAPSHOT.jar的文件，有需要的话可以用finalName来定义改文件的名称， 有需要的话可以复制这个jar文件到其他项目的classpath中从而使用HelloWorld， 使用```mvn clean install``` 让其他Maven项目直接使用jar,在打包之后又使用install:install 该任务将项目输出的jar安装到Maven本地仓库，结论是只有当构件被下载到本地仓库之后，其他项目才可以使用它
 6. 运行HelloWorld 项目，默认打包生成的jar是不能运行的，因为带有main方法的类信息不会添加到manifest(META-INF/MANIFEST.MF文件),生成可执行的jar文件，需要借助maven-shade-plugin
 ```
 <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-shade-plugin</artifactId>
                        <version>1.2.1</version>
                        <executions>
                                <execution>
                                        <phase>package</phase>
                                        <goals>
                                          <goal>shade</goal>
                                  </goals>
                                  <configuration>
                                  <transformers>
                                          <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                        <mainClass>com.yang.mvnbook.helloworld.HelloWorld</mainClass>
                                </transformer>
                                  </transformers>
                                  </configuration>
                          </execution>
                        </executions>
 </plugin>  
 ```
 此时执行```mvn install``` 会生成两个jar,hello-world-1.0.SNAPSHOT.jar和 original-hello-world-1.0.SNAPSHOT.jar,前者是可执行的，使用java -jar ...jar 可以看到输出
 
 7. 上述的所有步骤就是生成maven 的骨架，maven中可以使用archetype 来创建项目骨架,maven 3使用命令：```mvn archetype:generate```, 在空的项目中输入```groupId:com.yang.mvnbook```,```artifactId:hello-world```,```package com.yang.mvnbook.helloworld```创建之前项目一致的骨架
 

8. 完成了一个简单的spring项目（邮件的收发），体验由maven打包，已上传github

* 9. 依赖的基本元素： 
```
<project>
        <dependencies>
            <dependency>
                <groupId></groupId>
                <artifactId></artifactId>
                <version></version>
                <type></type>
                <scope></scope>
                <optional></optional>
                <exclusions>
                    <exclusion>
                    </exclusion>
                </exclusions>
            </dependency>
        </dependencies>
    </project>
```
   * 其中```groupId,artifactId,version```是依赖的基本坐标
   * ```type``` 是依赖的类型，对应于项目坐标定位的packaging,大部分情况不必声明默认为jar
   * ```scope```---```compile```,编译的依赖范围，如果没有指定，就会默认使用该范围编译，对于编译，测试，运行三种classpath都有效，---```test```,测试依赖范围，使用此范围的maven依赖只对测试有效,在编译主代码或者运行项目时无法使用此类依赖 ---```provided```,已提供范围依赖，使用此类范围依赖，对于编译和测试classpath有效，但在运行时无效，---```runtime```,运行范围依赖，测试运行时有效，编译时无效，---```system```,与```provided```一致，但依赖是通过```systemPath```元素显示指定依赖文件的路径，由于此类解析不是通过maven仓库解析，并往往与本机系统绑定，会造成构建的不可移植性，需要绑定```<systemPath>${java.home}/lib/rt.jar</systemPath>```
   * **依赖调整**，路径最近者优先，（2.0.9）相同路径，优先声明者优先
   * ```optional```--- 可选依赖不会传递，此时上一级项目需要显式的声明依赖，原因在于下级项目实现多个特性，这些特性都是互斥的，所有引用的项目需要进行选择，不过建议不要使用可选依赖，更好的做法是为多个选项建立不同的项目即基于同样的```groupId```选择不同```artifactId```
   * ```exclusions```---用于排除依赖的包重复引入相同的包，注意只要有```groupId```和```artifactId```即可，可以```<exclusion>```排除多个
   * 声明属性，```<properties>```,只需要修改一处就可以实现多处修改
   * ```mvn dependency :list```可以用来查看当前项目已解析依赖，```mvn dependency:tree```用来查看当前项目的依赖数，```mvn dependency:analyze```用来帮助分析项目的依赖，其中```unused decleared dependenices```指项目中未使用但显式声明的依赖，```used decleared dependencies```指项目中使用到但是未显式依赖的包，由于未声明，所以使用的是直接依赖，需要直接声明
  
10. maven 仓库
  * 设置修改maven本地仓库，修改.m2中```setting.xml```文件，
 ```
  <settings>
  <localRepository></localRepository>
  </settings>
```
  * 在```setting.xml```中配置仓库的认证信息，
  ```
   <settings>
        <servers>
            <server>
                <id></id>
                <username></username>
                <password></password>
            </server>
        </servers>
   </settings>
  ```
  
 * 在Pom配置构件部署地址
 ```
 <project>
    <distributionManagement>
        <repository>
            <id></id>
            <name></name>
            <url></url>
        </repository>
    </distributionManagement>
 </project>
 ```
 * 在```setting.xml```中配置镜像
 ```
 <settings>
    <mirrors>
       <mirror>
        <id></id>
        <name></name>
        <url></url>
        <mirrorOf></mirrorOf>
       </mirror>
    </mirrors>
 </settings>
 ```
 
 11. maven 生命周期
 * maven 有三套生命周期，clean,default和site,clean的目的是清理站点，default的目的是构建项目，site的目的是建立项目站点。
 * clean --- pre-clean,clean,post-clean
 * default 
 * site
```
mvn clean 调用clean生命周期clean阶段，pre-clean到clean
```
```
mvn test 调用default生命周期的直到test阶段
```
```
mvn clean install 调用clean生命周期直到clean阶段，调用default从validate 到install 全部阶段
```
```
mvn clean deploy site-deploy
调用clean生命周期直到clean,调用default全部生命周期，调用site全部生命周期
```

 ```
 <plugin>
    <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-source-plugin</artifactId>
          <version>2.2.1</version>
            <executions>
                    <execution>
                        <id>attach-sources</id>
                        <phase>verify</phase>
                   <goals>
                          <goal>jar-no-fork</goal>
                   </goals>  
                   </execution>
            </executions>
</plugin>
 ```
 这段示例代码展示了自定义绑定的方式，详情看7.4.2
 
 12. 命令行插件配置
  在命令行中加-D并伴随参数键=参数值的形式来配置插件目标的参数，eg. ```mvn install -Dmaven.test.skip=true```能够跳过测试
 
 13. maven还支持直接从命令行调用插件目标，eg.
 ```
 mvn help:describe-Dplugin=compiler
 
 mvn dependency:tree
 ```
