## spring boot

```
src
	- main
 		- java
        - resources
	- test
		- java
		- resources
		
target // maven输出目录

pom.xml
```





## maven工程

```
src
	- main
 		- java
        - resources
	- test
		- java
		- resources
		
target // maven输出目录

pom.xml
```

项目生命周期

【清理】-【编译】-【测试】-【报告】-【打包】-【部署】

```
mvn clean # 构建并清理输出target目录
	  mvn compile
			   mvn test

									  mvn install # 打包发布到本地仓库
```



mvn clean package依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)等７个阶段。
mvn clean install依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)、install等8个阶段。
mvn clean deploy依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)、install、deploy等９个阶段。

分析解释如下：
package命令完成了项目编译、单元测试、打包功能，但没有把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库
install命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库，但没有布署到远程maven私服仓库
deploy命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库　



## 注解

```java
@Resource // jdk注解

@Autowired // spring注解
@Qualifier // 合格者（一个interface多个实现类，具体装配哪个用name决定）

这两个注解的区别是：@Autowire 默认按照类型type，如果我们想使用按照名称name装配，可以结合@Qualifier注解一起使用;

@Resource默认按照名称装配，当找不到与名称匹配的bean才会按照类型装配，可以通过name属性指定，如果没有指定name属 性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找 依赖对象.
```



#