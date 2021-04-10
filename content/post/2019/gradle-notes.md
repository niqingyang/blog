---
title: "Gradle 学习笔记"
date: 2019-01-13T19:00:21+08:00
draft: false
categories:
    - develop
tags:
    - gradle
    - java
themeColor: "#4a90e2"
coverColor: "#2e5e86f7"
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410224913-gradle.png
---

## 创建一个新的Gradle构建

```shell
## 在项目目录下执行次命令，一路回车后，将会生成一个简单的项目
gradle init

## 生成的目录结构
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── settings.gradle
```

<info>

> **Gradle Wapper 是什么？为什么建议使用？怎么使用？**
> 详见文档 《[Gradle Wrapper](https://docs.gradle.org/5.0/userguide/gradle_wrapper.html#sec:wrapper_generation "Gradle Wrapper")》

</info>

## 生成 java 标准目录结构
```shell
## --type参数支持的选项：
## 'basic', 'groovy-library', 'java-application', 'java-library', 'pom', 'scala-library'
gradle init --type java-library

## 生成 eclipse 项目配置文件
## 一般在根项目中定义插件，然后在项目根目录下执行，会自动对所有子项目进行处理
gradle eclipse
gradle eclipseWtp

## 生成 idea 配置
gradle idea
```
生成的 java 标准目录结构和 eclipse 配置
```shell
├─.settings
├─bin
│  ├─main
│  │  └─acme
│  │      └─demo
│  └─test
│      └─acme
│          └─demo
├─build
│  ├─classes
│  │  └─java
│  │      ├─main
│  │      │  └─acme
│  │      │      └─demo
│  │      └─test
│  │          └─acme
│  │              └─demo
│  ├─libs
│  ├─reports
│  │  └─tests
│  │      └─test
│  │          ├─classes
│  │          ├─css
│  │          ├─js
│  │          └─packages
│  ├─test-results
│  │  └─test
│  │      └─binary
│  └─tmp
│      ├─compileJava
│      ├─compileTestJava
│      └─jar
└─src
    ├─main
    │  ├─java
    │  │  └─acme
    │  │      └─demo
    │  └─resources
    └─test
        ├─java
        │  └─acme
        │      └─demo
        └─resources
```

## 简单的任务

1. Copy - 将文件从一个位置复制到另一个位置（[详见文档](https://docs.gradle.org/4.10-rc-2/dsl/org.gradle.api.tasks.Copy.html "详见文档")）

**build.gradle**
```groovy
task copyDocs(type: Copy) {
    from 'src/main/doc'
    into 'build/target/doc'
}

//for Ant filter
import org.apache.tools.ant.filters.ReplaceTokens

//for including in the copy task
def dataContent = copySpec {
    from 'src/data'
    include '*.data'
}

task initConfig(type: Copy) {
    from('src/main/config') {
        include '**/*.properties'
        include '**/*.xml'
        filter(ReplaceTokens, tokens: [version: '2.3.1'])
    }
    from('src/main/config') {
        exclude '**/*.properties', '**/*.xml'
    }
    from('src/main/languages') {
        rename 'EN_US_(.*)', '$1'
    }
    into 'build/target/config'
    exclude '**/*.bak'

    includeEmptyDirs = false

    with dataContent
}
```

2. Zip - 将目录压缩成一个zip文件并输出

**build.gradle**
```groovy
task zip(type: Zip) {
    from "src"
    setArchiveName "acme-1.0.zip"
}
```

## 常用命令

```shell
## 查看所有任务
gradlew tasks

## 查看项目属性
gradlew properties
```

## 创建多项目构建

<info>

> 多项目构建有助于模块化。它允许一个人专注于一个更大的项目中的一个工作区域，而项目其他部分的依赖交由Gradle负责。
> 项目详细配置见 《[project api document](https://docs.gradle.org/4.10-rc-2/dsl/org.gradle.api.Project.html#org.gradle.api.Project:allprojects(groovy.lang.Closure) "project api document")》

</info>

**build.gradle**

```groovy

// 项目的名称
name 'acme'

// 此项目的默认任务的名称。在启动构建时未提供任务名称时使用这些
defaultTasks "clean", "build"

// 所有额外属性必须通过“ext”命名空间定义。一旦定义了额外的属性，它就可以直接在拥有对象上获得，并且可以读取和更新。只需要通过命名空间完成的初始声明。
ext {

}

// 应用于所有子项目和根项目的配置
allprojects {

    apply plugin: 'java'
    apply plugin: 'java-library'
    apply plugin: 'signing'
	// Jacoco 是一个免费的 Java 单元测试覆盖率分析工具，在 Gradle 中添加插件，在编译的同事进行单元测试覆盖率分析
    apply plugin: 'jacoco'
    apply plugin: 'eclipse'
    apply plugin: 'eclipse-wtp'
    apply plugin: 'idea'

	jacoco {
        toolVersion = "$gradleJacocoVersion"
    }

    signing {
        required = isArtifactSigningRequired
        sign configurations.archives
    }

	// idea的一些配置
	idea {
        module {
            downloadSources = false
            downloadJavadoc = false
            excludeDirs << file(".gradle")
            ["classes", "bin", "docs", "dependency-cache", "libs", "reports", "resources", "test-results", "tmp"].each {
                excludeDirs << file("$buildDir/$it")
            }
        }
    }

    // 分组
    group = 'top.acme'
    // 版本号
    version = '1.0'

    repositories {
        // 本地仓库
        mavenLocal()
        // maven 仓库
        mavenCentral()
        // jcenter 仓库
        jcenter()
    }

    // 依赖
    dependencies {
        //
        implementation 'com.google.guava:guava:26.0-jre'

        // Use JUnit test framework
        testImplementation 'junit:junit:4.12'
    }

    // 生成Java类的HTML API文档
	// 参考：https://docs.gradle.org/4.10-rc-2/dsl/org.gradle.api.tasks.javadoc.Javadoc.html
    javadoc {
        options.addBooleanOption('html5', true)
        failOnError = Boolean.getBoolean("ignoreJavadocFailures")
        excludes = ['**/generated/**']
    }
}

// 应用于所有子项目的配置
subprojects {

}

// 这个项目的直接孩子
childProjects {

}
```

**settings.gradle**

```groovy
// 根项目的名称
rootProject.name = 'acme'

include 'project A'
include 'project B'
include 'project C'
```

通过遍历项目目录自动识别并生成 `settings.gradle` 配置

```groovy
rootProject.name = 'acme'

FileTree buildFiles = fileTree(rootDir) {
	List excludes = gradle.startParameter.projectProperties.get("excludeProjects")?.split(",")
	include '**/*.gradle'
	exclude 'build', '**/gradle', 'settings.gradle', 'buildSrc', '/build.gradle', '.*', 'out'
	exclude '**/grails3'
	if(excludes) {
		exclude excludes
	}
}

String rootDirPath = rootDir.absolutePath + File.separator
buildFiles.each { File buildFile ->

	boolean isDefaultName = 'build.gradle'.equals(buildFile.name)
	if(isDefaultName) {
		String buildFilePath = buildFile.parentFile.absolutePath
		String projectPath = buildFilePath.replace(rootDirPath, '').replace(File.separator, ':')
		include projectPath
	} else {
		String projectName = buildFile.name.replace('.gradle', '');
		String projectPath = ':' + projectName;
		include projectPath
		def project = findProject("${projectPath}")
		project.name = projectName
		project.projectDir = buildFile.parentFile
		project.buildFileName = buildFile.name
	}
}
```

## 核心插件

```groovy
// https://docs.gradle.org/4.10-rc-2/dsl/org.gradle.api.Project.html#N1506F
apply plugin: 'java'
// https://docs.gradle.org/4.10-rc-2/dsl/org.gradle.api.Project.html#N14F08
apply plugin: 'application'
// https://docs.gradle.org/4.10-rc-2/dsl/org.gradle.api.Project.html#N1521A
apply plugin: 'war'
// https://docs.gradle.org/4.10-rc-2/dsl/org.gradle.api.Project.html#N1504B
apply plugin: 'jacoco'
// https://docs.gradle.org/4.10-rc-2/dsl/org.gradle.api.Project.html#N151D2
apply plugin: 'signing'

apply plugin: 'eclipse'

apply plugin: 'eclipse-wtp'

apply plugin: 'idea'
```