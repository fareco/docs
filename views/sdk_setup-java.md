# Java SDK 配置指南

## SDK 维护期说明

我们于 2018 年 9 月推出了新的 [Java Unified SDK](https://leancloudblog.com/java-unified-sdk-kai-fang-ce-shi-tong-zhi/)，兼容纯 Java、云引擎和 Android 等多个平台，老的 Android SDK（版本号低于 `5.0.0`，`groupId` 为 `cn.leancloud.android` 的 libraries）已于 2019 年 9 月底停止维护。

Java Unified SDK 根据底层依赖的 JSON 解析库的不同，有两个不同分支：

- 6.x 分支依赖 [fastjson](https://github.com/alibaba/fastjson) 来进行 JSON 解析；
- 7.0 以后版本使用 [Gson](https://github.com/google/gson) 来进行 JSON 解析（最新版本）；

两个版本的对外接口完全一致，但是考虑到平台兼容性和稳定性，我们推荐大家使用带 Gson 库的版本来进行开发。

从 2021 年 6 月开始，我们推出了 8.0 版本，与 7.x 版本相比，主要的变化是改变了公开类的前缀（`AV` -> `LC`），同时也删除了一些长期处于 `deprecated` 状态的接口，它将是我们今后会长期维护的版本。欢迎老 SDK 的用户尽快切换到新的 Java Unified SDK，具体迁移方法见 [Java SDK 的 README](https://github.com/leancloud/java-unified-sdk#migration-to-8x)。

请开发者注意，各个版本 SDK 的维护期如下：

1. 6.0 以前版本已经停止维护。
2. 原 7.x 版本从现在开始进入维护期（只改 bug，不增加新功能），并计划于 2022 年 6 月停止维护。
3. 8.x 会是我们今后长期维护版本。

> 下面的流程都是基于 8.x 版本 Java Unified SDK 书写.

## 平台与 SDK 对应关系
Java SDK 主要包含以下几个 library，其层次结构以及平台对应关系如下：

### 基础包（可以在纯 Java 环境下调用）
- storage-core：包含所有数据存储的功能，如
  - 结构化数据（LCObject）
  - 内建账户系统（LCUser）
  - 查询（LCQuery）
  - 文件存储（LCFile）
  - 社交关系（LCFriendship，当前版本暂不提供）
  - 朋友圈（LCStatus，当前版本暂不提供）
  - 短信（LCSMS）
  - 等等
- realtime-core：部分依赖 storage-core library，实现了 LiveQuery 以及即时通讯功能，如：
  - LiveQuery
  - LCIMClient
  - LCIMConversation 以及多种场景对话
  - LCIMMessage 以及多种子类化的多媒体消息
  - 等等

### Android 特有的包
- storage-android：是 storage-core 在 Android 平台的定制化实现，接口与 storage-core 完全相同。
- realtime-android：是 realtime-core 在 Android 平台的定制化实现，并且增加 Android 推送相关接口。
- mixpush-android：是 LeanCloud 混合推送的 library，支持华为、小米、魅族、vivo 以及 oppo 的官方推送。
- leancloud-fcm：是 Firebase Cloud Messaging 的封装 library，供美国节点的 app 使用推送服务。


### 模块依赖关系
Java SDK 一共包含如下几个模块：

目录 | 模块名 | 适用平台 | 依赖关系
---|---|---|---
./core | storage-core，存储核心 library | java | 无，它是 LeanCloud 最核心的 library
./realtime | realtime-core，LiveQuery 与实时通讯核心 library | java | storage-core
./android-sdk/storage-android | storage-android，Android 存储 library | Android | storage-core
./android-sdk/realtime-android | realtime-android，Android 推送、LiveQuery、即时通讯 library | Android | storage-android, realtime-core
./android-sdk/mixpush-android | Android 混合推送 library | Android | realtime-android
./android-sdk/leancloud-fcm | Firebase Cloud Messaging library | Android | realtime-android

## 获取 SDK

获取 SDK 有多种方式，较为推荐的方式是通过包依赖管理工具下载最新版本。

我们已经将所有的 library 发布到了 maven 中心仓库，开发者可以用以下任意包管理工具来安装 SDK，并可以在[公开的 release 列表页](https://github.com/leancloud/java-unified-sdk/releases)查看最新版本。

#### 使用存储功能

Maven：

```xml
<dependency>
    <groupId>cn.leancloud</groupId>
    <artifactId>storage-core</artifactId>
    <version>8.1.2</version>
</dependency>
```

Ivy：

```xml
<dependency org="cn.leancloud" name="storage-core" rev="8.1.2" />
```

SBT：

```scala
libraryDependencies += "cn.leancloud" %% "storage-core" % "8.1.2"
```

Gradle：

```groovy
implementation 'cn.leancloud:storage-core:8.1.2'
```

如果是 Android 项目，则换成以下这些包：

```groovy
implementation 'cn.leancloud:storage-android:8.1.2'
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
```

#### 使用即时通讯 / 推送服务

Maven：

```xml
<dependency>
    <groupId>cn.leancloud</groupId>
    <artifactId>realtime-core</artifactId>
    <version>8.1.2</version>
</dependency>
```

Ivy:

```xml
<dependency org="cn.leancloud" name="realtime-core" rev="8.1.2" />
```

SBT:

```scala
libraryDependencies += "cn.leancloud" %% "realtime-core" % "8.1.2"
```

Gradle:

```groovy
implementation 'cn.leancloud:realtime-android:8.1.2'
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
```

#### 使用混合推送服务

Gradle：

```groovy
implementation 'cn.leancloud:mixpush-android:8.1.2'
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
```


#### 对 maven 源的特别说明

我们发现有时候 Maven 源的 CDN 缓存同步策略出现问题，可能会导致我们某个版本或者该版本下某种格式的 library 文件无法下载，这时候你可以在配置文件中显式增加一个 Sonatype 的源，就可以解决找不到文件的问题。

Maven pom.xml 的修改示例如下：

```xml
<repositories>
    <repository>
        <id>oss-sonatype</id>
        <name>oss-sonatype</name>
        <url>https://oss.sonatype.org/content/groups/public/</url>
    </repository>
</repositories>
```

Gradle build.gradle 的修改示例如下：

```groovy
buildscript {
    repositories {
        google()
        jcenter()
        // 增加下面的配置
        maven {
            url "https://oss.sonatype.org/content/groups/public/"
        }
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        // 增加下面的配置
        maven {
            url "https://oss.sonatype.org/content/groups/public/"
        }
    }
}
```

### 手动安装

#### 从源码编译

可以执行以下命令获取 Java SDK 并安装：

```sh
$ git clone https://github.com/leancloud/java-unified-sdk.git
$ cd java-unified-sdk/
$ mvn clean install
```

获取和安装 Android SDK：

```sh
$ cd java-unified-sdk/
$ cd android-sdk/
$ gradle clean assemble
```

## 初始化

首先进入 **云服务控制台 > 设置 > 应用凭证** 来获取 **App ID**，**App Key** 以及**服务器地址**。

### Android 平台初始化

如果是一个 Android 项目，则向 `Application` 类的 `onCreate` 方法添加：

```java
import cn.leancloud.LeanCloud;

public class MyLeanCloudApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // 提供 this、App ID、App Key、Server Host 作为参数
        // 注意这里千万不要调用 cn.leancloud.core.LeanCloud 的 initialize 方法，否则会出现 NetworkOnMainThread 等错误。
        LeanCloud.initialize(this, "{{appid}}", "{{appkey}}", "https://please-replace-with-your-customized.domain.com");
    }
}
```

请将 `https://please-replace-with-your-customized.domain.com` 替换为你的应用[绑定的 API 域名](custom-api-domain-guide.html#API_域名)。

国际版应用不要求绑定自定义域名。
如果你的国际版应用（App ID 后缀为 `-MdYXbMMI`）没有绑定自定义域名，**初始化 SDK 时不用传入服务器地址参数**。
极个别　App ID　后缀不为 `-MdYXbMMI` 的国际版应用，请参见[这里的说明](custom-api-domain-guide.html#App_ID_后缀不为_-MdYXbMMI_的国际版应用如何初始化_SDK)。

然后指定 SDK 需要的权限并在 `AndroidManifest.xml` 里面声明 `MyLeanCloudApp` 类：

```xml
<!-- 基本模块（必须）START -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<!-- 基本模块 END -->

<application
  …
  android:name=".MyLeanCloudApp" >

  <!-- 即时通讯和推送 START -->
  <!-- 即时通讯和推送都需要 PushService -->
  <service android:name="cn.leancloud.push.PushService"/>
  <receiver android:name="cn.leancloud.push.LCBroadcastReceiver">
    <intent-filter>
      <action android:name="android.intent.action.BOOT_COMPLETED"/>
      <action android:name="android.intent.action.USER_PRESENT"/>
      <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
  </receiver>
  <!-- 即时通讯和推送 END -->
</application>

```

注意：对于**只使用即时通讯服务**，而**不使用推送服务**的应用来说，在初始化的时候设置 LCIMOptions#disableAutoLogin4Push 选项，可以加快即时通讯用户登录的过程。设置方法如下：

```java
// 在 LeanCloud#initialize 之后调用，禁止自动发送推送服务的 login 请求。
LCIMOptions.getGlobalOptions().setDisableAutoLogin4Push(true);
```

#### 更安全的客户端初始化方法

对 Android 开发者来说，从 6.1.0 版本开始，除了支持通过 appId + appKey 完成初始化，我们还提供一种更加安全的使用方式，支持仅仅通过 appId 来初始化应用，例如：

```java
import cn.leancloud.LeanCloud;

public class MyLeanCloudApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // 提供 this、App ID、绑定的自定义 API 域名作为参数
        LeanCloud.initializeSecurely(this, "{{appid}}", "https://please-replace-with-your-customized.domain.com");
    }
}
```

这时候程序初始化不再需要 appKey，避免了核心配置信息在客户端泄漏可能带来的潜在风险。具体的集成方法可参看文档 [Android SDK 更安全的接入和初始化方式](sdk_setup_android_securely.html)。


### Java 平台初始化代码

如果是一个普通 Java 项目，则在代码开头添加：

```java
import cn.leancloud.core.LeanCloud;

LeanCloud.initialize("{{appid}}", "{{appkey}}", "https://please-replace-with-your-customized.domain.com");
```

注意，云引擎内部访问 API 是通过内网，所以不需要也不应该配置 API 自定义域名（`serverUrl`）。
模板项目和《云引擎网站托管开发指南》中的示例代码均未配置 API 自定义域名，
请勿调用 `setServer`，否则会变成公网访问，影响性能。

realtime-core library 也支持在纯 Java Application 中使用，但是与 Android 的调用方式有细微差异，Java Application 中需要开发者显式建立与云端的长链接（Android 平台是通过 PushService 自动建立的）。建立长链接的方法如下：

```java
LCConnectionManager.getInstance().startConnection(new LCCallback() {
  @Override
  protected void internalDone0(Object o, LCException e) {
    if (e == null) {
      System.out.println("成功建立 WebSocket 链接");
    } else {
      System.out.println("建立 WebSocket 链接失败：" + e.getMessage());
    }
  }
});
```

只有长链接成功建立之后，后续的聊天请求才能开始。

## 开启调试日志

在应用开发阶段，你可以选择开启 SDK 的调试日志（debug log）来方便追踪问题。调试日志开启后，SDK 会把网络请求、错误消息等信息输出到 IDE 的日志窗口，或是浏览器 Console 或是云引擎日志（如果在云引擎下运行 SDK）。

```java
// 在 LeanCloud.initialize() 之前调用
LeanCloud.setLogLevel(LCLogger.Level.DEBUG);
```

详细调试流程可以参考[Android SDK 调试指南][android-debug-guide]。

[android-debug-guide]: https://forum.leancloud.cn/t/leancloud-sdk-android-sdk/21829

注意，在应用发布之前，请关闭调试日志，以免暴露敏感数据。

## 验证

首先，确认本地网络环境是可以访问云端服务器的，可以执行以下命令：

```sh
curl "https://{{host}}/1.1/date"
```

`{{host}}` 为绑定的 API 自定义域名。

如果当前网路正常会返回当前时间：

```json
{"__type":"Date","iso":"2020-10-12T06:46:56.000Z"}
```

然后在项目中编写如下测试代码：

```java
LCObject testObject = new LCObject("TestObject");
testObject.put("words", "Hello world!");
testObject.saveInBackground().blockingSubscribe();
```

保存后运行程序。

然后打开 **云服务控制台 > 数据存储 > 结构化数据 > `TestObject`**，如果看到数据表中出现一行「words」列的值为「Hello world!」的数据，说明 SDK 已经正确地执行了上述代码，配置完毕。

如果控制台没有发现对应的数据，请参考 [问题排查](#问题排查)。

## 问题排查

SDK 安装指南基于当前最新版本的 SDK 编写，所以排查问题前，请先检查下安装的 SDK 是不是最新版本。

### `401 Unauthorized`

如果 SDK 抛出 `401` 异常或者查看本地网络访问日志存在：

```json
{
  "code": 401,
  "error": "Unauthorized."
}
```

则可认定为 App ID 或者 App Key 输入有误，或者是不匹配，很多开发者同时注册了多个应用，导致拷贝粘贴的时候，用 A 应用的 App ID 匹配 B 应用的 App Key，这样就会出现服务端鉴权失败的错误。

### 客户端无法访问网络

客户端尤其是手机端，应用在访问网络的时候需要申请一定的权限。

## Android 代码混淆

为了保证 SDK 在代码混淆后能正常运作，需要保证部分类和第三方库不被混淆，参考下列配置：

```
# proguard.cfg

-keepattributes Signature
-dontwarn com.jcraft.jzlib.**
-keep class com.jcraft.jzlib.**  { *;}

-dontwarn sun.misc.**
-keep class sun.misc.** { *;}

-dontwarn com.alibaba.fastjson.**
-keep class com.alibaba.fastjson.** { *;}

-dontwarn org.ligboy.retrofit2.**
-keep class org.ligboy.retrofit2.** { *;}

-dontwarn io.reactivex.rxjava2.**
-keep class io.reactivex.rxjava2.** { *;}

-dontwarn sun.security.**
-keep class sun.security.** { *; }

-dontwarn com.google.**
-keep class com.google.** { *;}

-dontwarn com.avos.**
-keep class com.avos.** { *;}

-dontwarn cn.leancloud.**
-keep class cn.leancloud.** { *;}

-keep public class android.net.http.SslError
-keep public class android.webkit.WebViewClient

-dontwarn android.webkit.WebView
-dontwarn android.net.http.SslError
-dontwarn android.webkit.WebViewClient

-dontwarn android.support.**

-dontwarn org.apache.**
-keep class org.apache.** { *;}

-dontwarn org.jivesoftware.smack.**
-keep class org.jivesoftware.smack.** { *;}

-dontwarn com.loopj.**
-keep class com.loopj.** { *;}

-dontwarn com.squareup.okhttp.**
-keep class com.squareup.okhttp.** { *;}
-keep interface com.squareup.okhttp.** { *; }

-dontwarn okio.**

-dontwarn org.xbill.**
-keep class org.xbill.** { *;}

-keepattributes *Annotation*

```
