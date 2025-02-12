{% from "views/_data.njk" import android_groovy, android_key_init %}

# Android SDK 更安全的接入和初始化方式

LeanCloud Android SDK 从 6.1.0 版本开始，除了支持通过 appId + appKey 完成初始化，还提供一种更加安全的使用方式，支持仅仅通过 appId 来初始化应用，避免核心配置信息在客户端泄漏可能带来的潜在风险。

下面我们来详细说明新版本的新集成方式。

## 1 集成准备
在开发应用前，开发者需要到 LeanCloud 官网注册账号并创建应用，这些基本的流程我们在此略过。

### 1.1 生成签名证书指纹

Android 开发者在使用 LeanCloud 服务前必须将签名证书的指纹配置到应用后台，为此需要根据应用打包采用的签名证书在本地生成签名证书指纹。

这需要满足以下 2 个条件：
- 已创建应用程序的签名文件，签名证书创建可参考生成[签名证书](https://developer.android.com/studio/publish/app-signing?hl=zh-CN)。
- 当前开发机器已经安装 JDK。

具体操作步骤如下：
1. 打开命令行工具，执行命令 `keytool -list -v -keystore <keystore-file>`
	其中 `<keystore-file>` 为应用签名文件的完整路径，注意按命令行提示进行操作。
2. 获取对应的 SHA256 指纹，例如：
![images](images/security_android_sign_sha256.jpg)

### 1.2 在 LeanCloud 后台配置应用签名证书指纹

1. 登录 LeanCloud 控制台，选择目标应用。
2. 进入「设置 - 安全中心」，在「Android 安全设置」项下，填入应用包名和前一步获得的 SHA256 指纹。
![android_setting](images/security_android_package_sign.jpg)
3. 点击保存按钮将信息保存到云端。

## 2 集成 Android SDK
### 2.1 获取 SDK
推荐大家使用包依赖管理工具来安装 SDK，可以参考原文档：[SDK 安装指南](start.html)。
修改当前项目的 build.gradle 文件，加入如下依赖：

{{ android_groovy() }}

### 2.2 加入 Native library

新的认证方式，在客户端需要配合 LeanCloud native library 来进行请求签名。

请[点击这里](https://capacity-files.lncld.net/4facc18ba5c5a2ad0baf/leancloud-jniLibs-v2.zip)下载 JNI library，将下载文件在本地解压，可以得到如下文件：
```
jniLibs
├── arm64-v8a
│   └── libleancloud-core.so
├── armeabi
│   └── libleancloud-core.so
├── armeabi-v7a
│   └── libleancloud-core.so
├── mips
│   └── libleancloud-core.so
├── mips64
│   └── libleancloud-core.so
├── x86
│   └── libleancloud-core.so
└── x86_64
    └── libleancloud-core.so
```

将解压之后的 jniLibs 目录，拷贝到应用的 `src/main` 目录下，项目代码结构如下：
```
src/
└── main
    ├── AndroidManifest.xml
    ├── assets
    ├── java
    ├── jniLibs
    │   ├── arm64-v8a
    │   │   └── libleancloud-core.so
    │   ├── armeabi
    │   │   └── libleancloud-core.so
    │   ├── armeabi-v7a
    │   │   └── libleancloud-core.so
    │   ├── mips
    │   │   └── libleancloud-core.so
    │   ├── mips64
    │   │   └── libleancloud-core.so
    │   ├── x86
    │   │   └── libleancloud-core.so
    │   └── x86_64
    │       └── libleancloud-core.so
    └── res
```

> 注意：我们提供的 native library 是支持所有 Android 设备的，你也可以根据自己的需求删掉不必要的指令集文件，这样可以进一步减小安装包的体积。

### 2.3 初始化
与之前的版本不同，新的 SDK 支持只使用 appId 来完成初始化。

首先，需要修改一下 application 的 build.gradle 文件，以支持自动签名：

```groovy
android {
    compileSdkVersion 29
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "xxxx"
        minSdkVersion 21
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    // 增加自动签名的内容
    signingConfigs {
        config {
            keyAlias '{your key alias}'
            keyPassword '{your key password}'
            storeFile file('{your store file full name}')
            storePassword '{your store password}'
        }
    }
    buildTypes {
        debug {
            // 增加签名设置
            signingConfig signingConfigs.config
        }
        release {
            // 增加签名设置
            signingConfig signingConfigs.config
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

然后向 Application 类的 onCreate 方法添加初始化代码：

{{ android_key_init(appid) }}

至此，客户端的所有改动已经完成，开发者可以如以往一样调用 LeanCloud 服务了。


## 3 升级、部署云引擎实例

新的 Android SDK 会使用新的安全方式进行请求签名，与 LeanCloud 云端完成数据交互。
如果开发者没有在客户端直接调用云引擎中的[云函数](leanengine_cloudfunction_guide-node.html)，那么所有的集成都已经完成了，可以忽略本章内容。
如果你有在 Android 客户端调用过云函数（例如代码里有 `LCCloud#callFunctionInBackground` 或 `LCCloud#callRPCInBackground` 请求），为了保证 SDK 发出的请求能被云引擎正确处理，您还需要升级云引擎的 runtime 库并重新部署云引擎实例。

现在支持 Android SDK 新认证方式的云引擎 runtime SDK 版本如下：
- Python SDK：2.3.0 and later
- Node.js SDK：3.5.0 and later
- Java SDK (engine-core)：6.1.0 and later

大家更新云引擎代码依赖的版本，通过 `lean publish` 进行发布即可。

如果你当前使用的 runtime SDK 版本满足要求，那么只需要对云引擎实例进行重新发布即可。

## 4. 最后

新的使用方式只是去掉了 appKey 在客户端的使用，在一定程度上避免了暴露应用核心配置信息，它无法保证绝对的数据安全。我们还是推荐大家通过 ACL 机制来限制不同用户的数据访问权限，从源头降低安全风险，更多细节请参考[数据和安全指南](data_security.html)。

## 附：常见问题
### 1. 6.1.0 版本 SDK 能否支持老的初始化方式？
可以的。如果不使用新的初始化方式，还是可以按以前的方式来使用新版本 SDK 的，并且也不需要在工程中加入 native library。

### 2. 使用新的初始化方式，程序一运行就 crash，是怎么回事？

多半是因为工程中没有加入 native library 导致的，请回看 [2.2 节](#_2-2-加入-native-library)。

### 3. 加入 native library，使用新方式完成初始化之后，所有请求都返回 `{"code":401,"error":"Unauthorized."}` 的错误，是怎么回事？

所有请求都出现 401 的错误，多半是因为打包的 apk （debug 运行也一样）没有正确配置签名证书导致的，请回看 [2.3 节](#_2-3-初始化)。

### 4. 启用了新的初始化方式之后，android 平台上老版本的应用是否还能正常使用？

可以的。设置了 package name 和证书指纹之后，只是为应用增加了一种新的认证方式，并不影响老的认证方式的继续使用。

### 5. 如果多个 android 应用使用了同样的 appid，该怎么处理？

目前我们只支持为一个平台的 app 配置一套包名和证书指纹，会对应到某一种 apk。如果开发者在多个客户端应用中共享同一个平台 app 数据，那么可以选择其中某一个客户端使用新的认证方式，而其他客户端还是使用老的方式，在 Android 平台上多种认证方式是可以共存的。
