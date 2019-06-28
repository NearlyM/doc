# targetsdk切换到28

2019.04.16 ningerlei@outlook.com

### 1、 Notification

通知栏增加了通道号。

```java
String channelId = "02";
String channelName = "system";
if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
    NotificationChannel channel = new NotificationChannel(channelId, channelName, android.app.NotificationManager.IMPORTANCE_HIGH);
    android.app.NotificationManager manager = (android.app.NotificationManager) BaseApplication.get().getSystemService(Context.NOTIFICATION_SERVICE);
    manager.createNotificationChannel(channel);
}

NotificationCompat.Builder builder = new NotificationCompat.Builder(BaseApplication.mContext);
        builder.setAutoCancel(true);
        builder.setChannelId(channelId);
```

### 2、跨应用访问文件

`Uri.fromFile()`在android7.0上不能再继续使用了，会产生崩溃，可见该方式google认为有多不安全。推荐使用`FileProvider`来进行进程间的文件共享。`FileProvider`继承自`ContentProvider`。

下面介绍如何使用FileProvider：

1. 访问FileProvider，需要授予临时权限。

```java
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="xxx.xxx.xxx.fileprovider"
    android:grantUriPermissions="true"
    android:exported="false">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

其中的`xxx.xxx.xxx`为项目包名。

2. 创建xml/file_paths

`file_path.xml`文件内容如下：声明想要共享的路径，path是指要分享的文件路径，name是用来替换掉path，然后对外抛出uri。

举个例子：

```xml
<!-- /storage/emulated/0/HuaweiCamera/id_1tlawfor7ktu/RecordFiles/video/6eee37d259336e5c5a2b2fc19bc464d2_ch01_20190416_174124.dav
-->
    <external-cache-path name="hw_record" path="HuaweiCamera" />

<!--
content://com.huawei.ipc.fileprovider/root_path/storage/emulated/0/HuaweiCamera/id_1tlawfor7ktu/RecordFiles/video/6eee37d259336e5c5a2b2fc19bc464d2_ch01_20190416_174124.dav
-->
```



```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="hw_record" path="HuaweiCamera/RecordFiles" />
    <root-path name="root_path" path="."/>
</paths>
```

下面是对各种目录的介绍。

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">

    <!--代表外部存储区域的根目录下的文件 Environment.getExternalStorageDirectory()/DCIM/camerademo目录-->
    <external-path name="hm_DCIM" path="DCIM/camerademo" />
    <!--代表外部存储区域的根目录下的文件 Environment.getExternalStorageDirectory()/Pictures/camerademo目录-->
    <external-path name="hm_Pictures" path="Pictures/camerademo" />
    <!--代表app 私有的存储区域 Context.getFilesDir()目录下的images目录 /data/user/0/com.hm.camerademo/files/images-->
    <files-path name="hm_private_files" path="images" />
    <!--代表app 私有的存储区域 Context.getCacheDir()目录下的images目录 /data/user/0/com.hm.camerademo/cache/images-->
    <cache-path name="hm_private_cache" path="images" />
    <!--代表app 外部存储区域根目录下的文件 Context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)目录下的Pictures目录-->
    <!--/storage/emulated/0/Android/data/com.hm.camerademo/files/Pictures-->
    <external-files-path name="hm_external_files" path="Pictures" />
    <!--代表app 外部存储区域根目录下的文件 Context.getExternalCacheDir目录下的images目录-->
    <!--/storage/emulated/0/Android/data/com.hm.camerademo/cache/images-->
    <external-cache-path name="hm_external_cache" path="" />
</paths>
```

### 3、EncodingUtils被废弃，导致访问付费页面崩溃

### 4、AccountManager

为了保护用户隐私，在android8.0（26）开始，仅通过申请`GET_ACCOUNTS`权限是不能获取到账户信息的。

需要使用`AccountManager.newChooseAccountIntent()`去选择用户，然后再通过`AccountManager.getAccounts()`来获取。具体使用方式如下：

```java
Intent intent = AccountManager.newChooseAccountIntent(
        null,
        null,
        new String[]{"com.huawei.hwid"},
        "",
        "",
        null,
        null);
startActivityForResult(intent, PICK_ACCOUNT_REQUEST);
```

```java
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == PICK_ACCOUNT_REQUEST) {
        AccountManager systemAccountManager = AccountManager.get(this);
        Account[] accs = systemAccountManager.getAccountsByType("com.huawei.hwid");
    }
}
```

### 5、发现一个新问题（**非targetsdk相关问题**）

华为mate20 pro全面屏手势操作，当返回桌面的时候发送的广播中携带的数据

```java
private static final String SYSTEM_DIALOG_REASON_KEY = "reason";
private static final String SYSTEM_DIALOG_REASON_RECENT_APPS = "recentapps";
private static final String SYSTEM_DIALOG_REASON_HOME_KEY = "homekey";

String reason = intent.getStringExtra(SYSTEM_DIALOG_REASON_KEY);
```

> 华为手机全面屏返回桌面的时候携带的数据是`recentapps`，即任务栏，而非`homekey`，调出屏幕内的按钮，通过按钮操作一切正常。同对比三星全面屏手机，返回桌面等手势操作发送广播数据正常。（**应该是华为手机的Bug**）

