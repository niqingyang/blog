---
title: "Gradle – 生成 Java 类的 HTML API 文档"
date: 2019-01-25T23:57:52+08:00
draft: false
categories: 
    - develop
tags: 
    - gradle
    - java
    - javadoc
# themeColor: "#8bb02e"
coverColor: "#9bc333f7"
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410224857-gradle-javadoc.png
---

<info>

> Gradle is power !

</info>

## 构建单项目的 API 文档

在 build.gradle 中加入如下代码

```groovy
// 生成Java类的HTML API文档
task javadocs(type: Javadoc) {
    // 默认值
    source sourceSets.main.allJava
	// 生成文档的目标目录
    destinationDir = new File(buildDir, "docs")
	// 默认值
    classpath = sourceSets.main.compileClasspath
	// 设置发生错误不会导致生成文档失败
    failOnError = false
    excludes = ['**/generated/**']
    options.addStringOption('encoding', 'UTF-8')
    options.addStringOption('charSet', 'UTF-8')
}

task javadocsIntoJar(type: Jar, dependsOn: javadocs) {
    classifier = "javadoc"
    from javadoc
}

artifacts {
    archives javadocsIntoJar
}
```

## 构建多项目的 API 文档，并且汇总输出到一个目录下

在 build.gradle 中加入如下代码

```groovy
// 生成Java类的HTML API文档
task javadocs(type: Javadoc, description: "Aggregate all Javadocs into a single directory") {
	// 指向所有子项目
    source subprojects.collect { project -> project.sourceSets.main.allJava }
	// 生成文档的目标目录
    destinationDir = new File(buildDir, "javadoc")
	// 指向所有子项目
    classpath = files(subprojects.collect { project -> project.sourceSets.main.compileClasspath })
    failOnError = false
    excludes = ['**/generated/**']
}

task aggregateJavadocsIntoJar(type: Jar, dependsOn: javadocs, description: "Aggregate all Javadocs into a single directory") {
    classifier = "javadoc"
    from javadoc
}

artifacts {
    archives aggregateJavadocsIntoJar
}
```

参考项目：[https://github.com/apereo/cas](https://github.com/apereo/cas "https://github.com/apereo/cas")
参考文档：[https://docs.gradle.org/5.1/dsl/org.gradle.api.tasks.javadoc.Javadoc.html](https://docs.gradle.org/5.1/dsl/org.gradle.api.tasks.javadoc.Javadoc.html "https://docs.gradle.org/5.1/dsl/org.gradle.api.tasks.javadoc.Javadoc.html")