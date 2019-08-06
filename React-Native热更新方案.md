# React Native热更新方案

2019.06.28  ningerlei@outlook.com

React Native（下面成为均简写为RN）的出现，整合了移动端APP的开发，不仅缩短了APP开发的时间，也提高了开发的效率，更是为热更新提供了基础（android原生开发中热更新已经是老生常谈了，是比较成熟的，但是iOS的热更新一直是很大的问题）。本片文章主要整理了当前的RN热更新方案。

当前RN的热更新方案主要有**[微软推出的Code Push](https://blog.csdn.net/qq_33323251/article/details/79437932)**和**[RN中文开源的react-native-pushy](https://github.com/reactnativecn/react-native-pushy)**。

关于CodePush，也推荐阅读博客：

https://juejin.im/post/599962f36fb9a0247942a6f1

### Code Push

微软出品，当然也是精品了。

Code Push已经被微软开源在了git上，可以搜索[react-native-code-push](https://link.jianshu.com/?t=https://github.com/Microsoft/react-native-code-push)。

简单的讲解一下Code Push的使用过程。

#### Code Push安装

1. 安装Code Push客户端。

2. 安装Code Push CLI

3. 安装完成之后在Code Push上创建账号

4. 登录账号，在CodePush服务器上注册对应的应用。

   登录账号的相关命令：

   ```
   code-push login 登陆
   code-push logout 注销
   code-push access-key ls 列出登陆的token
   code-push access-key rm <accessKye> 删除某个 access-key
   ```

   

   android和iOS需要注册两个应用

   ```
   code-push app add MyApp-Android
   code-push app add MyApp-iOS
   ```

   app注册完成之后，会返回两套deployments，内容是每个应用对应的deployment key。可以在命令行查看，也可以在控制台查看。

   相关命令：

   ```
   code-push app add 在账号里面添加一个新的app
   code-push app remove 或者 rm 在账号里移除一个app
   code-push app rename 重命名一个存在app
   code-push app list 或则 ls 列出账号下面的所有app
   code-push app transfer 把app的所有权转移到另外一个账号 
   ```

#### Code Push集成

Android：

安装react-native-code-push

```
yarn add react-native-code-push
react-native-link react-native-code-push
//接下来会提示输入delopyment key，如果不输入，点击enter键直接跳过
//自己可以看看文件差异
主要是setting.gradle增加了依赖库，app.gradle修改了配置
```



#### Code Push配置更新

1. 设置更新策略(可以自动更新也可以手动更新)

   在js中家在Code Push模块（自动更新）：

   ```react
   import codePush from 'react-native-code-push'
   
   componentDidMount() {
       //sync方法可以设置参数来配置更新策略。
       CodePush.sync();
       CodePush.sync({installMode:CodePush.InstallMode.IMMEDIATE, mandatoryInstallMode:CodePush.InstallMode.IMMEDIATE});
   }
   ```

   介绍CodePush中几个类：

   ```react
   设置安装的模式
   enum InstallMode{
   	IMMEDIATE,//立即安装
       ON_NEXT_RESTART,//下一次重启时安装
       ON_NEXT_RESUME,//从后台返回前台时安扎ung
       ON_NEXT_SUSPEND//后台安装
   }
   ```

   ```react
   同步的状态，会通过函数回调给用户，具体状态可以参考源码。
   enum SyncStatus{
   	
   }
   ```

   还有其他几个重要的类：参考源码。

2.  发布更新 

   Code Push支持两种发布更新的方式，一种是通过`code-push release-react`简化方式，另一种是通过`code-push release`复杂方式。

   1. **第一种方式**：通过`code-push release-react`（**推荐**）

      这种方式将打包和发布合二为一，简化了操作，也就是代码直接发布更新。

      命令格式：

      `code-push release-react <appName> <platform>`

      eg:

      ```
      code-push release-react MyApp-iOS ios
      code-push release-react MyApp-Android android
      ```

      增加一些参数：

      ```
      code-push release-react MyApp-iOS ios  --t 1.0.0 --dev false --d Production --des "1.优化操作流程" --m true
      ```

      其中参数--t为二进制(.ipa与apk)安装包的的版本；--dev为是否启用开发者模式(默认为false)；--d是要发布更新的环境分Production与Staging(默认为Staging)；--des为更新说明；--m 是强制更新。

      关于`code-push release-react`更多可选的参数，可以在终端输入`code-push release-react`进行查看。

   2. **第二种方式**：通过code-push release发布

      首先先要将js和图片资源打包成bundle。

      生成bundle

      1. 在工程目录新增bundles文件：mkdir bundles
      2. 运行命令打包 react-native bundle --platform 平台 --entry-file 启动文件 -- bundle-output ./bundles/index.android.bundle -- dev false

      

      ![生成bundle](https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/master/React%20Native%E5%BA%94%E7%94%A8%E9%83%A8%E7%BD%B2%E3%80%81%E7%83%AD%E6%9B%B4%E6%96%B0-CodePush%E6%9C%80%E6%96%B0%E9%9B%86%E6%88%90%E6%80%BB%E7%BB%93/images/%E7%94%9F%E6%88%90bundle%E5%8C%85.png)





> 注意更新时如果想用热更新，则不能修改版本号和版本码，否则不能热更新。



#### 代码回滚

当发布的新版本有问题时，也可以将代码回滚到之前的某一个稳定版本

```
code-push rollback MyApp Staging --targetRelease v15
```

其实在后台是将之前的版本重新发布，生成一个新的版本，再推给用户去更新。