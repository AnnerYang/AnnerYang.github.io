---
layout: post
title: IntellijJ IDEA使用
categories: Tools
description: IntellijJ IDEA使用
keywords:  IDEA
---

记录日常使用IntellijJ IDEA出现的问题以及解决方法

## IDEA创建Maven-web项目

1.打开IntellijJ IDEA,选择File——new——project——Maven——勾选Create from archetype——org.apache.cocoon：cocoon-22-archetype-webapp——然后点击next

![image-20200410215811666](/images/posts/tools/idea/image-20200410215811666.png)

2.填写项目名称,项目存放路劲,公司名称

![image-20200410220126955](/images/posts/tools/idea/image-20200410220126955.png)

3.设置Maven目录,配置文件以及本地仓库路径

![image-20200410220426652](/images/posts/tools/idea/image-20200410220426652.png)

4.创建完成后目录结构如下

![image-20200410220739059](/images/posts/tools/idea/image-20200410220739059.png)

5.补全java目录结构

![image-20200410223554645](/images/posts/tools/idea/image-20200410223554645.png)

6.如果目录结构还不出来直接按图操作一波

![img](/images/posts/tools/idea/70)

7.为项目配置编译路径和artifact

![img](/images/posts/tools/idea/prizhi.jpg)

![image-20200410221552157](/images/posts/tools/idea/image-20200410221552157.png)

![image-20200410221648603](/images/posts/tools/idea/image-20200410221648603.png)

8.配置Tomcat，上面目录结构都出来了基本就没有坑了，现在我们给它配置个tomcat---按图操作吧

![image-20200410221800467](/images/posts/tools/idea/image-20200410221800467.png)

![image-20200410221848653](/images/posts/tools/idea/image-20200410221848653.png)

![image-20200410222210524](/images/posts/tools/idea/image-20200410222210524.png)

![image-20200410222445784](/images/posts/tools/idea/image-20200410222445784.png)

![image-20200410222530314](/images/posts/tools/idea/image-20200410222530314.png)

## Idea创建Maven Web工程的web.xml版本问题解决

一、问题描述
1.在使用Maven创建web工程的时候发现默认web.xml版本居然是2.4的，这个版本连EL表达式都用不了，所以很是糟心

2.所以为了解决Idea创建Maven Web工程的web.xml版本问题，给大家提供了两种解决办法

二、问题分析
1.首次创建Maven工程，会联网下载web有关的jar包，其中最重要的一个就是我们创建工程的时候选择的maven-archetype-webapp-1.3.jar这个jar包

2.在IdeanMaven web项目中生成的web.xml文件就是从该jar包中拷贝出来的，所以我们要做的就是改动web.xml和此jar包

三、问题解决
3.1 暂时解决
1.暂时解决方法只能解决当前项目，新建一个项目还会出现这个问题

2.要做的就是将项目的web.xml头换成需要的版本，比如我换成4.0版本

将需要的版本头替换原来的2.4版本头


需要4.0头的可以直接在这里复制
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
</web-app>
3.不建议随便粘贴一个头替换原来的web.xml头，最好是根据自己的服务器如Tomcat的版本来替换，推荐从Tomcat服务器中的web.xml中把头部分粘贴过来进行替换或者直接将web.xml文件拷贝过来替换为原来的web.xml，图示

在Tomcat安装目录下的webapp/ROOT/WEB-INF中有我们需要的web.xml

把不需要的部分删除，就可以得到我们需要的部分，也可以不删除，没什么影响
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
</web-app>
4.重启Idea 的服务器，如Tomcat，问题解决

3.2 永久解决
1.上述方法只能解决一个项目问题，但是我们并不想每次创建web项目都要像上面一样，很麻烦，所以我们这里永久性解决

2.我们创建web项目的时候发现使用:分隔了一个路径和jar包名称，前者其实就是Maven仓库坐标，后者就是web项目核心jar包

3.根据提供的坐标（路径）找到maven-archetype-webapp这个jar包

我的路径：d:\maven\MavenRepository\.m2\repository\org\apache\maven\archetypes\maven-archetype-webapp\1.3\


4.我们使用压缩软件打开这个jar包，注意是打开而不是解压，如使用2345好压打开，依次进入以下路径到WEB-INF目录中就可以看到有一个web.xml文件

选择打开


依次进入maven-archetype-webapp-1.3.jar\archetype-resources\src\main\webapp\WEB-INF目录中，找到web.xml

5.双击打开，注意不是解压！将此web.xml的头内容替换为我们需要的头信息（也可以直接删除这个web.xml，然后直接从Tomcat安装目录下的webapp/ROOT/WEB-INF中将web.xml给复制过来替换原来的web.xml），两种方式都行

6.修改完成，保存，然后关闭打开的文件，这个时候压缩软件会提示信息已经改变，是否重新压缩，选择是，修改完成

7.重新创建web工程，出现的web.xml就是我们刚刚修改的web.xml

## IDEA - 设置所有文件编码为UTF-8格式

![image-20200411094004460](/images/posts/tools/idea/image-20200411094004460.png)

![image-20200411094132056](/images/posts/tools/idea/image-20200411094132056.png)

![image-20200411094206439](/images/posts/tools/idea/image-20200411094206439.png)

Transparent native-to-ascii conversion属性主要用于转换ascii，不然Properties文件的中文会被转码，IntelliJ IDEA除了支持对整个Project设置编码之外，还支持对目录、文件进行编码设置。如果你要对目录进行编码设置的话，可能会出现需要Convert编码的弹出操作选择，强烈建议在转换之前做好文件备份，不然可能出现转换过程变成乱码，无法还原。对单独文件的编码修改还可以点击右下角的编码设置区，如果代码内容中包含中文，则会弹出演示中的操作选择，Reload 表示使用新编码重新加载，新编码不会保存到文件中，重新打开此文件，旧编码是什么依旧还是什么，Convert 表示使用新编码进行转换，新编码会保存到文件中，重新打开此文件，新编码是什么则是什么。![image-20200411094410832](/images/posts/tools/idea/image-20200411094410832.png)

## 关闭Intellij IDEA自动更新

在File->Settings->Appearance & Behavior->System Settings->Updates下取消Automatically check updates for勾选

![image-20200411094849368](/images/posts/tools/idea/image-20200411094849368.png)

## 隐藏.idea文件夹和.iml等文件

ntelliJ IDEA项目会自动生成一个.idea文件夹和.iml文讲，看着实在是碍眼，所以对以上文件进行隐藏处理
在File->Settings->Editor->File Types下的”Ignore files and folders”一栏添加 *.idea;*.iml;等配置如下图所示

![image-20200411095121563](/images/posts/tools/idea/image-20200411095121563.png)

## 代码格式化

代码格式化的快捷键为Ctrl+Alt+L，如果在类中执行代码格式化则会对代码进行排版，若焦点在类或者文件夹上，则会弹出格式化选项提示框，弹出框如下图所示:

![代码格式化](/images/posts/tools/idea/SouthEast.jpg)

```
1. Include subdirectories:是否对子目录也进行格式化
2. Optimize imports:优化导入的类和包
3. Rearrange enries:对代码顺序进行调整(将Filed放在Method前边)
4. Filters即配置过滤条件，表示对哪些文件进行格式化
```

## 自动导入所有包

在Intellij IDEA一次只能导入单个包，没有像Eclipse快速导入包的快捷键Ctrl+Shift+O，但是Intellij IDEA下有个自动导入包的功能。在File->Settings->Editor->General->Auto Import下进行配置，具体配置如下如所示:

```
Insert imports on paste:复制代码的时候，对于导入的包是否需要进行询问的一个选项。
    ASK(有需要导入的包名时会弹提示框，问你要不要导入)
    NONE(有需要导入的包名时不会弹提示框，也不会自动导入)
    ALL(有需要导入的包名时会自动导入，不会弹提示框)
Show import popup:当输入的类的声明没被导入时，会弹出一个选择的对话框
Optimize imports on fly:自动优化包导入，移除不需要的包
Add unambiguous imports on the fly:这个就是自动导入功能了，当你输入类名后声明就被自动导入了
Exclude from Import and Completion:这个其实就是你自定义import,可以不用关注，一般来说你是用不上的
```

![image-20200411100033373](/images/posts/tools/idea/image-20200411100033373.png)

## 生成serialVersionUID

默认情况下Intellij IDEA关闭了继承了Java.io.Serializable的类生成serialVersionUID的警告，如果需要提示生成serialVersionUID，那么需要做以下设置:在File->Settings->Editor->Inspections下勾选中Java->Serialization issues->Serializable class without ‘serialVersionUID’，将光标放到类名上按Atl＋Enter键就会提示生成serialVersionUID了

![生成serialVersionUID](/images/posts/tools/idea/SouthEast1.jpg)

## 代码提示忽略大小写

在File->Settings->Editor->General->Code Completion下设置Case sensitive completion为none

![image-20200411100523172](/images/posts/tools/idea/image-20200411100523172.png)

## 将idea中xml文件背景颜色去除

![image-20200411105214882](/images/posts/tools/idea/image-20200411105214882.png)

第一步：除去SQL代码块的背景颜色，步骤如下

![image-20200411105351939](/images/posts/tools/idea/image-20200411105351939.png)

第二步：除去代码背景颜色，步骤如下

![image-20200411105517179](/images/posts/tools/idea/image-20200411105517179.png)

## 实用插件推荐

### 快捷键提示插件

Key promoter是在你通过非快捷键方式使用某功能时 为你提供快捷键建议 在开始记不住快捷键的情况下 强烈推荐安装

### 翻译插件

翻译插件 TranslationPlugin,支持支持中英互译、单词朗读,详细安装文档请参考:[TranslationPlugin介绍与安装手册](https://github.com/YiiGuxing/TranslationPlugin)

### 热部署插件JRebel

JRebel热部署插件安装和使用请参考:[JRebel热部署插件安装和使用](https://youmeek.gitbooks.io/intellij-idea-tutorial/content/jrebel-setup.html)

### Maven Helper

Maven 辅助插件 用于查找Maven依赖冲突非常好用的一款插件 安装步骤请参考:[Maven Helper安装使用](https://my.oschina.net/u/136229/blog/678918)

### Properties to YAML Converter

在开发SpringBoot项目时，会需要把Properties的配置格式改为 YAML格式，Properties to YAML Converter提供了很好的支持

### 阿里巴巴代码规范插件p3c-pmd

详细安装和使用请参考:[阿里巴巴代码规范插件p3c-pmd](http://mp.weixin.qq.com/s/lBDe60LFpdXoi2xbA-L2Qg)

## 开发必备快捷键

IntelliJ IDEA提供了丰富的快捷键组合来加快开发效率，但是快捷键太多琳琅满目也会给人无从下手的感觉。下面是我个人整理的在开发过程中必备的快捷键，(注:IDEA快捷键可能会与其他软件快捷键产生冲突，在开发过程中有必要进行取舍)此外IntelliJ IDEA 官方提供了学习IDEA快捷键的一个插件:IDE Features Trainer:https://plugins.jetbrains.com/plugin/8554?pr=idea，大家可以自行去插件库下载学习

### Ctrl相关

| 快捷键         | 介绍                                                         |
| :------------- | :----------------------------------------------------------- |
| Ctrl + B       | 进入光标所在的方法/变量的接口或是定义处，等效于Ctrl + 左键单击 |
| Ctrl + D       | 复制光标所在行或复制选择内容，并把复制内容插入光标位置下面   |
| Ctrl + F       | 在当前文件进行文本查找                                       |
| Ctrl + H       | 查看类的继承结构                                             |
| Ctrl + N       | 通过类名定位文件                                             |
| Ctrl + O       | 快速重写父类方法                                             |
| Ctrl + P       | 方法参数提示                                                 |
| Ctrl + Y       | 删除光标所在行或删除选中的行                                 |
| Ctrl + W       | 递进式选择代码块                                             |
| Ctrl + Z       | 撤销                                                         |
| Ctrl + 1,2,3…9 | 定位到对应数值的书签位置 结合Ctrl + Shift + 1,2,3…9使用      |
| Ctrl + F1      | 在光标所在的错误代码出显示错误信息                           |
| Ctrl + F12     | 弹出当前文件结构层，可以在弹出的层上直接输入进行筛选         |
| Ctrl + Space   | 基础代码补全默认在Windows系统上被输入法占用，需要进行修改，建议修改为Ctrl + 逗号 |
| Ctrl + /       | 注释光标所在行代码，会根据当前不同文件类型使用不同的注释符号 |

### Alt相关

| 快捷键      | 介绍                                      |
| :---------- | :---------------------------------------- |
| Alt + Q     | 弹出一个提示，显示当前类的声明/上下文信息 |
| Alt + Enter | 根据光标所在问题，提供快速修复选择        |

### Shift相关

| 快捷键     | 介绍                             |
| :--------- | :------------------------------- |
| Shift + F3 | 在查找模式下，定位到上一个匹配处 |

### Ctrl+Alt相关

| 快捷键                | 介绍                                            |
| :-------------------- | :---------------------------------------------- |
| Ctrl + Alt + B        | 在某个调用的方法名上使用会跳到具体的实现处      |
| Ctrl + Alt + L        | 格式化代码 可以对当前文件和整个包目录使用       |
| Ctrl + Alt + M        | 快速抽取方法                                    |
| Ctrl + Alt + O        | 优化导入的类和包 可以对当前文件和整个包目录使用 |
| Ctrl + Alt + T        | 对选中的代码弹出环绕选项弹出层                  |
| Ctrl + Alt + V        | 快速引进变量                                    |
| Ctrl + Alt + F7       | 寻找类或是变量被调用的地方，以弹出框的方式显示  |
| Ctrl + Alt + 左方向键 | 退回到上一个操作的地方                          |
| Ctrl + Alt + 右方向键 | 前进到上一个操作的地方                          |

### Ctrl+Shift相关

| 快捷键                 | 介绍                                                         |
| :--------------------- | :----------------------------------------------------------- |
| Ctrl + Shift + F       | 根据输入内容查找整个项目或指定目录内文件                     |
| Ctrl + Shift + H       | 查看方法的继承结构                                           |
| Ctrl + Shift + J       | 自动将下一行合并到当前行末尾                                 |
| Ctrl + Shift + N       | 通过文件名定位打开文件/目录，打开目录需要在输入的内容后面多加一个正斜杠 |
| Ctrl + Shift + R       | 根据输入内容替换对应内容，范围为整个项目或指定目录内文件     |
| Ctrl + Shift + U       | 对选中的代码进行大/小写轮流转换                              |
| Ctrl + Shift + W       | 递进式取消选择代码块                                         |
| Ctrl + Shift + Z       | 取消撤销                                                     |
| Ctrl + Shift + /       | 代码块注释                                                   |
| Ctrl + Shift + +       | 展开所有代码                                                 |
| Ctrl + Shift + -       | 折叠所有代码                                                 |
| Ctrl + Shift + 1,2,3…9 | 快速添加指定数值的书签                                       |
| Ctrl + Shift + F7      | 高亮显示所有该选中文本，按Esc高亮消失                        |
| Ctrl + Shift + Space   | 智能代码提示                                                 |
| Ctrl + Shift + Enter   | 自动结束代码，行末自动添加分号                               |

### Alt+Shift相关

| 快捷键 | 介绍 |
| :----- | :--- |
|        |      |

### Ctrl+Alt+Shift相关

| 快捷键 | 介绍 |
| :----- | :--- |
|        |      |

### 其他

| 快捷键 | 介绍                             |
| :----- | :------------------------------- |
| F2     | 跳转到下一个高亮错误或警告位置   |
| F3     | 在查找模式下，定位到下一个匹配处 |
| F4     | 编辑源                           |