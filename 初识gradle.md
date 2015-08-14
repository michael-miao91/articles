#### 1.gradle和maven的区别
为什么要用gradle来创建java或者是spring的项目呢？在之前或许你也曾听说过`Ant`和`maven`这两个词，其实它们都是项目构建工具。而且之前`maven`一直都是大紫大红的，为什么现在又出现了`gradle`这样一个新的构建工具呢？它和`maven`这两者又有一些什么样的区别呢？

我觉得在jar包的管理上就有一个很大的区别，在`maven`中如果我想引用`spring-context`这个jar包。那么，需要下面的语句：

```xml
<dependencies>
    <dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-context</artifactId>
	    <version>4.2.0.RELEASE</version>
    </dependency>
</dependencies>
```
但是，如果同样的做法用在`gradle`上，就只是短短的下面几句代码：

```
dependencies {
    compile(
        'org.springframework:spring-context:4.2.0.RELEASE',
    )
}
```

可以看见，在对于包的依赖管理上。仅仅基于配置而言，`gradle`的代码量就会大大减少。

那么接下来，就来看一看如何利用gralde创建一个java项目。其实在主面板上按照`create new project`，之后选择`gradle`来创建项目。一步一步按照要求来做，就ok了。当创建一个完整的java项目后，可以看到他的目录结构和下面的差不多：

![gradle目录](http://7xjqpv.com1.z0.glb.clouddn.com/gradle.png?imageView2/1/w/400/h/700/q/75)

#### 2.resource还是resources？
在这个问题上，当我自己在尝试的时候，还是遇到了一些麻烦。因为我把那个`resources`目录错误的写成`resource`，然后当我在其目录下新建一个`spring.xml`的配置文件，接着如果我在`main`方法中调用下面的语句：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
```
会出现`java.io.FileNotFoundException: class path resource [spring.xml] cannot be opened because it does not exist`这样的错误，因为如果是`resource`这个目录，当我们在执行`gradle build`这条命令，它其实并没有把相应的配置文件放在build文件夹下。所以，在这种情况下，如果我们要获取`spring.xml`文件，可以用下面的一种方法（直接传递这个配置文件的路径）：

```java
ApplicationContext context = new FileSystemXmlApplicationContext("/src/main/resource/spring.xml");
```

#### 3.build.gradle文件解析
那么之后我们可以看一下，`build.gradle`文件到底是一个怎么样的文件？它的内容如下所示：

```
apply plugin: 'java'

sourceCompatibility = 1.5

repositories {
    mavenCentral()
}

dependencies {
    compile(
            'org.slf4j:slf4j-api:1.7.12',
            'org.apache.commons:commons-lang3:3.4',
            'org.springframework:spring-context:4.2.0.RELEASE',
            'org.springframework:spring-core:4.2.0.RELEASE',
            'org.springframework:spring-beans:4.2.0.RELEASE',
    )
    testCompile group: 'junit', name: 'junit', version: '4.11'
}

```
其中一些关键词的含义如下:

1. apply plugin: 'java': 指定项目为java项目，项目编译(在项目提示符下执行：gradle build)时生成项目的jar包。
2. sourceCompatibility = 1.5: 指的是源代码的JDK版本。
3. repositories {mavenCentral()}: 指的是申明仓库的源，这里指明的是 mavenCentral()。
4. 接下来的配置就是一些相应的dependencies, 需要什么额外的jar包，可以在[这个网站](http://mvnrepository.com/)上找。

####4. gradle最常用的命令

1. `gradle build`: 编译项目，生成build文件夹，并生成相应的jar包。注意：这个命令好像会跑相应的`test`。
2. `gradle clean`: 与build相反，删除build文件夹。


####5. gralde和./gradlew的区别

./表示当前目录，而gradlew表示的`gradle wrapper`这个文件夹，它其实就是gradle的一层包装，可以理解为在这个项目本地就封装了gradle。所以它们两者大概代表着同一个意思，只不过是用`./gradlew`命令代替了全局的`gradle`命令。
