---
layout: post
title: "gradle发布到github packages"
categories: android
author: "kekxv"
---

最近弄了个`Android`的模拟自动注入的库，在给别人用的时候，需要发送 `jar` 或者 `aar` 给对方，这就导致我可能需要保留一份，否则每次都需要重新编译生成。为了 ~~偷懒~~
增加效率，在查询资料之后，将其发布到了`github packages`。

准备工作：需要`github`账号（废话），生成`GitHub token`需要有写权限（如果只是使用不需要，如果需要推送则需要），将账号(`GITHUB_USER`)
和`token`(`GITHUB_PERSONAL_ACCESS_TOKEN`)配置到用户目录下`~/.gradle/gradle.properties`。

> 注意事项：
>
> 如果发布返回4**系列错误，原因可能是这几个：
> 1. `GroupPageId`或者`ArtificatId`使用了大写，需要改为小写。
> 2. 当前`GroupPageId`或者`ArtificatId`已经在使用了！！！，需要修改，或者把在使用的删掉。。。。
> 3. 账号密码(`Token`)错误。

新建或者使用已有仓库，新建 `Android library` 项目（`id 'com.android.library'`）。

修改`library`项目的`build.gradle`配置（不要改错了）:

1.

> ```
> plugins {
>   id 'com.android.library'
> }
> ```

改为

> ```
> plugins {
>   id 'com.android.library'
>   id 'maven-publish'
> }
> ```

2. 增加函数定义

> ```
> //def githubProperties = new Properties()
> //githubProperties.load(new FileInputStream(rootProject.file("github.properties"))) //Set env variable GPR_USER & GPR_API_KEY if not adding a properties file
> def getGroupPageId = { ->
>     return "com.kekxv" // Replace with version Name
> }
> def getVersionName = { ->
>     return "0.2.4" // Replace with version Name
> }
> def getArtificatId = { ->
>     return "autowired" // Replace with library name ID
> }
> ```

3. 增加

>
> ```
> task sourceJar(type: Jar) {
>     from android.sourceSets.main.java.srcDirs
>     classifier "sources"
> }
> publishing {
>     publications {
>         // Creates a Maven publication called "release".
>         release(MavenPublication) {
>             groupId getGroupPageId()
>             artifactId getArtificatId()
>             version getVersionName()
>             artifact("$buildDir/outputs/aar/${getArtificatId()}-release.aar")
>
>             artifact(sourceJar)
>             pom.withXml {
>                 final dependenciesNode = asNode().appendNode('dependencies')
>                 ext.addDependency = { Dependency dep, String scope ->
>                     if (dep.group == null || dep.version == null || dep.name == null || dep.name == "unspecified")
>                         return
>                     final dependencyNode = dependenciesNode.appendNode('dependency')
>                     dependencyNode.appendNode('groupId', dep.group)
>                     dependencyNode.appendNode('artifactId', dep.name)
>                     dependencyNode.appendNode('version', dep.version)
>                     dependencyNode.appendNode('scope', scope)
>                     if (!dep.transitive) {
>                         final exclusionNode = dependencyNode.appendNode('exclusions').appendNode('exclusion')
>                         exclusionNode.appendNode('groupId', '*')
>                         exclusionNode.appendNode('artifactId', '*')
>                     } else if (!dep.properties.excludeRules.empty) {
>                         final exclusionNode = dependencyNode.appendNode('exclusions').appendNode('exclusion')
>                         dep.properties.excludeRules.each { ExcludeRule rule ->
>                             exclusionNode.appendNode('groupId', rule.group ?: '*')
>                             exclusionNode.appendNode('artifactId', rule.module ?: '*')
>                         }
>                     }
>                 }
>                 configurations.compile.getDependencies().each { dep -> addDependency(dep, "compile") }
>                 configurations.api.getDependencies().each { dep -> addDependency(dep, "compile") }
>                 configurations.implementation.getDependencies().each { dep -> addDependency(dep, "runtime") }
>             }
>         }
>     }
>
>     repositories {
> //        maven {
> //            url "$buildDir/repo"
> //        }
>         maven {
>             name = "GitHubPackages"
>             url = uri("https://maven.pkg.github.com/用户名所有者/仓库名/")
>             credentials {
>                 username = System.getenv('GITHUB_USER') ?: project.properties['GITHUB_USER']
>                 password = System.getenv('GITHUB_PERSONAL_ACCESS_TOKEN') ?: project.properties['GITHUB_PERSONAL_ACCESS_TOKEN']
>             }
>         }
>     }
> }
>
> publish.dependsOn(build)
> ```
>

配置完成之后，在`AndroidStudio`右侧标签点击 `Gradle` ，选择`library`项目，点开`publishing`，执行`publish`，如果没有意外并且账号(`GITHUB_USER`)
和`token`(`GITHUB_PERSONAL_ACCESS_TOKEN`)配置正确，你将可以在你的仓库`packages`
可以看到项目，例如：[https://github.com/kekxv/JavaRepo/packages](https://github.com/kekxv/JavaRepo/packages) 。

## 使用方式

**引入方式**：

>
> 以下配置均在项目`build.gradle`下
>
> 方式一
>
> 1. [生成 GitHub Token，教程点击本链接](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token)
> 2. 配置 `gradle` ，增加仓库`https://maven.pkg.github.com/kekxv/JavaRepo/` ，github 的仓库需要授权（公开的也要），自己配置一下 `GITHUB_USER`、`GITHUB_PERSONAL_ACCESS_TOKEN`，建议在家目录的`gradle.properties`进行配置。
> ```
> //配置gradle
> android {
>     repositories {
>         maven {
>             name = "GitHubPackages"
>             url = uri("https://maven.pkg.github.com/kekxv/JavaRepo/")
>             credentials {
>                 username = System.getenv('GITHUB_USER') ?: project.properties['GITHUB_USER']
>                 password = System.getenv('GITHUB_PERSONAL_ACCESS_TOKEN') ?: project.properties['GITHUB_PERSONAL_ACCESS_TOKEN']
>             }
>         }
>     }
> }
> ```
> 3. 增加 `dependencies` `implementation 'com.kekxv:autowired:0.2.3'` (`0.2.3`为版本号，可更改为最新版本)
>
> 方式二
>
> 直接下载前往 [仓库](https://github.com/kekxv/JavaRepo/packages) 下载`autowired-0.2.3.aar`导入到项目。
> 根据情况，可能需要将`autowired-0.2.3.aar` 拷贝到`libs` 目录并 `implementation fileTree(dir: "libs", include: ["*.jar"])` 更改为`implementation fileTree(dir: "libs", include: ["*.jar","*.aar"])`
>


参考文档：

- [配置 Gradle 用于 GitHub 包](https://docs.github.com/cn/free-pro-team@latest/packages/guides/configuring-gradle-for-use-with-github-packages)
- [Android Libraries on GitHub Packages](https://proandroiddev.com/android-libraries-on-github-packages-21f135188d58)