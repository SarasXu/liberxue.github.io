---
layout: blog
technology: true
title:  "定制自定义maven-archetype"
background: red
background-image: https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1844720362,2711177956&fm=27&gp=0.jpg
date:   2017-03-09 16:13:54
category: 技术
tags:
- java
- maven
- archetype
---

# 项目搭建

1.   首先，搭建一个maven的模板项目，模板项目的内容可以根据公司和业务需求自行构建自己想要的模板。
2.   其次，项目为一个多模块的项目，子模块的命名如果需要变量则将需要替换的地方命名为```__artifactId__ ```。
3.   最后，在模板项目内管理好各个子模块的依赖和需要依赖的公共的JAR包以及需要的一些公共配置。
     如图，我自定义项目模板：
      ​  	![Alt text](https://sarasxu.github.io/blog/img/maven-archetype/1.png)

# 构建

1. 在项目根目录执行命令：```mvn archetype:create-from-project```，成功之后根目录会生成target文件。
2. 进入新生成`target/generated-sources/archetype`目录，如图。
3. 找到`src/main/resource/META-INF/maven下的archetype-metadata.xml`，如图。
   ![Alt text](https://sarasxu.github.io/blog/img/maven-archetype/2.png)


# 微调

1. 项目生成后，还需要在之前的文件内微调，微调的方式有很多，我这只是其中一种比较笨的方法。

2. 将子模块的id和name都更改为`{rootArtifactId}`，如`<module id="${rootArtifactId}" dir="__artifactId__-dal" name="${rootArtifactId}"></module>`
3. 然后进入`target/generated-sources/archetype/src/main/resource/archetype-resources`对各个子模块的pom进行微调。

4. 将子模块pom内的一些关于模块和项目名的命名进行替换，如
    ``` XML
        <dependency>
           <groupId>com.dph.archetype</groupId>
            <artifactId>__artifactId__-biz</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
        //替换为
         <dependency>
            <groupId>com.dph.${artifactId}</groupId>
            <artifactId>${artifactId}-biz</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
    ```
5. 最后再将子模块pom内的模块重新命名，将对应的模块，如facade的pom
 ```
 	<artifactId>${artifactId}</artifactId>
 	//更改为
 	<artifactId>${artifactId}-facade</artifactId>
 ```
6. 都更改之后，进入`target/generated-sources/archetype`目录，执行`mvn deploy`将定制的archetype发布到maven私库内。


# 生成

1. 进入想要生成项目的文件夹内，执行`mvn archetype:generate`，会出现很多maven archetype。
2. 找到自己的archetype，然后在控制台输入archetype的id。
3. 根据提示输入`groupId`、`artifactId`、`version`、`package`。
4. 最后输入Y，即可生成定制模板的项目。

# 项目地址

[maven-archetype](https://github.com/SarasXu/maven-archetype "maven-archetype")