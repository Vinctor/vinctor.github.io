
title: Gradle笔记（不定时更新）
date： 2017-11-05
---

# 命令 
gradle projects 查看工程项目信息
gradle tasks  查看task信息
gradle <project-path> :tasks  某个project中的 task信息  注意有冒号
 gradle <task-name> 执行某个task
gradlew :app:dependencies --configuration compile  //查看依赖树
gradlew sourseSets 查看sourseSets

# 工作流程
![image.png](http://upload-images.jianshu.io/upload_images/1583231-f8379b78795c6ac4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 初始化阶段：准备工作，比如执行setting.gradle，按照配置创建project的层次结构
* 配置阶段：解析每个project中的build.gradle文件，并绘制一个描述各个task执行顺序的有向图，为下一个执行步骤做准备
* 执行阶段：执行阶段

在每两个阶段中间，我们都可以添加自己的代码，在task中添加我们的代码，实现我们需要的额外功能。

# Gradle体系介绍

## Gradle对象
* 通过调用`Project.getGradle()`得到gradle的实例。
#### apply
void apply (map):map中每个键都是一个方法的名称：包括：
1. from：要应用的脚本。后面为可支持的路径。
2. plugin：需要使用的插件的id或实现类
3. to：目标委托对象

#### ext
添加额外属性：
*  gradle.ext.<properties-name>=xxxx
* `ext{   <properties-name>=xxx    }`

# Task

>Task是和Project关联的，所以，我们要利用Project的task函数来创建一个Task  
task myTask  <==myTask是新建Task的名字  
task myTask { configure closure }  
task myType << { task action } <==注意，<<符号是doLast的缩写  
task myTask(type: SomeType)  
task myTask(type: SomeType) { configure closure }  

上述代码中都用了Project的一个函数，名为task，注意：
*  一个Task包含若干Action。所以，Task有doFirst和doLast两个函数，用于添加需要最先执行的Action和需要和需要最后执行的Action。Action就是一个闭包。
*  Task创建的时候可以指定Type，通过type:名字表达。这是什么意思呢？其实就是告诉Gradle，这个新建的Task对象会从哪个基类Task派生。比如，Gradle本身提供了一些通用的Task，最常见的有Copy 任务。Copy是Gradle中的一个类。当我们：task myTask(type:Copy)的时候，创建的Task就是一个Copy Task。
*  当我们使用 task myTask{ xxx}的时候。花括号是一个closure。这会导致gradle在创建这个Task之后，返回给用户之前，会先执行closure的内容。
* 当我们使用task myTask << {xxx}的时候，我们创建了一个Task对象，同时把closure做为一个action加到这个Task的action队列中，并且告诉它“最后才执行这个closure”（注意，<<符号是doLast的代表）。

#### Script Block  
* subprojects：它会遍历posdevice中的每个子Project。在它的Closure中，默认参数是子Project对应的Project对象。由于其他SB都在subprojects花括号中，所以相当于对每个Project都配置了一些信息。
* allprojects  配置此项目及其各个子项目。
* buildscript  配置此项目的构建脚本类路径。用来配置所依赖的classpath等信息。它的closure是在一个类型为ScriptHandler的对象上执行的。在ScriptHandler中调用repositories和dependencies两个Script Block  
* repositories  配置该项目的存储库。
* dependencies  配置此项目的依赖关系。
















# -------------------------------------------
