# Android system/app/与system/priv-app/的区别

　　在system/priv-app目录主要是存放手机厂商定制的系统的系统级应用，比如phone app,settings app，systemui app等，这些应用需要系统及权限，而又不能被用户卸载掉。这个目录是在Android KitKat新增加的分区。在KitKat之前版本在系统分区的所有apks都可以使用系统权限，这个更改使手机厂商能够更好的控制捆绑软件对敏感权限的访问。手机厂商在定制一些系统软件的时候软件也会需要专门给priv-app添加selinux policy。当然应用需要获取系统权限还有其他的办法，在AndroidManifest.xml文件中添加 android:sharedUserId="android.uid.sysytem",同时给该apk添加系统签名，比如小米手机就需要给apk添加小米的系统权限。

　　其实从安全的角度考虑，谷歌也不希望使用WebView控件的system/app/的应用拥有系统权限，比如Chrome,Chrome一直是黑客喜欢利用的攻击点，所以谷歌在代码力会检测使用WebView控件的应用有没有系统权限。贴一段代码：

```java
static WebViewFactoryProvider getProvider() {
	synchronized (sProviderLock) {

// For now the main purpose of this function (and the factory abstraction) is to keep

// us honest and minimize usage of WebView internals when binding the proxy.

	if (sProviderInstance != null) return sProviderInstance;

	final int uid = android.os.Process.myUid();

	if (uid == android.os.Process.ROOT_UID || 
        uid == android.os.Process.SYSTEM_UID || 
        uid == android.os.Process.PHONE_UID || 
        uid == android.os.Process.NFC_UID || 
        uid == android.os.Process.BLUETOOTH_UID) {
		throw new UnsupportedOperationException("For security reasons, WebView is not allowed in privileged processes");
}
```

注意代码中的进程都是具有system权限的。

以上就是system/priv-app/分区的特殊之处。