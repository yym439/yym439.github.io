---
layout:     post
title:      AS发布aar包到JCenter
subtitle:   AS发布aar包到JCenter
date:       2020-12-03
author:     yym439
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - Android
---

### 一、注册登录JCenter

[JCenter](https://bintray.com/signup/oss)


### 二、发布流程

#### JCenter配置

1. 登录JCenter，获取用户名及API key (进入Edit Profile)
2. Add New Repository
    - 勾选：Public - anyone can download your files
    - Type选择：Maven
3. 进入Repository，创建Package
    - Name：要和配置文件中，上传的项目名一致
    - Version control: https://github.com/yym439/mylibrary.git

#### AS配置

1. 库工程创建bintray.gradle：

    ```
    apply plugin: 'com.jfrog.bintray'

    version = '1.0.0' //发布的版本号

    task sourcesJar(type: Jar) {
        from android.sourceSets.main.java.srcDirs
        classifier ('sources')
    }

    task javadoc(type: Javadoc) {
        source = android.sourceSets.main.java.srcDirs
        classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    }

    task javadocJar(type: Jar, dependsOn: javadoc) {
        classifier('javadoc')
        from javadoc.destinationDir
    }

    artifacts {
        archives javadocJar
        archives sourcesJar
    }

    bintray {

        user = "******"//用户名
        key = "******"//jcenter的Api key

        pkg {
            repo = 'jcenterDemo' //需要上传的仓库名
            name = 'mylibrary' //上传的项目名
            configurations = ['archives']
            desc = 'An android library.'
            websiteUrl = ''
            vcsUrl = ''//github对应地址，可以不写
            licenses = ["MIT"]
            publish = true
            publicDownloadNumbers = true
        }
    }
    ```

2. 库工程创建install.gradle：

    ```
    apply plugin: 'com.github.dcendents.android-maven'

    group = 'com.demo.mylibrary'

    install {
        repositories.mavenInstaller {
            pom {
                project {
                    licenses {
                        license {
                            name 'The MIT License'
                            url 'https://choosealicense.com/licenses/mit/'
                        }
                    }
                }
            }
        }
    }
    ```
3. 库工程的build.gradle添加：

    ```
    apply from: 'install.gradle'
    apply from: 'bintray.gradle'
    ```
4. 应用工程的build.gradle添加：

    ```
    dependencies {
        classpath "com.android.tools.build:gradle:4.1.1"

        classpath 'com.github.dcendents:android-maven-gradle-plugin:2.1'
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.8.5'
    }
    ```

#### 编译发布

1. 打开AS Terminal输入：

    ```
    gradlew :mylibrary:install
    gradlew :mylibrry:bintrayUpload
    ```
2. 上传成功后，进入JCenter仓库，Linked to - Add to JCenter 


### 三、常见问题

1. [add to jcenter失败](https://www.jianshu.com/p/44f3c333ce3c)：Failed to send a message: The version control 1.0.0 returns 404
    
    >修改库的VCS，为自己github创建的工程地址